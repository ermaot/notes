https://blog.csdn.net/chongyang198999/article/details/48707735
https://www.cnblogs.com/xianbei/archive/2012/11/23/2783818.html
/proc/sys/vm下的各个文件说明
文件名|内容|默认值|说明
---|---|---|---
admin_reserve_kbytes|给有cap_sys_admin权限的用户保留的内存数量|min(free pages * 3%, 8MB)|这些内存是为了给管理员登录和杀死进程恢复系统提供足够的内存
block_dump|如果非0，启用块IO调试|0|
compact_memory|内存紧致程序|默认权限0200|当向该文件(/proc/sys/vm/compact_memory)写入1时，所有的内存域都会被压缩，使空闲的内存尽可能形成连续的内存块。启用CONFIG_COMPACTION才有效
dirty_background_bytes|内存回收阈值||当脏页所占的内存数量超过dirty_background_bytes时，内核的flusher线程开始回写脏页，dirty_background_bytes参数和 dirty_background_ratio参数只能指定其中一个
dirty_background_ratio|内存回收阈值||当脏页所占的百分比（相对于所有可用内存，即空闲内存页+可回收内存页）达到dirty_background_ratio时回收脏页
dirty_bytes|||当脏页所占的内存数量达到dirty_bytes时，执行磁盘写操作的进程自己开始回写脏数据，与dirty_ratio只能设置一个
dirty_ratio|||当脏页所占的百分比（相对于所有可用内存，即空闲内存页+可回收内存页）达到dirty_ratio时，执行磁盘写操作的进程会自己开始回写脏数据。所有可用内存不等于总的系统内存
dirty_expire_centisecs|||脏数据的过期时间，超过该时间后内核的flusher线程被唤醒时会将脏数据回写到磁盘上，单位是百分之一秒
dirty_writeback_centisecs|||内核的flusher线程会周期性地唤醒，然后将老的脏数据写到磁盘上。 dirty_writeback_centisecs定义了唤醒的间隔，单位是百分之一秒。如果设置为0，则禁止周期性地唤醒回写线程
drop_caches|清理page cache|||echo [1,2,3] >  /proc/sys/vm/drop_caches。如果1，释放pagecache；如果2，释放dentries和inodes缓存；如果3，全部释放。非破坏性操作，不释放脏页。要释放脏页要运行sync
hugepages_treat_as_movable|控制是否可以从ZONE_MOVABLE内存域中分配大页面||如果设置为非零，大页面可以从ZONE_MOVABLE内存域分配；如果没有指定kernelcore启动参数， hugepages_treat_as_movable参数则没有效果
lowmem_reserve_ratio|决定了内核保护这些低端内存域的强度||预留的内存值和lowmem_reserve_ratio数组中的值是倒数关系，如果值是256，则代表1/256，即为0.39%的zone内存大小
max_map_count|进程中内存映射区域的最大数量|65536|在调用malloc，直接调用mmap和mprotect和加载共享库时会产生 内存映射区域
memory_failure_early_kill|控制在某个内核无法处理的内存错误发生（由硬件检测到）时，如何来杀死进程|||