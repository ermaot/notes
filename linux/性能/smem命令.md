## smem功能
1. 系统概况列表
1. 按进程、映射和用户列表
1. 按用户、映射或用户过滤
1. 来自多个数据源的可配置列
1. 可配置的输出单位和百分比
1. 可配置的标题和总和
1. 从/proc读取活动数据
1. 从目录镜像或经过压缩的打包文件读取数据快照
1. 面向嵌入式系统的轻型捕获工具
1. 内置的图表生成功能

## smem参数
```
# smem --help
Usage: smem [options]

Options:
  -h, --help            show this help message and exit
  -H, --no-header       disable header line
  -c COLUMNS, --columns=COLUMNS
                        columns to show
  -t, --totals          show totals
  -R REALMEM, --realmem=REALMEM
                        amount of physical RAM
  -K KERNEL, --kernel=KERNEL
                        path to kernel image
  -m, --mappings        show mappings
  -u, --users           show users
  -w, --system          show whole system
  -P PROCESSFILTER, --processfilter=PROCESSFILTER
                        process filter regex
  -M MAPFILTER, --mapfilter=MAPFILTER
                        map filter regex
  -U USERFILTER, --userfilter=USERFILTER
                        user filter regex
  -n, --numeric         numeric output
  -s SORT, --sort=SORT  field to sort on
  -r, --reverse         reverse sort
  -p, --percent         show percentage
  -k, --abbreviate      show unit suffixes
  --pie=PIE             show pie graph
  --bar=BAR             show bar graph
  -S SOURCE, --source=SOURCE
                        /proc data source
```

