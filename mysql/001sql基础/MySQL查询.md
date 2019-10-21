## sql语句关键词


```
select * from test;                                 //简单查询
select URL , IP from test;                          //查询具体字段
select distinct IP from test;                       //去除重复记录
select * from test where ip is null;                //选择特定记录
select * from test where ip is not null;            //同上
select * from test order by a [desc,asc];           //对记录排序
select * from test limit 1 [,20] ;                  //限定记录数与分页
select a,ifnull(b,'unknow')  as id from timestamp_test order by a limit 0,2;  //替换符合条件的记录为特定值
```

## 格式化

```
> select concat (monthname(current_time),' ',dayname(current_timestamp),' ' ,year(current_date)) from dual;		//格式化timestamp；monthname()   dayname()  year()都返回英文
> select concat (monthname(current_time),' ',dayname(current_timestamp),' ' ,year(current_date)) from dual;
+-----------------------------------------------------------------------------------------+
| concat (monthname(current_time),' ',dayname(current_timestamp),' ' ,year(current_date)) |
+-----------------------------------------------------------------------------------------+
| July Tuesday 2019                                                                       |
+-----------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

//注意mariadb5.56中返回结果不一样
> select concat (monthname(current_time),' ',dayname(current_timestamp),' ' ,year(current_date)) from dual;
+-----------------------------------------------------------------------------------------+
| concat (monthname(current_time),' ',dayname(current_timestamp),' ' ,year(current_date)) |
+-----------------------------------------------------------------------------------------+
| NULL                                                                                    |
+-----------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

原因在于，maria5.56中的current_time只保存当前的时间（几时几分），所以无法获取到monthname；而mysql8中查询select current_time只显示时间部分，而实际上是保存了整个timestamp的

select date_format(current_time, '%M %e ,%Y')  from dual;							//月份M使用英文，m使用数字；Y全部年份，y年份后两位；S和s对秒一样
```
## 关于null

```
select null is null;				//返回1  
+--------------+
| null is null |
+--------------+
|            1 |
+--------------+
1 row in set (0.00 sec)


select null <=> null;				//返回1 <=>符号，MySQL方言，表示相等
+---------------+
| null <=> null |
+---------------+
|             1 |
+---------------+
1 row in set (0.00 sec)


select 'a' <=> 'b';
+-------------+
| 'a' <=> 'b' |
+-------------+
|           0 |
+-------------+
1 row in set (0.00 sec)

select null = null;					//返回null 
+-------------+
| null = null |
+-------------+
|        NULL |
+-------------+
1 row in set (0.00 sec)


select a , IF(a is null ,'unknow' ,a) as id from a;			//if(expr1 is not null , expr1 ,expr2)   

+------+--------+
| a    | id     |
+------+--------+
|    1 | 1      |
| NULL | unknow |
+------+--------+
2 rows in set (0.00 sec)


select ifnull(a ,"unknow") from a; 						//ifnull(expr1,expr2)     
+---------------------+
| ifnull(a ,"unknow") |
+---------------------+
| 1                   |
| unknow              |
+---------------------+
2 rows in set (0.00 sec)    

```
## where表达式中使用别名

```
> select a as test from a where test<0;
ERROR 1054 (42S22): Unknown column 'test' in 'where clause'
```

此方法并不合法，也就是说这种方式并不存在。原因在于下图：


```
⑧select ⑨distinct  < select list >
①from < left_table >
③< join_type > join < right_table >
②on < join_condition >
④where < where_condition >
⑤group by < group_list >
⑥with {cube | rollup}
⑦having <having_condition>
⑩order by < order_by_list >
⑪limit < limit_number >
```

select处于步骤⑧，而where处于步骤④，也就是说where执行时，select中定义的别名尚未出现，所以会报错


## 本文来自《mysql cookbook》