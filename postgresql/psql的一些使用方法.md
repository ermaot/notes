#### 查看全部参数

```
[postgres@izm5edbv563hlvcbf71opez ~]# psql --help
psql is the PostgreSQL interactive terminal.

Usage:
  psql [OPTION]... [DBNAME [USERNAME]]

General options:
  -c, --command=COMMAND    run only single command (SQL or internal) and exit
  -d, --dbname=DBNAME      database name to connect to (default: "postgres")
  -f, --file=FILENAME      execute commands from file, then exit
  -l, --list               list available databases, then exit
  -v, --set=, --variable=NAME=VALUE
                           set psql variable NAME to VALUE
  -V, --version            output version information, then exit
  -X, --no-psqlrc          do not read startup file (~/.psqlrc)
  -1 ("one"), --single-transaction
                           execute command file as a single transaction
  -?, --help               show this help, then exit

Input and output options:
  -a, --echo-all           echo all input from script
  -e, --echo-queries       echo commands sent to server
  -E, --echo-hidden        display queries that internal commands generate
  -L, --log-file=FILENAME  send session log to file
  -n, --no-readline        disable enhanced command line editing (readline)
  -o, --output=FILENAME    send query results to file (or |pipe)
  -q, --quiet              run quietly (no messages, only query output)
  -s, --single-step        single-step mode (confirm each query)
  -S, --single-line        single-line mode (end of line terminates SQL command)

Output format options:
  -A, --no-align           unaligned table output mode
  -F, --field-separator=STRING
                           set field separator (default: "|")
  -H, --html               HTML table output mode
  -P, --pset=VAR[=ARG]     set printing option VAR to ARG (see \pset command)
  -R, --record-separator=STRING
                           set record separator (default: newline)
  -t, --tuples-only        print rows only
  -T, --table-attr=TEXT    set HTML table tag attributes (e.g., width, border)
  -x, --expanded           turn on expanded table output
  -z, --field-separator-zero
                           set field separator to zero byte
  -0, --record-separator-zero
                           set record separator to zero byte

Connection options:
  -h, --host=HOSTNAME      database server host or socket directory (default: "local socket")
  -p, --port=PORT          database server port (default: "5432")
  -U, --username=USERNAME  database user name (default: "postgres")
  -w, --no-password        never prompt for password
  -W, --password           force password prompt (should happen automatically)

For more information, type "\?" (for internal commands) or "\help" (for SQL
commands) from within psql, or consult the psql section in the PostgreSQL
documentation.

Report bugs to <pgsql-bugs@postgresql.org>.
```

#### 查看全部数据库

```
# psql -l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)
```

与在psql里执行\l是一样的

#### 格式设置

###### -A显示非对齐模式

```
# psql -A -c "select * from test order by a "
a|b
1|test    
2|2       
3|test3   
4|4       
(4 rows)

# psql  -c "select * from test order by a "
 a |    b     
---+----------
 1 | test    
 2 | 2       
 3 | test3   
 4 | 4       
(4 rows)
```

###### -t 只显示数据

```
# psql  -t -c "select * from test order by a "
 1 | test    
 2 | 2       
 3 | test3   
 4 | 4       
```

###### -q不显示输出信息

```
# psql  -q -c "create table test2(a int) " 
# psql  -c "select * from test2 order by a " 
 a 
---
(0 rows)
```

可以看到没有显示"create table"这种信息

###### -f执行sql文件

###### 传递变量

- psql内部定义变量

```
postgres=# \set value 1
postgres=# select * from test where a=:value;
 a |    b     
---+----------
 1 | test    
(1 row)
```

取消变量定义

```
postgres=# \set value
postgres=# select * from test where a=:value;
ERROR:  syntax error at or near ";"
LINE 1: select * from test where a=;
                                   ^
```

- psql 命令行传递变量

```
# psql  -v value=1 -f 1.sql 
 a |    b     
---+----------
 1 | test    
(1 row)
```

其中1.sql内容是

```
select * from test where a=:value;
```

###### 通过传递变量的方式定义脚本

修改或者新建.psqlrc文件，写入如下内容（查询活动会话，记忆pg_stat_activity视图）：

```
\set active_session 'select pid, usename, datname, query, client_addr from pg_stat_activity where pid <> pg_backend_pid() and state=\'active\' order by query;'
```

这样可以将定义的变量传递到psql中

```
postgres=# :active_session 
 pid | usename | datname | query | client_addr 
-----+---------+---------+-------+-------------
(0 rows)
```

其实也可以在psql中直接定义，当前会话中可以一直使用

如下是第二个例子。修改或者新建.psqlrc：

```
\set wait_event 'select pid,usename,datname,query,client_addr  from pg_stat_activity where pid <> pg_backend_pid() ;'
\set connections 'select datname, usename, client_addr, count(*) from pg_stat_activity where pid <> pg_backend_pid() group by 1,2,3 order by 1,2,4 desc;'
```

进入psql执行变量，即执行sql

```
postgres=# :wait_event
 pid | usename | datname | query | client_addr 
-----+---------+---------+-------+-------------
(0 rows)
postgres=# :connections
 datname  | usename  | client_addr | count 
----------+----------+-------------+-------
 postgres | postgres |             |     1
(1 row)
```

###### psql可以tab自动补全

这个类似于mysql的auto-hash，但比auto-hash更好用

###### 客户端提示符定制

参数|解释
---|---
%M|数据库服务器别名，不是指主机名，显示的是psql的-h参数设置的值；<br>当连接建立在Unix域套字上时则是【local】
%>|数据库服务器的端口号。
%n|数据库会话的用户名，在数据库会话期间<br>这个值可能会因为命令SET SESSION UTHORIZATION的结果而改变。
%/|当前数据库名称。
%#|如果是超级用户则显示“#”，其他用户显示“>”，在数据库会话期间，<br>这个值可能会因为命令 SET SESSION AUTHORIZATION的结果而改变。
%p|当前数据库连接的后台进程号
%R|在 PROMPT1中通常显示“=”，如果进程被断开则显示“！”。

```
postgres=# \echo :PROMPT1
%/%R%# 
postgres=# \set PROMPT1 %/%M%R%#
postgres[local]=#
```

