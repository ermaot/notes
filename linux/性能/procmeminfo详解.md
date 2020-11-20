本文改自http://www.dataguru.cn/article-10187-1.html

参考了https://www.kernel.org/doc/Documentation/filesystems/proc.rst

/proc/meminfo是了解Linux系统内存使用状况的主要接口，我们最常用的”free”、”vmstat”等命令就是通过它获取数据的 

## 一、查看/proc/meminfo

```
# cat /proc/meminfo 
MemTotal:         840948 kB
MemFree:           75644 kB
MemAvailable:     127964 kB
Buffers:           16168 kB
Cached:           168636 kB
SwapCached:            0 kB
Active:           582204 kB
Inactive:          65512 kB
Active(anon):     496504 kB
Inactive(anon):      316 kB
Active(file):      85700 kB
Inactive(file):    65196 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:               180 kB
Writeback:             0 kB
AnonPages:        462952 kB
Mapped:            67532 kB
Shmem:             33908 kB
Slab:              74616 kB
SReclaimable:      31540 kB
SUnreclaim:        43076 kB
KernelStack:        3292 kB
PageTables:        11764 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      420472 kB
Committed_AS:    1936712 kB
VmallocTotal:   34359738367 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
HardwareCorrupted:     0 kB
AnonHugePages:    161792 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:      264040 kB
DirectMap2M:      784384 kB
DirectMap1G:           0 kB

```



## 二、各项说明
项|说明
---|---
MemTotal|系统从加电开始到引导完成，firmware/BIOS要保留一些内存，kernel本身要占用一些内存，最后剩下可供kernel支配的内存就是MemTotal。这个值在系统运行期间一般是固定不变的。
MemFree|表示系统尚未使用的内存。(MemTotal-MemFree)就是已被用掉的内存
MemAvailable|系统中有些内存虽然已被使用但是可以回收的，比如cache/buffer、slab都有一部分可以回收，所以这部分可回收的内存加上MemFree才是系统可用的内存。该值是通过算法估算的，不准确
内存黑洞|Linux kernel并没有滴水不漏地统计所有的内存分配，kernel动态分配的内存中就有一部分没有计入/proc/meminfo中
Buffers|相对临时的磁盘块数据（不会特别巨大）
Cached|内存中文件读取的缓存，不包括SwapCached
SwapCached|内存一旦被swap out再swap回来，它仍然存在于swap file中，不需要再被换出。这样节省IO
Active|Active(anon) + Active(file)
Inactive|Inactive(anon) + Inactive(file)
Active(anon)|LRU_ACTIVE_ANON
Inactive(anon)|LRU_INACTIVE_ANON
Active(file)|LRU_ACTIVE_FILE
Inactive(file)|LRU_INACTIVE_FILE
Unevictable|LRU_UNEVICTABLE
Mlocked|
SwapTotal|全部的swap分区数据
SwapFree|空闲的swap
Dirty| 等待写入到磁盘的内存                                         
Writeback|正在写入磁盘的内存
AnonPages|
Mapped|files which have been mmaped, such as libraries
Shmem|Total memory used by shared memory (shmem) and tmpfs
Slab|SReclaimable + SUnreclaim
SReclaimable|slab中可回收的部分，调用kmem_getpages()时加上SLAB_RECLAIM_ACCOUNT标记
SUnreclaim|slab中不可回收的部分
KernelStack|每一个用户线程都会分配一个kernel stack（内核栈），但用户不能直接访问
PageTables|amount of memory dedicated to the lowest level of page tables
NFS_Unstable|一直为0。Previous counted pages which had been written to  the server, but has not been committed to stable storage
Bounce|有些老设备只能访问低端内存，比如16M以下的内存，当应用程序发出一个I/O 请求，DMA的目的地址却是高端内存时（比如在16M以上），内核将在低端内存中分配一个临时buffer作为跳转，把位于高端内存的缓存数据复制到此处。这种额外的数据拷贝被称为”bounce buffering”，会降低I/O 性能
WritebackTmp|Memory used by FUSE for temporary writeback buffers
CommitLimit|                                                              
Committed_AS|
VmallocTotal|total size of vmalloc memory area
VmallocUsed|amount of vmalloc area which is used
VmallocChunk|largest contiguous block of vmalloc area which is free
HardwareCorrupted|当系统检测到内存的硬件故障时，会把有问题的页面删除掉，不再使用，/proc/meminfo中的HardwareCorrupted统计了删除掉的内存页的总大小
AnonHugePages|
ShmemHugePages|Memory used by shared memory (shmem) and tmpfs allocated with huge pages
ShmemPmdMapped|Shared memory mapped into userspace with huge pages
HugePages_Total|
HugePages_Free|
HugePages_Rsvd|
HugePages_Surp|
Hugepagesize|
Hugetlb|
DirectMap4k|
DirectMap2M|
DirectMap1G|


