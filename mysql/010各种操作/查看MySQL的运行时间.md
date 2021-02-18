## 1.uptime

````
> show status like "uptime";
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| Uptime        | 427195 |
+---------------+--------+
1 row in set (0.01 sec)

> show global status like 'uptime';
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| Uptime        | 428200 |
+---------------+--------+
1 row in set (0.00 sec)

```
\s ==等同于status==
```
> \s
--------------
mysql  Ver 15.1 Distrib 5.5.56-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:		6148
Current database:	perf
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server:			MariaDB
Server version:		5.5.56-MariaDB MariaDB Server
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/var/lib/mysql/mysql.sock
Uptime:			4 days 22 hours 56 min 59 sec

Threads: 2  Questions: 234443  Slow queries: 2  Opens: 185  Flush tables: 2  Open tables: 191  Queries per second avg: 0.547
--------------
````



## 2. ps

````
# ps -ef | grep mysqld
mysql      385 32585  0 6月23 ?       00:20:05 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock
````

可以看到启动时间是6月23

## 3. **phpmyadmin**

## 4. mysqladmin

```
./mysqladmin -uroot -p -i 2 -c 5 status
Enter password: 
Uptime: 28705618  Threads: 6  Questions: 1717  Slow queries: 0  Opens: 1035  Flush tables: 3  Open tables: 867  Queries per second avg: 0.000
Uptime: 28705620  Threads: 6  Questions: 1718  Slow queries: 0  Opens: 1035  Flush tables: 3  Open tables: 867  Queries per second avg: 0.000
```



