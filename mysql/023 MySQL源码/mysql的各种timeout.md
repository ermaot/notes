## 与timeout相关的参数

本文来自：http://www.penglixun.com/tech/database/mysql_timeout.html

https://blog.csdn.net/weixin_39004901/article/details/102516046

以mysql8.0.16为准

```
> show variables like "%timeout%";
+-----------------------------------+----------+
| Variable_name                     | Value    |
+-----------------------------------+----------+
| connect_timeout                   | 10       |
| delayed_insert_timeout            | 300      |
| have_statement_timeout            | YES      |
| innodb_flush_log_at_timeout       | 1        |
| innodb_lock_wait_timeout          | 50       |
| innodb_rollback_on_timeout        | OFF      |
| interactive_timeout               | 28800    |
| lock_wait_timeout                 | 31536000 |
| mysqlx_connect_timeout            | 30       |
| mysqlx_idle_worker_thread_timeout | 60       |
| mysqlx_interactive_timeout        | 28800    |
| mysqlx_port_open_timeout          | 0        |
| mysqlx_read_timeout               | 30       |
| mysqlx_wait_timeout               | 28800    |
| mysqlx_write_timeout              | 60       |
| net_read_timeout                  | 30       |
| net_write_timeout                 | 60       |
| rpl_stop_slave_timeout            | 31536000 |
| slave_net_timeout                 | 60       |
| wait_timeout                      | 28800    |
+-----------------------------------+----------+
20 rows in set (0.01 sec)
```

## 各项参数解释

参数|说明
---|---
connect_timeout                   | 在获取链接时，等待握手的超时时间，只在登录时有效，登录成功这个参数就不管事了。主要是为了防止网络不佳时应用重连导致连接数涨太快，一般默认即可。
**delayed_insert_timeout**            | 这是为MyISAM INSERT DELAY设计的超时参数，在INSERT DELAY中止前等待INSERT语句的时间。已经废弃，不支持延迟插入。 
have_statement_timeout            | 是否开启语句执行超时 
innodb_flush_log_at_timeout       | 
innodb_lock_wait_timeout          | 就是事务遇到锁等待时的Query超时时间。跟死锁不一样，InnoDB一旦检测到死锁立刻就会回滚代价小的那个事务，锁等待是没有死锁的情况下一个事务持有另一个事务需要的锁资源，被回滚的肯定是请求锁的那个Query
innodb_rollback_on_timeout        | 这个参数关闭或不存在的话遇到超时只回滚事务最后一个Query，打开的话事务遇到超时就回滚整个事务
interactive_timeout               | 一个持续SLEEP状态的线程多久被关闭。线程每次被使用都会被唤醒为activity状态，执行完Query后成为interactive状态，重新开始计时。wait_timeout不同在于只作用于TCP/IP和Socket链接的线程，意义是一样的
lock_wait_timeout                 | 等待MDL的超时时间，单位为秒，取值范围1-31536000（一年），默认是一年。不应用于隐式的系统表的MDL锁等待，例如grant操作隐式地修改mysql.user表 
mysqlx_connect_timeout            | 
mysqlx_idle_worker_thread_timeout | 
mysqlx_interactive_timeout        | 
mysqlx_port_open_timeout          | 
mysqlx_read_timeout               | 
mysqlx_wait_timeout               | 
mysqlx_write_timeout              | 
net_read_timeout                  | MySQL在读取客户端数据时的空闲等待超时时间 
net_write_timeout                 | MySQL在写一个block到一个连接的等待超时时间。 
rpl_stop_slave_timeout            | 控制stop slave命令的前端等待时间，单位为秒，取值范围2-31536000（一年），默认是一年.例如发出stop slave，但是由于复制线程由于某些原因无法停下，所以stop slave暂时无法执行完成，所以客户端会一直卡住，而rpl_stop_slave_timeout控制客户端卡住的时间。超时后会转入后台执行。 
slave_net_timeout                 | 这是Slave判断主机是否挂掉的超时设置，在设定时间内依然没有获取到Master的回应就认为Master挂掉了。默认为60 
wait_timeout                      | 非交互式连接的空闲超时时间，单位为秒。 

## X Plugin Options and System Variables

X plugin是mysql新发版本5.7.12中新增的插件，利用它实现mysql作为文件存储数据库，也就是利用mysql 5.7版本json支持的特性完成

参数|说明
---|---
mysqlx_connect_timeout            | 
mysqlx_idle_worker_thread_timeout | 
mysqlx_interactive_timeout        | 
mysqlx_port_open_timeout          | 
mysqlx_read_timeout               | 
mysqlx_wait_timeout               | 
mysqlx_write_timeout              | 