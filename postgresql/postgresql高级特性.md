## 1.with查询

with查询是postgresql的高级特性，oracle 很早就支持，mysql的新版本也是支持的。说明这种with查询的必要性，能极大提升sql的表现力

```
with t as (select * from test) select * from t;
+------+
| a    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

```

多表中间结果

```
with t1 as ( select a from test), t2 as ( select a from b) select * from t1 , t2 where  t1.a=t2.a;
+------+------+
| a    | a    |
+------+------+
|    1 |    1 |
+------+------+
1 row in set (0.01 sec)
```

#### 递归使用with

with的一个重要特定就是可以递归使用，引用自己的结果。

```
# with recursive t (x) as ( select 1 union select x + 1 from t where x < 5) select sum(x) from t;
 sum 
-----
  15
(1 row)
```

## 2.returning 返回更新后的数据

创建表test

```
# \d test
        Table "public.test"
 Column |     Type     | Modifiers 
--------+--------------+-----------
 a      | integer      | 
 b      | character(8) | 
Indexes:
    "test_i" btree (a)
```

插入数据

```
insert into test values(1,'test1'),(2,'test2');
```

使用returning

```
# update test set a = 5 where a=1 returning * ;
 a |    b     
---+----------
 5 | test1   
(1 row)
```

## 3.upsert

```
# insert into test_insert(a,b) values (1,'test1') on conflict(a) do nothing;
# insert into test_insert(a,b) values (1,'test1') on conflict(a) do update set a=EXCLUDED.a;
```

**注意，这个EXCLUDED表示的是values里的数值，且必须大写**

## 4.数据抽样

普通的数据抽样

```
# select * from test order by random() limit 1;
```

tablesample数据抽样。9.5以后才支持

```
# select * from test tablesample system(0.00001);
# select * from test tablesample bernoulli(0.0001);
```

system抽样方式是随机抽取表的数据块上的数据，bernoulli抽样是随机抽取表的数据行。这会导致system抽样，每次返回的行数可能不一样，因为每个块上的数据行数不一定一样。

## 5.聚合函数

##### string_agg函数

```
# create table city( country varchar(64),city varchar(64));
# insert into city values('中国','台北'),('中国','上海'),('中国','香港'),('日本','东京'),('日本','大阪');
# select * from city;
 country | city 
---------+------
 中国    | 台北
 中国    | 上海
 中国    | 香港
 日本    | 东京
 日本    | 大阪
(5 rows)
# select country,string_agg(city,',')  from city group by country;
 country |   string_agg   
---------+----------------
 中国    | 台北,上海,香港
 日本    | 东京,大阪
(2 rows)
```

##### array_agg

与string_agg类似，但返回的是数组

```
# select country,array_agg(city)  from city group by country;
 country |    array_agg     
---------+------------------
 中国    | {台北,上海,香港}
 日本    | {东京,大阪}
(2 rows)
```

## 6. 窗口函数

窗口函数包括内置的，比如(),rank(),lag()，以及聚合函数、自定义函数后接over属性

创建基本表并插入数据

```
# CREATE TABLE score (id serial primary key, subject character varying(32), stu_name character varying(32), score numeric(3,0));
# INSERT INTO score(subject, stu_name, score) VALUEs('Chinese','francs',70),('Chinese','matiler',70),('Chinese','tutu', 80),('English','matiler',75),('English','francs',90),('English','tutu', 60),('Math','francs',80),('Math','matiler', 99),('Math','tutu', 65);
```

#### avg() over

如果想查每个学生的成绩并显示课程的平均分，传统方法就是通过表连接，即先查询平均分，然后再与原表连接。如下：

