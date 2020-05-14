## MySQL配置工作的原理
- mysql配置文件一般在/etc/my.cnf和/etc/mysql/my.cnf(类unix系统)
- 使用操作mysqld --verbose --help | grep -A 1  "Default options"查看配置文件

```
# mysqld --verbose --help | grep -A 1  "Default options"
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf 
```
- 配置文件分为多个部分，服务器读取[mysqld]的部分，客户端读取[client]的部分
#### 语法、作用域和动态性
- 配置项有多个作用域：
1. 全局（设置之后全局生效）

```
set global sort_buffer_size =<value>
set @@global.sort_buffer_size := <value>
```
2. 会话（对当前会话生效）

```
set @@session.sort_buffer_size := <value>
```
3. 对象级
#### 设置变量的副作用
- 动态设置变量，可能导致数据库做大量的工作

参数 | 效果
---|---
key_buffer_size | 一次性为键缓冲区分配指定空间<br>操作系统不立即分配，到使用时才真正分配<br>MySQL允许创建多个键缓存（==？？如何创建==）<br>对于一个已经存在的键缓存设置非零值，会阻塞所有对该键值的访问并刷新键缓存
table_cache_size | 下次有线程打开表才有效果<br>如果值大于缓存中表的数量，线程可以直接将最新打开的表放入缓存；<br>否则会删除不常使用的表（mariadb5.5和mysql8都不存在该参数，而是table_open_cache）<br>mysql手册上给的建议大小 是:table_cache=max_connections*n（n是MySQL最大的表数目）
thread_cache_size | 在有连接被关闭时生效<br>连接被关闭时，MySQL检查是否还有空间缓存线程，若没有则销毁线程
query_cache_size | MySQL启动的时候，一次性分配；如果修改该值（即使值与之前一样），MySQL会一个个清理缓存，并重新分配内存初始化
read_buffer_size | MySQL只会在需要查询时分配；一次性分配
read_rnd_buffer_size | MySQL只会在需要查询时分配；分配实际大小而非参数大小（更好的名字是max_read_rnd_buffer_size）
sort_buffer_size | MySQL在查询需要做排序操作时分配；然后一旦需要排序就立刻分配指定大小的内存，不管是否需要<br>连接级别参数，尽量小一些；按照需要在连接中调大

#### 入门
- 参数值不是越大越好，有可能导致服务器使用交换内存或者超过地址空间
- 通过监控确认生产环境中变量的修改
- 对配置文件写注释
- 对配置文件做版本控制
#### 通过基准测试迭代优化
## 什么不该做
- 不要根据一些比率调优。比如缓冲命中率。缓存命中率与缓存是否过大或者过小没有关系，取决于工作负载；缓存命中没有意义
- 不要使用调优脚本，但可以参考（http://tools.percona.com）
- 在互联网搜索如何配置，不一定可靠
- 不要相信MySQL崩溃时自身输出的内存消耗公式
## 创建MySQL配置文件
- MySQL默认配置其中大部分都比较合适，但并不都是靠谱的
- 配置样例

