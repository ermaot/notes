## 简述

Oracle支持并行查询，充分利用多核CPU等硬件功能。postgresql在9.6以后支持并行查询。9版本的并行支持的范围有限，比如顺序扫描、夺标关联、聚合查询支持，10版本增加了并行索引扫描、并行index-only扫描、并行bitmap heap扫描等

## 并行配置参数

参数|说明
---|---
max_worker_processes|设置系统支持的最大后台进程数，默认值为8，如果有备库，备库上此参数必须大于或等于主库上的此参数配置值，此参数调整后需重启数据库生效。
max_parallel_ workers|设置系统支持的并行查询进程数，默认值为8，此参数受 max worker processes参数限制，设置此参数的值比 max worker processes值高将无效。当调整这个参数时建议同时调整 max parallel workers per gather参数值
max_parallel_workers_per_gather|设置允许启用的并行进程的进程数，默认值为2，设置成0表示禁用并行查询，此参数受 max worker processes参数和 max parallel workers参数限制，因此并行查询的实际进程数可能比预期的少，并行查询比非并行查询消耗更多的CPU、IO、内存资源，对生产系统有一定影响，使用时需考虑这方面的因素，这三个参数的配置值大小关系通常如下所示max worker processes max parallel workers > max parallel_workers_per_gather
parallel_setup_cost|设置优化器启动并行进程的成本，默认为1000。
parallel_tuple_cost|设置优化器通过并行进程处理一行数据的成本，默认为0.1。
min_parallel_table_scan_size|设置开启并行的条件之一，表占用空间小于此值将不会开启并行，并行顺序扫描场景下扫描的数据大小通常等于表大小，默认值为8MB。
min_parallel_index_scan_size|设置开启并行的条件之一，实际上并行索引扫描不会扫描索引所有数据块，只是扫描索引相关数据块，默认值为512kb。
force_parallel_mode|强制开启并行，一般作为测试目的，OLTP生产环境开启需慎重，一般不建议开启。

## 并行扫描
本节使用参数
```
max_worker_processes=16
max_parallel_workers_per_gather=4
max_parallel_workers=8
parallel_tuple_cost=0.1
parallel_setup_cost=1000.0
min_parallel_table_scan_size=8MB
min_parallel_index_scan_size=512kB
force_parallel_mode=off
```

#### 并行顺序扫描

建表并插入数据

```
# create table test_seq_scan(id int,name varchar(32),create_time timestamp without time zone default clock_timestamp());
# insert into test_seq_scan(id,name) select n,n||'_test' from generate_series(1,50000000) n;
```

执行下面的sql

```
# explain select * from test_seq_scan where name = '1_test';
                                    QUERY PLAN                                    
----------------------------------------------------------------------------------
 Gather  (cost=1000.00..628048.81 rows=1 width=25)
   Workers Planned: 2
   ->  Parallel Seq Scan on test_seq_scan  (cost=0.00..627048.71 rows=1 width=25)
         Filter: ((name)::text = '1_test'::text)
(4 rows)

Time: 11.864 ms

# explain analyze select * from test_seq_scan where name = '1_test';
                                                             QUERY PLAN                                                             
------------------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..628048.81 rows=1 width=25) (actual time=0.257..27559.931 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Parallel Seq Scan on test_seq_scan  (cost=0.00..627048.71 rows=1 width=25) (actual time=18288.451..27473.662 rows=0 loops=3)
         Filter: ((name)::text = '1_test'::text)
         Rows Removed by Filter: 16666666
 Planning time: 0.072 ms
 Execution time: 27559.958 ms
(8 rows)

Time: 27560.426 ms (00:27.560)
```

可以看到"Workers Planned: 2 "和" Workers Launched: 2"，使用了并行扫描。

如果关闭并行

```
# explain analyze select * from test_seq_scan where name = '1_test';
                                                   QUERY PLAN                                                    
-----------------------------------------------------------------------------------------------------------------
 Seq Scan on test_seq_scan  (cost=0.00..991587.30 rows=1 width=25) (actual time=0.014..30361.248 rows=1 loops=1)
   Filter: ((name)::text = '1_test'::text)
   Rows Removed by Filter: 49999999
 Planning time: 0.056 ms
 Execution time: 30361.271 ms
(5 rows)

Time: 30361.730 ms (00:30.362)
```

