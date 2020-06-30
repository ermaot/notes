![image](pic/%E9%94%81%E4%B8%8Elatch.png)

## innodb锁分类
按照颗粒度划分，有行锁与表锁
#### 行锁
- 1.共享锁与排他锁（S锁与X锁）
- 2.记录锁
- 3.间隙锁
- 4.next-key lock
- 5.插入意向锁（insert intension lock）
#### 表锁
- 意向锁（intention lock）
- 自增锁（auto-inc lock）

## 行锁
- 通常情况下，普通查询属于非锁定读，不添加任何锁（一致性读）
- 锁定读
1. select……for share（mysql8.0新方式，之前是select …… lock in share mode），添加S锁，其他事务可以读但写被阻塞
2. select…… for update添加X锁，其他事务修改或者select…… for share都会被阻塞
- mysql记录锁都是加在索引上的（如果没有显式索引，会加在默认创建的隐式索引上）
- 间隙锁锁定范围是索引记录之间的间隙，防止不可重复读
- next-key lock防止幻读
- 插入意向锁：插入具有相同索引间隙的多个事务，如果插入值不同则不需要相互等待；插入意向锁之间不冲突，但可能和其他锁冲突（如next-key lock）

## 表锁
- 意向锁：mysql中是表级别的锁
1. select……for share，添加意向共享锁（IS）
2. select……for update，添加意向排他锁（IX）
- 在获取某行的共享锁之前，必须获得 表的IS锁；获取某行的独占锁之前，必须获取表的IX锁
- 意向锁不会阻止除了表锁（lock table……）请求之外的锁：即执行lock table时表记录不能存在锁
- 自增锁：自增锁是插入到具有auto_increment字段表中的事务所采用的特殊表锁。在最简单情况下，如果一个事务正在向表中插入值，其他任何事务都必须等待插入语句完成以保证插入值连续

## 锁模式对应的含义
show engine innodb status查看所信息时，lock_mode 字段含义
模式|含义
---|---
IX|意向排他锁
X:|nexy-key lock（X）
S:|nexy-key lock（S）
X|锁记录本身（X）
S|锁记录本身（S）
X,GAP :|间隙锁
S:GAP |间隙锁
X,GAP,INSERT_INTENTION:|插入意向锁                           

## 加锁原理
#### 1.RR级别 + 无显式主键和索引
````
> create table t(id int,name char(20));
> insert into t values(10,"t1"),(20,"t2"),(30,"t3");
````
可以通过show engine innodb status查看锁或者通过performance_schema.data_locks查看锁
```
> begin;
Query OK, 0 rows affected (0.00 sec)
> select * from t for update;
+------+------+
| id   | name |
+------+------+
|   10 | t1   |
|   20 | t2   |
|   30 | t3   |
+------+------+
3 rows in set (0.00 sec)
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from performance_schema.data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+-----------+-------------+------------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME      | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA              |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+-----------+-------------+------------------------+
| 139789757328504:1165:139789642828344   |                  5580 |        67 | test          | t           | NULL            | TABLE     | IX        | GRANTED     | NULL                   |
| 139789757328504:10:4:1:139789642825336 |                  5580 |        67 | test          | t           | GEN_CLUST_INDEX | RECORD    | X         | GRANTED     | supremum pseudo-record |
| 139789757328504:10:4:2:139789642825336 |                  5580 |        67 | test          | t           | GEN_CLUST_INDEX | RECORD    | X         | GRANTED     | 0x000000002303         |
| 139789757328504:10:4:3:139789642825336 |                  5580 |        67 | test          | t           | GEN_CLUST_INDEX | RECORD    | X         | GRANTED     | 0x000000002304         |
| 139789757328504:10:4:4:139789642825336 |                  5580 |        67 | test          | t           | GEN_CLUST_INDEX | RECORD    | X         | GRANTED     | 0x000000002305         |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+-----------+-------------+------------------------+
5 rows in set (0.01 sec)

```
加锁顺序：
1. 对表加IX锁
2. 在supremum加next-key lock
3. 在记录上分别加nexy-key lock锁

