## Innodb buffer pool
- buffer pool是一块连续的内存，是数据库系统中拥有最大块内存的系统模块，主要用来存储访问过的数据页面
- innodb的数据访问是按照页（也叫块，默认16KB）从数据库文件读取到buffer pool中，然后在内存中用同样大小的内存空间来映射。
- 提高访问效率，数据库系统预先分配了这样的空间
- 访问时按照LRU算法实现buffer pool管理
- buffer pool 大小使用innodb_buffer_pool_size大小来决定，默认为128M。MySQL5.7.4之前，启动后不能更改，5.7.5版本后可以动态修改
- 如果buffer pool大小超过1GB，应该调整innodb_buffer_pool_instances个数的方法分割为多个，提升并发能力（buffer pool通过链表方式管理页面，存取时要加锁，所以可能出现锁竞争和等待，如果使用多个，则可以提升并发）
## buffer pool实现原理
- MySQL启动时，会启动所有的内嵌存储引擎（包括Innodb），Innodb通过函数buf_pool_init初始化所有子系统（包括Innodb buffer pool）
- Innodb将innodb_buffer_pool_size平分成innodb_buffer_pool_instances个，每个实例各自初始化
- buffer pool实例用buf_pool_t结构体描述，包括：
1. FREE链表，存储实例中空闲页面
2. flush_list列表：用于存储所有被修改过且需要刷到文件的页面
3. mutex：保护buffer pool实例
4. chunks：指向这个buffer pool实例的第一个真正内存页面的首地址
- 上面的两个链表，管理的对象是结构体buf_page_t，是物理页面在内存中的管理结构，包括：所属表空间，PageID，最新及最早被修改的LSN，以及形成Page链表的指针
- buf_page_t被另外一个结构buf_block_t管理，他们一一对应，buf_page_t是逻辑的，buf_block_t包含物理的概念
- buf_chunk_init初始化buffer pool的实例内存空间
- buffer  pool实例的内存是一块连续的内存空间，包括两部分：前面是缓存页面的控制头结构信息（buf_block_t结构），每一个控制头信息管理一个物理页面
- Innodb_buffer_pool_bytes_data 总比Innodb_buffer_pool_size小，因为控制头占用了空间
```
> show  status like "%Innodb_buffer_pool_bytes_data%";
+-------------------------------+---------+
| Variable_name                 | Value   |
+-------------------------------+---------+
| Innodb_buffer_pool_bytes_data | 4931584 |
+-------------------------------+---------+
1 row in set (0.00 sec)

> show variables like "%Innodb_buffer_pool_size%";
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)
```
- buffer pool页面从实例池中从后向前分配，每次一个页面；而控制结构从前往后分配，每次一个buf_block_t大小，直到相遇。可能中间会剩余一部分没有被使用。所有的控制头连续在一起，而所有的page也连续在一起
![image](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%861.png)
- 初始化页面控制信息通过buf_block_t实现，包括4个信息：
1. 其对应的页面地址frame
2. 页面信息结构buf_page_t，包括表空间所属的ID号，页面号，被修改时产生的LSN、使用状态（9种）
3. 保护这个页面的互斥量mutex
4. 访问页面时对页面上的锁lock（read/write）
- 初始化完每一个页面，都加入到空闲页链表（buf_block_not)used）
- 多个buffer pool之间，完全独立、单独申请、单独管理、单独刷盘
- 不同的页面，通过hash算法，映射到一个具体的实例中

