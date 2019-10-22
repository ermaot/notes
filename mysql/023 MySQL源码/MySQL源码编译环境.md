## 安装gcc5.5

1. 下载gcc5.5包。可以在gnu官网上找合适的镜像，我这边是[US, San Jose:镜像](http://www.netgull.com/gcc/releases/gcc-5.5.0/gcc-5.5.0.tar.gz)比较快，大约2分钟下载完毕
2. 安装编译软件包

```
# yum install  mpfr-devel  libmpc-devel  glibc   libmpc
因为编译过程中出现过configure: error: error verifying int64_t uses long long
说明缺少g++（gcc-c++），安装
# yum install gcc-c++
```
3. 增加swap 分区<p>
因为出现过make[3]: *** [s-attrtab] 已杀死  说明内存不足
```
创建一个swap文件
# dd if=/dev/zero of=/swapfile bs=4096 count=512k
然后
# mkswap    /swapfile 
正在设置交换空间版本 1，大小 = 2097148 KiB
无标签，UUID=0f4d30f8-79a5-4a27-9e3f-1649a3335db3
# swapon /swapfile
查看
# free -m
              total        used        free      shared  buff/cache   available
Mem:            992         616          78          10         296         200
Swap:          2047           0        2047
```

4. 开始编译gcc5.5（花费2个半小时）

```
# ./configure --prefix=/usr/local/gcc-5.5.0 --disable-multilib
# make
# make install

```
## MySQL8编译
1. 安装cmake

```
# yum install cmake
```
2. 建个编译用的目录，注意不要建在源码包解压后的目录下，否则cmake会报错

```
# mkdir mysql_debug
# cd mysql_debug/
```
3. 安装openssl 和ncurses

```
# yum install openssl-devel
# yum install ncurses-devel
```
4. 设置环境变量，以便在编译的时候使用GCC5.5.0及其lib

```
# CC=/usr/local/gcc-5.5.0/bin/gcc
# CXX=/usr/local/gcc-5.5.0/bin/g++
# export CC CXX
# export LD_LIBRARY_PATH=/usr/local/gcc-5.5.0/lib64
```
5. 用cmake进行编译配置（花费1个小时）

../mysql-8.0.17/ 是源码解压后的目录；

-DCMAKE_BUILD_TYPE=Debug 是指明编译出一个用来debug（跟踪代码）的版本

-DWITH_BOOST=../mysql-8.0.17/boost/boost_1_69_0/ 指明boost头文件路径

-DCMAKE_INSTALL_PREFIX=/usr/local/mysql80 指明将来make install的时候，把MySQL软件安装在哪里，不设置的话，缺省是/usr/local/mysql；

```
cmake3 ../mysql-8.0.17/ -DCMAKE_BUILD_TYPE=Debug -DWITH_BOOST=../mysql-8.0.17/boost/boost_1_69_0/ -DCMAKE_INSTALL_PREFIX=/usr/local/mysql8017
```
## 调试
1. 查看编译结果
```
# ls /usr/local/mysql8017/ -l
total 816
drwxr-xr-x  2 root root   4096 Aug  8 13:50 bin
drwxr-xr-x  2 root root   4096 Aug  8 13:50 docs
drwxr-xr-x  3 root root   4096 Aug  8 13:50 include
drwxr-xr-x  5 root root   4096 Aug  8 13:50 lib
-rw-r--r--  1 root root 336955 Jun 25 18:23 LICENSE
-rw-r--r--  1 root root 101805 Jun 25 18:23 LICENSE.router
-rw-r--r--  1 root root 336955 Jun 25 18:23 LICENSE-test
drwxr-xr-x  4 root root   4096 Aug  8 13:50 man
drwxr-xr-x 10 root root   4096 Aug  8 13:50 mysql-test
-rw-r--r--  1 root root    687 Jun 25 18:23 README
-rw-r--r--  1 root root    700 Jun 25 18:23 README.router
-rw-r--r--  1 root root    687 Jun 25 18:23 README-test
drwxrwxr-x  2 root root   4096 Aug  8 13:50 run
drwxr-xr-x 28 root root   4096 Aug  8 13:50 share
drwxr-xr-x  2 root root   4096 Aug  8 13:50 support-files
drwxr-xr-x  3 root root   4096 Aug  8 13:50 var
```
2. 创建用户以及必要的文件夹

```
# groupadd mysql
# useradd -g mysql mysql
# mkdir /data/mysql8017 -p
# mkdir /data/mysql8017/datadir
# chown mysql:mysql /data/mysql8015/datadir/
```
3. 建立自己的配置文件

```
# cat /data/mysql8017/my.cnf
[mysqld]
user             = mysql
basedir          = /usr/local/mysql8017
datadir          = /data/mysql8017/datadir
socket           = /data/mysql8017/datadir/mysql.sock
mysqlx_socket    = /data/mysql8017/datadir/mysqlx.sock
log-error        = /data/mysql8017/datadir/mysqlerror.log
```
4. 初始化和启动

- 初始化
```
# export LD_LIBRARY_PATH=/usr/local/gcc-5.5.0/lib64 
# bin/mysqld --defaults-file=/data/mysql8015/my.cnf --initialize --user=mysql
# ls /data/mysql8017/datadir/ -l
total 154692
-rw-r----- 1 mysql mysql       56 Aug  8 13:59 auto.cnf
-rw------- 1 mysql mysql     1676 Aug  8 13:59 ca-key.pem
-rw-r--r-- 1 mysql mysql     1112 Aug  8 13:59 ca.pem
-rw-r--r-- 1 mysql mysql     1112 Aug  8 13:59 client-cert.pem
-rw------- 1 mysql mysql     1676 Aug  8 13:59 client-key.pem
-rw-r----- 1 mysql mysql     5332 Aug  8 13:59 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Aug  8 13:59 ibdata1
-rw-r----- 1 mysql mysql 50331648 Aug  8 13:59 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Aug  8 13:59 ib_logfile1
drwxr-x--- 2 mysql mysql     4096 Aug  8 13:59 #innodb_temp
drwxr-x--- 2 mysql mysql     4096 Aug  8 13:59 mysql
-rw-r----- 1 mysql mysql      442 Aug  8 13:59 mysqlerror.log
-rw-r----- 1 mysql mysql 24117248 Aug  8 13:59 mysql.ibd
drwxr-x--- 2 mysql mysql     4096 Aug  8 13:59 performance_schema
-rw------- 1 mysql mysql     1676 Aug  8 13:59 private_key.pem
-rw-r--r-- 1 mysql mysql      452 Aug  8 13:59 public_key.pem
-rw-r--r-- 1 mysql mysql     1112 Aug  8 13:59 server-cert.pem
-rw------- 1 mysql mysql     1680 Aug  8 13:59 server-key.pem
drwxr-x--- 2 mysql mysql     4096 Aug  8 13:59 sys
-rw-r----- 1 mysql mysql 10485760 Aug  8 13:59 undo_001
-rw-r----- 1 mysql mysql 10485760 Aug  8 13:59 undo_002
```
- 启动

```
# mysqld_safe --defaults-file=/data/mysql8017/my.cnf --user=mysql 
```
- 使用客户端并修改密码

```
# /mysql -S /data/mysql8017/datadir/mysql.sock   -uroot -p
mysql>  alter user root@localhost identified by '';
Query OK, 0 rows affected (0.02 sec)
```
