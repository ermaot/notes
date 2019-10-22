## 相关文件
主要文件：log0*.*
## mini-transaction
#### 基本概念
- 重做日志根据物理页组织的
- mini-transaction 实现innodb的物理逻辑日志写入，保证并发状态和数据库异常时数据的一致性（data consistency）
- mini-transaction包括下面步骤
1. lock the page in exclusive mode
2. transform page
3. generate redo and undo log
4. unlock the page
#### the fix rules
- 访问或者修改一个页的时候，需要持有该页的latch
- 获得这个页latch的操作称为fixing the page；页获取latch称为fixed；释放latch称为unfixing
- fixing rules
1. 修改页需要获得x-latch
2. 读取或者x-latch或者s-latch
3. 持有latch直到页的读取或者修改完成
- 每个innodb页都有buf_block_t对象，定义如下：

```
struct buf_block_struct{
    ……
    rw_lock_t lock;  //实现对该页的latch操作
    ulint buf_fix_count;   //表示有多少个操作fix改页
    ……
    
}
```
- innodb 对latch的rules做了调整，若操作的页是B+索引中的非叶子节点页，那么非叶子节点通过B+树的latch保护，因此不一定需要这些页的latch
- 判断是否被fix的标准不是是否持有latch，而是buf_fix_count 是否为0
#### write-ahead log(wal)
- WAL即要求一个页持久化时，首先必须将内存中的日志写入到持久化设备
- 实现方式：
1. 每一个页需要一个LSN
2. 每一个页的修改需要维护该LSN
3. 页刷新到持久化设备的时候，要求内存中小于该LSN的日志都需要刷新到持久设备
4. 日志的页持久到设备上之后，内存中的页写入到持久化设备
5. 内存中的页写入的时候，该页需要fixed，直到写入结束
#### force log at commit
- innodb_flush_log_at_trx_commit 值可为0，1，2，由函数 log_flush_up_to 实现
1. innodb_flush_log_at_trx_commit = 0 表示事务完成时不刷日志到磁盘，而等master每秒刷新（宕机可能损失最后一秒数据）
2. innodb_flush_log_at_trx_commit = 1 表示commit时同步刷到磁盘
3. innodb_flush_log_at_trx_commit = 2
## 具体实现
- mtr_struct 结构用来实现mini-transaction

变量 | 类型| 说明
---|---|---
state | ulint | 当前mini- transaction的状态,有效值为:<p>MTR_ACTIVE<p>MTR_COMMITTING<p>MTR_COMMITTED<p>
memo | dyn_array_t | 持有的 latch信息
log  | dyn_array_t| 产生的日志
modifications  | ibool | 事务是否有触发页的更改
n_log_recs   | ulint| 有多少页的日志被写人到变量log中.<p>一个mini-ransactin可以同时修改多个页
log_mode   | ulint| MTR_LOG_ALL<p>MTR_LOG_SHORT_INSERTS<p>MTR_LOG_NONE
start_ lsn | dulint | mini-transaction开始时的LSN
end_ lsn | dulint | mini-transaction结束时的LSN
magic_n | ulint | 仅在DEBUG模式下使用
- memo 存储latch信息，也是为了遵守fix rules规则，保存的数据结构为mtr_memo_slot_struct\

变量 | 类型| 说明
---|---|---
type | ulint | latch信息
object | void* | latch对象，可以是rw_lock_t对象，也可以是buf_block_t


#### 数据结构
#### 物理逻辑日志的实现
- 重做日志是物理逻辑的，需要记录重做日志的类型（共32种，在mtr0mtr.h中定义）
#### mini-transaction的使用（==此处需要再好好看看==）
- log_sys->mutex是热点，性能瓶颈：写入重做缓冲需要该量，从重做缓冲写入文件也需要该量
- 重做缓冲写重做日志文件的fsyc时，其他事物可以持有log_sys->mutex，所以事务可以组提交
- 事务没有修改页的时候，也需要mini-transaction，以符合fix rules原则；mtr_commit时，对页仅仅unfix操作
## 示例

