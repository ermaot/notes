崩溃恢复函数入口为 innobase_start_or_create_for_mysql

## 初始化崩溃恢复
- 首先初始化崩溃恢复所需要的内存对象
```
recv_sys_create();
recv_sys_init(buf_pool_get_curr_size());
```
- 当InnoDB正常shutdown，在flush redo log 和脏页后，会做一次完全同步的checkpoint，并将checkpoint的LSN写到ibdata的第一个page中（fil_write_flushed_lsn）。
- 在重启实例时，会打开系统表空间ibdata，并读取存储在其中的LSN：
```
err = srv_sys_space.open_or_create(
false, &sum_of_new_sizes, &flushed_lsn);
```
1. 上述调用将从ibdata中读取的LSN存储到变量flushed_lsn中，表示上次shutdown时的checkpoint点，在后面做崩溃恢复时会用到。
2. 另外这里也会将double write buffer内存储的page载入到内存中(buf_dblwr_init_or_load_pages)，如果ibdata的第一个page损坏了，就从dblwr中恢复出来。

```
注意在MySQL 5.6.16之前的版本中，如果InnoDB的表空间第一个page损坏了，就认为无法确定这个表空间的space id，也就无法决定使用dblwr中的哪个page来进行恢复，InnoDB将崩溃恢复失败(bug#70087)，
由于每个数据页上都存储着表空间id，因此后面将这里的逻辑修改成往后多读几个page，并尝试不同的page size，直到找到一个完好的数据页， (参考函数Datafile::find_space_id())。因此为了能安全的使用double write buffer保护数据，建议使用5.6.16及之后的MySQL版本
```
## 恢复truncate操作
- 为了保证对 undo log 独立表空间和用户独立表空间进行 truncate 操作的原子性，InnoDB 采用文件日志的方式为每个 truncate 操作创建一个独特的文件，如果在重启时这个文件存在，说明上次 truncate 操作还没完成实例就崩溃了，在重启时，我们需要继续完成truncate操作。
- 这一块的崩溃恢复是独立于redo log系统之外的。对于 undo log 表空间恢复，在初始化 undo 子系统时完成
```
err = srv_undo_tablespaces_init(
        create_new_db,
        srv_undo_tablespaces,
        &srv_undo_tablespaces_open);
```
- 对于用户表空间，扫描数据目录，找到 truncate 日志文件：如果文件中没有任何数据，表示truncate还没开始；如果文件中已经写了一个MAGIC NUM，表示truncate操作已经完成了；这两种情况都不需要处理
```
err = TruncateLogParser::scan_and_parse(srv_log_group_home_dir);

```
- 对用户表空间truncate操作的恢复是redo log apply完成后才进行的，这主要是因为恢复truncate可能涉及到系统表的更新操作（例如重建索引），需要在redo apply完成后才能进行

## redo log 恢复过程
入口函数： err = recv_recovery_from_checkpoint_start(flushed_lsn)。传递的参数flushed_lsn即为从ibdata第一个page读取的LSN，主要包含以下几步
- 为每个buffer pool instance创建一棵红黑树，指向buffer_pool_t::flush_rbt，主要用于加速插入flush list (buf_flush_init_flush_rbt)；
-  读取存储在第一个redo log文件头的CHECKPOINT LSN，并根据该LSN定位到redo日志文件中对应的位置，从该checkpoint点开始扫描。在这里会调用三次recv_group_scan_log_recs扫描redo log文件：
1. 第一次扫描找到正确的MLOG_CHECKPOINT位置（MySQL8放弃了MLOG_CHECKPOINT解决方案）
2. 第二次扫描解析 redo 日志并存储到hash中
3. 如果hash空间不够用，则再来一轮重新开始，解析一批，应用一批。


## 初始化事务子系统（trx_sys_init_at_db_start）
```
trx_sys_init_at_db_start
    --> trx_sysf_get -->
        ....->buf_page_io_complete --> recv_recover_page
```
在初始化回滚段的时候，通过读入回滚段页并进行redo log  apply，就可以将回滚段信息恢复到一致的状态