说明并行读取对执行性能有一定的提升。但随着max_parallel_workers_per_gather增大，实际使用的worker并不一定随着增大。在本机，该值设置为8以上，实际执行都只使用了6。

#### 并行索引扫描

在表上建索引

```
# create index test_id_idx on test_seq_scan using btree(id);
```

然后执行如下SQL

```
# explain select * from test_seq_scan where id < 10000000;
                                         QUERY PLAN                                          
---------------------------------------------------------------------------------------------
 Index Scan using test_id_idx on test_seq_scan  (cost=0.56..348550.83 rows=9735615 width=25)
   Index Cond: (id < 10000000)
(2 rows)

Time: 85.666 ms
```

设置max_parallel_workers_per_gather=0

```
# explain analyze select * from test_seq_scan where id < 10000000;
                                                                   QUERY PLAN                                                                   
------------------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using test_id_idx on test_seq_scan  (cost=0.56..348550.83 rows=9735615 width=25) (actual time=4.781..7118.360 rows=9999999 loops=1)
   Index Cond: (id < 10000000)
 Planning time: 0.108 ms
 Execution time: 7552.693 ms
(4 rows)

Time: 7553.274 ms (00:07.553)
```

设置max_parallel_workers_per_gather=8

```
# explain analyze select * from test_seq_scan where id < 10000000;
                                                                   QUERY PLAN                                                                   
------------------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using test_id_idx on test_seq_scan  (cost=0.56..348550.83 rows=9735615 width=25) (actual time=0.087..1795.658 rows=9999999 loops=1)
   Index Cond: (id < 10000000)
 Planning time: 0.115 ms
 Execution time: 2171.447 ms
(4 rows)


# explain analyze select count(name) from test_seq_scan where id < 10000000;
                                                                                QUERY PLAN                                                                           
      
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
------
 Finalize Aggregate  (cost=271485.29..271485.30 rows=1 width=8) (actual time=1884.590..1884.590 rows=1 loops=1)
   ->  Gather  (cost=271485.21..271485.28 rows=6 width=8) (actual time=1881.029..1885.658 rows=7 loops=1)
         Workers Planned: 6
         Workers Launched: 6
         ->  Partial Aggregate  (cost=271477.21..271477.22 rows=1 width=8) (actual time=1859.900..1859.900 rows=1 loops=7)
               ->  Parallel Index Scan using test_id_idx on test_seq_scan  (cost=0.56..267420.70 rows=1622602 width=13) (actual time=0.049..1758.041 rows=1428571 loo
ps=7)
                     Index Cond: (id < 10000000)
 Planning time: 2.369 ms
 Execution time: 1885.735 ms
```

可以看到select * 没有使用并行index扫描，而select count(name)使用了**原因在哪里**



#### 并行index-only扫描

```
# explain analyze select count(*) from test_seq_scan where id < 10000000;
                                                                                  QUERY PLAN                                                                         
          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------
 Finalize Aggregate  (cost=271485.29..271485.30 rows=1 width=8) (actual time=1715.924..1715.925 rows=1 loops=1)
   ->  Gather  (cost=271485.21..271485.28 rows=6 width=8) (actual time=1705.246..1717.562 rows=7 loops=1)
         Workers Planned: 6
         Workers Launched: 6
         ->  Partial Aggregate  (cost=271477.21..271477.22 rows=1 width=8) (actual time=1694.306..1694.306 rows=1 loops=7)
               ->  Parallel Index Only Scan using test_id_idx on test_seq_scan  (cost=0.56..267420.70 rows=1622602 width=0) (actual time=1.466..1594.807 rows=1428571
 loops=7)
                     Index Cond: (id < 10000000)
                     Heap Fetches: 1615524
 Planning time: 0.097 ms
 Execution time: 1717.628 ms
(10 rows)
```

如果设置为不并行扫描

```
# explain analyze select count(*) from test_seq_scan where id < 10000000;
                                                                        QUERY PLAN                                                                        
----------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=372889.86..372889.88 rows=1 width=8) (actual time=2791.811..2791.811 rows=1 loops=1)
   ->  Index Only Scan using test_id_idx on test_seq_scan  (cost=0.56..348550.83 rows=9735615 width=0) (actual time=0.096..2091.475 rows=9999999 loops=1)
         Index Cond: (id < 10000000)
         Heap Fetches: 9999999
 Planning time: 0.100 ms
 Execution time: 2791.863 ms
(6 rows)

Time: 2792.491 ms (00:02.792)
```

