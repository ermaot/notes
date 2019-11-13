## 三个数据库比较

| 名称               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| information_schema | 数据库元信息                                                 |
| performance_schema | 1. performance_schema收集性能相关的数据<br>2.数据库运行中“事件”（函数调用、操作系统等待、SQL执行的阶段、监视内部执行情况），提供存储引擎对磁盘、表IO、表锁等同步调用信息<br>3.当前活跃事件、历史事件、事件摘要等记录信息，可以分析某个事件的次数、时长，进而分析与某个特定线程特定对象的关联活动<br>4.performance_schema存储引擎使用server源代码的“检测点”实现事件收集<br>5.可以用SQL查询或者更新performance_schema中表记录<br>6.set_up开头的配置表更改会立刻生效<br>7.performance_schema保存在内存，重启会消失<br>8.事件监控在各平台皆可用，但稍有差异 |
| performance_schema | 目标：1.不该表server行为变化（线程调度、查询计划等）<br>2.持续不断检测，开销小<br>3.不增加关键字或者语句，不更改解析器<br>4.监测失败不影响server<br>5.事件信息在同时收集和被查询时，优先收集<br>6.容易增添instruments（事件采集配置项）<br>7.instruments代码版本化，新代码不影响旧版本工作 |
| sys schema         | 是一组对象（包括相关视图、存储过程和函数），方便访问performance_schema |



## performance_schema使用

#### 1.查看是否支持performance_schema

```
> select * from information_schema.engines where engine="performance_schema";
+--------------------+---------+--------------------+--------------+------+------------+
| ENGINE             | SUPPORT | COMMENT            | TRANSACTIONS | XA   | SAVEPOINTS |
+--------------------+---------+--------------------+--------------+------+------------+
| PERFORMANCE_SCHEMA | YES     | Performance Schema | NO           | NO   | NO         |
+--------------------+---------+--------------------+--------------+------+------------+
1 row in set (0.00 sec)

> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+

```
#### 2.启用PERFORMANCE_SCHEMA
```
[mysqld]
PERFORMANCE_SCHEMA=on				//只读参数，需重启
```
```
> show variables like "PERFORMANCE_SCHEMA";
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| performance_schema | ON    |
+--------------------+-------+
1 row in set (0.01 sec)
```
查看表
```
> select table_name from information_schema.tables where engine="PERFORMANCE_SCHEMA";
+------------------------------------------------------+
| TABLE_NAME                                           |
+------------------------------------------------------+
| accounts                                             |
| cond_instances                                       |
| data_lock_waits                                      |
……………………
+------------------------------------------------------+
103 rows in set (0.02 sec)

或者
> show tables from performance_schema;
+------------------------------------------------------+
| TABLE_NAME                                           |
+------------------------------------------------------+
| accounts                                             |
| cond_instances                                       |
| data_lock_waits                                      |
……………………
+------------------------------------------------------+
103 rows in set (0.01 sec)
结果一样
```
#### 3.performance_schema表类别

按照事件类型分：

1. 语句事件记录表
事件|说明
---|---
events_statements_current|当前语句事件表
events_statements_history|历史语句事件表
events_statements_history_long|历史长语句事件表
events_statements_histogram_by_digest|details about latency related to schema and query digest
events_statements_histogram_global|global latency summary across all schemas and queries
events_statements_summary_by_account_by_event_name<br>events_statements_summary_by_digest<br>events_statements_summary_by_host_by_event_name<br>events_statements_summary_by_program<br>events_statements_summary_by_thread_by_event_name<br> events_statements_summary_by_user_by_event_name<br>events_statements_summary_global_by_event_name<br>|聚合后的摘要表


2. 等待事件记录表
```
> show tables like "events_wait%";
+-----------------------------------------------+
| Tables_in_performance_schema (events_wait%)   |
+-----------------------------------------------+
| events_waits_current                          |
| events_waits_history                          |
| events_waits_history_long                     |
| events_waits_summary_by_account_by_event_name |
| events_waits_summary_by_host_by_event_name    |
| events_waits_summary_by_instance              |
| events_waits_summary_by_thread_by_event_name  |
| events_waits_summary_by_user_by_event_name    |
| events_waits_summary_global_by_event_name     |
+-----------------------------------------------+
9 rows in set (0.00 sec)
```