```
# select id,a.subject,stu_name,score,avg from score as a join (select subject,avg(score) from score group by subject) as b on a.subject=b.subject ;
 id | subject | stu_name | score |         avg         
----+---------+----------+-------+---------------------
  1 | Chinese | francs   |    70 | 73.3333333333333333
  2 | Chinese | matiler  |    70 | 73.3333333333333333
  3 | Chinese | tutu     |    80 | 73.3333333333333333
  4 | English | matiler  |    75 | 75.0000000000000000
  5 | English | francs   |    90 | 75.0000000000000000
  6 | English | tutu     |    60 | 75.0000000000000000
  7 | Math    | francs   |    80 | 81.3333333333333333
  8 | Math    | matiler  |    99 | 81.3333333333333333
  9 | Math    | tutu     |    65 | 81.3333333333333333
(9 rows)
```

如果使用窗口函数，可以方便很多：

```
# select id,subject,stu_name,score,avg(score) over(partition by subject) from score;
 id | subject | stu_name | score |         avg         
----+---------+----------+-------+---------------------
  1 | Chinese | francs   |    70 | 73.3333333333333333
  2 | Chinese | matiler  |    70 | 73.3333333333333333
  3 | Chinese | tutu     |    80 | 73.3333333333333333
  4 | English | matiler  |    75 | 75.0000000000000000
  5 | English | francs   |    90 | 75.0000000000000000
  6 | English | tutu     |    60 | 75.0000000000000000
  7 | Math    | francs   |    80 | 81.3333333333333333
  8 | Math    | matiler  |    99 | 81.3333333333333333
  9 | Math    | tutu     |    65 | 81.3333333333333333
(9 rows)
```

查看查询计划

```
# explain analyze select id,a.subject,stu_name,score,avg from score as a join (select subject,avg(score) from score group by subject) as b on a.subject=b.subject ;
                                                            QUERY PLAN                                                            
----------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=797.06..1643.18 rows=36864 width=53) (actual time=14.061..27.067 rows=36864 loops=1)
   Hash Cond: ((a.subject)::text = (b.subject)::text)
   ->  Seq Scan on score a  (cost=0.00..612.64 rows=36864 width=21) (actual time=0.013..3.193 rows=36864 loops=1)
   ->  Hash  (cost=797.03..797.03 rows=3 width=38) (actual time=14.037..14.037 rows=3 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         ->  Subquery Scan on b  (cost=796.96..797.03 rows=3 width=38) (actual time=14.024..14.030 rows=3 loops=1)
               ->  HashAggregate  (cost=796.96..797.00 rows=3 width=38) (actual time=14.023..14.027 rows=3 loops=1)
                     Group Key: score.subject
                     ->  Seq Scan on score  (cost=0.00..612.64 rows=36864 width=11) (actual time=0.003..3.231 rows=36864 loops=1)
 Planning time: 0.219 ms
 Execution time: 28.528 ms
(11 rows)
```

与窗口函数的查询计划

```
# explain analyze select id,subject,stu_name,score,avg(score) over(partition by subject) from score;
                                                      QUERY PLAN                                                      
----------------------------------------------------------------------------------------------------------------------
 WindowAgg  (cost=3408.76..4053.88 rows=36864 width=53) (actual time=22.146..40.023 rows=36864 loops=1)
   ->  Sort  (cost=3408.76..3500.92 rows=36864 width=21) (actual time=17.463..20.134 rows=36864 loops=1)
         Sort Key: subject
         Sort Method: quicksort  Memory: 3826kB
         ->  Seq Scan on score  (cost=0.00..612.64 rows=36864 width=21) (actual time=0.015..5.280 rows=36864 loops=1)
 Planning time: 0.130 ms
 Execution time: 42.077 ms
(7 rows)
```

虽然窗口函数方便阅读，但执行性能比表连接还是要差一些。而且随着数据量增大，这种趋势越发明显。

数据量|类型|执行时间
---|---|---
9|表连接|0.108ms
9|窗口函数|0.110ms
36864|表连接|28ms
36864|窗口函数|45ms
294912|表连接|180ms
294912|窗口函数|510ms

#### row_number

