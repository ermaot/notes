- 良好的逻辑设计和物理设计是高性能的基石，应该根据系统要求的查询语句来设计schema
- 考虑MySQL的独特特性和实现细节
## 选择优化的数据类型
- 更小的通常更好：准确预估存储的值范围；更改数据类型范围非常耗时和痛苦
- 简单就好：整型比字符型操作代价低（不考虑字符集和排序规则）；使用内建类型存日期而非字符串；使用整型存IP
- 尽量避免NULL：
1. 通常最好指定列为NOT NULL，除非确实有必要；
2. 包含null对MySQL使得索引、索引统计、值复杂比较难优化；
3. 为null的列使用更多存储空间；
4. null列被索引，每个索引记录需要额外的字节；
5. Myisam里固定大小的索引可能变成可变大小。
6. null变not null影响较小，可后面考虑
- timestamp存储时间范围比datetime小，占用空间为一半
- MySQL有数据类型别名，存储时还是会使用基本类型
#### 整数
- 整数有tinyint（8位），smallint（16位），mediumint（24位），int（32位），bigint（64位），存储范围是-2^(N-1)~2^(N-1)-1
- 加上unsigned后，不允许负值，存储范围是0~2^N-1
- 有符合与无符号使用相同的存储空间，性能相同
- 整数计算的时候一般使用64位的bigint，即使是32位环境中也是如此
- 整型指定宽度（int(11)）只是指在客户端等交互工具中用来显示字符的个数，对存储无影响。
#### 实数
- MySQL支持精确型，也支持不精确型
- float（4字节） 和double（8字节）支持使用==标准的浮点运算进行近似计算==（啥意思），取决于平台浮点数的具体实现
- decimal用于存储精确小数，但cpu不支持decimal的直接计算，MySQL服务器自身实现
- 浮点和decimal可以指定精度，但不建议自己指定精度（因为非标准的）
- 浮点数内部使用double计算
- 尽量只在对小数进行精确计算时使用decimal；数据量较大的时候，考虑使用bigint代替decimal
#### 字符串
- 字符串的排序规则会很大程度影响性能
- varchar和char在不同引擎内部可能存储不一样
- varchar可变长度，除非使用（row_format=fixed）
- varchar会使用一个或两个字节存储长度信息（大于255字节的时候使用2个字节存储长度）
- varchar节省了空间，对性能有帮助。update可能造成长度增加，如果页没有多余空间，Myisam分段存储，==innodb使用页分裂==（此处有疑问，需要再调查）
- innodb将过长的varchar存储为blob（使用溢出页来存储）
- char是定长的，存储时会删掉末尾的空格，同时会根据需要填充空格以方便比较
- char适合存储较短的字符串或者值都接近同一个长度（md5）
- 对于经常变更的数据，char不易产生碎片比varchar好
- 对于非常短的列，char比varchar有效率（因为varchar需要额外1字节存储长度）
- binary和varbinary：存储二进制字符串，以字节码存储，末尾以\0结束，检索时会去掉填充
- 存储二进制数据，按照每一个个字节比较，比字符比较简单速度快
- varchar(10)和varchar(50)相比，存储没有区别，但读取到内存的时候，varchar(50)占用内存更大；在临时表时，占用空间更大；排序时性能更糟糕
#### 日期和时间类型
- MySQL支持year
mariadb：支持year的长度语法，长度比较随意，实际最长只有4；小于2，则是year(2)
```
> create table year_test(a year,b year(2));
Query OK, 0 rows affected, 1 warning (0.03 sec)

> alter table year_test add column c year(1),add column d year(3),add column e year(5);
Query OK, 0 rows affected, 4 warnings (0.05 sec)   
Records: 0  Duplicates: 0  Warnings: 4

> desc year_test;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | year(4) | YES  |     | NULL    |       |
| b     | year(2) | YES  |     | NULL    |       |
| c     | year(4) | YES  |     | NULL    |       |
| d     | year(4) | YES  |     | NULL    |       |
| e     | year(4) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
5 rows in set (0.07 sec)
```
mysql8只支持长度为4的year

```
> create table year_test(a year,c year(3) ,b year(10));
ERROR 1818 (HY000): Supports only YEAR or YEAR(4) column.
```
- MySQL老版本支持最大的时间为秒（mariadb支持微秒，mysql8支持微秒）
- datetime最大范围从1001到9999，精度为秒（此处已经更新，mariadb和mysql8都已经能支持微秒了；mysql8会判断插入是否合法，mariadb5.5不会）
mariadb

```
> create table date_test(a datetime(4));
Query OK, 0 rows affected (0.03 sec)

> insert into date_test values('2001-9-00 00:00:00');
Query OK, 1 row affected (0.03 sec)

```
mysql8

```
create table date_time(a datetime(4));
> insert into date_test values('2001-09-00 00:00:00');
ERROR 1292 (22007): Incorrect datetime value: '2001-09-00 00:00:00' for column 'a' at row 1
```
- timestamp保存从1970年1月1日午夜依赖的秒数，与unix时间戳相同；4字节存储；表示从1970到2038年
- MySQL from_unixtime() 转换unix时间戳为日期；unixtimestamp()把日期转换为unix时间戳
- timestamp显示值依赖于时区
- 除特殊行为，一般选择timestamp而非datetime
- MySQL会将表的第一个timestamp值更新为当前时间
#### 位数据
- bit(N)，N最大为64
- bit存储依赖于存储引擎，Myisam会打包存储全部bit；而innodb 和memory使用足够存储的最小整数存储每一个bit列，所以不省空间
- MySQL把bit当字符串类型，而非数字类型。但在具体上下文中，会有诡异的行为

