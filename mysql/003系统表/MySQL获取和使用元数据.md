## information_schema

```
---------------------------------------------------------
|	information_schema		|			show			|
---------------------------------------------------------
|		schemata			|		show databases		|
|		tables				|		show tables			|
|		columns				|		show columns from	|
---------------------------------------------------------
```

#### 查看表定义或者表结构

```
> show create table test;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                        |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------+
| test  | CREATE TABLE `test` (
  `a` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`a`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

> desc test;
+-------+---------+------+-----+---------+----------------+
| Field | Type    | Null | Key | Default | Extra          |
+-------+---------+------+-----+---------+----------------+
| a     | int(11) | NO   | PRI | NULL    | auto_increment |
+-------+---------+------+-----+---------+----------------+
1 row in set (0.00 sec)


> show columns from test;
+-------+---------+------+-----+---------+----------------+
| Field | Type    | Null | Key | Default | Extra          |
+-------+---------+------+-----+---------+----------------+
| a     | int(11) | NO   | PRI | NULL    | auto_increment |
+-------+---------+------+-----+---------+----------------+
1 row in set (0.00 sec)

> select * from information_schema.columns where table_schema="test" and table_name="test";
+---------------+--------------+------------+-------------+------------------+----------------+-------------+-----------+--------------------------+------------------------+-------------------+---------------+--------------------+--------------------+----------------+-------------+------------+----------------+---------------------------------+----------------+-----------------------+--------+
| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME | ORDINAL_POSITION | COLUMN_DEFAULT | IS_NULLABLE | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH | CHARACTER_OCTET_LENGTH | NUMERIC_PRECISION | NUMERIC_SCALE | DATETIME_PRECISION | CHARACTER_SET_NAME | COLLATION_NAME | COLUMN_TYPE | COLUMN_KEY | EXTRA          | PRIVILEGES                      | COLUMN_COMMENT | GENERATION_EXPRESSION | SRS_ID |
+---------------+--------------+------------+-------------+------------------+----------------+-------------+-----------+--------------------------+------------------------+-------------------+---------------+--------------------+--------------------+----------------+-------------+------------+----------------+---------------------------------+----------------+-----------------------+--------+
| def           | test         | test       | a           |                1 | NULL           | NO          | int       |                     NULL |                   NULL |                10 |             0 |               NULL | NULL               | NULL           | int(11)     | PRI        | auto_increment | select,insert,update,references |                |                       |   NULL |
+---------------+--------------+------------+-------------+------------------+----------------+-------------+-----------+--------------------------+------------------------+-------------------+---------------+--------------------+--------------------+----------------+-------------+------------+----------------+---------------------------------+----------------+-----------------------+--------+
1 row in set (0.00 sec)

mysqldump --no-data  test test 
……………………
--
-- Table structure for table `test`
--

DROP TABLE IF EXISTS `test`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `test` (
  `a` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`a`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
……………………
```
## 改变set

```
alter table test modify  a set ("mon","tue") default "true";
```
## show

```
> show variables ;


> show variables like "%innodb_buffer%";
+-------------------------------------+----------------+
| Variable_name                       | Value          |
+-------------------------------------+----------------+
| innodb_buffer_pool_chunk_size       | 134217728      |
| innodb_buffer_pool_dump_at_shutdown | ON             |
| innodb_buffer_pool_dump_now         | OFF            |
| innodb_buffer_pool_dump_pct         | 25             |
| innodb_buffer_pool_filename         | ib_buffer_pool |
| innodb_buffer_pool_in_core_file     | ON             |
| innodb_buffer_pool_instances        | 1              |
| innodb_buffer_pool_load_abort       | OFF            |
| innodb_buffer_pool_load_at_startup  | ON             |
| innodb_buffer_pool_load_now         | OFF            |
| innodb_buffer_pool_size             | 134217728      |
+-------------------------------------+----------------+
11 rows in set (0.00 sec)


show status ;


> show status like "uptime";
//等价show session status ;
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| Uptime        | 193461 |
+---------------+--------+
1 row in set (0.00 sec)



show /*!50002 global */ status ;                //5.0.2版本
show global status ;

show engines ;
> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```
## select

```
> select version();

> select current_user();

> select user();

> select database();

```