```
# select row_number() over(partition by subject order by score) ,* from score;
 row_number | id | subject | stu_name | score 
------------+----+---------+----------+-------
          1 |  2 | Chinese | matiler  |    70
          2 |  1 | Chinese | francs   |    70
          3 |  3 | Chinese | tutu     |    80
          1 |  6 | English | tutu     |    60
          2 |  4 | English | matiler  |    75
          3 |  5 | English | francs   |    90
          1 |  9 | Math    | tutu     |    65
          2 |  7 | Math    | francs   |    80
          3 |  8 | Math    | matiler  |    99
(9 rows)
```

#### rank()与dense_rank()

两个函数很相似，都可以在组内排名，但rank在字段相同时会产生gap，而dense_rank不会

```
# select rank() over(partition by subject order by score) ,* from score;
 rank | id | subject | stu_name | score 
------+----+---------+----------+-------
    1 |  2 | Chinese | matiler  |    70
    1 |  1 | Chinese | francs   |    70
    3 |  3 | Chinese | tutu     |    80
    1 |  6 | English | tutu     |    60
    2 |  4 | English | matiler  |    75
    3 |  5 | English | francs   |    90
    1 |  9 | Math    | tutu     |    65
    2 |  7 | Math    | francs   |    80
    3 |  8 | Math    | matiler  |    99
(9 rows)

# select dense_rank() over(partition by subject order by score) ,* from score;
 dense_rank | id | subject | stu_name | score 
------------+----+---------+----------+-------
          1 |  2 | Chinese | matiler  |    70
          1 |  1 | Chinese | francs   |    70
          2 |  3 | Chinese | tutu     |    80
          1 |  6 | English | tutu     |    60
          2 |  4 | English | matiler  |    75
          3 |  5 | English | francs   |    90
          1 |  9 | Math    | tutu     |    65
          2 |  7 | Math    | francs   |    80
          3 |  8 | Math    | matiler  |    99
(9 rows)
```

#### lag()

```
# select lag(id,1) over(partition by stu_name),* from score;
 lag | id | subject | stu_name | score 
-----+----+---------+----------+-------
     |  1 | Chinese | francs   |    70
   1 |  5 | English | francs   |    90
   5 |  7 | Math    | francs   |    80
     |  4 | English | matiler  |    75
   4 |  8 | Math    | matiler  |    99
   8 |  2 | Chinese | matiler  |    70
     |  6 | English | tutu     |    60
   6 |  9 | Math    | tutu     |    65
   9 |  3 | Chinese | tutu     |    80
(9 rows)
```

#### first_value() 和last_value()

first_value获取组内的第一个值，last_value组内最后一个值

```
# select first_value(score) over(partition by subject ),* from score;
 first_value | id | subject | stu_name | score 
-------------+----+---------+----------+-------
          70 |  1 | Chinese | francs   |    70
          70 |  2 | Chinese | matiler  |    70
          70 |  3 | Chinese | tutu     |    80
          75 |  4 | English | matiler  |    75
          75 |  5 | English | francs   |    90
          75 |  6 | English | tutu     |    60
          80 |  7 | Math    | francs   |    80
          80 |  8 | Math    | matiler  |    99
          80 |  9 | Math    | tutu     |    65
(9 rows)

# select last_value(id) over(partition by subject  ),* from score;
 last_value | id | subject | stu_name | score 
------------+----+---------+----------+-------
          3 |  1 | Chinese | francs   |    70
          3 |  2 | Chinese | matiler  |    70
          3 |  3 | Chinese | tutu     |    80
          6 |  4 | English | matiler  |    75
          6 |  5 | English | francs   |    90
          6 |  6 | English | tutu     |    60
          9 |  7 | Math    | francs   |    80
          9 |  8 | Math    | matiler  |    99
          9 |  9 | Math    | tutu     |    65
(9 rows)

```

但在这里有一个bug，就是就是如果在窗口函数里面加了order by，会出现诡异现象

