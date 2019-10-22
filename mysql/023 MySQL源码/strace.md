## strace
按照strace官网的描述,strace是一个可用于诊断、调试和教学的Linux用户空间跟踪器。我们用它来监控用户空间进程和内核的交互，比如系统调用、信号传递、进程状态变更等。

==strace底层使用内核的ptrace特性来实现其功能==

## strace 模式
strace有两种运行模式。

1. 一种是通过它启动要跟踪的进程。用法很简单，在原本的命令前加上strace即可。比如我们要跟踪 "ls -lh /var/log/messages" 这个命令的执行，可以这样：


```
strace ls -lh /var/log/messages
```

2. 另外一种运行模式，是跟踪已经在运行的进程，在不中断进程执行的情况下，理解它在干嘛。 这种情况，给strace传递个-p pid 选项即可

```
strace -p 17553
```

## strace 参数

```

-tt 在每行输出的前面，显示毫秒级别的时间
-T 显示每次系统调用所花费的时间
-v 对于某些相关调用，把完整的环境变量，文件stat结构等打出来。
-f 跟踪目标进程，以及目标进程创建的所有子进程
-e 控制要跟踪的事件和跟踪行为,比如指定要跟踪的系统调用名称
-o 把strace的输出单独写到指定的文件
-s 当系统调用的某个参数是字符串时，最多输出指定长度的内容，默认是32个字节
-p 指定要跟踪的进程pid, 要同时跟踪多个pid, 重复多次-p选项即可
-u username 以username 的UID和GID执行被跟踪的命令
```
-e参数

```
-e trace=set 只跟踪指定的系统 调用.例如:-e trace=open,close,read,write表示只跟踪这四个系统调用.默认的为set=all. 

-e trace=file 只跟踪有关文件操作的系统调用. 

-e trace=process 只跟踪有关进程控制的系统调用. 

-e trace=network 跟踪与网络有关的所有系统调用. 

-e strace=signal 跟踪所有与系统信号有关的系统调用 

-e trace=ipc 跟踪所有与进程通讯有关的系统调用 

-e abbrev=set 设定 strace输出的系统调用的结果集.-v 等与abbrev=none.默认为abbrev=all. 

-e raw=set 将指 定的系统调用的参数以十六进制显示.

-e signal=set 指定跟踪的系统信号.默认为all.如signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号. 

-e read=set 输出从指定文件中读出的数据.例如: -e read=3,5 

-e write=set 输出写入到指定文件中的数据. 

-o filename 将strace的输出写入文件filename.

-p pid 跟踪指定的进程pid. 

-s strsize 指定输出的字符串的最大长度.默认为32.文件名一直全部输出. 


```

## strace功能
1. 它可以基于特定的系统调用或系统调用组进行过滤
1. 它可以通过统计特定系统调用的使用次数，所花费的时间，以及成功和错误的数量来分析系统调用的使用。
1. 它跟踪发送到进程的信号。
1. 可以通过pid附加到任何正在运行的进程。
1. 调试性能问题，查看系统调用的频率，找出耗时的程序段
1. 查看程序读取的是哪些文件从而定位比如配置文件加载错误问题
1. 查看某个php脚本长时间运行“假死”情况
1. 当程序出现“Out of memory”时被系统发出的SIGKILL信息所kill
1. 另外因为strace拿到的是系统调用相关信息，一般也即是IO操作信息，这个对于排查比如cpu占用100%问题是无能为力的。这个时候就可以使用GDB工具了。


### strace 样例

