## 统计数据的方式

innodb提供了永久和非永久的统计数据的方式。永久是指，统计信息存储在磁盘上，服务重启后统计数据依然存在；非永久是指统计数据存储在内存，服务关闭时会被清除，服务重启后会重新收集数据。默认方式是永久的。两者的统计方式和触发条件都不同。

#### 参数

可以看到innodb_stats_persistent默认是on

```
mysql> show variables like "%stats_persistent%";
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| innodb_stats_persistent              | ON    |
| innodb_stats_persistent_sample_pages | 20    |
+--------------------------------------+-------+
2 rows in set (0.00 sec)
```
并且可以看到，统计信息的采样是innodb_stats_persistent_sample_pages=20，即采样20个页面，并根据20个页面来计算统计信息。

如果想及时更新信息，可以

```
flush table ****
analyze table ***
```
#### 创建表时指定参数
```
mysql> create table test_stats(a int) stats_persistent=1;
Query OK, 0 rows affected (0.10 sec)

mysql> show create table test_stats;
+------------+------------------------------------------------------------------------------------------------------------------------------------------+
| Table      | Create Table                                                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------------------+
| test_stats | CREATE TABLE `test_stats` (
  `a` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci STATS_PERSISTENT=1 |
+------------+------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

```

## 统计信息保存

MySQL收集的统计信息会保存在以下两种表

- mysql.innodb_table_stats
- mysql.innodb_index_stats

#### mysql.innodb_table_stats表

```
mysql> select * from innodb_table_stats;
+---------------+----------------+---------------------+--------+----------------------+--------------------------+
| database_name | table_name     | last_update         | n_rows | clustered_index_size | sum_of_other_index_sizes |
+---------------+----------------+---------------------+--------+----------------------+--------------------------+
| mysql         | component      | 2020-12-01 11:17:28 |      0 |                    1 |                        0 |
| mysql         | gtid_executed  | 2020-12-01 11:17:28 |      0 |                    1 |                        0 |
| sys           | sys_config     | 2020-12-01 11:17:35 |      6 |                    1 |                        0 |
| test          | test           | 2020-12-28 15:45:41 |      0 |                    1 |                        0 |
| test          | test_pri       | 2020-12-29 09:34:34 |      6 |                    1 |                        0 |
| test          | test_pri2      | 2020-12-29 09:38:49 |  24656 |                   97 |                        0 |
| test          | test_rowformat | 2020-12-28 17:02:08 |  16392 |                 2084 |                      360 |
| test          | test_stats     | 2020-12-29 10:01:56 |      0 |                    1 |                        0 |
+---------------+----------------+---------------------+--------+----------------------+--------------------------+
8 rows in set (0.00 sec)
```

#### mysql.innodb_index_stats表

