## CPU使用率
- linux是多任务分时操作系统，将CPU时间划分为很短的时间片，通过调度器轮流给任务使用
- linux定义了节拍率（HZ），触发中断，并使用全局变量Jiffies记录开机依赖的节拍数。每一次时间终端，jiffies加1
```
# grep 'CONFIG_HZ=' /boot/config-$(uname -r)
CONFIG_HZ=1000

```
systemtap tutorial（==systemtap用法参看systemtap一节==）有个比较好玩的实验，也可以确定CONFIG_HZ的大小
```
global count_jiffies, count_ms
probe timer.jiffies(100) { count_jiffies ++ }
probe timer.ms(100) { count_ms ++ }
probe timer.ms(543210)
{
    hz=(1000*count_jiffies) / count_ms
    printf ("jiffies:ms ratio %d:%d => CONFIG_HZ=%d\n",
    count_jiffies, count_ms, hz)
    exit ()
}

```

- 用户空间不能直接访问节拍率，因此内核还提供了用户空间节拍率USER_HZ。用户空间程序不关心内核HZ，而只看到USER_HZ，总固定为100
```
# getconf CLK_TCK
100
# cat /proc/sys/net/ipv4/neigh/eth0/locktime 
100
```
- times 系统调用使用的时间单位是USER_HZ决定的（C代码）
```
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/times.h>
#include<time.h>

int main()
{
    int user_ticks_per_second ;
    user_ticks_per_second = (int)sysconf(_SC_CLK_TCK);
    if(user_ticks_per_second == -1)
    {
        fprintf(stderr,"failed to get ticks per second by sysconf\n");
        return -1;
    }

    printf("The Number of USER ticks per second is %d\n",user_ticks_per_second);

    return 0;
}


```

```
# gcc sys_times.c -o sys_times
# ./sys_times 
The Number of USER ticks per second is 100
```

## 查看CPU使用率
#### top（默认刷新时间3秒）

```
# top
top - 11:44:37 up 196 days,  9:28,  6 users,  load average: 0.05, 0.05, 0.05
Tasks:  98 total,   1 running,  97 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.8 us,  0.8 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8009696 total,   273352 free,  3633756 used,  4102588 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  4048408 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                           
24412 root      10 -10  129380  12788   9400 S   2.3  0.2  93:59.70 AliYunDun                                                                         
 2329 java      20   0 4839968 773752   8848 S   1.0  9.7 890:30.01 java                                                                              
13766 mysql     20   0 6158648 798924  16944 S   0.7 10.0 119:52.87 mysqld                                                                            
  722 redis     20   0  142952   5336    984 S   0.3  0.1 256:44.95 redis-server                                                                      
 2451 java      20   0 4790492 979.5m   8628 S   0.3 12.5 256:00.64 java                                                                              
    1 root      20   0  190884   3292   2048 S   0.0  0.0  12:37.36 systemd                                                                           
    2 root      20   0       0      0      0 S   0.0  0.0   0:03.25 kthreadd                                                                          
    3 root      20   0       0      0      0 S   0.0  0.0   0:02.40 ksoftirqd/0        
```
- 按1可以切换到具体每个CPU的使用率
- top不区分内核和用户态的cpu使用
#### ps
要选定特定的参数

```
# ps au
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       506  0.0  0.0 110088   852 tty1     Ss+  Feb16   0:00 /sbin/agetty --noclear tty1 linux
root       507  0.0  0.0 110088   868 ttyS0    Ss+  Feb16   0:00 /sbin/agetty --keep-baud 115200 38400 9600 ttyS0 vt220
root       835  0.0  0.0 115568  2320 pts/2    Ss+  Aug29   0:01 -bash
root      5275  0.0  0.0 115568  2160 pts/1    Ss   Aug21   0:00 -bash
root      8692  0.0  0.0 115568  2200 pts/3    Ss+  Aug30   0:00 -bash
root     15023  0.0  0.0 115568  2180 pts/5    Ss   10:39   0:00 -bash
```

#### pidstat
```
# pidstat 1 5
Linux 3.10.0-862.11.6.el7.x86_64 (izm5edbv563hlvcbf71ophz) 	08/31/2019 	_x86_64_	(2 CPU)

11:49:04 AM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
11:49:05 AM  1001      2329    1.00    0.00    0.00    1.00     1  java
11:49:05 AM  1001      2451    1.00    0.00    0.00    1.00     0  java
11:49:05 AM     0     24412    1.00    2.00    0.00    3.00     1  AliYunDun
```