```
# select last_value(id) over(partition by subject order by score ),* from score;
 last_value | id | subject | stu_name | score 
------------+----+---------+----------+-------
          1 |  2 | Chinese | matiler  |    70
          1 |  1 | Chinese | francs   |    70
          3 |  3 | Chinese | tutu     |    80
          6 |  6 | English | tutu     |    60
          4 |  4 | English | matiler  |    75
          5 |  5 | English | francs   |    90
          9 |  9 | Math    | tutu     |    65
          7 |  7 | Math    | francs   |    80
          8 |  8 | Math    | matiler  |    99
(9 rows)
```

这个的结果与下面语句相同

```
# select last_value(id) over(partition by subject ,score ),* from score;
 last_value | id | subject | stu_name | score 
------------+----+---------+----------+-------
          1 |  2 | Chinese | matiler  |    70
          1 |  1 | Chinese | francs   |    70
          3 |  3 | Chinese | tutu     |    80
          6 |  6 | English | tutu     |    60
          4 |  4 | English | matiler  |    75
          5 |  5 | English | francs   |    90
          9 |  9 | Math    | tutu     |    65
          7 |  7 | Math    | francs   |    80
          8 |  8 | Math    | matiler  |    99
(9 rows)
```

原因应该是分组的时候将order 里面的条件当成了partition的条件了，也就是将(subject,score)当分组条件

#### nth_value()

取结果集分组的指定行数据

```
# select nth_value(id,2) over(partition by subject ),* from score;
 nth_value | id | subject | stu_name | score 
-----------+----+---------+----------+-------
         2 |  1 | Chinese | francs   |    70
         2 |  2 | Chinese | matiler  |    70
         2 |  3 | Chinese | tutu     |    80
         5 |  4 | English | matiler  |    75
         5 |  5 | English | francs   |    90
         5 |  6 | English | tutu     |    60
         8 |  7 | Math    | francs   |    80
         8 |  8 | Math    | matiler  |    99
         8 |  9 | Math    | tutu     |    65
(9 rows)

```

但下面的结果无法理解

```
# select nth_value(id,2) over(partition by subject order by id desc),* from score;
 nth_value | id | subject | stu_name | score 
-----------+----+---------+----------+-------
           |  3 | Chinese | tutu     |    80
         2 |  2 | Chinese | matiler  |    70
         2 |  1 | Chinese | francs   |    70
           |  6 | English | tutu     |    60
         5 |  5 | English | francs   |    90
         5 |  4 | English | matiler  |    75
           |  9 | Math    | tutu     |    65
         8 |  8 | Math    | matiler  |    99
         8 |  7 | Math    | francs   |    80
```

使用subject字段来排序，却是正常的

```
# select nth_value(id,2) over(partition by subject order by  subject desc  ),* from score;
 nth_value | id | subject | stu_name | score 
-----------+----+---------+----------+-------
         7 |  9 | Math    | tutu     |    65
         7 |  7 | Math    | francs   |    80
         7 |  8 | Math    | matiler  |    99
         4 |  5 | English | francs   |    90
         4 |  4 | English | matiler  |    75
         4 |  6 | English | tutu     |    60
         3 |  1 | Chinese | francs   |    70
         3 |  3 | Chinese | tutu     |    80
         3 |  2 | Chinese | matiler  |    70
(9 rows)
```

#### 窗口函数别名

```
# select sum(score) over (r) ,avg(score) over (r) ,* from score window r as (partition by subject);
 sum |         avg         | id | subject | stu_name | score 
-----+---------------------+----+---------+----------+-------
 220 | 73.3333333333333333 |  1 | Chinese | francs   |    70
 220 | 73.3333333333333333 |  2 | Chinese | matiler  |    70
 220 | 73.3333333333333333 |  3 | Chinese | tutu     |    80
 225 | 75.0000000000000000 |  4 | English | matiler  |    75
 225 | 75.0000000000000000 |  5 | English | francs   |    90
 225 | 75.0000000000000000 |  6 | English | tutu     |    60
 244 | 81.3333333333333333 |  7 | Math    | francs   |    80
 244 | 81.3333333333333333 |  8 | Math    | matiler  |    99
 244 | 81.3333333333333333 |  9 | Math    | tutu     |    65
(9 rows)
```