```
mysql> select * from innodb_index_stats;
+---------------+----------------+--------------------+---------------------+--------------+------------+-------------+-----------------------------------+
| database_name | table_name     | index_name         | last_update         | stat_name    | stat_value | sample_size | stat_description                  |
+---------------+----------------+--------------------+---------------------+--------------+------------+-------------+-----------------------------------+
| mysql         | component      | PRIMARY            | 2020-12-01 11:17:28 | n_diff_pfx01 |          0 |           1 | component_id                      |
| mysql         | component      | PRIMARY            | 2020-12-01 11:17:28 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
| mysql         | component      | PRIMARY            | 2020-12-01 11:17:28 | size         |          1 |        NULL | Number of pages in the index      |
| mysql         | gtid_executed  | PRIMARY            | 2020-12-01 11:17:28 | n_diff_pfx01 |          0 |           1 | source_uuid                       |
| mysql         | gtid_executed  | PRIMARY            | 2020-12-01 11:17:28 | n_diff_pfx02 |          0 |           1 | source_uuid,interval_start        |
| mysql         | gtid_executed  | PRIMARY            | 2020-12-01 11:17:28 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
| mysql         | gtid_executed  | PRIMARY            | 2020-12-01 11:17:28 | size         |          1 |        NULL | Number of pages in the index      |
| sys           | sys_config     | PRIMARY            | 2020-12-01 11:17:35 | n_diff_pfx01 |          6 |           1 | variable                          |
| sys           | sys_config     | PRIMARY            | 2020-12-01 11:17:35 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
| sys           | sys_config     | PRIMARY            | 2020-12-01 11:17:35 | size         |          1 |        NULL | Number of pages in the index      |
| test          | test           | GEN_CLUST_INDEX    | 2020-12-28 15:45:41 | n_diff_pfx01 |          0 |           1 | DB_ROW_ID                         |
| test          | test           | GEN_CLUST_INDEX    | 2020-12-28 15:45:41 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
| test          | test           | GEN_CLUST_INDEX    | 2020-12-28 15:45:41 | size         |          1 |        NULL | Number of pages in the index      |
| test          | test_pri       | PRIMARY            | 2020-12-29 09:34:34 | n_diff_pfx01 |          6 |           1 | a                                 |
| test          | test_pri       | PRIMARY            | 2020-12-29 09:34:34 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
| test          | test_pri       | PRIMARY            | 2020-12-29 09:34:34 | size         |          1 |        NULL | Number of pages in the index      |
| test          | test_pri2      | PRIMARY            | 2020-12-29 09:38:49 | n_diff_pfx01 |      24656 |          20 | a                                 |
| test          | test_pri2      | PRIMARY            | 2020-12-29 09:38:49 | n_leaf_pages |         92 |        NULL | Number of leaf pages in the index |
| test          | test_pri2      | PRIMARY            | 2020-12-29 09:38:49 | size         |         97 |        NULL | Number of pages in the index      |
| test          | test_pri2      | test_pri2_idx_b    | 2020-12-29 09:38:49 | n_diff_pfx01 |          5 |           7 | b                                 |
| test          | test_pri2      | test_pri2_idx_b    | 2020-12-29 09:38:49 | n_diff_pfx02 |      24552 |          20 | b,a                               |
| test          | test_pri2      | test_pri2_idx_b    | 2020-12-29 09:38:49 | n_leaf_pages |         66 |        NULL | Number of leaf pages in the index |
| test          | test_pri2      | test_pri2_idx_b    | 2020-12-29 09:38:49 | size         |         97 |        NULL | Number of pages in the index      |
| test          | test_rowformat | GEN_CLUST_INDEX    | 2020-12-28 17:02:08 | n_diff_pfx01 |      16392 |          20 | DB_ROW_ID                         |
| test          | test_rowformat | GEN_CLUST_INDEX    | 2020-12-28 17:02:08 | n_leaf_pages |       2049 |        NULL | Number of leaf pages in the index |
| test          | test_rowformat | GEN_CLUST_INDEX    | 2020-12-28 17:02:08 | size         |       2084 |        NULL | Number of pages in the index      |
| test          | test_rowformat | test_rowformat_idx | 2020-12-28 17:02:08 | n_diff_pfx01 |          1 |           3 | a                                 |
| test          | test_rowformat | test_rowformat_idx | 2020-12-28 17:02:08 | n_diff_pfx02 |      17794 |          20 | a,DB_ROW_ID                       |
| test          | test_rowformat | test_rowformat_idx | 2020-12-28 17:02:08 | n_leaf_pages |        310 |        NULL | Number of leaf pages in the index |
| test          | test_rowformat | test_rowformat_idx | 2020-12-28 17:02:08 | size         |        360 |        NULL | Number of pages in the index      |
| test          | test_stats     | GEN_CLUST_INDEX    | 2020-12-29 10:01:56 | n_diff_pfx01 |          0 |           1 | DB_ROW_ID                         |
| test          | test_stats     | GEN_CLUST_INDEX    | 2020-12-29 10:01:56 | n_leaf_pages |          1 |        NULL | Number of leaf pages in the index |
| test          | test_stats     | GEN_CLUST_INDEX    | 2020-12-29 10:01:56 | size         |          1 |        NULL | Number of pages in the index      |
+---------------+----------------+--------------------+---------------------+--------------+------------+-------------+-----------------------------------+
33 rows in set (0.00 sec)

```

可以看到我们创建的用户表的sample_size是20，刚好与innodb_stats_persistent_sample_pages一致

## 更新统计数据

#### 持久的统计数据：

- 当 innodb_stats_auto_recalc = ON，默认开启
- 创建表时指定 STATS_AUTO_RECALC=1。
- 表数据变化超过10%时，触发自动收集。
- 但是可能会有延迟几秒，并不会马上触发收集。

- 统计信息收集是采样分析，默认是20个 page，由 innodb_stats_persistent_sample_pages 参数控制。
- 如果统计信息相差太多，可以增加这个参数，不过 ANAYLZE TABLE 收集的时间会增加。
- 也可以在创建表的时候指定 STATS_SAMPLE_PAGES。
- 如果感觉自动收集的统计信息不准时，甚至可以直接修改 mysql.innodb_table_stats 和 mysql.innodb_index_stats。修改完 flush table tbl_name 就可以了。

#### 非持久统计数据：

- ANALYZE TABLE
- innodb_stats_on_metadata = 1(该参数只有 innodb_stats_persistent =OFF 才生效)。启用 innodb_stats_on_metadata 可能会降低性能
  - 执行 SHOW TABLE STATUS， SHOW INDEX
  - 查询 INFORMATION_SCHEMA.TABLES 或 INFORMATION_SCHEMA.STATISTICS 表
- mysql客户端使用了 --auto-rehash，该 auto-rehash 选项会导致所有InnoDB表被打开，并且打开表操作会导致重新计算统计信息。建议禁用该操作 auto-rehash，使用参数 --disable-auto-rehash 禁用。
- 第一次打开表
- InnoDB 检测自上次更新统计信息以来表已修改了1/16。
- 默认的采样 page 是 8，由参数 innodb_stats_transient_sample_pages 控制 

```
mysql> show variables like "%innodb_stats_transient_sample_pages%";
+-------------------------------------+-------+
| Variable_name                       | Value |
+-------------------------------------+-------+
| innodb_stats_transient_sample_pages | 8     |
+-------------------------------------+-------+
1 row in set (0.01 sec)
```

## 对于统计时null值的处理

```
mysql> show variables like "%innodb_stats_method%";
+---------------------+-------------+
| Variable_name       | Value       |
+---------------------+-------------+
| innodb_stats_method | nulls_equal |
+---------------------+-------------+
1 row in set (0.01 sec)
```

该值可以设为三种：

1. nulls_equal:所有null都相等
2. nulls_unequal:所有null都不相等
3. nulls_ignored:所有null都被忽略

参考：https://www.cnblogs.com/ziroro/p/9948103.html