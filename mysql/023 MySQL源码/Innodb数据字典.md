## 背景
- 数据字典是用来存储数据原信息的表，属于系统表
- 数据字典的好处，让用户很简单明确了解MySQL元数据管理的实现原理
- Innodb数据字典不能被用户感知，只能通过源代码或者手册了解
## 系统表结构
Innodb有4个最基本的系统表，用来存储用户定义的表、列、索引、索引列等信息，分别是sys_tables,sys_columns,sys_indexes,sys_fileds
#### sys_tables
```
// mariadb5.6
MariaDB [information_schema]> select * from innodb_sys_tables;
+----------+-----------+-----------------------------+------+--------+-------+
| TABLE_ID | SCHEMA    | NAME                        | FLAG | N_COLS | SPACE |
+----------+-----------+-----------------------------+------+--------+-------+
|       11 |           | SYS_FOREIGN                 |    0 |      7 |     0 |
|       12 |           | SYS_FOREIGN_COLS            |    0 |      7 |     0 |
|      158 | mezz_temp | blog_blogpost               |    1 |     26 |     0 |
|       46 | mezzanine | auth_group                  |    1 |      5 |     0 |
|       53 | mezzanine | auth_group_permissions      |    1 |      6 |     0 |
|       63 | mezzanine | auth_permission             |    1 |      7 |     0 |
|       66 | mezzanine | auth_user                   |    1 |     14 |     0 |
|       55 | mezzanine | auth_user_groups            |    1 |      6 |     0 |


mysql5.7
> select * from INNODB_SYS_TABLES;
+----------+---------------------------------+------+--------+-------+-------------+------------+---------------+------------+
| TABLE_ID | NAME                            | FLAG | N_COLS | SPACE | FILE_FORMAT | ROW_FORMAT | ZIP_PAGE_SIZE | SPACE_TYPE |
+----------+---------------------------------+------+--------+-------+-------------+------------+---------------+------------+
|       14 | SYS_DATAFILES                   |    0 |      5 |     0 | Antelope    | Redundant  |             0 | System     |
|       11 | SYS_FOREIGN                     |    0 |      7 |     0 | Antelope    | Redundant  |             0 | System     |
|       12 | SYS_FOREIGN_COLS                |    0 |      7 |     0 | Antelope    | Redundant  |             0 | System     |
|       13 | SYS_TABLESPACES                 |    0 |      6 |     0 | Antelope    | Redundant  |             0 | System     |
|       15 | SYS_VIRTUAL                     |    0 |      6 |     0 | Antelope    | Redundant  |             0 | System     |

mysql8
l> select * from INNODB_TABLES;
+----------+---------------------------------+------+--------+------------+------------+---------------+------------+--------------+
| TABLE_ID | NAME                            | FLAG | N_COLS | SPACE      | ROW_FORMAT | ZIP_PAGE_SIZE | SPACE_TYPE | INSTANT_COLS |
+----------+---------------------------------+------+--------+------------+------------+---------------+------------+--------------+
|     1071 | mysql/component                 |  161 |      6 | 4294967294 | Dynamic    |             0 | General    |            0 |
|     1125 | mysql/columns_priv              |  161 |     10 | 4294967294 | Dynamic    |             0 | General    |            0 |
|     1126 | mysql/db                        |  161 |     25 | 4294967294 | Dynamic    |             0 | General    |            0 |

```
表列名|说明
---|---
name|表名
ID|ID号
N_COLS|表的列的个数（4个字节）
TYPE|表的存储类型、包括记录的格式、压缩等信息
space|表所在表空间的ID（4字节）。这个表对应的主键列为NAME，同时还有一个在ID号上的唯一索引

