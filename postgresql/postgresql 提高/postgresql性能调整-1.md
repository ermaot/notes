## 预读参数

预读是使用了局部性原理，认为“当前读到的数据，相邻的数据也很可能会读到”，这样预读一些数据，可以将随机读取转化为顺序读取，提高IO效率。

查看预读参数

```
# blockdev --getra /dev/vda1
8192
```

一些机器默认为256，本机为8192.对于一些新的机器，可以设置为16384.不过该值没有特别的设置方法，需要在具体环境中多次测试

设置预读参数

```
# /sbin/blockdev --setra 16384 /dev/vda1
# /sbin/blockdev --getra /dev/vda1
```

或者使用下面的方法

```
# cat  /sys/block/vda/queue/read_ahead_kb 
8192
# echo 16384 >   /sys/block/vda/queue/read_ahead_kb
# cat  /sys/block/vda/queue/read_ahead_kb 
16384
```



## swap

可以使用swapoff然后swapon来清理swap的内容，让swap“干净”

## 透明大页

Linux下的大页分为两种类型：标准大页（Huge Pages）和透明大页（Transparent Huge Pages）。

在 Linux 操作系统上运行内存需求量较大的应用程序时，由于其采用的默认页面大小为 4KB，因而将会产生较多 TLB Miss 和缺页中断，从而大大影响应用程序的性能。当操作系统以 2MB 甚至更大作为分页的单位时，将会大大减少 TLB Miss 和缺页中断的数量，显著提高应用程序的性能。

为了能以最小的代价实现大页面支持，Linux 操作系统采用了基于 hugetlbfs 特殊文件系统 2M 字节大页面支持。这种采用特殊文件系统形式支持大页面的方式，使得应用程序可以根据需要灵活地选择虚存页面大小，而不会被强制使用 2MB 大页面。

**标准大页管理是预分配的方式，而透明大页管理则是动态分配的方式**。

Oracle 建议禁用透明大页（Transparent Huge Pages），而使用标准大页

```
# cat /sys/kernel/mm/transparent_hugepage/enabled 
[always] madvise never

# echo never > /sys/kernel/mm/transparent_hugepage/enabled

# cat /sys/kernel/mm/transparent_hugepage/enabled 
always madvise [never]
```

或者修改grub2.conf

```
linux16 /boot/vmlinuz-5.2.11-1.el7.elrepo.x86_64 root=UUID=eb448abb-3012-4d8d-bcde-94434d586a31 ro crashkernel=auto rhgb quiet net.ifnames=0 console=tty0     console=ttyS0,115200n8  transparent_hugepage=never
```

## NUMA

NUMA架构会优先在请求的线程所在的CPU的local内存上分配，如果local内存不够则优先淘汰local内存中的无用页面，这可能会导致每个CPU上的内存分配不均，对数据库系统不友好。建议关闭。

```
# numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1
node 0 size: 8191 MB
node 0 free: 230 MB
node distances:
node   0 
  0:  10 
  
  # numastat 
                           node0
numa_hit               378516017
numa_miss                      0
numa_foreign                   0
interleave_hit             15232
local_node             378516017
other_node                     0
```



可以在grub2.conf中关闭，加numa=off即可

## 数据库参数

#### shared_buffers

postgresql数据库启动时，就会分配所有的共享内存，即使没有请求，共享内存页不会变化，固定为shared_buffers。当postgresql接收客户端请求时，服务进程首先在shared_buffers查找所需的数据。

默认值为128M，建议为机器内存的20~30%

```
# show shared_buffers;
 shared_buffers 
----------------
 128MB
(1 row)
```

#### work_mem

work_mem限制服务进程排序或者hash时的内存，针对每个服务进程设置，不宜设为太大，默认为4M；当work_mem不足时会借助磁盘临时文件，导致性能下降。

postgresql在排序的时候有Top-N heapsort 、quick sort、external merge几种方法，如果查询计划中发现使用了external merge说明需要适当增加work_mem

#### random_page_cost

随机访问磁盘块的代价估计，默认为4

```
# show random_page_cost;
 random_page_cost 
------------------
 4
(1 row)
# show seq_page_cost;
 seq_page_cost 
---------------
 1
(1 row)
```

现在越来越多使用固态硬盘，所以random_page_cost比seq_page_cost稍大即可。

