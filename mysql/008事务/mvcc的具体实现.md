## 隔离级别
#### 四种隔离级别
在SQL 标准中定义了四种隔离级别。较低的隔离级别通常可以执行更高的并发，系统的开销也是更低的。

级别|说明
---|---
READ UNCOMMITED 未提交读|事务中修改了数据，即便是未提交，也被其他事务可见，这样就造成了脏读。<p>脏读会带来很多问题，很少使用它。
READ COMMITED 提交读|这是大多数数据库的默认隔离级别，一个事务修改数据但未提交，<p>对其他任何事务都是不可见的。也就是不可重复读。
REPATABLE READ 可重复读|解决了脏读的问题，但是不能解决另外一个问题，幻读。<p>所谓幻读，就是当某个事务读取某个范围的记录，另一个事务又在该范围插入新纪录，<p>当之前的事务再次来读取该范围的数据时，发现还有新数据未被更新，<p>就像产生了幻觉一样，即产生幻行。<p>InnoDB 采取了（MVCC, Multiversion Concurrency Control）解决了幻读的问题。<p> ==Mysql 的默认事务隔离级别==。
SERIALIZABLE 可串行化|最高隔离界别。它通过强制事务串行化，避免了前面所说的幻读问题。<p>简单的说，就是SERIALIZABLE会在读取的每一行数据上都加锁，会导致大量的超时和锁竞争的问题。<p>实际上很少用它。


- 查看隔离级别：SELECT @@transaction_isolation
```
> show variables like "%isolation%";
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.00 sec)

```

- 设置mysql的隔离级别：set session transaction isolation level 设置事务隔离级别
```
> set session transaction isolation level  REPEATABLE READ;
Query OK, 0 rows affected (0.00 sec)

> set session transaction isolation level  read committed;
Query OK, 0 rows affected (0.00 sec)

> set session transaction isolation level  serializable;
Query OK, 0 rows affected (0.00 sec)

> set session transaction isolation level  read uncommitted;
Query OK, 0 rows affected (0.00 sec)


```
#### 不可重复读与幻读
特征|区别
---|---
表现|相似，都表现为两次读取的结果不一致
控制方法和锁|不可重复读只需要锁住满足条件的记录,幻读要锁住满足条件及其相近的记录
DML|可重复读重点在于update和delete，而幻读的重点在于insert

不可重复读和幻读最大的区别，就在于==如何通过锁机制来解决他们产生的问题==

## redo和undo
- redo log通常是物理日志（==这个需要商榷，物理逻辑日志吧？==），记录的是数据页的物理修改，它用来恢复提交后的物理数据页(有且只能恢复到最后一次提交的位置)
- undo用来回滚行记录到某个版本。undo log一般是逻辑日志，记录每行数据
- MVCC (Multiversion Concurrency Control)，即多版本并发控制技术,它使得大部分支持行锁的事务引擎，不再单纯的使用行锁来进行数据库的并发控制，取而代之的是把数据库的行锁与行的多个版本结合起来，只需要很小的开销,就可以实现==非锁定读==

## Innodb
- InnoDB是一个多版本存储引擎：它保存有关已更改行的旧版本的信息，以支持并发和回滚等事务功能。
- 此信息存储在表空间中称rollback segment的数据结构中。 
- InnoDB使用rollback segment中的信息来执行事务回滚中所需的撤消操作。
- 它还使用该信息构建行的早期版本以进行一致读取

#### Innodb机制
- 在内部，InnoDB为存储在数据库中的每一行添加三个字段。
1. 6字节的DB_TRX_ID字段指示插入或更新该行的最后一个事务的事务标识符。此外，删除在==内部被视为更新==，其中行中的special bit被设置为将其标记为已删除。
2. 每行还包含一个7字节的DB_ROLL_PTR字段，称为滚动指针。roll指针指向写入回滚段的undo log记录。如果更新了行，则undo log记录了能重建更新之前数据的所有必要信息。
3. 6字节的DB_ROW_ID字段包含在插入新行时单调增加的行ID。如果InnoDB自动生成聚簇索引，则索引包含行ID值。否则，DB_ROW_ID列不会出现在任何索引中

- 在InnoDB多版本控制方案中，当使用SQL语句删除行时，不会立即从数据库中物理删除该行。 InnoDB在丢弃为删除写入的更新undo log 记录时，仅物理删除相应的行及其索引记录。此删除操作称为清除，并且速度非常快，通常与执行删除的SQL语句的时间顺序相同。
- 如果你在表中以大约相同的速率插入和删除少量批次的行，则清除线程可能开始落后，并且由于有“死”行，表可以变得越来越大，使得所有磁盘都受到限制慢。在这种情况下，通过调整innodb_max_purge_lag系统变量来限制新行操作，并为清除线程分配更多资源
```
> show variables like "%innodb_max_purge_lag%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_max_purge_lag       | 0     |
| innodb_max_purge_lag_delay | 0     |
+----------------------------+-------+
2 rows in set (0.00 sec)

```

