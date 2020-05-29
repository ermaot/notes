本章来自《postgresql实战》和《postgresql修炼之道，从小工到专家》

## 数字类型

类型|长度|说明|范围|与其他db比较
---|---|---|---|---|
smallint|2字节|小范围整数类型|32768到+32767
integer|4字节|整数类型|2147483648到+2147483647
bigint|8字节|大范围整数类型|-9233203685477808到+9223203685477807
decimal|可变|用户指定|精度小数点前131072位；小数点后16383位
numeric|可变|用户指定|精度小数点前131072位；小数点后16383位
real|4字节|变长，不精确|6位十进制精度
double precision|8字节|变长，不精确|15位十进制精度
smallserial|2字节|smallint I自增序列|1到32767
serial|4字节 |Integer自增序列|1到2147483647
bigserial|8字节|bigint自增序列|1到922372036854775807
money|8字节|金钱类型|

#### 运算以及函数介绍：

```
#select 3+5,3-5,3*5,5/3,5.0/3,5%3,mod(5,3);
 ?column? | ?column? | ?column? | ?column? |      ?column?      | ?column? | mod 
----------+----------+----------+----------+--------------------+----------+-----
        8 |       -2 |       15 |        1 | 1.6666666666666667 |        2 |   2
(1 row)

#select round(10.1),round(10.9),ceil(4.3),ceil(4.9),ceil(-3.6),ceil(-3.4),floor(3.2),floor(-3.2);
 round | round | ceil | ceil | ceil | ceil | floor | floor 
-------+-------+------+------+------+------+-------+-------
    10 |    11 |    5 |    5 |   -3 |   -3 |     3 |    -4
(1 row)
```



## 字符类型
类型|长度|说明|范围|与其他db比较
---|---|---|---|---
character varying(n),varchar(n)|变长|字符最大数有限制
character(n), char(n)|定长|字符数没达到最大值则使用空白填充
text|变长|无长度限制

#### 字符函数

char_length与octet_length

```
=#select char_length('test'),octet_length('test'),char_length('中国'),octet_length('中国');
 char_length | octet_length | char_length | octet_length 
-------------+--------------+-------------+--------------
           4 |            4 |           2 |            6
(1 row)
```

substring与substr很相似，但substr不支持from……for这种用法

```
#select substring('test' from 3 for 5),substring('test',3,5),substring('test',3,4);
 substring | substring | substring 
-----------+-----------+-----------
 st        | st        | st
(1 row)

#select substr('test',3,5),substr('test',3,4);
 substr | substr 
--------+--------
 st     | st
(1 row)

#select substr('test' from 3 for 4);
ERROR:  syntax error at or near "from"
LINE 1: select substr('test' from 3 for 4);
                             ^
```

position返回第一个匹配的位置，若没匹配则返回0

```
#select position('a' in 'abcda'),position('ab' in 'abcda'),position('ad' in 'abcda');
 position | position | position 
----------+----------+----------
        1 |        1 |        0
(1 row)

```

split_part

```
#select split_part('abcdabcab','a',1),split_part('abcdabcab','a',2),split_part('bcdabcab','a',1);
 split_part | split_part | split_part | split_part 
------------+------------+------------+------------
            | bcd        | bcd        | bc
(1 row)
```

## 时间与日期
类型|长度|说明|范围|与其他db比较
---|---|---|---|---
timestamp\[(p)\]\[without time zone\]|8字节|包括日期和时间，不带时区，简写成 timestamp 
timestamp\[(p)\] with time zone|8字节|包括日期和时间，带时区，简写成 timestamp 
date|4字节|日期，但不包含一天中的时间
time\[(p)\]\[without time zone\]|8字节|天中的时间，不包含日期，不带时区
time\[(p)\]with time zone|12字节|天中的时间，不包含日期，带时区
interval\[fields\]\[(p)\]|16字节|时间间隔

###### 类型转换

```
#select now(),now():: timestamp with time zone,now():: timestamp without time zone;
              now              |              now              |            now             
-------------------------------+-------------------------------+----------------------------
 2020-05-22 14:26:34.770816+08 | 2020-05-22 14:26:34.770816+08 | 2020-05-22 14:26:34.770816
(1 row)
```

可以看到now是默认带时区的

```
#select now()::date,now():: time with time zone,now():: time without time zone,now():: time(3) without time zone;
    now     |        now         |       now       |     now      
------------+--------------------+-----------------+--------------
 2020-05-22 | 14:56:28.395949+08 | 14:56:28.395949 | 14:56:28.396
(1 row)
```

可以看到精度的区别，默认和最大为6，超过6会显示warning而直接截断

```
#select date '2017-08-09'+ interval '1 day' - interval '2 hour',interval '1 month'/double precision '3';
      ?column?       | ?column? 
---------------------+----------
 2017-08-09 22:00:00 | 10 days
(1 row)
```

###### 时间函数
```
#select current_date,current_time;
    date    |       timetz       
------------+--------------------
 2020-05-22 | 15:11:33.052683+08
(1 row)

#select extract(hour from current_time),extract(hour from current_date),extract(week from now());
 date_part | date_part | date_part 
-----------+-----------+-----------
        15 |         0 |        21
(1 row)
```

## 网络类型
类型|长度|说明|范围|与其他db比较
---|---|---|---|---
cidr|7或19字节|IPV4和IPv6网络
inet|7或19字节|IPv4和IPv6网络
macaddr|6字节|MAC地址|
macaddr8|8字节|MAC地址(EUI-64格式)

举例:

```
#select '192.168.1.1'::cidr,'192.168.1.100/24'::inet;
      cidr      |       inet       
----------------+------------------
 192.168.1.1/32 | 192.168.1.100/24
(1 row)
```

可以看到inet不一定会转换成普通的点分十进制的形式