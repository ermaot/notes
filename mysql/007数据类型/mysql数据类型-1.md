## mysql数据类型-1
参考来源：
- MYSQL技术内幕：姜承尧
### UNSIGNED与ZEROFILL
---
- ### UNSIGNED（无符号）
无符号导致溢出<br>
SET ==SQL_MODE== = "NO_UNSIGNED_SUBSTRACTION"
- ### ZEROFILL（零填充）
显示属性， int(M)中，M的值跟int(M)所占多少存储空间并无任何关系，都占4字节
1. 若长度不足M，则在前面显示时候添加0
1. 若使用zerofill，自动成为unsigned
```
MariaDB [test]> create table data_int(a int(4),b int(1) zerofill,c  int(4) zerofill);
MariaDB [test]> insert into data_int values(1,1,1);
MariaDB [test]> select * from data_int;
+------+------+------+
| a    | b    | c    |
+------+------+------+
|    1 |    1 | 0001 |
+------+------+------+

```
- ### 浮点型
- float（非精确型）
- double（非精确型）
- real<br>
==（decimal（M,D）M默认为10；要指定精度，否则D为0）==
```
MariaDB [test]> create table data_f2(a float,b double,c decimal,d decimal(10,5));
MariaDB [test]> insert into data_f2 values(1.23456,1.23456,1.23456,1.23456);
 [test]> select a *3.567 /3.567, b * 3.567 /3.567, c* 3.567 /3.567,d * 3.567 /3.567  from data_f2;


+--------------------+--------------------+-----------------+------------------+
| a *3.567 /3.567    | b * 3.567 /3.567   | c* 3.567 /3.567 | d * 3.567 /3.567 |
+--------------------+--------------------+-----------------+------------------+
| 1.2345600128173826 | 1.2345599999999999 |       1.0000000 |   1.234560000000 |
+--------------------+--------------------+-----------------+------------------+

```
- ### bit类型(bit数据不直接显示)

```
MariaDB [test]> create table b ( a bit);

MariaDB [test]> insert into b values(b'1');
MariaDB [test]> select a,  hex(a) from b;

+------+--------+
| a    | hex(a) |
+------+--------+
|     | 1      |
+------+--------+


```
