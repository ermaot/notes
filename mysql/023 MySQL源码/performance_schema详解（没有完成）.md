http://blog.itpub.net/26736162/viewspace-2651257/


## MySQL的performance schema 用于监控MySQL server在一个较低级别的运行过程中的资源消耗、资源等待等情况，它具有以下特点
- 提供了一种在数据库运行时实时检查server的内部执行情况的方法。performance_schema 数据库中的表使用==performance_schema存储引擎==。该数据库主要关注数据库运行过程中的性能相关的数据，与information_schema不同，information_schema主要关注server运行过程中的元数据信息
- performance_schema通过监视server的事件来实现监视server内部运行情况， “事件”就是server内部活动中所做的任何事情以及对应的时间消耗，利用这些信息来判断server中的相关资源消耗在了哪里。一般来说，事件可以是函数调用、操作系统的等待、SQL语句执行的阶段（如sql语句执行过程中的parsing 或 sorting阶段）或者整个SQL语句与SQL语句集合。事件的采集可以方便的提供server中的相关存储引擎对磁盘文件、表I/O、表锁等资源的同步调用信息。
- performance_schema中的事件与写入二进制日志中的事件（描述数据修改的events）、事件计划调度程序（这是一种存储程序）的事件不同。performance_schema中的事件记录的是server执行某些活动对某些资源的消耗、耗时、这些活动执行的次数等情况。
- performance_schema中的事件只记录在本地server的performance_schema中，其下的这些表中数据发生变化时不会被写入binlog中，也不会通过复制机制被复制到其他server中。
- 当前活跃事件、历史事件和事件摘要相关的表中记录的信息。能提供某个事件的执行次数、使用时长。进而可用于分析某个特定线程、特定对象（如mutex或file）相关联的活动。 
- PERFORMANCE_SCHEMA存储引擎使用server源代码中的“检测点”来实现事件数据的收集。对于performance_schema实现机制本身的代码没有相关的单独线程来检测，这与其他功能（如复制或事件计划程序）不同
- 收集的事件数据存储在performance_schema数据库的表中。这些表可以使用SELECT语句查询，也可以使用SQL语句更新performance_schema数据库中的表记录（如动态修改performance_schema的setup_*开头的几个配置表，但要注意：配置表的更改会立即生效，这会影响数据收集） 
- ==performance_schema的表中的数据不会持久化存储在磁盘中，而是保存在内存中，一旦服务器重启，这些数据会丢失==（包括配置表在内的整个performance_schema下的所有数据） 
- MySQL支持的所有平台中事件监控功能都可用，但不同平台中用于统计事件时间开销的计时器类型可能会有所差异。

## performance_schema实现机制遵循以下设计目标
- 启用performance_schema不会导致server的行为发生变化。例如，它不会改变线程调度机制，不会导致查询执行计划（如EXPLAIN）发生变化
- 启用performance_schema之后，server会持续不间断地监测，开销很小，不会导致server不可用
- 在该实现机制中没有增加新的关键字或语句，解析器不会变化
- 即使performance_schema的监测机制在内部对某事件执行监测失败，也不会影响server正常运行
- 如果在开始收集事件数据时碰到有其他线程正在针对这些事件信息进行查询，那么查询会优先执行事件数据的收集，因为事件数据的收集是一个持续不断的过程，而检索(查询)这些事件数据仅仅只是在需要查看的时候才进行检索。也可能某些事件数据永远都不会去检索
- 需要很容易地添加新的instruments监测点
- instruments(事件采集项)代码版本化：如果instruments的代码发生了变更，旧的instruments代码还可以继续工作。 
- MySQL sys schema是一组对象（包括相关的视图、存储过程和函数），可以方便地访问performance_schema收集的数据。同时检索的数据可读性也更高(例如：performance_schema中的时间单位是皮秒，经过sys schema查询时会转换为可读的us,ms,s,min,hour,day等单位)，sys schem在5.7.x版本默认安装