如果加where条件，加锁方式不变，因为要防止发生幻读（若有其他会话执行delete或者update都会幻读）

#### 2.RR级别+表有显式主键无索引
```
> create table lock2(id int primary key,name char(20) default null);
> insert into lock2 values(10,"t1"),(20,"t2"),(30,"t3");
```
- 1.不带where条件
可以看到与无索引的情况一样，加IX表锁并对所有记录和supremum记录加锁
```
> begin;
Query OK, 0 rows affected (0.00 sec)
> select * from lock2 for update;
+------+------+
| id   | name |
+------+------+
|   10 | t1   |
|   20 | t2   |
|   30 | t3   |
+------+------+
3 rows in set (0.00 sec)
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+-----------+-------------+------------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA              |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+-----------+-------------+------------------------+
| 139789757328504:1166:139789642828344   |                  5594 |        69 | test          | lock2       | NULL       | TABLE     | IX        | GRANTED     | NULL                   |
| 139789757328504:11:4:1:139789642825336 |                  5594 |        69 | test          | lock2       | PRIMARY    | RECORD    | X         | GRANTED     | supremum pseudo-record |
| 139789757328504:11:4:2:139789642825336 |                  5594 |        69 | test          | lock2       | PRIMARY    | RECORD    | X         | GRANTED     | 10                     |
| 139789757328504:11:4:3:139789642825336 |                  5594 |        69 | test          | lock2       | PRIMARY    | RECORD    | X         | GRANTED     | 20                     |
| 139789757328504:11:4:4:139789642825336 |                  5594 |        69 | test          | lock2       | PRIMARY    | RECORD    | X         | GRANTED     | 30                     |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+-----------+-------------+------------------------+
5 rows in set (0.00 sec)
```
- 2.带where且where是主键
对表加IX锁，只对记录加锁
```
> begin;
Query OK, 0 rows affected (0.00 sec)
> select * from lock2 where id=10 for update;
+------+------+
| id   | name |
+------+------+
|   10 | t1   |
+------+------+
3 rows in set (0.00 sec)
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
| 139789757328504:1166:139789642828344   |                  5595 |        69 | test          | lock2       | NULL       | TABLE     | IX            | GRANTED     | NULL      |
| 139789757328504:11:4:2:139789642825336 |                  5595 |        69 | test          | lock2       | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 10        |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
2 rows in set (0.00 sec)
```
- 3.where条件包含主键字段和非主键字段
对表加IX锁，只对记录加锁，与2中情况一致
```
> begin;
Query OK, 0 rows affected (0.00 sec)
> select * from lock2 where id=10 and name="t1" for update;
+------+------+
| id   | name |
+------+------+
|   10 | t1   |
+------+------+
3 rows in set (0.00 sec)
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
| 139789757328504:1166:139789642828344   |                  5596 |        69 | test          | lock2       | NULL       | TABLE     | IX            | GRANTED     | NULL      |
| 139789757328504:11:4:2:139789642825336 |                  5596 |        69 | test          | lock2       | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 10        |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
2 rows in set (0.00 sec)
```

#### 3.RR级别+表无显式主键但有索引
```
> create table lock3(id int ,name char(20) default null);
> create index lock3_inx_id on lock3(id);
> insert into lock2 values(10,"t1"),(20,"t2"),(30,"t3");
```
- 1.不带where
可以看到与无索引的情况一样，加IX表锁并对所有记录和supremum记录加锁
- 2.带where
###### 普通索引
**where条件中，只要包含索引字段，如下加锁**