## redo log日志文件管理的用途
- redo log是用来做数据库的crash recovery的，默认有2个日志文件（ib_logfile0和ib_logfile1）。如果不存在，Innodb会根据配置参数或者默认值，重新创建
- LSN是log sequence number，精确记录日志位置信息，连续增长，大小为8字节
- LSN增长量根据一个MTR写入日志量计算。是一个逻辑概念+物理概念（一些数据库中，LSN是完全的逻辑概念，每提交一个物理事务，LSN加1）
- innodb通过日志组管理日志文件，每个日志组包含若干个日志文件，每个日志文件大小相等
- 日志文件页面大小512字节，每次写入是机械硬盘块大小（512字节）的整数倍效率最高
- 4个页面（2048字节），主要用于管理日志内容及整个数据库状态
![image](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%862.png)
- 普通页面，都会有12个字节用来存储页面头信息
1. LOG_BLOCK_HDR_NO:4个字节,一个与LSN有关系的块号.
2. LOG_BLOCK_HDR_DATA_LEN:2个字节,表示当前页面中存储的日志长度,这个值一般都等于512-12(12为页面头的大小),因为日志在相连的块中是连续存储的,中间不会存在空闲空间,所以如果这个长度不为500,表示此时日志已经扫描完成( Crash Recovery的工作).
3. LOG_BLOCK_FIRST_REC_GROUP:2个字节,表示在当前块中是不是有一个MTR的开始位置.因为一个MTR所产生的日志量有可能是超过一个块大小的,那么如果一个MTR跨多个块时,这个值就表示了这个MTR的开始位置究竟是在哪一个块中.如果为0,则表示当前块的日志都属于同一个MTR;而如果其值大于0并且小于上面LOG_BLOCK_HDR_DATA_LEN所表示的值,则说明当前块中的日志是属于两个MTR的,后面MTR的开始位置就是LOG_BLOCK_FIRST_REC_GROUP所表示的位置.
4. LOG_ LOCK_CHECKPOINT_NO:4个字节,存储的是检查点的序号
![image](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%863.png)

## MTR Innodb物理事务
- 一种很重要的保证物理页面写入操作完整性和持久性的机制
- mini-transaction，物理事务，缩写MTR，相对逻辑事务而言
![image](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%864.png)
- 不管读还是写，只要使用到底层buffer pool都会用到MTR，是上层逻辑层与物理层的交互接口，保证物理层的数据正确性、完整性和持久性
- 访问页面，系统都会将要访问的页面载入到buffer pool
- innodb采用write ahead log（WAL），所有写操作都会有日志记录
- 写日志是物理操作，也需要完整性，即物理事务
- 物理事务包括开始与提交。开始就是对物理事务结构体mtr_struct初始化

mtr_struct 结构用来实现mini-transaction

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

