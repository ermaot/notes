本文参考：https://blog.csdn.net/c446591512/article/details/78129294 

https://www.cnblogs.com/hushaojun/p/4849739.html

## show engine

show engine显示有关存储引擎的操作信息。它需要process的权限。

```
SHOW ENGINE INNODB STATUS

SHOW ENGINE INNODB MUTEX

SHOW ENGINE {NDB | NDBCLUSTER} STATUS

SHOW ENGINE PERFORMANCE_SCHEMA STATUS
```

#### SHOW ENGINE INNODB MUTEX 
SHOW ENGINE INNODB MUTEX 显示InnoDB mutex和 rw-lock统计信息

字段名|解释
---|---
type|始终是innodb
name|实现互斥体的源文件以及创建互斥体的文件中的行号。行号特定于您的MySQL版本。
status|互斥体状态。<br>count 指示要求互斥体的次数。<br>spin_waits 表示自旋锁必须运行多少次。<br>spin_rounds表示旋锁圈数。（spin_rounds除以spin_waits平均圆数）<br>os_waits表示操作系统等待的次数。这种情况发生在自旋锁不起作用（互斥锁在自旋锁中没有锁定，并且有必要屈服于操作系统并等待）。<br>os_yields 指示尝试锁定互斥锁的线程放弃其时间片并产生到操作系统的次数（假设允许其他线程运行将释放互斥锁，以使其可以被锁定）。<br>os_wait_times 指示在操作系统等待中花费的时间量（以毫秒为单位）。在MySQL 5.5中，定时被禁用，该值始终为0。

## 其他的show
命令|说明
---|---
show tables<br>show tables from database_name| 显示当前数据库中所有表的名称
show databases| 显示mysql中所有数据库的名称
show columns from table_name from database_name; 或show columns from database_name.table_name| 显示表中列名称
show grants for user_name| 显示一个用户的权限，显示结果类似于grant 命令
show index from table_name| 显示表的索引
show status| 显示一些系统特定资源的信息，例如，正在运行的线程数量
show variables| 显示系统变量的名称和值
show [full] processlist| 显示系统中正在运行的所有进程，也就是当前正在执行的查询。大多数用户可以查看他们自己的进程，但是如果他们拥有process权限，就可以查看所有人的进程，包括密码。
show table status| 显示当前使用或者指定的database中的每个表的信息。信息包括表类型和表的最新更新时间
show privileges| 显示服务器所支持的不同权限
show create database database_name| 显示create database 语句是否能够创建指定的数据库
show create table table_name| 显示create database 语句是否能够创建指定的数据库
show logs| 显示BDB存储引擎的日志
show warnings| 显示最后一个执行的语句所产生的错误、警告和通知
show errors| 只显示最后一个执行语句所产生的错误
show [storage] engines|显示安装后的可用存储引擎和默认引擎
show procedure status|显示数据库中所有存储的存储过程基本信息，包括所属数据库，存储过程名称，创建时间等
show create procedure sp_name|显示某一个存储过程的详细信息
show plugins |显示数据库插件（包括存储引擎）

