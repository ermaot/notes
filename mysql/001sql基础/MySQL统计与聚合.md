## 使用count

```
select count(if(id in (1,2) , id , null )) from test ;	
+-----------------------------------+
| count(if(a in (1,2) , a , null )) |
+-----------------------------------+
|                                 2 |
+-----------------------------------+
1 row in set (0.00 sec)

> select count(if(a in (1,2) , a , 3 )) from int_test ;
+--------------------------------+
| count(if(a in (1,2) , a , 3 )) |
+--------------------------------+
|                              4 |
+--------------------------------+
1 row in set (0.00 sec)
//此处有些奇怪，test有4条记录，但执行结果只有2条
原因应该是
- count不计算expr产生的中间null值
- count(*)不考虑其内容
- count(具体列)也不统计空值
> select count(a) from int_test;
+----------+
| count(a) |
+----------+
|        3 |
+----------+
1 row in set (0.00 sec)

```
## 使用视图

```
create view int_test_view as select if(a in (1,2) , a , null ) from int_test ;

> select count(*) from int_test_view;
+----------+
| count(*) |
+----------+
|        4 |
+----------+
1 row in set (0.00 sec)

可以看到，类似的语句，一个是直接count，一个是视图之后count，二者结果不同，一个是2，一个是4
```
## 使用 min & max  & sum & avg &distinct  & group by  & having

```
select * from test where id = ( select max(id) from test ) ;
select count(*) , id  from test group by id having count(*) = 3 ;
```
## 查找最小或最大的摘要数值

```
select left(name , 1) as letter , count(*) as count from states group by letter 
having count = ( select count(*) from states group by left(name ,1) order by count(*) desc limit 1);	
```


## 本文来自《MySQL cookbook》