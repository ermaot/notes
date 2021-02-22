## Linux系统调用列表

### 一、进程控制：

序号|调用名|说明|所在文件
---|---|---|---
1|  fork|创建一个新进程|kernel/fork.c
2|  clone|按指定条件创建子进程|kernel/fork.c
3|  execve|运行可执行文件|fs/exec.c
4|  exit|中止进程|kernel/exit.c
5|  _exit|立即中止当前进程|
6|  getdtablesize|进程所能打开的最大文件数|kernel/sys.c
7|  getpgid|获取指定进程组标识号|kernel/sys.c
8|  setpgid|设置指定进程组标志号|kernel/sys.c
9|  getpgrp|获取当前进程组标识号|kernel/sys.c
10| setpgrp|设置当前进程组标志号|kernel/sys.c
11| getpid|获取进程标识号|kernel/sys.c
12| getppid|获取父进程标识号|kernel/sys.c
13| getpriority|获取调度优先级|kernel/sys.c
14| setpriority|设置调度优先级|kernel/sys.c
15| modify_ldt|读写进程的本地描述表|arch/x86/um/ldt.c
16| nanosleep|使进程睡眠指定的时间|kernel/hrtimer.c
17| nice|改变分时进程的优先级|
18| pause|挂起进程，等待信号|kernel/signal.c
19| personality|设置进程运行域|kernel/exec_domain.c
20| prctl|对进程进行特定操作|kernel/sys.c
21| ptrace|进程跟踪|kernel/ptrace.c
22| sched_get_priority_max|取得静态优先级的上限|kernel/sched/core.c
23| sched_get_priority_min|取得静态优先级的下限|kernel/sched/core.c
24| sched_getparam|取得进程的调度参数|kernel/sched/core.c
25| sched_getscheduler|取得指定进程的调度策略|kernel/sched/core.c
26| sched_rr_get_interval|取得按RR算法调度的实时进程的时间片长度|kernel/sched/core.c
27| sched_setparam|设置进程的调度参数|kernel/sched/core.c
28| sched_setscheduler|设置指定进程的调度策略和参数|kernel/sched/core.c
29| sched_yield|进程主动让出处理器,并将自己等候调度队列队尾|kernel/sched/core.c
30| vfork|创建一个子进程，以供执行新程序，常与execve等同时使用|kernel/fork.c
31| wait|等待子进程终止|
32| wait3|参见wait|
33| waitpid|等待指定子进程终止|
34| wait4|参见waitpid|kernel/exit.c
35| capget|获取进程权限|kernel/capability.c
36| capset|设置进程权限|kernel/capability.c
37| getsid|获取会晤标识号|kernel/sys.c
38| setsid|设置会晤标识号|kernel/sys.c

### 二、文件系统控制
序号|调用名|说明|所在文件
---|---|---|---
1|  fcntl|文件控制|fs/fcntl.c
2|  open|打开文件|fs/open.c
3|  creat|创建新文件|fs/open.c
4|  close|关闭文件描述字|fs/open.c
5|  read|读文件|fs/read_write.c
6|  write|写文件|fs/read_write.c
7|  readv|从文件读入数据到缓冲数组中|fs/read_write.c
8|  writev|将缓冲数组里的数据写入文件|fs/read_write.c
9|  pread|对文件随机读|fs/read_write.c
10| pwrite|对文件随机写|fs/read_write.c
11| lseek|移动文件指针|fs/read_write.c
12| _llseek|在64位地址空间里移动文件指针|
13| dup|复制已打开的文件描述字|fs/file.c
14| dup2|按指定条件复制文件描述字|fs/file.c
15| flock|文件加/解锁|fs/locks.c
16| poll|I/O多路转换|fs/select.c
17| truncate|截断文件|fs/open.c
18| ftruncate|参见truncate|fs/open.c
19| umask|设置文件权限掩码|kernel/sys.c
20| fsync|把文件在内存中的部分写回磁盘|fs/sync.c
21| access|确定文件的可存取性|fs/open.c
22| chdir|改变当前工作目录|fs/open.c
23| fchdir|参见chdir|fs/open.c
24| chmod|改变文件方式|fs/open.c
25| fchmod|参见chmod|fs/open.c
26| chown|改变文件的属主或用户组|fs/open.c
27| fchown|参见chown|fs/open.c
28| lchown|参见chown|fs/open.c
29| chroot|改变根目录|fs/open.c
30| stat|取文件状态信息|fs/stat.c
31| lstat|参见stat|fs/stat.c
32| fstat|参见stat|fs/stat.c
33| statfs|取文件系统信息|fs/statfs.c
34| fstatfs|参见statfs|fs/statfs.c
35| readdir|读取目录项|
36| getdents|读取目录项|fs/readdir.c
37| mkdir|创建目录|fs/namei.c
38| mknod|创建索引节点|fs/namei.c
39| rmdir|删除目录|fs/namei.c
40| rename|文件改名|fs/namei.c
41| link|创建链接|fs/namei.c
42| symlink|创建符号链接|fs/namei.c
43| unlink|删除链接|fs/namei.c
44| readlink|读符号链接的值|fs/stat.c
45| mount|安装文件系统|fs/namespace.c
46| umount|卸下文件系统|
47| ustat|取文件系统信息|fs/statfs.c
48| utime|改变文件的访问修改时间|fs/utimes.c
49| utimes|参见utime|fs/utimes.c
50| quotactl|控制磁盘配额|fs/quota/quota.c

