## 索引基础
#### 索引的类型
##### B+索引
1. 不特指，就是指B+索引
2. Myisam使用前缀压缩技术使得索引更小，innodb按照原数据格式存储
3. Myisam索引使用数据的物理位置引用被索引的行（堆索引），innodb根据主键引用被索引的行（索引组织表）
4. B+tree适合全键值、键值范围、键最左前缀查找
B+树索引的限制
1. 如果不是最左列开始查找，则无法使用索引
2. 不能跳过索引中的列
3. 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引优化查找

##### 哈希索引
- 哈希索引基于哈希表实现，适用于精确匹配
- MySQL只有memory引擎显式支持哈希索引；innodb支持自适应哈希索引
- 哈希索引只包含哈希值和行指针，而无字段值，所以无法使用覆盖索引
- 不按索引值顺序存储，所以无法用于排序
- 不支持部分索引列的查找
- 只支持等值比较，不支持范围查询
- 哈希冲突很多的话，维护代价很高

伪哈希索引
- 新增一个列，值为索引列的哈希值
##### 空间数据索引（R-Tree）
- 使用MySQL GIS函数MBRCONTAINS()维护
- 对GIS支持 并不完善（做得较好的是PostGIS）
##### 全文索引
- 特殊的索引，查找的是文本中的关键词
- 全文索引适用于match agianst操作
#### 其他索引
TokuDB使用分形树索引（fractal tree index），scaleDB使用patricia tries
## 索引的优点
优点：
1. 索引大大减少服务器需要扫描的数据量
2. 索引可以帮助服务器避免排序和临时表
3. 索引可以将随机IO转化为顺序IO
（参阅relational database index design and optimizers）
## 高性能的索引策略
#### 独立的列
- 索引列不是表达式的一部分，也不是函数的参数
- ==始终将索引列单独放比较符号的一侧==
#### 前缀索引和索引的选择性
- 使用前缀索引，可以减少索引空间提高效率，但降低选择性
- 选择性，即不重复的索引值（cardinality）和数据表总数的比值
- blob，text或者很长的varchar，必须使用前缀索引，因为不允许索引全部长度

