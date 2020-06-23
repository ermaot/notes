## 网络模型
#### OSI
国际标准化组织制定的开放式系统互联通信参考模型（Open System Interconnection Reference Model），简称为 OSI 网络模型。
1. 应用层，负责为应用程序提供统一的接口。
2. 表示层，负责把数据转换成兼容接收系统的格式。
3. 会话层，负责维护计算机之间的通信连接。
4. 传输层，负责为数据加上传输表头，形成数据包。
5. 网络层，负责数据的路由和转发。
6. 数据链路层，负责 MAC 寻址、错误侦测和改错。
7. 物理层，负责在物理网络中传输数据帧。
#### tcp/ip
1. 应用层，负责向用户提供一组应用程序，比如 HTTP、FTP、DNS 等。
2. 传输层，负责端到端的通信，比如 TCP、UDP 等。
3. 网络层，负责网络包的封装、寻址和路由，比如 IP、ICMP 等。
4. 网络接口层，负责网络包在物理网络中的传输，比如 MAC 寻址、错误侦测以及通过网卡传输网络帧等。

#### 两种模型相互比较
![linux网络基础1.png](https://github.com/ermaot/notes/blob/master/linux/%E6%80%A7%E8%83%BD/pic/linux%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%801.png?raw=true)

## linux网络栈
![image](https://github.com/ermaot/notes/blob/master/linux/%E6%80%A7%E8%83%BD/pic/linux%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%802.png)

- 网络接口配置的最大传输单元（MTU），就规定了最大的 IP 包大小。在我们最常用的以太网中，MTU 默认值是 1500（这也是 Linux 的默认值）。
- 一旦网络包超过MTU的大小，就会在网络层分片。MTU 越大，需要的分包也就越少，网络吞吐能力就越好。
- Linux 通用 IP 网络栈
![image](https://github.com/ermaot/notes/blob/master/linux/%E6%80%A7%E8%83%BD/pic/linux%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%803.png)
## Linux 网络收发流程
#### 网络包的接收流程
- 当一个网络帧到达网卡后，网卡会通过 DMA 方式，把这个网络包放到收包队列中；然后通过硬中断，告诉中断处理程序已经收到了网络包。
- 接着，网卡中断处理程序会为网络帧分配内核数据结构（sk_buff），并将其拷贝到 sk_buff 缓冲区中；然后再通过软中断，通知内核收到了新的网络帧。
- 接下来，内核协议栈从缓冲区中取出网络帧，并通过网络协议栈，从下到上逐层处理这个网络帧。
1. 在链路层检查报文的合法性，找出上层协议的类型（比如 IPv4 还是 IPv6），再去掉帧头、帧尾，然后交给网络层。
2. 网络层取出 IP 头，判断网络包下一步的走向，比如是交给上层处理还是转发。当网络层确认这个包是要发送到本机后，就会取出上层协议的类型（比如 TCP 还是 UDP），去掉 IP 头，再交给传输层处理。
3. 传输层取出 TCP 头或者 UDP 头后，根据 < 源 IP、源端口、目的 IP、目的端口 > 四元组作为标识，找出对应的 Socket，并把数据拷贝到 Socket 的接收缓存中。
- 最后，应用程序就可以使用 Socket 接口，读取到新接收到的数据了
![image](https://github.com/ermaot/notes/blob/master/linux/%E6%80%A7%E8%83%BD/pic/linux%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%804.png)

#### 网络包的发送流程
- 首先，应用程序调用 Socket API（比如 sendmsg）发送网络包。这是一个系统调用，所以会陷入到内核态的套接字层中。套接字层会把数据包放到 Socket 发送缓冲区中。
- 接下来，网络协议栈从 Socket 发送缓冲区中，取出数据包；再按照 TCP/IP 栈，从上到下逐层处理。比如，传输层和网络层，分别为其增加 TCP 头和 IP 头，执行路由查找确认下一跳的 IP，并按照 MTU 大小进行分片。
- 分片后的网络包，再送到网络接口层，进行物理地址寻址，以找到下一跳的 MAC 地址。然后添加帧头和帧尾，放到发包队列中。这一切完成后，会有软中断通知驱动程序：发包队列中有新的网络帧需要发送。
- 最后，驱动程序通过 DMA ，从发包队列中读出网络帧，并通过物理网卡把它发送出去。

## 网络性能指标
我们通常用带宽、吞吐量、延时、PPS（Packet Per Second）等指标衡量网络的性能。
- 带宽：表示链路的最大传输速率，单位通常为 b/s （比特 / 秒）。
- 吞吐量：表示单位时间内成功传输的数据量，单位通常为 b/s（比特 / 秒）或者 B/s（字节 / 秒）。吞吐量受带宽限制，而吞吐量 / 带宽，也就是该网络的使用率。
- 延时：表示从网络请求发出后，一直到收到远端响应，所需要的时间延迟。在不同场景中，这一指标可能会有不同含义。比如，它可以表示，建立连接需要的时间（比如 TCP 握手延时），或一个数据包往返所需的时间（比如 RTT）。
- PPS：是 Packet Per Second（包 / 秒）的缩写，表示以网络包为单位的传输速率。PPS 通常用来评估网络的转发能力，比如硬件交换机，通常可以达到线性转发（即 PPS 可以达到或者接近理论最大值）。而基于 Linux 服务器的转发，则容易受网络包大小的影响。

除了这些指标，网络的可用性（网络能否正常通信）、并发连接数（TCP 连接数量）、丢包率（丢包百分比）、重传率（重新传输的网络包比例）等也是常用的性能指标。

#### 网络配置
ifconfig和ip命令
```
ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
      inet 10.240.0.30 netmask 255.240.0.0 broadcast 10.255.255.255
      inet6 fe80::20d:3aff:fe07:cf2a prefixlen 64 scopeid 0x20<link>
      ether 78:0d:3a:07:cf:3a txqueuelen 1000 (Ethernet)
      RX packets 40809142 bytes 9542369803 (9.5 GB)
      RX errors 0 dropped 0 overruns 0 frame 0
      TX packets 32637401 bytes 4815573306 (4.8 GB)
      TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
​
$ ip -s addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
  link/ether 78:0d:3a:07:cf:3a brd ff:ff:ff:ff:ff:ff
  inet 10.240.0.30/12 brd 10.255.255.255 scope global eth0
      valid_lft forever preferred_lft forever
  inet6 fe80::20d:3aff:fe07:cf2a/64 scope link
      valid_lft forever preferred_lft forever
  RX: bytes packets errors dropped overrun mcast
   9542432350 40809397 0       0       0       193
  TX: bytes packets errors dropped carrier collsns
   4815625265 32637658 0       0       0       0

```
- 第一，网络接口的状态标志。ifconfig 输出中的 RUNNING ，或 ip 输出中的 LOWER_UP ，都表示物理网络是连通的，即网卡已经连接到了交换机或者路由器中。如果你看不到它们，通常表示网线被拔掉了。
- 第二，MTU 的大小。MTU 默认大小是 1500，根据网络架构的不同（比如是否使用了 VXLAN 等叠加网络），你可能需要调大或者调小 MTU 的数值。
- 第三，网络接口的 IP 地址、子网以及 MAC 地址。这些都是保障网络功能正常工作所必需的，你需要确保配置正确。
- 第四，网络收发的字节数、包数、错误数以及丢包情况，特别是 TX 和 RX 部分的 errors、dropped、overruns、carrier 以及 collisions 等指标不为 0 时，通常表示出现了网络 I/O 问题。其中：
1. errors 表示发生错误的数据包数，比如校验错误、帧同步错误等；
2. dropped 表示丢弃的数据包数，即数据包已经收到了
3. Ring Buffer，但因为内存不足等原因丢包；
4. overruns 表示超限数据包数，即网络 I/O 速度过快，导致 Ring Buffer 中的数据包来不及处理（队列满）而导致的丢包；
5. carrier 表示发生 carrirer 错误的数据包数，比如双工模式不匹配、物理电缆出现问题等；
6. collisions 表示碰撞数据包数。

#### netstat 和 ss
```
# netstat -s
Ip:
    45463260 total packets received
    0 forwarded
    0 incoming packets discarded
    45463260 incoming packets delivered
    56637298 requests sent out
    324 dropped because of missing route
Icmp:
    29887 ICMP messages received
    4932 input ICMP message failed.
    InCsumErrors: 6
    ICMP input histogram:
        destination unreachable: 5804
        timeout in transit: 65
        echo requests: 23675
        echo replies: 249
        timestamp request: 88
    29897 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 4
        echo request: 6130
        echo replies: 23675
        timestamp replies: 88
IcmpMsg:
        InType0: 249
        InType3: 5804
        InType8: 23675
        InType11: 65
        InType13: 88
        OutType0: 23675
        OutType3: 4
        OutType8: 6130
        OutType14: 88
Tcp:
    3042129 active connections openings
    98688 passive connection openings
    414521 failed connection attempts
    6403 connection resets received
    29 connections established
    43618656 segments received
    53228245 segments send out
    1580601 segments retransmited
    2920 bad segments received.
    554407 resets sent
    InCsumErrors: 2911
Udp:
    1814713 packets received
    4 packets to unknown port received.
    0 packet receive errors
    2122158 packets sent
    0 receive buffer errors
    0 send buffer errors
UdpLite:
TcpExt:
    173429 invalid SYN cookies received
    413731 resets received for embryonic SYN_RECV sockets
    227 packets pruned from receive queue because of socket buffer overrun
    9 ICMP packets dropped because they were out-of-window
    2332149 TCP sockets finished time wait in fast timer
    32 packets rejects in established connections because of timestamp
    3464713 delayed acks sent
    442 delayed acks further delayed because of locked socket
    Quick ack mode was activated 6592 times
    246 SYNs to LISTEN sockets dropped
    5785387 packets directly queued to recvmsg prequeue.
    4548430 bytes directly in process context from backlog
    39260414 bytes directly received in process context from prequeue
    11389578 packet headers predicted
    11143 packets header predicted and directly queued to user
    4171355 acknowledgments not containing data payload received
    13461468 predicted acknowledgments
    218 times recovered from packet loss by selective acknowledgements
    Detected reordering 1 times using FACK
    Detected reordering 13 times using time stamp
    21 congestion windows fully recovered without slow start
    13 congestion windows partially recovered using Hoe heuristic
    98 congestion windows recovered without slow start by DSACK
    15937 congestion windows recovered without slow start after partial ack
    TCPLostRetransmit: 146
    153 timeouts after SACK recovery
    16 timeouts in loss state
    770 fast retransmits
    23 forward retransmits
    11 retransmits in slow start
    861215 other TCP timeouts
    TCPLossProbes: 22902
    TCPLossProbeRecovery: 2907
    22 SACK retransmits failed
    5600 DSACKs sent for old packets
    48 DSACKs sent for out of order packets
    37131 DSACKs received
    116 connections reset due to unexpected data
    1614 connections reset due to early user close
    199 connections aborted due to timeout
    TCPDSACKIgnoredOld: 1
    TCPDSACKIgnoredNoUndo: 4802
    TCPSpuriousRTOs: 15
    TCPSackShifted: 57
    TCPSackMerged: 438
    TCPSackShiftFallback: 5984
    TCPRcvCoalesce: 414355
    TCPOFOQueue: 62052
    TCPOFOMerge: 55
    TCPChallengeACK: 4220
    TCPSYNChallenge: 55478
    TCPFromZeroWindowAdv: 21
    TCPToZeroWindowAdv: 21
    TCPWantZeroWindowAdv: 349
    TCPSynRetrans: 1533916
    TCPOrigDataSent: 24805079
    TCPHystartTrainDetect: 159
    TCPHystartTrainCwnd: 7130
    TCPHystartDelayDetect: 1
    TCPHystartDelayCwnd: 48
    TCPACKSkippedSynRecv: 35
    TCPACKSkippedPAWS: 2
    TCPACKSkippedSeq: 9
    TCPACKSkippedChallenge: 51790
IpExt:
    InOctets: 12315577006
    OutOctets: 6448112060
    InNoECTPkts: 50192886
    InECT1Pkts: 14
    InECT0Pkts: 8924
    InCEPkts: 132
```

```
# ss -s
Total: 153 (kernel 204)
TCP:   39 (estab 29, closed 2, orphaned 0, synrecv 0, timewait 2/0), ports 0

Transport Total     IP        IPv6
*	  204       -         -        
RAW	  0         0         0        
UDP	  4         3         1        
TCP	  37        35        2        
INET	  41        38        3        
FRAG	  0         0         0        
```
#### 网络吞吐和 PPS

```
# 数字 1 表示每隔 1 秒输出一组数据
$ sar -n DEV 1
Linux 4.15.0-1035-azure (ubuntu) 	01/06/19 	_x86_64_	(2 CPU)

13:21:40        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
13:21:41         eth0     18.00     20.00      5.79      4.25      0.00      0.00      0.00      0.00
13:21:41      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
13:21:41           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

```
- rxpck/s 和 txpck/s 分别是接收和发送的 PPS，单位为包 / 秒。
- rxkB/s 和 txkB/s 分别是接收和发送的吞吐量，单位是 KB/ 秒。
- rxcmp/s 和 txcmp/s 分别是接收和发送的压缩数据包数，单位是包 / 秒。
- %ifutil 是网络接口的使用率，即半双工模式下为 (rxkB/s+txkB/s)/Bandwidth，而全双工模式下为 max(rxkB/s, txkB/s)/Bandwidth。
- Bandwidth 可以用 ethtool 来查询
```
ethtool eth0 | grep Speed
	Speed: 1000Mb/s

```
