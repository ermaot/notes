mysql日志包括：

1. 普通查询日志：general_log
2. 慢查询日志：slow_query_log
3. 错误日志：log_error
4. 二进制日志：binlog
5. 中继日志：relay_log
6. DDL日志：ddl_log

## log的路径
默认情况下，日志都放在datadir下
```
> show variables like "%datadir%";
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.01 sec)

> show variables like "%log%";
+--------------------+--------------------------------------------------------+
| Variable_name      | Value                                                  |
+--------------------+--------------------------------------------------------+
| ………………………………                                                                |
| general_log        | OFF                                                    |
| general_log_file   | /var/lib/mysql/izm5edbv563hlvcbf71opjz.log             |
| ………………………………                                                                |
| log_bin_basename   | /var/lib/mysql/binlog                                  |
| log_bin_index      | /var/lib/mysql/binlog.index                            |
| ………………………………                                                                |
| log_error          | /var/log/mysqld.log                                    |
| ………………………………                                                                |
| relay_log_basename | /var/lib/mysql/izm5edbv563hlvcbf71opjz-relay-bin       |
| relay_log_index    | /var/lib/mysql/izm5edbv563hlvcbf71opjz-relay-bin.index |
| relay_log_info_file| relay-log.info                                         |
|……………………………………                                                               |
| slow_query_log     | OFF                                                    |
| slow_query_log_file| /var/lib/mysql/izm5edbv563hlvcbf71opjz-slow.log        |
| ……………………………………                                                              |
+--------------------+--------------------------------------------------------+

```
- 默认情况下，所有日志都写到磁盘中，但普通查询日志和慢查询日志可以通过log_output=TABLE保存到mysql.general_log和mysql.slow_log表中
- DDL日志在mysql8中可以配置：可以打印到错误日志，也可以保存在innodb_ddl_log表中
- 默认情况下，二进制日志根据max_binlog_size大小自动滚动，中继日志根据max_relay_log_size滚动（如果max_relay_log_size=0，则按照max_binlog_size滚动）
- 日志增长过大之后，切割方法：
1. 移走原始文件内容，然后生成一个空文件（flush [binary|error|general|relay] logs），定期重复该步骤
2. 登录数据库实例使用flush tables或者flush tables with read lock
3. 使用命令行工具选项产生新的日志文件（本质也是执行刷新语句），比如使用mysqladmin的flush-logs选项或者mysqldump 命令的flush-logs和--master-data选项

## 日志表详解
#### 1. general_log
- 提供查询普通SQL的执行信息。还可以用企业版的audit log插件（参见mysql工具/audit插件）
- 按照接收请求的顺序将语句写入查询日志中（可能与执行顺序不同）
- 主从复制架构：
1. 主库上使用statement日志格式时，从库在重放语句后，会记录到自己的查询日志中。mysqlbinlog命令解析并导入，也会记录
2. 主库上使用row的日志格式，从刘重放变更之后，不记录从库查询日志
3. 主库上使用基于mixed日志格式，statement记录的语句会记录到从库的查询日志，否则不记录
- 可以使用sql_log_off系统变量动态关闭当前会话（不使用global）或者所有会话的查询日志功能（使用global）
- 默认情况下，服务器执行的语句若带有用户密码，则该语句被服务器重写后再写入查询日志；使用--low-raw选项启动服务器会明文记录密码；不建议明文记录
- --low-raw会记录所有原始sql语句（包括有语法错误的语句）

#### 2.slow_log
- slow_log表提供查询执行时间超过long_query_time设置值的SQL语句、未使用索引的语句（开启参数long_queries_not_using_indexes=on）、管理语句（log_sloq_admin_statement=on，管理语句包括alter table、analyze table、check table、create index 、drop index、optimize table、repair table）
- 默认情况下，慢查询日志不记录管理语句和未使用索引语句
- 慢查询语句只记录获取锁之后释放锁之前的这段时间
- 几个参数：
1. 启用慢查询日志，使用--slow_query_log=1
2. 指定慢查询日志文件名称，使用--slow_query_log_file=file_name
3. 指定输出目标，使用--log-output=FILE|TABLE|NONE
4. 输出信息的详细程度：--log-short-format（启动时）
- 慢查询日志判断步骤：  
1. 判断log_slow_admin_statements
2. 判断log_queries_not_using_indexes
3. 判断是否超过log_query_time
4. 判断min_examined_row_limit，如果超过该值，则记录，否则不记录