```
> create table index_test( a varchar(4096));
Query OK, 0 rows affected (0.02 sec)

> insert into index_test values(repeat("*",4096))

> create index index_test_idx on index_test(a);                               
ERROR 1170 (42000): BLOB/TEXT column 'a' used in key specification without a key length

> create index index_test_idx on index_test(a(3072));
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
- 前缀索引可提升效率，但无法使用前缀索引order by和group by，也无法使用覆盖索引
#### 多列索引
- 索引合并有时候是优化的结果，但更多时候说明索引建得糟糕
1. 如果服务器有索引相交的操作，说明需要一个相关列的组合索引
2. 如果服务器对索引做联合操作，要耗费大量的cpu、内存在缓存、排序、合并上，优化器只关心随机页面读取，所以这些不会放到查询成本中。往往不如走全表扫描。使用union方式更好。
3. 如果在explain的extra中有索引合并，要检查表结构并优化
4. 使用optimizer_switch可以关闭索引合并
5. 使用ignore index可以忽略索引

```
SELECT * FROM TABLE1 IGNORE INDEX (FIELD1, FIELD2)
```

#### 选择合适的索引列顺序
- 选择性最高的列放最前面（有用，但重要性不如随机IO与排序）

```
> select sum(s_w_id=1) ,sum(s_quantity=29) from stock;
+---------------+--------------------+
| sum(s_w_id=1) | sum(s_quantity=29) |
+---------------+--------------------+
|        100000 |               5477 |
+---------------+--------------------+
1 row in set (0.19 sec)
```

#### 聚簇索引
- 聚簇索引，是指数据行与相邻的键值紧凑地存储在一起（即索引组织表）
- 如果没有定义主键，innodb会隐式定义一个6字节的主键
- innodb只聚集同一个页面中的记录，相邻键值的页面可能相距甚远
- 聚簇索引优点：
1. 把相关的数据保存在一起，数据比较紧凑
2. 索引与数据保存在同一个B+tree中，数据访问更快
3. 可以使用覆盖索引
- 聚集索引缺点
1. 数据在内存中的话，聚簇索引没有优势
2. 插入速度严重依赖于插入顺序。如果不是按照主键顺序加载数据，则==加载完成后最好使用optimize table重新组织==一下表
3. 更新聚簇索引代价很高
4. 插入新行，或者主键更新导致行需要移动的时候，可能造成页分裂
5. 聚簇索引可能导致全表扫描变慢，尤其是行比较系数或者页分裂导致存储不连续的时候
6. 二级索引可能比想象的要更大，因为二级索引的叶子节点包含了引用行的主键列
7. 二级索引需要两次索引查找
- Myisam 和 innodb 的数据分布
1. Myisam数据分布很简单，按照数据插入的顺序存储在磁盘上
2. Myisam主键索引和非主键索引结构相同，主键索引就是一个==名为primary==的唯一非空索引
3. innodb索引结构
![innodb所以结构](https://github.com/ermaot/notes/blob/master/mysql/015%E6%9F%A5%E8%AF%A2%E8%AE%A1%E5%88%92%E4%B8%8E%E4%BC%98%E5%8C%96/pic/MySQL%E7%B4%A2%E5%BC%95.png)
4. 每一个叶子节点包含了主键值、事务ID、回滚指针（用于事务和mvcc）以及所有剩余列
5. innodb二级索引和聚簇索引很不同，二级索引叶子节点存储的是主键值（移动行的时候，无需更新二级索引中的指针）
- innodb最大的填充因子默认是15/16，最小是50%
#### 覆盖索引
查询只需要扫描索引而无须回表，成为覆盖索引
1. 索引数目通常远小于数据行，一方面便于查询，另一方面更容易放入内存提高性能（尤其是可压缩索引的Myisam）
2. 索引按照列值顺序存储，有利于范围查询
3. innodb的二级索引的叶子节点保存了行的主键值，如果二级主键能覆盖查询，则可以避免对主键索引的二次查询
- 不是所有的索引都能成为覆盖索引：覆盖索引必须存储索引列的值，而哈希索引、空间所用、全文索引都不可
- 当发起一个索引覆盖查询，explain的extra里可以看到using index信息
- 延迟关联（deferred join）

```
SELECT * FROM products J0IN (SELECT prod. _id FROM products WHERE actor= ' SEAN CARREY' AND title LIKE '%APOLL0%') AS t1 ON (t1.prod_ id=products .prod. id)
```
- MySQL目前的API设计不允许将过滤条件传递到存储引擎层，5.6以后会有索引条件推送（index condition pushdown）的改进
#### 使用索引扫描来排序
MySQL两种方式生成有序结果：
1. 排序操作
2. 按照索引顺序扫描（explain的type列值为index）
- 如果可能，设计索引时尽可能同时满足满足和查找
- 只有当索引的列顺序与order by 子句的顺序完全一致，并且所有列的排序方向都一样时，才能使用索引排序
- 如果排序需要关联多张表，则只有当order by子句引用的字段全部为第一个表时候，才能使用索引排序；order by排序要满足索引最左前缀要求或者前导列为常量
#### 压缩（压缩前缀）索引
- Myisam使用前缀压缩减少索引大小，默认只压缩字符串（==参数设置可以压缩整数==哪个参数）
- 压缩方法是先完全保存索引块中第一个值，然后将其他值与第一个值比较相同前缀字节数和剩余的不同后缀部分
- 压缩索引可能只需要十分之一大小的磁盘空间，对IO秘籍应用查询好处可能比较大
- create table语句中指定 pack_keys 参数控制压缩索引的方式
1. 如果您希望索引更小，则把此选项设置为1
2. 把选项设置为0可以取消所有的关键字压缩
3. 把此选项设置为DEFAULT时，存储引擎只压缩长的CHAR或VARCHAR列（仅限于MyISAM）

```
CREATE  TABLE <TABLE_NAME> (
`id` INT NOT NULL ,
`name` VARCHAR(250) NULL ,
PRIMARY KEY (`id`) )
PACK_KEYS = 1;
ALTER TABLE table_name PACK_KEYS = 1;
```

#### 冗余和重复索引
- 重复索引是指在相同的列上按照相同的顺序创建的相同类型的索引，应该马上被移除
- 冗余索引大部分情况下要被移除
找到冗余重复索引方法：
1. 复杂查询访问information_schema
2. ==common_schema==（shlomi Noach）==写一篇==
3. percona toolkit中的pt-duplicate-key-checker
#### 未使用的索引
- 永远也不用的索引，应该被删除
- userstates 变量打开索引统计，然后查询 information_schema.index_statistics 可以查看索引被使用的频率(mysql8使用performance_schema.table_io_waits_summary_by_index_usage)（==关于这部分，写一篇）<p>[MySQL索引统计信息更新相关的参数](https://www.cnblogs.com/phpper/p/6592831.html)
- pt-index-usage
#### 索引和锁
- 索引让查询锁定更少的行
- innodb只有在访问行的时候才会对起加锁
- 如果索引无法过滤掉无效的行，那么innodb检索到数据并返回给服务器层之后MySQL才能用where子句过滤
- 在MySQL5.1之前，innodb只有在事务提交之后才能释放掉锁；MySQL5.1以及以后的版本innodb可以在服务器端过滤掉行之后就释放
- innodb在二级索引上使用共享（读），但访问主键索引需要排他（写）锁。这消除了覆盖索引的可能性，并且是的select for update比lock in share mode或者非锁定查询要慢很多（==不太明白？？==）
## 索引案例学习
#### 支持多种过滤条件
#### 避免多个范围条件
#### 优化排序
## 维护索引和表
#### 找到并修复损坏的表
- check table通畅能找出大多数的表和索引的错误

```
> check table warehouse;
+----------------+-------+----------+----------+
| Table          | Op    | Msg_type | Msg_text |
+----------------+-------+----------+----------+
| test.warehouse | check | status   | OK       |
+----------------+-------+----------+----------+
1 row in set (0.00 sec)
```

- repair table修复损坏的表（支持Myisam）

```
> repair table  warehouse;
+----------------+--------+----------+---------------------------------------------------------+
| Table          | Op     | Msg_type | Msg_text                                                |
+----------------+--------+----------+---------------------------------------------------------+
| test.warehouse | repair | note     | The storage engine for the table doesn't support repair |
+----------------+--------+----------+---------------------------------------------------------+
```
- 更改存储引擎，以重建表
- 离线工具myisamchk
- 将数据导出再重新导入
- 如果损坏的是系统区域或者表的行数据区域，而不是索引，上面办法皆无效，则可以从备份中恢复或者从损坏的数据文件中尽可能恢复

#### 更新索引统计信息
MySQL通过两个API了解存储引擎索引值的分布信息
1. 第一个API是records_in_range()，向存储引擎传入两个边界值获取在这个范围大概有多少条记录；Myisam返回精确值，innodb返回估算值
2. 第二个API是info()，返回各种类型的数据，包括索引的基数（每个键值有多少记录）
- 如果存储引擎向优化器提供的扫描行数的信息是不准确的，或者执行计划本身太复杂以至于无法准确获取各个阶段匹配的行数，优化器会使用索引统计信息来估算
- ==MySQL优化器使用的是基本成本的模型==，衡量成本的主要指标是查询的扫描行数
- 运行analyze table重新生成统计信息
1. memory引擎不存储索引的统计信息
2. Myisam将索引统计信息存储在磁盘中，analyze table需要全表扫描而锁表
3. MySQL5.5以及之前，innodb不在磁盘存储索引统计信息，而是通过随机索引访问进行评估并存储在内存中（每次服务器重启会丢失并重新统计）
- cardinality
1. show index from table_name查看cardinality
warehouse有5行，但cardinality是4，analyze之后，cardinality与行数相同
```
> show index from warehouse;
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table     | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| warehouse |          0 | PRIMARY  |            1 | w_id        | A         |           4 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
1 row in set (0.01 sec)