#### sys_columns
```
mariadb5.6
> select * from innodb_sys_columns limit 5;
+----------+----------+-----+-------+---------+-----+
| TABLE_ID | NAME     | POS | MTYPE | PRTYPE  | LEN |
+----------+----------+-----+-------+---------+-----+
|       11 | ID       |   0 |     1 | 2162692 |   0 |
|       11 | FOR_NAME |   1 |     1 | 2162692 |   0 |
|       11 | REF_NAME |   2 |     1 | 2162692 |   0 |


mysql5.7
> select * from innodb_sys_columns limit 5;
+----------+----------+-----+-------+---------+-----+
| TABLE_ID | NAME     | POS | MTYPE | PRTYPE  | LEN |
+----------+----------+-----+-------+---------+-----+
|       11 | ID       |   0 |     1 | 2162692 |   0 |
|       11 | FOR_NAME |   1 |     1 | 2162692 |   0 |
|       11 | REF_NAME |   2 |     1 | 2162692 |   0 |
|       11 | N_COLS   |   3 |     6 |       0 |   4 |
|       12 | ID       |   0 |     1 | 2162692 |   0 |
+----------+----------+-----+-------+---------+-----+
5 rows in set (0.00 sec)



MySQL8
> select * from INNODB_COLUMNS limit 10;
+----------+--------------------------+-----+-------+---------+-----+-------------+---------------+
| TABLE_ID | NAME                     | POS | MTYPE | PRTYPE  | LEN | HAS_DEFAULT | DEFAULT_VALUE |
+----------+--------------------------+-----+-------+---------+-----+-------------+---------------+
|        1 | properties               |   0 |     5 | 4130044 |  11 |           0 | NULL          |
|        2 | table_id                 |   0 |     6 |    1800 |   8 |           0 | NULL          |
|        2 | version                  |   1 |     6 |    1800 |   8 |           0 | NULL          |
|        2 | metadata                 |   2 |     5 | 4130300 |  10 |           0 | NULL          |
```
列名|说明
---|---
TABLE_ID|表示这个列所属的表的ID号(8字节).
POS|表示这个列在表中是第几列(4字节).
NAME|表示这个列的列名.
MTYPE|表示这个列的主数据类型(4字节).
PRTYPE|表示这个列的一些精确数据类型,它是一个组合值,包括NUL标志、是否有符号数的标志、是否是二进制字符串的标志及表示这个列是真的 VARCHAR(数据存储用两个字节)(4字节)
LEN|表示这个列的数据长度,但不包括VARCHAR类型,因为这个类型在记录里面存储了数据长度(4字节).
PREC|表示这个列数据的精度,但目前好像没有使用(4字节).这个表的主键列为( TABLE ID、POS).

#### SYS_INDEXES

```
mariadb5.6
> select * from innodb_sys_indexes limit 5;
+----------+---------+----------+------+----------+---------+-------+
| INDEX_ID | NAME    | TABLE_ID | TYPE | N_FIELDS | PAGE_NO | SPACE |
+----------+---------+----------+------+----------+---------+-------+
|       11 | ID_IND  |       11 |    3 |        1 |     303 |     0 |
|       12 | FOR_IND |       11 |    0 |        1 |     304 |     0 |
|       13 | REF_IND |       11 |    0 |        1 |     305 |     0 |
|       14 | ID_IND  |       12 |    3 |        2 |     306 |     0 |
|       75 | PRIMARY |       43 |    3 |        1 |     494 |     0 |
+----------+---------+----------+------+----------+---------+-------+
5 rows in set (0.00 sec)

mysql5.7
> select * from INNODB_SYS_indexeS limit 5;
+----------+-----------------------+----------+------+----------+---------+-------+-----------------+
| INDEX_ID | NAME                  | TABLE_ID | TYPE | N_FIELDS | PAGE_NO | SPACE | MERGE_THRESHOLD |
+----------+-----------------------+----------+------+----------+---------+-------+-----------------+
|       11 | ID_IND                |       11 |    3 |        1 |     270 |     0 |              50 |
|       12 | FOR_IND               |       11 |    0 |        1 |     271 |     0 |              50 |
|       13 | REF_IND               |       11 |    0 |        1 |     272 |     0 |              50 |
|       14 | ID_IND                |       12 |    3 |        2 |     273 |     0 |              50 |
|       15 | SYS_TABLESPACES_SPACE |       13 |    3 |        1 |     275 |     0 |              50 |
+----------+-----------------------+----------+------+----------+---------+-------+-----------------+
5 rows in set (0.00 sec)


mysql8
> select * from innodb_indexes limit 5;
+----------+-----------------+----------+------+----------+---------+-------+-----------------+
| INDEX_ID | NAME            | TABLE_ID | TYPE | N_FIELDS | PAGE_NO | SPACE | MERGE_THRESHOLD |
+----------+-----------------+----------+------+----------+---------+-------+-----------------+
|      142 | PRIMARY         |     1058 |    3 |        6 |       4 |     1 |              50 |
|      300 | GEN_CLUST_INDEX |     1158 |    1 |        4 |       4 |     3 |              50 |
+----------+-----------------+----------+------+----------+---------+-------+-----------------+
2 rows in set (0.01 sec)

```

