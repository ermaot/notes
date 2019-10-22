performance schema提供了如下功能
- 元数据锁（metadata locking）：对于了解会话之间元数据锁依赖关系至关重要。metadata_locks表了解元数据锁的先关信息
1. 哪些会话拥有哪些元数据锁
2. 哪些会话正在等待元数据锁
3. 哪些请求由于死锁被杀掉或者锁等待超时而被丢弃
```
mysql 5.7.27
> desc metadata_locks;
+-----------------------+---------------------+------+-----+---------+-------+
| Field                 | Type                | Null | Key | Default | Extra |
+-----------------------+---------------------+------+-----+---------+-------+
| OBJECT_TYPE           | varchar(64)         | NO   |     | NULL    |       |
| OBJECT_SCHEMA         | varchar(64)         | YES  |     | NULL    |       |
| OBJECT_NAME           | varchar(64)         | YES  |     | NULL    |       |
| OBJECT_INSTANCE_BEGIN | bigint(20) unsigned | NO   |     | NULL    |       |
| LOCK_TYPE             | varchar(32)         | NO   |     | NULL    |       |
| LOCK_DURATION         | varchar(32)         | NO   |     | NULL    |       |
| LOCK_STATUS           | varchar(32)         | NO   |     | NULL    |       |
| SOURCE                | varchar(64)         | YES  |     | NULL    |       |
| OWNER_THREAD_ID       | bigint(20) unsigned | YES  |     | NULL    |       |
| OWNER_EVENT_ID        | bigint(20) unsigned | YES  |     | NULL    |       |
+-----------------------+---------------------+------+-----+---------+-------+
10 rows in set (0.01 sec)

mysql8
> desc metadata_locks;
+-----------------------+---------------------+------+-----+---------+-------+
| Field                 | Type                | Null | Key | Default | Extra |
+-----------------------+---------------------+------+-----+---------+-------+
| OBJECT_TYPE           | varchar(64)         | NO   | MUL | NULL    |       |
| OBJECT_SCHEMA         | varchar(64)         | YES  |     | NULL    |       |
| OBJECT_NAME           | varchar(64)         | YES  |     | NULL    |       |
| COLUMN_NAME           | varchar(64)         | YES  |     | NULL    |       |
| OBJECT_INSTANCE_BEGIN | bigint(20) unsigned | NO   | PRI | NULL    |       |
| LOCK_TYPE             | varchar(32)         | NO   |     | NULL    |       |
| LOCK_DURATION         | varchar(32)         | NO   |     | NULL    |       |
| LOCK_STATUS           | varchar(32)         | NO   |     | NULL    |       |
| SOURCE                | varchar(64)         | YES  |     | NULL    |       |
| OWNER_THREAD_ID       | bigint(20) unsigned | YES  | MUL | NULL    |       |
| OWNER_EVENT_ID        | bigint(20) unsigned | YES  |     | NULL    |       |
+-----------------------+---------------------+------+-----+---------+-------+
11 rows in set (0.00 sec)
> select * from metadata_locks;
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
| OBJECT_TYPE | OBJECT_SCHEMA      | OBJECT_NAME    | COLUMN_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE   | LOCK_DURATION | LOCK_STATUS | SOURCE            | OWNER_THREAD_ID | OWNER_EVENT_ID |
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
| TABLE       | performance_schema | metadata_locks | NULL        |       140162510872336 | SHARED_READ | TRANSACTION   | GRANTED     | sql_parse.cc:5964 |              58 |          69352 |
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
1 row in set (0.00 sec)
```

