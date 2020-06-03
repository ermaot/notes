使用gmake命令编译安装postgresql时，如果指定world选项，将安装一些扩展模块到$PGHOME/share/extension目录

```
# gmake world
# gmake install-world
```

可以在$PGHOME/share/extension下看到以下文件：

```
………………
-rw-r--r-- 1 postgres postgres 1246 May 14 20:25 pg_stat_statements--1.0--1.1.sql
-rw-r--r-- 1 postgres postgres 1336 May 14 20:25 pg_stat_statements--1.1--1.2.sql
-rw-r--r-- 1 postgres postgres 1454 May 14 20:25 pg_stat_statements--1.2--1.3.sql
-rw-r--r-- 1 postgres postgres  345 May 14 20:25 pg_stat_statements--1.3--1.4.sql
-rw-r--r-- 1 postgres postgres  305 May 14 20:25 pg_stat_statements--1.4--1.5.sql
-rw-r--r-- 1 postgres postgres 1427 May 14 20:25 pg_stat_statements--1.4.sql
-rw-r--r-- 1 postgres postgres  376 May 14 20:25 pg_stat_statements--1.5--1.6.sql
-rw-r--r-- 1 postgres postgres  191 May 14 20:25 pg_stat_statements.control
-rw-r--r-- 1 postgres postgres  449 May 14 20:25 pg_stat_statements--unpackaged--1.0.sql
………………
```

然后使用create extension命令安装

```
# create extension [ if not exists ] pg_stat_statement;
```

使用\dx查看已安装的extension

```
# \dx
                                     List of installed extensions
        Name        | Version |   Schema   |                        Description                        
--------------------+---------+------------+-----------------------------------------------------
 file_fdw           | 1.0     | public     | foreign-data wrapper for flat file access
 pg_stat_statements | 1.6     | public     | track execution statistics of all SQL statements executed
 plpgsql            | 1.0     | pg_catalog | PL/pgSQL procedural language
(3 rows)
```

或者使用pg_extension表查看

```
# select * from pg_extension ;
  extname      | extowner | extnamespace | extrelocatable | extversion | extconfig | extcondition 
--------------------+----------+--------------+----------------+------------+-----------+------
 plpgsql            |       10 |           11 | f              | 1.0        |           | 
 pg_stat_statements |       10 |         2200 | t              | 1.6        |           | 
 file_fdw           |       10 |         2200 | t              | 1.0        |           | 
(3 rows)
```

#### pg_stat_statement

pg_stat_statement收集SQL运行信息，例如总运行时间，调用次数，共享内存命中情况等.

在postgresql.conf配置如下参数

```
shared_preload_libraries='pg_stat_statements'				#需重启postgresql
pg_stat_statements.max_=10000								#记录最大SQL数
pg_stat_statements.track=all								#all是指全部SQL，top指最外层SQL
pg_stat_statements.track_utility_=on						#是否记录增删改查以外的SQL
pg_stat_statements.save=on									#数据库关闭时是否保存到文件中
```

#### auto_explain

postgresql默认不提供历史查询计划的查看，但可以通过加载扩展模块实现

现在postgresql.conf配置如下参数

```
shared_preload_libraries='pg_stat_statements,auto_explain'  
auto_explain_log_min_duration=0								#SQL的执行时间，单位为毫秒，-1表示不启用
auto_explain_log_analyze=on									#是否输出analyze模式，默认为off
auto_explain_log_buffers=off								#explain是否启用buffers，默认为off
```

```
# explain  (analyze true, buffers true ) select * from passwd order by uid desc limit 5;
                                                    QUERY PLAN                                                     
-------------------------------------------------------------------------------------------------------------------
 Limit  (cost=1.80..1.81 rows=5 width=168) (actual time=0.064..0.065 rows=5 loops=1)
   ->  Sort  (cost=1.80..1.82 rows=7 width=168) (actual time=0.064..0.064 rows=5 loops=1)
         Sort Key: uid DESC
         Sort Method: top-N heapsort  Memory: 26kB
         ->  Foreign Scan on passwd  (cost=0.00..1.70 rows=7 width=168) (actual time=0.023..0.044 rows=27 loops=1)
               Foreign File: /etc/passwd
               Foreign File Size: 1271
 Planning time: 0.131 ms
 Execution time: 0.128 ms
(9 rows)
```

#### pg_prewarm