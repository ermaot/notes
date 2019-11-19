本文来自https://www.cnblogs.com/wclwcw/p/6933294.html
## 插件类型
当前audit 插件有以下几种：
- 1、MySQL Enterprise Audit Plugin – MySQL企业版本才能使用
- 2、Percona Audit Log Plugin – 只能给Percona_sever使用，Percona来维护

- 3、McAfee MySQL Audit Plugin
下载地址：http://dl.bintray.com/mcafee/mysql-audit-plugin/
部署可参考：http://blog.csdn.net/bzfys/article/details/53695855

- 4、MariaDB Audit Plugin 
下载地址：https://mariadb.com/kb/en/mariadb/about-the-mariadb-audit-plugin/ （可以直接下载MariaDB对应的版本后，解压后在plugin目录下有server_audit.so插件）
MariaDB_5.5.37版本和MariaDB_10.0.10以后版本的audit插件支持MariaDB, MySQL、Percona Server使用
备注：MariaDB_5.x.x和MariaDB_10.x.x区别
MariaDB_5.x.x：兼容MySQL5.x.x的，接口几乎一致，只限于社区版
MariaDB_10.x.x：10.x.x使用新技术，接口会与mysql逐渐区别开来。目标就是以后想MariaDB新接口过渡


## 安装MariaDB Audit Plugin 
1. 下载mariadb-5.5.56-linux-x86_64.tar.gz解压获取server_audit.so插件
2. 登录MySQL，执行命令获取MySQL的plugin目录
```
l> show variables like "%plugin_dir%";
+---------------+-------------------------------+
| Variable_name | Value                         |
+---------------+-------------------------------+
| plugin_dir    | /root/mysql_debug/lib/plugin/ |
+---------------+-------------------------------+
1 row in set (0.01 sec)
```
3. 将server_audit.so上传到/root/mysql_debug/lib/plugin/下
4. install
```
> INSTALL PLUGIN server_audit SONAME 'server_audit.so';
Query OK, 0 rows affected (0.87 sec)
```
5. 查看变量开启设置情况，默认貌似都是关闭的
```
> show variables like '%audit%';
+-------------------------------+-----------------------+
| Variable_name                 | Value                 |
+-------------------------------+-----------------------+
| server_audit_events           |                       |
| server_audit_excl_users       |                       |
| server_audit_file_path        | server_audit.log      |
| server_audit_file_rotate_now  | OFF                   |
| server_audit_file_rotate_size | 1000000               |
| server_audit_file_rotations   | 9                     |
| server_audit_incl_users       |                       |
| server_audit_logging          | OFF                   |
| server_audit_mode             | 0                     |
| server_audit_output_type      | file                  |
| server_audit_query_log_limit  | 1024                  |
| server_audit_syslog_facility  | LOG_USER              |
| server_audit_syslog_ident     | mysql-server_auditing |
| server_audit_syslog_info      |                       |
| server_audit_syslog_priority  | LOG_INFO              |
+-------------------------------+-----------------------+
15 rows in set (0.01 sec)
```
6. 编辑my.cnf,添加配置
```
server_audit_events='CONNECT,QUERY,TABLE,QUERY_DDL,QUERY_DML,QUERY_DCL'
备注：指定哪些操作被记录到日志文件中
server_audit_logging=on
server_audit_file_path =/data/mysql/auditlogs/
备注：审计日志存放路径，该路径下会生成一个server_audit.log文件，就会记录相关操作记录了
server_audit_file_rotate_size=200000000
server_audit_file_rotations=200
server_audit_file_rotate_now=ON
```
7.  重启数据库