### 三、系统控制
序号|调用名|说明|所在文件
---|---|---|---
1|  ioctl|I/O总控制函数|fs/ioctl.c
2|  _sysctl|读/写系统参数|kernel/sysctl_binary.c
3|  acct|启用或禁止进程记账|kernel/acct.c
4|  getrlimit|获取系统资源上限|kernel/sys.c
5|  setrlimit|设置系统资源上限|kernel/sys.c
6|  getrusage|获取系统资源使用情况|kernel/sys.c
7|  uselib|选择要使用的二进制函数库|fs/exec.c
8|  ioperm|设置端口I/O权限|arch/x86/kernel/ioport.c
9|  iopl|改变进程I/O权限级别|arch/x86/kernel/ioport.c
10| outb|低级端口操作|
11| reboot|重新启动|kernel/reboot.c
12| swapon|打开交换文件和设备|mm/swapfile.c
13| swapoff|关闭交换文件和设备|mm/swapfile.c
14| bdflush|控制bdflush守护进程|
15| sysfs|取核心支持的文件系统类型|fs/filesystems.c
16| sysinfo|取得系统信息|kernel/sys.c
17| adjtimex|调整系统时钟|kernel/time.c
18| alarm|设置进程的闹钟|kernel/timer.c
19| getitimer|获取计时器值|kernel/itimer.c
20| setitimer|设置计时器值|kernel/itimer.c
21| gettimeofday|取时间和时区|kernel/time.c
22| settimeofday|设置时间和时区|kernel/time.c
23| stime|设置系统日期和时间|
24| time|取得系统时间|
25| times|取进程运行时间|kernel/sys.c
26| uname|获取当前UNIX系统的名称、版本和主机等信息|kernel/sys.c
27| vhangup|挂起当前终端|fs/open.c
28| nfsservctl|对NFS守护进程进行控制|
29| vm86|进入模拟8086模式|
30| create_module|创建可装载的模块项|
31| delete_module|删除可装载的模块项|kernel/module.c
32| init_module|初始化模块|kernel/module.c
33| query_module|查询模块信息|
34| *get_kernel_syms|取得核心符号,已被query_module代替

### 四、内存管理
序号|调用名|说明|所在文件
---|---|---|---
1| brk|改变数据段空间的分配|mm/mmap.c
2| sbrk|参见brk|
3| mlock|内存页面加锁|mm/mlock.c
4| munlock|内存页面解锁|mm/mlock.c
5| mlockall|调用进程所有内存页面加锁|mm/mlock.c
6| munlockall|调用进程所有内存页面解锁|mm/mlock.c
7| mmap|映射虚拟内存页|arch/x86/kernel/sys_x86_64.c
8| munmap|去除内存页映射|mm/mmap.c
9| mremap|重新映射虚拟内存地址|mm/mmap.c
10|msync|将映射内存中的数据写回磁盘|mm/msync.c
11|mprotect|设置内存映像保护|mm/mprotect.c
12|getpagesize|获取页面大小|
13|sync|将内存缓冲区数据写回硬盘|fs/sync.c
14|cacheflush|将指定缓冲区中的内容写回磁盘|

### 五、网络管理

序号|调用名|说明|所在文件
---|---|---|---
1| getdomainname|取域名|
2| setdomainname|设置域名|kernel/sys.c
3| gethostid|获取主机标识号|
4| sethostid|设置主机标识号|
5| gethostname|获取本主机名称|
6| sethostname|设置主机名称|kernel/sys.c
### 六、socket控制