1. 对表加IX锁
2. 对id=10的记录加nexy-key锁，区间是(-无穷大,10]
3. 对索引加X锁
4. 对索引区间(10,20)加gap锁以防止幻读
```
> begin;
Query OK, 0 rows affected (0.01 sec)
> select * from lock3 where  id=10  for update;
+------+------+
| id   | name |
+------+------+
|   10 | t1   |
+------+------+
1 row in set (0.00 sec)
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+---------------+-------------+--------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME      | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA          |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+---------------+-------------+--------------------+
| 139789757328504:1167:139789642828344   |                  5619 |        69 | test          | lock3       | NULL            | TABLE     | IX            | GRANTED     | NULL               |
| 139789757328504:12:5:2:139789642825336 |                  5619 |        69 | test          | lock3       | lock3_inx_id    | RECORD    | X             | GRANTED     | 10, 0x000000002306 |
| 139789757328504:12:4:2:139789642825688 |                  5619 |        69 | test          | lock3       | GEN_CLUST_INDEX | RECORD    | X,REC_NOT_GAP | GRANTED     | 0x000000002306     |
| 139789757328504:12:5:3:139789642826040 |                  5619 |        69 | test          | lock3       | lock3_inx_id    | RECORD    | X,GAP         | GRANTED     | 20, 0x000000002307 |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+---------------+-------------+--------------------+
4 rows in set (0.00 sec)
```
如果插入9~19的记录会被阻塞（因为需要加间隙锁），而20不会
```
> insert into lock3 values(20,"t4");
Query OK, 1 row affected (0.01 sec)
> insert into lock3 values(19,"t4");
…………阻塞中
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+------------------------+-------------+--------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME      | LOCK_TYPE | LOCK_MODE              | LOCK_STATUS | LOCK_DATA          |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+------------------------+-------------+--------------------+
| 139789757330600:1167:139789642840680   |                  5622 |        71 | test          | lock3       | NULL            | TABLE     | IX                     | GRANTED     | NULL               |
| 139789757330600:12:5:3:139789642837736 |                  5622 |        71 | test          | lock3       | lock3_inx_id    | RECORD    | X,GAP,INSERT_INTENTION | WAITING     | 20, 0x000000002307 |
| 139789757328504:1167:139789642828344   |                  5619 |        69 | test          | lock3       | NULL            | TABLE     | IX                     | GRANTED     | NULL               |
| 139789757328504:12:5:2:139789642825336 |                  5619 |        69 | test          | lock3       | lock3_inx_id    | RECORD    | X                      | GRANTED     | 10, 0x000000002306 |
| 139789757328504:12:4:2:139789642825688 |                  5619 |        69 | test          | lock3       | GEN_CLUST_INDEX | RECORD    | X,REC_NOT_GAP          | GRANTED     | 0x000000002306     |
| 139789757328504:12:5:3:139789642826040 |                  5619 |        69 | test          | lock3       | lock3_inx_id    | RECORD    | X,GAP                  | GRANTED     | 20, 0x000000002307 |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+------------------------+-------------+--------------------+
6 rows in set (0.01 sec)
```

- 3.唯一索引
```
> create table lock4(id int,name char(20) default null);  
> create unique index lock4_inx_id on lock4(id); 
> insert into lock4 values(10,"t1"),(20,"t2"),(30,"t3");  
```
where条件是索引字段：加表IX锁，加记录锁，加GEN_CLUST_INDEX锁
```
> begin;
> select * from lock4  where id=10  for update;
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+---------------+-------------+--------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME      | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA          |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+---------------+-------------+--------------------+
| 139789757328504:1168:139789642828344   |                  5649 |        69 | test          | lock4       | NULL            | TABLE     | IX            | GRANTED     | NULL               |
| 139789757328504:13:5:2:139789642825336 |                  5649 |        69 | test          | lock4       | lock4_inx_id    | RECORD    | X,REC_NOT_GAP | GRANTED     | 10, 0x00000000230C |
| 139789757328504:13:4:2:139789642825688 |                  5649 |        69 | test          | lock4       | GEN_CLUST_INDEX | RECORD    | X,REC_NOT_GAP | GRANTED     | 0x00000000230C     |
+----------------------------------------+-----------------------+-----------+---------------+-------------+-----------------+-----------+---------------+-------------+--------------------+
3 rows in set (0.00 sec)
```
where条件中包含索引字段和非索引字段与上述相同