#### mtr过程
1. 系统指定获取方式（读或者写）、页面表空间ID、页面号，通过buf_page_get获取指定页面，然后通过结构体buf_block_struct的lock检查页面上锁情况（没有上锁就直接上锁，如果上锁了可以兼容，则可以上锁，否则必须等待）
2. 上锁成功后，物理事务将页面内存结构存储到memo动态数组中，然后物理事务可以访问该页面
3. 物理事务有两种操作：读（简单读取指定页面内偏移和长度的数据）和写（指定某一个偏移开始写入指定长度的新数据）。如果物理事务是写日志的（MTR_LOG_ALL），则写入REDO，然后将其存储到log动态数组中，自增结构体的n_log_recs维护物理事务的日志计数值
4. 物理事务提交用mtr_commit实现，一个逻辑事务由多个物理事务组成。物理事务的提交主要是将所有这个物理事务产生的日志写入到innodb日志系统日志缓冲区中，然后等待srv_master_thread线程定时将日志系统的日志缓冲区的日志数据刷到日志文件中
5. 日志缓冲区存储的是暂时的中间状态，缓冲区大小可以用过innodb_log_buffer_size设置，一般较小
6. 最后提交的MTR是整个数据库的LSN，即show engine innodb status显示的log中第一行的LSN
```
> show engine innodb status;
---
LOG
---
Log sequence number 2625628
Log flushed up to   2625628
Pages flushed up to 2625628
Last checkpoint at  2625619
```
7. 日志缓冲去如果被占满，会刷盘到日志文件中。缓冲区中的内容，完全一模一样转入文件中，MTR完全连续
8. 最新写入日志文件的MTR的LSN值（Log flushed up to ），就是日志最新写入文件的LSN。到这个LSN位置，所有的修改都是完整的，数据库crash了这个位置的数据是可恢复的
9. 设置innodb_flush_log_at_trx_commit为1，当前事务提交肯定会将日志缓冲区的日志刷到日志文件中，如果设为2，则只是写入了操作系统的缓存，可能会丢失部分已经提交的事务；如果设为0，innodb只将事务对应的日志写入日志缓冲区，完全没有安全性
10. 日志循环使用，必须按照顺序写，即从LSN最小日志开始，按照从小到大的顺序不断让日志失效
11. 每次检查点时系统会根据最小有效LSN（min_valid_lsn）和检查点处理的日志比例计算出最大的将要失效的LSN（lsn_checkpoint_up_to），然后再去扫描buffer pool的flush list链表，找出所有被更新过的页面中，曾经修改这些页面的MTR对应的LSN最小值（即buf_block_t的oldest_modification）
12. lsn_checkpoint_up_to就是show engine innodb status的last checkpoint at值
13. show engine innodb status的page flushed up to是buffer pool中page 刷盘时刷到的最新的LSN 
14. 在物理日志后写一些特殊日志标记，提交时一起将这些特殊日志写入，重做时如果当前日志信息最后存在这个标志，则说明日志是完整的
15. 动态数组memo中存储的是该物理事务访问过的页面，并且已经上锁。如果发现被修改，则加入到innodb buffer pool中的更新链表，srv_master_thread线程定时检查链表，将一定数目的脏页刷到磁盘并释放锁；如果页面没脏或者只是读，则只需要释放共享锁即可

## 日志的意义
- 日志是逻辑事务在对整个数据库做DML操作时，其所包含的物理事务MTR所记录的，针对所有涉及的buffer pool页面的修改记录
- 日志是数据库管理系统和文件系统的最核心的区别
- 日志可以提高数据库性能，原因是
1. 日志是用来记录buffer pool的page修改记录，把对page的写转换为对日志的写，也就是把磁盘写转化为内存写，性能会非常好
2. 通常页大小为16KB，如果不写日志，磁盘的写入单位就是16KB，容易造成写IO浪费
3. 如果没有日志，page写入就是随机而并非顺序，性能很差（==有了redo，就能增加顺序比例？==）
- 日志太小，容易造成太多checkpoint，性能不好；日志太大，重启恢复需要很多时间。一般情况，buffer pool与日志容量大小比例最好10~5：1范围
#### 日志记录的格式
![image](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%865.png)
字段|说明
---|---
type|日志类型，处记录的最高位，一个字节空间
space|表空间ID，如果是系统页面（undo页面或者字典表页面），为0；如果是索引页面，则是索引所在表空间的ID值
offset|space指定的文件中的页面号（从0开始计数），以页面大小为单位
data|表示这条日志记录对应的数据，不同的type有不同的data格式

- type的典型类型

