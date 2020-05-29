#### 查看全部

```
postgres=# \?
General
  \copyright             show PostgreSQL usage and distribution terms
  \g [FILE] or ;         execute query (and send results to file or |pipe)
  \h [NAME]              help on syntax of SQL commands, * for all commands
  \q                     quit psql

Query Buffer
  \e [FILE] [LINE]       edit the query buffer (or file) with external editor
  \ef [FUNCNAME [LINE]]  edit function definition with external editor
  \p                     show the contents of the query buffer
  \r                     reset (clear) the query buffer
  \s [FILE]              display history or save it to file
  \w FILE                write query buffer to file

Input/Output
  \copy ...              perform SQL COPY with data stream to the client host
  \echo [STRING]         write string to standard output
  \i FILE                execute commands from file
  \ir FILE               as \i, but relative to location of current script
  \o [FILE]              send all query results to file or |pipe
  \qecho [STRING]        write string to query output stream (see \o)

Informational
  (options: S = show system objects, + = additional detail)
  \d[S+]                 list tables, views, and sequences
  \d[S+]  NAME           describe table, view, sequence, or index
  \da[S]  [PATTERN]      list aggregates
  \db[+]  [PATTERN]      list tablespaces
  \dc[S+] [PATTERN]      list conversions
  \dC[+]  [PATTERN]      list casts
  \dd[S]  [PATTERN]      show object descriptions not displayed elsewhere
  \ddp    [PATTERN]      list default privileges
  \dD[S+] [PATTERN]      list domains
  \det[+] [PATTERN]      list foreign tables
  \des[+] [PATTERN]      list foreign servers
  \deu[+] [PATTERN]      list user mappings
  \dew[+] [PATTERN]      list foreign-data wrappers
  \df[antw][S+] [PATRN]  list [only agg/normal/trigger/window] functions
  \dF[+]  [PATTERN]      list text search configurations
  \dFd[+] [PATTERN]      list text search dictionaries
  \dFp[+] [PATTERN]      list text search parsers
  \dFt[+] [PATTERN]      list text search templates
  \dg[+]  [PATTERN]      list roles
  \di[S+] [PATTERN]      list indexes
  \dl                    list large objects, same as \lo_list
  \dL[S+] [PATTERN]      list procedural languages
  \dn[S+] [PATTERN]      list schemas
  \do[S]  [PATTERN]      list operators
  \dO[S+] [PATTERN]      list collations
  \dp     [PATTERN]      list table, view, and sequence access privileges
  \drds [PATRN1 [PATRN2]] list per-database role settings
  \ds[S+] [PATTERN]      list sequences
  \dt[S+] [PATTERN]      list tables
  \dT[S+] [PATTERN]      list data types
  \du[+]  [PATTERN]      list roles
  \dv[S+] [PATTERN]      list views
  \dE[S+] [PATTERN]      list foreign tables
  \dx[+]  [PATTERN]      list extensions
  \l[+]                  list all databases
  \sf[+] FUNCNAME        show a function's definition
  \z      [PATTERN]      same as \dp

Formatting
  \a                     toggle between unaligned and aligned output mode
  \C [STRING]            set table title, or unset if none
  \f [STRING]            show or set field separator for unaligned query output
  \H                     toggle HTML output mode (currently off)
  \pset NAME [VALUE]     set table output option
                         (NAME := {format|border|expanded|fieldsep|fieldsep_zero|footer|null|
                         numericlocale|recordsep|recordsep_zero|tuples_only|title|tableattr|pager})
  \t [on|off]            show only rows (currently off)
  \T [STRING]            set HTML <table> tag attributes, or unset if none
  \x [on|off|auto]       toggle expanded output (currently off)

Connection
  \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "postgres")
  \encoding [ENCODING]   show or set client encoding
  \password [USERNAME]   securely change the password for a user
  \conninfo              display information about current connection

Operating System
  \cd [DIR]              change the current working directory
  \setenv NAME [VALUE]   set or unset environment variable
  \timing [on|off]       toggle timing of commands (currently off)
  \! [COMMAND]           execute command in shell or start interactive shell

Variables
  \prompt [TEXT] NAME    prompt user to set internal variable
  \set [NAME [VALUE]]    set internal variable, or list all if no parameters
  \unset NAME            unset (delete) internal variable

Large Objects
  \lo_export LOBOID FILE
  \lo_import FILE [COMMENT]
  \lo_list
  \lo_unlink LOBOID      large object operations
```