用来存储 InnoDB中所有表的索引信息,每条记录对应一个索引.
列名|说明
---|---
TABLE ID|表示这个索引所属的表的ID号(8字节).
ID|表示这个索引的索引ID号(8字节).
NAME|表示这个索引的索引名.
N_FIELDS|表示这个索引包含的列个数(4字节).
TYPE|表示这个索引的类型,包括聚簇索引、唯一索引、 DICT_UNIVERSAL、 DICT_IBUF(插入缓冲区B+树)(4字节).
SPACE|表示这个索引数据所在的表空间ID号(4字节).
PAGE_NO|表示这个索引对应的B+树的根页面(4字节)这个表的主键列为( TABL_ID,ID )

#### SYS_FIELDS
用来存储所有索引中定义的索引列,每一条记录对应一个索引列
```
mariadb5.6
> select * from innodb_sys_fields limit 5;
+----------+----------+-----+
| INDEX_ID | NAME     | POS |
+----------+----------+-----+
|       11 | ID       |   0 |
|       12 | FOR_NAME |   0 |
|       13 | REF_NAME |   0 |
|       14 | ID       |   0 |
|       14 | POS      |   1 |
+----------+----------+-----+
5 rows in set (0.01 sec)


mysql5.7
> select * from INNODB_SYS_fieldS limit 5;
+----------+----------+-----+
| INDEX_ID | NAME     | POS |
+----------+----------+-----+
|       11 | ID       |   0 |
|       12 | FOR_NAME |   0 |
|       13 | REF_NAME |   0 |
|       14 | ID       |   0 |
|       14 | POS      |   1 |
+----------+----------+-----+
5 rows in set (0.00 sec)

mysql8
> select * from innodb_fields limit 5;
+----------+---------------+-----+
| INDEX_ID | NAME          | POS |
+----------+---------------+-----+
| 3        | database_name |   0 |
| 3        | table_name    |   1 |
| 4        | database_name |   0 |
| 4        | table_name    |   1 |
| 4        | index_name    |   2 |
+----------+---------------+-----+
5 rows in set (0.01 sec)

```

列名|说明
---|---
INDEX_ID|这个列所在的索引(8字节).
POS|这个列在某个索引中是第几个索引列(4字节).
COL_NAME|这个索引列的列名.这个表的主键列为( INDEX ID、POS).


## 字典表加载
- 如果是新建数据库，系统表结构和个数都固定，初始化的时候只需要创建表存储B+树，同时将这几个B+树根页号存储在一个固定位置
- 数据字典根页面，Innodb用了一个专门的页面（0号表空间0号文件的7号页面）管理字典信息。存储了四个表的5个根页面（5个索引，sys_tables包含两个索引），下一个表ID值，下一个索引ID值，下一个表空间ID值，ROWID
- Innodb初始化B+树以及其他操作完成后，通过函数dict_boot加载常驻内存的4个系统表并读取信息（代码中硬编码读取这几个表）
```
//./storage/innobase/dict/dict0boot.cc 的302行
比如：
index = dict_mem_index_create("SYS_TABLES", "CLUST_IND", DICT_HDR_SPACE,
                                  DICT_UNIQUE | DICT_CLUSTERED, 1);
```
- 初始化后，系统表常驻缓存

## 普通用户表的加载
- 当用户访问一个表时，系统首先会从表对象缓存池中查找这个表的share对象，如果找到了则直接从实例化的表空间链表中拿一个空闲的实例化表对象出来使用；如果没有，则重新实例化这个表


## rowid加载
- rowid只有在一个表没有定义主键时，需要rowid作为聚簇索引列才被分配
- 插入操作时，系统在内存中对rowid加1，rowid每到256的倍数是才写入一次
- 数据库启动时，函数dict_boot会将上次写入的rowid向上对齐256后再加上256（也就是会跳过一些ID）