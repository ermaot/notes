## linux中的信号
- 运行在用户模式下的进程会接收信号.如果接收的进程正运行在内核模式,那么信号的执行只有在该进程返回到用户模式时才会开始.
- 发送到非运行进程的信号一定是由内核保存,直到进程重新执行为止.休眠的进程可以是可中断的,也可以是不可中断的.如果一个在可中断休眠状态的进程(例如,等待终端输入的进程)收到了一个信号,那么内核会唤醒这个进程来处理信号.如果一个在不可中断休眠状态的进程收到了一个信号,那么内核会拖延此信号,直到该事件完成为止.
- 当进程收到一个信号时,可能会发生以下3种情况:
1. 进程可能会忽略此信号.有些信号不能被忽略,而有些没有默认行为的信号,默认会被忽略
2. 进程可能会捕获此信号,并执行一个被称为信号处理器的特殊函数.
3. 进程可能会执行信号的默认行为.例如,信号15( SIGTERM)的默认行为是结束进程.
- 当一个进程执行信号处理时,如果还有其他信号到达,那么新的信号会被阻断直到处
理器返回为止.

## 信号的名称和值

```
# kill -l 
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	
```

## signal标准信号表

信号|默认行为|说明|信号值
---|---|---|---
SIGABRT|生成core文件然后终止进程|这个信号告诉进程终止操作.ABRT通常由进程本身发送,即当进程调用abort函数发出个非正常终止信号时|6
SIGALRM|终止|警告时钟|14
SIGBUS|生成core文件然后终止进程|当进程引起一个总线错误时,BUS信号将被发送到进程.例如,访问了一部分未定义的内存对象|10
SIGCHLD|忽略|当子进程结束、被中断或是在被中断之后重新恢复时,CHLD信号会被发送到进程|20
SIGCONT|继续进程|CONT信号指示操作系统重新开始先前被STOP或TSTP暂停的进程|19
SIGFPE|生成core文件然后终止进程|进程执行一个错误的算术运算时,FPE信号会被发送到进程|18
SIGHUP|终止|当进程的控制终端关闭时,HUP信号会被发送到进程|1
SIGILL|生成core文件然后终止进程|当一个进程尝试执行一个非法指令时,ILL信号会被发送到进程|4
SIGINT|终止|当用户想要中断进程时,INT信号被进程的控制终端发送到进程|2
SIGKILL|终止|发送到进程的KILL信号会使进程立即终止.KLL信号不能被捕获或忽略|9
SIGPIPE|终止|当一个进程尝试向一个没有连接到其他目标的管道写入时,PPE信号会被发送到进程|13
SIGQUIT|终止|当用户要求进程执行coredump时,QUT信号由进程的控制终端发送到进程|3
SIGSEGV|生成core文件然后终止进程|当进程生成了一个无效的内存引用时,SEGV信号会被发送到进程|11
SIGSTOP|停止进程|STOP信号指示操作系统停止进程的执行|17
SIGTERM|终止|发送到进程的TERM信号用于要求进程终止|15
SIGTSTP|停止进程|TSTP信号由进程的控制终端发送到进程来要求它立即终止|18
SIGTTIN|停止进程|后台进程尝试读取时,TTN信号会被发送到进程|21
SIGTTOU|停止进程|后台进程尝试输出时,TTOU信号会被发送到进程|22
SIGUSRI|终止|发送到进程的USR1信号用于指示用户定义的条件|30
SIGUSR2|终止|同上|31
SIGPOLL|终止|一个异步输入输出时间事件发生时,POLL信号会被发送到进程|23
SIGPROF|终止|当仿形计时器过期时,PROF信号会被发送到进程|27
SIGSYS|生成core文件然后终止进程|发生有错的系统调用时,SYS信号会被发送到进程|12
SIGTRAP|生成core文件然后终止进程|追踪捕获/断点捕获时,会产生TRAP信号|5
SIGURG|忽略|当有一个socket有紧急的或是带外数据可被读取时,URG信号会被发送到进程|16
SIGVTALRM|终止|当进程使用的虚拟计时器过期时,VTALRM信号会被发送到进程|26
SIGXCPU|终止|当进程使用的CPU时间超出限制时,XCPU信号会被发送到进程|24
SIGXFSZ|生成core文件然后终止进程|当文件大小超过限制时,会产生XFSZ信号|25

