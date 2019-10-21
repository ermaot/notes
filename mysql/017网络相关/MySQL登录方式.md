MySQL连接方式到底有几种呢？到MySQL5.7为止，总共有五种，分别是TCP/IP，TLS/SSL，Unix Sockets，Shared Memory，Named pipes，下面我们就来看看这五种的区别：

方式|默认开启|支持系统|只支持本机|如何开启|参数配置
---|---|---|---|---|---
TCP/IP|	是|	所有系统|	否|	--skip-networking=yes/no.|	--port --bind-address
TLS/SSL	|是	|所有系统（基于TCP/IP)之上	|否	|--ssl=yes/no.|	--ssl-* options
Unix Sockets|	是	|类Unix系统|	是|	设置--socket=<empty> 来关闭|--socket=socket path
Shared Memory|	否|	Windows系统|	是|	--shared-memory=on/off|--shared-memory-base-name=<name>
Named pipes|	否|	Windows系统	|否	|--enable-named-pipe=on/off|	--socket=<name>

## Unix Sockets：
mysql -uroot
若你在本机使用这种方式连接MySQL数据库的话，它默认会使用Unix Sockets。

## TCP/IP：

```
mysql --protocol=tcp -uroot
mysql -P3306 -h127.0.0.1 -uroot
```

连接的时候我们指定连接协议，或者指定相应的IP及端口，我们的连接方式就变成了TCP/IP方式。

## TLS/SSL：

```
mysql --protocol=tcp -uroot --ssl=on
mysql -P3306 -h127.0.0.1 -uroot --ssl=on
```

上表说过，TLS/SSL是基于TCP/IP的，所以我们只需再指定打开ssl配置即可。

然后我们可以通过以下语句来查询目前数据库的连接情况：


```
SELECT DISTINCT connection_type from performance_schema.threads where connection_type is not null
```

## Shared Memory

```
服务器端配置 : shared_memory=ON share_memory_base_name=mysql 
mysql --protocol=memory --shared-memory-base-name=mysql
```

## named pipes

```
服务器端配置   --enable-named-pipe --socket=SOCKET
mysql --protocol=PIPE --socket=SOCKET
```

那么我们如何选择连接方式呢？

1. 若是你能确定程序和数据库在同一台机子(类Unix系统)上，推荐使用Unix Sockets，因为它效率更高；
2. 若数据库分布在不同的机子上，且能确保连接安全或者安全性要求不是那么高，推荐使用TCP/IP，反之使用TLS/SSL