基本规则

1. 使用\开头，后面接简写
2. 简写命令，一般都可以加+，获得更加详细的信息
3. \d命令指describ，是最重要的元命令。默认是显示表、视图和序列，但往往后面可以跟其他单字母，扩展命令的使用
4. \d加S（大写），显示的是包括系统表的关系
5. \d等命令后可以明确紧跟对象名，精确显示该对象
6. 可以在psql里执行shell命令，改变当前目录等

#### 查看数据库
```
postgres=# \l
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
#### 查看表空间

```
postgres=# \db
       List of tablespaces
    Name    |  Owner   | Location 
------------+----------+----------
 pg_default | postgres | 
 pg_global  | postgres | 
(2 rows)
```



#### 查看表等

```
postgres=# \d
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | test   | table | postgres
 public | v_test | view  | postgres
(2 rows)

postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# \dv
         List of relations
 Schema |  Name  | Type |  Owner   
--------+--------+------+----------
 public | v_test | view | postgres
(1 row)

postgres=# \d test
     Table "public.test"
 Column |  Type   | Modifiers 
--------+---------+-----------
 a      | integer | 
 
 postgres=# \dv v_test
         List of relations
 Schema |  Name  | Type |  Owner   
--------+--------+------+----------
 public | v_test | view | postgres
(1 row)

postgres=# \di test_i
             List of relations
 Schema |  Name  | Type  |  Owner   | Table 
--------+--------+-------+----------+-------
 public | test_i | index | postgres | test
(1 row)

postgres=# \di+ test_i
                        List of relations
 Schema |  Name  | Type  |  Owner   | Table | Size  | Description 
--------+--------+-------+----------+-------+-------+-------------
 public | test_i | index | postgres | test  | 16 kB | 
(1 row)
```

#### 查看函数定义

```
\sf
```

#### 切换显示模式

类似于MySQL的\G

```
postgres=# \x
Expanded display is on.
postgres=# \dv v_test
List of relations
-[ RECORD 1 ]----
Schema | public
Name   | v_test
Type   | view
Owner  | postgres

postgres=# \dv+ v_test
List of relations
-[ RECORD 1 ]---------
Schema      | public
Name        | v_test
Type        | view
Owner       | postgres
Size        | 0 bytes
Description | 

postgres=# \x
Expanded display is off.
```



#### 获取元命令对应的SQL代码

```
[postgres@izm5edbv563hlvcbf71opez ~]# psql -E
psql (9.2.24)
Type "help" for help.

postgres=# 
postgres=# 
postgres=# \d
********* QUERY **********
SELECT n.nspname as "Schema",
  c.relname as "Name",
  CASE c.relkind WHEN 'r' THEN 'table' WHEN 'v' THEN 'view' WHEN 'i' THEN 'index' WHEN 'S' THEN 'sequence' WHEN 's' THEN 'special' WHEN 'f' THEN 'foreign table' END as "Type",
  pg_catalog.pg_get_userbyid(c.relowner) as "Owner"
FROM pg_catalog.pg_class c
     LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r','v','S','f','')
      AND n.nspname <> 'pg_catalog'
      AND n.nspname <> 'information_schema'
      AND n.nspname !~ '^pg_toast'
  AND pg_catalog.pg_table_is_visible(c.oid)
ORDER BY 1,2;
**************************

         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | test   | table | postgres
 public | v_test | view  | postgres
(2 rows)
```

#### 导入导出数据

copy和\copy都可以导入

1. COPY命令是SQL命令，\copy是元命令
2. COPY命令必须具有 SUPERUSER超级权限（将数据通过 stdin、 stdout方式导入导出情况除外），而\copy元命令不需要 SUPERUSER权限。
3. COPY命令读取或写入数据库服务端主机上的文件，而\copy元命令是从psql客户端
4. 大数据量导出到文件或大文件数据导入数据库，COPY比\copy能高