## 应用redo日志
入口函数recv_apply_hashed_log_recs。根据之前搜集到recv_sys->addr_hash中的日志记录，依次将page读入内存，并对每个page进行崩溃恢复操作（recv_recover_page_func）
- 已经被删除的表空间，直接跳过其对应的日志记录；
- 在读入需要恢复的文件页时，会主动尝试采用预读的方式多读点page (recv_read_in_area)，搜集最多连续32个（RECV_READ_AHEAD_AREA）需要做恢复的page no，然后发送异步读请求。 page 读入buffer pool时，会主动做崩溃恢复逻辑；
- 只有LSN大于数据页上LSN的日志才会被apply; 忽略被truncate的表的redo日志；
- 在恢复数据页的过程中不产生新的redo 日志；
- 在完成修复page后，需要将脏页加入到buffer pool的flush list上；
- innodb需要保证flush list的有序性，而崩溃恢复过程中修改page的LSN是基于redo 的LSN而不是全局的LSN，无法保证有序性，所以InnoDB另外维护了一颗红黑树来维持有序性，每次插入到flush list前，查找红黑树找到合适的插入位置，然后加入到flush list上。（buf_flush_recv_note_modification）


## 完成崩溃恢复
函数recv_recovery_from_checkpoint_finish。在完成所有redo日志apply后，基本的崩溃恢复也完成了，此时可以释放资源，等待recv writer线程退出 (崩溃恢复产生的脏页已经被清理掉)，释放红黑树，回滚所有数据词典操作产生的非prepare状态的事务 (trx_rollback_or_clean_recovered)


## 无效数据清理及事务回滚
调用函数recv_recovery_rollback_active完成下述工作：
1. 删除临时创建的索引，例如在DDL创建索引时crash时的残留临时索引(row_merge_drop_temp_indexes())；
2. 清理InnoDB临时表 (row_mysql_drop_temp_tables)；
3. 清理全文索引的无效的辅助表(fts_drop_orphaned_tables())；
4. 创建后台线程，线程函数为trx_rollback_or_clean_all_recovered，和在recv_recovery_from_checkpoint_finish中的调用不同，该后台线程会回滚所有不处于prepare状态的事务。
然后Server层的binlog联合来进行崩溃恢复

## Binlog/InnoDB XA Recover
Server层，在初始化完了各个存储引擎后，如果binlog打开了，我们就可以通过binlog来进行XA恢复:
1. 首先扫描最后一个binlog文件，找到其中所有的XID事件，并将其中的XID记录到一个hash结构中（MYSQL_BIN_LOG::recover）；
2. 然后对每个引擎调用接口函数xarecover_handlerton, 拿到每个事务引擎中处于prepare状态的事务xid，如果这个xid存在于binlog中，则提交；否则回滚事务。

- 如果我们弱化配置的持久性(innodb_flush_log_at_trx_commit != 1 或者 sync_binlog != 1)， 宕机可能导致两种丢数据的场景：
1. 引擎层提交了，但binlog没写入，备库丢事务；
2. 引擎层没有prepare，但binlog写入了，主库丢事务（这个可能吗？）

- innodb_flush_log_at_trx_commit =1 和 sync_binlog = 1而MySQL崩溃，提升备库为主库，同样会导致主备不一致。可用semi sync解决：
1. 设置双1强持久化配置;
2. 我们将semisync的超时时间设到极大值，同时使用semisync AFTER_SYNC模式，即用户线程在写入binlog后，引擎层提交前等待备库ACK；
3. 基于步骤1的配置，我们可以保证在主库crash时，所有老主库比备库多出来的事务都处于prepare状态；
4. 备库完全apply日志后，记下其执行到的relay log对应的位点，然后将备库提升为新主库；
5. 将老主库的最后一个binlog进行截断，截断的位点即为步骤3记录的位点;
6. 启动老主库，那些已经传递到备库的事务都会提交掉，未传递到备库的binlog都会回滚掉。