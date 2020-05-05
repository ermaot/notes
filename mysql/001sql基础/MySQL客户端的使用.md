## 新建用户名和密码

```
GRANT ALL [PRIVILEGES] ON test.* TO 'test'@'LOCALHOST' IDENTIFIED BY '123456';  //
GRANT ALL [PRIVILEGES] ON test.* TO 'ROOT'@'%' IDENTIFIED BY '123456';
```
！！注意以下命令，可以执行，而且会给test 用户 select,delete之外的权限

```
GRANT selete,delete ON test TO 'test'@'LOCALHOST' IDENTIFIED BY '123456';  //需在特定的数据库中执行才不报错
# mysql -utest   -p123456   test
> create table test2(a int);
> insert into test2 values(1);
> select * from test2;
+------+
| a    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)
> update test2 set a = 2;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
> delete from test2;
Query OK, 1 row affected (0.01 sec)

// 查看授权
> show grants for test@localhost;
+-------------------------------------------------------------------------------------------------------------+
| Grants for test@localhost                                                                                   |
+-------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test'@'localhost' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
| GRANT SELECT, DELETE ON `test`.`test` TO 'test'@'localhost'                                                 |
+-------------------------------------------------------------------------------------------------------------+

//如果更改授权语句
GRANT select,delete ON test.* TO 'test'@'LOCALHOST' IDENTIFIED BY '123456';
> delete from test2;
Query OK, 0 rows affected (0.00 sec)
> update test2 set a = 3;
ERROR 1142 (42000): UPDATE command denied to user 'test'@'localhost' for table 'test2'

//这个时候比较有意思的是，如果再度执行
GRANT select,delete ON test TO 'test'@'LOCALHOST' IDENTIFIED BY '123456'; 

```

## 登录数据库

```
#mysql -utest -p                //如果不跟数据库名，则登录后不进入任何库中
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4772
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>

```
以下方式都可以登录到test库中，数据库名的位置非常灵活
```
# mysql -utest -p test
# mysql test -utest -p 
# mysql -utest test -p 
# mysql -utest test  -p  -h localhost
# mysql -utest   -p  -h localhost test
```

参数类型 |可选项语法格式| 默认值
---|---|---
Hostname|-h hostname <P> --host = hos tname|localhost
Username|-U username Your <P>--user = username|login name
Password|-P <P>--password|None

- MySQL对于127.0.0.1和localhost表现不同
1. 依照惯例,它们特殊对待主机名localhost, 这时它们会尝试使用一个Unix domain socket文件来连接本地服务器。
2. 要强制使用TCP/IP连接到本地服务器,那就使用IP地址127.0.0.1而不是主机名localhost。
3. 可选的,也可以通过指定--protocol=tcp选项来强制使用TCP/IP进行连接

==以上可以通过tcpdump抓包验证。localhost是不能通过tcpdump -i lo port 3306 抓到包的，而127.0.0.1可以==

不论查看权限还是删除用户，都需要带登录主机选项。==原因在于MySQL判断的时候是加上该字符串一起的==

- socket登录
1. Unix domain Socket的路径名常常变化,通常情况为/tmp/mysql.sock
2. 为显式指定套接字文件路径名,使用-s file_ name或--socket= file_ name选项

##查看版本和当前数据库

```
> select database();
+------------+
| database() |
+------------+
| test       |
+------------+
1 row in set (0.00 sec)

> select version();
+----------------+
| version()      |
+----------------+
| 5.5.56-MariaDB |
+----------------+
1 row in set (0.00 sec)
```

## 查看权限  
```
> show grants for test@localhost;

```
user权限列决定了用户的权限，描述了用户在==全局范围内==允许对数据库和数据库进行的操作
```
select * from mysql.user
```

## 删除用户


```
> drop user test@localhost;
```


## 创建数据库和表

```
create database cookbook;
use cookbook;
create table cookbook_test(a int , b int);
```
## 删除表

```
drop table XXX;
```

## MySQL自动补全
my.cnf

```
auto-rehash//可以使用tab自动补全 
//对应的是skip-auto-rehash
```

## 避免手工输入参数（cnf文件中保存密码）
my.cnf
```
[client]
host = localhost
user = cbuser
password = cbpasswd
```
## 显示分页的定制
1. 写入my.cnf
```
[client]
pager="/usr/bin/less -E"
//pager="/usr/bin/more -100"
pager=grep sync;   //后续执行命令时，后面相当于都加上了  |grep sync;
//nopager;
```
2. 通过登录选项

```
mysql -utest  --pager="/usr/bin/more -100"
mysql -utest  --pager="/usr/bin/less -E"
mysql -utest  --pager="/usr/bin/less -E| grep a"
```
3. 在MySQL中的shell行尾加\P或者单独执行 \P ，甚至可以接命令定制翻页方式

```
\P			//pager分页
\P /usr/bin/less
```
使用方法
```
mysql> select * from test ;\P /usr/bin/more
```
## 查看登录选项

```
 # mysql --print-defaults
 或者
 # my_print_defaults client mysql
 
 注意：这就是执行命令，不需要加其他的参数（比如-u -p之类）
```
## 取消尚未完成的sql

```
select test from \c

```
## 改变显示格式

```
select * from test \G // 这时候不需要加分号，否则\G会单独作为一个语句执行
```
## sql输入操作技巧

```
;			//分号
\g			//结束
\c			//取消
Ctrl + A	//行首
Ctrl + E	//行尾
>
->
\P			//pager分页
\P /usr/bin/less
\n			//nopager
```

## 文件中执行语句

```
# mysql -utest -p123456 test < filename.sql
> source '$file_path'
# cat file.sql | mysql test
# mysql -utest -p123456 test -e "select * from test;"    //执行单行sql
# mysql -utest -p123456 test < infile.sql > outfile.txt  //执行命令到文件中

```

## 输出控制

//替换输出的tab成其他字符                      
```
% mysql -utest-p123456 test < infile.sql | sed -e "s/\t/:/g"	                              
% mysql -utest-p123456 test < infile.sql | sed -e "s/\t/:/g"
```
//以html格式输出 
```
% mysql -utest-p123456 test -e "select * from url"  -H  > 1.txt                              
% mysql -utest-p123456 test -e "select * from url"  --html  > 1.txt
```
//以xml格式输出  -X也可以放前面
```
% mysql -utest-p123456 test -e "select * from url"  -X  > 1.txt
% mysql -utest-p123456 test -e "select * from url"  --xml  > 1.txt
```
//跳过头部
```
% mysql -utest-p123456 test -e "select * from url" --skip-column-names	              
% mysql -utest-p123456 test -e "select * from url" -ss
```
//控制垂直输出（三者效果一样） 
```
mysql > select * from test \G					                            
% mysql -utest-p123456 test -e "select * from url" -E                                                  
% mysql -utest-p123456 test -e "select * from url" --vertical
```
//控制信息详细程度，详细输出 
```
% mysql -utest-p123456 test -e "select * from url" -v                                                  
% mysql -utest-p123456 test -e "select * from url" -vv
```
//记录交互式mysql 
```
mysql > \T temp.out                                                                
mysql > \t
```

## 历史命令
```
~/.mysql_history
```

## 本文来自《mysql cookbook》