可见并行比不并行性能强不少

#### 并行bitmap heap扫描

不开并行扫描

```
# explain analyze select count(*) from test_seq_scan where id < 1000000 or id>49000000;
                                                                    QUERY PLAN                                                                     
---------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=436347.81..436347.82 rows=1 width=8) (actual time=1382.229..1382.229 rows=1 loops=1)
   ->  Bitmap Heap Scan on test_seq_scan  (cost=36265.72..431612.96 rows=1893940 width=0) (actual time=1010.087..1251.145 rows=1999999 loops=1)
         Recheck Cond: ((id < 1000000) OR (id > 49000000))
         Heap Blocks: exact=13724
         ->  BitmapOr  (cost=36265.72..36265.72 rows=1912216 width=0) (actual time=1008.168..1008.168 rows=0 loops=1)
               ->  Bitmap Index Scan on test_id_idx  (cost=0.00..17309.00 rows=937125 width=0) (actual time=779.609..779.609 rows=999999 loops=1)
                     Index Cond: (id < 1000000)
               ->  Bitmap Index Scan on test_id_idx  (cost=0.00..18009.75 rows=975091 width=0) (actual time=228.555..228.555 rows=1000000 loops=1)
                     Index Cond: (id > 49000000)
 Planning time: 16.121 ms
 Execution time: 1382.303 ms
(11 rows)

Time: 1399.153 ms (00:01.399)
```

开启并行扫描

```
# explain analyze select count(*) from test_seq_scan where id < 1000000 or id>49000000;
                                                                           QUERY PLAN                                                                           
----------------------------------------------------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=408499.49..408499.50 rows=1 width=8) (actual time=345.149..345.149 rows=1 loops=1)
   ->  Gather  (cost=408499.40..408499.47 rows=6 width=8) (actual time=339.603..346.423 rows=7 loops=1)
         Workers Planned: 6
         Workers Launched: 6
         ->  Partial Aggregate  (cost=408499.40..408499.41 rows=1 width=8) (actual time=245.002..245.002 rows=1 loops=7)
               ->  Parallel Bitmap Heap Scan on test_seq_scan  (cost=36265.72..407710.26 rows=315657 width=0) (actual time=23.770..202.725 rows=285714 loops=7)
                     Recheck Cond: ((id < 1000000) OR (id > 49000000))
                     Heap Blocks: exact=1691
                     ->  BitmapOr  (cost=36265.72..36265.72 rows=1912216 width=0) (actual time=110.395..110.395 rows=0 loops=1)
                           ->  Bitmap Index Scan on test_id_idx  (cost=0.00..17309.00 rows=937125 width=0) (actual time=54.533..54.534 rows=999999 loops=1)
                                 Index Cond: (id < 1000000)
                           ->  Bitmap Index Scan on test_id_idx  (cost=0.00..18009.75 rows=975091 width=0) (actual time=55.858..55.858 rows=1000000 loops=1)
                                 Index Cond: (id > 49000000)
 Planning time: 0.234 ms
 Execution time: 346.508 ms
(15 rows)

Time: 347.154 ms
```



## 并行聚合

## 多表关联

创建表，表索引

```
# create table test_seq_scan(id int,name varchar(32),create_time timestamp without time zone default clock_timestamp());
# insert into test_seq_scan(id,name) select n,n||'_test' from generate_series(1,5000000) n;
# create table test_seq_scan2(id int,name varchar(32));
# insert into test_seq_scan2(id,name) select n,n||'_test' from generate_series(1,8000000) n;
# create index test_id_idx on test_seq_scan using btree(id);
# create index test_id_idx2 on test_seq_scan using btree(id);
# analyze test_seq_scan2;
# analyze test_seq_scan;

```

#### nested loop多表关联

多表关联其实就是一个嵌套的多重循环