```
[mysqld]
# GENERAL
datadir =/var/lib/mysql
socket = /var/lib/mysq1/mysq1. sock
pid. file = /var/lib/mysq1/mysq1.pid
user = mysql
port = 3306
default_ storage_engine = InnoDB
# INNODB
innodb_buffer_pool_size = <value>
innodb log file_size = <value>
innodb_file_per table =1
innodb flush_method = 0 DIRECT
# MyISAM
key_buffer_size = <value>
#LOGGING
1og_error = /var/lib/mysq1/mysql-error .1og
slow_query_log = /var/1ib/mysq1/mysq1-s1ow.1og
# OTHER
tmp_table_size=32M
max_heap table_size = 32M
query_cache_type
query_cache_size
max_connections = <value>
thread_cache = <value>
table_cache = <value>
open_files_limit = 65535
[client]
socket = /var/lib/mysq1/mysq1.sock
port = 3306
```
- 第一件事是设置数据位置
- 不要把socket文件和pid文件放MySQL编译默认的位置，要明确指定，因为不同的MySQL版本可能导致错误
- 一般情况下，如果决定默认存储引擎，最好显式配置
- innodb需要设置合适的缓冲池（buffer pool）和log file，默认值过小。缓冲池大小设置过程如下：
1.  从服务器内存总量开始.
2. 减去操作系统的内存占用,如果MySQL不是唯一运行在这个服务器上的程序,还要扣掉其他程序可能占用的内存.
3. 减去一些MySQL自身需要的内存,例如为每个查询操作分配的一些缓冲.
4. 减去足够让操作系统缓存InnoDB日志文件的内存,至少是足够缓存最近经常访问的部分.(此建议适用于标准的MySQL, Percona Server可以配置日志文件用O_DIRECT方式打开,绕过操作系统缓存),留一些内存至少可以缓存二进制日志的最后一部分也是个很好的选择,尤其是如果复制产生了延迟,备库就可能读取主库上旧的二进制日志文件,给主库的内存造成压力.
5. 减去其他配置的MySQL缓冲和缓存需要的内存,例如MyISAM的键缓存(Key Cache),或者查询缓存(Query Cache).
6. 除以105%,这差不多接近InnoDB管理缓冲池增加的自身管理开销.
7. 把结果四舍五入,向下取一个合理的数值.向下舍入不会太影响结果,但是如果分配太多可能就会是件很糟糕的事情.
#### 检查MySQL服务器状态变量
- 有时候可以使用show global status输出，作为配置的输入
- 使用 mysqladmin extended-status -ri60 -uroot -p  查看每60秒状态变量的增量变化
## 配置内存使用
配置内存步骤
1. 确定可以使用的内存上限
2. 确定每个连接MySQL需要使用多少内存，例如sort buffer和tmp table
3. 确定操作系统需要多少内存
4. 把剩下的内存全部给MySQL缓存，如innodb缓冲池
#### MySQL可以使用多少内存
- 上限：物理内存
- 操作系统以及架构限制：32bit操作系统任意进程在2.5G~2.7G范围
- 系统glibc可能限制每次分配内存的大小（单次最大2GB，那么innodb_buffer_pool值不能设置大于2GB）
#### 每个连接需要多少内存
- 保持一个连接（线程）只需要少量内存
- 还需要一些基本的内存执行查询（比如sort_buffer，绑定变量等）
#### 为操作系统保留内存
- 至少1~2G内存，内存足就多留一些
- ==没有使用交换内存，表明操作系统内存足够==
#### 为缓存分配内存
缓存可以避免磁盘访问，提高性能
1. innodb缓冲池
2. innodb日志文件和Myisam数据的操作系统缓存
3. Myisam键缓存
4. 查询缓存
5. 无法手工配置的缓存（二进制日志和表定义文件的操作系统缓存）
#### innodb缓冲池（buffer pool）
- innodb缓冲池是MySQL需求量最大的内存区
- innodb缓冲池包括索引缓存、行缓存、自适应哈希索引缓存、插入缓冲、锁，以及其他内部数据结构
- innodb缓冲池还帮助延迟写入（写入合并，把随机写入转换成顺序写入）
- ==innotop工具==
#### Myisam键缓存（key caches）
- Myisam缓冲池也称键缓冲，默认只有一个键缓存，但可以创建多个
- Myisam自身只缓存索引不缓存数据（依赖于操作系统缓存数据）
- 最重要的配置项是key_buffer_size，值可以通过下面计算（不超过该值或者不超过系统保留总内存的20%~50%）

```
> select sum(index_length) from information_schema.tables where engine="myisam";

# du  -csh *.MYI 
```
- 默认Myisam将所有索引都缓存在默认键缓存中，但可以创建多个键缓冲

```
key_buffer_1.key_buffer_size = 1G
key_buffer_2.key_buffer_size = 1G

cache index tablename in key_buffer_1;

load index into cache tablename1,tablename2;        //预加载索引到缓存中

//不支持innodb
> load index into cache store;
+--------------+--------------+----------+---------------------------------------------------------------+
| Table        | Op           | Msg_type | Msg_text                                                      |
+--------------+--------------+----------+---------------------------------------------------------------+
| sakila.store | preload_keys | note     | The storage engine for the table doesn't support preload_keys |
+--------------+--------------+----------+---------------------------------------------------------------+
1 row in set (0.00 sec)
```
- 键缓冲使用率计算：

```
p  = 100 - ((key_blocks_unused * key_cache_block_size)*100/key_buffer_size)

 show status like "%key_blocks_unused%";
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Key_blocks_unused | 6697  |
+-------------------+-------+

> show variables like "key_cache_block_size";                                                                                                                   
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| key_cache_block_size | 1024  |
+----------------------+-------+
1 row in set (0.00 sec)

> show variables like "key_buffer_size";
+-----------------+---------+
| Variable_name   | Value   |
+-----------------+---------+
| key_buffer_size | 8388608 |
+-----------------+---------+
1 row in set (0.01 sec)

p = 100 - 6697 * 1024 * 100 / 8388608 = 18.2465
```
- 键缓存命中率计算方法

```
读命中率=1-Key_reads /Key_read_requests

写命中率=1-Key_writes / Key_write_requests 
```

- 从经验上来说，每秒缓存未命中的次数更有用，用key_read / uptime计算

