## 显示过滤器表达示及其书写规律
一条基本的表达式由过滤项、过滤关系、过滤值三项组成。比如ip.addr == 192.168.1.1
#### 过滤项
wireshark的过滤项是“协议“+”.“+”协议字段”的模式，以端口为例，端口出现于tcp协议中所以有端口这个过滤项且其写法就是tcp.port。其他协议，如eth、ip、udp、http、telnet、ftp、icmp、snmp等等其他协议都是如此
#### 过滤关系
过滤关系就是大于、小于、等于等几种等式关系
![image](07BA42F68A3A4280BC5BC8B05A1259E0)

#### 复合过滤表达式
![image](2CFC0C0D54C9412C868A970AD27ECCA6)


#### 常见用显示过滤需求及其对应表达式
- 数据链路层：


```
//筛选mac地址为04:f9:38:ad:13:26的数据包
eth.src == 04:f9:38:ad:13:26

//筛选源mac地址为04:f9:38:ad:13:26的数据包
eth.src == 04:f9:38:ad:13:26
```


- 网络层：


```
//筛选ip地址为192.168.1.1的数据包
ip.addr == 192.168.1.1

//筛选192.168.1.0网段的数据
ip contains "192.168.1"

//筛选192.168.1.1和192.168.1.2之间的数据包
ip.addr == 192.168.1.1 && ip.addr == 192.168.1.2

//筛选从192.168.1.1到192.168.1.2的数据包
ip.src == 192.168.1.1 && ip.dst == 192.168.1.2
```


- 传输层：


```
//筛选tcp协议的数据包
tcp

//筛选除tcp协议以外的数据包
!tcp

//筛选端口为80的数据包
tcp.port == 80

//筛选12345端口和80端口之间的数据包
tcp.port == 12345 && tcp.port == 80

//筛选从12345端口到80端口的数据包
tcp.srcport == 12345 && tcp.dstport == 80
```


- 应用层：


```
特别说明----http中http.request表示请求头中的第一行（如GET index.jsp HTTP/1.1），http.response表示响应头中的第一行（如HTTP/1.1 200 OK），其他头部都用http.header_name形式。

//筛选url中包含.php的http数据包
http.request.uri contains ".php"

//筛选内容包含username的http数据包
http contains "username"
```