## CPU过高如何排查
- gdb不适合性能分析的早期应用（中断进程运行，且比较细节）
- 使用linux2.6后内置的分析工具perf
#### perf
######  perf top：实时显示占用CPU时钟最多的函数或者指令
```
# perf top

Samples: 1K of event 'cpu-clock', Event count (approx.): 311980101                                                                                     
Overhead  Shared Object            Symbol                                                                                                              
  15.40%  [kernel]                 [k] finish_task_switch
   9.43%  [kernel]                 [k] __do_softirq
   6.03%  [kernel]                 [k] _raw_spin_unlock_irqrestore
   4.87%  [kernel]                 [k] run_timer_softirq
   4.67%  [kernel]                 [k] tick_nohz_idle_exit
   3.15%  libc-2.17.so             [.] __GI___libc_nanosleep
   2.90%  [kernel]                 [k] tick_nohz_idle_enter
   2.46%  libpthread-2.17.so       [.] pthread_cond_timedwait@@GLIBC_2.3.2
   1.16%  libpthread-2.17.so       [.] pthread_mutex_unlock
   1.13%  [kernel]                 [k] system_call_after_swapgs
```
- 第一行：包含三个数据，采样数（samples），事件类型（event）和事件总数量（event count）
- 采样数比较重要，如果过少则没有价值
- 表格样式的数据：
1. verhead，表示该符号的性能事件在所有采样中的比例
2. shared，是指该函数或者指令所在的动态共享对象（Dynamic shared object），如内核、进程名、动态链接库名、内核模块名等
3. object，是动态动向对象的类型。比如[.]表示用户控件的可执行程序，或者动态链接库，[k]表示内核空间
4. 最后一列symbol是符号名，也就是函数名。如果函数名位置，用十六进制的地址表示

```
# perf top -g -p 13766

Samples: 1K of event 'cpu-clock', Event count (approx.): 130372124                                                                                     
  Children      Self  Shared Object        Symbol                                                                                                     
-   63.55%     2.31%  mysqld               [.] log_checkpointer                                                                                       
   - 12.01% log_checkpointer                                                                                                                          
      - 15.43% os_event::wait_time_low                                                                                                                
         - 8.98% pthread_cond_timedwait@@GLIBC_2.3.2                                                                                                  
            - 3.70% system_call_fastpath                                                                                                              
                 4.18% sys_futex                                                                                                                      
           2.51% os_event::timed_wait                                                                                                                 
           1.55% ut_usectime                                                                                                                          
        2.18% pthread_mutex_unlock                                                                                                                    
        1.71% pthread_mutex_unlock@plt                    
```
使用上下键选择，使用enter键展开
###### perf record 和perf report
- perf top不保存数据，无法离线或者后续分析
- 使用perf record 和perf report
```
# perf record
^C[ perf record: Woken up 8 times to write data ]

# perf report
Samples: 40K of event 'cpu-clock', Event count (approx.): 10052750000                                                                                  
Overhead  Command          Shared Object                           Symbol                                                                              
  97.04%  swapper          [kernel.kallsyms]                       [k] native_safe_halt
   0.17%  swapper          [kernel.kallsyms]                       [k] finish_task_switch
   0.14%  swapper          [kernel.kallsyms]                       [k] __do_softirq
   0.13%  AliYunDun        [kernel.kallsyms]                       [k] finish_task_switch
   0.13%  swapper          [kernel.kallsyms]                       [k] tick_nohz_idle_exit
   0.12%  swapper          [kernel.kallsyms]                       [k] tick_nohz_idle_enter
   0.11%  swapper          [kernel.kallsyms]                       [k] _raw_spin_unlock_irqrestore
   0.10%  swapper          [kernel.kallsyms]                       [k] run_timer_softirq
   0.08%  java             [kernel.kallsyms]                       [k] finish_task_switch
   0.07%  AliYunDun        libc-2.17.so                            [.] __GI___libc_nanosleep
   0.04%  java             libpthread-2.17.so                      [.] pthread_cond_timedwait@@GLIBC_2.3.2   0.04%  mysqld           [kernel.kallsyms]                       [k] finish_task_switch
```
###### perf stat
stat命令只会统计事件发生次数
```
# perf stat
^C
 Performance counter stats for 'system wide':

       7871.889274      cpu-clock (msec)          #    2.000 CPUs utilized          
            13,279      context-switches          #    0.002 M/sec                  
             1,034      cpu-migrations            #    0.131 K/sec                  
                 7      page-faults               #    0.001 K/sec                  
   <not supported>      cycles                                                      
   <not supported>      instructions                                                
   <not supported>      branches                                                    
   <not supported>      branch-misses                                               

       3.936045168 seconds time elapsed
```

## 大量CPU被占用却找不到对应的进程
可能原因：
1. 进程不停崩溃重启，比如配置错误、段错误等，被系统拉起
2. 进程都是短时进程，运行很短之后就结束
排查方法：
perf record -g
perf report
建议使用execsnoop


## 如何分析CPU性能问题
#### cpu相关的性能指标
![CPU性能指标清单](B8002C1CA9D048B69C1F15D2B53A2298)
- CPU 缓存的速度介于 CPU 和内存之间，缓存的是热点的内存数据。根据不断增长的热点数据，这些缓存按照大小不同分为 L1、L2、L3 等三级缓存，其中 L1 和 L2 常用在单核中， L3 则用在多核中
- 从 L1 到 L3，三级缓存的大小依次增大，相应的，性能依次降低（当然比内存还是好得多）。而它们的命中率，衡量的是 CPU 缓存的复用情况，命中率越高，则表示性能越好。

#### 各种指标的对应命令
![image](D112D05BCFC7443EB133E67C902778D6)

## 各种命令可以查看的指标
![image](4ACEFBEB8B89407C9147F4211E5C700D)

#### 常用方法
![image](C11DF49D0700431DA181D263F51A3F2E)


## CPU优化的思路