序号|调用名|说明|所在文件
---|---|---|---
1| socketcall|socket系统调用|
2| socket|建立socket|net/socket.c
3| bind|绑定socket到端口|net/socket.c
4| connect|连接远程主机|net/socket.c
5| accept|响应socket连接请求|net/socket.c
6| send|通过socket发送信息|
7| sendto|发送UDP信息|net/socket.c
8| sendmsg|参见send|net/socket.c
9| recv|通过socket接收信息|
10|recvfrom|接收UDP信息|net/socket.c
11|recvmsg|参见recv|net/socket.c
12|listen|监听socket端口|net/socket.c
13|select|对多路同步I/O进行轮询|fs/select.c
14|shutdown|关闭socket上的连接|net/socket.c
15|getsockname|取得本地socket名字|net/socket.c
16|getpeername|获取通信对方的socket名字|net/socket.c
17|getsockopt|取端口设置|net/socket.c
18|setsockopt|设置端口参数|net/socket.c
19|sendfile|在文件或端口间传输数据|fs/read_write.c
20|socketpair|创建一对已联接的无名socket|net/socket.c
### 七、用户管理

序号|调用名|说明|所在文件
---|---|---|---
1| getuid|获取用户标识号|kernel/sys.c
2| setuid|设置用户标志号|kernel/sys.c
3| getgid|获取组标识号|kernel/sys.c
4| setgid|设置组标志号|kernel/sys.c
5| getegid|获取有效组标识号|kernel/sys.c
6| setegid|设置有效组标识号|kernel/sys.c
7| geteuid|获取有效用户标识号|kernel/sys.c
8| seteuid|设置有效用户标识号|kernel/sys.c
9| setregid|分别设置真实和有效的的组标识号|kernel/sys.c
10|setreuid|分别设置真实和有效的用户标识号|kernel/sys.c
11|getresgid|分别获取真实的,有效的和保存过的组标识号|kernel/sys.c
12|setresgid|分别设置真实的,有效的和保存过的组标识号|kernel/sys.c
13|getresuid|分别获取真实的,有效的和保存过的用户标识号|kernel/sys.c
14|setresuid|分别设置真实的,有效的和保存过的用户标识号|kernel/sys.c
15|setfsgid|设置文件系统检查时使用的组标识号|kernel/sys.c
16|setfsuid|设置文件系统检查时使用的用户标识号|kernel/sys.c
17|getgroups|获取后补组标志清单|kernel/groups.c
18|setgroups|设置后补组标志清单|kernel/groups.c
### 八、进程间通信

序号|调用名|说明|所在文件
---|---|---|---
**1、信号**|||
2| sigaction|设置对指定信号的处理方法|
3| sigprocmask|根据参数对信号集中的信号执行阻塞/解除阻塞等操作|
4| sigpending|为指定的被阻塞信号设置队列|
5| sigsuspend|挂起进程等待特定信号|
6| signal|参见signal|
7| kill|向进程或进程组发信号|kernel/signal.c
8| *sigblock|向被阻塞信号掩码中添加信号,已被sigprocmask代替|
9|*siggetmask|取得现有阻塞信号掩码,已被sigprocmask代替|
10|*sigsetmask||
11|*sigmask|将给定的信号转化为掩码,已被sigprocmask代替|
12|*sigpause|作用同sigsuspend,已被sigsuspend代替|
13|sigvec|为兼容BSD而设的信号处理函数,作用类似sigaction|
14|ssetmask|ANSI C的信号处理函数,作用类似sigaction|
**2、消息**|||
1|msgctl|消息控制操作|ipc/msg.c
2|msgget|获取消息队列|ipc/msg.c
3|msgsnd|发消息|ipc/msg.c
4|msgrcv|取消息|ipc/msg.c
**3、管道**|||
1|pipe|创建管道|fs/pipe.c
**4、信号量**|||
1|semctl|信号量控制|ipc/sem.c
2|semget|获取一组信号量|ipc/sem.c
3|semop|信号量操作|ipc/sem.c
**5、共享内存**|||
1|shmctl|控制共享内存|ipc/shm.c
2|shmget|获取共享内存|ipc/shm.c
3|shmat|连接共享内存|ipc/shm.c
4|shmdt|拆卸共享内存|ipc/shm.c
