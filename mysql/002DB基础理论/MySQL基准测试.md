## 为何需要基准测试
- 为什么基准测试很重要：因为基准测试是唯一方便有效的、可以学习系统在给定的工作负载下会发生什么的方法.
- 基准测试可以观察系统在不同压力下的行为,评估系统的容量,掌握哪些是重要的变化,或者观察系统如何处理不同的数据
- 基准测试可以完成以下工作,或者更多:
1. 验证基于系统的一些假设,确认这些假设是否符合实际情况.
2. 重现系统中的某些异常行为,以解决这些异常.
3. 测试系统当前的运行情况.如果不清楚系统当前的性能,就无法确认某些优化的效果如何.也可以利用历史的基准测试结果来分析诊断一些无法预测的问题.
4. 模拟比当前系统更高的负载,以找出系统随着压力增加而可能遇到的扩展性瓶颈.
5. 规划未来的业务增长.基准测试可以评估在项目未来的负载下,需要什么样的硬件,需要多大容量的网络,以及其他相关资源.这有助于降低系统升级和重大变更的风险.
6. 测试应用适应可变环境的能力.例如,通过基准测试,可以发现系统在随机的并发峰值下的性能表现,或者是不同配置的服务器之间的性能表现.基准测试也可以测试系统对不同数据分布的处理能力
7. 测试不同的硬件、软件和操作系统配置.比如RAID 5还是RAID 10更适合当前的系统?如果系统从ATA硬盘升级到SAN存储,对于随机写性能有什么帮助? Linux2.4系列的内核会比2.6系列的可扩展性更好吗?升级MySQL的版本能改善性能吗?为当前的数据采用不同的存储引擎会有什么效果?所有这类问题都可以通过专门的基准测试来获得答案
8. 证明新采购的设备是否配置正确
- 基准测试不是压力测试：基准测试相对比较简单，比较固定，而压力测试不可预期且变化多端

## 基准测试策略
策略有两种：对系统整体测试与单独对MySQL测试
- 整体测试
1. 测试整个应用系统,包括Web服务器、应用代码、网络和数据库是非常有用的,因为用户关注的并不仅仅是MySQL本身的性能,而是应用整体的性能
2. MySQL并非总是应用的瓶颈,通过整体的测试可以揭示这一点
3. 只有对应用做整体测试,才能发现各部分之间的缓存带来的影响
4. 整体应用的集成式测试更能揭示应用的真实表现,而单独组件的测试很难故到这一点
- 单独测试
1. 需要比较不同的schema或查询的性能.
2. 针对应用中某个具体问题的测试.
3. 为了避免漫长的基准测试,可以通过一个短期的基准测试,做快速的"周期循环",来检测出某些调整后的效果
4. 

#### 测试指标
- 吞吐量（output）：单位时间内的事务数。测试方法：tpcc等，单位是tps（每秒事务数）或者tpm（每分钟事务数）
- 响应时间或延迟（latency）：平均响应时间、最小响应时间、最大响应时间和所占百分比。通常私用百分比响应时间来替代最大响应时间，如95%响应时间是5毫秒
- 并发性：任意时间内同时发生的并发请求数。注意：不能混淆连接和并发数。可以通过sysbench指定32、64或者128个线程测试
- 可扩展性：横向扩展（水平扩展）或者纵向扩展（垂直扩展）
## 基准测试方法
#### 设计和规划基准测试
- 第一步：提出问题并明确目标
- 第二步，确定采用标准的基准测试或是设计专用的测试
1. 标准基准测试：tpcc适合OLTP，tpch适合olap
2. 专用专用基准测试：首先获得生产数据集的可还原并可反复使用的快照；然后针对数据运行查询（最好是回放生产系统的查询）
- 在不同级别记录查询：http请求；MySQL查询日志；多线程执行
- 详细写下测试规划（包括测试数据、系统配置、操作步骤、如何测量和分析结果、如何预热等），便于交接和重用
- 建立参数和结果文档化的规范
#### 基准测试运行时间、
- 测试时长要足够，才能得到有代表性的结果
- 测试时长根据针对具体情况确定
#### 获取系统的性能和状态
- 执行基准测试时候，需要尽可能多手机被测试系统的信息。最好为基准测试建立一个目录，并每一轮测试都创建单独的子目录，将测试结果、配置文件、测试指标、脚本、其他说明都保存下来
- 记录的信息包括：cpu使用、磁盘IO、网络流量、show global status计数器等
- 使用脚本、命令等工具记录信息（比如pt-diskstats工具捕获/proc/diskstats的数据分析磁盘IO）
#### 获得准确的测试结果
- 开始测试前回答：是否选择了正确的基准测试？是否为了问题手机了相关数据：是否采用了正确的测试标准？
- 确认测试结果是否可重复。确认每次测试系统状态一致；有必要的话重启系统；预热系统（预热方式也要一致）；还原每一次数据（数据碎片度、数据磁盘分布）；外部压力和外部程序（比如外部cron任务；raid的定时一致性检查）
- 各次测试之间，结果要可参照、可比对（控制变量，确定影响）
- 迭代法逐步修改基准测试的参数，可采用分治法（每次运行将参数对分减半）
- 如果测试中出现异常结果，应该仔细分析
#### 运行基准测试并分析结果
- 可自动化基准测试，更加可靠
- 多次运行基准测试，甚至可以使用统计方法
- 分析结果并定量结论
#### 绘图
图形更加直观，容易发现测试的趋势和异常
## 基准测试工具
#### 集成式测试工具
- ab：测试http服务器每秒最多处理多少请求。针对单个URL进行尽可能快的压力测试，用途有限
- http_load：类似ab，针对web服务器测试，可随机URL

