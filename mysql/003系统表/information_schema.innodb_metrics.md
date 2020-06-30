本文来自https://yq.aliyun.com/articles/41056
## 简介
- 除了Performance Schema外，在MySQL 5.6中还提供了一个新的information_schema表来监控Innodb的内部运行状态——INNODB_METRICS；该表维护了一组计数器，用户可以通过这些计数器，来监控Innodb内部运行是否健康
- MySQL5.6.12版本中，共有210个计数器;mysql8.0.17-debug中，有299个
```
> select count(*) from INNODB_METRICS;
+----------+
| count(*) |
+----------+
|      299 |
+----------+
1 row in set (0.00 sec)
```

## 使用

```
 select * from information_schema.innodb_metrics limit 1;
+-------------------------------+-----------+-------+-----------+-----------+-----------+-------------+-----------------+-----------------+-----------------+--------------+---------------+--------------+------------+----------+---------+--------------------------------+
| NAME                          | SUBSYSTEM | COUNT | MAX_COUNT | MIN_COUNT | AVG_COUNT | COUNT_RESET | MAX_COUNT_RESET | MIN_COUNT_RESET | AVG_COUNT_RESET | TIME_ENABLED | TIME_DISABLED | TIME_ELAPSED | TIME_RESET | STATUS   | TYPE    | COMMENT                        |
+-------------------------------+-----------+-------+-----------+-----------+-----------+-------------+-----------------+-----------------+-----------------+--------------+---------------+--------------+------------+----------+---------+--------------------------------+
| metadata_table_handles_opened | metadata  |     0 |      NULL |      NULL |      NULL |           0 |            NULL |            NULL |            NULL | NULL         | NULL          |         NULL | NULL       | disabled | counter | Number of table handles opened |
+-------------------------------+-----------+-------+-----------+-----------+-----------+-------------+-----------------+-----------------+-----------------+--------------+---------------+--------------+------------+----------+---------+--------------------------------+
1 row in set (0.00 sec)
```

#### 各列介绍

Column name|Description
---|---
NAME|计数器名称，唯一
SUBSYSTEM|The aspect of InnoDB that the metric applies to. See the list following the table for the corresponding module names to use with the SET GLOBAL syntax.
COUNT|计数器使能后的数值
MAX_COUNT|计数器使能后的极大值
MIN_COUNT|计数器使能后的极小值
AVG_COUNT|计数器使能后的平均值
COUNT_RESET|从最后一次重置后的计数值
MAX_COUNT_RESET|从最后一次重置后的计数极大值
MIN_COUNT_RESET|从最后一次重置后的计数极小值
AVG_COUNT_RESET|从最后一次重置后的计数平均值
TIME_ENABLED|最后使能时间
TIME_DISABLED|最后停止时间
TIME_ELAPSED|使能后流逝的秒数
TIME_RESET|最后停止时间
STATUS|计数器是否在运行
TYPE|Whether the item is a cumulative counter, or measures the current value of some resource.
COMMENT|附加描述

#### 运行样例

我们要查询DML的执行量

```
> select status,  NAME, COUNT, SUBSYSTEM from information_schema.INNODB_METRICS where name like 'dml%';
+----------+-------------+----------+-----------+
| status   | NAME        | COUNT    | SUBSYSTEM |
+----------+-------------+----------+-----------+
| disabled | dml_reads   |        0 | dml       |
| enabled  | dml_inserts | 67109492 | dml       |
| enabled  | dml_deletes |      130 | dml       |
| enabled  | dml_updates |      671 | dml       |
+----------+-------------+----------+-----------+
4 rows in set (0.00 sec)
```



## 打开或者关闭

AHI相关的计数器为例，默认情况下他们是关闭的

