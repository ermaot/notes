## 什么是锁
- 锁机制用于对共享资源的并发访问
- 锁是数据库区别于文件系统的重要特性
- 有多少种数据库，就有多少种锁的实现方法
- MyISAM是表所，而innodb 支持（多粒度锁定）
- SQL SERVER 2005之前 是页锁，现在支持乐观并发和悲观并发
## innodb 引擎中的锁
共享锁（S） 和排他锁（X）

  锁类型   | X | S 
---|---|---
X | 冲突| 冲突
S | 冲突| 兼容

- show engine innodb status 查看锁信息

```
> show engine innodb status;
…………………………
------------
TRANSACTIONS
------------
Trx id counter 151B45
Purge done for trx's n:o < 151B3E undo n:o < 0
History list length 1561
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 0, not started
MySQL thread id 3353, OS thread handle 0x7fbd1c168700, query id 39258 localhost root
show engine innodb status
---TRANSACTION 151B44, ACTIVE 2 sec
mysql tables in use 1, locked 1
2 lock struct(s), heap size 376, 1 row lock(s)
MySQL thread id 3797, OS thread handle 0x7fbd1c11e700, query id 39257 localhost root User sleep
select 1 from a where sleep(100) lock in share mode
----------------------------
END OF INNODB MONITOR OUTPUT
============================

```

- show processlist 和 show engine innodb status 查看当前数据库请求（innodb plugin 之前）
- information_schema下的 innodb_trx、innodb_locks、innodb_lock_waits
innodb_trx 由21个字段组成（innodb内幕一书中说8个字段，属未更新错误）

列名 | 说明
---|---
trx_id | 唯一事务ID
trx_state | 当前事务状态<p>允许值RUNNING，LOCK WAIT， ROLLING BACK，和 COMMITTING
trx_started | 事务开始时间
trx_requested_lock_id | 等待事务的锁ID。<p>trx_state不是lock wait，则为null；否则则是当前事务占用的锁的ID
trx_wait_started | 事务等待的开始时间<p>trx_state不是lock wait，则为null
trx_weight | 事务的权重，反映的是修改和锁住的行数。<p>发生死锁需要回滚的时候，选择该值较小的回滚
trx_mysql_thread_id | 对应show processlist中的线程id
trx_query | 事务运行的sql，有时候会是null
trx_operation_state | 交易的当前操作，如果有的话; 否则 NULL
trx_tables_in_use | InnoDB处理此事务的当前SQL语句时使用 的表数
trx_tables_locked | InnoDB当前SQL语句具有行锁定 的表的数量<p>（行锁，所以仍可以通过多个事务读取和写入表，尽管某些行被锁定。）
trx_lock_structs | 事务保留的锁数
trx_lock_memory_bytes | 内存中此事务的锁结构占用的总大小
trx_rows_locked | 锁定的大致数字或行数。<p>该值可能包括实际存在但对事务不可见的删除标记行
trx_rows_modified | 此事务中已修改和插入的行数
trx_concurrency_tickets | 一个值，指示当前事务在被换出之前可以执行多少工作
trx_isolation_level | 当前事务的隔离级别
trx_unique_checks | 是否为当前事务打开或关闭唯一检查
trx_foreign_key_checks | 是否为当前事务打开或关闭唯一检查
trx_last_foreign_key_error | 最后一个外键错误的详细错误消息（如果有）; 否则NULL
trx_adaptive_hash_latched | 自适应哈希索引是否被当前事务锁定<p>当自适应哈希索引搜索系统被分区时，单个事务不会锁定整个自适应哈希索引。<p>自适应哈希索引分区由innodb_adaptive_hash_index_parts，默认设置为8
trx_adaptive_hash_timeout | 是否立即为自适应哈希索引放弃搜索锁存器，或者在MySQL的调用之间保留它。<p>当没有自适应哈希索引争用时，该值保持为零，语句保留锁存器直到它们完成。<p>在争用期间，它倒计时到零，并且语句在每次行查找后立即释放锁存器。<p>当自适应散列索引搜索系统被分区（受控制 innodb_adaptive_hash_index_parts）时，该值保持为0。