## 三、一些其他的解释
#### 内存分配方法
Kernel的动态内存分配通过以下几种接口：

1. alloc_pages/__get_free_page: 以页为单位分配
2. vmalloc: 以字节为单位分配虚拟地址连续的内存块
3. slab allocator
4. kmalloc: 以字节为单位分配物理地址连续的内存块，它是以slab为基础的，使用slab层的general caches — 大小为2^n，名称是kmalloc-32、kmalloc-64等（在老kernel上的名称是size-32、size-64等）。

#### 哪些会被统计到

1. 通过slab层分配的内存会被较精确统计，可以参见/proc/meminfo中的slab/SReclaimable/SUnreclaim
2. 通过vmalloc分配的内存也有统计，参见/proc/meminfo中的VmallocUsed 和 /proc/vmallocinfo
3. alloc_pages分配的内存不会自动统计，除非调用alloc_pages的内核模块或驱动程序主动进行统计，否则我们只能看到free memory减少了，但从/proc/meminfo中看不出它们具体用到哪里去了

## 四、内存去向

#### 1. 内核

内核所用内存的静态部分，比如内核代码、页描述符等数据在引导阶段就分配掉了，并不计入MemTotal里，而是算作Reserved(在dmesg中能看到)。

```
# dmesg -T | grep -i reserv
[Wed Sep 16 10:26:22 2020] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
[Wed Sep 16 10:26:22 2020] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[Wed Sep 16 10:26:22 2020] BIOS-e820: [mem 0x000000003ffda000-0x000000003fffffff] reserved
[Wed Sep 16 10:26:22 2020] BIOS-e820: [mem 0x00000000feffc000-0x00000000feffffff] reserved
[Wed Sep 16 10:26:22 2020] BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
[Wed Sep 16 10:26:22 2020] e820: update [mem 0x00000000-0x00000fff] usable ==> reserved
```

内核所用内存的动态部分，是通过上文提到的几个接口申请的，其中通过alloc_pages申请的内存有可能未纳入统计，就像黑洞一样。

内核又会又以下方面

##### 1.1 SLAB

通过slab分配的内存被统计在以下三个值中：

1. SReclaimable: slab中可回收的部分。调用kmem_getpages()时加上SLAB_RECLAIM_ACCOUNT标记，表明是可回收的，计入SReclaimable，否则计入SUnreclaim。
2. SUnreclaim: slab中不可回收的部分。
3. Slab: slab中所有的内存，等于以上两者之和。

##### 1.2 VmallocUsed

通过vmalloc分配的内存都统计在/proc/meminfo的 VmallocUsed 值中，但是要注意这个值不止包括了分配的物理内存，还统计了VM_IOREMAP、VM_MAP等操作的值，譬如VM_IOREMAP是把IO地址映射到内核空间、并未消耗物理内存，所以我们要把它们排除在外。