- 进度跟踪（stage tracking）：跟踪长时间操作的进度（比如alter table）。通过events_stages_current了解进度
```
> desc events_stages_current;
+--------------------+------------------------------------------------+------+-----+---------+-------+
| Field              | Type                                           | Null | Key | Default | Extra |
+--------------------+------------------------------------------------+------+-----+---------+-------+
| THREAD_ID          | bigint(20) unsigned                            | NO   | PRI | NULL    |       |
| EVENT_ID           | bigint(20) unsigned                            | NO   | PRI | NULL    |       |
| END_EVENT_ID       | bigint(20) unsigned                            | YES  |     | NULL    |       |
| EVENT_NAME         | varchar(128)                                   | NO   |     | NULL    |       |
| SOURCE             | varchar(64)                                    | YES  |     | NULL    |       |
| TIMER_START        | bigint(20) unsigned                            | YES  |     | NULL    |       |
| TIMER_END          | bigint(20) unsigned                            | YES  |     | NULL    |       |
| TIMER_WAIT         | bigint(20) unsigned                            | YES  |     | NULL    |       |
| WORK_COMPLETED     | bigint(20) unsigned                            | YES  |     | NULL    |       |
| WORK_ESTIMATED     | bigint(20) unsigned                            | YES  |     | NULL    |       |
| NESTING_EVENT_ID   | bigint(20) unsigned                            | YES  |     | NULL    |       |
| NESTING_EVENT_TYPE | enum('TRANSACTION','STATEMENT','STAGE','WAIT') | YES  |     | NULL    |       |
+--------------------+------------------------------------------------+------+-----+---------+-------+
12 rows in set (0.01 sec)
```

- 事务：监控服务层和存储引擎层事务的全部方面。新增events_transactions_current表
```
> desc events_transactions_current;
+---------------------------------+------------------------------------------------+------+-----+---------+-------+
| Field                           | Type                                           | Null | Key | Default | Extra |
+---------------------------------+------------------------------------------------+------+-----+---------+-------+
| THREAD_ID                       | bigint(20) unsigned                            | NO   | PRI | NULL    |       |
| EVENT_ID                        | bigint(20) unsigned                            | NO   | PRI | NULL    |       |
| END_EVENT_ID                    | bigint(20) unsigned                            | YES  |     | NULL    |       |
| EVENT_NAME                      | varchar(128)                                   | NO   |     | NULL    |       |
| STATE                           | enum('ACTIVE','COMMITTED','ROLLED BACK')       | YES  |     | NULL    |       |
| TRX_ID                          | bigint(20) unsigned                            | YES  |     | NULL    |       |
| GTID                            | varchar(64)                                    | YES  |     | NULL    |       |
| XID_FORMAT_ID                   | int(11)                                        | YES  |     | NULL    |       |
| XID_GTRID                       | varchar(130)                                   | YES  |     | NULL    |       |
| XID_BQUAL                       | varchar(130)                                   | YES  |     | NULL    |       |
| XA_STATE                        | varchar(64)                                    | YES  |     | NULL    |       |
| SOURCE                          | varchar(64)                                    | YES  |     | NULL    |       |
| TIMER_START                     | bigint(20) unsigned                            | YES  |     | NULL    |       |
| TIMER_END                       | bigint(20) unsigned                            | YES  |     | NULL    |       |
| TIMER_WAIT                      | bigint(20) unsigned                            | YES  |     | NULL    |       |
| ACCESS_MODE                     | enum('READ ONLY','READ WRITE')                 | YES  |     | NULL    |       |
| ISOLATION_LEVEL                 | varchar(64)                                    | YES  |     | NULL    |       |
| AUTOCOMMIT                      | enum('YES','NO')                               | NO   |     | NULL    |       |
| NUMBER_OF_SAVEPOINTS            | bigint(20) unsigned                            | YES  |     | NULL    |       |
| NUMBER_OF_ROLLBACK_TO_SAVEPOINT | bigint(20) unsigned                            | YES  |     | NULL    |       |
| NUMBER_OF_RELEASE_SAVEPOINT     | bigint(20) unsigned                            | YES  |     | NULL    |       |
| OBJECT_INSTANCE_BEGIN           | bigint(20) unsigned                            | YES  |     | NULL    |       |
| NESTING_EVENT_ID                | bigint(20) unsigned                            | YES  |     | NULL    |       |
| NESTING_EVENT_TYPE              | enum('TRANSACTION','STATEMENT','STAGE','WAIT') | YES  |     | NULL    |       |
+---------------------------------+------------------------------------------------+------+-----+---------+-------+
24 rows in set (0.00 sec)

```
- 内存使用：提供内存使用信息统计。mysql5.7.2开始，分别从账号、访问主机、线程、用户、及事件的角度统计内存
```
> show tables like "%memory%";
+-----------------------------------------+
| Tables_in_performance_schema (%memory%) |
+-----------------------------------------+
| memory_summary_by_account_by_event_name |
| memory_summary_by_host_by_event_name    |
| memory_summary_by_thread_by_event_name  |
| memory_summary_by_user_by_event_name    |
| memory_summary_global_by_event_name     |
+-----------------------------------------+
5 rows in set (0.01 sec)
```
使用情况
- 存储程序（stored programs）：存储过程、存储方法、事件调度器和表触发器的检测器，在setup_objects表中。

