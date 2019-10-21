## ip命令之对应
![image](https://github.com/ermaot/notes/blob/master/linux/%E6%80%A7%E8%83%BD/pic/ip%E5%91%BD%E4%BB%A4.png)
## 介绍
#### 对象
```
link 网络设备
address 设备上的协议（IP或IPv6）地址
addrlabel 协议地址选择的标签配置
neighbour ARP或NDISC缓存条目
route 路由表条目
rule 路由策略数据库中的规则
maddress 组播地址
mroute 组播路由缓存条目
tunnel IP隧道
xfrm IPSec协议框架
```
==所有对象的名称可以用完整或缩写形式书写，例如address可以缩写成addr或只是a==
#### 选项
```
-V，-Version 显示指令版本信息
-s,-stats,statistics 输出详细信息
-h,-human,-human-readable 输出人类可读的统计信息和后缀
-iec 以IEC标准单位打印人类可读速率（例如1K=1024）
-f,-family <FAMILY> 指定要使用的协议族。协议族标识可以是inet、inet6、ipx、dnet或link之一。如果此选项不存在，则从其他参数中推测协议族。如果命令行的其余部分没有提供足够的信息来推测该族，则ip会退回到默认值，通常是inet或any。link是一个特殊的系列标识符，表示不涉及网络协议。
-4 –family inet的快捷方式
-6 –family inet6的快捷方式
-0 –family link的快捷方式
-o,-oneline 将每条记录输出到一行，用’\’字符替换换行符。
-r,-resolve 使用系统名称解析程序来打印DNS名称而不是主机地址。
```


## 使用样例
#### 设置和删除Ip地址
```
# ip addr add 192.168.17.30/24 dev eth0
```
#### 显示全部IP
```
# ip addr
# ip a/addr/address
# ip a/addr/address sh/show
```
#### 显示指定设备IP
```
# ip a/addr/address sh/show dev eth1
# ip a/addr/address sh/show eth1

# ip a sh eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:04:87:43 brd ff:ff:ff:ff:ff:ff
    inet 172.31.180.74/20 brd 172.31.191.255 scope global eth0
       valid_lft forever preferred_lft forever
```
#### 增加或删除IP地址
```
# ip a/addr/address add 192.168.78.130/24 dev eth1

# ip a/addr/address del/delete 192.168.78.130/24 dev eth1
```
#### 删除eth1所有IP地址
```
# ip a flush dev eth1
```
#### 删除eth1的所有IPv4的IP地址
```
# ip -4 a flush dev eth1
```
#### 查看网络设备信息
```
# ip link sh/show/l/list/ls
# ip link sh/show/l/ls/lsit eth1
# ip link sh/show/l/ls/list dev eth1


# ip l show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:04:87:43 brd ff:ff:ff:ff:ff:ff
```
#### 停止与激活网络设备
```
# ip link set dev eth1 down
# ip link set dev eth1 up
```
#### 查看路由表
```
# ip r/ro/route
# ip r/ro/route sh/show
# ip r/ro/route sh/show dev eth1


# ip route  show dev eth0
default via 172.31.191.253 
169.254.0.0/16 scope link metric 1002 
172.31.176.0/20 proto kernel scope link src 172.31.180.74 

# ip r show 
default via 172.31.191.253 dev eth0 
169.254.0.0/16 dev eth0 scope link metric 1002 
172.31.176.0/20 dev eth0 proto kernel scope link src 172.31.180.74 
```
#### 添加或删除路由
```
# ip r/ro/route add 192.168.79.0/24 dev eth1
# ip r/ro/route d/del/delete 192.168.79.0/24
# ip r/ro/route d/del/delete 192.168.79.0/24 dev eth1

```
#### 默认路由的删除、添加与修改
```
# ip r/ro/route d/del/delete default

# ip r/ro/route add default via 192.168.78.1

# ip r/ro/route chg/change default via 192.168.78.2


```
#### 查看ARP表
```
# ip n/neigh/neighbuor sh/show

```

```
# ip n
172.31.191.253 dev eth0 lladdr ee:ff:ff:ff:ff:ff REACHABLE
172.31.180.77 dev eth0 lladdr ee:ff:ff:ff:ff:ff STALE

# ip n show
172.31.191.253 dev eth0 lladdr ee:ff:ff:ff:ff:ff REACHABLE
172.31.180.77 dev eth0 lladdr ee:ff:ff:ff:ff:ff STALE

# ip n show dev eth0
172.31.191.253 lladdr ee:ff:ff:ff:ff:ff REACHABLE
172.31.180.77 lladdr ee:ff:ff:ff:ff:ff STALE
```
#### 显示不同网络接口的统计数据
```
# ip -s link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    RX: bytes  packets  errors  dropped overrun mcast   
    4996       38       0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    4996       38       0       0       0       0       
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:04:87:43 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    13285419487 50822591 0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    7266892928 57256966 0       0       0       0       
# ip -s link show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:04:87:43 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    13285423612 50822638 0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    7266897562 57257006 0       0       0       0       
```
详细信息
```
# ip -s -s link ls eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:04:87:43 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    13285427672 50822684 0       0       0       0       
    RX errors: length   crc     frame   fifo    missed
               0        0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    7266903283 57257069 0       0       0       0       
    TX errors: aborted  fifo   window heartbeat transns
               0        0       0       0       2       
```
#### 监控netlink消息
```

```
