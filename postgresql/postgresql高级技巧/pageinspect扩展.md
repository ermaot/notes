https://blog.csdn.net/liuhuayang/article/details/105804004
https://blog.csdn.net/yanzongshuai/article/details/107551637


![img](pic/Untitled/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9pYUxWdHVKUGpqdzdDb256bzlUV1dJTGdvSWRaTWhuWW5Zc0dBN0VQV2s2akRBWlpua3JJcnp1RWNFMWlia0twd3gzWmNpYkFOam9lMDZ3dlV1VXRUNGxYUS82NDA)



## pageinspect扩展

PG自身自带的pageinspect 工具，可以让你对页面级别的层次来进行一个 “透心凉” 的查看和分析。
#### 几个函数
```

```

## 创建扩展

```
postgres=# select * from pg_extension;
 extname | extowner | extnamespace | extrelocatable | extversion | extconfig | extcondition 
---------+----------+--------------+----------------+------------+-----------+--------------
 plpgsql |       10 |           11 | f              | 1.0        |           | 
(1 row)

postgres=# create extension pageinspect;
CREATE EXTENSION
postgres=# select * from pg_extension;
   extname   | extowner | extnamespace | extrelocatable | extversion | extconfig | extcondition 
-------------+----------+--------------+----------------+------------+-----------+--------------
 plpgsql     |       10 |           11 | f              | 1.0        |           | 
 pageinspect |       10 |         2200 | t              | 1.0        |           | 
(2 rows)

```

创建测试用的表
```
postgres=# create table test(a int,b int);
CREATE TABLE

postgres=# insert into test(a,b) values(1,2);
INSERT 0 1
postgres=# select ctid,a from test;
 ctid  | a 
-------+---
 (0,1) | 1
(1 row)

postgres=# insert into test(a,b) values(1,2);
INSERT 0 1
postgres=# insert into test(a,b) values(1,2);
INSERT 0 1
postgres=# insert into test(a,b) values(1,2);
INSERT 0 1
postgres=# insert into test(a,b) values(1,2);
INSERT 0 1
postgres=# select ctid,a from test;
 ctid  | a 
-------+---
 (0,1) | 1
 (0,2) | 1
 (0,3) | 1
 (0,4) | 1
 (0,5) | 1
(5 rows)

postgres=# delete from test;
DELETE 5
postgres=# select ctid,a from test;
 ctid | a 
------+---
(0 rows)

postgres=# insert into test(a,b) values(1,2);
INSERT 0 1

```


## 使用扩展

```
postgres=# select * from heap_page_items(get_raw_page('test',0)) order by lp_off desc;
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     32 |   1893 |   1898 |        0 | (0,1)  |           2 |       1280 |     24 |        |      
  2 |   8128 |        1 |     32 |   1894 |   1898 |        0 | (0,2)  |           2 |       1280 |     24 |        |      
  3 |   8096 |        1 |     32 |   1895 |   1898 |        0 | (0,3)  |           2 |       1280 |     24 |        |      
  4 |   8064 |        1 |     32 |   1896 |   1898 |        0 | (0,4)  |           2 |       1280 |     24 |        |      
  5 |   8032 |        1 |     32 |   1897 |   1898 |        0 | (0,5)  |           2 |       1280 |     24 |        |      
  6 |   8000 |        1 |     32 |   1899 |      0 |        0 | (0,6)  |           2 |       2304 |     24 |        |      
(6 rows)

postgres=# update test set a=3 where a=1;
UPDATE 1
postgres=# select * from heap_page_items(get_raw_page('test',0)) order by lp_off desc;
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     32 |   1893 |   1898 |        0 | (0,1)  |           2 |       1280 |     24 |        |      
  2 |   8128 |        1 |     32 |   1894 |   1898 |        0 | (0,2)  |           2 |       1280 |     24 |        |      
  3 |   8096 |        1 |     32 |   1895 |   1898 |        0 | (0,3)  |           2 |       1280 |     24 |        |      
  4 |   8064 |        1 |     32 |   1896 |   1898 |        0 | (0,4)  |           2 |       1280 |     24 |        |      
  5 |   8032 |        1 |     32 |   1897 |   1898 |        0 | (0,5)  |           2 |       1280 |     24 |        |      
  6 |   8000 |        1 |     32 |   1899 |   1901 |        0 | (0,7)  |       16386 |        256 |     24 |        |      
  7 |   7968 |        1 |     32 |   1901 |      0 |        0 | (0,7)  |       32770 |      10240 |     24 |        |      
(7 rows)

```

1. 从lp_off看到数据递减，所以页面是从页尾开始的
2. 删除的5条数据，其实在数据库中都存在。update之前和之后的数据，也是存在的。

```
postgres=# SELECT * from page_header(get_raw_page('test', 0));
    lsn    | tli | flags | lower | upper | special | pagesize | version | prune_xid 
-----------+-----+-------+-------+-------+---------+----------+---------+-----------
 0/1837458 |   1 |     0 |    52 |  7968 |    8192 |     8192 |       4 |      1898

```

lower = 52 , 通过这里可以获知当前PG的表TEST 中曾经有过多少turple(在这一刻)，PG的每页有28bytes 的页头，同时每个指针是4bytes ，(52 - 28)/4 = 6 ,证明当前的指针有6个

```
postgres=# vacuum;
VACUUM
postgres=# SELECT * from page_header(get_raw_page('test', 0));
    lsn    | tli | flags | lower | upper | special | pagesize | version | prune_xid 
-----------+-----+-------+-------+-------+---------+----------+---------+-----------
 0/183EAC8 |   1 |     1 |    52 |  8160 |    8192 |     8192 |       4 |         0
(1 row)

postgres=# select * from heap_page_items(get_raw_page('test',0)) order by lp_off desc;
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  7 |   8160 |        1 |     32 |   1901 |      0 |        0 | (0,7)  |       32770 |      10496 |     24 |        |      
  6 |      7 |        2 |      0 |        |        |          |        |             |            |        |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  1 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)
```

vacuum之后，upper变大，之前使用的页面被回收。再插入一条。
```
postgres=# insert into test(a,b) values(1,2);
INSERT 0 1
postgres=# select * from heap_page_items(get_raw_page('test',0)) order by lp_off desc;
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  7 |   8160 |        1 |     32 |   1901 |      0 |        0 | (0,7)  |       32770 |      10496 |     24 |        |      
  1 |   8128 |        1 |     32 |   1902 |      0 |        0 | (0,1)  |           2 |       2048 |     24 |        |      
  6 |      7 |        2 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)


```

可以看到回收了之前的空间



##
```
SELECT  get_raw_page::text FROM    get_raw_page('test', 0);
```