show engine显示有关存储引擎的操作信息。它需要process的权限。



SHOW ENGINE INNODB STATUS

SHOW ENGINE INNODB MUTEX

SHOW ENGINE {NDB | NDBCLUSTER} STATUS

SHOW ENGINE PERFORMANCE_SCHEMA STATUS



SHOW ENGINE INNODB MUTEX 显示InnoDB mutex和 rw-lock统计信息

type：始终是innodb

name：实现互斥体的源文件以及创建互斥体的文件中的行号。行号特定于您的MySQL版本。

status：互斥体状态。




count 指示要求互斥体的次数。

spin_waits 表示自旋锁必须运行多少次。

spin_rounds表示旋锁圈数。（spin_rounds除以spin_waits平均圆数）

os_waits表示操作系统等待的次数。这种情况发生在自旋锁不起作用（互斥锁在自旋锁中没有锁定，并且有必要屈服于操作系统并等待）。

os_yields 指示尝试锁定互斥锁的线程放弃其时间片并产生到操作系统的次数（假设允许其他线程运行将释放互斥锁，以使其可以被锁定）。

os_wait_times指示在操作系统等待中花费的时间量（以毫秒为单位）。在MySQL 5.5中，定时被禁用，该值始终为0。
--------------------- 
作者：大海的水 
原文：https://blog.csdn.net/c446591512/article/details/78129294 