## sys schema介绍
- mysql sys schema是由一系列对象组成的database schema，是一种视图，将performance_schema和information_schema中的数据以更容易理解的方式展现
- sys schema可用于典型的调优和诊断
1. 将性能模式数据汇总成更易于理解的视图
2. 性能模式配置和生成诊断报告等操作的存储过程
3. 用于查询性能模式配置并提供格式化服务的存储函数

- sys视图展示信息有：
1. 主机相关信息:以host_summary开头的视图,主要汇总了Io延迟的信息,从主机、文件事件类型、语句类型等角度展示文件IO的信息
2. innodb相关信息:以 innodb开头的视图,汇总了 innodb buffer page信息和事务等待InnoDB锁信息.
3. io使用情况:以io开头的视图,总结了io使用者的信息,包括等待I/O的情况、I/O使用量情况,从各个角度分组展示.
4. 内存使用情况:以memory开头的视图,从主机、线程、用户、事件角度展示内存使用情况.
5. 连接与会话信息:其中, processlist和 session相关的视图,总结了会话相关信息.
6. 表相关信息:以 schema_table开头的视图,从全表扫描、 InnoDB缓冲池等方面展示了表统计信息
7. 索引信息:其中包含index的视图,统计了索引使用情况,以及重复索引和未使用的索引情况.
8. 语句相关信息:以statement开头的视图,统计的规范化后的语句使用情况,包括错误数、警告数、执行全表扫描的、使用临时表、执行排序等信息.
9. 用户的相关信息:以user开头的视图,统计了用户使用的文件IO、执行的语句统计信息等.
10. 等待事件相关信息:以wait开头的视图,从主机和事件角度展示等待类事件的延迟情况.

