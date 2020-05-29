全文搜索一般很少在数据库内部实现，一般使用专门的全文搜索引擎，比如sphinx。PostgreSQL支持全文检索，对于规模不大的应用如果不想搭建专门的搜索引擎， PostgreSQL的全文检索也可以满足需求。
如果没有使用专门的搜索引擎，大部检索需要通过数据库like操作匹配，这种检索方式的主要缺点在于：

1. 不能很好地支持索引，通常需全表扫描检索数据，数据量大时检索性能很低。
2. 不提供检索结果排序，当结果集非常大时表现格外明显

postgresql使用tsvector、tsquery类型实现

### tsvector

tsvector全文检索数据类型代表一个被优化的可以基于搜索的文档，要将一串字符串转换成 tsvector全文检索数据类型，代码如下所示：

```
=# select 'How are you ? I''m smiling':: tsvector;
                tsvector                 
-----------------------------------------
 '?' 'How' 'I''m' 'are' 'smiling' 'you'
(1 row)

# select E'How are you ,I\'m smiling':: tsvector;
               tsvector               
--------------------------------------
 ',I''m' 'How' 'are' 'smiling' 'you'
(1 row)
```

对文本标准化处理

```
# select to_tsvector(E'How are you ,I\'m smiling');
                   to_tsvector                   
-------------------------------------------------
 'are':2 'how':1 'i':4 'm':5 'smiling':6 'you':3
(1 row)
```

### tsquery

tsquery表示一个文本查询，存储用于搜索的词，并且支持布尔操作“&”、“|”、“！”将字符串转换成 tsquery

```
# select 'Hello | cat' :: tsquery;
     tsquery     
-----------------
 'Hello' | 'cat'
(1 row)

# select 'Hello & cat' :: tsquery;
     tsquery     
-----------------
 'Hello' & 'cat'
(1 row)
```

上述只是转换成 tsquery类型，而并没有做标准化，使用 to tsquery函数可以执行标准化，如下所示：

```
# select to_tsquery('Hello | cat') ;
   to_tsquery    
-----------------
 'hello' | 'cat'
(1 row)

# select to_tsquery('Hello & cat') ;
   to_tsquery    
-----------------
 'hello' & 'cat'
(1 row)
```

### 查询示例

```
postgres=# select to_tsvector('How are you? my cat') @@ to_tsquery('cat & How');
 ?column? 
----------
 t
(1 row)

postgres=# select to_tsvector('How are you? my cat') @@ to_tsquery('cat | How');
 ?column? 
----------
 t
(1 row)

postgres=# select to_tsvector('How are you? my catty') @@ to_tsquery('cat & How');
 ?column? 
----------
 f
(1 row)

postgres=# select to_tsvector('How are you? my catty') @@ to_tsquery('cat | How');
 ?column? 
----------
 t
(1 row)
```

### like查询与全文检索

#### 1. 建表与插入测试数据

```
# create table full_text(id int4,name text);
CREATE TABLE
# insert into full_text(id,name) select n, n||'_fulltext' from generate_series(1,500000) n;
INSERT 0 500000
```

#### 2. 如果使用like

```
# select * from full_text where name like '1_fulltext';
 id |    name    
----+------------
  1 | 1_fulltext
(1 row)

# explain analyze select * from full_text where name like '1_fulltext';
                                                       QUERY PLAN                                                        
-------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..6794.17 rows=50 width=19) (actual time=0.259..34.528 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Parallel Seq Scan on full_text  (cost=0.00..5789.17 rows=21 width=19) (actual time=16.127..27.012 rows=0 loops=3)
         Filter: (name ~~ '1_fulltext'::text)
         Rows Removed by Filter: 166666
 Planning time: 0.100 ms
 Execution time: 34.562 ms
(8 rows)
```

可以看到走的是全表扫描

#### 3. 使用全文索引

创建索引

```
# create index fulltext_gin on full_text using gin(to_tsvector('english',name));
CREATE INDEX
```

查询结果

```
# explain analyze select * from full_text where to_tsvector('english',name)@@ to_tsquery('english','1_fulltext');
                                                      QUERY PLAN                                                       
-----------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on full_text  (cost=28.10..77.04 rows=12 width=19) (actual time=0.072..0.072 rows=1 loops=1)
   Recheck Cond: (to_tsvector('english'::regconfig, name) @@ '''1'' & ''fulltext'''::tsquery)
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on fulltext_gin  (cost=0.00..28.09 rows=12 width=0) (actual time=0.066..0.067 rows=1 loops=1)
         Index Cond: (to_tsvector('english'::regconfig, name) @@ '''1'' & ''fulltext'''::tsquery)
 Planning time: 0.119 ms
 Execution time: 0.099 ms
(7 rows)
```

如果不走索引，性能会很差

```
# explain analyze select * from full_text where to_tsvector(name)@@ to_tsquery('1_fulltext');
                                                         QUERY PLAN                                                          
-----------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..110957.03 rows=12 width=19) (actual time=0.274..1021.251 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Parallel Seq Scan on full_text  (cost=0.00..109955.83 rows=5 width=19) (actual time=675.477..1015.079 rows=0 loops=3)
         Filter: (to_tsvector(name) @@ to_tsquery('1_fulltext'::text))
         Rows Removed by Filter: 166666
 Planning time: 0.147 ms
 Execution time: 1021.277 ms
(8 rows)
```

