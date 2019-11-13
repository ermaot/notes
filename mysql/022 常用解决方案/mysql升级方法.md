## 升级mysql5.5.54到mysql5.6.35

#### 1.使用mysql_upgrade直接升级数据字典库

- 1.停止mysql5.5.54
1. 修改innodb_fast_shutdown=0，以执行full purge，保证mysql干净关闭
```
set global innodb_fast_shutdown=0;
```
2. 关闭mysql服务
```
service mysqld stop
```

- 2.在mysql.cnf中添加skip_grant_tables参数
在mysql.cnf中添加skip_grant_tables参数，保证升级前不加载系统字典库
- 3.替换basedir
-  4.备份数据
备份整个data目录
- 5. 启动并升级mysql
- 6. 启动后使用mysql_upgrade升级数据字典
- 7. 去掉skip_grant_tables并重启mysql，查看是否正常
- 8.着重观察sql_mode值是否变化了


#### 2. 使用mysqldump逻辑备份数据
- 1.查看sql_mode值，并记下
- 2.加全局read lock
```
flush table with read lock
set global read_only=on
```
- 3.替换basedir
- 4.备份数据目录
- 5.加入skip_grant_tables，启动mysql，导入sql数据
- 6.执行mysql_upgrade升级字典数据库
- 7.重启数据库，并确认是否正常

#### 3.注意事项
- 对于复制架构，如果主库有低于mysql5.7版本的，从库不建议升到mysql5.7以上
- mysql5.7不允许对主键设置null，而mysql5.6允许该sql语句执行（内部会忽略这个动作）