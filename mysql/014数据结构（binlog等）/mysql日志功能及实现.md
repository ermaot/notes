- MySQL常见的日志
1. 错误日志:记录 MySQL的启动、停止信息以及在MySQL运行过程中的错误信息
2. 普通日志:记录客户的连接请求和MySQL从客户端收到的SQL语句
3. 慢查询日志:记录查询时间大于MySQL参数long_query_time所设置的值以及那些未使用索引的查询语句
4. 二进制日志:记录所有修改数据的SQL语句,也用于MySQL的复制
- 以上各种日志信息都会被记录到文件中,在默认情况下,这些文件被保存到数据库的目录下
- 从MySQL 5.1.16开始,普通日志和慢查询日志还可以保存到数据库的表中
## 错误日志
- 默认情况下，错误信息会记录到数据库目录下，文件名$hostname.err
- 如果执行了flush error logs会重建新的空文件，旧错误日志会重命名为后缀带old的文件
- log_error参数

```
> show variables like "%log_error%";
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| log_error     | /var/log/mariadb/mariadb.log |
+---------------+------------------------------+
```
- 日志格式：时间 [错误级别] 错误信息内容（错误级别不一定有）
#### 错误日志初始化
1. sql/mysqld.cc的int main(int argc,char **argv)

```
/*对logger对象基本初始化*/
logger.init_base();
```

2. sql/log.cc 中的LOGGER::init_base()

```
/**
进行基本的logger对象初始化,创建基于文件的日志处理器,并进行错误日志处理相关的操作
*/
void LOGGER: :init_ base ()
{
DBUG_ ASSERT(inited == 0) ;
inited= 1 ;
/*
创建-一个文件日志处理器,这里并不对基于table方式的日志处理器进行初始化,原因是系统变量的解析将在后面才进行
*/
if (!file_1og_handler)
file_1og_handler= new Log_to_file_event_handler;
/*默认情况下,错误日志只记录到文件中,所以初始化的参数为LOG_FILE */
init_error_1og (LOG_FILE) ;
file_1og_handler->init_pthread_objects() ;
my_rwlock_init(&LOCK_logger, NULL) ;
```
3. sql/log.cc中的LOGGER::init_error_log
#### 错误日志的记录实现
- 错误日志有3个级别：error、warning和information，在include/mysys.h中有定义

```
enum 1oglevel {
ERROR_LEVEL, //错误级别
WARNING_LEVEL,//警告级别
INFORMATION_LEVEL //信息级别
};
```

## 普通日志
- 普通日志记录MySQL客户连接到MySQL直到从MySQL断开的信息，并包含MySQL服务器收到的每一个SQL语句（无论是否被正确执行）
- 默认情况下，MySQL不记录普通日志，需要设置general_log选项。日志名为$hostname.log

```
> show variables like "%general_log%";
+------------------+----------------------+
| Variable_name    | Value                |
+------------------+----------------------+
| general_log      | OFF                  |
| general_log_file | VM_112_34_centos.log |
+------------------+----------------------+
```
- 从5.1.6开始，普通日志可以记录到文件和数据库表中。参数为log_output，可以为TABLE、FILE、NONE。如果希望同时记录到文件和数据库，可以设置set global log_output='TABLE,FILE';

```
> set global log_output='TABLE,FILE';
Query OK, 0 rows affected (0.00 sec)
> show variables like "%output%";
+---------------+------------+
| Variable_name | Value      |
+---------------+------------+
| log_output    | FILE,TABLE |
+---------------+------------+

> set global log_output='FILE';
Query OK, 0 rows affected (0.00 sec)

> show variables like "%output%";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+
```
- general_log表

```
> show create table mysql.general_log;
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table       | Create Table                                                                                                                                                                                                                                                                                                                                                         |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| general_log | CREATE TABLE `general_log` (
  `event_time` timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `user_host` mediumtext NOT NULL,
  `thread_id` int(11) NOT NULL,
  `server_id` int(10) unsigned NOT NULL,
  `command_type` varchar(64) NOT NULL,
  `argument` mediumtext NOT NULL
) ENGINE=CSV DEFAULT CHARSET=utf8 COMMENT='General log' |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


> select * from mysql.general_log order by event_time limit 1;
+----------------------------+---------------------------+-----------+-----------+--------------+------------------------------------+
| event_time                 | user_host                 | thread_id | server_id | command_type | argument                           |
+----------------------------+---------------------------+-----------+-----------+--------------+------------------------------------+
| 2019-06-04 15:56:56.725080 | root[root] @ localhost [] |     16402 |         0 | Query        | show variables like "%log_output%" |
+----------------------------+---------------------------+-----------+-----------+--------------+------------------------------------+
```
#### 普通日志初始化
- 在sql/log.cc中定义了init_general_log方法来初始化用于记录普通日志的日志处理器:
#### 普通日志实现

