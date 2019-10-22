原文：http://blog.itpub.net/26736162/viewspace-2651254/

## sys系统库使用基础环境
- sys系统库支持MySQL 5.6或更高版本，5.5.x及其以下版本不支持；
- 因为sys系统库提供了一些代替直接访问performance_schema的视图，所以必须启用performance_schema(performance_schema系统参数设置为ON)之后sys系统库的大部分功能才能正常使用；
- 要完全访问sys系统库，用户必须具有以下权限： 
1. 对所有sys表和视图具有SELECT权限  
1. 对所有sys存储过程和函数具有EXECUTE权限  
1. 对sys_config表具有INSERT、UPDATE权限  
1. 对某些特定的sys系统库存储过程和函数需要额外权限，如，ps_setup_save()存储过程，需要临时表相关的权限
- 还有sys系统库执行访问的对象相关的权限： 
1. 任何被sys系统库访问的performance_schema表需要有SELECT权限，如果要使用sys系统库对performance_schema相关表执行更新，则需要performance_schema相关表的UPDATE权限
2. INFORMATION_SCHEMA.INNODB_BUFFER_PAGE表的PROCESS
- 如果要充分使用sys系统库的功能，则必须启用某些performance_schema的instruments和consumers，如下： 
1. 所有wait instruments  
2. 所有stage instruments
3. 所有statement instruments
4. 对于所启用的类型事件的instruments，还需要启用对应类型的consumers(xxx_current和xxx_history_long)，要了解某存储过程具体做了什么事情可能通过show create procedure procedure_name;语句查看

#### 可以使用 sys 系统库本身来启用所有需要的 instruments 和 consumers ：
- 启用所有wait instruments：CALLsys.ps_setup_enable_instrument('wait');
- 启用所有stage instruments：CALLsys.ps_setup_enable_instrument('stage');
- 启用所有statement instruments：CALLsys.ps_setup_enable_instrument('statement')
- 启用所有事件类型的current表：CALLsys.ps_setup_enable_consumer('current');
- 启用所有事件类型的history_long表：CALLsys.ps_setup_enable_consumer('history_long');

#### 注意
1. performance_schema的默认配置就可以满足sys系统库的大部分数据收集功能。
2. 启用上述所提及的所有instruments和consumers会对性能产生一定影响，因此最好仅启用所需的配置。
3. 如果你在启用了一些默认配置之外的配置，则可以使用存储过程：CALLsys.ps_setup_reset_to_default(TRUE); 来快速恢复到performance_schema的默认配置

#### 其他说明
1. 对于以上繁杂的权限要求，通常创建一个具有管理员权限的账号即可，当然如果你有明确的需求，那另当别论，
2. sys系统库通常都是提供给专业的DBA人员排查一些特定问题使用的，其下所涉及的各项查询或多或少都会对性能有一定影响（主要体现在performance_schema功能实现的性能开销），在不明需求的情况下，不建议开放这些功能来作为常规的监控手段使用。


## sys系统库初体验
#### 查看版本
```
> select * from version;
+-------------+---------------+
| sys_version | mysql_version |
+-------------+---------------+
| 2.1.0       | 8.0.17-debug  |
+-------------+---------------+
1 row in set (0.00 sec)
```
#### 视图使用
- sys 系统库下包含许多视图，它们以各种方式对performance_schema表进行聚合计算展示
- 这些视图中大部分都是成对出现，两个视图名称相同，但有一个视图是带'x$'字符前缀的，例如：host_summary_by_file_io和x$host_summary_by_file_io，代表按照主机进行汇总统计的文件I/O性能数据，两个视图访问数据源是相同的，但是创建视图的语句中，不带x$的视图是把相关数值数据经过单位换算再显示的(显示为毫秒、秒、分钟、小时、天等)，带x$前缀的视图显示的是原始的数据(皮秒)，如下：
```
> SELECT * FROM host_summary_by_file_io;
+------------+-------+------------+
| host       | ios   | io_latency |
+------------+-------+------------+
| background | 89259 | 54.50 s    |
| localhost  |    14 | 7.26 ms    |
+------------+-------+------------+
2 rows in set (0.01 sec)

> SELECT * FROM x$host_summary_by_file_io;
+------------+-------+----------------+
| host       | ios   | io_latency     |
+------------+-------+----------------+
| background | 89259 | 54500923899227 |
| localhost  |    14 |     7255477861 |
+------------+-------+----------------+
2 rows in set (0.01 sec)

```
#### 查看视图说明
- create view 或者create table
- 访问sys 系统库开发网站https://github.com/mysql/mysql-sys上的各个.sql文件
- 使用mysqldump与mysqlpump工具导出sys库
```
// 默认情况下，mysqldump和mysqlpump都不会导出sys 系统库，需要指定
# mysqldump --databases --routines sys> sys_dump.sql
# mysqlpump sys> sys_dump.sql
```

## sys 系统库的进度报告功能
- 从MySQL 5.7.9开始，sys系统库视图提供查看长时间运行的事务的进度报告
- 通过processlist和session以及x$前缀的视图进行查看
- 中processlist包含了后台线程和前台线程当前的事件信息，session不包含后台线程和command为Daemon的线程
- session视图是直接调用processlist视图过滤了后台线程和command为Daemon的线程（所以两个视图输出结果的字段相同）
- processlist线程联结查询了threads、events_waits_current、events_stages_current、events_statements_current、events_transactions_current、sys.x$memory_by_thread_by_current_bytes、session_connect_attrs表，所以，需要打开相应的instruments和consumers，没打开对应的信息字段列就为NULL
- trx_state字段为ACTIVE的线程，progress可以输出百分比进度信息