## sys schema重点场景
- 查看表访问量
```
> select table_schema,table_name,sum(io_read_requests+io_write_requests) from schema_table_statistics group by table_schema,table_name
```
- 冗余索引和未送索引检查
```
> select * from schema_redundant_indexes;
+--------------+------------+----------------------+-------------------------+----------------------------+---------------------+------------------------+---------------------------+----------------+---------------------------------------------------------+
| table_schema | table_name | redundant_index_name | redundant_index_columns | redundant_index_non_unique | dominant_index_name | dominant_index_columns | dominant_index_non_unique | subpart_exists | sql_drop_index                                          |
+--------------+------------+----------------------+-------------------------+----------------------------+---------------------+------------------------+---------------------------+----------------+---------------------------------------------------------+
| test         | test0906   | test0906_b2          | b                       |                          1 | test0906_b          | b                      |                         1 |              0 | ALTER TABLE `test`.`test0906` DROP INDEX `test0906_b2`  |
| test         | test0906   | test0906_pri         | a                       |                          1 | PRIMARY             | a                      |                         0 |              0 | ALTER TABLE `test`.`test0906` DROP INDEX `test0906_pri` |
+--------------+------------+----------------------+-------------------------+----------------------------+---------------------+------------------------+---------------------------+----------------+---------------------------------------------------------+
2 rows in set (0.02 sec)

> select * from schema_unused_indexes where object_schema!="performance_schema";
+---------------+-------------+--------------+
| object_schema | object_name | index_name   |
+---------------+-------------+--------------+
| test          | test0906    | test0906_pri |
| test          | test0906    | test0906_b   |
| test          | test0906    | test0906_b2  |
+---------------+-------------+--------------+
3 rows in set, 1 warning (0.03 sec)
```
- 表自增监控
```
> desc schema_auto_increment_columns;
+----------------------+------------------------+------+-----+---------+-------+
| Field                | Type                   | Null | Key | Default | Extra |
+----------------------+------------------------+------+-----+---------+-------+
| table_schema         | varchar(64)            | NO   |     |         |       |
| table_name           | varchar(64)            | NO   |     |         |       |
| column_name          | varchar(64)            | NO   |     |         |       |
| data_type            | varchar(64)            | NO   |     |         |       |
| column_type          | longtext               | NO   |     | NULL    |       |
| is_signed            | int(1)                 | NO   |     | 0       |       |
| is_unsigned          | int(1)                 | NO   |     | 0       |       |
| max_value            | bigint(21) unsigned    | YES  |     | NULL    |       |
| auto_increment       | bigint(21) unsigned    | YES  |     | NULL    |       |
| auto_increment_ratio | decimal(25,4) unsigned | YES  |     | NULL    |       |
+----------------------+------------------------+------+-----+---------+-------+
10 rows in set (0.00 sec)
```
- 监控全表扫描
```
> select * from statements_with_full_table_scans limit 1;
+---------------------------------------------+--------------------+------------+---------------+---------------------+--------------------------+-------------------+-----------+---------------+---------------+-------------------+---------------------+---------------------+----------------------------------+
| query                                       | db                 | exec_count | total_latency | no_index_used_count | no_good_index_used_count | no_index_used_pct | rows_sent | rows_examined | rows_sent_avg | rows_examined_avg | first_seen          | last_seen           | digest                           |
+---------------------------------------------+--------------------+------------+---------------+---------------------+--------------------------+-------------------+-----------+---------------+---------------+-------------------+---------------------+---------------------+----------------------------------+
| SELECT * FROM `INNODB_SYS_columnS` LIMIT ?  | information_schema |          2 | 616.01 us     |                   2 |                        0 |               100 |         6 |             6 |             3 |                 3 | 2019-09-06 10:47:16 | 2019-09-06 10:47:19 | 9122aece3d09fbffc0329d493e353f68 |
+---------------------------------------------+--------------------+------------+---------------+---------------------+--------------------------+-------------------+-----------+---------------+---------------+-------------------+---------------------+---------------------+----------------------------------+
1 row in set (0.01 sec)
```
- 查看实例消耗的磁盘IO
```
> select file,avg_read+avg_write as avg_io from io_global_by_file_by_bytes order by avg_io desc limit 10;
+--------------------------------------------------------------------+--------+
| file                                                               | avg_io |
+--------------------------------------------------------------------+--------+
| @@datadir/sys/schema_tables_with_full_table_scans.frm              |   1023 |
| @@datadir/sys/x@0024schema_tables_with_full_table_scans.frm        |    994 |
| @@datadir/mysql/db.MYD                                             |    976 |
| @@datadir/mysql/tables_priv.MYD                                    |    947 |
| @@datadir/sys/x@0024ps_digest_95th_percentile_by_avg_us.frm        |    906 |
| @@datadir/performance_schema/table_lock_waits_summary_by_table.frm |    897 |
| @@datadir/mysql/proxies_priv.MYD                                   |    837 |
| @@datadir/mysql/innodb_index_stats.frm                             |    734 |
| @@datadir/sys/sys_config.TRG                                       |    720 |
| @@datadir/performance_schema/replication_connection_status.frm     |    668 |
+--------------------------------------------------------------------+--------+
10 rows in set (0.17 sec)
```

## 使用风险
#### 性能影响
- sys schema数据来源是performance_schema和information_schema，而performance_schema引入到MySQL时性能消耗巨大
- 测试发现，开启performance_schema性能消耗10%

#### 操作风险
建议尽量不要线上大量部署sys或者performance_schema或者information_schema收集相关信息，消耗资源，严重可能导致业务请求被阻塞引起故障