GTID，全局事务标识符，从MySQL5.6推出的主从复制的重量级特性，有如下便利性
1. 根据GTID快速知道事务最初在哪个实例上提交
2. 基于GTID搭建主从复制更加简单，确保每个事务只执行一次
3. 基于GTID，可以方便实现replication的failover，不用像传统模式复制那样找master_log_file和master_log_pos
4. MySQL Group Replication节点间的复制完全依赖GTID。并且在Group Replication集群节点进行Recovery重新加入到集群中的操作，会选择一个节点作为Donor，然后基于Pruged的GTID开始同步数据
5. MGR中集群使用GTID标记事务，或者叫冲突验证，用于跟踪每个实例上提交的事务，确定哪些事务可能有冲突
6. GTID让每一个事务在集群中有了秩序
## GTID相关概念
#### 什么是GTID
GTID是在复制环境和源MySQL实例中的全局唯一的标识符，格式如下
```
GTID=source_id:sequence_id
```
- source_id是源服务器的唯一标识，通常用服务器的server_uuid表示
- sequence_id是在事务提交时由系统顺序分配的一个序列号，相同的source_id下sequence_id在binlog文件中是递增且连续有序
```
> show master status\G
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 3352
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```
## GTID 生命周期
- master产生GTID：master中执行一个事务，会产生一个GTID信息，并保存到binlog中
- 发送binlog信息到从库上：将二进制日志信息发送到slave，并且存储在relay log中，slave读取GTID并设置其GTID_NEXT值为该GTID值，并且告知slave必须使用此GTID记录下一个事务
- slave执行GTID：slave首先验证是否已经在二进制日志中使用了该GTID号，如果没有就写入GTID应用其事务，并将事务写入二进制日志。提交事务前，slave还要保证slave没有应用具有该GTID的 事务，还要保证没有其他会话已经读取了该GTID但尚未提交
- slave不生成GTID：由于GTID_next不为空，slave不会尝试为该事务生成新GTID，而是从gtid_next读取GTID并写入二进制日志。
#### gtid_executed表
从5.75版本开始，mysql库下新增了gtid_executed表
```
> desc mysql.gtid_executed;
+----------------+------------+------+-----+---------+-------+
| Field          | Type       | Null | Key | Default | Extra |
+----------------+------------+------+-----+---------+-------+
| source_uuid    | char(36)   | NO   | PRI | NULL    |       |
| interval_start | bigint(20) | NO   | PRI | NULL    |       |
| interval_end   | bigint(20) | NO   |     | NULL    |       |
+----------------+------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

```
> show variables like "%gtid_mode%";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```
- 只有当gtid_mode为on或者on_permissive时，GTID才会保存在mysql.gtid_executed表中，不考虑是否启用了二进制日志
1. 未启用binlog时，每个事务都会记录到gtid_executed中
2. 启动binlog时，每个事务不仅会记录到grtid_executed表中，而且当binlog rotate或者服务器关闭时，服务器会将gtid信息写入新的二进制日志。如果服务器异常关闭，GTID不会被存入mysql.gtid_executed中，恢复是将这些GTID加入表中，并写入到gtid_executed系统变量中
3. reset master会情况gtid_executed表

#### gtid_executed表压缩
- 可以通过事务的间隔来代替原来每一个GTID信息
```
> select * from mysql.gtid_executed;
+----------------------------------+----------------+--------------+
| source_uuid                      | interval_start | interval_end |
+----------------------------------+----------------+--------------+
| 26334-33234-2344-2345-234edfewww |              1 |            10 |
+----------------------------------+----------------+--------------+
1 row in set (0.00 sec)
```
- MySQL启用GTID时，服务器会定期对mysql.gtid_executed执行此类型的压缩，可以通过设置gtid_executed_compression_period变量控制压缩表之前允许的事务数
- MySQL有一个单独的后台线程执行gtid_executed表压缩
```
> select * from performance_schema.threads where name like "%gtid%"\G
*************************** 1. row ***************************
          THREAD_ID: 26
               NAME: thread/sql/compress_gtid_table
               TYPE: FOREGROUND
     PROCESSLIST_ID: 1
   PROCESSLIST_USER: NULL
   PROCESSLIST_HOST: NULL
     PROCESSLIST_DB: NULL
PROCESSLIST_COMMAND: Daemon
   PROCESSLIST_TIME: 434009
  PROCESSLIST_STATE: Suspending
   PROCESSLIST_INFO: NULL
   PARENT_THREAD_ID: 1
               ROLE: NULL
       INSTRUMENTED: YES
            HISTORY: YES
    CONNECTION_TYPE: NULL
       THREAD_OS_ID: 23741
1 row in set (0.00 sec)
```
## GTID搭建 主从
#### 主从搭建的参数
参数|说明
---|---
server_id|设置 MySQL实例的 server id,每个实例的 server_id不能一样
gtid_mode=ON| MySQL实例开启GTD模式.
enforce_gtid_consitency=ON|使用GTD模式复制时,需要开启此参数,用来保证GTID的一致性.
log-bin| MySQL必须开启 Binlog.
log-slave-updates=1|决定 SLAVE从 MASTER接收到的更新且执行完之后,执行的 Binlog是否记录到 SLAVE的 Binlog中,建议开启.
binlog_format=ROW|强烈建议 binlog_ format使用ROW格式,其他格式可能造成数据不一致.
skip-slave-start=1|当 SLAVE数据库启动的时候, SLAVE不会自动开启复制.

#### 开启GTID
- 如果数据库已经启动，在MySQL5.7.6之前需要重启MySQL数据库
1. 关闭master写入，保证slave和master同步
2. 在slave配置参数skip-slave-start=1，避免slave启动后，继续使用传统复制模式
3. 修改配置，开启GTID
4. 重启所有MySQL数据库
- 如果数据库是新搭建，只需要配置参数启动即可
#### 搭建主从
根据master情况，常见的基于GTID模式的主从搭建有以下情况：
- master是新搭建的且无数据：直接使用change master语句搭建
- master刚运行不久，所有binlog保留完整：可以使用类似上面的方式搭建，直接change master。如果binlog较多，slave同步时间较长，可能导致网络压力过大
- master有大量数据：原始binlog可能已经删除，无法获得开始的GTID信息。
1. 利用备份获取master数据及GTID范围。使用innobackupex备份会将该信息保留在xtrabackup_binlog_info中
2. 利用备份数据，搭建slave实例
3. 启动slave实例，并且设置gtid_purged值
```
set @@global.gtid_purged="uuid:interval[-interval]"
```
4. change master，配置主从复制
5. 启动slave复制，slave自动跳过这段GTID付，拉取最新GTID信息
