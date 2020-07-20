## innodb索引概述
- 支持B+树索引，支持哈希索引
- B+树索引类似二叉树，根据键值对（key-value）来查找数据
- B不代表binary，而是balance，B+树不是二叉树，而是平衡二叉树演化来的平衡树

## 二分查找法
## 平衡二叉树
## B+树
- 填充因子。mysql最小的填充因子可设置为50%
- 删除操作必须保证删除后记录是有序的
- 扇出（fan out）：
## B+树索引
- innodb是索引组织表（IOT），数据按照主键顺序存放
- 聚集索引
1. 按照表的主键构造B+树，并且叶节点存放整张表的行记录数据，因此也让数据的叶子节点成为数据页
2. 索引组织表中的数据也是索引的一部分
3. 每一个页都是通过双向的链表连接
4. 每一张表只能有一个聚集索引
5. 聚集索引能使我们直接在叶节点上找到数据，因此查询优化器倾向于使用聚集索引
6. 覆盖索引。如果一个索引包含(或覆盖)所有需要查询的字段的值，称为‘覆盖索引’。即只需扫描索引而无须回表。
7. 聚集索引可以很快查询范围数据
- 非聚集索引
1. 又称辅助索引，叶级别不包含全部的数据
2. 每一个叶级别的索引行中包含了书签（bookmark），指向索引对应的行数据
3. 每一张表可以有多个辅助索引
4. 使用辅助索引查找数据时，innodb遍历辅助索引并通过叶级别的指针获取指向主键索引的主键，然后通过主键索引来找到完整的行记录（回表操作）
- 堆表
1. sql server 有一种非索引组织表，称为堆表。
2. 存放按照插入顺序，与MyISAM有点类似
3. 堆表索引都是非聚集的，也没有主键
4. 书签是一个行标识符（row identifier，RID），比如“文件号：页号：槽号”
5. 由于堆表不需要使用主键访问索引组织表，因此某些情况下比索引组织表快。但具体情况具体分析
- B+ 树索引的管理
1. 索引的创建和删除可以有两种方法，alter table 或者 create index / drop index

```
> create table index_test(a int primary key,b int );
]> show index from index_test;
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table      | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| index_test |          0 | PRIMARY  |            1 | a           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+

> drop index PRIMARY on index_test;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'PRIMARY on index_test' at line 1

> create index index_sec on index_test(b);
> drop index index_sec  on index_test;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

> alter table index_test drop  primary key;
Query OK, 0 rows affected (0.11 sec)               
Records: 0  Duplicates: 0  Warnings: 0

```

2. mysql索引添加或者删除操作：先创建一张临时表，然后导入数据，然后删除原表，再修改为原表的表名
3. 表很大的时候，删除或者重建索引很耗时
4. 快速索引方法（针对辅助索引）：创建过程中对表加S锁，速度较快但同时不可更新；删除索引只需要将辅助索引的空间标注为可用，同时在mysq内部视图将其定义删除
5. 各列的解释

列名 | 解释
---|---
table| 表名
Non_unique  |  是否唯一
key_name  |  索引名称（如果是联合索引，表中可能会有相同列名）
seq_in_index  |  列在索引中的序号（联合索引的每一列在表中都是单独存放的）
column_name |列名
collation | 列以什么方式存储在索引中。B+总是A，即排序的<p>如果使用了heap引擎并建立了hash索引则为NULL
Cardinality  |  表示索引中唯一数目的估计值
sub_part  |  表示列是否部分索引。数字1表示索引列C的前1个字符
packed  |  关键字的压缩方式，如果未被压缩则为NULL
Null  |  表示是否可以为null
index_type  |  索引类型，可以为BTREE，HASH
comment  |  注释
index_comment  |  索引注释



```
> create table index_test(a int primary key,b int );
> create index index_test_u on index_test(a,b);
> alter table index_test add column c varchar(100);
> create index index_test_part1 on index_test (c(1));
> show index from index_test;
+------------+------------+------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table      | Non_unique | Key_name         | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+------------+------------+------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| index_test |          0 | PRIMARY          |            1 | a           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| index_test |          1 | index_test_u     |            1 | a           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| index_test |          1 | index_test_u     |            2 | b           | A         |           0 |     NULL | NULL   | YES  | BTREE      |         |               |
| index_test |          1 | index_test_part1 |            1 | c           | A         |           0 |        1 | NULL   | YES  | BTREE      |         |               |
+------------+------------+------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+

```
6. Cardinality 值很关键，是一个估计值，并且每次执行可能不太一样。可使用analyze table 表名来更新该值
7. Cardinality 为null可能是索引建立了但实际上并没有用到
8. 非高峰时间，对核心表做analyze table的操作
## B+树索引的使用
- 在高选择性、取少量数据的时候，使用索引
- 顺序读（顺序读磁盘上的块）与随机读（离散读磁盘上的块）
- write back 与 write through<p>
Cache写机制分为write through和write back两种
1. Write-through（直写模式）在数据更新时，同时写入缓存Cache和后端存储。此模式的优点是操作简单；缺点是因为数据修改需要同时写入存储，数据写入速度较慢
2. Write-back（回写模式）在数据更新时只写入缓存Cache。只在数据被替换出缓存时，被修改的缓存数据才会被写到后端存储。此模式的优点是数据写入速度快，因为不需要写存储；缺点是一旦更新后的数据未被写入存储时出现系统掉电的情况，数据将无法找回。
## 哈希算法
#### 哈希
- 哈希（hash）表，又称散列表，由直接寻址表改进而来
- 两个不同的值可能会映射到同一个槽上，这叫碰撞（collsion）
- 数据库解决碰撞的方式比较简单：把散列同一个槽的数据都放同一个链表中
- 哈希函数：一般将关键字转化为自然数，然后通过==除法散列==、乘法散列、全域散列来实现。<p>
h(k) ≡ k mod m
#### innodb 的哈希算法
- innodb 使用哈希算法对字典进行查找，冲突解决采用链表方式，哈希函数采用除法散列
- 除法散列，m取略大于2倍缓存池页数量大小的质数
- 哈希表本身内存由innodb_additional_mem_pool_size分配（自动分配，无参数可控制），而槽内存由系统分配
- innodb如何查找页：innodb 的表空间都有一个space号，将存储引擎左移20位，然后找某个表空间连续的16KB页的offset<p>
关键字k = space << 20 + space + offset 
#### 自适应哈希索引
- innodb_adaptive_hash_index 可以启用自适应哈希索引
- 数据库启动的时候会创建 innodb_buffer_pool_size / 256 个槽数的哈希表
- 查看哈希索引使用状况

```
show engine innodb status
……………………
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 18749, node heap has 38 buffer(s)
472.06 hash searches/s, 326.21 non-hash searches/s
………………
```

## 小结