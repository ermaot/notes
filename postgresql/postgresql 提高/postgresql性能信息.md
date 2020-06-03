postgresql在运行期间，会大量收集postgresql数据库、表、索引的统计信息，查询优化器通过这些统计信息估算查询统计时间，然后选择最快的查询路径。这些统计信息都保存在postgresql的系统表中，以pg_stat或pg_statio开头。这些信息分为两类：

1. 支撑数据库内部方法的决策数据，如决定何时运行autovacuum和如何解释查询计划，这些保存在pg_statistics中，超级用户可读，普通用户只能从pg_stats视图中查询
2. 统计数据用于检测数据库级、表级、语句级的信息

视图|说明
---|---
pg_stat_activity |查看当前活动会话状态的视图
pg_stat_bgwriter |只有一行数据，显示集群内后台写的相关情况，记录一些checkpoint ,buffer 的信息 showing cluster-wide statistics from the background writer
pg_stat_database | 显示集群内数据库信息的视图
pg_stat_all_tables |记录当前数据库中所有表的统计信息，包括（toast表）
pg_stat_all_indexes |记录当前数据库中所有的索引的使用情况
pg_stat_sys_indexes | 记录当前数据库中所有系统表的索引的使用情况
pg_stat_user_indexes|记录当前数据库中所有用户表的索引的使用情况
pg_stat_database_conflicts |每个数据库一行数据，记录数据库里面冲突信息，记录由于冲突而导致被取消掉是查询语句的次数
pg_stat_replication |记录复制的相关信息，包括复制用的用户名，复制类型，同步状态 等
pg_stat_sys_tables |与 pg_stat_all_tables相似，不过只是记录了系统表的统计信息
pg_stat_user_tables |与 pg_stat_all_tables相似，不过只是记录了用户自己建的表统计信息
pg_stat_xact_all_tables| 与 pg_stat_all_tables相似，记录所有表在当前会话中的统计信息，仅统计当前会话发生在表上的统计统计信息
pg_stat_xact_sys_tables|与 pg_stat_sys_tables相似，记录系统表表在当前会话中的统计信息，仅统计当前会话发生在表上的统计统计信息
pg_stat_xact_user_tables|与 pg_stat_user_tables相似，记录系统表表在当前会话中的统计信息，仅统计当前会话发生在表上的统计统计信息
pg_statio_all_tables|记录当前数据库中所有表的IO信息，包括（toast表），包括堆栈块的读取数，堆栈块命中的次数，索引块读取数，索引块命中的次数等
pg_statio_sys_tables|与 pg_statio_all_tables相似,只是记录系统表IO的信息
pg_statio_user_tables|与 pg_statio_all_tables相似,只是记录用户表IO的信息
pg_statio_all_indexes | 记录当前数据库中所有索引的IO信息，其中数值idx_blks_read 跟 idx_blks_hit 与 pg_statio_all_tables 中索引的读取跟命中是一致的
pg_statio_sys_indexes | 与 pg_statio_all_indexes 类似，只是记录系统表的索引的IO信息
pg_statio_user_indexes | 与 pg_statio_user_indexes 类似，只是记录用户表的索引的IO信息
pg_statio_all_sequences | 记录当前库中所有序列的读取数跟命中数
pg_statio_sys_sequences |与 pg_statio_all_sequences 类似，只是展示系统建立序列的IO信息而已
pg_statio_user_sequences |与 pg_statio_all_sequences 类似，只是展示用户建立序列的IO信息而已
pg_stat_user_functions | 记录用户创建的函数的统计信息,只有开启了 track_functions = all ，才会有数据
pg_stat_xact_user_functions |显示当前会话中所使用的函数的统计信息

### pg_stat_database

```
# \d  pg_stat_database;
                     View "pg_catalog.pg_stat_database"
     Column     |           Type           | Collation | Nullable | Default 
----------------+--------------------------+-----------+----------+---------
 datid          | oid                      |           |          | 
 datname        | name                     |           |          | 
 numbackends    | integer                  |           |          | 
 xact_commit    | bigint                   |           |          | 
 xact_rollback  | bigint                   |           |          | 
 blks_read      | bigint                   |           |          | 
 blks_hit       | bigint                   |           |          | 
 tup_returned   | bigint                   |           |          | 
 tup_fetched    | bigint                   |           |          | 
 tup_inserted   | bigint                   |           |          | 
 tup_updated    | bigint                   |           |          | 
 tup_deleted    | bigint                   |           |          | 
 conflicts      | bigint                   |           |          | 
 temp_files     | bigint                   |           |          | 
 temp_bytes     | bigint                   |           |          | 
 deadlocks      | bigint                   |           |          | 
 blk_read_time  | double precision         |           |          | 
 blk_write_time | double precision         |           |          | 
 stats_reset    | timestamp with time zone |           |          | 
```

1. numbackends:当前有多少并发连接，为CPU核数的1.5左右最好
2. blks_read，blks_hit读取磁盘块的次数和这些块的缓存命中次数：blks_hit/(blks_read+blks_hit)为缓存命中次数
3. xact_commit，xact_rollback提交和回滚的次数
4. deadlocks：从上次stats_reset以来的死锁次数

```
# select pg_stat_reset();
 pg_stat_reset 
---------------
 
(1 row)
重置stats_reset值
```

### pg_stat_user_tables

```
 \d pg_stat_user_tables;
                      View "pg_catalog.pg_stat_user_tables"
       Column        |           Type           | Collation | Nullable | Default 
---------------------+--------------------------+-----------+----------+---------
 relid               | oid                      |           |          | 
 schemaname          | name                     |           |          | 
 relname             | name                     |           |          | 
 seq_scan            | bigint                   |           |          | 
 seq_tup_read        | bigint                   |           |          | 
 idx_scan            | bigint                   |           |          | 
 idx_tup_fetch       | bigint                   |           |          | 
 n_tup_ins           | bigint                   |           |          | 
 n_tup_upd           | bigint                   |           |          | 
 n_tup_del           | bigint                   |           |          | 
 n_tup_hot_upd       | bigint                   |           |          | 
 n_live_tup          | bigint                   |           |          | 
 n_dead_tup          | bigint                   |           |          | 
 n_mod_since_analyze | bigint                   |           |          | 
 last_vacuum         | timestamp with time zone |           |          | 
 last_autovacuum     | timestamp with time zone |           |          | 
 last_analyze        | timestamp with time zone |           |          | 
 last_autoanalyze    | timestamp with time zone |           |          | 
 vacuum_count        | bigint                   |           |          | 
 autovacuum_count    | bigint                   |           |          | 
 analyze_count       | bigint                   |           |          | 
 autoanalyze_count   | bigint                   |           |          | 
```

主要关注seq_scan和idx_scan。尽量少使用seq_scan，最好使得索引扫描的比率接近1，即idx_scan/(idx_scan+seq_scan)接近1



#### pg_stat_statements

语句级别的统计信息通过pg_stat_statements,postgres日志，auto_explain来获取

开启pg_stat_statements,先需要在postgresql.conf中设置

```
shared_preload_libraries='pg_stat_statements'
pg_stat_statements.track=all
```

然后开启extension