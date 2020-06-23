
MySQL5.7以后的版本，安装成功之后不支持默认的无密码登录

```
# service mysqld start
Redirecting to /bin/systemctl start mysqld.service
# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```
1. 获取MySQL的初始化密码

```
 cat /var/log/mysqld.log 
2019-07-16T01:05:21.390057Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.16) initializing of server in progress as process 11868
2019-07-16T01:05:24.924594Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: e6&d7pPr>B7m
2019-07-16T01:05:26.315439Z 0 [System] [MY-013170] [Server] /usr/sbin/mysqld (mysqld 8.0.16) initializing of server has completed
2019-07-16T01:05:28.009342Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.16) starting as process 11916
2019-07-16T01:05:28.579964Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2019-07-16T01:05:28.643781Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.16'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server - GPL.
2019-07-16T01:05:28.663993Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock' bind-address: '::' port: 33060
```

2. 使用默认密码登录
3. 如果不修改密码，MySQL会报错

```
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```
4. 默认的密码规则比较强，如果不想那么麻烦就先修改密码的强度规则

```
MySQL 5.x
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;

MySQL 8.0
mysql> set global validate_password.policy=0;
mysql> set global validate_password.length=1;

mysql> alter user 'root'@'localhost' identified by '123456';
Query OK, 0 rows affected (0.05 sec)
```
