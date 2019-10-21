
## 组提交简述
- 组提交(group commit)是MYSQL处理日志的一种优化方式，主要为了解决写日志时频繁刷磁盘的问题。
- 组提交伴随着MYSQL的发展不断优化，从最初只支持redo log 组提交，到目前5.6官方版本同时支持redo log 和binlog组提交。
- 组提交的实现大大提高了mysql的事务处理性能

## redo log 组提交
- WAL(Write-Ahead-Logging)是实现事务持久性的一个常用技术，基本原理是在提交事务时，为了避免磁盘页面的随机写，只需要保证事务的redo log写入磁盘即可，可以通过redo log的顺序写代替页面的随机写，但每次事务提交，仍然需要有一次日志刷盘动作
- 组提交思想是，将多个事务redo log的刷盘动作合并，减少磁盘顺序写
#### 组提交过程：
- Innodb的日志系统里面，每条redo log都有一个LSN(Log Sequence Number)，LSN是单调递增的。
- 每个事务执行更新操作都会包含一条或多条redo log，各个事务将日志拷贝到log_sys_buffer时(log_sys_buffer 通过log_mutex 保护)，都会获取当前最大的LSN，因此可以保证不同事务的LSN不会重复
- 组提交的基本流程如下：
1. 获取 log_mutex
2. 若flushed_to_disk_lsn>=lsn，表示日志已经被刷盘,跳转5
3. 若 current_flush_lsn>=lsn，表示日志正在刷盘中，跳转5后进入等待状态
4. 将小于LSN的日志刷盘(flush and sync)
5. 退出log_mutex


#### 组提交优化
- 开启binlog的情况下，prepare阶段，会对redo log进行一次刷盘操作(innodb_flush_log_at_trx_commit=1)，确保对data页和undo 页的更新已经刷新到磁盘
- commit阶段，会进行刷binlog操作(sync_binlog=1),并且会对事务的undo log从prepare状态设置为提交状态(可清理状态)
- 通过两阶段提交方式(innodb_support_xa=1)，可以保证事务的binlog和redo log顺序一致。
- 故障恢复时，扫描最后一个binlog文件(在flush阶段，判断binlog是否超过阈值，进行rotate binlog文件，rotate的binlog文件中对应的事务一定是已经提交的，处于prepared的事务的binlog还没有刷进来，因为还没进入ordered_commit函数)，提取其中的xid；重做检查点以后的redo日志，读取事务的undo段信息，搜集处于prepare阶段的事务链表，将事务的xid与binlog中的xid对比，若存在，则提交，否则就回滚
- 每个事务提交时，都会触发一次redo flush动作，因此很影响系统的吞吐量。==可将prepare阶段的刷redo动作移到了commit(flush-sync-commit)的flush阶段之前，刷redo刷到当前的log_sys->lsn（即最大的lsn），就保证刷binlog之前，一定会刷redo==

## binlog的组提交
- 5.6以前，mysql在开启binlog的情况下，通过一个的prepare_commit_mutex，将redo log和binlog刷盘串行化，以保证redo log-Binlog一致，无法组提交，牺牲了性能
- binlog组提交的基本思想是，引入队列机制保证innodb commit顺序与binlog落盘顺序一致，并将事务分组，组内的binlog刷盘动作交给一个事务进行，实现组提交目的

#### binlog组提交原理
- binlog提交将提交分为了3个阶段，FLUSH阶段，SYNC阶段和COMMIT阶段。
- 每个阶段都有一个队列，每个队列有一个mutex保护
- 约定进入队列第一个线程为leader，其他线程为follower，所有事情交由leader去做，leader做完所有动作后，通知follower刷盘结束

#### binlog组提交流程
- FLUSH 阶段
1. 持有Lock_log mutex [leader持有，follower等待]
2. 获取队列中的一组binlog(队列中的所有事务)
3. 将binlog buffer到I/O cache
4. 通知dump线程dump binlog
- SYNC阶段
1. 释放Lock_log mutex，持有Lock_sync mutex[leader持有，follower等待]
2. 将一组binlog 落盘(sync动作，最耗时，假设sync_binlog为1)
- COMMIT阶段
1. 释放Lock_sync mutex，持有Lock_commit mutex[leader持有，follower等待]
2. 遍历队列中的事务，逐一进行innodb commit
3. 释放Lock_commit mutex
4. 唤醒队列中等待的线程

由于有多个队列，每个队列各自有mutex保护，队列之间是顺序的，约定进入队列的一个线程为leader，因此FLUSH阶段的leader可能是SYNC阶段的follower，但是follower永远是follower