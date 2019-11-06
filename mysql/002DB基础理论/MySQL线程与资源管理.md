## 线程与进程
- 操作系统中，以进程为独立运行的基本单位。数据库本身也是一个进程
- 20世纪80年代中期，提出了比进程更小的能独立运行的基本单位：线程
- 引入进程是为了使得多个程序并发执行，改善资源使用率和系统吞吐量；引入线程是为了减少程序并发执行时付出的时空开销，使得操作系统有更好的并发性
- 进程的两个基本属性：
1. 进程是一个可拥有资源的独立单位
2. 进程也是一个可以独立调度和分派的基本单位
- 为了使程序可以并发执行，系统需要做下列操作
1. 创建进程，并为之分配必须的资源，如cpu、内存空间、IO设备以及建立相应的进程控制块（PCB）
2. 进程切换。对进程切换时，保留当前进程的CPU环境和设置新进程的cpu环境，需要耗费很多CPU时间
3. 撤销进程。回收进程的资源，并撤销PCB
- 因此系统中进程数目不宜过多，切换频率也不宜太高
#### 线程与进程的比较
- 线程具有传统进程的特征，又成轻型进程（light-weight process），而传统进程又成重型进程（heavy-weight process）。通常一个进程有若干个线程
- 进程与线程比较
1. 线程是调度和分派的基本单位，而进程作为资源拥有的基本单位，从而使得传统进程的两个基本属性分开
2. 线程系统中，不仅进程之间可以并发执行，而且一个进程中的多个线程之间，也可以并发执行
3. 进程是拥有资源的基本单位，而同一进程下的线程之间的资源是共享的
4. 创建和撤销进程时，系统都要分配或者回收资源；切换时，也要保存当前cpu环境并设置新进程的cpu环境，因而开销较大；而线程只需要保存和设置少量寄存器的内容，并不设计存储器管理方面的操作，因此开销较小。有的系统中，线程的切换、同步和通信都无需底层操作系统的干预
## MySQL线程问题和解决方案
#### 标准C函数调用
- 使用锁来保护临界资源，但锁不能滥用
- malloc函数是线程安全的
- MySQL极少直接使用标准C函数库，而进行包装，主要在mysys库下
- 使用C语言的预编译功能，在不同平台获得统一的操作
#### 互斥锁
- 多个线程访问同一个资源可使用互斥方式保证同步，互斥实现依赖于互斥锁
- MySQL把所有的全局资源和变量按照相关性分组，当线程使用某些资源的时候，MySQL直接给一个分组加锁。

MySQL 的全局互斥锁

名称 |说明
---|---
全局锁|
LOCK_open |所有与表缓存、打开表、关闭表相关的变量和数据结构
LOCK_trolley_status| 保护SHOWSTATUS命令输出的那些变量
LOCK_crypt| 保护对Unix C库函数的调用.
LOCK_error_log |保护错误日志的写入操作
LOCK_localtime_r|当本地操作系统中的C标准库无localtime0函数时<p>该锁用于保护mysys/my_pthread.c中函数localtime._r()调用的localtime()
LOCK_manager| 表管理线程的数据结构需要该锁来保护
LOCK_active_mi |保护变量active_mi,该active_mi指向从服务器描述类
LOCK_bytes_received|变量byte_received用于记录服务器从其所有客户端收到的字节数<p>在5.0之前的版本中,该锁用于保护byte_received状态变量
线程锁|
THR LOCK_heap| 保护与Memory存储引擎有关的结构和变量
THR_LOCK_charset| 保护与字符集有关的结构和变量
THR_LOCK_myisam| 保护与MyISAM存储引擎有关的结构和变量
THR_LOCK_LOCK|保护与表锁管理有关的结构和变量
THR_LOCK_open |保护与文件打开有关的结构和变量

#### 线程同步
- POSIX提供了一个机制来完成线程同步的目的：条件变量.当一个线程等待某个资源时,线程调用pthread_cond_wait(&cond_trolleys_ready,&lock_trolleys)函数,其中cond_trolleys_ready为条件变量,lock trolleys为互斥锁。当该线程获得资源并继续执行后,线程调用pthread_cond_signal()或pthread_cond_broadcast()将自己的状态通知到其他线程
- MySQL 全局条件变量

名称 |说明
---|---
COND_thread_cache |与LOCK thread_count结合使用,唤醒线程缓存队列中的等待线程
COND_thread_count| 与LOCK _thread_count结合使用,当线程被创建或销毁时被设置.
COND_refresh| 和LOCK open一起使用,当table cache中的数据被更新时被设置.
COND_manager|强制使得管理线程(见sql/sql_manager.cc)启动,进行各种清理工作，和LOCK_manager共同使用

