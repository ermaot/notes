## 参数文件
1. mysql启动的时候，会去读一个参数文件，寻找==数据库的各种文件所在位置==与==某些初始化参数==
2. mysql --help | grep my.cnf
```
# mysql --help | grep my.cnf
/etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf 
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
```
3. oracle启动的时候找不到参数文件，不可启动；mysql可以不用，编译和源码中有默认值
4. oracle有二进制参数文件spfile和文件参数文件init.ora，mysql是全文本参数文件
5. 两种查看参数文件的方式：information_shema.GLOBAL_VARIABLES 表与 show [global] variables

```
> select * from information_schema.GLOBAL_VARIABLES;
………………
419 rows in set

> show global variables ;
………………
419 rows in set

> show  variables ;
………………
433 rows in set
```
6. 参数有动态参数（可在实例中修改）和静态参数（实例生命周期中不可修改）

```
set [global | session ] sys_var_name = expr
set [@@global | @@session |@@] sys_var_name = expr
> select @@session.read_buffer_size;
+----------------------------+
| @@session.read_buffer_size |
+----------------------------+
|                     131072 |
+----------------------------+
> set  @@session.read_buffer_size=131071;
Query OK, 0 rows affected, 1 warning (0.00 sec)

> set read_buffer_size=131071;
Query OK, 0 rows affected, 1 warning (0.00 sec)


-------------session 和 global的同一个参数的不同数值------------------

> select @@session.read_buffer_size as s;
+--------+
| s      |
+--------+
| 126976 |
+--------+
1 row in set (0.00 sec)


> select @@global.read_buffer_size as s;
+--------+
| s      |
+--------+
| 131072 |
+--------+
1 row in set (0.00 sec)

```


7. 有的动态参数只能在会话中修改，如autocommit；有的修改后在整个实例生命周期中都有效，如binlog_cache_size；有的既可以在会话中，也可以在实例周期中生效，如read_buffer_size
8. 在实例周期中有效的参数，不会自动写入到配置文件中，下次mysql启动依然会读取原配置文件
## 日志文件
#### 错误日志
- 类似于oracle的alert文件，但以err结尾

```
> show variables like "%log_err%";
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| log_error     | /var/log/mariadb/mariadb.log |
+---------------+------------------------------+

```

#### 二进制日志
二进制日志，记录所有==对数据库做修改==的操作（不包括show，select）
- 用于恢复（recovery）
- 复制（replication）
- 
```
> show variables like "log_bin";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+

> set global log_bin=on;
ERROR 1238 (HY000): Variable 'log_bin' is a read only variable

> show variables like "%datadir%";
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+

```
- max_binlog_size 指定了单个bin log的最大值。超过则新生成一个，序号加1，并且记录到.index文件中。默认值1073741824，即1G
- binlog_cache_size 
1. 未提交的二进制日志，会记录到缓存中，提交时将缓存中的日志写入到二进制日志文件。缓冲大小由 binlog_cache_size 决定
2. binlog_cache_size 基于会话，即线程开始一个事务时，mysql会自动分配一个binlog_cache_size 大小的缓存 。太小则事务写不下，会写临时文件降低性能。
3. 查看 Binlog_cache_disk_use 和 Binlog_cache_use ，可以判断是否合适
#### 慢查询日志
- 与long_query_time刚好相等的语句，不会被记录
- 开启慢查询日志 slow_query_log = on
```
> show variables like "%slow_query_log%";
+---------------------+---------------------------+
| Variable_name       | Value                     |
+---------------------+---------------------------+
| slow_query_log      | ON                        |
| slow_query_log_file | VM_112_34_centos-slow.log |
+---------------------+---------------------------+


> show variables like "%long_query%"
    -> ;
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 0.500000 |
+-----------------+----------+


> show variables like "%log_slow_queries%";
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| log_slow_queries | ON    |
+------------------+-------+
The --log-slow-queries option is deprecated and is removed (along with the log_slow_queries system variable) in MySQL 5.6
```
- 开启未使用索引的查询

```
> show variables like "%queries%";
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | OFF   |
+-------------------------------+-------+
> set global log_queries_not_using_indexes=on;
```

- mysqldumpslow

```
# mysqldumpslow VM_112_34_centos-slow.log 
# 查看锁定时间最长的10条语句
# mysqldumpslow -s al -t 10 VM_112_34_centos-slow.log 
//此处应该使用-t而非-n，innodb内幕一书中命令有误
Reading mysql slow query log from VM_112_34_centos-slow.log

```

- 输出格式是"FIL" 还是 "TABLE"

```
> show variables like "%log_output%";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+

> set global log_output="table";
Query OK, 0 rows affected (0.15 sec)

> select * from mysql.slow_log limit 1;
```

- 的
#### 查询日志
- 查询日志记录了所有的请求信息，不管请求是否正确执行
- 默认文件为 主机名.log
- 可在 mysql.general_log中查询

```
> show variables like "%general_log%";
+------------------+----------------------+
| Variable_name    | Value                |
+------------------+----------------------+
| general_log      | ON                   |
| general_log_file | VM_112_34_centos.log |
+------------------+----------------------+

```


