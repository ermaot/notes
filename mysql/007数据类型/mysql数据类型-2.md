## 字符型
### 1.字符集
#### ✔mysql默认字符集:latin1
#### show charset 查看所有支持的字符集

```
MariaDB [information_schema]> select * from CHARACTER_SETS;或者
MariaDB [(none)]> show charset;
+----------+-----------------------------+---------------------+--------+
| Charset  | Description                 | Default collation   | Maxlen |
+----------+-----------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese    | big5_chinese_ci     |      2 |
| dec8     | DEC West European           | dec8_swedish_ci     |      1 |
| cp850    | DOS West European           | cp850_general_ci    |      1 |
| hp8      | HP West European            | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian       | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European        | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European | latin2_general_ci   |      1 |
| swe7     | 7bit Swedish                | swe7_swedish_ci     |      1 |
| ascii    | US ASCII                    | ascii_general_ci    |      1 |
| ujis     | EUC-JP Japanese             | ujis_japanese_ci    |      3 |
| sjis     | Shift-JIS Japanese          | sjis_japanese_ci    |      2 |
| hebrew   | ISO 8859-8 Hebrew           | hebrew_general_ci   |      1 |
| tis620   | TIS620 Thai                 | tis620_thai_ci      |      1 |
| euckr    | EUC-KR Korean               | euckr_korean_ci     |      2 |
| koi8u    | KOI8-U Ukrainian            | koi8u_general_ci    |      1 |
| gb2312   | GB2312 Simplified Chinese   | gb2312_chinese_ci   |      2 |
| greek    | ISO 8859-7 Greek            | greek_general_ci    |      1 |
| cp1250   | Windows Central European    | cp1250_general_ci   |      1 |
| gbk      | GBK Simplified Chinese      | gbk_chinese_ci      |      2 |
| latin5   | ISO 8859-9 Turkish          | latin5_turkish_ci   |      1 |
| armscii8 | ARMSCII-8 Armenian          | armscii8_general_ci |      1 |
| utf8     | UTF-8 Unicode               | utf8_general_ci     |      3 |
| ucs2     | UCS-2 Unicode               | ucs2_general_ci     |      2 |
| cp866    | DOS Russian                 | cp866_general_ci    |      1 |
| keybcs2  | DOS Kamenicky Czech-Slovak  | keybcs2_general_ci  |      1 |
| macce    | Mac Central European        | macce_general_ci    |      1 |
| macroman | Mac West European           | macroman_general_ci |      1 |
| cp852    | DOS Central European        | cp852_general_ci    |      1 |
| latin7   | ISO 8859-13 Baltic          | latin7_general_ci   |      1 |
| utf8mb4  | UTF-8 Unicode               | utf8mb4_general_ci  |      4 |
| cp1251   | Windows Cyrillic            | cp1251_general_ci   |      1 |
| utf16    | UTF-16 Unicode              | utf16_general_ci    |      4 |
| cp1256   | Windows Arabic              | cp1256_general_ci   |      1 |
| cp1257   | Windows Baltic              | cp1257_general_ci   |      1 |
| utf32    | UTF-32 Unicode              | utf32_general_ci    |      4 |
| binary   | Binary pseudo charset       | binary              |      1 |
| geostd8  | GEOSTD8 Georgian            | geostd8_general_ci  |      1 |
| cp932    | SJIS for Windows Japanese   | cp932_japanese_ci   |      2 |
| eucjpms  | UJIS for Windows Japanese   | eucjpms_japanese_ci |      3 |
+----------+-----------------------------+---------------------+--------+
39 rows in set (0.01 sec)

```
#### status查看当前字符集

```
MariaDB [(none)]> status
--------------
mysql  Ver 15.1 Distrib 5.5.56-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:		57
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server:			MariaDB
Server version:		5.5.56-MariaDB MariaDB Server
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/var/lib/mysql/mysql.sock
Uptime:			30 min 22 sec

```
#### 字符集参数设置方法（服务器端）
1.配置参数修改法
```
[mysqld]
character_set_server = utf8
```
2.会话生效

```
set names "gbk"
```
#### 列字符集

```
MariaDB [test]> create table test_charset(a char(10) charset "gbk",b char(10) charset "utf8");
Query OK, 0 rows affected (0.08 sec)

```
### 2.排序规则（Collation）
- 不同的字符集，排序规则不同
- 每一个字符集有默认规则
- 规则末尾_ci结尾，表大小写不敏感, _cs表大小写敏感
#### 查看排序

```
show collation like "%utf8%"
select * from COLLATIONS where ……;
MariaDB [test]> show collation like "%utf8%";
+--------------------------+---------+-----+---------+----------+---------+
| Collation                | Charset | Id  | Default | Compiled | Sortlen |
+--------------------------+---------+-----+---------+----------+---------+
| utf8_general_ci          | utf8    |  33 | Yes     | Yes      |       1 |
| utf8_bin                 | utf8    |  83 |         | Yes      |       1 |
.
.
.
.

```
#### 修改会话的排序

```
set names collate utf8_bin
```
- utf8_bin 与 utf8_general_ci（大小写敏感与不敏感的区别）

#### 修改表的默认排序

```
alter table test_charset modify column a varchar(10) collate   utf8_general_ci;
```
### 3.char 和varchar

特点 | char| varchar
---|---|---
长度定义 | char(N)| varchar(N)
长度范围<p>（字节长度） | N~(0,255)| N~(0,65535)
自动填充<p>（右填充） | 是<p>（算长度的时候自动删除填充的空格）| 否
SQL_MODE   | pad_char_to_full_length<p>（算长度的时候不自动删除填充空格）| 
占用字节数 | char(N)：N| varchar(N)：<p>最大N+1，需要存字节长度
值比较 | 删掉填充比较值| 
存储 | | 大varchar转成text或者blob存储

### 4.binary 和 varbinary
特点 | binary| varbinary
---|---|---
长度定义 | binary(N)| varbinary(N)
长度范围<p>（字符长度） | N~(0,255)| N~(0,65535)
值比较 | 二进制值比较| 二进制值比较

### 5. blob 和 text
特点 | blob| text
---|---|---
子类型 | tinyblob(2^8)<p> blob(2^16)<p> mediumblob(2^24)<p> longblob(2^32)<p>| tinytext(2^8)<p> text(2^16)<p> mediumtext(2^24)<p> longtext(2^32)<p>
索引建立 | 指定前缀索引长度| 指定前缀索引长度
默认值 | 不可有| 不可有
排序 | max_sort_length（默认1024）| max_sort_length（默认1024）
存储 | Innoeb存储前20字节，其余放溢出页| 


```
> select @@max_sort_length;
+-------------------+
| @@max_sort_length |
+-------------------+
|              1024 |
+-------------------+

```
### 6. enum 和 set
特点 | enum| set
---|---|---
元素个数 | 最多65536| 最多64
长度范围<p>（字符长度） | N~(0,255)| N~(0,65535)
值比较 | 二进制值比较| 二进制值比较

## ==mysql不支持check约束==