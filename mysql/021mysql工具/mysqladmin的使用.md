本文参考：https://www.cnblogs.com/dadonggg/p/8625500.html

https://blog.51cto.com/auskangaroo/577687

## 介绍

mysqladmin是一个执行管理操作的客户端程序。它可以用来检查服务器的配置和当前状态、创建和删除数据库等。

命令选项分为两大类：

1. command选项
2. 标准格式选项，比如mysqladmin shutdown，结合-u -p等选项来指定关闭哪个实例

## 选项

option 选项：
选项|说明
---|---
-c number |自动运行次数统计，必须和 -i 一起使用
-i number |间隔多长时间重复执行。例如：<br>每个两秒查看一次服务器的状态，总共重复5次。<br>./mysqladmin -uroot -p -i 2 -c 5 status
-h, --host=name |Connect to host. 连接的主机名或iP
-p, --password[=name] |登录密码，如果不写于参数后，则会提示输入
-P, --port=# |指定数据库端口
-s, --silent |Silently exit if one can't connect to server.
-S, --socket=name | 指定socket file
-i, --sleep=# Execute| 间隔一段时间执行一次
-u, --user=name |登录数据库用户名
-v, --verbose | 写更多的信息
-V, --version | 显示版本

## 命令

```
mysqladmin
…………………………
Where command is a one or more of: (Commands may be shortened)
  create databasename	  Create a new database
  debug			  Instruct server to write debug information to log
  drop databasename	  Delete a database and all its tables
  extended-status         Gives an extended status message from the server
  flush-all-statistics    Flush all statistics tables
  flush-all-status        Flush status and statistics
  flush-client-statistics Flush client statistics
  flush-hosts             Flush all cached hosts
  flush-index-statistics  Flush index statistics
  flush-logs              Flush all logs
  flush-privileges        Reload grant tables (same as reload)
  flush-slow-log          Flush slow query log
  flush-status		  Clear status variables
  flush-table-statistics  Clear table statistics
  flush-tables            Flush all tables
  flush-threads           Flush the thread cache
  flush-user-statistics   Flush user statistics
  kill id,id,...	Kill mysql threads
  password [new-password] Change old password to new-password in current format
  old-password [new-password] Change old password to new-password in old format
  ping			Check if mysqld is alive
  processlist		Show list of active threads in server
  reload		Reload grant tables
  refresh		Flush all tables and close and open logfiles
  shutdown		Take server down
  status		Gives a short status message from the server
  start-slave		Start slave
  stop-slave		Stop slave
  variables             Prints variables available
  version		Get version info from server

```

## 实战样例

### 1. 每秒间隔打印状态信息

```
# mysqladmin -uroot -p123456 extended-status   -r -i 1 -c 2 | grep "bytes"
| Bytes_received                                        | 129616557    
| Bytes_sent                                            | 2912210      
| Innodb_buffer_pool_bytes_data                         | 118308864    
| Innodb_buffer_pool_bytes_dirty                        | 0            
| Mysqlx_bytes_received                                 | 0            
| Mysqlx_bytes_sent                                     | 0            
| Bytes_received                                        | 35           
| Bytes_sent                                            | 14107        
| Innodb_buffer_pool_bytes_data                         | 0            
| Innodb_buffer_pool_bytes_dirty                        | 0            
| Mysqlx_bytes_received                                 | 0            
| Mysqlx_bytes_sent                                     | 0    
```

第一次打印的是总的统计，第二次是相对值，即变化值

实际上，数据类似于

```
> show status like "%bytes%";
+--------------------------------+-----------+
| Variable_name                  | Value     |
+--------------------------------+-----------+
| Bytes_received                 | 394       |
| Bytes_sent                     | 1766      |
| Innodb_buffer_pool_bytes_data  | 118308864 |
| Innodb_buffer_pool_bytes_dirty | 0         |
| Mysqlx_bytes_received          | 0         |
| Mysqlx_bytes_sent              | 0         |
+--------------------------------+-----------+
```

### 2. 查看系统参数

```
# mysqladmin -uroot -p123456 variables   -r -i 1 -c 2  | grep "innodb"
```

### 3.查看线程和锁

```
# mysqladmin -uroot -p123456 debug
```

### 4.修改root 密码
```
# mysqladmin -u root -poldpassword password 'newpassword'
```

### 5.检查mysqlserver是否可用：
```
mysqladmin -uroot -p ping
```


### 6.查询服务器的版本
```
mysqladmin -uroot -p version
```


### 7.显示服务器所有运行的进程：
```
mysqladmin -uroot -p processlist
mysqladmin -uroot -p-i 1 processlist 每秒刷新一次
```

### 8.创建数据库
```
mysqladmin -uroot -p create daba-test

```

### 9.服务器上的所有数据库
```
mysqlshow -uroot -p

```

### 10.显示数据库daba-test下有些什么表：
```
mysqlshow -uroot -p daba-test

```

### 11.统计daba-test 下数据库表列的汇总
```
mysqlshow -uroot -p daba-test -v

```

### 12.统计daba-test 下数据库表的列数和行数
```
mysqlshow -uroot -p daba-test -v -v

```

###  13.删除数据库 daba-test
```
mysqladmin -uroot -p drop daba-test

```

###  14.重载权限信息
```
mysqladmin -uroot -p reload

```

### 15.刷新所有表缓存，并关闭和打开log
```
mysqladmin -uroot -p refresh

```