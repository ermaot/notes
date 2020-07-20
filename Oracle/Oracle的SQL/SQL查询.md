#### 查找空值

不能用"="，要用"is"

```
select * from emp where comm is NULL;
```

同时，NULL不支持加减乘除、大小、相等的比较，一律结果是空

```
select 1+NULL,1-NULL,1*NULL,NULL/1,NULL>1,NULL<1;
+--------+--------+--------+--------+--------+--------+
| 1+NULL | 1-NULL | 1*NULL | NULL/1 | NULL>1 | NULL<1 |
+--------+--------+--------+--------+--------+--------+
|   NULL |   NULL |   NULL |   NULL |   NULL |   NULL |
+--------+--------+--------+--------+--------+--------+


```

函数执行NULL相关运算时候：

```
select replace('abcde','a',NULL) as str from dual;
str
----
bcde
```
replace函数MySQL和postgresql都有，但执行这个语句得到的结果都是NULL
```
select greatest(1,NULL) from dual;
greatest(1,NULL)
----------------
NULL
```
greatest函数postgresql有，MySQL无；postgresql执行结果忽视NULL
```
postgres=# select greatest(1,2,3,4,5,NULL);
 greatest 
----------
        5
(1 row)
```
#### 空值转为实际值

coalesce将第一个不为NULL的值取出

NVL(eExpression1, eExpression2)

```
NVL(eExpression1, eExpression2)
```

取出结果中不为NULL的值，如果两个都是NULL则为NULL

#### 拼接列

使用 || 拼接（postgresql出了concat函数，也可以用||，但MySQL不可）

#### 条件逻辑

```
select uaername,pass ,case when uid>100  then 'OK' else 'Bad' END as uid from passwd ;
```

#### 限制返回的函数

```
select rownum as sn,emp.* from emp where rownum<=2
```

返回前两行

```
select * from (select rownum as sn,emp.* from emp where rownum<=2) where sn=2;
```

返回第二行

```
select * from (select rownum as sn,emp.* from emp order by dbms_random.value()) where rownum<=2;
```

随机返回2行。即先排序再随机取数

而下面的是先取数再随机排序，也就是返回的结果虽然排序不同，但结果集每次执行都是一样的

```
select * from emp where rownum<=2 order by dbms.random.value;
```

#### 模糊查询

下划线"_"匹配一个字符,"%"匹配多个字符

#### replace和translate的区别

```
select translate('abcdefg','abc','1234');
 translate 
-----------
 123defg
 
 select replace('abcdefg','abc','1234');
 replace  
----------
 1234defg
```

oracle和postgresql是一样的结果，MySQL没有translate函数

#### 查询排序的NULL处理

可以使用NULL first或者NULLlast对NULL处理

```
select * from test order by id  NULL first;
```

#### inner join ，left join，right join，full join