```
# cat /proc/vmallocinfo  | grep vmalloc
0xffffa0ef80002000-0xffffa0ef80004000    8192 bpf_prog_alloc+0x47/0xd0 pages=1 vmalloc N0=1
0xffffa0ef80008000-0xffffa0ef80109000 1052672 alloc_large_system_hash+0x19b/0x25e pages=256 vmalloc N0=256
0xffffa0ef80109000-0xffffa0ef8018a000  528384 alloc_large_system_hash+0x19b/0x25e pages=128 vmalloc N0=128
0xffffa0ef8018a000-0xffffa0ef8018f000   20480 alloc_large_system_hash+0x19b/0x25e pages=4 vmalloc N0=4
0xffffa0ef8018f000-0xffffa0ef80194000   20480 alloc_large_system_hash+0x19b/0x25e pages=4 vmalloc N0=4
0xffffa0ef80194000-0xffffa0ef80199000   20480 _do_fork+0xe2/0x390 pages=4 vmalloc N0=4
0xffffa0ef8019c000-0xffffa0ef801a1000   20480 _do_fork+0xe2/0x390 pages=4 vmalloc N0=4
0xffffa0ef801a4000-0xffffa0ef801a9000   20480 _do_fork+0xe2/0x390 pages=4 vmalloc N0=4
0xffffa0ef801a9000-0xffffa0ef801ac000   12288 alloc_large_system_hash+0x19b/0x25e pages=2 vmalloc N0=2
0xffffa0ef801ac000-0xffffa0ef801b1000   20480 _do_fork+0xe2/0x390 pages=4 vmalloc N0=4
0xffffa0ef801b4000-0xffffa0ef801b9000   20480 _do_fork+0xe2/0x390 pages=4 vmalloc N0=4
0xffffa0ef801bc000-0xffffa0ef801c1000   20480 _do_fork+0xe2/0x390 pages=4 vmalloc N0=4
```

/proc/vmallocinfo中能看到vmalloc来自哪个调用者(caller)

##### 1.3 kernel modules (内核模块)

系统已经加载的内核模块可以用 lsmod 命令查看，注意第二列就是内核模块所占内存的大小，通过它可以统计内核模块所占用的内存大小，但这并不准，因为”lsmod”列出的是[init_size+core_size]，而实际给kernel module分配的内存是以page为单位的，不足 1 page的部分也会得到整个page，此外每个module还会分到一页额外的guard page。

```
# lsmod | less
Module                  Size  Used by
binfmt_misc            20480  1
nfnetlink              16384  0
loop                   32768  0
nls_utf8               16384  0
isofs                  45056  0
crct10dif_pclmul       16384  0
crc32_pclmul           16384  0
sg                     40960  0
ghash_clmulni_intel    16384  0
joydev                 24576  0
pcspkr                 16384  0
virtio_balloon         20480  0
```

lsmod的信息来自/proc/modules，它显示的size包括init_size和core_size。源代码在kernel/module.c中

![image-20201120135013390](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201120135013390.png)

##### 1.4 HardwareCorrupted

当系统检测到内存的硬件故障时，会把有问题的页面删除掉，不再使用，/proc/meminfo中的HardwareCorrupted统计了删除掉的内存页的总大小。相应的代码参见 mm/memory-failure.c: memory_failure ()

##### 1.5 PageTables

Page Table用于将内存的虚拟地址翻译成物理地址，随着内存地址分配得越来越多，Page Table会增大，/proc/meminfo中的PageTables统计了Page Table所占用的内存大小

##### 1.6 KernelStack

每一个用户线程都会分配一个kernel stack（内核栈），内核栈虽然属于线程，但用户态的代码不能访问，只有通过系统调用(syscall)、自陷(trap)或异常(exception)进入内核态的时候才会用到，也就是说内核栈是给kernel code使用的

##### 1.7 Buffers

Buffers统计的是直接访问块设备时的缓冲区的总大小，有时候对文件系统元数据的操作也会用到buffers。这部分内存不好直接对应到某个用户进程，应该算作kernel占用。

##### 1.8 Bounce

有些老设备只能访问低端内存，比如16M以下的内存，当应用程序发出一个I/O 请求，DMA的目的地址却是高端内存时（比如在16M以上），内核将在低端内存中分配一个临时buffer作为跳转，把位于高端内存的缓存数据复制到此处。这种额外的数据拷贝被称为”bounce buffering”，会降低I/O 性能。大量分配的bounce buffers 也会占用额外的内存。

#### 2.用户进程

/proc/meminfo统计的是系统全局的内存使用状况，单个进程的情况要看/proc/下的smaps等等

```
# cat /proc/2470278/smaps | more
55f933856000-55f933920000 r-xp 00000000 fd:01 282048                     /usr/sbin/sshd
Size:                808 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Rss:                 680 kB
Pss:                 135 kB
Shared_Clean:        680 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:         0 kB
Referenced:          680 kB
Anonymous:             0 kB
LazyFree:              0 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
Locked:                0 kB
………………
```

