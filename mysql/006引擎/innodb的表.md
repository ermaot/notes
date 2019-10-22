## innodb存储表引擎类型
- 类似oracle中的索引组织表
- 每张表都有主键
- 若没有显式定义，则
1. 查看是否有非空唯一索引（unique not null），如果有则为主键
2. 如果没有，则自动创建一个6字节的指针

## innodb逻辑存储结构
#### 表空间
表空间由段（segment）组成，段由区（extent）组成，区由页（page）组成。页又叫块（block）
![innodb逻辑结构](850C67D1D25B4A9FA92A0088B96BF49E)
- innodb_file_per_table
1. 如果该参数启用，存储表的数据、索引、插入缓冲等信息会存入单独的.ibd中，其余信息（undo信息，事务信息，二次写缓冲等）还是存放默认共享表空间
2. 共享表空间依旧会随着undo操作增大
3. 回滚不会自动回收undo表空间，但会标记为可用空间并被重用
- py_innodb_page_info 小工具
#### 段
- 段有数据段、索引段、回滚段
- innodb是索引组织表，所以数据即索引，索引即数据
- B+的叶节点即数据段（leaf node segment）
- B+的非叶节点即索引段（non-leaf node segment）
- oracle有ASSM和MSSM，mysql只有段的自动管理由引擎本身完成
- 并非每个对象都有段
- 表空间由分散的页和段组成（？）
#### 区
- 区由64个连续的页组成，每个页大小16KB，所以每个区大小1MB
- 每个段开始时，先有32个碎片页（fragment page）存放数据
#### 页
- 页（page），也叫块（block），是innodb磁盘管理的最小单位
- 默认大小16KB，oracle和sql server默认大小8KB，postgresql默认大小也是8KB
- 数据页（B-Tree Node），Undo页（Undo log page），系统页（system page），事务数据页（transaction system page），插入缓冲位图页（insert buffer bitmap page），插入缓冲空闲列表页（insert buffer free list），未压缩的二进制大对象页（uncompressed BLOB page），压缩的二进制大对象页（compressed BLOB page）
#### 行
- innodb 是面向行的（row-oriented），即按行存放
- 每页最多存放16KB /2 -200 = 7992 行
- mysql infobright是列存数据库（column-oriented）
## 物理存储结构
- innodb表由共享表空间、日志文件组（redo log文件组）、表结构定义文件组成
- 若innodb_file_per_table为on，则每个表会产生单独的表空间.ibd文件，数据、索引、表结构的数据字典信息会保存在该表空间中
- 表结构以.frm结尾
## 行结构
1. mysql5.1以后使用compat行格式，查看表的时候Row_format表示表结构的类型

```
> show table status like "t"\G
*************************** 1. row ***************************
           Name: t
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 10
 Avg_row_length: 1638
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 437256192
 Auto_increment: NULL
    Create_time: 2019-05-24 11:41:29
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)

```

2. compat格式
![compat格式](B9AE4C8B01B84DD580E2BEF3AC1C92DB)
首部是变长字段长度列表，最大为2字节（所以mysqlvarchar最大长度为2^16-1 = 65535）<p>
NULL标志位，所用字节为byte<p>
记录头

名称 | 大小（位） | 描述
---|--- | ---
（） | 1 | 未知
（） | 1 | 未知
delete_flag | 1 | 该行是否已经被删除
min_rec_flag | 1 | col3
n_owned | 4 | 该记录拥有的记录数
heap_no | 13 | 索引堆中该记录的排序记录
recored_type | 3 | 记录类型：000=普通 001=B+树节点指针 010=infimum 011=supermum 1xx=保留
next_recorder | 16 | 页中下一条记录的相对未知
合计 | 40 | 即5个字节

隐藏列：两个（事务ID列（6个字节）和回滚指针列（7个字节））<p>
==null不占存储空间==
3. 行溢出数据<p>
innodb可以将一行数据存储在页面之外，即行溢出数据<p>
varchar在sql_mode为非严格模式，会将大的varchar转化成text
```
> create table varch(a varchar(65535));
Query OK, 0 rows affected, 1 warning (0.50 sec)

MariaDB [test]> show warnings;
+-------+------+--------------------------------------------+
| Level | Code | Message                                    |
+-------+------+--------------------------------------------+
| Note  | 1246 | Converting column 'a' from VARCHAR to TEXT |
+-------+------+--------------------------------------------+

> show create table varch;
+-------+------------------------------------------------------------------------------+
| Table | Create Table                                                                 |
+-------+------------------------------------------------------------------------------+
| varch | CREATE TABLE `varch` (  `a` mediumtext) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+------------------------------------------------------------------------------+

```
varchar(n)的n指字符长度，而最大的65535是指字节<p>
varchar的65535是一行所有varchar的长度之和不能超过65535<p>
innodb plugin采用了新的页格式，对blob完全采取了行溢出方式，之前存储了前768个字节<p>
innodb plugin采用zlib压缩数据，对blob，text，varchar可有效存储
4. char的存储<p>
char(n)的n是指字符长度，也就是==char的字节长度可变==，所以varchar与char的行存储基本没有区别
## innodb的页结构