## 多版本控制和二级索引
- InnoDB多版本并发控制（MVCC）以不同于聚簇索引的方式处理二级索引。
1. 聚集索引中的记录就地更新，其隐藏的系统列指向可以重建早期版本记录的撤消日志条目。（==这个似乎不对，对于主键和非主键，处理应该是不一样的==）
2. 与聚簇索引记录不同，二级索引记录不包含隐藏的系统列，不会就地更新。
3. 更新二级索引列时，旧的二级索引记录将被删除标记，插入新记录，最终清除delete-marked的记录。
4. 当二级索引记录被delete-marked或二级索引页面由较新的事务更新时，InnoDB在聚簇索引中查找数据库记录。在聚簇索引中，将检查记录的DB_TRX_ID，如果在启动读取事务后修改了记录，则会从undo log中检索正确的记录版本

## 操作过程
- 对于insert
1. begin->用排他锁锁定该行->记录redo log->记录undo log->插入当前行的新值，写事务编号。若回滚，回滚时把insert undo log丢弃
2. InnoDB为每个新增行记录当前系统版本号（事务id）
- 对于update：
1. begin->用排他锁锁定该行->记录redo log->记录undo log->修改当前行的值，写事务编号。若回滚，回滚指针指向undo log中的修改前的行
2. InnoDB为插入一行新记录，保存当前系统版本号作为行版本号，同时保存当前系统版本到原来的行作为行删除标识（special bit）
- 对于delete
1. InnoDB为删除的每一行，保存当前系统版本号作为删除标识
2. 之后再通过purge的方式进行删除

## read view 判断当前版本数据项是否可见
- 为了实现事务隔离性数据库一般使用两种实现手段：分别是==当前读和快照读==
1. ==当前读就是加锁读==，事务对访问的数据进行加锁，事务提交时释放所持有的锁。实现简单，但并发性能差
2. 快照读具体做法是一份数据保存多个历史版本，不同事务访问不同历史版本的数据彼此之间互不影响。缺点是事务对数据只能读不能修改
- InnoDB的事务在修改一份数据时对其进行加锁读，而只读不改数据的时候进行快照读
- 快照读的具体实现，涉及到几个问题：1，历史版本如何保存；2，如何拿到正确版本的历史数据；3，历史数据如何回收
1. 一种保存全量的历史数据，另一种保存能够回滚到历史数据的undo日志。InnoDB采用的是后一种方式，事务更新数据同时产生一份对应的undo日志，并且日志中记录了事务id
2. 事务使用快照（readview）去读取数据正确的历史版本，快照中包含的信息是当前所有活跃事务的id。
3. InnoDB回收历史数据的任务由后台purge线程完成。具体做法是：purge线程从当前所有readview中找到创建时间最久的那个oldest readview，那些比oldest readview还要旧的undo日志就是可以被安全回收的。因此，InnoDB维护了一个全局的readview链表，链表中的readview按照创建时间排序，purge线程只须找到链表尾端的readview就是oldest readview

#### 5.7改进
- 事务进行一次快照读的步骤如下
1.  分配一个快照对象
2.  将快照对象加入到全局readview链表头部
3.  对当前活跃事务打快照
4.  进行快照读
5.  完成后，将快照对象从全局readview链表中移除
- rr隔离级别下每个事务使用一个快照，而rc隔离级别下每条query使用一个readview，所以快照的分配和释放与事务的开始和结束不完全对应
- 后台purge线程进行一次purge操作的过程
1. 从全局readview链表中找到oldest readview
2. 使用oldest readview去回收undo日志
- InnoDB事务系统的==全局大锁（trx_sys->mutex）==，保护了全局的readview链表，还保护了全局的活跃事务链表等对象，事务在begin和commit等过程中也要竞争这把大锁。很热
- 优化：如果上次快照读与这次快照读之间没有写事务发生，那么一定也没有任何数据被修改，所以理论上可以直接复用上次的readview。autocommit只读事务完成快照读后，并不会将快照从全局readview链表中删除，而只是将它设置成close状态。如果不能复用快照，依然需要操作全局readview链表，但是相对于改进前少了一次全局事务锁的争用
- 缺点：全局readview链表中多出了很多close状态的readview，链表长度变长，最直接的影响是purge线程在获取oldest readview的开销变大