## performance_schema使用快速入门
#### 检查当前数据库版本是否支持
==performance_schema被视为存储引擎==。如果该引擎可用，则应该在INFORMATION_SCHEMA.ENGINES表或SHOW ENGINES语句的输出中都可以看到它的SUPPORT值为YES
```
> SELECT * FROM INFORMATION_SCHEMA.ENGINES WHERE ENGINE =  'PERFORMANCE_SCHEMA';
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
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)



> show engine PERFORMANCE_SCHEMA status;
+--------------------+-------------------------------------------------------------+-----------+
| Type               | Name                                                        | Status    |
+--------------------+-------------------------------------------------------------+-----------+
| performance_schema | events_waits_current.size                                   | 176       |
| performance_schema | events_waits_current.count                                  | 1536      |
| performance_schema | events_waits_history.size                                   | 176       |
| performance_schema | events_waits_history.count                                  | 2560      |
| performance_schema | events_waits_history.memory                                 | 450560    |
| performance_schema | events_waits_history_long.size                              | 176       |
| performance_schema | events_waits_history_long.count                             | 10000     |

……………………


> show variables like "performance_schema";
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| performance_schema | ON    |
+--------------------+-------+
1 row in set (0.01 sec)
```
#### 启用performance_schema（如果没有默认启用的话）
```
[mysqld]
performance_schema = ON
# 注意：该参数为只读参数，需要在实例启动之前设置才生效
```
查看是否成功启用
```
> show variables like "performance_schema";
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| performance_schema | ON    |
+--------------------+-------+
1 row in set (0.01 sec)
```

## 表介绍
在 MySQL 5.7.17 版本中，performance_schema 下一共有87张表；MySQL8有103张
#### 按照事件类型分组记录性能事件数据的表：
- 当前语句事件表events_statements_current、
- 历史语句事件表events_statements_history
- 长语句历史事件表events_statements_history_long
- 以及聚合后的摘要表summary，其中，summary表还可以根据帐号(account)，主机(host)，程序(program)，线程(thread)，用户(user)和全局(global)再进行细分)
```
> show tables like "events_statement%";
+----------------------------------------------------+
| Tables_in_performance_schema (events_statement%)   |
+----------------------------------------------------+
| events_statements_current                          |
| events_statements_histogram_by_digest              |
| events_statements_histogram_global                 |
| events_statements_history                          |
| events_statements_history_long                     |
| events_statements_summary_by_account_by_event_name |
| events_statements_summary_by_digest                |
| events_statements_summary_by_host_by_event_name    |
| events_statements_summary_by_program               |
| events_statements_summary_by_thread_by_event_name  |
| events_statements_summary_by_user_by_event_name    |
| events_statements_summary_global_by_event_name     |
+----------------------------------------------------+
12 rows in set (0.01 sec)
```
#### 等待事件记录表
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
9 rows in set (0.01 sec)
```

#### 阶段事件记录表

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
8 rows in set (0.01 sec)
```
记录语句执行的阶段事件的表，与语句事件类型的相关记录表类似：

#### 事务事件记录表
记录事务相关的事件的表，与语句事件类型的相关记录表类似
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

#### 监视文件系统层调用的表
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

#### 监视内存使用的表

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
#### 动态对performance_schema进行配置的配置表
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