#### RR 级别+表有显示主键和索引
```
> create table rr_pri_t(id int not null primary key,name char(20) default null ,key idx_name(name));
> insert into rr_pri_t values(10,"t1"),(20,"t2"),(30,"t3");     
```
- 1. 表有显式主键和普通索引
**不带where条件**:
对表加IX锁；对supremum加nexy-key 锁；对索引加next-key锁；对主键全部记录加X锁
```
> begin;
> select * from rr_pri_t for update;
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+----------------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA                  |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+----------------------------+
| 139789757328504:1169:139789642828344   |                  5710 |        69 | test          | rr_pri_t    | NULL       | TABLE     | IX            | GRANTED     | NULL                       |
| 139789757328504:14:5:1:139789642825336 |                  5710 |        69 | test          | rr_pri_t    | idx_name   | RECORD    | X             | GRANTED     | supremum pseudo-record     |
| 139789757328504:14:5:2:139789642825336 |                  5710 |        69 | test          | rr_pri_t    | idx_name   | RECORD    | X             | GRANTED     | 't1                  ', 10 |
| 139789757328504:14:5:3:139789642825336 |                  5710 |        69 | test          | rr_pri_t    | idx_name   | RECORD    | X             | GRANTED     | 't2                  ', 20 |
| 139789757328504:14:5:4:139789642825336 |                  5710 |        69 | test          | rr_pri_t    | idx_name   | RECORD    | X             | GRANTED     | 't3                  ', 30 |
| 139789757328504:14:4:2:139789642825688 |                  5710 |        69 | test          | rr_pri_t    | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 10                         |
| 139789757328504:14:4:3:139789642825688 |                  5710 |        69 | test          | rr_pri_t    | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 20                         |
| 139789757328504:14:4:4:139789642825688 |                  5710 |        69 | test          | rr_pri_t    | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 30                         |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+----------------------------+
8 rows in set (0.01 sec)
```
**where条件是普通索引**
对表加IX锁；对记录加next-key锁；对记录加X锁；对记录加GAP锁
```
> begin;
> select * from rr_pri_t  where name="t1" for update;
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+----------------------------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA                  |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+----------------------------+
| 139789757328504:1169:139789642828344   |                  5711 |        69 | test          | rr_pri_t    | NULL       | TABLE     | IX            | GRANTED     | NULL                       |
| 139789757328504:14:5:2:139789642825336 |                  5711 |        69 | test          | rr_pri_t    | idx_name   | RECORD    | X             | GRANTED     | 't1                  ', 10 |
| 139789757328504:14:4:2:139789642825688 |                  5711 |        69 | test          | rr_pri_t    | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 10                         |
| 139789757328504:14:5:3:139789642826040 |                  5711 |        69 | test          | rr_pri_t    | idx_name   | RECORD    | X,GAP         | GRANTED     | 't2                  ', 20 |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+----------------------------+
4 rows in set (0.00 sec)
```
**where条件是主键**
```
> begin;
> select * from rr_pri_t  where id=10 for update;
> select ENGINE_LOCK_ID,ENGINE_TRANSACTION_ID,THREAD_ID,OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA from data_locks;
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
| ENGINE_LOCK_ID                         | ENGINE_TRANSACTION_ID | THREAD_ID | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
| 139789757328504:1169:139789642828344   |                  5712 |        69 | test          | rr_pri_t    | NULL       | TABLE     | IX            | GRANTED     | NULL      |
| 139789757328504:14:4:2:139789642825336 |                  5712 |        69 | test          | rr_pri_t    | PRIMARY    | RECORD    | X,REC_NOT_GAP | GRANTED     | 10        |
+----------------------------------------+-----------------------+-----------+---------------+-------------+------------+-----------+---------------+-------------+-----------+
2 rows in set (0.00 sec)
```
- 2. 表有显式主键和唯一索引
  情况与上面一致
  3. 