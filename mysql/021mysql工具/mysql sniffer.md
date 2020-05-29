本文参考https://www.cnblogs.com/276815076/p/6518824.html

https://www.cnblogs.com/MakeInstall/p/10868983.html

## 简介

MySQL Sniffer 是一个基于 MySQL 协议的抓包工具，实时抓取 MySQLServer 端的请求，并格式化输出。输出内容包访问括时间、访问用户、来源 IP、访问 Database、命令耗时、返回数据行数、执行语句等。有批量抓取多个端口，后台运行，日志分割等多种使用方式，操作便捷，输出友好。

由奇虎360公司开发

建议在 centos6.2 及以上编译安装，并用 root 运行。

## sniffer安装

```
git clone https://github.com/Qihoo360/mysql-sniffer
cd mysql-sniffer
mkdir proj
cd proj
cmake ../
make
cd bin/
```

需要依赖于glib2-devel(2.28.8)、libpcap-devel(1.4.0)、libnet-devel(1.1.6)

如果出现了下面的问题

```
Linking CXX executable mysql-sniffer
/usr/bin/ld: /root/mysql-sniffer/lib/libgthread-2.0.a(gthread-impl.o): undefined reference to symbol 'pthread_setspecific@@GLIBC_2.2.5'
//usr/lib64/libpthread.so.0: error adding symbols: DSO missing from command line
collect2: error: ld returned 1 exit status
make[2]: *** [bin/mysql-sniffer] Error 1
make[1]: *** [bin/CMakeFiles/mysql-sniffer.dir/all] Error 2
make: *** [all] Error 2
```

原因在于pthread 库不是 Linux 系统默认的库，连接时需要使用静态库 libpthread.a，所以在使用[pthread_create](https://www.baidu.com/s?wd=pthread_create&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)()创建线程，以及调用 pthread_atfork()函数建立fork处理程序时，需要链接该库。所以在编译中要加 -lpthread参数。

```
#vim CMakeFiles/mysql-sniffer.dir/link.txt
/usr/local/gcc-5.5.0/bin/g++   -DENABLE_TCPREASM -O3 -Wall  -DENABLE_TCPREASM -O2    CMakeFiles/mysql-sniffer.dir/main.c.o CMakeFiles/mysql-sniffer.dir/mysql-dissector.c.o CMakeFiles/mysql-sniffer.dir/util.c.o CMakeFiles/mysql-sniffer.dir/session.cpp.o CMakeFiles/mysql-sniffer.dir/sniff-config.cpp.o CMakeFiles/mysql-sniffer.dir/sniff-log.cpp.o  -o mysql-sniffer  -L/root/mysql-sniffer/lib -rdynamic -Wl,-Bstatic -lnidstcpreasm -lnet -lpcap -lglib-2.0 -lgthread-2.0 -Wl,-Bdynamic -lrt -Wl,-Bstatic -lnet -lpcap -lglib-2.0 -lgthread-2.0 -Wl,-Bdynamic -lrt -Wl,-rpath,/root/mysql-sniffer/lib -lpthread
```

在最后加上-lpthread即可编译通过



## sniffer 使用方法

```
./mysql-sniffer -h
Usage mysql-sniffer [-d] -i eth0 -p 3306,3307,3308 -l /var/log/mysql-sniffer/ -e stderr
         [-d] -i eth0 -r 3000-4000
         -d daemon mode.
         -s how often to split the log file(minute, eg. 1440). if less than 0, split log everyday
         -i interface. Default to eth0
         -p port, default to 3306. Multiple ports should be splited by ','. eg. 3306,3307
            this option has no effect when -f is set.
         -r port range, Don't use -r and -p at the same time
         -l query log DIRECTORY. Make sure that the directory is accessible. Default to stdout.
         -e error log FILENAME or 'stderr'. if set to /dev/null, runtime error will not be recorded
         -f filename. use pcap file instead capturing the network interface
         -w white list. dont capture the port. Multiple ports should be splited by ','.
         -t truncation length. truncate long query if it's longer than specified length. Less than 0 means no truncation
         -n keeping tcp stream count, if not set, default is 65536. if active tcp count is larger than the specified count, mysql-sniffer will remove the oldest one
```

## sniffer 样例

```
# ./mysql-sniffer -p 3306 -i eth0


2020-05-14 11:08:16	 commerce_person	 172.31.180.74	 commerce_person	         45ms	          0	 SET character_set_results = NULL
2020-05-14 11:08:17	 commerce_person	 172.31.180.74	 commerce_person	         44ms	          0	 SET autocommit=1
2020-05-14 11:08:17	 commerce_person	 172.31.180.74	 commerce_person	         44ms	          0	 SET sql_mode='STRICT_TRANS_TABLES'
2020-05-14 11:08:17	 commerce_person	 172.31.180.74	 commerce_person	         44ms	          0	 SET autocommit=1
```

似乎不能抓取lo端口的包

可以-p指定多个端口，端口之间用逗号分隔
```
mysql-sniffer -i eth0 -p 3306,3307,3310 -l /tmp
```
可以-r指定端口范围
```
mysql-sniffer -i eth0 -r 3306-3310 -l /tmp
```

可以使用-w过滤端口
```
mysql-sniffer -i eth0 -r 3306-3310  -w 3308,3309 -l /tmp
```
可以使用守护模式
```
mysql-sniffer -i eth0 -p 3306  -l /tmp -d
```