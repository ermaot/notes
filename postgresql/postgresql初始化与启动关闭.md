## 创建操作系统用户

```
# groupadd -g 1000 postgres
# useradd -g 1000 -u 1000 postgres
# id postgres
uid=1000(postgres) gid=1000(postgres) groups=1000(postgres)

```

**注：如果使用yum安装，用户和用户组就直接存在了，无需额外创建**

## initdb数据库

一定要initdb，否则会报以下错误

```
May 19 17:26:12 izm5edbv563hlvcbf71opez postgresql-check-db-dir[13693]: "/var/lib/pgsql/data" is missing or empty.
May 19 17:26:12 izm5edbv563hlvcbf71opez postgresql-check-db-dir[13693]: Use "postgresql-setup initdb" to initialize the database cluster.
```



```
# initdb -D ./data/  -W
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

fixing permissions on existing directory ./data ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 32MB
creating configuration files ... ok
creating template1 database in ./data/base/1 ... ok
initializing pg_authid ... ok
Enter new superuser password: 
Enter it again: 
setting password ... ok
initializing dependencies ... ok
creating system views ... ok
loading system objects' descriptions ... ok
creating collations ... ok
creating conversions ... ok
creating dictionaries ... ok
setting privileges on built-in objects ... ok
creating information schema ... ok
loading PL/pgSQL server-side language ... ok
vacuuming database template1 ... ok
copying template1 to template0 ... ok
copying template1 to postgres ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    postgres -D ./data
or
    pg_ctl -D ./data -l logfile start
```

## 启动数据库

```
# service postgresql start
Redirecting to /bin/systemctl start postgresql.service
```

或者

```
# pg_ctl -D data/   start
server starting
```



## 查看数据库状态

```
# su - postgres
# pg_ctl -D data/ status
pg_ctl: server is running (PID: 13827)
/usr/bin/postgres "-D" "/var/lib/pgsql/data" "-p" "5432"
```

## 关闭数据库

关闭数据库因为紧急程度，有多种模式

smart:等待活动事务结束，并且等待客户端主动断开连接后才关闭

fast:回滚所有活动的事务，并主动断开客户端的连接

immediate:立刻关闭数据库，下一次启动会进入到恢复模式

三种情况可以分别简写为:-ms   -mf  -mi

```
# pg_ctl -D data/  -ms stop
waiting for server to shut down................................................... done
server stopped
```

