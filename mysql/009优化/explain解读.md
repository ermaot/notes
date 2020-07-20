使用MySQL的开发人员或者DBA，基本都知道explain工具，也会使用并定位一些问题。但并不一定了解全部的内容。本篇就对MySQL的explain工具做一个全面的解读。

本文参考：https://www.cnblogs.com/gomysql/p/3720123.html
https://www.cnblogs.com/renchengtianshao/p/9816085.html

https://www.cnblogs.com/duanxz/p/4470607.html

## 一、explain简介

EXPLAIN命令是查看优化器如何决定执行查询的主要方法。

1. 可以帮助我们深入了解MySQL的基于开销的优化器
2. 还可以获得很多可能被优化器考虑到的访问策略的细节
3. 以及当运行SQL语句时哪种策略预计会被优化器采用。

需要注意的是，生成的QEP并不确定，它可能会根据很多因素发生改变。MySQL不会将一个QEP和某个给定查询绑定，QEP将由SQL语句每次执行时的实际情况确定，即便使用存储过程也是如此。尽管在存储过程中SQL语句都是预先解析过的，但QEP仍然会在每次调用存储过程的时候才被确定。

## 二、explain的基本使用

```
> explain select count(*) from partition_perf1 use index(partition_perf1_idx) where b<1545000000;
+------+-------------+-----------------+-------+---------------------+---------------------+---------+------+---------+--------------------------+
| id   | select_type | table           | type  | possible_keys       | key                 | key_len | ref  | rows    | Extra                    |
+------+-------------+-----------------+-------+---------------------+---------------------+---------+------+---------+--------------------------+
|    1 | SIMPLE      | partition_perf1 | range | partition_perf1_idx | partition_perf1_idx | 5       | NULL | 4194060 | Using where; Using index |
+------+-------------+-----------------+-------+---------------------+---------------------+---------+------+---------+--------------------------+
1 row in set (0.00 sec)
```

#### 一般方式

```
EXPLAIN SELECT ……
```

#### 变体：

1. EXPLAIN EXTENDED SELECT ……

将执行计划"反编译"成SELECT语句，运行SHOW WARNINGS,可得到被MySQL优化器优化后的查询语句

```
> explain extended  select count(*) from partition_perf1 use index(partition_perf1_idx) where b<1545000000;
+------+-------------+-----------------+-------+---------------------+---------------------+---------+------+---------+----------+--------------------------+
| id   | select_type | table           | type  | possible_keys       | key                 | key_len | ref  | rows    | filtered | Extra                    |
+------+-------------+-----------------+-------+---------------------+---------------------+---------+------+---------+----------+--------------------------+
|    1 | SIMPLE      | partition_perf1 | range | partition_perf1_idx | partition_perf1_idx | 5       | NULL | 4194060 |   100.00 | Using where; Using index |
+------+-------------+-----------------+-------+---------------------+---------------------+---------+------+---------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)

> show warnings;
+-------+------+-------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                         |
+-------+------+-------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1003 | select count(0) AS `count(*)` from `perf`.`partition_perf1` USE INDEX (`partition_perf1_idx`) where (`perf`.`partition_perf1`.`b` < 1545000000) |
+-------+------+-------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```



2. EXPLAIN PARTITIONS SELECT ……
   用于分区表的EXPLAIN生成QEP的信息

## 三、explain各项信息的解释

#### 1. id

- 包含一组数字，表示查询中执行select子句或操作表的顺序
- 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
- id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行
- id大小并不一定从上到下排列，也就是说，执行顺序不一定是从上到下或者从下到上

#### 2.select_type

查询中每个select子句的类型（简单OR复杂）
类型|说明
---|---
SIMPLE|查询中不包含子查询或者UNION
PRIMARY|查询中若包含任何复杂的子部分，最外层查询
SUBQUERY| 在SELECT或WHERE列表中包含了子查询
DERIVED|在FROM列表中包含的子查询被标记为：DERIVED（衍生）用来表示包含在from子句中的子查询的select，mysql会递归执行并将结果放到一个临时表中。服务器内部称为"派生表"，因为该临时表是从子查询中派生出来的
UNION|若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
UNION RESULT|从UNION表获取结果的SELECT被标记为：UNION RESULT
DEPENDENT|select依赖于外层查询中发现的数据
UNCACHEABLE|意味着select中的某些 特性阻止结果被缓存于一个item_cache中

#### #### 3.type