## 套接字文件

```
> show variables like "%socket%";
+---------------+---------------------------+
| Variable_name | Value                     |
+---------------+---------------------------+
| socket        | /var/lib/mysql/mysql.sock |
+---------------+---------------------------+

# mysql -u root -S /var/lib/mysql/mysql.sock
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16676
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```

## pid文件
- mysql实例启动的时候，会将进程ID写入到文件中
```
> show variables like "%pid_file%";
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| pid_file      | /var/run/mariadb/mariadb.pid |
+---------------+------------------------------+

# cat /var/run/mariadb/mariadb.pid
9465

```

## 表结构定义文件
- mysql存储是按照表，所以每个表都有对应的存储文件
- mysql使用.frm文件存储表的定义
- mysql使用.frm文件存储视图的定义，且该文件是简单文本形式

```
> create view v_test as select * from test;
# cat v_test.frm 
TYPE=VIEW
query=select `test`.`test`.`Id` AS `Id`,`test`.`test`.`title` AS `title`,`test`.`test`.`uid` AS `uid`,`test`.`test`.`money` AS `money`,`test`.`test`.`name` AS `name` from `test`.`test`
md5=a7a838a285acc3ba26b4f33b15ba0cb2
updatable=1
algorithm=0
definer_user=root
definer_host=localhost
suid=2
with_check_option=0
timestamp=2019-06-04 08:56:06
create-version=1
source=select * from test
client_cs_name=utf8
connection_cl_name=utf8_general_ci
view_body_utf8=select `test`.`test`.`Id` AS `Id`,`test`.`test`.`title` AS `title`,`test`.`test`.`uid` AS `uid`,`test`.`test`.`money` AS `money`,`test`.`test`.`name` AS `name` from `test`.`test`
mariadb-version=50556

```

## innodb存储引擎文件
#### 表空间文件
- innodb 模仿oracle，有单独的表空间存放数据
- 默认配置有大小10M，名称ibdata1的文件

```
# locate ibdata1
/var/lib/mysql/ibdata1


```

- innodb_data_file_path的配置

```

> show variables like "%file_path%";
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:10M:autoextend |
+-----------------------+------------------------+
```
配置格式：
innodb_data_file_path = datafile_spec1[;datafile_spec2...]<p>
如：<p>
[mysqld]
innodb_data_file_path = /db/ibdata1:2000M;/dr/ibdata2:2000M:autoextend

- innodb_file_per_table

```
> show variables like "%file_per%";
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | OFF   |
+-----------------------+-------+

```
1. 配置了该参数，存储表的数据、索引、插入缓冲等信息会存入单独的.ibd中
2. 其余信息（undo信息，事务信息，二次写缓冲等）还是存放默认表空间
![innodb 文件的存储方式](A25F134EF929435F82D6BEEFB8025653)
#### 重做日志文件
- 默认情况下有两个ib_logfile1 和 ib_logfile1
- 可使用重做日志返回到宕机前的时刻，防止介质失效（media failure）
- 每一个mysql实例都有一个重做日志组（redo log group），每一个日志组都至少有2个重做日志
- 提高可靠性，可设置多个镜像日志组（mirrored log groups）
- 重做日志写满，会切换到下一个，循环往复
- innodb_log_file_size 指定重做日志大小

```
> show variables like "%innodb_log_file_size%";
+----------------------+---------+
| Variable_name        | Value   |
+----------------------+---------+
| innodb_log_file_size | 5242880 |
+----------------------+---------+

```

- innodb_log_files_in_group  每组中重做日志个数，默认为2

```
> show variables like "%innodb_log_files_in_group%";
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| innodb_log_files_in_group | 2     |
+---------------------------+-------+

```
- innodb_mirrored_log_groups 重做日志的镜像数，默认为1，即无其他镜像

```
> show variables like "%innodb_mirrored_log_groups%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_mirrored_log_groups | 1     |
+----------------------------+-------+

```
- innodb_log_group_home_dir 重做日志的路径，默认家目录（数据库路径下）

```
> show variables like "%innodb_log_group_home_dir%";
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| innodb_log_group_home_dir | ./    |
+---------------------------+-------+

```
- redo log大小不能太小，以免经常切换；不可太大，恢复时候太慢
- redo log 与二进制日志的区别
1. 二进制日志会记录所有与mysql有关的日志记录，不论引擎是什么，innodb只记录其本身的事务日志
2. 二进制日志记录的是事务的具体操作内容，不管格式是statement还是row，而redo log记录的是每一个页的更改情况
![重做日志结构](8B513EF5771C4248A2901B2021C2BF0F)
3. 写入时间不同，二进制日志是事务提交前记录的，而事务在进行过程中不断有日志产生
- innodb_flush_log_at_trx_commit 值可为0，1，2
1. innodb_flush_log_at_trx_commit = 0 表示事务完成时不刷日志到磁盘，而等master每秒刷新
2. innodb_flush_log_at_trx_commit = 1 表示commit时同步刷到磁盘
3. innodb_flush_log_at_trx_commit = 2 表示commit时异步刷到磁盘（有这个动作，但不是肯定会写入）