## 客户请求的处理
- MySQL服务器进程一直监听会话请求，对于任何一个连接总是分配一个线程去处理
- 根据系统配置文件my.cnf和系统当时的状态，服务器会创建一个新的线程或者从线程池分配一个线程来处理请求
- 用户与服务器的连接持续到客户端发送quit命令或者超时或者客户端异常中断
#### MySQL启动过程
- C/C++程序都从main函数开始，MySQL的main函数在sql/mysqld.cc中

```
int main(int argc,char* argv[]) {
/*初始化日志类*/
logger.init_base() ;
/*初始化系统全局变量,如文件描述符,字符集等;
给MySQL服务器设置默认的参数*/
static int init_common_variables (const char *conf_file_name, int argc,char **argv, const char * *groups)
/*初始化各种同步信号*/
static void init_signals (void) ;
....
if ((user_info = check__user (mysqld_user)))
....
if (user_info)
set_user (mysqla_user, user_info) ;
....
/*初始化服务器模块*/
init_server_components() ;

/*初始化网络子系统,根据配置和平台,监听网络通道或unix socket */
network_init() ;
start_signal_handler() ; //开始接收信号
acl_init(...); //初始化ACL (Access Control List)
servers_init(0) ; //服务器初始化
init_status_vars() ; //状态变量初始化
create_shutdown_thread() ; //创建关闭线程
create_maintenance_thread() ; //创建维护线程
sq1_print_information(...); //信息输出函数,打印信息/*主要的服务处理函数,循环等待并接受命令,进行查询,返回结果,下一小节中我们详细关注该函数*/

handle_connections_sockets(0) ;
wait_for_signal_thread_to_end() ; //线程结束

/*各种清理工作*/
cleanup ;
/*正常退出*/
exit (0) ;
}
```
#### 执行流
MySQL使用函数static void handle_connections_methods()处理客户端的连接
###### 主线程
- unix socket连接主要由handle_connections_sockets(0)处理

```
pthread_handler_t handle_connect ions_sockets (void *arg_attribute__( (unused)))
{
FD_ZERO (&clientFDs) ;
FD_SET (unix_sock, &clientFDs); // unix_socket在network_init 中已被打开
socket_flags = fcntl (unix_sock, F__GETFL,0) ;
while (!abort_1oop) { // abort_loop 是全局变量,在某些情况下被置为1表示要退出
readFDs = ClientFDs; //需要监听的socket
select( (int) max_used_connection, &readFDs,0,0,0); // select异步监听,当接收到请求之后返回
sock = unix_sock;
flags = socket_flags;
fcntl (sock,F_SETFL,flags | O_NONBLOCK) ;
new_sock = accept (sock, my_reinterpret_cast (
struct sockaddr *) (&CAddr), &length); // 接受请求
getsockname (new__sock, &dummy, &dummyLen) ;
thd = new THD; /1 创建mysqld任务线程描述符,封装了客户端连接请求的所有信息
vio_tmp =
vio_new(new_sock, VIO_TYPE SOCKET,VIO_LOCALHOST); // 网络操作抽象层
my_net_init (&thd->net, vio_tmp)) ; //初始化任务线程描述符的网络操作
create_new_thread(thd) ; //创建任务线程
}
```
- 若我们在配置文件中启动了==线程缓存（写一篇文章）==，当用户完成请求后，老的线程将以休眠的状态进入线程池；若有新的连接请求，服务器会首先检查线程缓存中是否有可用的线程，如果有则唤醒该线程。
- 若没有配置线程缓存或者线程缓存中暂无可用的线程，那么一个新的线程将被创建处理用户连接请求

