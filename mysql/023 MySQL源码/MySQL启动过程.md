

本文来自MySQL运维内参

## 创建数据库

- my.cnf配置
```
[mysqld]
port=3306
datadir=/var/mysql/data_3306
log error=/var/mysql/data_3306/errorlog
basedir=/var/mysql/
```
2 初始化

```
/var/ mysql/bin/mysqld --defaults-file=/var/mysql/data_3306/my.cnf --initialize
--user=mysql
```
3. 启动数据库
```
/var/mysql/bin/mysqld --defaults-file=/var/mysql/data_3306/my.cnf --user=mysql
```
mysql5.7以后的初始密码要从errorlog中获取，如果不想这样，可以初始化的时候，加 --initialize-insecure参数


## mysql启动过程
1. mysqld服务器是C++生成的可执行文件，main()函数是总的入口函数
2. 入口函数在sql/main.cc中

sql/main.cc
```
extern int mysqld_main(int argc, char **argv);

int main(int argc, char **argv) { return mysqld_main(argc, argv); }
```
3. 所有操作在mysqld_main中完成


```
int mysqld_main(int argc, char **argv) {
……
//
if (load_defaults(MYSQL_CONFIG_NAME, load_default_groups, &argc, &argv,
                    &argv_alloc)) {
    flush_error_log_messages();
    return 1;
  }
}
```

| 文件        | 步骤                                                         | 解释                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| mysqld_main | load_defaults()                                              | 处理配置文件及启动参数等。如果有defaults-file则读取，没有则从特定路径查找文件 |
| mysqld_main | handle_early_options()                                       | 继续处理参数变量                                             |
| mysqld_main | logger.init_base()                                           | 日志系统初始化                                               |
| mysqld_main | init_common_variables()                                      | 初始化很多系统内部变量                                       |
| mysqld_main | my_init_signals()                                            | 信号系统初始化                                               |
| mysqld_main | init_server_componets()                                      | 核心模块启动，包括存储引擎等                                 |
| mysqld_main | reopen_fstream()                                             | 终端重定向处理                                               |
| mysqld_main | netword_init()                                               | 网络初始化                                                   |
| mysqld_main | init_status_vars()                                           | 状态量初始化                                                 |
| mysqld_main | lchecl_binlog_cache_size(NULL)<br>check_binlog_stmt_cache_size(NULL) | binlog相关检查初始化                                         |
| mysqld_main | handle_connections_sockets()                                 | 服务器监听线程创建                                           |
| mysqld_main |                                                              | 服务器启动线程等待                                           |

#### 导入参数

handle_default_option()函数，导入到内存使用数据结构如下：

```
struct handle_option_ctx{
MEM_ROOT *alloc
DYNAMIC_ARRAY *args;
TYPELIB *group;
}

```