类型|说明
---|---
MLOG_1BYTE、MLOG_2BYTES、MLOG_4BYTES、MLOG_8BYTES|表示在某一个位置，写入特定字节的内容。data中，前2字节描述数据内容在页面内的偏移值，后面是真正需要写入的内容（2，4，8字节）
MLOG_WRITE_STRING|编程数据，2字节写位置，2字节写长度，最后写字符串
MLOG_COMP_REC_MIN_MARK|将一条记录设置为页面中的最小记录，打个标记，只需要记录最小记录在页面内的偏移位置
MLOG_UNDO_INSERT|插入一条记录时，还要写一个针对该记录的回滚记录
MLOG_INIT_FILE_PAGE|比较简单，只有前面的基本头信息，没有data，表示初始化一个页面
MLOG_COMP_PAGE_CREATE|页面索引、数据存储方面管理信息的初始化（创建页面中最小记录与最大记录，初始化页面中的记录数，heap大小、heap首地址和槽信息），在MLOG_INIT_FILE_PAGE基础上做
MLOG_MULTI_REC_END|REDO的额外日志，表示MTR日志写成功
MLOG_COMP_REC_CLUST_DELETE_MARK|表示需要将聚集索引中的某一个记录打删除标志。内容除了基本字段外，data部分主要包括两个字节的索引列的合数n，两个字节的唯一索引个数u，以及索引中两个系统列信息（trxid在索引中的列位置信息、rollptr值和trxid值），以及当前删除记录在所在页面中的偏移值（记录头指针信息）
MLOG_COMP_REC_UPDATE_IN_PLACE|就地更新记录
MLOG_COMP_REC_DELETE|删除操作是通过打删除标记然后事务提交之后purge。data中记录的是记录在页面内的偏移
MLOG_COMP_PAGE_REORGANIZE|重组指定的页面，没有data部分。
MLOG_COMP_REC_INSERT|记录基本的日志头信息，被插入记录在页面内的偏移，然后计算当前插入记录与前一条记录第一个不同字节的位置，再在日志中记录从此位置开始到当前记录结束位置之间的数据（前半部分依赖前一条记录，当前记录存储后半部分数据）

![MLOG_COMP_REC_CLUST_DELETE_MARK格式](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%866.png)
MLOG_COMP_REC_CLUST_DELETE_MARK格式为什么要记录索引字段数、唯一键数等信息？因为REDO是半逻辑半物理的，恢复时对应的数据字典可能是无用的

#### 日志刷盘的时机
- log buffer空间用完，便会刷log buffer到磁盘。比较普遍
- master后台每秒刷一次
- 每次执行DML，都会主动检查日志空间是否足够；如果超过预设的经验值，就主动刷日志（写入os缓存）
- 检查点时，保证所有要刷的数据页面中LSN最小日志已经刷入
- 提交逻辑事务，innodb_flush_log_at_trx_commit的值为0，只写入log buffer；为1，直接写磁盘；为2，只写os cache

## REDO恢复
- 不管数据库用何种方式关闭，启动时都会redo。只是正常关闭的数据库checkpoint已经最新，无需恢复
#### 日志扫描
- 数据库恢复时，首先要做的是扫描日志文件找到最新检查点信息。日志文件最开始的4个页面中，检查点信息存储在第一号页面和第三号页面（LOG_CHECKPOINT_1和LOG_CHECKPOINT_2），轮换使用
- 检查点最新位置，就是LOG_CHECKPOINT_1和LOG_CHECKPOINT_2的最大值，其余日志都是失效的，已写入
- 函数recv_recovery_from_checkpoint_start_func做两个操作：
1. 从日志文件的固定位置找到最新的检查点信息
2. 从最新的检查点位置开始扫描日志文件，做数据库恢复
- 函数recv_group_scan_log_recs，将checkpoint_lsn位置开始的日志分片处理，每一片大小为2MB
- innodb本身为了管理日志文件将连续的日志内容以块为单位存储，加上头尾信息，连续存储；而使用的时候，以块为单位读取掐头去尾
- innodb将每一个日志记录分开之后，存储到了表空间ID和页面号为键值的hash表中。所以相同页面肯定存储在一起的，保证REDO的有序性
- 做完redo后，做一次检查点以说明这次数据库恢复完成
- innodb将日志hash存储的原因：
1. 同一个页面的REDO，必然存储在同一个hash桶
2. 对于某一个页面所有的日志记录，按照先后顺序管理
3. redo 可以并行恢复，只要同一个hash桶进入同一个线程

## 数据库回滚
- 调用recv_recovery_from_checkpoint_start后又调用recv_recovery_from_checkpoint_finish（数据库回滚）
- innodb的REDO先于undo之前
#### 数据库undo段管理
- innodb中用第6个页面（5号）来管理回滚段信息