3. 阶段事件记录表
```
> show tables like "events_stage%";
+------------------------------------------------+
| Tables_in_performance_schema (events_stage%)   |
+------------------------------------------------+
| events_stages_current                          |
| events_stages_history                          |
| events_stages_history_long                     |
| events_stages_summary_by_account_by_event_name |
| events_stages_summary_by_host_by_event_name    |
| events_stages_summary_by_thread_by_event_name  |
| events_stages_summary_by_user_by_event_name    |
| events_stages_summary_global_by_event_name     |
+------------------------------------------------+
8 rows in set (0.02 sec)
```
4. 事务事件记录表
```
> show tables like "events_transaction%";
+------------------------------------------------------+
| Tables_in_performance_schema (events_transaction%)   |
+------------------------------------------------------+
| events_transactions_current                          |
| events_transactions_history                          |
| events_transactions_history_long                     |
| events_transactions_summary_by_account_by_event_name |
| events_transactions_summary_by_host_by_event_name    |
| events_transactions_summary_by_thread_by_event_name  |
| events_transactions_summary_by_user_by_event_name    |
| events_transactions_summary_global_by_event_name     |
+------------------------------------------------------+
8 rows in set (0.01 sec)
```
5. 监视文件系统层调用表
```
> show tables like "%file%";
+---------------------------------------+
| Tables_in_performance_schema (%file%) |
+---------------------------------------+
| file_instances                        |
| file_summary_by_event_name            |
| file_summary_by_instance              |
+---------------------------------------+
3 rows in set (0.01 sec)
```
6. 监视内存使用表
```
> show tables like "%memory%";
+-----------------------------------------+
| Tables_in_performance_schema (%memory%) |
+-----------------------------------------+
| memory_summary_by_account_by_event_name |
| memory_summary_by_host_by_event_name    |
| memory_summary_by_thread_by_event_name  |
| memory_summary_by_user_by_event_name    |
| memory_summary_global_by_event_name     |
+-----------------------------------------+
5 rows in set (0.01 sec)
```
7. performance_schema动态配置表
```
> show tables like "%setup%";
+----------------------------------------+
| Tables_in_performance_schema (%setup%) |
+----------------------------------------+
| setup_actors                           |
| setup_consumers                        |
| setup_instruments                      |
| setup_objects                          |
| setup_threads                          |
+----------------------------------------+
5 rows in set (0.01 sec)
```
#### 4.performance_schema配置与使用
1. 打开采集项
```
> update setup_instruments set enabled="YES",timed="yes" where name like "wait%";
Query OK, 338 rows affected (0.02 sec)
Rows matched: 390  Changed: 338  Warnings: 0

> update setup_consumers set enabled="YES" where name like "%wait%";
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0
```
2. 查看
```
> select * from events_waits_current limit 1\G
*************************** 1. row ***************************
            THREAD_ID: 13
             EVENT_ID: 3328
         END_EVENT_ID: 3328
           EVENT_NAME: wait/synch/mutex/innodb/buf_dblwr_mutex
               SOURCE: buf0dblwr.cc:899
          TIMER_START: 1211991431581708452
            TIMER_END: 1211991431582830049
           TIMER_WAIT: 1121597
                SPINS: NULL
        OBJECT_SCHEMA: NULL
          OBJECT_NAME: NULL
           INDEX_NAME: NULL
          OBJECT_TYPE: NULL
OBJECT_INSTANCE_BEGIN: 139789646346184
     NESTING_EVENT_ID: NULL
   NESTING_EVENT_TYPE: NULL
            OPERATION: lock
      NUMBER_OF_BYTES: NULL
                FLAGS: NULL
1 row in set (0.00 sec)
```
- TIMER_WAIT单位是皮秒<br>
-  *_current每个线程只保留一个记录<br>
-  *_history每个线程保留10条记录<br>
-  *_history_log总数保留10000条

3. 编译时配置
4. 启动时配置
5. 运行时配置

## information_schema

- sys库有一部分视图来自于information_schema

- information_schema提供了对数据库元数据、统计信息的访问（数据字典或者系统目录）

- 每一个mysql实例都有一个独立的information_schema，每个information_schema包含多个**只读非持久表**（没有对应的数据库文件，不能触发器，不能insert、update、delete）

