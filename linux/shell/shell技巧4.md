## wget

```
//简单下载
# wget https://www.github.com
# wget https://www.github.com http://www.baidu.com http://www.163.com

//ftp
wget ftp://………………

//指定输出文件名并输出日志到文件
wget http://www.…….rar  -O test.rar -o log.out

//重试次数
wget -t 5  url

//限速
wget --limit-rate 20k url

//限制流量配额
wget -Q 100m url
wget --quota 100m url

//断点续传
wget -c url

//镜像网站
wget --mirror url
wget -r -N -l DEPTH url

//用户名和密码
wget --user username --password pass url
```

## lynux
下载文本形式网页

## curl


## 发送post

```
curl url -d "host=test-host&user=slynux"
curl --data "name=value" url -o output.html
```

## tar 
```

```


## 列出网络上所有活动的主机

```
//方法1
# for i in  172.31.180.{1..100} ;do ping $i -c 2;if [ $? -eq 0 ]; then echo "$i is active";fi;done

//方法2。同时发送，性能好
# fping -a 172.31.180.{77..80}
172.31.180.77
172.31.180.79
```

## lftp
```
lftp username@host
lcd     //改变本地目录
cd      //改变服务器目录
get  file // 下载文件
put file    //上传文件
```
## sftp
类似lftp，可以使用-oPort=参数指定端口

## scp传输文件

## ssh远程执行命令

```
# ssh root@172.31.180.79 uptime
root@172.31.180.79's password: 
 16:18:45 up 195 days, 14:02,  0 users,  load average: 0.09, 0.05, 0.05
```
## 挂载远程驱动器到本地

```
# sshfs root@172.31.180.79:/root/  /tmp/mnt
root@172.31.180.79's password: 
```


## 网络上发送多播消息

```
//当前登录的所有会话都会收到消息
# echo "test message"  | wall
```

## 查看网络端口
```
#lsof -i
# netstat -tnp
```


## 统计磁盘
```
# df -lh
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G   17G  2.4G  88% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  696K  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs           783M     0  783M   0% /run/user/0
tmpfs           783M     0  783M   0% /run/user/1004

# du /* -sh  
0	/bin
202M	/boot
8.0K	/data
0	/dev
38M	/etc

//显示文件的使用情况
# du * -ah
144K	build/temp.linux-x86_64-2.7/cythonfn.o
148K	build/temp.linux-x86_64-2.7
152K	build
80K	calculate.so

//命令参数的所有文件和目录的磁盘使用情况总计
# du * -c
148	build/temp.linux-x86_64-2.7
152	build
80	calculate.so
132	cythonfn.c
36	cythonfn.html

//排除某些通配项
# du --exclude *.txt *

//通配项在文件中
# du --exclude-from file.txt *

//搜索深度
# du --max-depth 2 *
```

## 计算命令执行时间
time
```
//%e是real，%U是user，%S是sys
# /usr/bin/time -f "Time:%e  %U   %S"   sleep 5
Time:5.00  0.00   0.00
```

## 当前用户启动日志启动故障相关的信息

```
# users
root root root root test

# who
root     pts/0        2019-08-19 22:03 (212.64.81.71)
root     pts/1        2019-08-21 00:17 (212.64.81.71)
root     pts/2        2019-08-29 11:14 (212.64.81.71)
root     pts/3        2019-08-30 11:12 (212.64.81.71)
test     pts/4        2019-08-30 16:25 (212.64.81.71)

# w
 16:41:53 up 195 days, 14:25,  5 users,  load average: 0.03, 0.06, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    212.64.81.71     19Aug19 29:46m  0.23s  0.21s /usr/bin/python2 /usr/bin/ipython
root     pts/1    212.64.81.71     21Aug19 18:01   0.29s  0.26s /usr/bin/python2 /usr/bin/ipython
root     pts/2    212.64.81.71     Thu11   15:05   1.51s  1.50s -bash
root     pts/3    212.64.81.71     11:12    1.00s  0.11s  0.00s w
test     pts/4    212.64.81.71     16:25   16:45   0.00s  0.00s -bash
```
last命令以/var/log/wtmp文件为输入
```
# last
test     pts/4        212.64.81.71     Fri Aug 30 16:25   still logged in   
root     pts/3        212.64.81.71     Fri Aug 30 11:12   still logged in   
root     pts/4                         Fri Aug 30 09:40 - 09:42  (00:02)    
root     pts/4                         Fri Aug 30 09:30 - 09:31  (00:01)   

# last -f /var/log/wtmp | more
test     pts/4        212.64.81.71     Fri Aug 30 16:25   still logged in   
root     pts/3        212.64.81.71     Fri Aug 30 11:12   still logged in   
root     pts/4                         Fri Aug 30 09:40 - 09:42  (00:02)    
root     pts/4                         Fri Aug 30 09:30 - 09:31  (00:01) 

# last root
test     pts/4        212.64.81.71     Fri Aug 30 16:25   still logged in   
root     pts/3        212.64.81.71     Fri Aug 30 11:12   still logged in   
root     pts/4                         Fri Aug 30 09:40 - 09:42  (00:02)    
root     pts/4                         Fri Aug 30 09:30 - 09:31  (00:01) 

//查看登录失败
# lastb
root     ssh:notty    212.64.81.71     Fri Aug 30 16:24 - 16:24  (00:00)    
test     ssh:notty    212.64.81.71     Fri Aug 30 16:24 - 16:24  (00:00)    
admin    ssh:notty    113.172.172.61   Fri Aug 30 03:31 - 03:31  (00:00)    
admin    ssh:notty    113.172.172.61   Fri Aug 30 03:31 - 03:31  (00:00)    
admin    ssh:notty    14.186.244.135   Fri Aug 30 03:31 - 03:31  (00:00)    
admin    ssh:notty    14.186.244.135   Fri Aug 30 03:31 - 03:31  (00:00)    
admin    ssh:notty    197.35.230.141   Thu Aug 29 07:07 - 07:07  (00:00) 
```

## 观察命令输出并刷新
-n 表示间隔时间；-d表示高亮显示
```
# watch -n 5 -d "ls | head -1"
```

## logrotate


## syslog
日志文件|描述
---|---
/var/log/boot.log|系统启动信息
/var/log/httpd|Apache Web服务器日志
/var/log/message|发布内核启动信息
/var/log/auth.log|用户认证日志
/var/log/dmesg|系统启动信息
/var/log/mail.log|邮件服务器日志
/var/log/Xorg.0.log|X服务器日志
```
//向/var/log/message中记录
logger "This is a Test"
```