```
> select status,  NAME, COUNT, SUBSYSTEM from information_schema.INNODB_METRICS where status='disabled' and subsystem like '%adaptive_hash_index%';
+---------+------------------------------------------+----------+---------------------+
| status  | NAME                                     | COUNT    | SUBSYSTEM           |
+---------+------------------------------------------+----------+---------------------+
| disabled  | adaptive_hash_searches                   | 66329189 | adaptive_hash_index |
| disabled  | adaptive_hash_searches_btree             |  1315894 | adaptive_hash_index |
| disabled  | adaptive_hash_pages_added                |        0 | adaptive_hash_index |
| disabled  | adaptive_hash_pages_removed              |        0 | adaptive_hash_index |
| disabled  | adaptive_hash_rows_added                 |        0 | adaptive_hash_index |
| disabled  | adaptive_hash_rows_removed               |        0 | adaptive_hash_index |
| disabled  | adaptive_hash_rows_deleted_no_hash_entry |        0 | adaptive_hash_index |
| disabled  | adaptive_hash_rows_updated               |        0 | adaptive_hash_index |
+---------+------------------------------------------+----------+---------------------+
8 rows in set (0.00 sec)
```



- 1.打开计数器：
```
mysql> set global innodb_monitor_enable = 'adaptive_hash_%';
```

- 2.关闭计数器：
```
mysql> set global innodb_monitor_disable = 'adaptive_hash_%';

Query OK, 0 rows affected (0.00 sec)
```
- 3.重置AHI所有列的值：
```
mysql> set global innodb_monitor_reset_all = "adaptive_hash_%";
Query OK, 0 rows affected (0.00 sec)
```
- 4.只重置COUNTER的值：
```
mysql> set global innodb_monitor_reset = "adaptive_hash_%";
Query OK, 0 rows affected (0.00 sec)
```
- 5.根据模块名打开：
```
mysql> set global innodb_monitor_enable = module_adaptive_hash;
Query OK, 0 rows affected (0.00 sec)
```
- 6.打开所有计数器：
```
mysql>  set global innodb_monitor_enable = all;
Query OK, 0 rows affected (0.00 sec)
```
- 7.关闭所有计数器：
```
mysql> set global innodb_monitor_disable =  all;
Query OK, 0 rows affected (0.00 sec)
```

## 模块名与subsystem的对应关系

| 模块名               | 对应subsystem       | 描述                                                         |
| -------------------- | ------------------- | ------------------------------------------------------------ |
| module_metadata      | metadata            | 表级别的打开、关闭、引用次数等                               |
| module_lock          | lock                | 锁系统相关信息，例如死锁次数, 创建/移除/请求的记录锁，包括表锁等统计信息，锁等待/持有时间等等。。 |
| module_buffer        | buffer              | 跟buffer pool相关的操作，                                    |
| module_buffer_page   | buffer_page_io      | buffer pool做写操作的计数                                    |
| module_os            | os                  | os层的数据读写等信息                                         |
| module_trx           | transaction         | 事务量统计，例如只读事务，写事务，回滚事务，活跃事务，事务Undo信息等。 |
| module_purge         | purge               | purge操作统计，例如purge 标记删除的记录树，Purge undo日志的page数等 |
| module_compress      | compression         | 压缩表相关统计信息，例如压缩，解压，增加/减少padding的次数等。 |
| module_file          | file_system         | 只有一个counter:file_num_open_files 表示打开的文件数         |
| module_index         | index               | 索引分裂和合并的次数                                         |
| module_adaptive_hash | adaptive_hash_index | 自适应hash相关操作                                           |
| module_ibuf_system   | change_buffer       | change buffer相关操作统计                                    |
| module_srv           | server              | 实例内部运行状态，例如bp size , page size ,master线程信息，spin 统计，读写锁信息，写double write buffer的计数 |
| module_ddl           | ddl                 | DDL统计                                                      |
| module_dml           | dml                 | 读/插入/删除/更新的次数                                      |
| module_log           | recovery            | 跟redo log相关的信息，例如reodo checkpoinr信息，flush 信息，同步/异步刷日志点，日志写入量，pending的日志请求等。。 |
| module_icp           | icp                 | 在Innodb层的index condition pushdown的相关信息               |