字段|说明
---|---
TRX_SYS_TRX_ID_STORE|用于存储事务号。每次启动新事务，就会检查当前事务号是不是已经达到TRX_SYS_TRX_ID_WRITE_NARGIN（256）的倍数，如果达到就会将最大事务号写入，下次启动就取出该只再加上步长256，保证事务号唯一
TRX_SYS_FSEG_HEADER|存储事务段信息
TRX_SYS_RSEGS|数组，innodb有128个回滚段，数组长度就是128，每一个元素8个字节，对应回滚段存储内容是回滚段首页面的表空间ID和页面号

从页面偏移38开始（38之前是文件的通用管理信息），存储5个信息
字段名|说明
---|---
TRX_RSEG_MAX_SIZ|回滚段管理页面的总数量，即所有undo段页面之和，一般为ULINT_MAX无上限
TRX_RSEG_HISTORY_SIZE|history list中有多少页面，即需要pruge的回滚段页面数
TRX_RSEG_HISTORY|history list链表首地址，事务提交后对应的回滚段如果还不能purge就会加入到这个链表
TRX_RSEG_FSEG_HEADER|用来存储回滚段的inode
TRX_RSEG_UNDO_SLOTS|一个数组，长度为1024，每一个元素都是一个页面号，初始化为FIL_NULL空值

- innodb支持的回滚段为128*1024=131072个，TRX_RSEG_UNDO_SLOTS数组每一个元素都会指向一个页面，这个页面对应一个段，页面号就是段首页的页面号
- 事务执行过程中，会产生两种回滚日志，一种是insert的undo，一种是update的undo（包括delete）
==（这部分暂放，后面继续写）==
#### 数据库undo日志记录格式
undo类型有4种
类型|说明
---|---
TRX_UNDO_INSERT_REC|记录插入的undo日志类型，只记录了表ID和主键，回滚时通过主键找到原B+树记录删除即可
TRX_UNDO_UPD_EXIST_REC|更新的UNDO。记录除表ID信息外还需要记录每一个被更新的列的原始值和新值以及主键信息。根据主键找到记录以旧换新
TRX_UNDO_UPD_DEL_REC|更新一条打了删除标志的记录的undo。格式与上一样，回滚方式也一样
TRX_UNDO_DEL_MARK_REC|删除记录时对记录打删除标志的undo。格式与插入操作undo一样。根据主键找到记录去掉删除标志即可。

- 除了上面说到的tableid信息主键信息外，还会包括一些公有的信息比如回滚段指针、最新更新事务号，方便mvcc回溯记录找到之前的版本
- 记录格式如下：
![image](pic/Innodb%E7%9A%84%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%867.png)
1. 最先的2字节和最后的2字节都是用来方便找到每一个记录并通过他们找到所有记录（类似于Undo记录组成的双向链表）
2. 第二个位置存储的是记录类型
3. 第三个位置存储的是事务的undo_no，用来区分一个事务中的多个undo顺序
4. 第四个位置存储的回滚记录对应的表ID
5. 第五个位置trx_ID就是更新这条记录时的事务ID 
6. roll_ptr用来存储当前被更新记录上一个版本在回滚段中的位置
7. 存储的是真实主键信息
- undo日志很多位置都是压缩存储的
- undo不会跨页面
#### 回滚时刻
- redo完成后，innodb通过函数trx_sys_init_at_db_start将所有回滚段相关的128*1024个undo扫描出来，然后缓存
- 然后通过函数trx_lists_iit_at_db_start依次处理每个undo段，undo段的状态如果是TRX_UNDO_REPEATED和TRX_UNDO_ACTIVE，则需要回滚
- 识别出需要回滚的undo，然后根据TRX_UNDO_TRX_NO从大到小排列
- 最后在innodb启动时函数recv_recovery_from_checkpopint_finish做回滚