innodb的页结构 |说明
---| ---|
file header（文件头） | 8部分，38字节<p>fil_page_space_or_checksum（4字节） mysql4.1之后表示checksum值<p>fil_page_offset（4字节）表空间中页的偏移值<p>fil_page_prev（4字节）,fil_page_next（4字节），顾名思义，表示前一个页面和后一个页面的指针<p>fil_page_LSN（8字节）该页最后修改的LSN<p>fil_page_type（2字节），页类型<p>fil_page_file_flush_LSN（8字节），在数据文件的某一个页才有，代表文件至少被更新到该LSN<p>fil_page_arch_log_no_or_space_id（4字节）mysql4.1开始代表数据页属于哪个表空间
page header（页头） | 14部分，56字节<p>PAGE_N_DIR_SLOTS（2字节） page directory 的slots数<p> PAGE_HEAP_TOP（2字节） 堆中第一个记录的指针<p> PAGE_N_HEAP（2字节）  堆中的记录数 <p> PAGE_FREE（2字节）指向空闲列表的首指针<p>PAGE_GARBAGE（2字节）已删除的字节数，即行记录结构中 delete flag为1的记录的数目<p>PAGE_LAST_INSERT（2字节） 最后插入记录的位置<p> PAGE_DIRECTION（2字节）插入的方向 <p> PAGE_N_DIRECTION（2字节）一个方向连续插入的数量<p>PAGE_N_RECS（2字节）该页中记录的数量<p> PAGE_MAX_TRX_ID（8字节）当前页的最大事务ID<p> PAGE_LEVEL（2字节）当前页在索引中的位置，0x00代表叶子节点<p> PAGE_INDEX_ID（8字节）当前页属于哪个索引ID <p> PAGE_BTR_SEG_LEAF（10字节）B+树叶节点中文件段的首指针位置，仅B+树的root页中定义<p>PAGE_BTR_SEG_TOP（10字节）B+树非叶节点中文件段的首指针位置，仅B+树的root页中定义<p>
infimum + supremum records | innodb 每个数据页会有两个虚拟行记录以限定记录边界，infimum记录比任何主键还小的值，supremum记录比任何主键还大的值<p>页建立的时候创建，并任何情况都不会删除
user records（用户记录，即行记录） | innodb是索引组织表
free space（空闲空间） | 链表数据结构；记录被删除时，会加入到空闲链表中
page directory（页目录） | 存放了记录的相对位置
file trailer（文件结尾信息） | 为了页面写入时候的完整性，file trailer只有8字节的FIL_PAGE_END_LSN<p>前4个字节是本页的checksum，后四个字节与 FIL_PAGE_LSN 的LSN相同<p>前4字节与fil_page_space_or_checksum比较，后4字节与FIL_PAGE_LSN比较，确认是否相同

## named file formats
- innodb_file_format 指定文件格式

```
> select @@version;
+----------------+
| @@version      |
+----------------+
| 5.5.56-MariaDB |
+----------------+

> show variables like "innodb_version";
+----------------+---------------------+
| Variable_name  | Value               |
+----------------+---------------------+
| innodb_version | 5.5.52-MariaDB-38.3 |
+----------------+---------------------+


> show variables like "innodb_file_format";
+--------------------+----------+
| Variable_name      | Value    |
+--------------------+----------+
| innodb_file_format | Antelope |
+--------------------+----------+

```

- innodb_file_format_check 检测当前innodb文件格式的支持度，默认为ON

```
> show variables like "innodb_file_format_check";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_file_format_check | ON    |
+--------------------------+-------+

```

## 约束
#### 数据完整性
- **实体完整性**<p>
保证表中有一个主键
- **域完整性**<p>
保证数据的值满足一定的条件，有以下途径保证
1. 选择特定和合适的类型的值
2. 外键foreign key
3. 编写触发器
4. default约束
- **参照完整性<p>**
保证两个表之间的参照关系，有以下途径：
1. 外键foreign key
2. 编写触发器以强制执行
- **innodb提供了5种约束**（原书中说4种，应该是笔误）
1. primary key
2. unique key
3. foreign key
4. default
5. NOT NULL
#### 约束的创建与查找
- 创建方式：建表的时候创建；alter table修改
- create unique index，默认约束名与列名一样
- primary key默认约束名称就叫==primary==
- 例子