## smem使用样例
- 无参数时显示所有进程的内存使用情况
```
# smem
  PID User     Command                         Swap      USS      PSS      RSS 
19916 java     tail -f logs/spring.log            0      104      114      292 
  506 root     /sbin/agetty --keep-baud 11        0      128      151      504 
  505 root     /sbin/agetty --noclear tty1        0      132      156      516 
  500 root     /usr/sbin/atd -f                   0      228      252      632 
  476 root     /usr/sbin/irqbalance --fore        0      268      305      688 
  412 root     /sbin/auditd                       0      504      534      904 
```
- 参数-u 显示每个用户所耗用的内存总量
```
-r 参数可以反序
# smem -u
User     Count     Swap      USS      PSS      RSS 
ntp          1        0      908      996     1644 
dbus         1        0     1044     1199     1904 
polkitd      1        0     9260     9445    10296 
root        23        0    55464    64494    89864 
mysql        1        0   582688   583535   585836 
java         4        0  2211828  2218223  2233176 
```
- 参数-p 查看耗用内存情况的百分比
```
# smem -p
  PID User     Command                         Swap      USS      PSS      RSS 
19916 java     tail -f logs/spring.log          N/A    0.00%    0.00%    0.00% 
  506 root     /sbin/agetty --keep-baud 11      N/A    0.00%    0.00%    0.01% 
  505 root     /sbin/agetty --noclear tty1      N/A    0.00%    0.00%    0.01% 
  500 root     /usr/sbin/atd -f                 N/A    0.00%    0.00%    0.01% 
  476 root     /usr/sbin/irqbalance --fore      N/A    0.00%    0.00%    0.01% 
  412 root     /sbin/auditd                     N/A    0.01%    0.01%    0.01% 
21433 root     /usr/sbin/lvmetad -f             N/A    0.01%    0.01%    0.02% 
```
- 参数-up合用
```
# smem -up
User     Count     Swap      USS      PSS      RSS 
ntp          1      N/A    0.01%    0.01%    0.02% 
dbus         1      N/A    0.01%    0.01%    0.02% 
polkitd      1      N/A    0.12%    0.12%    0.13% 
root        23      N/A    0.69%    0.81%    1.12% 
mysql        1      N/A    7.27%    7.29%    7.31% 
java         4      N/A   27.61%   27.69%   27.88% 
```
- 参数-w 查看系统内存使用情况
```
# smem -w
Area                           Used      Cache   Noncache 
firmware/hardware                 0          0          0 
kernel image                      0          0          0 
kernel dynamic memory       4963556    4858832     104724 
userspace memory            2874360      61192    2813168 
free memory                  171780     171780          0 
```
- 参数-c 用来显示需要展示的列
```
# smem -c "name user pss"
Name                     User          PSS 
tail                     java          114 
agetty                   root          151 
agetty                   root          156 
atd                      root          252 
irqbalance               root          305 
auditd                   root          534 
lvmetad                  root          621 
crond                    root          769 
```
- 参数-s 根据某一列（例如 rss）来排序
```
# smem -r -s pss
  PID User     Command                         Swap      USS      PSS      RSS 
 4032 java     java -jar cf81-commerce-bas        0   943884   946043   951000 
 3936 java     java -jar cf81-commerce-bas        0   695472   697628   702580 
18668 mysql    /root/mysql_debug/bin/mysql        0   582688   583535   585836 
24539 java     java -jar cf81-commerce-bac        0   572368   574438   579304 
  722 root     /usr/bin/python -Es /usr/sb        0    11120    11513    12504 
  471 polkitd  /usr/lib/polkit-1/polkitd -        0     9260     9445    10296 
15151 root     python /usr/bin/smem -r -s         0     7864     8095     9028 
```
- 参数-M 过滤相关进程（这个参数似乎结果不太对，使用了-M与不使用结果相差太远）
```
# smem -M java
  PID User     Command                         Swap      USS      PSS      RSS 
24539 java     java -jar cf81-commerce-bac        0      976     2889     6716 
 3936 java     java -jar cf81-commerce-bas        0      984     2983     6896 
 4032 java     java -jar cf81-commerce-bas        0     1008     3009     6924 
```
- 参数-U按照用户
```
# smem -U root -r
  PID User     Command                         Swap      USS      PSS      RSS 
  722 root     /usr/bin/python -Es /usr/sb        0    11120    11513    12504 
 5168 root     /usr/local/aegis/aegis_clie        0     7856     8085     9092 
15360 root     python /usr/bin/smem -U roo        0     6768     7006     8012 
  336 root     /usr/lib/systemd/systemd-jo        0     4032     5412     7268 
  723 root     /usr/sbin/rsyslogd -n              0     3972     5362     7312 
16857 root     mysql -u root --port 33060         0     3260     4048     6192 
```
## 使用smem图形化显示内存使用情况
- 基于PSS/RSS值，生成直方图
```
smem --bar name -c "pss uss" -U alice
```
- 生成饼图
```
smem --pie name -c "pss"
```
## 按照某个指标排序
```
# smem -s --help
unknown field --help
known fields:
command  process command line
maps     total number of mappings
name     name of process
pid      process ID
pss      proportional set size (including sharing)
rss      resident set size (ignoring sharing)
swap     amount of swap space consumed (ignoring sharing)
user     owner of process
uss      unique set size
vss      virtual set size (total virtual memory mapped)
```
- 按照swap使用排序
```
# smem -s swap
  PID User     Command                         Swap      USS      PSS      RSS 
    1 root     /usr/lib/systemd/systemd --        0     2748     2885     3836 
  336 root     /usr/lib/systemd/systemd-jo        0     4040     5420     7276 
  412 root     /sbin/auditd                       0      504      534      904 
  471 polkitd  /usr/lib/polkit-1/polkitd -        0     9260     9445    10296 
  472 dbus     /usr/bin/dbus-daemon --syst        0     1044     1199     1904 
  473 ntp      /usr/sbin/ntpd -u ntp:ntp -        0      908      996     1644 
```
- 按照uss排序
```
# smem -s uss -r
  PID User     Command                         Swap      USS      PSS      RSS 
 4032 java     java -jar cf81-commerce-bas        0   943884   946043   951000 
 3936 java     java -jar cf81-commerce-bas        0   695472   697628   702580 
18668 mysql    /root/mysql_debug/bin/mysql        0   582688   583535   585836 
24539 java     java -jar cf81-commerce-bac        0   572372   574442   579308 
  722 root     /usr/bin/python -Es /usr/sb        0    11120    11513    12504 
  471 polkitd  /usr/lib/polkit-1/polkitd -        0     9260     9445    10296 
15620 root     python /usr/bin/smem -s uss        0     7864     8095     902
```
