## 1. mysql的多字段更新

#### 创建表并插入数据

```
> create table insert_test(a int,b int);
> insert into insert_test values(1,2),(2,3);
> select * from insert_test;
+------+------+
| a    | b    |
+------+------+
|    1 |    2 |
|    2 |   3 |
+------+------+
```

#### 更新多字段

```
> update insert_test set a=a+1,b=a+10 where a=2;
```

按照一般的想法，a=2这一行，此时应该得到(3,12)

#### 查询

```
> select * from insert_test;
+------+------+
| a    | b    |
+------+------+
|    1 |    2 |
|    3 |   13 |
+------+------+
```

实际结果却是(3,13)

#### 为甚么？

这里涉及到两个概念：当前读，快照读。

普通的select（不包括select in share mode,select for update等）都是使用快照读，而update，insert，delete select in share mode,select for update都要做当前读。

因为update a=a+1要做当前读，a=2+1后等于3，再做当前读，执行b=a+10得到13

#### 解决方法

```
> select a into @a from insert_test where a=2
> update insert_test set a=@a+1,b=@a+10
```

## 2. postgresql的多字段更新表现

``` 
postgres=# create table insert_test(a int,b int);
CREATE TABLE
postgres=# insert into insert_test values(1,2),(2,3);
INSERT 0 2
postgres=# select * from insert_test ;
 a | b 
---+---
 1 | 2
 2 | 3
(2 行记录)

postgres=# update insert_test set a=a+1,b=a+10 where a=2;
UPDATE 1
postgres=# select * from insert_test ;
 a | b  
---+----
 1 |  2
 3 | 12
(2 行记录)
```

postgresql表现和mysql不一样，比较符合我们直觉上的结果

## 3. 为甚么