> analyze table warehouse;
+----------------+---------+----------+----------+
| Table          | Op      | Msg_type | Msg_text |
+----------------+---------+----------+----------+
| test.warehouse | analyze | status   | OK       |
+----------------+---------+----------+----------+
1 row in set (0.01 sec)

> show index from warehouse;
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table     | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| warehouse |          0 | PRIMARY  |            1 | w_id        | A         |           5 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
1 row in set (0.00 sec)
```
2. information_schema.statistics

```
> select * from information_schema.statistics where table_name ="warehouse" ;
+---------------+--------------+------------+------------+--------------+------------+--------------+-------------+-----------+-------------+----------+--------+----------+------------+---------+---------------+------------+------------+
| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | NON_UNIQUE | INDEX_SCHEMA | INDEX_NAME | SEQ_IN_INDEX | COLUMN_NAME | COLLATION | CARDINALITY | SUB_PART | PACKED | NULLABLE | INDEX_TYPE | COMMENT | INDEX_COMMENT | IS_VISIBLE | EXPRESSION |
+---------------+--------------+------------+------------+--------------+------------+--------------+-------------+-----------+-------------+----------+--------+----------+------------+---------+---------------+------------+------------+
| def           | test         | warehouse  |          0 | test         | PRIMARY    |            1 | w_id        | A         |           5 |     NULL |   NULL |          | BTREE      |         |               | YES        | NULL       |
+---------------+--------------+------------+------------+--------------+------------+--------------+-------------+-----------+-------------+----------+--------+----------+------------+---------+---------------+------------+------------+
1 row in set (0.00 sec)
```
- innodb 通过抽样获取统计信息：首先随机读取少量索引页面，然后以此为样本计算索引的统计信息（老版本样本页面是8，新版本innodb通过参数 innodb_stats_sample_pages 设置。mysql8不支持该参数，而使用 innodb_stats_transient_sample_pages ）[参考MySQL索引统计信息更新相关的参数](https://www.cnblogs.com/tangshiguang/p/6741037.html)

```
mariadb5.5
> select @@innodb_stats_sample_pages;
+-----------------------------+
| @@innodb_stats_sample_pages |
+-----------------------------+
|                           8 |
+-----------------------------+
1 row in set (0.10 sec)

