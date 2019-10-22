## 复制概述
- MySQL内置的复制功能是构建基于MySQL水平扩展、大规模高性能应用的基础
- 复制还有利于构建高可用、扩展、灾难恢复、备份、以及数据仓库等工作的基础
- 解决的基本问题是让一台服务器的数据与其他服务器保持同步
- 一台主库的数据可以同步到备库上，而备库也可以是其他服务器的主库
- 有两种复制方式：基于行的复制（MySQL5.1以后才加入）和基于语句的复制（MySQL3.23即存在）
- 通过主库上记录二进制日志，在备库重放日志实现数据复制
1. 同一时间点备库上的数据可能与主库不一致，并且无法保证主备之间的延迟（几秒，几分钟甚至几个小时）
2. MySQL复制是向后兼容，即新版本可以做老版本的备库，但反过来通常不行
3. 大版本升级前，最好对复制设置进行测试
- 复制通常不会增加主库的开销，主要是启用二进制日志的开销
- 备库会增加主库的负载（网络开销）
- 锁竞争也会阻碍事务的提交（==什么情况下会有锁竞争==）
- 从一个高吞吐量（5000或者更高）的主库上复制到多个备库，唤醒多个复制线程发送事件开销将会累加
- 复制可以获得良好的读扩展（读写分离），但不适合于写扩展
- 采用一主多备的架构，会造成浪费，复制了大量不必要的重复数据
#### 复制解决的问题
比较常见的用途
- 数据分布：MySQL复制通常不会对带宽造成很大压力，但注意最好有一个稳定的、低延迟的连接
- 负载均衡：通过MySQL复制将读操作分布到各个服务器上，对读密集型的应用优化，并且简单操作就能实现基本的负载均衡。小规模的应用，可以简单对机器名做硬编码或者DNS轮询（将一个机器名指向多个IP地址）或者网络负载均衡（LVS，linux虚拟服务器Linux virtual server）
- 备份：是一项有益的补充，但不能替代
- 高可用和故障切换：避免MySQL单点失败，良好设计后可以显著缩短宕机时间
- MySQL升级测试：比较普遍；升级全部实例前，查询先在备库执行
#### 复制如何工作
- 复制三步骤：
1. 主库上把数据更改都记录到二进制日志（binary log）中：每次提交事务完成数据更新之前，主库按照事务提交的顺序将将数据更新事件记录到二进制日志中，完成后，主库通知存储引擎可以提交事务
2. 备库将主库的日志复制到自己的中继日志（relay log）中：首先备库启动一个IO工作线程，与主库建立普通的客户端连接，然后主库启动一个特殊的二进制转储（binlog dump）线程赌气主库二进制日志中的事件，备库IO线程会将接收到的事件记录到中继日志
3. 备库 读取中继日志中的事件，重放到备库数据库上。==可以配置中继日志是否写入自己的二进制日志中。==
![复制拓扑](https://github.com/ermaot/notes/blob/master/mysql/012%E5%A4%8D%E5%88%B6%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D/pic/MySQL%E5%A4%8D%E5%88%B6.png)
- 这种方式实现了获取事件和重放事件的解耦，可以异步进行；但导致主库上可以并发但备库只能串行化
## 配置复制
分三步：
1. 在每台服务器上创建复制账号（非必须但最好）
2. 配置主库和备库
3. 通知备库连接到主库并从主库复制数据
#### 创建复制账号
- 主库和备库都创建相同的 账号

```
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO repl@'192.168.0%'' IDENTIFIED BY password;
```
如果是mysql8要使用create user

```
> create user repl@'47.104.%' IDENTIFIED BY '123456';

> GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO repl@'47.104.%';
```

- 复制账号只需要Replication slave权限，但依然赋予了Replication client权限，原因是：
1. 用来监控和管理复制的账号需要Replication client权限，使用同一个账号更方便
2. 方便交换主备库的角色
#### 配置主库和备库
主库：
server_id 最好是有意义的值，成为约定并遵循；可以为服务器IP地址末8位
```
my.cnf
log_bin = mysql-bin
server_id = 10
```
从库

```
log_bin = mysql-bin
server_id= 2
relay_log= /var/lib/mysql/mysql-relay-bin
log_slave_updates = 1
read_only =1
```
1. server_id 是必须
2. log_bin是二进制日志的名称，在此处设置和主库名称一样
3. relay_log指定中继日志的位置和命名
4. log_slave_updates：允许各库将重放事件也记录到自身的二进制日志中（如果不开，可能会遇到奇怪的现象，比如配置错误的时候可能导致备库数据库被修改）
5. read_only阻止没有特权权限的线程修改数据
#### 启动复制

```
change master to master_host="47.104.149.52" , master_user="repl", master_password="123456", master_log_file="mysql-bin.000001", master_log_pos=0;

start slave;
show slave status;

> show processlist;
+----+-----------------+-----------+------+---------+------+----------------------------------+------------------+
| Id | User            | Host      | db   | Command | Time | State                            | Info             |
+----+-----------------+-----------+------+---------+------+----------------------------------+------------------+
|  5 | event_scheduler | localhost | NULL | Daemon  | 4614 | Waiting on empty queue           | NULL             |
| 10 | root            | localhost | test | Query   |    0 | starting                         | show processlist |
| 19 | system user     |           | NULL | Connect | 3302 | Waiting for master to send event | NULL             |
+----+-----------------+-----------+------+---------+------+----------------------------------+------------------+
3 rows in set (0.00 sec)
```

#### 从另一个服务器开始复制
- 如果主库已经运行了一段时间，但从库是新的，则需要三个条件让二者保持同步
1. 某个时间点的主库的数据快照
2. 主库当前的二进制文件，和获得数据快照时在该二进制日志中的偏移量（此二值合称日志文件坐标（log file coordinates）），通过show master status命令获取
3. 从快照时间到现在的二进制日志
- 从其他服务器克隆备库的方法：
1. 冷备份：关闭主库，把数据复制到备库；重启主库后，会使用新的二进制文件，此时在备库执行change maser to指向这个文件的起始处；缺点是复制数据必须关闭主库
2. 热备份：如果仅使用了Myisam表，可以在主库运行时使用mysqlhotcopy或者rsync复制数据
3. 使用mysqldump：如果是包含innodb表，可以使用下面命令转储主库并加载到备库且设置响应的二进制日志坐标：

```
mysqldump --single-transaction --all-databases --master-data=1 --host=server1 | mysql --host=server2
```
选项--single-transaction使得转储的数据为事务开始前的数据；如果是非事务型的表，可以使用--lock-all-tables选项
- 使用快照或者备份：只需要备份或者快照到备库，然后使用change master to 执行二进制日志坐标
4. 使用percona xtrabackup：开源的热备工具，不阻塞服务器的操作。
<p>如果从主库获得备份，可以xtrabackup_bin_log_pos_innodb文件中获得复制开始的位置
<p>如果是另外的备库获得备份，可以从xtrabackup_slave_info文件中获得复制开始的位置
5. 使用另外的备库
6. 不要使用load data from master或者load table from master，这些命令过时、缓慢并且危险，并且只适用于Myisam存储引擎

#### 推荐的复制配置
- 主库上最重要的设置：sync_binlog = 1：每次提交之前将二进制日志同步到磁盘
- 如果无法容忍服务器崩溃导致表损坏，推荐innodb。Myisam在服务器崩溃后容易出现不一致
- 如果是innodb，强烈建议如下配置：

```
innodb_flush_logs_at_trx_commit
innodb_support_xa=1
innodb_safe_binlog
```
- 在主库上，至少有单独的命名，最好写明log-bin的绝对路径
- 备库上，推荐开启如下配置：

```
relay_log=/path
skip_slave_start                //防止复制随着mysql启动而自动启动
read_only
```
- 如果使用MySQL5.5并且不介意额外的fsync()导致的性能开销，最好设置：

```
sync_master_info = 1        //执行每个event都去更新 mysql.slave_master_info
sync_relay_log = 1
sync_relay_log_info = 1
```

## 复制的原理
#### 基于语句的复制
- MySQL5.0以及以前的版本只支持基于语句的复制（逻辑复制），实现简单，二进制日志里事件紧凑
- mysqlbinlog工具查看binlog
- 缺点
1. 主库和备库的一些元数据信息不一定相同，比如时间戳
2. 还存在一些无法被正确复制的SQL，如current_user()
3. 存储过程和触发器在基于语句的复制模式也有可能存在问题
4. 备库的重放必须串行
#### 基于行的复制
- MySQL5.1开始支持基于行的复制
- 好处：
1. 可以正确复制每一行
2. 一些语句可以被更有效复制（如从大表查询数据插入到小表中）
- 缺点：
1. 一些语句代价可能比基于行的代价高（比如全表更新）
2. 无法进行时间点恢复

#### 基于行或者基于语句：哪种更好
整体上，基于行的复制模式更优，在实际应用中也适用于大多数场景
- 基于语句复制模式：
1. 主备schema不同时，逻辑复制能够在多种情况下工作：主备上表的定义不同但数据类型兼容；列的顺序不同
2. 复制的执行就是sql，变更容易理解
3. 很多情况下无法正确复制：5.0和5.1版本的存储过程和触发器的复制存在大量bug；==如果使用触发器或者存储过程，就不用基于语句的复制==
- 基于行的复制：
1. 几乎没有行复制无法处理的场景
2. 可以减少锁的使用
3. 在一些情况下基于行的二进制日志还会记录发生改变之前的数据，有利于某些数据的恢复
4. 无需像基于语句的复制建立查询计划，占用cpu更少
5. 某些情况下基于行的复制能帮助更快找到并解决数据不一直的情况
6. 没有记录语句，因此无法判断执行了哪些SQL
#### 复制文件
复制会使用到的文件
- mysql-bin.index：开启二进制日志时，同时生成与二进制日志同名但以.index为后缀的文件
- mysql-relay-bin-index：中继日志的索引文件
- master.info：保存备库连接到主库所需要的信息，格式为纯文本；同时记录了复制用户的密码        //==5.1以后就不存在这个文件了==
- relay-log.info：包含了当前备库复制的二进制日志和中继日志的坐标
- expire_logs_days 定义了MySQL过期日志的清理方式，会与.index有交互
#### 发送复制事件到其他备库
- log_slave_updates 选项可以让备库变成其他服务器的主库（MySQL会将其执行过的事件记录到自己的二进制日志中）
![image](https://github.com/ermaot/notes/blob/master/mysql/012%E5%A4%8D%E5%88%B6%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D/pic/MySQL%E5%A4%8D%E5%88%B62.png)
#### 复制过滤器
- 可以使用binlog_do_db 和 binlog_ignore_db控制过滤，但最好别开启
## 复制拓扑
基本原则：
1. 个 MySQL备库实例只能有一个主库
2. 每个备库必须有一个唯一的服务器ID
3. 一个主库可以有多个备库(或者相应的,一个备库可以有多个兄弟备库)
4. 如果打开了Log_slave_updates选项,一个备库可以把其主库上的数据变化传播到其他备库.
#### 一主多备
- 一主多备与一主一备类似，备库之间没有交互（除非是备库ID相同而导致相互踢出）
- 少量写和大量读的时候，这种配置非常有用，用途如下：
1. 为不同的角色使用不同的备库（例如添加不同的索引或者使用不同的存储引擎）
2. 把一台备库当代用的主库，除了复制没其他数据传输
3. 将一台备库放远程数据中心，用作灾难恢复
4. 延迟一个或者多个备库，已备灾难恢复
5. 使用其中一个备库，作为备份、培训、开发或者测试使用服务器
#### 主动-主动模式下的主主复制
主主复制（双主复制或者双向复制）包含两台服务器，每一个都被配置成对方的主库和备库
![主主复制](https://github.com/ermaot/notes/blob/master/mysql/012%E5%A4%8D%E5%88%B6%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D/pic/MySQL%E5%A4%8D%E5%88%B63.png)
- 场景：两个不同地理位置的办公室，都需要一份可写的数据拷贝
- 最大问题：如何解决冲突。
- 双主复制导致的问题非常多
- MySQL不支持多主库复制
#### 主动-被动模式下的主主复制
主主结构的变体（其中一个是只读的被动服务器）
- 这种方式使得反复切换主动和被动服务器非常方便，便于故障转移和故障恢复。

```
执行alter table操作可能锁住整个表，在该配置下，可以先停止主动服务器上的备库复制线程，然后在被动服务器上执行alter操作，交换角色，再在先前的主动服务器上启动复制线程。这个服务器会读取中继日志并执行相同的alter语句
```

#### 拥有备库的主主结构
#### 环形复制
![image](https://github.com/ermaot/notes/blob/master/mysql/012%E5%A4%8D%E5%88%B6%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D/pic/MySQL%E5%A4%8D%E5%88%B64.png)
- 无双主复制的优点（对称配置和简单的故障转移）
- 大大增加了整个系统的失效概率，一个节点失效就整个系统失效
尽量避免环形复制
#### 主库、分发主库以及备库
- 当备库很多的时候，有大的事件出现，主库负载会明显上升，可能导致内存耗尽并崩溃
- 如果备库请求的数据不在文件系统的缓存中，可能导致大量的磁盘检索，影响主库性能
- 可以使用分发主库（实际是专用备库）
![image](https://github.com/ermaot/notes/blob/master/mysql/012%E5%A4%8D%E5%88%B6%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D/pic/MySQL%E5%A4%8D%E5%88%B65.png)


#### 树或者金字塔形
![image](https://github.com/ermaot/notes/blob/master/mysql/012%E5%A4%8D%E5%88%B6%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D/pic/MySQL%E5%A4%8D%E5%88%B66.png)
#### 定制的复制方案
## 复制和容量规划
#### 为什么复制无法扩展写操作
#### 备库什么时候开始延迟

```
打开userstat=ON
然后> select * from information_schema.user_statistics;
```

#### 规划冗余容量
## 复制管理和维护
#### 监控复制
- 复制的大部分工作是在备库上完成的，也是最容易出问题的地方
- show master status 和 show master logs

```
> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      155 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

purge master logs before "2019-08-04 00:00:00"; 

> show binlog events in 'mysql-bin.000001' from 24151805 limit 1;
+------------------+----------+-------------+-----------+-------------+--------------------------------+
| Log_name         | Pos      | Event_type  | Server_id | End_log_pos | Info                           |
+------------------+----------+-------------+-----------+-------------+--------------------------------+
| mysql-bin.000001 | 24151805 | Update_rows |       100 |    24151999 | table_id: 91 flags: STMT_END_F |
+------------------+----------+-------------+-----------+-------------+--------------------------------+
1 row in set (0.00 sec)
```

#### 测量备库延迟
- show slave status

```
> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 47.104.149.52
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 155
               Relay_Log_File: izm5edbv563hlvcbf71opez-relay-bin.000002
                Relay_Log_Pos: 369
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1146
                   Last_Error: Error executing row event: 'Table 'test.warehouse' doesn't exist'
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 155
              Relay_Log_Space: 62701944
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 1146
               Last_SQL_Error: Error executing row event: 'Table 'test.warehouse' doesn't exist'
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 100
                  Master_UUID: caac3cb7-a765-11e9-b63b-00163e04816e
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 190805 10:32:30
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set (0.00 sec)
```
Seconds_Behind_Master
1. Seconds_Behind_Master 是服务器当前时间戳与二进制日志中的事件时间戳对比得到，所以在执行事件时才报告延迟，如果备库复制线程没有运行，延迟为NULL
2. 一些错误（max_allowed_packet不匹配或者网络不稳定）可能终端复制或者停止复制线程，此时Seconds_Behind_Master显示为0
3. 即使备库线程在运行，有时也可能无法计算延迟，此时备库会报0或者NULL
4. 一个大事务可能倒持延迟或者波动。当备库执行一个1小时的事务，执行完毕后会报告备库延迟1小时，然后很快变0
5. 如果分发主库落后，并且其本身也有已经追上它的备库，备库延迟显示为0，而实际上与源库是有延迟
#### 确定主备是否一致
- MySQL没有内置的方法比较服务器之间的数据是否相同
- checksum table可以为数据和表生成校验值，但复制进行中则不可
- percona toolkit的pt-table-checksum

```
$pt-table-checksum --replicate=test.checksum <master_host>
```

#### 从库重新同步备库
#### 改变主库
#### 在一个主主配置中交换角色
## 复制问题和解决方案
#### 数据损坏或者丢失的错误
- 大部分非正常关机导致的复制问题是没有把数据及时刷到磁盘
- 主库意外关闭
1. 如果sync_binlog没有设置，崩溃前最后几个二进制日志没有刷新到磁盘，重新启动后，备库重连并读取，但主库会告诉备库没有这个二进制偏移量。只能指定备库从下一个二进制日志开头读日志（一些日志事件永远丢失）。建议使用percona toolkit中的pt-table-checksum工具来检查主备一致性
2. sync_binlog开启，Myisam表的数据仍然可能在崩溃时损坏；innodb事务在innodb_flush_log_at_trx_commit没有设为1，也能丢失数据
- 备库意外关闭
1. 备库在非计划关闭后重启，会读master.info文件以找到上次停止复制的位置，但该文件没有同步写到磁盘，所以文件中的相信是错的。percona toolkit中的pt-slave-restart工具可以帮助忽略错误
2. 如果使用innodb，可以重启后观察MySQL错误日志，会有恢复点的二进制坐标
- 主库上的二进制日志损坏：唯一办法是忽略损坏位置
1. 执行flush logs命令，主库生成新的日志文件，备库指向该文件
2. 通过set global SQL_SLAVE_SKIP_COUNTER = 1忽略一个损坏的事件，如果有多个就反复执行
- 备库上的中继日志损坏：如果主库上的日志是好的，则可以通过change master to 命令丢弃并重新获取损坏事件（没有MySQL5.5可以自动重新获取中继日志）
- 二进制日志和innodb事务日志不同步：主库崩溃时，innodb可能将一个事务标记为已提交，但此时该事务可能还没有记录到二进制日志中，除非备库已经保存中继日志，否则没有任何办法恢复。MySQL使用sync_binlog防止该问题。
#### 使用非事务型表
- 事务中途服务器被kill，则会出现不一致。所以关闭MySQL之前确保已经运行了stop slave
#### 混合事务型和非事务型表
#### 不确定语句
- 使用基于语句的复制模式时，如果通过不确定的方式更改数据可能会导致主备不一致
1. 一个带limit的update语句
2. 一个拥有多个唯一索引的表上使用replace或者insert ignore语句（MySQL在主库和备库上可能选择不同的索引）
- 涉及information_schema的语句容易在主备之间不一致
- 许多系统变量，如@@server_id和@@hostname，在MySQL5.1之前无法正确复制
- 基于行的复制，没有上述限制
#### 主库和备库使用不同的存储引擎

#### 备库发生数据改变
如果备库数据发生改变，则这个改变会在表之间传播，唯一办法就是重新同步主从数据库
#### 不确定的服务器ID
会导致备库相互踢开对方，导致反复的重连和连接断开信息，但不会提及被错误配置的服务器ID。唯一办法就是小心设置备库的服务器ID
#### 未定义的服务器ID
如果没有在my.cnf中定义服务器ID，可以通过change master to设置备库但无法启动复制

```
mysql> START SLAVE;
ERROR 1200(HYooo ) The server is not configured as slave; fix in config file or with CHANGE MASTER TO
```

#### 对未复制数据的依赖性
如果在主库上有备库不存在的数据库或者表，复制很容易中断。唯一有的办法就是避免在主库上创建备库上没有的表
#### 丢失的临时表
- 临时表在某些时候有用，但它与基于语句的复制方式不相容。如果备库崩溃或者正常关闭，任何复制线程拥有的临时表都会丢失，重启备库后，所有依赖于该临时表的语句都会失败
- 如果备库重启找不到临时表而导致复制停止，可以：1.直接跳过错误；2.手动创建一个名字和结构相同的表代替临时消失的表
- 不管什么办法，如果写入查询依赖于临时表，都可能造成数据不一致
#### 不复制所有的更新
#### innodb加锁读引起的锁争用
#### 在主主复制结构中写入两台主库
#### 过大的复制延迟
#### 来自主库的过大的包
#### 受限复制的复制带宽
#### 磁盘空间不足
#### 复制的局限性
## 复制有多快
## MySQL复制的高级特性
## 其他复制技术