方法名 |定义位置 |说明
---|---|---
general_log_print| sql/log.cc|判断是否需要记录日志,并调用LOGGER::general_log_print
general_log_write| sql/log.cc|判断是否需要记录日志,并调用LOGGER::general log write
LOGGER::general_log_print |sql/log.cc|调用已初始化的日志处理器记录普通日志, 该方法的参数是不定的,在对参数进行读取之后将调用LOGGER:general_log_write完成对日志的写入
LOGGER::general_log_write| sql/log.cc| 调用已初始化的日志处理器记录普通日志
Log_to_file_event handler::log_general|sql/log.cc| 将日志内容写入到文件中
Log_to_Csv_event._handler::log general|sql/log.cc |将日志内容写入到数据库表中
## 慢查询日志
- 慢查询日志的实现代码在sql/log.cc中

慢查询日志重要方法

方法名 |定义位置 |说明
---|---|---
slow_log print |sql/log.cc |被定义在sql parse.cc中的dispatch command方法调用记录慢查询日志
LOGGER::slow_log _print |sql/log.cc |计算SQL语句执行时间,获取用户信息(包括主机名和用户名)等.然后调用被初始化的日志处理器记录日志
Log_to_file_event_handler::log_slow |sql/log.cc| 封装对MYSQL QUERY LOG::write的调用
Log_to_csv_event_handler::log_slow |sql/log.cc |将日志记录到数据库表中
MYSQL_QUERY_LOG::write| sql/log.cc| 负责把日志内容写入到文件中

- 与普通日志一样,从MySQL5.1.6开始,慢查询日志可以被同时记录到文件和数据库表中。MySQL分别定义了Log_to_file_event_handler  和 Log_to_csv_event_handler来实现这个功能(慢日志表引擎为csv)
- log_queries_not_using_indexes参数决定慢日志是否记录没有使用index的查询语句，默认未启用
- min_examined_row_limit 默认为0，如果设置了，SQL检查行数未超过该值则不会记录

```
> show variables like "%min_examined_row_limit%";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| min_examined_row_limit | 0     |
+------------------------+-------+
1 row in set (0.00 sec)

show variables like "%not_using%";
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | OFF   |
+-------------------------------+-------+
```

## 二进制日志
- 二进制日志记录所有可能更新数据的SQL语句（比如未匹配到行的delete语句）
- 二进制日志还包含了与执行SQL语句相关的内容，包括SQL执行时间、错误代码
#### 二进制日志功能介绍
- 用于数据恢复和数据复制（集群）
- 默认情况下，MySQL不记录二进制日志，控制参数是log_bin

```
> show variables like "log_bin";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
```
- 设置log_bin=[basename]后，启用二进制日志功能，日志名称为$basebane.0000x；若basename未定义，则使用pid_file参数设置的值作为二进制日志文件的基础名字
- mysqlbinlog可以查看二进制日志
- reset master 可以重置二进制日志
- 二进制记录内容说明

内容|说明
---|---
09071317:20:08 |该SQL语句执行时间
server id 1|执行SQL语句的ServerID,在数据库复制情形下,<p>serverid用于标识每个MySQL服务器.
end_log_pos 199 |表示该条记录的终止位置为199,对应日志文件中的第14行
Query| SQL语句的类型
thread_id=1|发出SQL语句请求的线程ID<p>当每个客户端连接到MySQL服务器时,<p>MySQL会为该用户分配唯一的ID,用以标识该用户的线程
exec_time=0 |SQL语句执行的时间
error_code=0| SQL语句执行的错误代码.0表示SQL语句执行成功

![二进制日志格式样例](0C5B7718E18645E9AA6FB48BDC8470BF)

#### 二进制日志初始化
#### 二进制日志实现