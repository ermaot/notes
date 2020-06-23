## 字符串属性
```
show character set ;   //查看字符集。mysql之前默认latin1，mysql8默认是utf8mb4
select * from information_schema.CHARACTER_SETS ;
select * from url where length(url) = char_length(url) ;	//char_length()  && length()
show collation like 'latin%' ;              //查看排序列表
```
使用collation排序

```
select * from a order by a collate utf8_general_ci;
```

#### utf8mb4 与 utf8
- MySQL在5.5.3之后增加了 utf8mb4 的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。
- utf8mb4是utf8的超集，除了将编码改为utf8mb4外不需要做其他转换
- MySQL原来的utf8不是真正的UTF-8，所以发布utf8mb4修复这个问题
-  要使用 utf8mb4 节省空间，使用 VARCHAR 替换 CHAR。否则，MySQL必须为使用 utf8mb4字符集的列的每一个字符保留四字节的空间。例如，MySQL必须为一个使用 utf8mb4 字符集的  char（10）的列保留40字节空间。
-  utf8mb4支持emoji
-  utf8升级utf8mb4具体步骤
1. sql

```
1 # 修改数据库:  
2 ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;  
3 # 修改表:  
4 ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;  
5 # 修改表字段:  
6 ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;  

```
2. 修改MySQL配置文件

```
default-character-set = utf8mb4 
default-character-set = utf8mb4    
character-set-client-handshake = FALSE  
character-set-server = utf8mb4  
collation-server = utf8mb4_unicode_ci  
init_connect='SET NAMES utf8mb4'
```
3. 查看修改结果

```
SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';  
```
## 设置客户端字符集
配置文件
```
[mysql]
default-character-set=utf8mb4
```
建立连接后执行SET NAMES语句:

```
mysql> SET NAMES 'utf8mb4' ;
```

SET NAMES同时也允许指定连接的Collation:

```
mysql> SET NAMES 'utf8mb4' COLLATE 'utf8mb4_unicode_ci' ;
```
查看当前collation

```
> show variables like "%collation%";
+-------------------------------+--------------------+
| Variable_name                 | Value              |
+-------------------------------+--------------------+
| collation_connection          | utf8mb4_unicode_ci |    //当前会话collation
| collation_database            | utf8mb4_0900_ai_ci |
| collation_server              | utf8mb4_0900_ai_ci |
| default_collation_for_utf8mb4 | utf8mb4_0900_ai_ci |
+-------------------------------+--------------------+
4 rows in set (0.00 sec)
```

## 选择字符串类型
#### 类型说明
二进制类型| 非二进制类型|最大长度
---|---|---
binary	|	char	|	255
varbinary	|varchar|		65535
tinyblob|	tinytext|	255
blob|		text	|	65535
mediumblob|	mediumtext|	16777215
longblob|	longtext|	4294967295


## 类型tips
- char(10) ：存储时填充nul(0x00)
- binary(10) ：存储时填充空白
- 检索读取的时候会将填充符去除
- varbinary varchar blob text：  根据实际长度填充


## 列字符集

```
create table mytbl
(
	utf8data varchar(100) character set utf8 collate utf8_danish_ci,
	sjisdata varchar(100) character set sjis collate sjis_japanese_ci
);
```
查看列字符集

```
select user(), charset(user()), collation(user()) ;
select collation(sjisdata) from mytbl;
select user(), charset(user()), collation(convert(user() using utf8)) ;						//convert转换字符集
select user(), charset(user()), collation(convert(user() using binary)) ;
select  concat(upper(left('thing',1)),mid('thing',3,4), right('thing', 3), lower('THing')) ;	//Tingingthing  (T + ing + ing + thing )
```
## 字符串操作

```
select  'ab cd' ;
select  "ab cd" ;
select 0x61626364				//输出 abcd
select x'61626364';				//输出 abcd
select X'61626364';				//输出 abcd
select _utf8 'abcd' ;			//输出 abcd
select _latin1 'abcd' ;			//输出 abcd
select _ucs2 'abcd' ;			//输出 慢捤
select "He said \"I'm OK\"" ;   //字符转义
```

## 字符大小写

```
> select lower("TEST"),upper("test");
+---------------+---------------+
| lower("TEST") | upper("test") |
+---------------+---------------+
| test          | TEST          |
+---------------+---------------+
1 row in set (0.00 sec)

```
如果转换失败，可能是由于字符串是binary类型，先转换成char

```
> select upper(binary("test" )),upper(convert(binary("test") using utf8));
+------------------------+-------------------------------------------+
| upper(binary("test" )) | upper(convert(binary("test") using utf8)) |
+------------------------+-------------------------------------------+
| test                   | TEST                                      |
+------------------------+-------------------------------------------+
```
## 字符串比较

