## 查看当前用户和版本
用户
```
> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```
当前版本
```
> select version();
+--------------+
| version()    |
+--------------+
| 8.0.17-debug |
+--------------+
1 row in set (0.00 sec)
```

## 删除非root或非localhost用户并修改root密码
- 可以使用mysql_secure_installation安全加固，一步步执行即可
#### 删除非root或非localhost用户并修改root密码
如果是5.7以前
```
delete from mysql.user where user !="root" or host !="localhost";
```
如果是5.7以后或者8
```
delete from mysql.user where user not in ("mysql.sys","mysql.session","mysqlxsys","mysql.infoschema") or host not in ("localhost");
```

## 删除test库
```
drop database test;
```

## 清理mysql.db表
```
truncate mysql.db
```