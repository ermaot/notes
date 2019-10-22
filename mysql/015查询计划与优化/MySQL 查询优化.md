## 为什么查询会慢
- 查询生命周期：从客户端，到服务器，然后服务器解析，生成执行计划，执行，返回结果给客户端
- 其中执行阶段最为重要，包含了大量为检索数据到存储引擎的调用和后面的数据处理（排序、分组）等
- 花费时间组成：网络，cpu计算，生成统计信息和执行计划，锁等待（调用存储引擎需要内存、cpu、io、上下文切换、系统调用等）
## 慢查询基础：优化数据访问
大部分性能地下的查询都可以通过减少访问数据量的方式优化
1. 确认应用程序是否在检索大量超过真正所需的数据
2. 确认MySQL是否在分析大量超过需要的数据行
#### 是否向服务器请求了不需要的数据
有的查询会请求超过实际需要的数据，然后把数据丢弃，额外消耗cpu、内存、io和网络
- 查询不需要的记录：select 大量结果，然后取其中部分，再丢弃其余；可以使用limit 限定结果
- 多表关联时返回全部列：select * from a join b where ……，可以改成select a.* from a join b where ……
- 总是取出全部列：select * 造成无法使用到覆盖索引，也额外带来IO、内存、cpu消耗。谨慎考虑是否应该用select * 
- 查询相同的数据：缓存相同的数据，查询的时候从缓存中取出就好
#### MySQL是否在扫描额外的记录
最简单衡量查询开销的指标：
1. 响应时间：包括服务时间和等待时间
2. 扫描行数
3. 返回行数：理想情况下扫描行数和返回行数应该相同

- explain的type列是访问类型：全表扫描、范围扫描、唯一索引查询、常数引用，速度从慢到快，扫描行数从大到小
- 扫描表、扫描索引、范围方位、单值访问
MySQL使用三种方式应用where条件，从好到坏依次为：
1. 索引中使用where过滤（存储引擎层完成）
2. 使用索引覆盖扫描（extra列中出现using index）返回记录（服务器层完成，无需回表）
3. 从数据表中返回记录，然后过滤不满足条件的记录（extra中出现using where）

