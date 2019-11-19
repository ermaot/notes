## 相关文件
主要文件log0log.*、log0recv.*

## 相关概念
#### 简介
- redo log用来实现ACID中的D（持久化）
- 包括重做日志缓冲（redo log buffer易失的）与重做日志文件（redo log file持久的）
- force log at commit：事务提交时，必须将该事务所有的重做日志进行持久化
- redo log 是用来保证事务持久性（D特性），顺序写，数据库运行时不需要读redo
- undo log 是用来帮助事务回滚以及mvcc，undo 随机读写
- redo 没有使用 O_DIRECT 选项 ，所以先写到文件系统缓存然后fsync写入磁盘；磁盘性能决定事务提交的性能
- innodb 允许 事务提交的时候不立刻写磁盘，显著提高性能的同时也可能宕机而丢失数据
- innodb_flush_log_at_trx_commit 值可为0，1，2
1. innodb_flush_log_at_trx_commit = 0 表示事务完成时不刷日志到磁盘，而等master每秒刷新（宕机可能损失最后一秒数据）
2. innodb_flush_log_at_trx_commit = 1 表示commit时同步刷到磁盘
3. innodb_flush_log_at_trx_commit = 2 表示commit时异步刷到磁盘（有这个动作，但不是肯定会写入而由操作系统决定写入）；mysql宕机但操作系统不宕机，则也不会丢失数据；操作系统宕机，可能丢失未刷入磁盘的那部分数据
![参数不同导致的性能区别](pic/innodb%20%E9%87%8D%E5%81%9A1.png)
- 二进制日志和重做日志
1. 重做日志是innodb产生，而二进制日志是数据库上层产生
2. 内容不同。二进制日志本质是逻辑日志，是sql语句；而重做日志是对每个页的修改，物理日志
3. 写入磁盘时间点不同。redo log 一直在产生和写入，二进制日志是事务提交完毕一次性写入
#### 物理逻辑日志
重做日志分为物理日志、逻辑日志、物理逻辑日志
- 物理日志是记录对每个页的字节的改变；多次重做不会导致数据不一致（幂等性
- ），但日志量大
```
struct value_log_record_for_page_update{
int opcode;
filename fname;
long pageno;
char old_value[PAGESIZE];
char new_values[PAGESIZE];
}

```
记录的是4元组数据，哪个表空间，哪个文件，哪个页，哪个偏移位置，插入的字节内容
- 逻辑日志记录的是对表的操作，类似于二进制日志；恢复时无法保证数据一致性
- 物理逻辑日志：对页是物理的，但页内部是逻辑的

一条插入语句"insert into t1(id,name,age) values(1,'小明',26);"会记录3条类似的日志（t1有name和age两个索引）：
```
<insert op,base filename = T1,page number = 502, record value = {1,'小明',26}>
<insert op,index1 filename = idx_name ,page number = 72,index1 record value = index_name of r = "小明">
<insert op,index1 filename = idx_age ,page number = 50,idx_age record value = idx_age of r = 26>
```
#### lsn
- log sequence number，日志编号，单调递增，每一个号码与对应一个日志
- lsn 8个字节，dulint_struct 结构

```
typedef	struct dulint_struct	dulint;
struct dulint_struct{
	ulint	high;	/* most significant 32 bits */
	ulint	low;	/* least significant 32 bits */
};
```
- LSN 初始值由 LOG_START_LSN 定义

```
#define LOG_START_LSN	ut_dulint_create(0, 16 * OS_FILE_LOG_BLOCK_SIZE)

```
- LSN记录的是日志的增量，单位是字节（此处可继续研究）
- LSN存在于（重做日志、检查点、页）中
- 页LSN：每一个页的头部，有一个 FIL_PAGE_LSN ，表示最后刷新的事务LSN大小，以判断是否需重做
- 检查点LSN：检查点用LSN形式保存，记录已经刷新到磁盘的检查点序号。
- 查看LSN

```
> show engine innodb status;
……………………
……………………
LOG
---
Log sequence number 3356699403
Log flushed up to   3356699403
Last checkpoint at  3356699403
Max checkpoint age    7782360
Checkpoint age target 7539162
Modified age          0
Checkpoint age        0
0 pending log writes, 0 pending chkp writes
687 log i/o's done, 0.00 log i/o's/second
……………………
```
#### REDO落盘时间点
1. 首先写入日志缓冲区（log buffer），由参数innodb_log_buffer_size控制
2. 接着从日志缓冲区刷新到磁盘（并非事务提交后再刷盘）
#### 检查点
- innodb为了实现持久性，采用WAL（write ahead logging）。即事务提交的时候，先将重做日志写入到文件，实际数据页刷新到磁盘的操作由检查点完成。
- innodb中存在sharp checkpoint 和 fuzzy checkpoint
1. sharp checkpoint 将脏页一次性全部刷新到磁盘，速度快但不能同时其他的dml操作（一般不建议线上使用）
2. fuzzy checkpoint 将脏页慢慢刷到磁盘，但需要将脏页根据LSN排序并依次刷到磁盘
#### 归档日志
- round robin（重做日志组满了，就循环使用重做日志）
- 为了前面的redo log不被覆盖，设置归档日志
#### 恢复
- 数据库启动时，都会做数据库恢复操作
- 重做日志是物理日志，比逻辑日志快
- innodb 对恢复做了顺序读取以及并行优化，提升恢复速度
## 物理存储结构
#### 重做日志物理架构
- 重做日志包括：重做日志缓存；重做日志文件组；重做日志文件组中的日志文件；归档重做日志文件
- 重做日志缓存：内存中，易失；innodb_log_buffer_size 控制，默认1M
- 重做日志文件组内容完全一样，镜像关系，提高可用性，但innodb_mirrored_log_groups 不能取1 之外的值
- 重做日志文件可以由多个同样大小文件组成，前缀是“ib_logfile”
- 归档日志防止重做日志丢失（==这部分需要再看==）
#### 重做日志块
- 重做日志都是以512字节存储的；如果大于512，需要分割多个来存储
![image](pic/innodb%20%E9%87%8D%E5%81%9A2.png)
- 512与扇区大小一致，所以无需double write
- 重做日志块有块头（log block header），块尾（log block tail），日志本身三个部分
![重做日志块结构](pic/innodb%20%E9%87%8D%E5%81%9A3.png)
- LSN计算举例

```
当前重做日志的LSN = 2048,这时候innodb调用log_write_low写入一个长度为700的日志，
2048刚好是4个block长度，那么需要存储700长度的日志，
需要量个BLOCK(单个block只能存496个字节)。那么很容易得出新的
LSN = 2048 + 700 + 2 * LOG_BLOCK_HDR_SIZE(12) + LOG_BLOCK_TRL_SIZE(4) = 2776

原文：https://blog.csdn.net/yuanrxdu/article/details/42430187 

```

#### 重做日志组和文件
## 相关数据结构
#### log_group_struct
#### log_struct
## 组提交
## 恢复

## REDO设置
- 1.show engine innodb
```
---
LOG
---
Log sequence number          44845773		//REDO缓冲区中的LSN
Log buffer assigned up to    44845773			
Log buffer completed up to   44845773
Log written up to            44845773
Log flushed up to            44845773		//REDO日志文件中的LSN
Added dirty pages up to      44845773
Pages flushed up to          44845773		//写入磁盘数据页对应的LSN
Last checkpoint at           44845773		//最后checkpoint对应的LSN
562 log i/o's done, 0.00 log i/o's/second
----------------------
```
- 2.innodb_metrics
```

```
- 3.sys schema