## 进程状态
进程有以下状态：
- D(不可中断休眠状态)——进程正在休眠并且不能恢复,直到一个事件发生为止
- R(运行状态)—进程正在运行
- S(休眠状态)—进程没有在运行,而在等待一个事件或是信号.
- T(停止状态)——进程被信号停止,比如,信号 SIGINT或 SIGSTOP.
- Z(僵死状态)——标记为< defunct>的进程是僵死的进程
还有以下选项
- < 高优先级
- N 低优先级
- L 有pages在内存中locked。用于实时或者自定义IO。
- s 进程领导者，其有子进程。
- l 多线程
- + 位于前台进程组。
```
# ps -C sshd -o pid,cmd,stat
  PID CMD                         STAT
  833 sshd: root@pts/2            Ss
 3046 /usr/sbin/sshd -D           Ss
 5272 sshd: root@pts/1            Ss
18951 sshd: root@pts/3            Ss
23508 sshd: root@pts/0            Ss
```
## pstree

```
//指定特定用户的进程
# pstree root
systemd─┬─AliYunDun───19*[{AliYunDun}]
        ├─AliYunDunUpdate───3*[{AliYunDunUpdate}]
        ├─2*[agetty]
        ├─aliyun-service───6*[{aliyun-service}]
        ├─atd
        ├─auditd───{auditd}
        ├─crond
        ├─dbus-daemon
        ├─gogs───6*[{gogs}]
        ├─irqbalance
        ├─java───63*[{java}]
        ├─java───74*[{java}]
        ├─java───53*[{java}]
        ├─mysqld───43*[{mysqld}]
        ├─nginx───2*[nginx]
        ├─ntpd
        ├─polkitd───5*[{polkitd}]
        ├─redis-server───2*[{redis-server}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd─┬─sshd───bash───pstree
        │      ├─2*[sshd───bash───ipython───{ipython}]
        │      └─sshd───bash
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]

//pstree -g 可以显示pid
```

## pgrep
```
//指定root和sshd用户的进程
# pgrep -u root,sshd

//如果不加逗号，则指定的是进程名
# pgrep -u root sshd
833
3046
5272
18951
23508
```

## 向进程发信号

#### 前台信号
组合键|含义
---|---
Cr+C|中断信号,发送 SIGINT信号到运行在前台的进程
Ctrl+Y|延时挂起信号,使运行的进程在尝试从终端读取输入时停止<p>控制权返回给 Shell,使用户可以将进程放在前台或后台,或杀掉该进程
Ctrl+Z|挂起信号,发送 SIGTSTP信号到运行的进程,由此将其停止,并将控制权返回给 Shell

#### 后台信号
- 一般最开始发SIGTERM信号，让结果按自己的流程结束
- 如果失败，则使用SIGINT或者SIGKILL信号

信号|数值
---|---
SIGHUP |(1)
SIGINT |(2)
SIGKILL|(9)
SIGCONT |(18)
SIGSTOP |(19)
```
//三者等同
#kill -9 123
#kill -kill 123
#kill -SIGKILL 123
```

```
//杀死job
# jobs -l
[1]+  8751 Running                 sleep 30 &
#kill %1
# jobs -l
[1]+  8751 Terminated              sleep 30
```

###### killall默认发送SIGTERM信号
```
# killall firefox

# killall -s SIGKILL firefox
```

###### pkill 可以指定进程名、用户名、组名、终端、UID、EUID、GID等属性来杀掉相应的进程

```
#pkill firefox
# pkill -KILL -u root firefox
//让sshd重新加载配置文件
# pkill -HUP  sshd
```
HUP有两种解释.
1. 他被许多守护进程理解为一个重新设置的请求.如果一个进程不用重新启动就能重新读取它的配置文件并调整自给以适应变化的话,那么HUP通常来触发这种行为.
2. HUP信号有时候又终端驱动程序生成,试图来"清除"("终止")跟某个特定终端相连的那些进程.例如:某个终端会话结束时,或者当调制解调器被挂断时,shell后台不接受HUP的信号的影响.有的的用户可以使用nohup来模仿这种行为



## 捕获

```
#!/bin/bash
echo "Adjust the size of the window"
trap "echo Window size changed " SIGWINCH
count=0
while [[ $count -lt 30 ]];do
    count=$(($count+1))
    sleep 1
done
```