```
> create table cons(a int,b int not null,primary key(a),unique key(b));

> show index from cons;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| cons  |          0 | PRIMARY  |            1 | a           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| cons  |          0 | b        |            1 | b           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+

> select * from information_schema.table_constraints where table_schema="test" and table_name = "cons";
+--------------------+-------------------+-----------------+--------------+------------+-----------------+
| CONSTRAINT_CATALOG | CONSTRAINT_SCHEMA | CONSTRAINT_NAME | TABLE_SCHEMA | TABLE_NAME | CONSTRAINT_TYPE |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+
| def                | test              | PRIMARY         | test         | cons       | PRIMARY KEY     |
| def                | test              | b               | test         | cons       | UNIQUE          |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+

> alter table cons add unique key cons_a (a);
Query OK, 0 rows affected (0.16 sec)
Records: 0  Duplicates: 0  Warnings: 0

> alter table cons drop index cons_a ;
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

> create table cons2( a int,b int ,primary key(a ),foreign key (a) references cons(a));
Query OK, 0 rows affected (0.06 sec)

> select * from information_schema.table_constraints where table_schema="test" and table_name = "cons2";
+--------------------+-------------------+-----------------+--------------+------------+-----------------+
| CONSTRAINT_CATALOG | CONSTRAINT_SCHEMA | CONSTRAINT_NAME | TABLE_SCHEMA | TABLE_NAME | CONSTRAINT_TYPE |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+
| def                | test              | PRIMARY         | test         | cons2      | PRIMARY KEY     |
| def                | test              | cons2_ibfk_1    | test         | cons2      | FOREIGN KEY     |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+

> alter table cons2 drop foreign key cons2_ibfk_1;
Query OK, 0 rows affected (0.08 sec)               
Records: 0  Duplicates: 0  Warnings: 0

```

#### 约束与索引的区别
- 索引既是逻辑概念也是物理概念
- 约束是逻辑概念，保证数据完整性
#### 对错误数据的约束
- 如果sql_mode不为"strict_trans_tables",即使列为not null，插入null时会被转化为数值0，并提示warning
- sql_mode

```
> set  sql_mode = "strict_trans_tables";
Query OK, 0 rows affected (0.00 sec)

```

#### enum 和 set约束

```
> create table enum_test(a int,sex enum("male","female"));
> set  sql_mode = "strict_trans_tables";
> insert into enum_test values(1,"male");
Query OK, 1 row affected (0.01 sec)

------此处疑问需进一步调查--------
> insert into enum_test values(a,"male");
Query OK, 1 row affected (0.01 sec)

> insert into enum_test values(1,"bi");
ERROR 1265 (01000): Data truncated for column 'sex' at row 1
```

#### 触发器与约束
- 最对可以为一个表创建6个触发器
- 
#### 外键
## 视图
- 视图是一个命名的虚表，由一个查询定义，无物理形式（物化视图除外）
- 存在可更新视图（updatable view）

```
> create table view_test(a int);
> create view  view_test2 as select * from view_test where a<10;
> insert into view_test2 values(20);
Query OK, 1 row affected (0.05 sec)

> select * from view_test2;
Empty set (0.01 sec)

> select * from view_test;
+------+
| a    |
+------+
|   20 |
+------+
1 row in set (0.00 sec)

> alter view view_test2 as select * from view_test where a<10 with check option;
Query OK, 0 rows affected (0.30 sec)

> insert into view_test2 values(20);
ERROR 1369 (HY000): CHECK OPTION failed 'test.view_test2'
```
- 物化视图
1. oracle的物化视图是根据基表更新的实表，适用于复杂查询，避免联接、聚集等耗时多的操作
2. oracle 创建物化视图有build immediate （创建视图时即生成数据）与build deferred（创建时不生成，需要的时候生成）
3. 查询重写是指对物化视图的基表查询的时候，是否可以通过物化视图来优化操作
4. 物化视图的刷新是发生DML后，视图如何与基表进行同步。有on demand（用户需要的时候刷新） 和on commit（DML提交的时候刷新）<p>刷新方式有:fast（增量刷新），complete（完全刷新），force（oracle判断是否可快速刷新，若可，则fast，否则complete），never（从不刷新）
5. mysql不支持物化视图。可以使用变通方式实现类似功能
## 分区表
- 分区表不是在引擎层实现，所以很多引擎都支持（但并非全部），比如innodb，MyISAM，NDB等
- mysql支持水平分区，但不支持垂直分区
- mysql支持range，list，hash，key分区