## 存储引擎层面丢失数据
如何控制事务写入到磁盘（redo log）的时机？通过配置参数innodb_flush_log_at_trx_commit
- 0 ：每秒 write cache & flush disk（最不安全数据会丢失，性能为最高的）
- 1 ：每次commit都 write cache & flush disk（最为安全数据不会丢失，每次commit都保证redo写入了disk。但是性能比较低）
- 2 ：每次commit都 write cache，然后根据innodb_flush_log_at_timeout（默认为1s）时间 flush disk

## 主从复制层面丢失数据
MySQL XA分为两类，内部XA与外部XA
- 内部XA用于同一实例下跨多个引擎的事务，由Binlog作为协调者
- 外部XA用于跨多个MySQL实例的分布式事务，需要应用层介入作为协调者(崩溃时的悬挂事务，全局提交还是回滚，需要由应用层决定，对应用层的实现要求较高)；

#### 内部XA事务原理
内部XA事务简化的大致流程（2PC）：
1. 事务提交后，InnoDB存储引擎会做一个prepare操作，将事务的XID写入到redo log中。
2. 写binlog日志。
3. 再该事务的commit信息写入到redo log中

如果是在步骤1和2时失败，整个事务回滚。
如果是在步骤3时失败，MySQL在重启后会首先检查UXID是否已经提交，若没有提交，则在存储引擎再执行一次提交操作

#### binlog刷新机制
1. sync_binlog = 0 ：表示MySQL不控制binlog的刷新，由文件系统自己控制它的缓存的刷新（不安全）
2. sync_binlog> 0 ：表示每sync_binlog次事务提交，MySQL调用文件系统的刷新操作将缓存刷下去。其中最安全的就是sync_binlog设置为1，表示每次事务提交，MySQL都会把binlog缓存刷下去


innodb_flush_log_at_trx_commit不等于1，可能会丢数据

#### slave非实时写redo和binlog丢失数据
在slave机器上会存在三个文件来保证事件的正确重放：relay log、 relay log info、 master info：
- relay log：即读取过来的master的binlog，内容与格式与master的binlog一致
- relay log info：记录SQL Thread应用的relay log的位置、文件号等信息
- master info：记录IO Thread读取master的binlog的位置、文件号、延迟等信息

因此如果当这3个文件如果不及时落地，则MySQL crash后会导致数据的不一致

#### Master宕机后无法及时恢复造成的丢失数据
当master出现故障后：
- 如果master不切换，则整个数据库只能只读，影响应用的运行
- 如果将某个的slave提升为新的master，那么原master未来得及传到slave的binlog的数据则会丢失。
1. 各个slave之间接收到的binlog不一致，如果强制拉起一个slave，则slave之间数据会不一致
2. 原master恢复正常后，由于新的master日志丢弃了部分原master的binlog日志

解决办法：
- 确保binlog传到从库，或者说保证主库的binlog有多个拷贝
1. 使用semi sync（半同步）方式，事务提交后，必须要传到slave，事务才能算结束。对性能影响很大，依赖网络适合小tps系统
2. 双写binlog，通过OS层的文件系统复制到备机，或者使用共享盘保存binlog日志
3. 在数据层做文章，比如保证数据库写成功后，再异步队列的方式写一份，部分业务可以借助设计和数据流解决
- 允许数据丢失，制定一定的策略，保证最小化丢失数据。
1. 淘宝的TMHA复制管理工具。当master宕机后，TMHA会选择一个binlog接收最大的slave作为master。当原master宕机恢复后，通过binlog的逆向应用，把原master上多执行的事务回退掉