mysql8
> select @@innodb_stats_sample_pages;
ERROR 1193 (HY000): Unknown system variable 'innodb_stats_sample_pages'
```
- innodb会在表首次打开，或者analyze table或者表的大小发生非常大的变化（大小变化超过十六分之一和插入了20亿行都会触发）索引统计信息的计算
- innodb在打开某些information_schema表或者使用show table status和show index，或MySQL客户端打开自动补全功能都会触发索引统计信息的更新。这可能造成严重问题，可以关闭 innodb_stats_on_metadata 关闭统计
#### 减少索引和数据的碎片
有三种类型的碎片
1. 行碎片（row fragmentation）：数据行被存储多个地方的多个片段中
2. 行间碎片（intra-row fragmentation）：逻辑上顺序的页或者行在磁盘上不是顺序存储的，对全表扫描和聚簇索引扫描有很大影响
3. 剩余空间碎片（free space fragmentation）：数据页有大量剩余空间，造成读浪费
- Myisam三种都会发生；innodb不会出现短小的行碎片，会移动短小的行并重写到一个片段中
- optimize table 或者导入再导出的方式重整数据（innodb不支持）
- 新版本的innodb新增添加和删除索引的功能，重建索引来消除碎片化
- alter table <table> engine=<engine>（engine不变，还是原表的engine）可以重建表

```
> alter table item engine=innodb;
Query OK, 0 rows affected (1.59 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
- xtrabackup有--stats参数以非备份方式运行，可以打印索引和表的统计情况，包括页中的数据量和空余空间