```
# explain analyze select * from test_seq_scan a ,test_seq_scan2 b where a.id = b.id and b.id <10000;
                                                                  QUERY PLAN                                                                  
----------------------------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=0.43..93154.58 rows=6622 width=40) (actual time=0.562..410.887 rows=9999 loops=1)
   Workers Planned: 4
   Workers Launched: 4
   ->  Nested Loop  (cost=0.43..93088.36 rows=1656 width=40) (actual time=226.463..386.048 rows=2000 loops=5)
         ->  Parallel Seq Scan on test_seq_scan2 b  (cost=0.00..74996.42 rows=2649 width=16) (actual time=226.447..379.894 rows=2000 loops=5)
               Filter: (id < 10000)
               Rows Removed by Filter: 1598000
         ->  Index Scan using test_id_idx2 on test_seq_scan a  (cost=0.43..6.82 rows=1 width=24) (actual time=0.003..0.003 rows=1 loops=9999)
               Index Cond: (id = b.id)
 Planning time: 0.281 ms
 Execution time: 411.315 ms
(11 rows)

Time: 412.095 ms
```

如果关闭并行 

```
# explain analyze select * from test_seq_scan a ,test_seq_scan2 b where a.id = b.id and b.id <1000;
                                                              QUERY PLAN                                                               
---------------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.43..158082.62 rows=610 width=40) (actual time=0.185..648.493 rows=999 loops=1)
   ->  Seq Scan on test_seq_scan2 b  (cost=0.00..150009.66 rows=976 width=16) (actual time=0.164..645.488 rows=999 loops=1)
         Filter: (id < 1000)
         Rows Removed by Filter: 7999001
   ->  Index Scan using test_id_idx2 on test_seq_scan a  (cost=0.43..8.26 rows=1 width=24) (actual time=0.002..0.002 rows=1 loops=999)
         Index Cond: (id = b.id)
 Planning time: 0.259 ms
 Execution time: 648.590 ms
(8 rows)
```

可以看到性能下降不少

#### merge join多表关联

merge join首先将两个表排序，然后匹配关联字段

#### hash join多表关联

```
# explain analyze select b.name from test_seq_scan a ,test_seq_scan2 b where a.id = b.id and b.id <2000000;
                                                                QUERY PLAN                                                                
------------------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=184320.90..356133.14 rows=1233385 width=12) (actual time=1186.662..4179.682 rows=1999999 loops=1)
   Hash Cond: (a.id = b.id)
   ->  Seq Scan on test_seq_scan a  (cost=0.00..85779.59 rows=4999759 width=4) (actual time=0.009..632.402 rows=5000000 loops=1)
   ->  Hash  (cost=150009.66..150009.66 rows=1973859 width=16) (actual time=1185.925..1185.925 rows=1999999 loops=1)
         Buckets: 131072 (originally 131072)  Batches: 64 (originally 32)  Memory Usage: 4090kB
         ->  Seq Scan on test_seq_scan2 b  (cost=0.00..150009.66 rows=1973859 width=16) (actual time=0.145..799.253 rows=1999999 loops=1)
               Filter: (id < 2000000)
               Rows Removed by Filter: 6000001
 Planning time: 0.273 ms
 Execution time: 4258.366 ms
(10 rows)

```

如果删除索引

```
# explain analyze select b.name from test_seq_scan a ,test_seq_scan2 b where a.id = b.id and b.id <100;
                                                                   QUERY PLAN                                                                    
-------------------------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=76086.42..180620.10 rows=500 width=12) (actual time=374.042..1231.771 rows=99 loops=1)
   Hash Cond: (a.id = b.id)
   ->  Seq Scan on test_seq_scan a  (cost=0.00..85779.59 rows=4999759 width=4) (actual time=0.017..461.827 rows=5000000 loops=1)
   ->  Hash  (cost=76076.42..76076.42 rows=800 width=16) (actual time=374.011..374.011 rows=99 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 13kB
         ->  Gather  (cost=1000.00..76076.42 rows=800 width=16) (actual time=0.401..374.127 rows=99 loops=1)
               Workers Planned: 4
               Workers Launched: 4
               ->  Parallel Seq Scan on test_seq_scan2 b  (cost=0.00..74996.42 rows=200 width=16) (actual time=273.587..347.432 rows=20 loops=5)
                     Filter: (id < 100)
                     Rows Removed by Filter: 1599980
 Planning time: 0.163 ms
 Execution time: 1231.964 ms
(13 rows)
```