```
//当一个二进制字符串和非二进制字符串比较的时候，转换成二进制字符串
> select _utf8 "test" = binary("test");
+-------------------------------+
| _utf8 "test" = binary("test") |
+-------------------------------+
|                             1 |
+-------------------------------+
1 row in set, 1 warning (0.00 sec)

//binary可以加括号，也可以不加
> select _utf8 "test" = binary "test";
+------------------------------+
| _utf8 "test" = binary "test" |
+------------------------------+
|                            1 |
+------------------------------+
1 row in set, 1 warning (0.00 sec)
```
## SQL字符串匹配

```
select * from test where a like '%test%' and b like '_test%' and  c like '____test__%' ;	//null匹配任何，都是返回null;_代表一个字符
```
#### 正则

```
^					//字符串开头
$					//字符串结尾
.					//任意一个字符，包括回车和换行
[...]				//括号内任意一个字符
[^...]				//括号内任意字符外的其他字符
p1 | p2 | p3		//p1 p2 p3 任意一个模式或者串
*					//匹配0个或者多个*前的序列
+					//匹配1个或者多个*前的序列
{ n }				//匹配n个{n}之前的序列
{ m , n }			//匹配{m,n}之前的序列m次和n次之间
select * from url where url regexp '^http' and url regexp 'htm$' and url regexp '^.......ww'  and url regexp 'den' limit 10;
select * from url where url regexp '[[:digit:]]{3}' limit 5;
select * from url where url regexp 'd{3}' limit 5;
```
POSIX类 |匹配定义
---|---
[ : alnum: ] |字符和数字
[ :alpha:] |字母
[ :blank: ] |空格或制表符(tab)
[ :cntrl: ] |控制符
[ :digit: ] |数字
[ :graph: ] |图形符号(不包括空格)
[:lower:] |小写字母
[ :print: ] |图形符号(包括空格)
[ :punct: ] |标点符号
[ : space: ] |空格、制表符、换行、回车换行
[ :upper : ] |大写字母
[ :xdigit:] |十六进制符(0-9、 a-f、 A-F)

#### 查询子串

```
//locate第一个参数表示字符串模式，第二个字段名，第三个是从该处开始
select cip,url , locate('in', url),  locate('in', url,3) from url  where locate('in', url)>3 order by  url desc limit 5;	
```
## 对非字符型的匹配
#### 数字型

```
int 列
> desc a;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+

> select * from a where a like "1%";
+------+
| a    |
+------+
|    1 |
|    1 |
+------+
2 rows in set (0.01 sec)

```

#### 日期型
函数值测试| 模式匹配测试
---|---
YEAR(a) = 1976 |a LIKE '1976-8'
MONTH(a) = 4 |a LIKE '%-04-%'
DAYOFMONTH(a) = 1 |a LIKE '%-01'