如果扫描大量的数据但只返回少数行，可以如下优化：
1. 使用覆盖索引扫描，把所有需要用的列放索引中
2. 该表库表结构，比如使用单独的汇总表
3. 重写查询，让MySQL可以更好执行
## 重构查询的方式
#### 一个复杂查询还是多个简单查询
- 传统实现中，总强调数据库层完成尽可能多的工作，基于网络通信、查询解析和优化代价高
- MySQL连接和断开连接很轻量级，小查询很高校
- 网络速度快，带宽大延迟小，千兆网卡可轻松2000次以上查询
#### 切分查询
- 对大查询可以分而治之：删除旧数据可以采用多次小事务
#### 分解关联查询
- 可以将关联查询分解成若干个单表查询
1. 缓存效率高：许多应用程序可以方便地缓存单表查询对应的结果对象
2. 查询分解成单个查询减少锁竞争
3. 应用层做关联可以更容易对数据库拆分从而高性能和可扩展
4. 查询本身效率也可能会提升
5. 可以减少冗余记录的查询。数据库中关联查询可能会重复访问一部分数据，因此重构后可能减少网络和内存的消耗
6. 可以哈希关联而非MySQL的嵌套循环关联，提高效率
## 查询执行的基础
- 查询执行过程
![查询执行过程](C307B3037C8242EAA1E2E195AA9B2901)
1. 客户端发送一条查询给服务器
2. 服务器先检查查询缓存,如果命中了缓存,则立刻返回存储在缓存中的结果，否则进入下一阶段
3. 服务器端进行SQL解析、预处理,再由优化器生成对应的执行计划.
4. MySQL 根据优化器生成的执行计划,调用存储引擎的API来执行查询.
5. 将结果返回给客户端.
#### MySQL客户端/服务器通信协议
- MySQL通信是半双工，无法进行流量控制
- max_allowed_packet 参数很重要
- show processlist(show full processlist)
```
> show full processlist;
+----+------+-----------+------+---------+------+-------+-----------------------+----------+
| Id | User | Host      | db   | Command | Time | State | Info                  | Progress |
+----+------+-----------+------+---------+------+-------+-----------------------+----------+
| 21 | root | localhost | test | Query   |    0 | NULL  | show full processlist |    0.000 |
+----+------+-----------+------+---------+------+-------+-----------------------+----------+
1 row in set (0.00 sec)


```
[本段来自CSDN](https://blog.csdn.net/weixin_38756990/article/details/72870333)

各列的含义和用途，

列名|说明
---|---
id| 一个标识 
user| 显示当前用户，如果不是root，这 个命令就只显示你权限范围内的sql语句。 
host|显示这个语句是从哪个ip的哪个端口上发出的 
db|显示 这个进程目前连接的数据库。 
command|显示当前连接的执行的命令，一般就是休眠（sleep），<p>查询（query），连接 （connect）。 
time|此这个状态持续的时间，单位是秒。 
state|显示使用当前连接的sql语句的状态，只是语句执行中的某一个状态，<p>一个sql语句，查询为例，可能需要经过copying to tmp table，<p>Sorting result，Sending data等状态才可以完成 
info|显示这个sql语句，因为长度有限，所以长的sql语句就显示不全

这个命令中最关键的就是state列，mysql列出的状态主要有以下几种：

状态名|说明
---|---
Checking table |正在检查数据表（这是自动的）。 
Closing tables |正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。 
Connect Out |复制从服务器正在连接主服务器。 
Copying to tmp table on disk |由于临时结果集大于 tmp_table_size，正在将临时表从内存存储转为磁盘存储以此节省内存。 
Creating tmp table |正在创建临时表以存放部分查询结果。 
deleting from main table |服务器正在执行多表删除中的第一部分，刚删除第一个表。 
deleting from reference tables |服务器正在执行多表删除中的第二部分，正在删除其他表的记录。 
Flushing tables |正在执行 FLUSH TABLES，等待其他线程关闭数据表。 
Killed |发送了一个kill请求给某线程，那么这个线程将会检查kill标志位，同时会放弃下一个kill请求。MySQL会在每次的主循环中检查kill标志位，不过有些情况下该线程可能会过一小段才能死掉。如果该线程程被其他线程锁住了，那么kill请求会在锁释放时马上生效。 
Locked |被其他查询锁住了。 
Sending data |正在处理 SELECT 查询的记录，同时正在把结果发送给客户端。 
Sorting for group |正在为 GROUP BY 做排序。 
Sorting for order |正在为 ORDER BY 做排序。 
Opening tables |这个过程应该会很快，除非受到其他因素的干扰。例如，在执 ALTER TABLE 或 LOCK TABLE 语句行完以前，数据表无法被其他线程打开。 正尝试打开一个表。 
Removing duplicates |正在执行一个 SELECT DISTINCT方式的查询，但是MySQL无法在前一个阶段优化掉那些重复的记录。因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。 
Reopen table |获得了对一个表的锁，但是必须在表结构修改之后才能获得这个锁。已经释放锁，关闭数据表，正尝试重新打开数据表。 
Repair by sorting |修复指令正在排序以创建索引。 
Repair with keycache |修复指令正在利用索引缓存一个一个地创建新索引。它会比 Repair by sorting慢些。
Searching rows for update |正在讲符合条件的记录找出来以备更新。它必须在UPDATE要修改相关的记录之前就完成了。 
Sleeping |正在等待客户端发送新请求. 
System lock |正在等待取得一个外部的系统锁。如果当前没有运行多个mysqld服务器同时请求同一个表，那么可以通过增加–skip-external-locking参数来禁止外部系统锁。 
Upgrading lock INSERT DELAYED |正在尝试取得一个锁表以插入新记录。 
Updating |正在搜索匹配的记录，并且修改它们。 
User Lock |正在等待 GET_LOCK()。 
Waiting for tables |该线程得到通知，数据表结构已经被修改了，需要重新打开数据表以取得新的结构。然后，为了能的重新打开数据表，必须等到所有其他线程关闭这个表。以下几种情况下会产生这个通知：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, 或 OPTIMIZE TABLE。 
waiting for handler insert |INSERT DELAYED 已经处理完了所有待处理的插入操作，正在等待新的请求。大部分状态对应很快的操作，只要有一个线程保持同一个状态好几秒钟，那么可能是有问题发生了，需要检查一下。

#### 查询缓存
- 解析查询语句之前，如果查询缓存是打开的，MySQL会优先检查这个查询是否命中
- 如果命中，返回结果之前MySQL会检查用户权限，如果通过则直接返回结果给客户端
#### 查询优化处理
###### 语法解析和预处理
1. MySQL通过关键字将SQL语句解析，并生成一棵对应的解析树。解析器使用MySQL语法规则验证和解析查询（验证是否使用错误的关键字，关键字顺序是否争取额，引号是否正确匹配等）
2. 预处理器根据规则进一步检查解析树是否合法（数据表和数据列是否存在，解析名字和别名是否存在歧义）
3. 验证权限
###### 查询优化器
- 优化器转化语法树为执行计划，==MySQL是基于代价的优化器==

```
> select sql_no_cache count(*) from item;
+----------+
| count(*) |
+----------+
|   100000 |
+----------+
1 row in set, 1 warning (0.00 sec)

> show status like "last_query_cost";
+-----------------+--------------+
| Variable_name   | Value        |
+-----------------+--------------+
| Last_query_cost | 10695.262136 |
+-----------------+--------------+
1 row in set (0.01 sec)
```
- 多种原因导致MySQL优化器选择错误的执行计划
1. 统计信息不准确。MySQL依赖于存储引擎提供的统计信息估计成本，比如innodb不维护一个数据表的行数的精确统计信息
2. 执行计划中的成本估算不等同于实际执行的成本
3. MySQL的判断最优的标准不是你想的最优
4. MySQL没有考虑其他并发的查询，而并发可能影响执行速度
5. MySQL有时也会基于规则优化。比如存在全文索引match()子句，则使用全文索引，即使别人的索引和where条件可能比这快
6. MySQL不会考虑不受其控制的操作的成本，比如存储过程或者用户自定义函数的成本
7. 优化器有时候无法估算所有有可能的执行计划，所以有可能错过最优的执行计划
- MySQL优化分为静态优化和动态优化
1. 静态优化：直接对解析树分析，完成优化（比如简单的代数变换将where条件转换成另一种等价形式）；第一次完成后一直有效
2. 动态优化：与查询的上下文相关，与很多因素相关（where条件中的取值，索引中条目对应的数据行数）
- MySQL能处理的优化类型
1. 重新定义关联表的顺序
2. 将外连接转化成内连接
3. 使用等价变换规则：合并和减少一些比较，移除一些恒成立和恒不成立的判断
4. 优化count()，min()，max()：索引、列是否为空可以帮助MySQL优化这类表达式。

```
最小值，只需要查询对应B-tree索引最左端的记录而直接获取，因此优化器会把这个表达式当常数
> explain select min(i_id) from item;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)

Select tables optimized away  表示优化器已经从执行计划中移除了该表
```
5. 预估并转化为常数表达式：当检测到一个表达式可以转化为常数的时候，就一直会把该表达式当常数优化处理
6. 覆盖索引扫描
7. 子查询优化
8. 提前终止查询：

```
mariadb5.5
> explain select * from item where i_id =0;
+------+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                                               |
+------+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
|    1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE noticed after reading const tables |
+------+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+


mysql8
> explain select * from item where i_id =0;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | no matching row in const table |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
1 row in set, 1 warning (0.00 sec)

```
9. 等值传播：如果两个列的值通过等式关联，那么可以把一个列的where条件传递到另外一个列上

10. 列表in()的比较：在很多数据库中in()完全等同于or条件的子句，O(n)；MySQL将in()数据先进行排序，然后二分查找确定列表中是否满足条件，O(logn)
- ==不要以为比优化器更聪明，让它按照自己的方式工作==
#### 查询执行引擎
- 查询引擎根据执行计划完成整个查询，查询计划是一个数据结构而不是字节码
- 存储引擎实现的接口，即“handle api”
- 底层接口只有几十个，搭积木一样完成下旬大部分操作（比如查询索引第一行的接口，查询索引下一个条目等）
- 存储引擎接口使得插件式架构成为可能，但也给优化器带来限制
#### 返回结果给客户端
- 查询执行最后阶段是返回结果给客户端。即使不需要返回结果集给客户端，仍然会返回这个查询的一些信息，比如查询影响到的行数
- 如果可以，会缓存查询结果
## MySQL查询优化器的局限性
#### 关联子查询（MySQL5.5之前都存在）
- MySQL关联子查询实现得很糟糕，最糟糕的是where条件中包含in()：MySQL会把相关联的外层表压到里层（造成全表扫描）
- 改写方式：
1. 使用join重写
2. 使用group_concat()在in()中构造一个逗号分隔的列表
- 子查询性能不一定很差，要根据实际情况测试并确认具体的性能表现
#### union的限制
MySQL无法将限制条件从外层下推到内层，导致限制条件无法应用在内层的查询优化上

```
(SELECT first_ name, last_ name FROM sakila. actor ORDER BY last name) UNION ALL(SELECT first_ name, last_ name FROM sakila. customer ORDER BY last name)LIMIT 20;
```
可以改写为

```
(SELECT first_ name, last_ name FROM sakila. actor ORDER BY last name LIMIT 20) UNION ALL(SELECT first_ name, last_ name FROM sakila. customer ORDER BY last name LIMIT 20)LIMIT 20;
```
如果想得到正确的顺序，可以加order by
#### 索引的合并优化
MySQL5.0及以上的版本中，where子句包含多个复杂条件时，MySQL能够访问单个表的多个索引以合并和交叉过滤的方式定位行
#### 等值传递
如果存在in()列表，MySQL会将in()作为筛选条件复制到关联的各个表中，如果in()表非常大，会导致优化和执行都变慢
#### 并行执行
==MySQL无法啊利用多核特性并行执行查询==
#### 哈希关联
- MySQL所有的关联都是嵌套循环关联，不支持哈希关联
- mariadb已经实现真正的哈希关联
#### 松散索引扫描
MySQL暂时不支持松散索引扫描（Oracle的skip index scan）
```
//有索引(a,b)
select .... from tbl where b between 2 and 3;
因为前导列是a，查询中只指定了字段b，所以只能全表扫描
松散索引是：先扫描a列第一个值对应的b范围，然后扫描第二个值对应的b范围……这样可以减少扫描次数
```
- 松散索引扫描的限制会通过索引条件下推（index condition pushdown）解决
#### 最大值和最小值优化

```
create table min_test(a int primary key  ,b char(10));
select min(a) from min_test use index(primary) where b = "test36"; //b上无索引，所以会全表扫描
> explain  select a from min_test where b = "test1";
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | min_test | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1536 |    10.00 | Using where |
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
select a from min_test use index(primary) where b = "test36" limit 1;  //书上说，可以减少扫描行数（但实际测试并未如此）
> explain  select a from min_test use index(primary) where b = "test1" limit 1;
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | min_test | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1536 |    10.00 | Using where |
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

#### 在同一个表上查询和更新
MySQL不允许在同一个表上同时查询和更新

```
UPDATE tbl AS outer_tbl SET cnt=(SELECT count(*) FROM tbl AS inner_tbl WHERE inner_tbl.type = outer_tbl. type);
```
可以改写成生成表

```
UPDATE tbl INNER JOIN(SELECT type, count(*) AS cnt FROM tbl GROUP BY type
) AS der USING(type) SET tbl.cnt = der .cnt;
```

## 查询优化器的提示（hint）
- HIGH_ PRIORITY 和LOW_ PRIORITY
1. 这个提示告诉MySQL,当多个语句同时访问某一个表的时候,哪些语句的优先级相对高些、哪些语句的优先级相对低些
2. HIGH_PRIORITY 用于SELECT语句的时候,MySQL会将此SELECT语句重新调度到所有正在等待表锁以便修改数据的语句之前.实际上MySQL是将其放在表的队列的最前面,而不是按照常规顺序等待.HIGH_PRIORITY还可以用于INSERT语句,其效果只是简单地抵消了全局LOW_PRIORITY设置对该语句的影响.

```
SELECT HIGH_PRIORITY * FROM TABLE1;
```

3. LOW_PRIORITY 则正好相反:它会让该语句一直处于等待状态,只要队列中还有需要访问同一个表的语句一即使是那些比该语句还晚提交到服务器的语句.LOW_PRIORITY提示在SELECT、INSERT. UPDATE 和DELETE语句中都可以使用

```
update LOW_PRIORITY table1 set field1= where field1= …
```

4. 这两个提示只对使用表锁的存储引擎有效,千万不要在InnoDB或者其他有细粒度
锁机制和并发控制的引擎中使用.即使是在MyISAM中使用也要注意,因为这两个
提示会导致并发插入被禁用,可能会严重降低性能.

- DELAYED
这个提示对INSERT和REPLACE有效.MySQL会将使用该提示的语句立即返回给客户端,并将插入的行数据放入到缓冲区,然后在表空闲时批量将数据写入。日志系统使用这样的提示非常有效,或者是其他需要写入大量数据但是客户端却不需要等待单条语句完成I/O的应用。这个用法有一些限制:并不是所有的存储引擎都支持这样的做法;并且该提示会导致函数LAST_INSERT_ID()无法正常工作.

```
INSERT DELAYED INTO table1 set field1= …
```
- 强制连接顺序 STRAIGHT_JOIN
通过STRAIGHT_JOIN强迫MySQL按TABLE1、TABLE2的顺序连接表。如果你认为按自己的顺序比MySQL推荐的顺序进行连接的效率高的话，就可以通过STRAIGHT_JOIN来确定连接顺序。
```
SELECT TABLE1.FIELD1, TABLE2.FIELD2 FROM TABLE1 STRAIGHT_JOIN TABLE2 WHERE …
```
- SQL_BIG_RESULT和SQL_SMALL_RESULT
一般用于分组或DISTINCT关键字，这个选项通知MySQL，如果有必要，就将查询结果放到临时表中，甚至在临时表中进行排序。SQL_SMALL_RESULT比起SQL_BIG_RESULT差不多，告诉优化器结果会很小，倾向于放内存中。这两参数都很少使用。
```
SELECT SQL_BUFFER_RESULT FIELD1, COUNT(*) FROM TABLE1 GROUP BY FIELD1;
```
-  SQL_NO_CACHE和 SQL_CACHE

有一些SQL语句需要实时地查询数据，或者并不经常使用（可能一天就执行一两次）,这样就需要把缓冲关了,不管这条SQL语句是否被执行过，服务器都不会在缓冲区中查找，每次都会执行它
```
SELECT SQL_NO_CACHE field1, field2 FROM TABLE1;
```
强制查询缓冲 
如果在my.ini中的query_cache_type设成2，这样只有在使用了SQL_CACHE后，才使用查询缓冲。

```
SELECT SQL_CALHE * FROM TABLE1;
```
- SQL_CALC_FOUND_ROWS
并非优化器提示。查询中加上该提示MySQL会具区limit子句后这个查询要返回的结果集总数

```
select SQL_CALC_FOUND_ROWS * from min_test limit 1;
+---+-------+
| a | b     |
+---+-------+
| 1 | test1 |
+---+-------+
1 row in set (0.00 sec)

> SELECT FOUND_ROWS();
+--------------+
| FOUND_ROWS() |
+--------------+
|         1536 |
+--------------+
1 row in set (0.00 sec)
```
- 强制索引 FORCE INDEX 

```
SELECT * FROM TABLE1 FORCE INDEX(FIELD1) …
```

以上的SQL语句只使用建立在FIELD1上的索引，而不使用其它字段上的索引。
- 忽略索引 IGNORE INDEX 

```
SELECT * FROM TABLE1 IGNORE INDEX (FIELD1, FIELD2) …
```

在上面的SQL语句中，TABLE1表中FIELD1和FIELD2上的索引不被使用。 

- optimizer_search_depth

这个参数控制优化器在穷举执行计划时的限度.如果查询长时间处于"Statistics"状态,那么可以考虑调低此参数.
- optimizer_prune_level

该参数默认是打开的,这让优化器会根据需要扫描的行数来决定是否跳过某些执行计划.
- optimizer_switch

这个变量包含了一些开启/关闭优化器特性的标志位.例如在MySQL 5.1中可以通过这个参数来控制禁用索引合并的特性.
## 优化特定类型的查询
#### 优化count()查询
- count有两种作用
1. 统计某列的数量（要求列值非空，否则不统计）
2. 统计行数（不管列值空不空）
- 关于Myisam：Myisam没有where条件的count会比较快，因为Myisam存储了总行数，不需要全表扫描得到这个值
- 如果需要查询某个条件的统计数目，如果该条件的反面数量比较少，可以利用总数减去反条件得到条件总数（Myisam统计存在很快）
- 使用近似值：explain可以得到近似值
- 快速，精确，实现简单，三者不可同时兼得
#### 优化关联查询
- 确保on或者using子句上有索引：==只需要在关联顺序中第二个表上相应列建索引（？？==）
- 确保group by 和oy表达式只涉及一个表中的列，这样才有可能使用索引来优化
- 升级MySQL时候需要注意：关联语法，运算符优先级等其他可能发生变化的地方
#### 优化子查询
如果使用的是MySQL5.6或更新版本或mriadb，只查询的建议
#### 优化group by 和distinct
- 最有效的优化方法：使用索引
- 无法使用索引的时候，group by使用临时表或者filesort，同时可以使用 SQL_SMALL_RESULT 或者 SQL_BIG_RESULT 控制SQL的行为
- 关联查询做group by，那么group by后最好使用标识列

```
SELECT actor .first_name, actor.last_name, COUNT(*) FROM sakila.film_actor INNER JOIN sakila.actor USING(actor_id) GROUP BY actor.first_name, actor.last_name;
可改写成
SELECT actor .first_name, actor.last_name, COUNT(*) FROM sakila.film_actor INNER JOIN sakila.actor USING(actor_id) GROUP BY film_actor.actor_id;
```
#### 优化group by with rollup
最好将rollup放应用程序中处理
#### 优化limit分页
- limit m,n，当m偏移量很大的时候，MySQL需要查询m+n条记录并丢弃m，代价很高
- 优化办法：
1. 尽量使用覆盖索引
2. 转化为已知位置的查询
#### 优化SQL_CALC_FOUND_ROWS
- 加上这个hint之后，MySQL扫描全部满足条件的行，才能知道行数，代价可能会非常高
- 先获取并核内存较多的数据，然后每次分页都从缓存中获取
- 考虑使用explain的rows近似得到结果集的近似值；需要精确结果的时候再count(*)
#### 优化union查询
- MySQL总是通过创建并填充临时表的方式执行union，手工地将where、limit、order by等子句下推到union各个子查询中

```
> explain select * from min_test where b="test3" union select * from min_test where b="test30";
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type  | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | PRIMARY      | min_test   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1537 |    10.00 | Using where     |
|  2 | UNION        | min_test   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1537 |    10.00 | Using where     |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
3 rows in set, 1 warning (0.00 sec)
```
- 除非是确实需要服务器消除重复行，否则有要使用union all，因为MySQL会给临时表加distinct选项
#### 静态查询分析
percona toolkit的 ==pt-query-advisor（已经被去掉了）== 能够解析查询日志、分析存模式，然后给出所有可能潜在问题的查询，并给出详细建议
#### 使用用户自定义变量

- 以下情况不能使用自定义变量
1. 使用自定义变量的查询，无法使用查询缓存
2. 不能在使用常量或者标识符的地方使用自定义变量（表名，列名，limit子句）
3. 如果使用连接池或者持久化连接，自定义变量可能让看起来毫无关系的代码发生交互（主要是因为bug）
4. 5.0版本之前，大小写敏感，所以要注意版本兼容性
5. 不能显式地声明自定义变量类型
6. MySQL优化器某些场景会将变量优化掉
7. 赋值顺序和时间点并不固定，依赖于优化器的决定
8. 赋值符号:=优先级很低，所以赋值表达式应该明确使用括号
9. 使用未定义变量不产生任何语法错误

- 避免重复查询刚更新的数据 

```
updatet t1 set lastupdated = now() where id = 1;
selct lastupdated from t1 where id = 1;

可以 改成：
update t1 set lastupdated = now() where id = 1 and @now := now();
select @now()
// 此处now()非常快
```
- 统计更新和插入数量

```
INSERT INTO t1(C1, c2) VALUES(4, 4), (2, 1), (3, 1)
ON DUPLICATE KEY UPDATE
C1 = VALUES(c1) + (0*(@x :=@x+1) );
```
可以获得因冲突而更新的数据

- 确定取值顺序

```
SET @rownum := 0;
SELECT actor_id, @rownum := @rownum + 1 AS cnt FROM sakila.actor WHERE @rownum <= 1;
+---------+-----+
|actor_id | cnt |
+---------+-----+
|        1|    1|
|        2|    2|
+---------+-----+

SET @rownum := 0;
SELECT actor_id, @rownum AS rownum FROM sakila.actor wHERE (@rownum := @rownum + 1) <= 1;
+---------+--------+
|actor_id | rownum |
+---------+--------+
|        1|       1|
+---------+--------+


SET @rownum := 0;
SELECT actor_id, first_name, @rownum AS rownum FROM sakila.actor WHERE @rownum<= 1 ORDER BY first_name, LEAST(0, @rownum : @rownum + 1);
```

## 案例学习
#### 使用MySQL构建一个对列表
#### 计算两点之间的距离
#### 使用用户自定义函数