项|含义
---|---
Size|表示该映射区域在虚拟内存空间中的大小。
Rss|表示该映射区域当前在物理内存中占用了多少空间　　　　　　
Shared_Clean|和其他进程共享的未被改写的page的大小
Shared_Dirty| 和其他进程共享的被改写的page的大小
Private_Clean|未被改写的私有页面的大小。
Private_Dirty| 已被改写的私有页面的大小。
Swap|表示非mmap内存（也叫anonymous memory，比如malloc动态分配出来的内存）由于物理内存不足被swap到交换空间的大小。
Pss|该虚拟内存区域平摊计算后使用的物理内存大小(有些内存会和其他进程共享，例如mmap进来的)。比如该区域所映射的物理内存部分同时也被另一个进程映射了，且该部分物理内存的大小为1000KB，那么该进程分摊其中一半的内存，即Pss=500KB

##### 2.1 Hugepages

1. Hugepages在/proc/meminfo中是被独立统计的，与其它统计项不重叠，既不计入进程的RSS/PSS中，又不计入LRU Active/Inactive，也不会计入cache/buffer。如果进程使用了Hugepages，它的RSS/PSS不会增加
2. Transparent HugePages (THP)跟 Hugepages 搞混了，THP的统计值是/proc/meminfo中的”AnonHugePages”，在/proc/<pid>/smaps中也有单个进程的统计，这个统计值与进程的RSS/PSS是有重叠的，如果用户进程用到了THP，进程的RSS/PSS也会相应增加，这与Hugepages是不同的
3. HugePages_Total 对应内核参数 vm.nr_hugepages，也可以在运行中的系统上直接修改 /proc/sys/vm/nr_hugepages，修改的结果会立即影响空闲内存 MemFree的大小，因为HugePages在内核中独立管理，只要一经定义，无论是否被使用，都不再属于free memory。

```
# cat /proc/meminfo | grep Huge
AnonHugePages:    161792 kB
ShmemHugePages:        0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
```

可以看到Hugepagesize为2M

如果直接通过手动修改/proc/sys/vm/nr_hugepages

```

```

使用Hugepages有三种方式：(详见 https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt )

1. mount一个特殊的 hugetlbfs 文件系统，在上面创建文件，然后用mmap() 进行访问，如果要用 read() 访问也是可以的，但是 write() 不行。
2. 通过shmget/shmat也可以使用Hugepages，调用shmget申请共享内存时要加上 SHM_HUGETLB 标志。
3. 通过 mmap()，调用时指定MAP_HUGETLB 标志也可以使用Huagepages

##### 2.2 AnonHugePages

AnonHugePages统计的是Transparent HugePages (THP)，THP与Hugepages不是一回事，区别很大

##### 2.3 LRU

LRU是Kernel的页面回收算法(Page Frame Reclaiming)使用的数据结构，在 解读vmstat中的Active/Inactive memory 一文中有介绍。Page cache和所有用户进程的内存（kernel stack和huge pages除外）都在LRU lists上。
LRU lists包括如下几种，在/proc/meminfo中都有对应的统计值：

1. LRU_INACTIVE_ANON – 对应 Inactive(anon)
2. LRU_ACTIVE_ANON – 对应 Active(anon)
3. LRU_INACTIVE_FILE – 对应 Inactive(file)
4. LRU_ACTIVE_FILE – 对应 Active(file)
5. LRU_UNEVICTABLE – 对应 Unevictable

用户进程的内存页分为两种：

1. file-backed pages（与文件对应的内存页），比如进程的代码、映射的文件都是file-backed，。file-backed pages在内存不足的时候可以直接写回对应的硬盘文件里，称为page-out，不需要用到交换区(swap)；
2. anonymous pages（匿名页），进程的堆、栈都是不与文件相对应的、就属于匿名页。而anonymous pages在内存不足时就只能写到硬盘上的交换区(swap)里，称为swap-out。

##### 2.4 Shmem

/proc/meminfo中的Shmem统计的内容包括：

1. shared memory
2. tmpfs

此处所讲的shared memory又包括：

SysV shared memory [shmget etc.]

POSIX shared memory [shm_open etc.]

shared anonymous mmap [ mmap(…MAP_ANONYMOUS|MAP_SHARED…)]

