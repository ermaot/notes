两次写（double write）是Innodb很独特的功能点
## 为何需要两次写
- Innodb的日志是逻辑的。插入一条记录时，会在某一个页面的多个偏移位置写入某个长度的值，例如页头的记录数、槽数、页尾槽数据、页中的记录值等。Innodb为了节约日志量，设计为逻辑处理的方式，真正物理插入的时候，才会将日志逻辑操作转化为物理操作
- 如果页本身是错误的（写断裂导致页面不完整），写就会出现问题
## 两次写详述
##### 两次写过程
两次写
![double write](pic/Innodb%E4%B8%A4%E6%AC%A1%E5%86%99.png)
共享表空间为2个区（extent），128个页，共2M
- 两次写提升可靠性
- 部分写失效：数据库宕机的时候，可能出现页未写完的情况
- 两次写步骤：
1. 缓冲池的脏页刷新时，通过memcpy拷贝到double write buffer
2. 通过double write buffer 分两次，每次1MB到的共享表空间磁盘，然后fsync函数同步到磁盘（顺序写，性能好）
3. double write页完成后，再将double write buffer中的页写入到表空间的文件中（离散写）
4. 查看double write 命令如下：
```
> show global status like "%dblw%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Innodb_dblwr_pages_written | 7928  |
| Innodb_dblwr_writes        | 2394  |
+----------------------------+-------+

```
- 如果Innodb_dblwr_pages_written / Innodb_dblwr_writes 远小于64，说明压力不是很大
- skip_innodb_double_writer可以禁止double write
- 在一些已经有防止写失效的文件系统（zfs），可以关闭double write
两次写，包括两部分，一部分是对单独一个页面刷盘时两次写，另一部分是批量刷盘时的两次写
#### 单一页面刷盘
单一页面刷盘，是MySQL5.5的实现方式。MySQL在系统页面（ibdata文件的第五个页面）存储两次写的信息，偏移位置是页面结束为止的最后200字节处
- 第一次建库的时候，会分配一个段的空间，段地址会存储到两次写信息偏移TRX_SYS_DOUBLEWRITE_FSEG的位置，每次都从这个位置读取段位置今儿找到段首地址
- 偏移TRX_SYS_DOUBLEWRITE_MAGIC的位置，存储当前两次写是否正常或者已经申请的标志
- 偏移TRX_SYS_DOUBLEWRITE_BLOCK1和TRX_SYS_DOUBLEWRITE_BLOCK2存储的是两次写空间的位置，在ibdata文件中同属一个段。对应的空间大小是一个簇（1MB），所以共2MB
- 偏移TRX_SYS_DOUBLEWRITE_REPEAT所指的位置用来将前面的MAGIC、BLOCK1、BLOCK2重复存储一次，保证数据安全性
- 每次刷盘前，都将刷盘的页面信息临时保存到数组中，即两次写缓存数组
- 两次写过程：
1. 先在两次写缓存数组中，找到一个空闲位置，并将这个位置标记为已使用
2. 再把整个页面的数据复制到空闲位置对应的缓存空间中
3. 复制完成后，系统将页面的数据刷到两次写文件中（ibdata文件）。复制的数据量是一个页面大小，偏移位置是这个页面在两次写缓存空间中的位置，与TRX_SYS_DOUBLEWRITE_BLOCK1和TRX_SYS_DOUBLEWRITE_BLOCK2对应
4. 单一页面刷到ibdata文件的两次写位置后，页面就可以刷盘到真实位置。buffer pool页面，刷到真实文件时是异步的，只有当自己表空间的刷盘操作完成后，两次写缓存数组的数据才可以被覆盖


## 批量页面刷盘
考虑性能，两次写批量刷盘（原文说MySQL5.7以后采用，实际是percona5.7以后采用）
- 新增一个文件，名称和路径可以通过参数innodb_parallel_doublewrite_path控制
```
> select version();
+-----------+
| version() |
+-----------+
| 5.7.27    |
+-----------+
1 row in set (0.00 sec)
> show global variables like "%innodb_parallel_doublewrite_path%";
Empty set (0.00 sec)
```
- 批量刷盘有两种方式LRU和LIST方式
1. buffer poll空间不足时，再载入新的页面就必须将一些不怎么用到的、老的页面淘汰出去，用LRU方式
2. 日志空间不足，或者后台master线程定时刷盘，选择LSN最小的页面，此时就是LIST方式

#### 两次写组织结构
- Innodb自身的buffer poool分为多个实例，每一个实例管理自身的一套两次写空间，每一个实例的刷盘缓存空间大小，由innodb_doublewrite_batch_size控制，默认120
```
> show variables like "%doublewrite%";
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| innodb_doublewrite            | ON    |
| innodb_doublewrite_batch_size | 120   |
+-------------------------------+-------+
2 rows in set (0.00 sec)
```
两次写文件页面个数 = innodb_buffer_pool_instances * 2*(LIST + LRU) * innodb_doublewrite_batch_size

#### 批量刷盘两次写实现原理
- LRU将页面淘汰，淘汰出的页面加入到两次写缓存，根据页面所在的实例号和刷盘类型可以找到对应的shard缓存
- 然后判断shard是否已满（是否达到innodb_doublewrite_batch_size），如果没满则直接写入shard中，刷盘完成
- 如果满了，先将当前shared页面写入两次写文件中，写完之后再将两次写文件flush到磁盘，最后将对应的真实页面刷盘
- 表空间的页面刷盘，是异步IO。需要等异步IO且整个shard中的页面都刷盘之后，表空间的刷盘才能继续向后进行。

#### 两次写的作用
- 数据库异常关闭后再启动时，会做数据库redo，恢复过程中，数据库会检查页面是不是合法。如果发现页面的校验结果不一致，会用到两次写恢复异常页面
- 将两次写的2个簇都读出来，再将innodb_parallel_doublewrite_path的文件内容读取出来，然后写到对应的页面，以保证页面正确

## 总结
- 看上去每个页面都写了两遍，但由于页面先写到缓存再写到文件，因此不会产生太大的性能差距（大约10%）
- 一些硬件厂商与MySQL合作，对两次写做了更彻底的优化，以保证16K页面的原子写入
- 一些数据库没有两次写的设计，是因为采用了全物理的REDO，将一个页面的写操作都拆成一个个小的物理写入，就不会写断裂