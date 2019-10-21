参考链接：[专题：性能调优之工具---perf](https://blog.csdn.net/chichi123137/article/details/80139237)
## perf是什么
Perf 是内置于Linux 内核源码树中的性能剖析（profiling）工具。==它基于事件采样原理==，以==性能事件为基==础，支持针对==处理器相关性能指标==与==操作系统相关性能指标==的性能剖析。可用于==性能瓶颈的查找与热点代码的定位==。linux2.6及后续版本都自带该工具，几乎能够处理所有与性能相关的事件。

## 主要作用
Perf工具可用来==对软件进行优化==，包括
- 算法优化（空间复杂度、时间复杂度）和代码优化（提高执行速度、减少内存占用）。
- 还可以评估程序对硬件资源的使用情况，例如各级cache的访问次数，各级cache的丢失次数、流水线停顿周期、前端总线访问次数等。
- 也可以评估程序对操作系统资源的使用情况，系统调用次数、上下文切换次数、任务迁移次数等


## perf原理
perf基于性能事件的采集和分析，有两种方法
1. 基于硬件的采集方法，需要采用CPU的PMU（performance monitoring unit）部件，在特定条件下探测性能事件是否发生以及发生的次数
2. 基于软件的方法，需要将代码内置于kernel分布在各个功能模块中，统计和操作系统相关的性能事件

#### perf硬件实现原理
1. perf会通过系统调用sys_perf_event_open在内核中注册一个监测"cycles"事件的性能计数器
2. 内核根据perf提供的信息在PMU上初始化一个硬件性能计数器（performance monitoring counter），PMC随着CPU周期而自动累加
3. PMC溢出时，PMU触发一个PMI（performance monitoring interrupt）中断
4. 内核在PMI中断的处理函数中保存PMC的值，出发中断时的指令地址（register IP：instruction pointer），当前时间戳以及当前进程的PID，TID，comm等信息。这些信息统称为一个采样（sample），内核将收集到的sample放入用于跟用户空间通信的ring buffer
5. 用户空间的perf分析程序采用mmap机制从ring buffer读取采样并分析
6. pid，comm等信息可以找到对应的进程；根据指令地址与elf文件中的符号表可以查到触发PMI中断指令所在的函数（所以必须有符号表）

#### perf软件实现原理
原文链接：[Perf -- Linux下的系统性能调优工具，第 1 部分](https://www.ibm.com/developerworks/cn/linux/l-cn-perf1/)


## Tracepoint
- Tracepoint 是散落在内核源代码中的一些 hook，一旦使能，它们便可以在特定的代码被运行到时被触发，这一特性可以被各种 trace/debug 工具所使用。Perf 就是该特性的用户之一。

- 假如您想知道在应用程序运行期间，内核内存管理模块的行为，便可以利用潜伏在 slab 分配器中的 tracepoint。当内核运行到这些 tracepoint 时，便会通知 perf。

- Perf 将 tracepoint 产生的事件记录下来，生成报告，通过分析这些报告，调优人员便可以了解程序运行时期内核的种种细节，对性能症状作出更准确的诊断


## perf的使用
#### perf list
```
# perf list 
 List of pre-defined events (to be used in -e): 
 cpu-cycles OR cycles [Hardware event] 
 instructions [Hardware event] 
…
 cpu-clock [Software event] 
 task-clock [Software event] 
 context-switches OR cs [Software event] 
…
 ext4:ext4_allocate_inode [Tracepoint event] 
 kmem:kmalloc [Tracepoint event] 
 module:module_load [Tracepoint event] 
 workqueue:workqueue_execution [Tracepoint event] 
 sched:sched_{wakeup,switch} [Tracepoint event] 
 syscalls:sys_{enter,exit}_epoll_wait [Tracepoint event] 
…
```
perf支持的事件有3类：
1. Hardware Event 是由 PMU 硬件产生的事件，比如 cache 命中，当您需要了解程序对硬件特性的使用情况时，便需要对这些事件进行采样
2. Software Event 是内核软件产生的事件，比如进程切换，tick 数等
3. Tracepoint event 是内核中的静态 tracepoint 所触发的事件，这些 tracepoint 用来判断程序运行期间内核的行为细节，比如 slab 分配器的分配次数等

#### Perf stat
Perf stat 通过概括精简的方式提供被调试程序运行的整体情况和汇总数据

指标|说明
---|---
Task-clock-msecs|CPU利用率，该值高，说明程序的多数时间花费在 CPU 计算上而非 IO。
Context-switches|进程切换次数，记录了程序运行过程中发生了多少次进程切换，频繁的进程切换是应该避免的。
Cache-misses|程序运行过程中总体的 cache 利用情况，如果该值过高，说明程序的 cache 利用不好
CPU-migrations|表示进程 t1 运行过程中发生了多少次 CPU 迁移，即被调度器从一个 CPU 转移到另外一个 CPU 上运行。
Cycles|处理器时钟，一条机器指令可能需要多个 cycles，
Instructions|机器指令数目。
IPC|是 Instructions/Cycles的比值，该值越大越好，说明程序充分利用了处理器的特性。
Cache-references|cache 命中的次数
Cache-misses|cache 失效的次数。

```
//c代码样例test.c 
void longa() 
{ 
  int i,j; 
  for(i = 0; i < 1000000; i++) 
  j=i; //am I silly or crazy? I feel boring and desperate. 
} 
 
void foo2() 
{ 
  int i; 
  for(i=0 ; i < 10; i++) 
       longa(); 
} 
 
void foo1() 
{ 
  int i; 
  for(i = 0; i< 100; i++) 
     longa(); 
} 
 
int main(void) 
{ 
  foo1(); 
  foo2(); 
}
```

```
# perf stat ./test

 Performance counter stats for './test':

        284.391601      task-clock (msec)         #    0.999 CPUs utilized          
                 0      context-switches          #    0.000 K/sec                  
                21      cpu-migrations            #    0.074 K/sec                  
               111      page-faults               #    0.390 K/sec                  
   <not supported>      cycles                                                      
   <not supported>      instructions                                                
   <not supported>      branches                                                    
   <not supported>      branch-misses                                               

       0.284814727 seconds time elapsed
  
  //程序 t1 是一个 CPU bound 型，因为 task-clock-msecs 接近 1
```
#### perf top
Perf top 用于实时显示当前系统的性能统计信息。该命令主要用来观察整个系统当前的状态，比如可以通过查看该命令的输出来查看当前系统最耗时的内核函数或某个用户进程

```
# perf record  -e cpu-clock ./test
# perf report
```

```
-g显示调用关系
# perf record   -g ./test
# perf report 

Samples: 1K of event 'cpu-clock', Event count (approx.): 299250000                                                                                     
  Children      Self  Command  Shared Object      Symbol                                                                                              
-   99.83%    99.83%  test     test               [.] longa                                                                                           
     __libc_start_main                                                                                                                                
   + main                                                                                                                                             
-   99.83%     0.00%  test     libc-2.17.so       [.] __libc_start_main                                                                               
     __libc_start_main                                                                                                                                
   + main                                                                                                                                             
-   99.83%     0.00%  test     test               [.] main                                                                                            
   + main                                                                                                                                             
-   90.56%     0.00%  test     test               [.] foo1                                                                                            
     foo1                                                                                                                                             
     longa                                                                                                                                            
-    9.27%     0.00%  test     test               [.] foo2                                                                                            
     foo2                                                                                                                                             
     longa                                                                                                                                            
     0.08%     0.08%  test     [kernel.kallsyms]  [k] unmap_page_range                                                                                
     0.08%     0.00%  test     [kernel.kallsyms]  [k] system_call_fastpath 
```


## 使用tracepoint
人们使用 tracepoint 的基本需求是对内核的运行时行为的关心，如前所述，有些内核开发人员需要专注于特定的子系统，比如内存管理模块。这便需要统计相关内核函数的运行情况
```
# perf stat -e raw_syscalls:sys_enter ls
 Performance counter stats for 'ls':

               123      raw_syscalls:sys_enter                                      

       0.002231434 seconds time elapsed
```

## 具体样例
原文链接[linux 性能调优工具perf + 火焰图 常用命令](https://blog.csdn.net/a6788578/article/details/80337291)
```
-e：Select the PMU event.
-a：System-wide collection from all CPUs.
-p：Record events on existing process ID (comma separated list).
-A：Append to the output file to do incremental profiling.
-f：Overwrite existing data file.
-o：Output file name.
-g：Do call-graph (stack chain/backtrace) recording.
-C：Collect samples only on the list of CPUs provided.
-p <pid>：Profile events on existing Process ID (comma sperated list).仅分析目标进程及其创建的线程。
-k <path>：Path to vmlinux. Required for annotation functionality.带符号表的内核映像所在的路径。
-K：不显示属于内核或模块的符号。
-U：不显示属于用户态程序的符号。
-d <n>：界面的刷新周期，默认为2s，因为perf top默认每2s从mmap的内存区域读取一次性能数据

```
生成火焰图（执行1-4步骤）：

```
1、perf record -e cpu-clock -g -p pid    （perf record -F 99 -g -p pid  99HZ采样）
-g 选项是告诉perf record额外记录函数的调用关系
-e cpu-clock 指perf record监控的指标为cpu周期
-p 指定需要record的进程pid
perf report -i perf.data
-i 指定要查看的文件
2、perf script -i perf.data &> perf.unfold
用perf script工具对perf.data进行解析
3、./stackcollapse-perf.pl perf.unfold &> perf.folded 
将perf.unfold中的符号进行折叠
4、./flamegraph.pl perf.folded > perf.svg
最后生成svg图
火焰图项目地址：git clone https://github.com/brendangregg/FlameGraph.git
```

#### 统计事件，stat：statistics

```
//CPU counter statistics for the specified command:
# perf stat command
//Detailed CPU counter statistics (includes extras) for the specified command:
# perf stat -d command
//CPU counter statistics for the specified PID, until Ctrl-C:
# perf stat -p PID
//CPU counter statistics for the entire system, for 5 seconds:
# perf stat -a sleep 5
//Various basic CPU statistics, system wide, for 10 seconds:
# perf stat -e cycles,instructions,cache-references,cache-misses,bus-cycles -a sleep 10
```


#### 剖析 Profiling

```
//Sample on-CPU functions for the specified command, at 99 Hertz:
# perf record -F 99 command
//Sample on-CPU functions for the specified PID, at 99 Hertz, until Ctrl-C:
# perf record -F 99 -p PID
//Sample on-CPU functions for the specified PID, at 99 Hertz, for 10 seconds:
# perf record -F 99 -p PID sleep 10
//Sample CPU stack traces (via frame pointers) for the specified PID, at 99 Hertz, for 10 seconds:
# perf record -F 99 -p PID -g -- sleep 10
```


#### Static Tracing

```
//Trace new processes, until Ctrl-C:
# perf record -e sched:sched_process_exec -a
//Trace all context-switches, until Ctrl-C:
# perf record -e context-switches -a
//Trace context-switches via sched tracepoint, until Ctrl-C:
# perf record -e sched:sched_switch -a
//Trace all context-switches with stack traces, until Ctrl-C:
# perf record -e context-switches -ag
//Trace all context-switches with stack traces, for 10 seconds:
# perf record -e context-switches -ag -- sleep 10
```


#### Dynamic Tracing

```
//Add a tracepoint for the kernel tcp_sendmsg() function entry ("--add" is optional):
# perf probe --add tcp_sendmsg
//Remove the tcp_sendmsg() tracepoint (or use "--del"):
# perf probe -d tcp_sendmsg
//Add a tracepoint for the kernel tcp_sendmsg() function return:
# perf probe 'tcp_sendmsg%return'
//Show available variables for the kernel tcp_sendmsg() function (needs debuginfo):
# perf probe -V tcp_sendmsg
//Show available variables for the kernel tcp_sendmsg() function, plus external vars (needs debuginfo):
# perf probe -V tcp_sendmsg --externs
```


#### Mixed

```
//Sample stacks at 99 Hertz, and, context switches:
# perf record -F99 -e cpu-clock -e cs -a -g 
//Sample stacks to 2 levels deep, and, context switch stacks to 5 levels (needs 4.8):
# perf record -F99 -e cpu-clock/max-stack=2/ -e cs/max-stack=5/ -a -g
```


#### Reporting

```
//Show perf.data in an ncurses browser (TUI) if possible:
# perf report
//Show perf.data with a column for sample count:
# perf report -n
//Show perf.data as a text report, with data coalesced and percentages:
# perf report --stdio
//Report, with stacks in folded format: one line per stack (needs 4.4):
# perf report --stdio -n -g folded
//List all events from perf.data:
# perf script
//List all perf.data events, with data header (newer kernels; was previously default):
# perf script --header
```
## perf top

```
显示内核模块中，消耗最多CPU周期的函数：

# perf top -e cycles:k

显示分配高速缓存最多的函数：

# perf top -e kmem:kmem_cache_alloc
```

原文  [系统级性能分析工具 — Perf](https://blog.csdn.net/zhangskd/article/details/37902159)
## 性能事件
-e <event> : u // userspace

-e <event> : k // kernel

-e <event> : h // hypervisor

-e <event> : G // guest counting (in KVM guests)

-e <event> : H // host counting (not in KVM guests)


设置perf调用的阈值
```
# perf report -g graph,0.3
```