```
// 下面命令可以获取每10秒状态变化量
mysqladmin extended-status -r -i 10 | grep Key_reads
```
- 即使没有任何Myisam表，依然需要将key_buffer_size设置为较小的值，如32M
- MySQL服务器有时在内部使用Myisam表，例如group by语句有可能会使用Myisam做临时表
###### MySQL键缓存大小（key block size）
- 如果缓存块太小，可能会碰到写时读取（read around write）
- MySQL5.1以及以后的版本，可以设计Myisam索引块大小与操作系统一样，参数是 myisam_block_size

```
mariadb5.5
> show  variables like "%myisam_block_size%";
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| myisam_block_size | 1024  |
+-------------------+-------+

mysql8
> show variables like "%myisam_block_size%";
Empty set (0.01 sec)
```

#### 线程缓存
- 线程缓存保存哪些当前没有与连接关联但是准备为后面新的连接服务的线程
- thread_cache_size变量指定了MySQL可以保持在缓存中的线程数，除非服务器有很多连接请求，否则无需配置
- 查看线程缓存是否足够大，可以查看threads_created状态变量
- 一个好办法是观察 Threads_connected。如果Threads_connected保持在100~120，可以设置缓存大小为20，如果保持在500~700，缓存大小为200足够
- 把线程缓存设置非常大没有必要，设置很小也不能节省很多内存，也没必要（每个线程通常256K内存）
#### 表缓存（table cache）
- 表缓存可以避免打开一个文件描述符的开销（但开销不大，所以这一点上的益处并不多）
- 可以避免服务器修改Myisam文件头来标记表“正在使用中”
- 表缓存是服务器和存储引擎之间分离不彻底的产物，对innodb重要性比较小
- mysql5.1中，表缓存分离成两个部分：
1. 打开表的缓存
2. 表定义缓存（通过 table_open_cache 和 table_definition_cache 变量来配置），通常把table_definition_cache设置比较高

```
> show variables like "%table_open_cache%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| table_open_cache           | 4000  |
| table_open_cache_instances | 16    |
+----------------------------+-------+

> show variables like "%table_definition_cache%";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| table_definition_cache | 2000  |
+------------------------+-------+
```

#### innodb数据字典
- innodb 有自己的表缓存，可以成为表定义缓存或者数据字典，目前不能配置
- innodb 打开一张表，就增加了一个对应的对象到数据字典，每张表占用4KB或者更多内存
- 表关闭时不会从数据字典移除（类似于内存泄漏的现象，伪泄漏。percona有选项可以控制数据字典大小，做数据字典更新操作）
- 打开表的时候会统计信息，需要很多IO操作，innodb没有信息持久化，代价很高
- 打开表之后，每隔一段时间或者遇到触发事件（改变表内容或者查询information_schema），重新计算统计信息
- 如果有很多表，服务器可能会花费数个小时启动并完全预热。percona server使用 innodb_use_sys_stats_table 或者MySQL5.6以上版本有参数 innodb_stats_persistent 来控制是否持久化

```
mariadb5.5
> show  variables like "%sys_stats_table%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_use_sys_stats_table | OFF   |
+----------------------------+-------+
1 row in set (0.00 sec)

MySQL8
> show  variables like "%innodb_stats_persistent%";
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| innodb_stats_persistent              | ON    |
| innodb_stats_persistent_sample_pages | 20    |
+--------------------------------------+-------+
2 rows in set (0.00 sec)
```
- MySQL启动后，innodb统计操作可能对服务器和一些特定查询产生冲击，可以关闭    innodb_stats_on_metadata

```
> show  variables like "%innodb_stats_on_metadata%";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_stats_on_metadata | OFF   |
+--------------------------+-------+
1 row in set (0.00 sec)
```
- innodb任意时刻可以打开的ibd文件是有数量限制的，由innodb引擎控制。innodb_open_files 最好能保证服务器把所有的ibd文件打开

```
> show  variables like "%innodb_file_per_table%";
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.00 sec)

> show  variables like "%innodb_open_files%";
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| innodb_open_files | 4000  |
+-------------------+-------+
1 row in set (0.00 sec)
```

## 配置MySQL的IO行为
==一些配置项影响MySQL如何同步数据和如何恢复，对性能影响非常大==
#### innodb 的IO配置
对于常见应用，最重要的一小部分内容是innodb日志文件大小、innodb怎么样刷新它的日志缓冲、以及innodb怎么样执行IO
#### Myisam 的IO配置
## 配置MySQL的并发
#### innodb并发配置
#### Myisam 并发配置
## 基于工作负载的配置
#### 优化blob和text的场景
#### 优化排序（filesorts）
#### 完成基本配置
#### 安全和稳定的配置
#### 高级innodb配置