# 如何在Linux中禁用Nagle的算法？

echo 1 > /proc/sys/net/ipv4/tcp_low_latency


本文链接：https://www.codercto.com/a/51306.html





一些设置

```
echo 0.2 > /proc/sys/net/ipv4/tcp_ack_timeout 
echo 0.2 > /proc/sys/net/ipv4/tcp_syn_timeout 
echo 0.2 > /proc/sys/net/ipv4/tcp_synack_timeout 
echo 0 > /proc/sys/net/ipv4/tcp_rst_timeout 
echo 0.2.5 > /proc/sys/net/ipv4/tcp_fin_timeout 
echo 0.2 > /proc/sys/net/ipv4/tcp_urg_timeout 
echo 0.2 > /proc/sys/net/ipv4/tcp_psh_timeout 
echo 10000 > /proc/sys/net/ipv4/tcp_syn_retries 
echo 10000 > /proc/sys/net/ipv4/tcp_synack_retries 
echo 0 > /proc/sys/net/ipv4/tcp_rst_retries 
echo 10000 > /proc/sys/net/ipv4/tcp_psh_retries 
echo 10000 > /proc/sys/net/ipv4/tcp_urg_retries 
echo 10000 > /proc/sys/net/ipv4/tcp_ack_retries 
echo 10000 > /proc/sys/net/ipv4/tcp_fin_retries 
echo 0 > /proc/sys/net/ipv4/tcp_timestam
```

http://blog.chinaunix.net/uid-20639775-id-3529535.html



```  
net.ipv4.tcp_sack = 1
```

开启sack





修改MTU

#### 1、ifconfig命令修改 

```bsh
ifconfig ${Interface} mtu ${SIZE} up
ifconfig eth1 mtu 9000 up
```

这个是最通用的方法，对所有的linux 发行版本都有效。缺点就是重启后失效，需要在开机项中加载。

#### 2、修改配置文件 

CentOS / RHEL / Fedora Linux下

```bsh
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
#增加如下内容
MTU="9000"
#保存后重启网卡生效
# service network restart
#启用IPv6地址的，修改IPv6 mtu的参数为
IPV6_MTU="1280"
```



Debian / Ubuntu Linux下

```bsh
# vi /etc/network/interfaces
#增加如下值
mtu 9000
#保存后，重启网络生效
# /etc/init.d/networking restart
```