```
> desc date_test;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| a     | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------+-----------+------+-----+-------------------+-----------------------------+
> select * from date_test where a like "20%";
+---------------------+
| a                   |
+---------------------+
| 2019-07-17 10:41:41 |
+---------------------+

//注意，这种日期型比较在mysql8不再有效
> select version();
+-----------+
| version() |
+-----------+
| 8.0.16    |
+-----------+
1 row in set (0.00 sec)

> select * from date_test where a like "20%";
ERROR 1525 (HY000): Incorrect TIMESTAMP value: '20%'
```
## 字符串函数
函数名| 解释
---|---
ASCII(str)|返回字符串str的最左面字符的ASCII代码值。<br>如果str是空字符串，返回0。如果str是NULL，返回NULL。
ORD(str)|              
CONV(N,from_base,to_base)|进制转换
BIN(N)|返回二进制值N的一个字符串表示，在此N是一个长整数(BIGINT)数字，这等价于CONV(N,10,2)。如果N是NULL，返回NUL
OCT(N)|返回八进制值N的一个字符串的表示，在此N是一个长整型数字，这等价于CONV(N,10,8)。如果N是NULL，返回NULL
HEX(N)|返回十六进制值N一个字符串的表示，在此N是一个长整型(BIGINT)数字，这等价于CONV(N,10,16)。如果N是NULL，返回NULL
CHAR(N,...)|CHAR()将参数解释为字符并且返回由这些整数的ASCII代码字符组成的一个字符串
LENGTH(str)|返回字符串str的长度
OCTET_LENGTH(str)|返回字符串str的长度
CHAR_LENGTH(str)|返回字符串str的长度
CHARACTER_LENGTH(str)|返回字符串str的长度
LOCATE(substr,str)|　返回子串substr在字符串str第一个出现的位置，如果substr不是在str里面，返回0
LOCATE(substr,str,pos)|返回子串substr在字符串str第一个出现的位置，从位置pos开始。如果substr不是在str里面，返回0
POSITION(substr IN str)|返回子串substr在字符串str第一个出现的位置，如果substr不是在str里面，返回0
INSTR(str,substr)|返回子串substr在字符串str中的第一个出现的位置，如果不在就返回0
LPAD(str,len,padstr)|返回字符串str，左面用字符串padstr填补直到str是len个字符长
RPAD(str,len,padstr)|返回字符串str，右面用字符串padstr填补直到str是len个字符长
LEFT(str,len)|返回字符串str的最左面len个字符
RIGHT(str,len)|返回字符串str的最右面len个字符
SUBSTRING(str,pos,len)　|SUBSTRING(str FROM pos FOR len)　
MID(str,pos,len)|从字符串str返回一个len个字符的子串，从位置pos开始
SUBSTRING(str,pos)<br>SUBSTRING(str FROM pos)|从字符串str的起始位置pos返回一个子串
SUBSTRING_INDEX(str,delim,count)|返回从字符串str的第count个出现的分隔符delim之后的子串，count可正可负
LTRIM(str)|返回删除了其前置空格字符的字符串str
RTRIM(str)|返回删除了其拖后空格字符的字符串str
TRIM([[BOTH | LEADING 
SOUNDEX(str)|返回str的一个同音字符串
SPACE(N)|返回由N个空格字符组成的一个字符串
REPLACE(str,from_str,to_str)|返回字符串str，其字符串from_str的所有出现由字符串to_str代替
REPEAT(str,count)|返回由重复countTimes次的字符串str组成的一个字符串。如果count <= 0，返回一个空字符串。如果str或count是NULL，返回NULL
REVERSE(str)|返回颠倒字符顺序的字符串str
INSERT(str,pos,len,newstr)|返回字符串str，在位置pos起始的子串且len个字符长得子串由字符串newstr代替
ELT(N,str1,str2,str3,...)|如果N= 1，返回str1，如果N= 2，返回str2，等等。如果N小于1或大于参数个数，返回NULL。ELT()是FIELD()反运算
FIELD(str,str1,str2,str3,...)|返回str在str1, str2, str3, ...清单的索引。如果str没找到，返回0。FIELD()是ELT()反运算
FIND_IN_SET(str,strlist)|如果字符串str在由N子串组成的表strlist之中，返回一个1到N的值
MAKE_SET(bits,str1,str2,...)|返回一个集合 (包含由“,”字符分隔的子串组成的一个字符串)，由相应的位在bits集合中的的字符串组成
LCASE(str)<br>LOWER(str)|返回字符串str
UCASE(str)<br>UPPER(str)|返回字符串str


## 字符串截取

#### left(),mid(),right(),substring(),substr()
- 可以看到mid(),substring(),substr()同义
- SUBSTRING_INDEX(str, c,n)： 该函数从str左侧查找字符c第n次出现的位置,然后返回该位置左侧的整个子串；如果n是负数,从str右侧开始查找c第Inl次出现的位置,并返回其右侧的整个子串
```
> create table char_test(a varchar(10));
> insert into char_test values("qwertyui"),("asdfghj"),("zxcvbn");
> select left(a,2)  left1 ,mid(a,3,1) as mid1,mid(a,3) as mid2,right(a,1) as right1 ,substring(a,3,1) as substring1,substring(a,3) as substring2 ,substr(a,3) as substr1,substr(a,3,1) as substr2,substriing_index(a,'e',1) as substr_idx1,substring_index(a,'e',-1) as substr_idx2 from char_test;
+-------+------+--------+--------+------------+------------+---------+---------+-------------+-------------+
| left1 | mid1 | mid2   | right1 | substring1 | substring2 | substr1 | substr2 | substr_idx1 | substr_idx2 |
+-------+------+--------+--------+------------+------------+---------+---------+-------------+-------------+
| qw    | e    | ertyui | i      | e          | ertyui     | ertyui  | e       | qw          | rtyui       |
| as    | d    | dfghj  | j      | d          | dfghj      | dfghj   | d       | asdfghj     | asdfghj     |
| zx    | c    | cvbn   | n      | c          | cvbn       | cvbn    | c       | zxcvbn      | zxcvbn      |
+-------+------+--------+--------+------------+------------+---------+---------+-------------+-------------+
3 rows in set (0.00 sec)
```

## 全文索引
- myisam表才可fulltext ; 太常见的fulltext词语会被忽略

```
create table full_text3(cip varchar(10), dip varchar(10), url text ) engine=myisam;
load data local infile 'f322350dffcaef18155bf2553af2fde6.urls' into table full_text;
alter table full_text3 add fulltext(url);
select * from full_text3 where match(url) against("offer") and match(url) against("world");
select * from full_text3 where match(url) against("+offer +world -http") in boolen mode ;           // boolean mode，排除某些词
```
- 全文索引参数

```
全文索引参数
[mysqld]
ft_min_word_len = 3
重启mysql然后
repair table full_text3 quick ;
```

## 本文来自《MySQL cookbook》