## performance_schema配置与使用
数据库刚刚初始化并启动时，并非所有instruments(事件采集项，在采集项的配置表中每一项都有一个开关字段，或为YES，或为NO)和consumers(与采集项类似，也有一个对应的事件类型保存表配置项，为YES就表示对应的表保存性能数据，为NO就表示对应的表不保存性能数据)都启用了，所以检测的事件需要进行设置。
#### 以配置监测等待事件数据为例
1. 打开等待事件的采集器配置项开关，需要修改setup_instruments配置表中对应的采集器配置项
```
> update  setup_instruments SET ENABLED = 'YES', TIMED = 'YES'  where name like "%wait%";
Query OK, 392 rows affected (0.02 sec)
Rows matched: 444  Changed: 392  Warnings: 0
```
2. 打开等待事件的保存表配置开关，修修改setup_consumers 配置表中对应的配置
```
UPDATE setup_consumers SET ENABLED = 'YES' where name like '%wait%';
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0
```
3. 通过查询events_waits_current查看server当前事件
```
> SELECT * FROM events_waits_current limit 1;
+-----------+----------+--------------+-----------------------------------------+------------------+-------------------+-------------------+------------+-------+---------------+-------------+------------+-------------+-----------------------+------------------+--------------------+-----------+-----------------+-------+
| THREAD_ID | EVENT_ID | END_EVENT_ID | EVENT_NAME                              | SOURCE           | TIMER_START       | TIMER_END         | TIMER_WAIT | SPINS | OBJECT_SCHEMA | OBJECT_NAME | INDEX_NAME | OBJECT_TYPE | OBJECT_INSTANCE_BEGIN | NESTING_EVENT_ID | NESTING_EVENT_TYPE | OPERATION | NUMBER_OF_BYTES | FLAGS |
+-----------+----------+--------------+-----------------------------------------+------------------+-------------------+-------------------+------------+-------+---------------+-------------+------------+-------------+-----------------------+------------------+--------------------+-----------+-----------------+-------+
|        13 |      552 |          552 | wait/synch/mutex/innodb/buf_dblwr_mutex | buf0dblwr.cc:899 | 97223830373297024 | 97223830374046894 |     749870 |  NULL | NULL          | NULL        | NULL       | NULL        |       140163040065256 |             NULL | NULL               | lock      |            NULL |  NULL |
+-----------+----------+--------------+-----------------------------------------+------------------+-------------------+-------------------+------------+-------+---------------+-------------+------------+-------------+-----------------------+------------------+--------------------+-----------+-----------------+-------+
1 row in set (0.01 sec)
```
*_ID列表示事件来自哪个线程、事件编号是多少；EVENT_NAME表示检测到的具体的内容；SOURCE表示这个检测代码在哪个源文件中以及行号；计时器字段TIMER_START、TIMER_END、TIMER_WAIT分别表示该事件的开始时间、结束时间、以及总的花费时间，如果该事件正在运行而没有结束，那么TIMER_END和TIMER_WAIT的值显示为NULL。注：计时器统计的值是近似值，并不是完全精确

## 启动选项
这些启动选项是用于指定consumers和instruments配置项在MySQL启动时是否跟随打开的，需要在mysqld启动时就需要通过命令行指定或者需要在my.cnf中指定，启动之后通过show variables命令无法查看，因为他们不属于system variables

#### statement选项
- performance_schema_consumer_events_statements_current=TRUE
1. 是否在mysql server启动时就开启events_statements_current表的记录功能(该表记录当前的语句事件信息)
2. 启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新setup_consumers配置表中的events_statements_current配置项，默认值为TRUE
- performance_schema_consumer_events_statements_history=TRUE
与 performance_schema_consumer_events_statements_current选项类似，但该选项是用于配置是否记录语句事件短历史信息，默认为TRUE
- performance_schema_consumer_events_stages_history_long=FALSE
1. 与 performance_schema_consumer_events_statements_current选项类似，但该选项是用于配置是否记录语句事件长历史信息，默认为FALSE
#### 其他events选项
除了statement(语句)事件之外，还支持：wait(等待)事件、stage(阶段)事件、transaction(事务)事件，==他们与statement事件一样都有三个启动项分别进行配置==，但这些等待事件默认未启用，如果需要在MySQL Server启动时一同启动，则通常需要写进my.cnf配置文件中