```
> CREATE TABLE bittest(a bit(8));
> INSERT INTO bittest VALUES(b'00111001');
Query OK, 1 row affected (0.01 sec)
> SELECT a, a + 0 FROM bittest;
+------+-------+
| a    | a + 0 |
+------+-------+
| 9    |    57 |
+------+-------+
1 row in set (0.00 sec)
```
- 如果要存储少数几种可能的离散值，可以使用SET；MySQL有find_in_set()和field()函数方便查找

#### 选择标识符
- 整数类型通常是最好的类型，还可以使用auto_increment
- enum和set类型通常是糟糕的选择，它们比较适合存储固定的信息
- 字符串应该被避免当作标识列：很消耗空间，且通常比数字类型慢；也需要注意md5、sha1、uuid产生的字符串，因为比较分散
- 存储uuid应该溢出“-”符号；更好是unhex()函数转换uuid为16字节的数字，并存储在binary(16)列中，检索时通过hex()函数格式化为16进制
- uuid分布不均匀，但也还是有一定顺序（==？？研究一下？？==）
#### 特殊类型数据
- IP地址是32位无符号整数，应该使用无符号整数存储ID
- MySQL使用INET_ATON() 和 INET_NTOA()函数在两种表示方法之间转换

```
> select INET_ATON("192.168.1.1"),INET_ATON("255.255.255.255");
+--------------------------+------------------------------+
| INET_ATON("192.168.1.1") | INET_ATON("255.255.255.255") |
+--------------------------+------------------------------+
|               3232235777 |                   4294967295 |
+--------------------------+------------------------------+
1 row in set (0.00 sec)

> select INET_NTOA(3532235777),INET_NTOA(4294967296);
+-----------------------+-----------------------+
| INET_NTOA(3532235777) | INET_NTOA(4294967296) |
+-----------------------+-----------------------+
| 210.137.164.1         | NULL                  |
+-----------------------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```

## MySQL schema设计中的陷阱
- 太多列：从行缓冲中将编码过的列转换成行数据结构的操作代价很高，转换的代价正比于列的数量，过宽的表会导致过高的代价
- 太多关联：实体-属性-值（EAV）设计模式常见但糟糕，容易超过MySQL最多61张表的关联限制；单个查询最好在12个表以内
## 范式与反范式
#### 范式的优点与缺点
***优点***
- 范式化的更新操作通常比反范式化要快.
- 当数据较好地范式化时,就只有很少或者没有重复数据,所以只需要修改更少的数据.
- 范式化的表通常更小,可以更好地放在内存里,所以执行操作会更快.
- 很少有多余的数据意味着检索列表数据时更少需要DISTINCT或者GROUPBY语句.
***缺点***：
- 关联比较多，导致昂贵代价，也有可能导致索引无效
#### 反范式的优点与缺点
- 可以很好避免关联
#### 混用范式化与反范式化
## 缓存表与汇总表
#### 物化视图
- 物化视图实际上是预先计算并且存储磁盘上的表，并通过各种策略更新
- MySQL不原生支持物化视图，justin swanhart开源工具flexviews
1. 变更数据抓取（change data capture，CDC），读取服务器二进制日志并解析相关行的变更
2. 一系列可以帮助创建和管理视图定义的存储过程
3. 一些可以应用变更到数据库的物化视图工具
#### 计数器表
- 计数器表很常见，可以缓存用户朋友数、文件下载次数等，通常创建一张独立的表存储计数器
- 可以建立多行，更新的时候随机更新，减少冲突
- 统计结果使用sum聚合查询
## 加快alter table的操作速度
- MySQL的alter table过程：用新结构创建一个空表，从旧表中查询所有数据插入新表，然后删除旧表。耗时长，业务影响大
- innodb plugin支持排序索引，建索引更快且更紧凑
- alter table两种方式：
1. 先在一台不提供服务的机器上执行alter table，然后和提供服务的主库切换
2. 影子拷贝。创建一张和原表无关的新表，然后重命名和删表交换两张表，比如online schema change（facebook）、openark toolkit（shlomi noach）、percona toolkit、flexviews的CDC工具
- alter table不引起表重建：

```
alter table sakila.film modify column rental_duration tinyint(3) not null default 5               //引起重建               

alter table sakila.file alter column rental_duration set default 5      //仅仅修改.frm文件
```

#### 只修改.frm文件
- 不需要重建表的操作：移除一个列的auto_increment属性；增加、移除、更改enum和set常量
#### 快速创建myiasn索引
- 先disable索引，load data，再enable索引（仅对非唯一索引有效）
- alter table骇客方法：
1. 创建所需的表（不包括索引）
2. 载入数据到表中，构建MYD文件
3. 按照需要的结构创建空表，包含索引，这会创建.frm和MYI文件
4. 获取读锁并刷新表
5. 重命名第二张表的.frm和MYI文件，让MySQL认为是第一张表
6. 释放读锁
7. 使用repair table重建表索引