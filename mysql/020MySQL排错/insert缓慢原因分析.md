本文的分析是以5.6 innodb 引擎为主
## insert几个可能的性能瓶颈点
基于insert源码分析，和MySQL事务的一般过程，可以看出insert语句执行过程中几个可能的瓶颈点，包括加锁，io和网络几个方面：
- MDL锁， insert语句需要拿IX MDL 锁
- 外键检查对主表加S行锁
- insert转update操作需要的对老记录index entry的行锁
- iops限制
- 写binlog
- semi-sync消息延迟

## 示例分析

#### MDL锁等待
- 一般都是因为表上有运行时间比较长的DDL语句在运行，比如Optimize， Truncate table，Alter table等
- 通过运行show processlist，就会看到被阻塞的语句的状态是 Waiting for table metadata lock ，如果权限足够的话，还可以看到blocker thread
- 规避的方法是==避免在业务高峰运行DDL语句==，特别是耗时很长的对大表的DDL

#### 外键检查等待对主表记录的S锁
- 如果是insert的目标表有定义外键依赖，MySQL需要做参照完整性（RI）检查，会对主表的对应记录加S锁
- 如果主表记录上正好有没有提交的修改，就会带来insert事务的锁等待
- 下面的语句可以==当场查看正在阻塞的关系==
```
SELECT
r.trx_id waiting_trx_id,
r.trx_mysql_thread_id waiting_thread,
r.trx_query waiting_query,
b.trx_id blocking_trx_id,
b.trx_mysql_thread_id blocking_thread,
b.trx_query blocking_query,
(Unix_timestamp() - Unix_timestamp(r.trx_started)) blocked_time
from information_schema.innodb_lock_waits w
inner join information_schema.innodb_trx b
on b.trx_id = w.blocking_trx_id
inner join information_schema.innodb_trx r
on r.trx_id = w.requesting_trx_id
```
- 规避方法
1. 加速释放主表的X行锁，避免长事务；
2. 或者是通过业务而不是MySQL数据库来保证参照完整性

#### insert转update操作需要拿老记录index entry上的S/X锁
- 如果新插入的记录项已经存在，但已经被标记为已删除，或者是使用了INSERT ON DUPLICATE KEY UPDATE, MySQL会将insert操作转成update操作
- 就会对老的index 项目加上X锁(如果是cluster index对应的记录则加S锁)，来确保原有记录的删除/修改事务已经提交
- 定位方法：
1. 查看慢日志的lock time字段来判断
2. 如果发生了死锁，还可以查看master error log
3. 也可类似MDL锁等待，从I_S表查看当前的阻塞关系


#### iops限制
后台大IO应用导致insert变慢

#### 写binlog延迟
磁盘无空间，导致binlog写入hang住

#### semi-sync消息延迟
rpl_semi_sync_master_timeout参数值，semi-sync可能导致insert延迟对应的时间