#### 其他选项
- performance_schema_consumer_global_instrumentation=TRUE
是否在MySQL Server启动时就开启全局表（如：mutex_instances、rwlock_instances、cond_instances、file_instances、users、hostsaccounts、socket_summary_by_event_name、file_summary_by_instance等大部分的全局对象计数统计和事件汇总统计信息表）的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新全局配置项
- performance_schema_consumer_statements_digest=TRUE
是否在MySQL Server启动时就开启events_statements_summary_by_digest表的记录功能，启动之后也可以在 setup_consumers 表中使用UPDATE语句进行动态更新digest配置项
- performance_schema_consumer_thread_instrumentation=TRUE
是否在MySQL Server启动时就开启events_xxx_summary_by_yyy_by_event_name表的记录功能，启动之后也可以在 setup_consumers表中使用UPDATE语句进行动态更新线程配置项默认值为TRUE
- performance_schema_instrument[=name]
是否在MySQL Server启动时就启用某些采集器，由于instruments配置项多达数千个，所以该配置项支持key-value模式，还支持%号进行通配等，如下:
```
# [=name]可以指定为具体的Instruments名称（但是这样如果有多个需要指定的时候，就需要使用该选项多次），也可以使用通配符，可以指定instruments相同的前缀+通配符，也可以使用%代表所有的instruments
## 指定开启单个instruments
--performance-schema-instrument='instrument_name=value'
## 使用通配符指定开启多个instruments
--performance-schema-instrument='wait/synch/cond/%=COUNTED'
## 开关所有的instruments
--performance-schema-instrument='%=ON'
--performance-schema-instrument='%=OFF'
```
这些启动选项要生效的前提是，需要设置performance_schema=ON。另外，这些启动选项虽然无法使用show variables语句查看，但我们可以通过setup_instruments和setup_consumers表查询这些选项指定的值

## 运行时配置的选项
#### performance_timers表
performance_timers表中记录了server中有哪些可用的事件计时器（注意：该表中的配置项不支持增删改，是只读的。有哪些计时器就表示当前的版本支持哪些计时器），setup_timers配置表中的配置项引用此表中的计时器
```
> select * from performance_timers;
+-------------+-----------------+------------------+----------------+
| TIMER_NAME  | TIMER_FREQUENCY | TIMER_RESOLUTION | TIMER_OVERHEAD |
+-------------+-----------------+------------------+----------------+
| CYCLE       |      2491621890 |                1 |             18 |
| NANOSECOND  |      1000000000 |                1 |             60 |
| MICROSECOND |         1000000 |                1 |            148 |
| MILLISECOND |            1036 |                1 |             76 |
+-------------+-----------------+------------------+----------------+
4 rows in set (0.00 sec)
```
1. TIMER_NAME：表示可用计时器名称，CYCLE是指基于CPU（处理器）周期计数器的定时器。在 setup_timers 表中可以使用 performance_timers 表中列值不为null的计时器（如果 performance_timers 表中有某字段值为 NULL ，则表示该定时器可能不支持当前 server 所在平台）
2. TIMER_FREQUENCY：表示每秒钟对应的计时器单位的数量（即，相对于每秒时间换算为对应的计时器单位之后的数值，例如：每秒= 1000 毫秒= 1000000 微秒= 1000000000 纳秒）。对于 CYCLE 计时器的换算值，通常与 CPU 的频率相关。对于 performance_timers 表中查看到的 CYCLE 计时器的 TIMER_FREQUENCY 列值 ，是根据 2.4GHz 处理器的系统上获得的预设值（在 2.4GHz 处理器的系统上， CYCLE 可能接近 2400000000 ）。 NANOSECOND 、MICROSECOND 、MILLISECOND  计时器是基于固定的1秒换算而来。对于 TICK 计时器， TIMER_FREQUENCY列值可能会因平台而异（例如，某些平台使用100个tick/秒，某些平台使用1000个tick/秒）
3. TIMER_RESOLUTION：计时器精度值，表示在每个计时器被调用时额外增加的值（即使用该计时器时，计时器被调用一次，需要额外增加的值）。如果计时器的分辨率为10，则其计时器的时间值在计时器每次被调用时，相当于TIMER_FREQUENCY值+10
4. TIMER_OVERHEAD：表示在使用定时器获取事件时开销的最小周期值（performance_schema 在初始化期间调用计时器20次，选择一个最小值作为此字段值），每个事件的时间开销值是计时器显示值的两倍，因为在事件的开始和结束时都调用计时器。注意：计时器代码仅用于支持计时事件，对于非计时类事件（如调用次数的统计事件），这种计时器统计开销方法不适用

PS：对于 performance_timers 表，不允许使用 TRUNCATE TABLE 语句