- mysql5.6有59个表，5.5.56-MariaDB 有62个表，mysql8.0有67个表（原来的数据字典表迁移到了mysql schema 下）
```
> select TABLE_NAME,engine from tables where table_schema="information_schema" order by engine;
+---------------------------------------+--------+
| TABLE_NAME                            | engine |
+---------------------------------------+--------+
| EVENTS                                | Aria   |
| ROUTINES                              | Aria   |
| PROCESSLIST                           | Aria   |
| COLUMNS                               | Aria   |
| PLUGINS                               | Aria   |
| PARTITIONS                            | Aria   |
| PARAMETERS                            | Aria   |
| VIEWS                                 | Aria   |
| TRIGGERS                              | Aria   |
| INNODB_INDEX_STATS                    | MEMORY |
| INNODB_LOCK_WAITS                     | MEMORY |
| ENGINES                               | MEMORY |
| INNODB_SYS_INDEXES                    | MEMORY |
| REFERENTIAL_CONSTRAINTS               | MEMORY |
| INNODB_SYS_TABLESTATS                 | MEMORY |
| COLUMN_PRIVILEGES                     | MEMORY |
| INNODB_SYS_FOREIGN                    | MEMORY |
| PROFILING                             | MEMORY |
| INNODB_SYS_STATS                      | MEMORY |
| INNODB_BUFFER_PAGE                    | MEMORY |
| INNODB_CMPMEM                         | MEMORY |
| INNODB_BUFFER_POOL_STATS              | MEMORY |
| INNODB_UNDO_LOGS                      | MEMORY |
| COLLATION_CHARACTER_SET_APPLICABILITY | MEMORY |
| INNODB_SYS_COLUMNS                    | MEMORY |
| INNODB_BUFFER_PAGE_LRU                | MEMORY |
| INNODB_RSEG                           | MEMORY |
| COLLATIONS                            | MEMORY |
| INNODB_SYS_FIELDS                     | MEMORY |
| INNODB_SYS_FOREIGN_COLS               | MEMORY |
| INNODB_CMPMEM_RESET                   | MEMORY |
| CLIENT_STATISTICS                     | MEMORY |
| INNODB_SYS_TABLES                     | MEMORY |
| INNODB_TABLE_STATS                    | MEMORY |
| CHARACTER_SETS                        | MEMORY |
| INNODB_BUFFER_POOL_PAGES_BLOB         | MEMORY |
| XTRADB_ADMIN_COMMAND                  | MEMORY |
| USER_STATISTICS                       | MEMORY |
| INNODB_LOCKS                          | MEMORY |
| USER_PRIVILEGES                       | MEMORY |
| INNODB_BUFFER_POOL_PAGES_INDEX        | MEMORY |
| TABLESPACES                           | MEMORY |
| INNODB_TRX                            | MEMORY |
| KEY_COLUMN_USAGE                      | MEMORY |
| TABLES                                | MEMORY |
| KEY_CACHES                            | MEMORY |
| STATISTICS                            | MEMORY |
| TABLE_STATISTICS                      | MEMORY |
| INNODB_BUFFER_POOL_PAGES              | MEMORY |
| SESSION_VARIABLES                     | MEMORY |
| TABLE_PRIVILEGES                      | MEMORY |
| INNODB_CHANGED_PAGES                  | MEMORY |
| INDEX_STATISTICS                      | MEMORY |
| SESSION_STATUS                        | MEMORY |
| TABLE_CONSTRAINTS                     | MEMORY |
| INNODB_CMP_RESET                      | MEMORY |
| GLOBAL_VARIABLES                      | MEMORY |
| SCHEMA_PRIVILEGES                     | MEMORY |
| INNODB_CMP                            | MEMORY |
| GLOBAL_STATUS                         | MEMORY |
| FILES                                 | MEMORY |
| SCHEMATA                              | MEMORY |
+---------------------------------------+--------+
62 rows in set (0.00 sec)
```
#### information_schema组成对象
##### 1.server层统计信息字典表
表名|类型|说明
---|---|---
columns|表的字段信息|innodb临时表
KEY_COLUMN_USAGE|索引列的约束条件信息|memory临时表<br>（mysql8是基于mysql shema表的视图）
REFERENTIAL_CONSTRAINTS|外键约束信息|memory临时表<br>（mysql8是基于mysql shema表的视图）
STATISTICS|关于索引的统计信息，一个索引一行|memory临时表<br>（mysql8是基于mysql shema表的视图）
TABLE_CONSTRAINTS|与表相关的约束的信息|memory临时表<br>（mysql8是基于mysql shema表的视图）
FILES|提供mysql数据表空间文件相关信息|memory临时表<br>（mysql8是基于mysql shema表的视图）
ENGINES|支持的引擎|memory临时表
TABLESPACES|活跃表空间相关信息（主要是NDB，不提供innodb）<br>innodb查询INNODB_TABLESPACES和INNODB_DATAFILES<br>（INNODB_SYS_TABLESPACES和INNODB_SYS_DATAFILES（mysql5.6））|memory临时表
SCHEMATA|数据库列表信息，一个SCHEMATA即一个数据库|memory临时表<br>（mysql8是基于mysql shema表的视图）

