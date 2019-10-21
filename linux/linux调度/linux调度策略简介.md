linux调度策略
### 调度策略类型

---

TSS（分时操作系统，time sharing system）
RT（实时操作系统）
###### 静态优先级：1~99
==TSS优先级==：0
###### 实时优先级：1~99

### 策略介绍：

---

#### ==sched_other==
&gt; 1. 标准的linux调度策略
&gt; 1. 时间片动态确定
&gt; 1. 平等赋予执行权（可能加入经验性判断）
#### ==sched_fifo==
&gt; 1. 实时调度策略（具有静态的优先级）
&gt; 1. 三个条件才释放执行权：等待I/O，自动休眠，更高优先级
&gt; 1. 进程死循环，又不释放执行权，会死机
#### ==sched_rr==
&gt; 1. RR（round robin）轮询机制
&gt; 1. 具有时间片（相比FIFO），长度100ms（使用了CFS）
#### sched_batch
非会话型

#### sched_idle
&gt; 1. 最低等级
&gt; 1. cpu空闲的时候，其他优先级的进程消失，则获得执行权
##### 标志：sched_reset_on_fork
设置了该标志，则子进程创建的时候会变成系统的默认策略（即sched_other）

### chrt命令

---
chrt --help

chrt (util-linux 2.13-pre7)

usage: chrt [options] [prio] [pid | cmd [args...]]
manipulate real-time attributes of a process

  -b, --batch    set policy to SCHED_BATCH
  
  -f, --fifo     set policy to SCHED_FF
  
  -p, --pid      operate on existing given pid
  
  -m, --max      show min and max valid priorities
  
  -o, --other    set policy to SCHED_OTHER
  
  -r, --rr       set policy to SCHED_RR (default)
  
  -h, --help     display this help
  
  -v, --verbose   display status information
  
  -V, --version   output version information