```
# http_load -parallel 1 -seconds 10 url.txt
# http_load -rate 20 -seconds 10 url.txt
```

- jmeter：灵活而强大
#### 组件测试工具
- mysqlslap：MySQL自带工具，可设定并发数、指定SQL语句
- MySQL benchmark suite（sql-bench）：MySQL基准测试套件。大量预定义测试；单线程；主要测试查询
- super smack：用于测试MySQL和postgresql的基准测试工具，提供压力测试和复杂生成
- database test suite：测试某些工业标准（tpc）测试的工具集，比如dbt2测试tpcc
- percona's tpcc-mysql tool：tpcc基准测试工具集
- sysbench：多线程的系统压测工具<p>
**测试cpu：**
```

# sysbench --test=cpu --cpu-max-prime=2000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 2000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:  8741.14

General statistics:
    total time:                          10.0001s
    total number of events:              87430

Latency (ms):
         min:                                    0.11
         avg:                                    0.11
         max:                                    1.13
         95th percentile:                        0.12
         sum:                                 9967.77

Threads fairness:
    events (avg/stddev):           87430.0000/0.00
    execution time (avg/stddev):   9.9678/0.00
```
测试IO

```
# sysbench --test=fileio  --file-num=10 --file-test-mode=seqwr run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Extra file open flags: (none)
10 files, 204.8MiB each
2GiB total file size
Block size 16KiB
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing sequential write (creation) test
Initializing worker threads...

Threads started!


File operations:
    reads/s:                      0.00
    writes/s:                     6501.89
    fsyncs/s:                     650.69

Throughput:
    read, MiB/s:                  0.00
    written, MiB/s:               101.59

General statistics:
    total time:                          10.0107s
    total number of events:              71605

Latency (ms):
         min:                                    0.00
         avg:                                    0.14
         max:                                   57.22
         95th percentile:                        0.04
         sum:                                 9966.18

Threads fairness:
    events (avg/stddev):           71605.0000/0.00
    execution time (avg/stddev):   9.9662/0.00
```

测试oltp(prepare,run,cleanup)

```
# sysbench --test=oltp_insert.lua    --mysql-user=root --mysql-password=123456  --mysql-db=test  prepare
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Creating table 'sbtest1'...
Inserting 10000 records into 'sbtest1'
Creating a secondary index on 'sbtest1'...

# sysbench --test=oltp_insert.lua    --mysql-user=root --mysql-password=123456  --mysql-db=test  run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Initializing worker threads...

Threads started!

SQL statistics:
    queries performed:
        read:                            0
        write:                           2386
        other:                           0
        total:                           2386
    transactions:                        2386   (238.52 per sec.)
    queries:                             2386   (238.52 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0013s
    total number of events:              2386

Latency (ms):
         min:                                    2.94
         avg:                                    4.19
         max:                                   39.76
         95th percentile:                        6.09
         sum:                                 9990.88

Threads fairness:
    events (avg/stddev):           2386.0000/0.00
    execution time (avg/stddev):   9.9909/0.00

# sysbench --test=oltp_insert.lua    --mysql-user=root --mysql-password=123456  --mysql-db=test  cleanup
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Dropping table 'sbtest1'...
```




- MySQL的benchmark

```
mysql> select benchmark(1000,sha1(1));
+-------------------------+
| benchmark(1000,sha1(1)) |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)

mysql> select benchmark(10000000,sha1(1));
+-----------------------------+
| benchmark(10000000,sha1(1)) |
+-----------------------------+
|                           0 |
+-----------------------------+
1 row in set (4.50 sec)

mysql> select benchmark(10000000,md5(1));
+----------------------------+
| benchmark(10000000,md5(1)) |
+----------------------------+
|                          0 |
+----------------------------+
1 row in set (1.95 sec)

mysql> select benchmark(10000000,md5("test"));
+---------------------------------+
| benchmark(10000000,md5("test")) |
+---------------------------------+
|                               0 |
+---------------------------------+
1 row in set (1.78 sec)

mysql> select benchmark(10000000,sha1("test"));
+----------------------------------+
| benchmark(10000000,sha1("test")) |
+----------------------------------+
|                                0 |
+----------------------------------+
1 row in set (4.34 sec)
```