##### 2.server层的表级别对象字典表
表名|类型|说明
---|---|---
VIEWS|视图信息|innodb临时表<br>（mysql8基于mysql shema表的视图）
TRIGGERS|触发器信息|innodb临时表<br>（mysql8基于mysql shema表的视图）
TABLES|表相关信息|memory临时表<br>（mysql8是基于mysql shema表的视图）
ROUTINES|存储过程与存储函数信息<br>与mysql.proc信息对应|innodb临时表<br>（mysql8基于mysql shema表的视图）
PARTITIONS|分区表信息|innodb临时表<br>（mysql8基于mysql shema表的视图）
EVENTS|计划任务相关信息|innodb临时表<br>（mysql8基于mysql shema表的视图）
PARAMETERS|存储过程和函数的参数信息<br>与mysql.proc表的param_list对应|innodb临时表<br>（mysql8基于mysql shema表的视图）

##### 3.server层的混杂信息字典表

表名|类型|说明
---|---|---
GLOBAL_STATUS<br>GLOBAL_VARIABLES<br>SESSION_STATUS<BR>SESSION_VARIABLES|提供全局、会话级别的状态变量与系统变量信息|memory临时表<br>（mysql8是基于mysql shema表的视图）<br>mysql8无此4表
OPTIMIZER_TRACE|跟踪优化程序功能<br>默认关闭<br>跟踪当前会话的最后一条语句|innodb临时表
PLUGINS|支持的插件|innodb临时表
processlist|线程运行中的状态|innodb临时表
PROFILING|语句性能分析的信息，对应于SHOW PROFILE和SHOW PROFILES。只有在profiling=1才记录。mysql5.7.2开始不推荐使用|memory临时表<br>
CHARACTER_SETS|支持的字符集|memory临时表<br>（mysql8基于mysql shema表的视图）
COLLATIONS|校对规则|memory临时表<br>（mysql8基于mysql shema表的视图）
COLLATION_CHARACTER_SET_APPLICABILITY|字符集适用的校对规则，似无大用|memory临时表<br>（mysql8基于mysql shema表的视图）
COLUMN_PRIVILEGES|字段权限信息，来自mysql.column_priv|memory临时表
SCHEMA_PRIVILEGES|库级别权限信息，来自mysql.db表|memory临时表
TABLE_PRIVILEGES|表级别权限信息，来自mysql.table_priv|memory临时表
USER_PRIVILEGES|全局权限信息，来自mysql.user|memory临时表

##### 4.innodb层的系统字典表
表名|类型|说明
---|---|---
INNODB_SYS_TABLES<br>INNODB_TABLES（mysql8）|所有表空间文件的元数据，等同于innodb数据字典内部sys_datafiles|memory临时表
INNODB_SYS_VIRTUAL/INNODB_VIRTUAL|虚拟列信息，来自innodb数据字典内部sys_virtual|memory临时表


##### 5.innodb层的锁、事务、统计信息字典表

##### 6.innodb层的全文索引字典表

##### 7.innodb层的压缩相关字典表


## sys库
#### 1.sys库基础环境

