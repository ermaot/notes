## 案例1
#### 现象
```
# 按 1 切换到每个 CPU 的使用情况 
$ top 
top - 14:43:43 up 1 day,  1:39,  2 users,  load average: 2.48, 1.09, 0.63 
Tasks: 130 total,   2 running,  74 sleeping,   0 stopped,   0 zombie 
%Cpu0  :  0.7 us,  6.0 sy,  0.0 ni,  0.7 id, 92.7 wa,  0.0 hi,  0.0 si,  0.0 st 
%Cpu1  :  0.0 us,  0.3 sy,  0.0 ni, 92.3 id,  7.3 wa,  0.0 hi,  0.0 si,  0.0 st 
KiB Mem :  8169308 total,   747684 free,   741336 used,  6680288 buff/cache 
KiB Swap:        0 total,        0 free,        0 used.  7113124 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND 
18940 root      20   0  656108 355740   5236 R   6.3  4.4   0:12.56 python 
1312 root      20   0  236532  24116   9648 S   0.3  0.3   9:29.80 python3 

```
1. CPU占用都不高，其中一个cpu0较高，占用为6%
2. 内存8G，剩余730M，而buff/cache占用达6G多
3. CPU0的iowait为92.7%
判断：CPU使用率中的IOwait是瓶颈

#### iostat
```
# -d 表示显示 I/O 性能指标，-x 表示显示扩展统计（即所有 I/O 指标） 
$ iostat -x -d 1 
Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util 
loop0            0.00    0.00      0.00      0.00     0.00     0.00   0.00   0.00    0.00    0.00   0.00     0.00     0.00   0.00   0.00 
sdb              0.00    0.00      0.00      0.00     0.00     0.00   0.00   0.00    0.00    0.00   0.00     0.00     0.00   0.00   0.00 
sda              0.00   64.00      0.00  32768.00     0.00     0.00   0.00   0.00    0.00 7270.44 1102.18     0.00   512.00  15.50  99.20

```
1. sda的IO使用率达到99%，接近饱和
2. 每秒写磁盘请求数64，写大小32M，写请求响应时间7秒，请求队列长度达到1100

#### pidstat
```
pidstat -d 1 

15:08:35      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command 
15:08:36        0     18940      0.00  45816.00      0.00      96  python 

15:08:36      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command 
15:08:37        0       354      0.00      0.00      0.00     350  jbd2/sda1-8 
15:08:37        0     18940      0.00  46000.00      0.00      96  python 
15:08:37        0     20065      0.00      0.00      0.00    1503  kworker/u4:2 

```
可见python进程（进程号18940）的写比较大


#### strace
```
strace -p 18940 
strace: Process 18940 attached 
...
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0f7aee9000 
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0f682e8000 
write(3, "2018-12-05 15:23:01,709 - __main"..., 314572844 
) = 314572844 
munmap(0x7f0f682e8000, 314576896)       = 0 
write(3, "\n", 1)                       = 1 
munmap(0x7f0f7aee9000, 314576896)       = 0 
close(3)                                = 0 
stat("/tmp/logtest.txt.1", {st_mode=S_IFREG|0644, st_size=943718535, ...}) = 0 

```
- write() 系统调用上，我们可以看到，进程向文件描述符编号为 3 的文件中，写入了300MB的数据。它应该是我们要找的文件
- stat() 调用，你可以看到，它正在获取 /tmp/logtest.txt.1 的状态

#### lsof
```
lsof -p 18940 
COMMAND   PID USER   FD   TYPE DEVICE  SIZE/OFF    NODE NAME 
python  18940 root  cwd    DIR   0,50      4096 1549389 / 
python  18940 root  rtd    DIR   0,50      4096 1549389 / 
… 
python  18940 root    2u   CHR  136,0       0t0       3 /dev/pts/0 
python  18940 root    3w   REG    8,1 117944320     303 /tmp/logtest.txt 

```


## 案例2 swap
- 创建swap文件
```
# 创建 Swap 文件
$ fallocate -l 8G /mnt/swapfile
# 修改权限只有根用户可以访问
$ chmod 600 /mnt/swapfile
# 配置 Swap 文件
$ mkswap /mnt/swapfile
# 开启 Swap
$ swapon /mnt/swapfile

```
- sar输出
```
# 间隔 1 秒输出一组数据
# -r 表示显示内存使用情况，-S 表示显示 Swap 使用情况
$ sar -r -S 1
04:39:56    kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
04:39:57      6249676   6839824   1919632     23.50    740512     67316   1691736     10.22    815156    841868         4

04:39:56    kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
04:39:57      8388604         0      0.00         0      0.00

04:39:57    kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
04:39:58      6184472   6807064   1984836     24.30    772768     67380   1691736     10.22    847932    874224        20

04:39:57    kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
04:39:58      8388604         0      0.00         0      0.00

…


04:44:06    kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
04:44:07       152780   6525716   8016528     98.13   6530440     51316   1691736     10.22    867124   6869332         0

04:44:06    kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
04:44:07      8384508      4096      0.05        52      1.27

```
1. kbcommit，表示当前系统负载需要的内存。它实际上是为了保证系统内存不溢出，对需要内存的估计值。%commit，就是这个值相对总内存的百分比。
2. kbactive，表示活跃内存，也就是最近使用过的内存，一般不会被系统回收。
3. kbinact，表示非活跃内存，也就是不常访问的内存，有可能会被系统回收。
- 通过 /proc/zoneinfo  ，观察剩余内存、内存阈值以及匿名页和文件页的活跃情况。
```
# -d 表示高亮变化的字段
# -A 表示仅显示 Normal 行以及之后的 15 行输出
$ watch -d grep -A 15 'Normal' /proc/zoneinfo
Node 0, zone   Normal
  pages free     21328
        min      14896
        low      18620
        high     22344
        spanned  1835008
        present  1835008
        managed  1796710
        protection: (0, 0, 0, 0, 0)
      nr_free_pages 21328
      nr_zone_inactive_anon 79776
      nr_zone_active_anon 206854
      nr_zone_inactive_file 918561
      nr_zone_active_file 496695
      nr_zone_unevictable 2251
      nr_zone_write_pending 0

```
- 打开和关闭swap可以清理
```
swapoff -a && swapon -a 

```
- 查看进程swap大小
这里我还是推荐 proc 文件系统，用来查看进程 Swap 换出的虚拟内存大小，它保存在 /proc/pid/status 中的 VmSwap 中（推荐你执行 man proc 来查询其他字段的含义）。
```
# 按 VmSwap 使用量对进程排序，输出进程名称、进程 ID 以及 SWAP 用量
$ for file in /proc/*/status ; do awk '/VmSwap|Name|^Pid/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 3 -n -r | head
dockerd 2226 10728 kB
docker-containe 2251 8516 kB
snapd 936 4020 kB
networkd-dispat 911 836 kB
polkitd 1004 44 kB

```