因为shared memory在内核中都是基于tmpfs实现的，参见：https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt。也就是说它们被视为基于tmpfs文件系统的内存页，既然基于文件系统，就不算匿名页，所以不被计入/proc/meminfo中的AnonPages，而是被统计进了：Cached (i.e. page cache)和Mapped (当shmem被attached时候)

##### 2.5 AnonPages

如上所提

##### 2.6 Mapped

##### 2.7 Cached

Page Cache里包括所有file-backed pages，统计在/proc/meminfo的”Cached”中。

Cached不仅包括mapped，也包括unmapped的页面，当一个文件不再与进程关联之后，原来在page cache中的页面并不会立即回收，仍然被计入Cached，还留在LRU中，但是 Mapped 统计值会减小。

##### 2.8 SwapCached

我们说过，匿名页(anonymous pages)要用到交换区，而shared memory和tmpfs虽然未统计在AnonPages里，但它们背后没有硬盘文件，所以也是需要交换区的。

##### 2.9 Mlocked

“Mlocked”统计的是被mlock()系统调用锁定的内存大小。

##### DirectMap

/proc/meminfo中的DirectMap所统计的不是关于内存的使用，而是一个反映TLB效率的指标。”DirectMap4k”表示映射为4kB的内存数量， “DirectMap2M”表示映射为2MB的内存数量，以此类推



## 五、一些等式与不等式

#### 5.1 Dirty pages

/proc/meminfo 中有一个Dirty统计值，但是它未能包括系统中全部的dirty pages，应该再加上另外两项：NFS_Unstable 和 Writeback，NFS_Unstable是发给NFS server但尚未写入硬盘的缓存页，Writeback是正准备回写硬盘的缓存页

```
dirty pages = ( Dirty + NFS_Unstable + Writeback )
```

1. NFS_Unstable的内存被包含在Slab中，因为nfs request内存是调用kmem_cache_zalloc()申请的。
2. anonymous pages不属于dirty pages

#### 5.2 Active(file)+Inactive(file)不等于Mapped

LRU Active(anon)和Inactive(anon)中包含unmapped页面；Mapped中包含shared memory & tmpfs，这部分内存被计入了LRU Active(anon)或Inactive(anon)、而不在Active(file)和Inactive(file)中

#### 5.3 Active(file)+Inactive(file) != Cached

因为shared memory & tmpfs包含在Cached中，而不在Active(file)和Inactive(file)中。如果不考虑mlock添乱的话，一个更符合逻辑的等式是：

Active(file) + Inactive(file) + Shmem == Cached

#### 5.4 用户进程的内存主要有三种统计口径

##### 5.4.1 围绕LRU进行统计

【(Active + Inactive + Unevictable) + (HugePages_Total * Hugepagesize)】

##### 5.4.2 围绕Page Cache进行统计

当SwapCached为0的时候，用户进程的内存总计如下：

【(Cached + AnonPages) + (HugePages_Total * Hugepagesize)】

当SwapCached不为0的时候，以上公式不成立

##### 5.4.3 围绕RSS/PSS进行统计

把/proc/[1-9]*/smaps 中的 Pss 累加起来就是所有用户进程占用的内存，但是还没有包括Page Cache中unmapped部分、以及HugePages，所以公式如下：

ΣPss + (Cached – mapped) + (HugePages_Total * Hugepagesize)

#### 5.5 系统内存的使用情况可以用以下公式表示：

```
MemTotal = MemFree +【Slab+ VmallocUsed + PageTables + KernelStack + Buffers + HardwareCorrupted + Bounce + X】+【Active + Inactive + Unevictable + (HugePages_Total * Hugepagesize)】
```

```
MemTotal = MemFree +【Slab+ VmallocUsed + PageTables + KernelStack + Buffers + HardwareCorrupted + Bounce + X】+【Cached + AnonPages + (HugePages_Total * Hugepagesize)】
```

```
MemTotal = MemFree +【Slab+ VmallocUsed + PageTables + KernelStack + Buffers + HardwareCorrupted + Bounce + X】+【ΣPss + (Cached – mapped) + (HugePages_Total * Hugepagesize)】
```

##### 5.6 free命令的buff/cache

free命令的buff/cache计算方法

```
buff/cache = Buffers + Cached + SReclaimable
```

