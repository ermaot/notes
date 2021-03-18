## bpf命令集锦

### 1. tcplife

```
# tcplife
PID   COMM       LADDR           LPORT RADDR           RPORT TX_KB RX_KB MS
391883 ssh        10.105.112.34   59214 39.156.69.79    22        0     0 3064.70

```

使用tcp_set_state(）内核函数或者sock:inet_sock_set_state跟踪点

### 2. 

```
# bpftrace -e 'tracepoint:sched:sched_process_exec {printf("exec by %s\n",comm)}'
Attaching 1 probe...
exec by ls
exec by date

```

### 3. perf_open_events

```
bpftrace -e 'tracepoint:syscalls:sys_enter_perf_event_open {printf("exec by %s\n",comm)}'
```

### 4.vfstat

```
# vfsstat 
libbpf: failed to find valid kernel BTF
libbpf: vmlinux BTF is not found
TIME         READ/s  WRITE/s  FSYNC/s   OPEN/s CREATE/s
22:14:10:        30        2        0       90        0
22:14:11:        22        2        0       88        0
22:14:12:        34        3        0       98        0
22:14:13:        24        3        0       89        0
```

```
# bpftrace -e 'kprobe:vfs_read  { @[comm] =count()}'
Attaching 1 probe...
^C

@[sshd]: 1
@[in:imjournal]: 4
@[redis-server]: 68

# bpftrace -e 'kprobe:vfs_read,kprobe:vfs_write  { @[comm] =count()}'
Attaching 2 probes...
^C

@[bpftrace]: 1
@[sshd]: 3
@[in:imjournal]: 5
@[redis-server]: 78

```

### 查看内核函数

```
/proc/kallsyms
/boot/system.map
```

### funccount

```
# funccount tcpdrop
# funccount 'vfs_*'
# funccount 'tcp_*'
# founccount -i 1 c:pthread_mutex_lock

//libc库中调用最频繁的字符串函数
# funcount 'c:str*'

//最频繁的系统调用
funccount 't:syscalls:sys_enter_*'

//统计每秒tcp发送函数的调用
funccount -i 1 'tcp_send*'

//每秒块IO事件的数量
funccount -i 1 't:block:*'

//每秒新创建的进程数量
funccount -i 1 t:sched:sched_process_fork

//每秒libc中getaddrinfo函数的调用次数
funccount -i 1 c:getaddrinfo

//对libgo中的全部os.*进行计数
funccount 'go:os.*'
```

stackcount



trace 'do_sys_open' -p 43257

trace 'do_sys_open "%s" ,arg2'

argdist 

零窗口宣告(zero-window advertisement)

bpftrace -e 'kretprobe:vfs_read {@bytes=hist(retval)}'