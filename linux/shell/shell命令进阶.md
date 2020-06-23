## paste
合并文件中的行
```
//合并两个文件，用|做分隔符
#paste -d '|' p1.txt  p2.txt 
1|21
2|22
3|23
4|24
5|25
6|26
|27
|28
|29


//合并多个文件
#paste -d '|,' p1.txt  p2.txt  p2.txt
1|21,21
2|22,22
3|23,23
4|24,24
5|25,25
6|26,26
|27,27
|28,28
|29,29

//-s参数
#paste  -s -d "|" p1.txt p2.txt p2.txt
1|2|3|4|5|6
21|22|23|24|25|26|27|28|29
21|22|23|24|25|26|27|28|29

//一列转化成两列
#paste - - <p1.txt
1	2
3	4
5	6

#paste - - -d :<p1.txt
1:2
3:4
5:6

//转化成3列
#paste - - - -d :<p1.txt
1:2:3
4:5:6
[root@izm5edbv563hlvcbf71ophz dir1]#
```
## dd

## gzip 、bzip2

```
//压缩，但不保留原始文件
#gzip p1.txt 
#ls p1.txt*
p1.txt.gz

#gzip -c p2.txt  > p2.txt.gz

//递归压缩
# gzip -r

//指定压缩级别（级别可为1~9）
#gzip -3

//解压
#gzip -d

//bzip压缩并保留原始文件
# bzip -k

//解压缩，-f代表强制覆盖
# bzip -df
```

## tar

```
-c：创建一个新的归档，即压缩
-v：显示详情
-f：指定归档文件名
-z：指定gzip压缩
-j：指定bzip压缩
--wildcards：指定压缩文件名的模式
tar -xvf test.tar  --wildcards '*.jpg'

-t：不解压而显示文件列表
-tjvf：可以在不解压包的情况下显示bzip2压缩的tar包文件列表
-tzvf：可以在不解压包的情况下显示gzip压缩的tar包文件列表
-rvf：可以添加文件或者目录到一个已存在的tar包

-W：何时tar包内容
#tar -cvf test.tar test*.txt   
test2.txt
test3.txt
test.txt
#tar -tvf test.tar 
-rw-r--r-- root/root     20480 2019-08-27 10:56 test2.txt
-rw-r--r-- root/root        38 2019-08-26 17:13 test3.txt
-rw-r--r-- root/root     10240 2019-08-27 10:56 test.txt
```
## mount和umount

```
//显示所有挂载
mount

//显示指定文件系统的挂载
mount -t ext4

//挂载/etc/fstab下的所有配置
mount -a

//重新以只读方式挂载
mount -t nfs -o remount,ro nasstore:/vol/volume_share/share


//要当前的挂载点没有被使用，可以用fuser或者lsof查看是否被使用
umount  nasstore:/vol/volume_share/share

```
## df

```
//不加参数
#df
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/vda1       20510332 16884244   2561180  87% /
devtmpfs         3994256        0   3994256   0% /dev
tmpfs            4004848        0   4004848   0% /dev/shm
tmpfs            4004848      668   4004180   1% /run
tmpfs            4004848        0   4004848   0% /sys/fs/cgroup
tmpfs             800972        0    800972   0% /run/user/0
//显示全部的文件系统信息（没有粘贴全部结果）
#df -a
Filesystem     1K-blocks     Used Available Use% Mounted on
rootfs                 -        -         -    - /
sysfs                  0        0         0    - /sys
proc                   0        0         0    - /proc
devtmpfs         3994256        0   3994256   0% /dev
securityfs             0        0         0    - /sys/kernel/security
tmpfs            4004848        0   4004848   0% /dev/shm
devpts                 0        0         0    - /dev/pts
//以人类可读的格式显示
#df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G   17G  2.5G  87% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  668K  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs           783M     0  783M   0% /run/user/0

//显示文件系统类型信息
#df -T
Filesystem     Type     1K-blocks     Used Available Use% Mounted on
/dev/vda1      ext4      20510332 16884312   2561112  87% /
devtmpfs       devtmpfs   3994256        0   3994256   0% /dev
tmpfs          tmpfs      4004848        0   4004848   0% /dev/shm
tmpfs          tmpfs      4004848      668   4004180   1% /run
tmpfs          tmpfs      4004848        0   4004848   0% /sys/fs/cgroup
tmpfs          tmpfs       800972        0    800972   0% /run/user/0

//指定显示特定的文件系统
#df -t ext4
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/vda1       20510332 16884344   2561080  87% /

//不显示特定的文件系统
#df -x tmpfs
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/vda1       20510332 16884380   2561044  87% /
devtmpfs         3994256        0   3994256   0% /dev

//
#df -m
Filesystem     1M-blocks  Used Available Use% Mounted on
/dev/vda1          20030 16489      2501  87% /
devtmpfs            3901     0      3901   0% /dev
tmpfs               3911     0      3911   0% /dev/shm
tmpfs               3911     1      3911   1% /run
tmpfs               3911     0      3911   0% /sys/fs/cgroup
tmpfs                783     0       783   0% /run/user/0
```
## du

```
//du
//du -a递归显示文件夹
//du -h 人类可读的方式
//du -s 仅显示当前目录
//--exclude 排除某种模式的文件
#du -a --exclude "*.txt*"
//--time 显示修改时间
```

## crontab  cron

## at

## & 

```
jobs 查看后台运行任务的命令

fg <id> 移动到前台

ctrl + Z挂起任务，然后bg 或者 %1 &放到后台
```

## nohup