innodb_locks<p>
列名 | 说明
---|---
lock_id | 锁的ID
lock_trx_id | 事务ID
lock_mode | 锁的模式<p>允许锁定模式描述符 S，X， IS，IX， GAP，AUTO_INC，和 UNKNOWN
lock_type | 锁的类型（行锁还是表锁）
lock_table | 加锁的表
lock_index | 加锁的索引
lock_space | innodb 表空间的sapce 号
lock_page | 被锁住的页的数量。如果是表锁，为null
lock_rec | 被锁住的行的数量。如果是表锁，为null
lock_data | 被锁住的行的主键值。如果是表锁，为null<p>如果没有主键，LOCK_DATA则是唯一的InnoDB内部行ID号<p>若对键值或范围高于索引最大值的间隙锁定LOCK_DATA报告supremum pseudo-record


innodb_lock_waits<p>
列名 | 说明
---|---
requesting_trx_id | 申请锁资源的事务的ID
requested_lock_id | 申请的锁的ID
blocking_trx_id | 阻塞的事务的ID
blocking_lock_id | 阻塞的锁的ID


[本部分参考了该博客](https://blog.csdn.net/yuyinghua0302/article/details/82318408)
#### 一致性的非锁定行读
- 一致性的非锁定行读（consistant nonlocking read） 是指innodb通过多版本控制当前时间数据库中行的数据
- 如果读取的数据行正在update，delete操作，读操作不会等待行上锁的释放而读取一个快照数据
- 非锁定读，即不需要等待X锁的释放
- 快照数据是指该行之前的版本的数据，由undo段实现
- 非锁定读提高数据库的并发性，是innodb默认的读取方式
- 一个行可以有多个版本，叫多版本技术，带来的并发控制叫多版本并发控制mvcc（multi version concurrency control）
- Read COmmitted 和 Repeatable Read 下，innodb采用非锁定读
- Read COmmitted 级别下，总是读取最新的快照数据
- Repeatable 和 Repeatable Read 下，读取事务开始时的数据版本
## 锁的算法
innodb有3种锁的算法设计
锁类型 | 说明
---|---
Record lock | 行锁；如果无索引，innodb使用隐式索引来锁定
Gap Lock | 间隙锁，锁定一个范围但不包括本身
Next-key Lock | 行锁+间隙锁，锁定一个范围并包括记录自身<p>在Repeatable read级别下的默认行记录锁定算法




## 锁问题
#### 丢失更新（lost update）

序号 | 步骤
---|---
1 | 事务T1查询一行数据，传输显示给用户U1
2 | 事务T1查询同一行数据，传输显示给用户U2
3 | U1修改此记录，并提交
4 | U2修改此记录，并提交
结果 | ==U1修改丢失==
此情况下，查询需加排他锁，以保证事务串行

#### 脏读
- 脏读与脏页。脏读是读到没有提交的数据，脏页是缓冲池中被修改但尚未写入到磁盘的页
- 脏读并不常发生，发生条件是read uncommitted
- 一般数据库至少是read committed
- innodb是Repeatable read，sql server 和 Oracle是 read committed


#### 不可重复读
- 不可重复读是指在同一个事务内部多次读同一个数据，结果不一样
- 脏读是读到未提交的数据；不可重复读读到已提交的数据
- 不可重复读可接受
## 阻塞
- 有时候一个事务中的锁需要等待另一个事务锁住的资源释放，此时即发生阻塞
- innodb使用mutex数据结构来实现锁，访问资源前使用mutex_enter()函数申请，结束后使用mutex_exit()释放
- innodb使用innodb_lock_wait_timeout控制等待时间，默认为50，动态参数，可在实例中调整

```
> show variables like "%innodb_lock_wait_timeout%";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 50    |
+--------------------------+-------+

> set innodb_lock_wait_timeout=60;
Query OK, 0 rows affected (0.01 sec)
```

- innodb_rollback_on_timeout 控制超时后是否回滚，默认为OFF（是否回滚需要用户自己判断）

```
> show variables like "%innodb_rollback_on_timeout%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_rollback_on_timeout | OFF   |
+----------------------------+-------+

```

## 死锁
- 并发才可能死锁，串行则不会
- innodb并不会回滚大部分的错误异常，但死锁例外
- oracle产生死锁的常见原因是对外键没有添加索引；innodb 会自动为外键索引，且不可认为删除

```
> create table a(a int ,primary key(a));
> create table d(b int,foreign key(b) references a(a));
> show index from d;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| d     |          1 | b        |            1 | b           | A         |           0 |     NULL | NULL   | YES  | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+

```

## 锁升级
锁升级（lock escalation）是将当前锁的粒度降低