- mysql5.6以及更高版本
- 需启用performance_schema
- 权限
- 需启用需启用performance_schema的一些instruments和consumers
1. 所有wait instruments：call sys.ps_setup_enable_instrument('wait')
2. 所有stage instruments：call sys.ps_setup_enable_instrument('stage')
3. 所有staement instruments：call sys.ps_setup_enable_instrument('statement')
4. 所有事件类型的current表：call sys.ps_setup_enable_consumer('current')
5. 所有事件类型的history_long表：call sys.ps_setup_enable_consumer('history_long')
- performance_schema默认配置可以满足sys大部分的数据收集功能，启用上述instruments和consumers会对性能产生一定影响（call sys.ps_setup_reset_to——default(TRUE)快速重置）

- 概念：

1. instruments：生产者，用于采集MySQL 中各种各样的操作产生的事件信息，对应配置表中的配置项我们可以称为监控采集配置项，以下提及生产者均统称为instruments
2. consumers：消费者，对应的消费者表用于存储来自instruments采集的数据，对应配置表中的配置项我们可以称为消费存储配置项，以下提及消费者均统称为consumers

#### 2.sys初体验
```
> use sys

> select * from version;
+-------------+---------------+
| sys_version | mysql_version |
+-------------+---------------+
| 2.1.0       | 8.0.17-debug  |
+-------------+---------------+
1 row in set (0.00 sec)
```
- 带X$与不带x$的视图成对出现，数据相同，但不带x$的表的数据是经过转换的，容易理解
- 可以访问https://github.com/mysql/mysql-sys上各个sql文件查看对象的定义

#### 3.sys系统库的进度报告功能

- 通过processlist和session以及带x$前缀的视图（x$processlist和x$session）查看
- processlist包含后台和前台线程的当前事件信息
- session视图直接调用processlist，过滤了后台线程和command为Deamon的线程
- processlist基于performance_schema下的threads、events_waits_current、events_stages_current、events_statements_current、events_transactions_current、session_connect_attrs、sys.x$memory_by_thread_by_current_bytes，需打开对应的instruments和consumers，否则对应字段为null
- trx_state字段为ACTIVE的线程，progress可以输出百分比进度信息
- 对于stage事件进度报告，必需启用events_stages_current以及与进度相关的instruments

#### 4.sys库配置
```
> select * from sys_config;
+--------------------------------------+-------+---------------------+--------+
| variable                             | value | set_time            | set_by |
+--------------------------------------+-------+---------------------+--------+
| diagnostics.allow_i_s_tables         | OFF   | 2019-08-04 21:33:56 | NULL   |
| diagnostics.include_raw              | OFF   | 2019-08-04 21:33:56 | NULL   |
| ps_thread_trx_info.max_length        | 65535 | 2019-08-04 21:33:56 | NULL   |
| statement_performance_analyzer.limit | 100   | 2019-08-04 21:33:56 | NULL   |
| statement_performance_analyzer.view  | NULL  | 2019-08-04 21:33:56 | NULL   |
| statement_truncate_len               | 64    | 2019-08-04 21:33:56 | NULL   |
+--------------------------------------+-------+---------------------+--------+
6 rows in set (0.00 sec)
```
字段|说明
---|---
variable|配置项名称
value|配置项的值
set_time|该配置最近的修改时间
set_by|对配置修改的账户名；如果sys库安装后从未被修改，则为null
- 为了减少对sys_config的读取，系统需要配置项时会先检查对应的自定义配置选项变量
- 如果没有设置，则会读取sys_config同时更新到用户自定义变量中
举例：
```
sys.statement_truncate_len默认等于64
> select format_statement("select variable,value,set_time,set_by from sys_config");
+---------------------------------------------------------------------------+
| format_statement("select variable,value,set_time,set_by from sys_config") |
+---------------------------------------------------------------------------+
| select variable,value,set_time,set_by from sys_config                     |
+---------------------------------------------------------------------------+
1 row in set (0.01 sec)

> set @sys.statement_truncate_len=32;
Query OK, 0 rows affected (0.00 sec)

//可以看到语句显示的时候被截断了
> select format_statement("select variable,value,set_time,set_by from sys_config");
+---------------------------------------------------------------------------+
| format_statement("select variable,value,set_time,set_by from sys_config") |
+---------------------------------------------------------------------------+
| select variabl ... rom sys_config                                         |
+---------------------------------------------------------------------------+
1 row in set (0.00 sec)
```