```
if (cached_thread_count > wake_thread)
start_cached_thread(tha) ;
}
如果cached_thread_count <= wake_thread,系统将执行下面这段代码:
if ((error=pthread_create (&thd->real_ id, &connection_attrib,
handle_one_connection, (void*) thd) ) )
```
###### 新线程
新分配的线程从sql/sql_parse.cc文件的handle_one_connections()开始执行
## 类、API和结构体
==此处暂放==
## MySQL内存分配
- MySQL实例组成包括许多内存共享块以及大量的后台线程
- MySQL的内存共享块包括：
1. 索引缓冲(Key Buffer)
2. 查询高速缓存(Query Cache)
3. 表缓存(Table Cache)
4. 线程缓存(Thread Cache)
5. 二进制日志缓冲区(Binlog Buffer)
6. InnoDB日志缓冲区(InnoDB Log Buffer)
- 用户会话也需要服务器端的内存，但此内存不共享，成为线程内存区域或者TMA（Thread Memory Area）
- 内存块的管理，MySQL比较依赖于DBA设置，目前尚无法像Oracle那样动态自动调整
#### 内存共享块
###### 索引缓冲
- MySQL为MyIsam表的索引分配了缓冲区，由所有线程共享
- 从性能实验的角度，通常为MySQL所在机器内存的25%
- MySQL使用哈希和反向链表算法快速缓存最近使用的索引块，同时又能将变动内容快速写入磁盘，参见mysys/mf_keycache.c
###### 查询缓存（query chache）（==可与postgresql比较，写一篇文章==）
- 查询缓存用来缓存特定查询的结果集，且共享给所有客户端
- 查询缓存工作原理：打开查询缓存后，服务器收到每一个查询语句都会通过哈希算法得到该查询的哈希值，然后查询缓存中查找是否有对应的结果集；如果有则直接返回结果，如果没有就进入解析、执行通道，得到的结果集又被缓存
- 表的数据发生任何变化后，与该表相关的所有query cache都会失效
- 因此查询缓存对于更新频繁的表不适用（新的版本直接取消了查询缓存）
###### 表缓存
- 当客户端程序提交查询给MySQL时,MySQL需要对查询树所涉及到的每-一个表都取得表句柄信息,如果没有表缓存,那么服务器就不得不频繁地进行打开和关闭文件的操作
- 使用存放的就是各表句柄的信息的表缓存之后,每次MySQL需要获取某个表句柄时,会先检查表缓存中是否存在已缓存的表句柄结构.执行查询时,MySQL首先会从表缓存中查找是否存在空闲状态的表文件句柄.如果有,则取出直接使用,反之,则打开文件操作获得文件句柄信息
- 实现这一操作的代码可参考sql/sql_base.cc
###### 线程缓存
连接线程是MySQL为了提高创建连接线程的效率,将部分空闲的连接线程保持在一个
缓存区以备新连接发起请求时使用,这尤其对那些使用短连接的应用程序来说,可以极大地
提高创建连接的效率
#### 线程内存区域(TMA)
每个连接使用具体线程的空间包括:
1. 堆栈(thread_stack)：主要用来存放线程描述符结构(THD), THD中包含线程id和线程运行时基本信息等,服务器通过配置参数thread_stack来设置为每个线程栈分配多大的内存
2. 连接缓存区(net_buffer_length)：连接缓存区用于缓存客户端连接线程的连接信息和将返回客户端的结果集。查询结果在被发送到网络上之前,服务器将这些信息缓存到连接缓存区，以等待累计到某个具体值时,一起发送出去
3. 数据读取缓冲区(Read Buffer)：这个区域是针对MyISAM存储引擎而言的,因为Innodb存储引擎已经自带了预取技术。该缓冲区使得大量读取文件的操作时,服务器一起读入大量信息,以减少IO次数
4. 线程内存区域还包括其他类型的区域:批量插入缓冲、链接操作缓冲、排序缓冲等
###### MySQL如何分配内存
- sql/mysqld.cc中提供宏观的内存分配接口
- mysys核心库提供具体的内存分配函数
- 在简单情况下,MySQL 服务器使用mysys/my malloc.c中的函数分配内存:
1. void* my_malloc(size. _t size, myf my. flags);
2. void* my_memdup[const void *from, size_t length, myf my_flags);
3. char* my_strdup(const char *from, myf my_fAlags);
4. char* my_strndup(const char *from, size_t length, myf my_flags);
5. void my_no_flags_free(void *ptr);
- my_malloc()的分配方法代码如下(函数中先判断size是否合理,再判断分配成功与否，最后用bzero0为分配的内存置零)
- my_memdup()函数申请一个length大小的内存空间,并复制from指向的内容,拷贝到新申请的内存空间:

```
void* my_memdup(const void *from, size_t length, myf my__flags)
{
void *ptr;
if ((ptr=my_malloc(length, my_flags)) != 0)
memcpy (ptr, from, length) ;
return(ptr) ;
}
```
- my_no_flags_free()函数无条件释放ptr指向的内存空间:

```
void my_no_flags_free(void *ptr)
DBUG_ENTER("my_free") ;
DBUG_PRINT("my", ("ptr: 0x81x", (1ong) ptr));
if (ptr)
free(ptr) ;
DBUG__VOID__RETURN;
} /* my_free */
```
## 来自祝定泽《mysql核心内幕》