#### setup_timers 表
==这个表在mysql8中没有==，mariadb5.6有
- setup_timers表中记录当前使用的事件计时器信息（注意：该表不支持增加和删除记录，只支持修改和查询），可以通过UPDATE语句来更改setup_timers.TIMER_NAME列值，以给不同的事件类别选择不同的计时器， setup_timers.TIMER_NAME 列有效值来自performance_timers.TIMER_NAME 列值。
- 对 setup_timers 表的修改会立即影响监控。正在执行的事件可能会使用修改之前的计时器作为开始时间，但可能会使用修改之后的新的计时器作为结束时间，为了避免计时器更改后可能产生时间信息收集到不可预测的结果，请在修改之后使用 TRUNCATE TABLE 语句来重置 performance_schema 中相关表中的统计信息
```
> SELECT * FROM setup_timers;
+------+------------+
| NAME | TIMER_NAME |
+------+------------+
| wait | CYCLE      |
+------+------------+
1 row in set (0.00 sec)
```
PS：对于 setup_timers 表，不允许使用 TRUNCATE TABLE 语句
#### setup_consumers 表
setup_consumers表列出了consumers可配置列表项(注意：该表不支持增加和删除记录，只支持修改和查询)，如下
```
> SELECT * FROM setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| events_stages_current            | NO      |
| events_stages_history            | NO      |
| events_stages_history_long       | NO      |
| events_statements_current        | YES     |
| events_statements_history        | YES     |
| events_statements_history_long   | NO      |
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | NO      |
| events_waits_current             | YES     |
| events_waits_history             | YES     |
| events_waits_history_long        | YES     |
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| statements_digest                | YES     |
+----------------------------------+---------+
15 rows in set (0.00 sec)
```

## 一些参数

参数|说明
---|---
performance_schema_digests_size=10000|控制events_statements_summary_by_digest表中的最大行数。如果产生的语句摘要信息超过此最大值，便无法继续存入该表，此时performance_schema会增加状态变量
performance_schema_events_statements_history_long_size|控制events_statements_history_long表中的最大行数，该参数控制所有会话在events_statements_history_long表中能够存放的总事件记录数，超过这个限制之后，最早的记录将被覆盖。全局变量，只读变量
performance_schema_events_statements_history_size=10|控制events_statements_history表中单个线程（会话）的最大行数，该参数控制单个会话在events_statements_history表中能够存放的事件记录数，超过这个限制之后，单个会话最早的记录将被覆盖，全局变量，只读变量
performance_schema_max_digest_length=1024|用于控制标准化形式的SQL语句文本在存入performance_schema时的限制长度，该变量与max_digest_length变量相关
performance_schema_max_sql_text_length=1024|控制存入events_statements_current，events_statements_history和events_statements_history_long语句事件表中的SQL_TEXT列的最大SQL长度字节数。 超出系统变量performance_schema_max_sql_text_length的部分将被丢弃，不会记录，一般情况下不需要调整该参数，除非被截断的部分与其他SQL比起来有很大差异。全局变量，只读变量

```
performance_schema_digests_size=10000
/*

*/

/*
，整型值，5.6.3版本引入 * 5.6.x版本中，5.6.5及其之前的版本默认为10000，5.6.6及其之后的版本默认值为-1，通常情况下，自动计算的值都是10000 * 5.7.x版本中，默认值为-1，通常情况下，自动计算的值都是10000
*/

/*
，整型值，5.6.3版本引入 * 5.6.x版本中，5.6.5及其之前的版本默认为10，5.6.6及其之后的版本默认值为-1，通常情况下，自动计算的值都是10 * 5.7.x版本中，默认值为-1，通常情况下，自动计算的值都是10
除了statement(语句)事件之外，wait(等待)事件、state(阶段)事件、transaction(事务)事件，他们与statement事件一样都有三个参数分别进行存储限制配置，有兴趣的同学自行研究，这里不再赘述
*/

/*
(max_digest_length变量含义请自行查阅相关资料)
全局变量，只读变量，默认值1024字节，整型值，取值范围0~1048576
*/

/*
，整型值，默认值为1024字节，取值范围为0~1048576，5.7.6版本引入
降低系统变量performance_schema_max_sql_text_length值可以减少内存使用，但如果汇总的SQL中，被截断部分有较大差异，会导致没有办法再对这些有较大差异的SQL进行区分。 增加该系统变量值会增加内存使用，但对于汇总SQL来讲可以更精准地区分不同的部分。
*/

```

