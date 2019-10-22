## 源码结构概览
![源码结构](https://github.com/ermaot/notes/blob/master/mysql/005mysql%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84/pic/mysql%E6%BA%90%E7%A0%81%E7%BB%93%E6%9E%84.png)

==来自622463 MySQL运维内参：MySQL、Galera、Inception核心原理与最佳实践==
文件夹|说明
---|---
BUILD|里面包含各个平台、各种编译器下进行编译的脚本
CMakeLists. txt |CMake入口编译文件
client|客户端工具，所有的客户端工具都在这里，比如 mysql、 mysqlbinlog、 mysqladmin mysqldump等cmake为 CMake编译服务的，这里定义了很多在 CMake编译时使用的方法或变量
cmd-line-utils|一些小工具
config.h.cmake|用于生成编译时配置头文件的 cmake文件
dbug|提供一些调试用的宏定义，可以很好地跟踪数据库执行到的执行函数、运行栈桢等信息，可以定位一些问题
extra|包含了用来做网络消息认证的SSL包，并提供了 comp-err， resolver等一些小工具
include|MySQL代码包含的所有头文件，这里不包括存储引擎的头文件
libbinlogevents| MySQL57版本开始新增的、用于解析 Binlog的lib服务
libmysql|用来创建嵌入式系统的 MySQL客户端程序API 
libmysqld| MySQL服务器的核心级API文件，也用来开发嵌入式系统
mysql-test| mysqld的测试工具
mysys |MySQL自己实现的一些常用的数据结构和算法，比如aray、list和hash，以及些区分不同底层操作系统平台的函数封装，比如 my_file、my_fopen等函数，这类型的函数都以my开头
mysys_ssl |MySQL中SSL相关的服务
plugin|包括一些系统内置的插件，比如auth、 password_validation等，同时包含了可动态载入的插件，比如 fulltext、 semisync等
regex|一些关于正则表达式的算法
scripts|实现包含一些系统工具脚本，比如 mysql_install_db、 mysqld_safe及 mysql_multi等
sql |MySQL服务器主要代码，这里包含了main函数（ maIn cc），将会生成 mysqld可执行文件
sql-common|存放部分服务器端和客户端都会用到的代码
storage|所有存储引擎的源代码都在这个目录中，文件夹名一般就是其存储引擎的名称，包括 innobase、 myisam、 blackhole、ndb及 perfschema等
strings|包含了很多关于字符串处理的函数，比如 strmov、 strapped及 my_atof等函数
support- files| my.cnf示例配置文件及编译所需的一些工具
unittest|单元测试文件目录
vio|虚拟网络IO处理系统，是对不同平台或不同协议的网络通信API的封装
win|在 windows平台编译所需的文件和一些说明
zlib|zlib压缩算法库（GNU）
## 主要关键目录
1. BUILD
2. client
3. storage
4. mysys
5. sql
6. vio
#### BUILD
- BUILD是编译和安装脚本目录
- 比如目录下的 compile-pentium64-debug 文件，调用了SETUP.sh 和 FINISH.sh

```
path=`dirname $0`
set -- "$@" --with-debug=full
. "$path/SETUP.sh"

extra_flags="$pentium64_cflags $debug_cflags"
extra_configs="$pentium_configs $debug_configs $static_link"

extra_configs="$extra_configs "
CC="$CC --pipe"
. "$path/FINISH.sh"
```
#### client
- 在这里,你可以找到亲切熟悉的mysql、mysqladmin、 mysqlshow 等常用命令和客户端工具的源代码

大小 |文件名 |注释
---|---|---
137966 |mysql.cc| MySQL客户端
37139 |mysqladmin.c| mysqladmin工具,mysqladmin用于服务器的运作
24631| mysqlshow.c| mysqlshow,显示数据库、表和列等
- MySQL的绝大部分代码是由C或者C++写成,大部分代码都是以.c、.cc 或h结尾的
- 还包括密码确认功能get_password.c、SSL连接可行性检查等功能

#### storage
- 该目录存放MySQL存储引擎的代码
#### mysys
- mysys代表MySQL system library,是MySQL的库函数文件。库函数是一些预先编译好的函数的集合。这些库文件编译以后在Windows上是.lib 和.dll文件,在Linux和Unix上是.so和.a文件
- mysys目录其实就是个大杂烩,包含了各种各样的功能库文件,包括==文件打开、数据读写、内存分配、OS/2系统特别优化、线程控制、权限控制、Raid Table、动态字符串处理、队列算法、网络传输协议、初始化函数、错误处理、平衡二叉树算法、符号连接处理、唯一临时文件名生成、hash函数、排序算法、压缩传输协议==等


#### sql
- MySQL源代码中需要经常变化的目录之一,另外大部分已有bug也来自于该目录中的文件。20MB左右的代码量,说明它是MySQL服务器内核最为核心和重要的目录
- [线程、查询解析和查询优化器和存储引擎接口)的大部分篇幅也用于描述这个目录中的文件
- 包含mysqld.cc MySQL main函数
-还包括各类SQL语句的解析/实现

文件|说明
---|---
sql_lex.cc| 词法解析模块<p>lex是词法分析程序的自动生成工具。它输入描述构词规则的一系列正则表达式,然后构建有穷自动机和这个有穷自动机的一个驱动程序,进而生成词法分析程序
sql_yacc.yy |语法解析模块<p>yacc(Yet Another Compiler Compiler),是Unix/Linux上用来生成编译器的编译器
- ==这两个文件决定了MySQL如何解析输入的字符流和SQL语句,并最终获得解析树==
- 存储引擎接口模块的代码/storage下的各存储引擎目录中,存在的是各类存储引擎的实现代码(ha_innodb、ha_myisam等类),而/sql下存放的是处理接口handler,handler中有很多虚函数，需要其子类来实现

```
virtual void change_table_ptr (TABLE *table_arg,TABLE_SHARE *share)
virtual double scan_time ()
virtual double read_time (uint index, uint ranges, ha_rows rows)
virtual double index__only_read_.t ime (uint keynr, double records)
```
- 各种SQL语句的执行代码也可以在sql目录中找到:

文件 | 说明
---|---
sql_delete.cc |delete语句
sql_do.cc |操作语句
sq1_help.cc |help,帮助语句
sql_insert.cc| 插入语句
sql_select.cc |select语句
sql_.show. cC| show语句
sql_update.cc |update操作
这类文件常以sql开始对文件命名。以此类推,insert 语句可查看sql_insert.cc
- MySQL语句的SQL函数代码同样也在sql目录下。MySQL将UNION和ROLLUP等操作,看作内部函数:

文件 | 说明
---|---
sq1_string.c |处理string的各种函数
sql_olap.c |处理olap的各种函数,目前对于MySQL只是rollup

#### vio
VIO意指Virtual I/0,主要用来处理各种网络协议的IO。Virtual I/O使得各种模块的网络协议能够无缝地调用I/O功能.

## mysql执行流
用户执行SELECT语句后,MySQL 的执行流：
1. 第一步,客户端应用程序mysql, 发送了一个SQL语句
2. MySQL 服务器通过vio模块收到了网络传输过来的SQL语句
3. 下一步,Lex和YACC将SQL语句解析并生成语法树,该语法树最终由底层sql目录中sql  select.cc执行
4. 独立的handler.cc接受到最后的请求,并执行这个语句
5. Handler 依靠storage目录下的具体存储引擎代码和函数读取数据

## 开源模块
1. dbug：Fred Fish提供，编译时使用with-debug会显示dbug输出
2. pstack：显示进程的stack信息
3. regex：Henry Spencer提供，执行正则匹配函数REGEXP时，会需要
4. strings
5. zlib
## 操作系统相关代码目录
1. netware
2. win
## 例解 sql_delete.cc
- sql_delete.cc 是delete语句代码，delete和truncate语句会调用（==包括多表删除==（==额外写一篇关于多表删除==））

```
/*实现MySQL DELETE语句与其他的SQL语句,如INSERT、UPDATE一样,dispatch_command()函数会调用这些函数*/
bool mysql_delete(THD *thd, TABLE_LIST *table_list, COND *conds,SQL_LIST *order, ha_rows limit, ulonglong opt ions,bool reset_auto_increment)
{
bool will_batch;
int error, 1oc_error;
/*


/*下面这个函数为DELETE语句准备Item类.
参数说明:
mysql_prepare_delete()
thd -线程描述符
table_list -全局或本地表的列表
conds -条件参数
*/
int mysq1_prepare_delete(THD *thd, TABLE_LIST *tab1e_list, Item **conds)
Item *fake_conds = 0;
SELECT_LEX *select_1ex = &thd->1ex->select_1ex;
DBUG_ENTER( "mysq1_prepare_delete") ;
List<Item> all_fields; /* 将数据表的所有列放到Item对象的容器*/
if (thd->lex->current_select->select_limit)
tha->lex->set_stmt_unsafe() ; /* 在基于语句复制功能中,delete .... limit语句是不安全的,因为行顺序是未定的*/
thd->set_current_stmt_binlog_row_based_if_mixed(); /* 如果复制功能是基于混合模式,那么我们将delete操作转化为基于行的复制*/
}




```


==这里关于delete 和truncate 额外写一篇==
```

/***************************************************
TRUNCATE表
********************************************************************* /
/*
某些存储引擎,如InnoDB的存储引擎,无法支持表重建,故此对这类表的truncate操作是按照一行一行地刪除
*/

static bool mysq1_truncate_by_delete (THD *thd, TABLE LIST *table__list)
bool error, save_binlog_row_based = thd->current_stmt_binlog_row_based;
DBUG_ENTER("mysql_truncate_by_delete") ;
table_list->lock_type= TL_WRITE;
mysq1_init_select (thd->lex) ;
thd->clear_current_stmt_binlog_row_based() ;
error =mysq1_delete(thd,table_list, NULL, NULL,HA_POS_ERROR, LL(0), TRUE);
ha_autocommit_or_rollback(thd, error) ;
end_trans(thd, error ? ROLLBACK : COMMIT); //事务处理
tha->current_stmt_binlog_row_based =save_binlog_row_based; / /基于语句的二进制日志
DBUG_RETURN (error) ;
```

所有行删除操作被优化成自动转化为表重建，即使MYD和MYI表损坏也如此<p>
在下列情况中,经常需要设置dont_ send_ ok:<p>
我们不想发送OK给客户端(don't want an ok to be sent to the end user)<p>
我们不想truncate命令被日志记录下来
*/

```
bool mysq1_truncate(THD *thd, TABLE LIST *table_list, bool dont_send_ok){
HA_CREATE_INFO create_info;
char path[FN_REFLEN]; //记录表在文件系统中的文件地址
TABLE *table;
bool error;
uint path_length;
DBUG_ENTER( "mysql_truncate") ;
bzerol(char*) &create__info, sizeof (create_info));
/*如果是临时表,则关闭这个临时表并重建该表*/
if (!dont_send ok && (table= find temporary_table(thd, table_list))){
handlerton *table_type = table->s->db_type();
TABLE_SHARE *share = table->s;
if (!ha_check_storage_engine_flag(table_type, HTON_CAN_RECREATE) )
goto trunc_by_del;
```
## ==写一篇mysql编译的文章==
本文来自《MySQL核心内幕-祝定泽》