```
# strace -p `pidof mysql`
ioctl(0, SNDCTL_TMR_STOP or TCSETSW, {B38400 opost isig icanon echo ...}) = 0
rt_sigprocmask(SIG_BLOCK, [HUP INT QUIT TERM CONT TSTP WINCH], [], 8) = 0
rt_sigaction(SIGINT, {0x40ca80, [INT], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigaction(SIGTSTP, {SIG_DFL, [], SA_RESTORER, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigaction(SIGQUIT, {0x40de90, [QUIT], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigaction(SIGHUP, {0x40e190, [HUP], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigaction(SIGTERM, {SIG_DFL, [], SA_RESTORER, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigaction(SIGCONT, {SIG_DFL, [], SA_RESTORER, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigaction(SIGWINCH, {0x40a9c0, [WINCH], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, NULL, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
times({tms_utime=60, tms_stime=42, tms_cutime=0, tms_cstime=0}) = 1929275554
sendto(3, "\31\0\0\0\3insert into t2 values(4)", 29, 0, NULL, 0) = 29
recvfrom(3, "\7\0\0\1\0\1\0\2\10\0\0", 16384, 0, NULL, NULL) = 11
times({tms_utime=60, tms_stime=42, tms_cutime=0, tms_cstime=0}) = 1929275555
write(1, "Query OK, 1 row affected (0.01 s"..., 36) = 36
write(1, "\n", 1)                       = 1
stat("/etc/localtime", {st_mode=S_IFREG|0644, st_size=388, ...}) = 0
rt_sigprocmask(SIG_BLOCK, [HUP INT QUIT TERM CONT TSTP WINCH], [], 8) = 0
rt_sigaction(SIGINT, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {0x40ca80, [INT], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, 8) = 0
rt_sigaction(SIGTSTP, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {SIG_DFL, [], SA_RESTORER, 0x7fd37b6d96d0}, 8) = 0
rt_sigaction(SIGQUIT, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {0x40de90, [QUIT], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, 8) = 0
rt_sigaction(SIGHUP, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {0x40e190, [HUP], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, 8) = 0
rt_sigaction(SIGTERM, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {SIG_DFL, [], SA_RESTORER, 0x7fd37b6d96d0}, 8) = 0
rt_sigaction(SIGCONT, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {SIG_DFL, [], SA_RESTORER, 0x7fd37b6d96d0}, 8) = 0
rt_sigaction(SIGWINCH, {0x492960, [], SA_RESTORER, 0x7fd37b6d96d0}, {0x40a9c0, [WINCH], SA_RESTORER|SA_RESTART, 0x7fd37b6d96d0}, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
rt_sigprocmask(SIG_BLOCK, [WINCH], [], 8) = 0
ioctl(0, TIOCGWINSZ, {ws_row=27, ws_col=230, ws_xpixel=0, ws_ypixel=0}) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
write(1, "mysql> ", 7)                  = 7
ioctl(0, TCGETS, {B38400 opost isig icanon echo ...}) = 0
ioctl(0, SNDCTL_TMR_STOP or TCSETSW, {B38400 opost isig -icanon -echo ...}) = 0
```

```
# strace  -T -tt -f -e trace=read,open,write,pwrite64,pread64 -p `pidof mysqld`
[pid 24991] 16:24:18.724129 pwrite64(50, "\377\4\0\0\0", 5, 21) = 5 <0.000124>
[pid 24991] 16:24:18.724379 pwrite64(50, "\0\0", 2, 26) = 2 <0.000078>
[pid 24991] 16:24:18.724878 write(46, "2\334K]\"d\0\0\0O\0\0\0\17\36\240\1\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 318) = 318 <0.000147>
[pid 24991] 16:24:18.727515 pwrite64(44, "\0*\0\0\0\1\0\0\0\0\0\0\0\0\0\2\0\0\0\0\0\1\0\0\0\3\0\0\0\0\0\2"..., 1024, 1024 <unfinished ...>
[pid 24993] 16:24:18.727558 read(35, "2\334K]\"d\0\0\0O\0\0\0\17\36\240\1\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 4672) = 318 <0.000064>
[pid 24991] 16:24:18.727647 <... pwrite64 resumed> ) = 1024 <0.000108>
[pid 24991] 16:24:18.727707 pwrite64(44, "\376\376\7\1\0\0\1T\0\260\0d\0\304\0\1\0\0\1\0\377\1\0\0\0\0019\377\0\0\0\0"..., 140, 0) = 140 <0.000070>
[pid 24991] 16:24:18.728243 write(33, "# Time: 2019-08-08T08:24:18.7281"..., 213) = 213 <0.000081>
```
