## mysql体系结构
参考来源：
- MYSQL技术内幕：姜承尧
- MySQL实战45讲：：丁奇（林晓斌）-极客时间
### mysql物理结构
1. 连接池
1. 管理服务于工具
1. SQL接口
1. 查询分析（语法分析，词法分析）
1. 优化器
1. 缓存
1. 存储引擎
1. 物理文件
<img src="pic/mysql%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%841.png" alt="姜承尧-mysql物理结构" style="zoom:200%;" />
对照
### mysql执行流程
![丁奇-mysql结果与执行流程](pic/mysql%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%842.jpg)
1.mysql客户端对server监听端口发起请求
2.在连接组件层创建连接、分配线程、验证用户名、密码和库表权限
3.如果打开了query_cache，则先查询缓存，有数据则直接返回，没有则继续执行
4.SQL接口组件接收SQL语句，将SQL语句分解成数据结构（分析器，包括词法分析、语法分析）
5.查询优化器生成查询路径数，并选取一条最优查询路径
6.调用存储引擎接口，打开表，执行查询，检查存储引擎缓存中是否有对应的缓存记录
7.到磁盘物理文件中寻找数据
8.查询到数据后写入存储引擎缓存中，如果有query_cache，也写入
9.返回数据给客户端
10.依次关闭表、线程、连接

### 存储引擎
#### - innodb（==最常用==）
1. 支持事务，面向OLTP
1. 行锁（相对于MyISAM的表所）
1. 支持外键
2. 非锁定读
3. 数据放逻辑表空间
4. 可将表放独立ibd文件中（MySQL>4.1）
5. 支持裸设备
6. MVCC
7. SQL标准==4种隔离级别==，READ REPEATABLE为默认，==next-key locking==
8. 插入缓冲（insert buffer），二次写（double write），自适应哈希索引（adaptive hash index），预读（read ahead）
9. 聚集索引（表按主键顺序存储）
10.若无主键，自动6字节ROWID作为主键
#### - MyISAM
1. 不支持事务
1. 表锁（相对于innodb）
1. 支持全文索引
1. 缓存索引文件而非数据文件
1. 面向OLAP

## innodb内存结构
- 1.buffer pool：
1. innodb启动时分配，用于访问数据时缓存表和索引数据
2. 可以合并一些经常访问的数据
3. 在专用的数据库服务器上，可以将80%左右的物理内存分配给innodb
4. 页面链表的方式+LRU算法管理
- 2.change buffer:
1. 一种特殊的数据结构（之前叫insert buffer）
2. 如果页不在缓冲池中，则缓存对辅助索引页的更改（insert、update、delete）
3. 当其他读取操作从磁盘加载数据页，可以将操作合并
- 3.adaptive hash index
用于管理缓冲池的内部数据结构
- 4.redo log buffer
1. 重做日志缓冲，用于保存重做日志文件中的数据的内存缓冲区
2. 由innodb_log_buffer_size配置
3. 定期刷盘到日志文件（innodb_flush_log_at_timeout控制刷新频率）
4. 如果场景有大事务，可增加日志缓冲区
5. innodb_flush_log_at_commit参数控制如何刷盘，如果为1，则事务提交时会写入日志文件中

## innodb 磁盘结构
- 1.system tablespace
1. 包括innodb data dictionary，double write buffer磁盘部分和undo logs，表和索引数据
2. 称为系统表空间，可以被用户共享
3. 由一个或者多个数据文件构成
4. 默认情况只创建一个ibdata1的共享表空间文件，可以由innodb_data_file_path选项控制共享表空间的文件数量和大小
- 2.data dictionary
1. innodb数据字典由内部系统表组成，包含表、索引、字段列的相关的元数据
2. 元数据放在innodb的系统表空间中
3. 历史原因，数据字典元数据与.frm文件中的信息一定程度上重叠
- 3.doublewrite buffer
1. 位于系统表空间的存储区域
2. innodb刷脏时，将脏数据写入数据文件中的正确位置之前先把脏页从innodb缓冲池写入双写缓冲中
3. 双写，但不需要两倍IO开销，因为是以一次1MB、大的顺序块写入双写缓冲、执行一次fsync调用
4. 双写缓冲innodb_doublewrite是全局的，对于非fusion-io设备也会禁用双写缓冲，所以原子写功能在fusion-io且启用了fusion-io nvfms时生效，建议将innodb_flush_method设置为O_DIRECT
- 4.undo logs
- 5.file-per-table tablespaces
- general tablespaces
- undo tablespaces
- temporary tablespaces
- redo logs


## innodb后台进程
```
> select name from performance_schema.threads where type="BACKGROUND";
+---------------------------------------------+
| name                                        |
+---------------------------------------------+
| thread/sql/main                             |
| thread/innodb/io_ibuf_thread                |
| thread/innodb/io_log_thread                 |
| thread/innodb/io_read_thread                |
| thread/innodb/io_read_thread                |
| thread/innodb/io_read_thread                |
| thread/innodb/io_read_thread                |
| thread/innodb/io_write_thread               |
| thread/innodb/io_write_thread               |
| thread/innodb/io_write_thread               |
| thread/innodb/io_write_thread               |
| thread/innodb/page_flush_coordinator_thread |
| thread/innodb/log_checkpointer_thread       |
| thread/innodb/log_closer_thread             |
| thread/innodb/log_flush_notifier_thread     |
| thread/innodb/log_flusher_thread            |
| thread/innodb/log_write_notifier_thread     |
| thread/innodb/log_writer_thread             |
| thread/innodb/srv_lock_timeout_thread       |
| thread/innodb/srv_error_monitor_thread      |
| thread/innodb/srv_monitor_thread            |
| thread/innodb/buf_resize_thread             |
| thread/innodb/srv_master_thread             |
| thread/innodb/buf_dump_thread               |
| thread/innodb/dict_stats_thread             |
| thread/innodb/fts_optimize_thread           |
| thread/mysqlx/worker                        |
| thread/mysqlx/acceptor_network              |
| thread/mysqlx/worker                        |
| thread/mysqlx/acceptor_network              |
| thread/innodb/clone_gtid_thread             |
| thread/innodb/srv_purge_thread              |
| thread/innodb/srv_purge_thread              |
| thread/innodb/srv_worker_thread             |
| thread/innodb/srv_worker_thread             |
| thread/innodb/srv_worker_thread             |
| thread/innodb/srv_worker_thread             |
| thread/innodb/srv_worker_thread             |
| thread/innodb/srv_worker_thread             |
| thread/sql/signal_handler                   |
+---------------------------------------------+
40 rows in set (0.00 sec)
```

#### mysql 前台进程
（双主复制架构实例）
```
> select name from performance_schema.threads where type="FOREGROUND" ;
+--------------------------------+
| name                           |
+--------------------------------+
| thread/sql/compress_gtid_table |
| thread/sql/one_connection		 |
| thread/sql/event_scheduler     |
| thread/sql/compress_gtid_table |
| thread/sql/one_connection 	 |
| thread/sql/slave_io			 |
| thread/sql/slave_sql			 |
| thread/sql/slave_worker		 |
+--------------------------------+
```
5种前台进程
1. compress_gtid_table
2. one_connection（用户连接线程）：处理用户请求的线程
3. slave_io：用于拉取主库binlog日志的线程
4. slave_sql：应用从主库拉取binlog日志的线程。多线程复制中，为协调器线程，用于分发binlog日志给工作线程
5. slave_worker（工作线程）：接受并应用sql线程分发的主库binlog日志