表示MySQL在表中找到所需行的方式，又称“访问类型”，常见类型如下：
类型|说明
---|---
ALL|Full Table Scan， MySQL将遍历全表以找到匹配的行
index|Full Index Scan，index与ALL区别为index类型只遍历索引树
range|索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行。显而易见的索引范围扫描是带有between或者where子句里带有<, >查询。当mysql使用索引去查找一系列值时，例如IN()和OR列表，也会显示range（范围扫描）,当然性能上面是有差异的。
index_subquery|子查询中的返回结果字段组合是一个索引（或索引组合），但不是一个主键或唯一索引
unique_subquery|子查询中的返回结果字段组合是主键或唯一约束
index_merge|查询中同时使用两个（或更多）索引，然后对索引结果进行合并（merge），再读取表数据
ref_or_null|与ref的唯一区别就是在使用索引引用的查询之外再增加一个空值的查询
fulltext|进行全文索引检索
ref|使用非唯一索引扫描或者唯一索引的前缀扫描，返回匹配某个单独值的记录行
eq_ref|类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
const|当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量
system|system是const类型的特例，当查询的表只有一行的情况下，使用system
NULL|MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成

从上到下，性能从最差到最好

#### 4.possible_keys

指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用

#### 5.key

显示MySQL在查询中实际使用的索引，若没有使用索引，显示为NULL

#### 6.key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）

#### 7. ref
表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

#### 8. rows
表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数

#### 9.Extra
包含不适合在其他列中显示但十分重要的额外信息
类型|说明
---|---
Using index|该值表示相应的select操作中使用了覆盖索引（Covering Index）
Using where|表示mysql服务器将在存储引擎检索行后再进行过滤。<br>许多where条件里涉及索引中的列，当（并且如果）它读取索引时<br>就能被存储引擎检验，因此不是所有带where字句的查询都会显示"Using where"。有时"Using where"的出现就是一个暗示：查询可受益于不同的索引。
Using temporary|表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询。<br>两个常见的原因是在来自不同表的上使用了DISTINCT<br>或者使用了不同的ORDER BY和GROUP BY列
Using filesort|MySQL中无法利用索引完成的排序操作称为“文件排序”
Using join buffer|改值强调了在获取连接条件时没有使用索引，<br>并且需要连接缓冲区来存储中间结果。如果出现了这个值，<br>那应该根据查询的具体情况可能需要添加索引来改进能
Impossible where|这个值强调了where语句会导致没有符合条件的行。
Select tables optimized away|这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行
Index merges|当MySQL 决定要在一个给定的表上使用超过一个索引的时候，<br>就会出现以下格式中的一个，详细说明使用的索引以及合并的类型
Using where with pushed condition|这是一个仅仅在 NDBCluster存储引擎中才会出现的信息，<br>而且还须要通过打开 Condition Pushdown 优化功能才可能被使用。<br>控制参数为 engine_condition_pushdown 。
Impossible WHERE noticed after reading const tables|MySQL Query Optimizer 通过收集到的统计信息判断出不可能存在结果。
No tables|Query 语句中使用 FROM DUAL或不包含任何 FROM子句。
Not exists|在某些左连接中，MySQL Query Optimizer通过改变原有 Query 的组成<br>而使用的优化方法，可以部分减少数据访问次数
Using index for group-by|数据访问和Using index一样，所需数据只需要读取索引即可，<br>而当Query中使用了GROUPBY或者DISTINCT子句的时候，<br>如果分组字段也在索引中，Extra中的信息就会是Using index for group-by
Distinct|查找distinct值，所以当mysql找到了第一条匹配的结果后，将停止该值的查询而转为后面其他值的查询
Full scan on NULL key|子查询中的一种优化方式，主要在遇到无法通过索引访问null值的使用使用；

## 四、需要关注的关键点

# EXPLAIN结果中哪些信息要引起关注

我们使用EXPLAIN解析SQL执行计划时，如果有下面几种情况，就需要特别关注下了：

#### 首先看下 **type** 这列的结果

如果有类型是 ALL 时，表示预计会进行全表扫描（full table scan）。通常全表扫描的代价是比较大的，建议创建适当的索引，通过索引检索避免全表扫描。此外，全索引扫描（full index scan）的代价有时候是比全表扫描还要高的，除非是基于InnoDB表的主键索引扫描。

####再来看下 **Extra** 列的结果，

如果有出现 **Using temporary** 或者 **Using filesort** 则要多加关注：

**Using temporary**，表示需要创建临时表以满足需求，通常是因为GROUP BY的列没有索引，或者GROUP BY和ORDER BY的列不一样，也需要创建临时表，建议添加适当的索引。

**Using filesort**，表示无法利用索引完成排序，也有可能是因为多表连接时，排序字段不是驱动表中的字段，因此也没办法利用索引完成排序，建议添加适当的索引。

**Using where**，通常是因为全表扫描或全索引扫描时（**type** 列显示为 **ALL** 或 **index**），又加上了WHERE条件，建议添加适当的索引。