命令功能：
ionice – 获取或设置程序的IO调度与优先级。

命令格式：
ionice [[-c class] [-n classdata] [-t]] -p PID [PID]…

ionice [-c class] [-n classdata] [-t] COMMAND [ARG]…

IO调度策略：

ionice将磁盘IO调度分为三类：

ilde：空闲磁盘调度，该调度策略是在当前系统没有其他进程需要进行磁盘IO时，才能进行磁盘；因此该策略对当前系统的影响基本为0；当然，该调度策略不能带有任何优先级参数；目前，普通用户是可以使用该调度策略（自从内核2.6.25开始）。

Best effort：是缺省的磁盘IO调度策略；(1)该调度策略可以指定优先级参数(范围是0~7，数值越小，优先级越高)；(2)针对处于同一优先级的程序将采round-robin方式；(3)对于best effort调度策略，8个优先级等级可以说明在给定的一个调度窗口中时间片的大小。(4)目前，普调用户(非root用户)是可以使用该调度策略。(5)在内核2.6.26之前，没有设置IO优先级的进程会使用“none”作为调度策略，但是这种策略使得进程看起来像是采用了best effort调度策略，因为其优先级是通过关于cpu nice有关的公式计算得到的：io_priority = (cpu_nice + 20) /5。(6)在内核2.6.26之后，如果当前系统使用的是CFQ调度器，那么如果进程没有设置IO优先级级别，将采用与内核2.6.26之前版本同样的方式，推到出io优先级级别。

Real time：实时调度策略，如果设置了该磁盘IO调度策略，则立即访问磁盘，不管系统中其他进程是否有IO。因此使用实时调度策略，需要注意的是，该访问策略可能会使得其他进程处于等待状态。

参数说明：

-c class ：class表示调度策略，其中0 for none, 1 for real time, 2 for best-effort, 3 for idle。
-n classdata：classdata表示IO优先级级别，对于best effort和real time，classdata可以设置为0~7。
-p pid：指定要查看或设置的进程号或者线程号，如果没有指定pid参数，ionice will run the listed program with the given parameters。
-t ：忽视设置优先级时产生的错误。
COMMAND：表示命令名



有的时候，生产系统很繁忙，但又需要删除大文件，就会陷入一个两难的境地：普通的删除命令，会导致大IO，可能造成系统的抖动；而删除操作又必须得做。可以使用ionice将IO优先级降低，等IO空闲的时候再删除。