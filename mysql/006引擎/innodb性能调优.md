## 选择合适CPU

- OLTP 
1. 用户操作并发量大 
1. 事务操作时间短 
1. 查询较为简单，一般都走索引 
1. 复杂查询少<p>
==总结：对CPU要求不高，IO密集型==

- innodb 后台操作都是在单独的一个master thread完成，支持多核并不好
- innodb plugin 多核支持有了提升，优先选innodb plugin
- innodb_read_io_threads 和 innodb_write_io_threads 控制IO 线程数量

```
> show variables like "%innodb_%_io_threads%";
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_read_io_threads  | 4     |
| innodb_write_io_threads | 4     |
+-------------------------+-------+
2 rows in set (0.00 sec)

```

- innodb 一条sql一般都是在单核上完成，不支持单条sql的多核并行

## 内存的重要性
- 内存大小最能直接反映数据库性能 
- innodb buffer pool 缓存数据，也缓存索引，直接影响数据库性能


参数名 | 解释
---|---
Innodb_buffer_pool_read_ahead | 预读的次数
Innodb_buffer_pool_read_ahead_evicted | 预读的页，没有被读取就从缓存中被替换的页数量<p>用以判断预读的效率
Innodb_buffer_pool_read_ahead_rnd | 记录进行随机读的时候产生的预读次数
Innodb_buffer_pool_read_requests | 从缓冲池中读取页的次数
Innodb_buffer_pool_reads | 从物理磁盘读取页的次数
Innodb_data_pending_reads | innodb当前挂起的读操作数。单位是次
Innodb_data_read | 总共读入的字节数
Innodb_data_reads | 发起读取的次数
Innodb_master_thread_1_second_loops | 
Innodb_master_thread_10_second_loops | 
Innodb_master_thread_background_loops | 
Innodb_master_thread_main_flush_loops | 
Innodb_master_thread_sleeps | 
Innodb_pages_read | innodb总共读取的页数量。单位是page
Innodb_read_views_memory | This status variable shows the total amount of memory allocated for the InnoDB read view (in bytes)
Innodb_rows_read | 物理读数据行数



```
> show global status like "innodb%read%";
+---------------------------------------+-------------+
| Variable_name                         | Value       |
+---------------------------------------+-------------+
| Innodb_buffer_pool_read_ahead         | 600569      |
| Innodb_buffer_pool_read_ahead_evicted | 8633        |
| Innodb_buffer_pool_read_ahead_rnd     | 0           |
| Innodb_buffer_pool_read_requests      | 12332710    |
| Innodb_buffer_pool_reads              | 227018      |
| Innodb_data_pending_reads             | 0           |
| Innodb_data_read                      | 13563466240 |
| Innodb_data_reads                     | 827726      |
| Innodb_master_thread_1_second_loops   | 2046        |
| Innodb_master_thread_10_second_loops  | 186         |
| Innodb_master_thread_background_loops | 179         |
| Innodb_master_thread_main_flush_loops | 179         |
| Innodb_master_thread_sleeps           | 2045        |
| Innodb_pages_read                     | 827586      |
| Innodb_read_views_memory              | 104         |
| Innodb_rows_read                      | 6754125     |
+---------------------------------------+-------------+
---可以计算缓存命中率 = 93.7%
```
[参考博文1](https://blog.csdn.net/yang1982_0907/article/details/20123055)<p>
[参考博文2](http://www.php.cn/mysql-tutorials-369195.html)<p>
[参考博文3](https://blog.csdn.net/shenchaohao12321/article/details/83722357)<p>

InnoDB缓存池的命中率:
命中率 = Innodb_buffer_pool_read_requests / (Innodb_buffer_pool_read_requests + Innodb_buffer_pool_read_ahead + Innodb_buffer_pool_reads ) <p>
平均每次读取字节数 = Innodb_data_read / Innodb_data_reads

## 磁盘对数据库性能的重要影响
#### 传统机械硬盘
- 寻道时间。当前寻道时间大约在3ms
- 转速。可达15000rpm
- 可顺序访问也可随机访问，顺序访问速度远高于随机访问，因此要充分利用顺序读写
- 可以做RAID
#### 固态硬盘
- 企业级一般使用固态硬盘，并且并联多块闪存以提高吞吐量
- 固态硬盘不需要移动磁头来旋转和定位，提供一致的随机访问时间
- 闪存中的数据不可更新，只能在耗时的擦除操作后通过扇区覆盖重写
- 擦除不能在扇区上完成，只能在擦除块上完成，通常大小128kb
- 闪存读写性能不对称，读取远大于写入
- 访问延时一般小于0.1ms
![image](pic/innodb%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%981.png)
- innodb_io_capacity 可以增加该值
- 修改innodb源码禁用预读、邻接页特性
## 合理设置RAID
raid作用：增加数据集成度；增加容错；增加容量和处理量

raid | 说明
---|---
RAID0 | 将多个磁盘合并成大的磁盘，并行IO，速度最快<p>无冗余，可靠性最低，一块盘损坏则丢失全部数据<p>受限于IO瓶颈等因素，RAID效能边际递减<p>
RAID1 | N个磁盘相互镜像，可靠性最高，读取性能好，但写入性能低，磁盘利用率最低
RAID5 | 
RAID10 和 RAID01 | 
RAID50 | 

#### RAID write through 和write back
- write back是指raid可以将数据放入到自身缓存中，并安排到后面执行
- RAID卡提供电池备份单元BBU（battery backup unit），防止断电造成的数据丢失
## 操作系统的选择
- 考虑稳定性，而非新特性
- solaris高可靠性高性能，zfs很适合mysql，可以考虑开源solaris
- windows要注意大小写
## 不同文件系统对性能的影响
- 
## 选择合适的基准性能测试工具
#### sysbench
#### mysql-tpcc