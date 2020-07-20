参考https://cloud.tencent.com/developer/article/1494875

https://blog.csdn.net/benpaobagzb/article/details/51044420

https://blog.csdn.net/superbfly/article/details/93118105

https://blog.csdn.net/vbaspdelphi/article/details/53725281

https://www.cnblogs.com/bhlsheji/p/5061800.html

## 查看网卡以及性能参数

#### 查看网卡

目前我们在公有云上买到的服务器，一般网卡类型都是virtio，这个可以通过ethtool -i 获取

```
# ethtool -i eth0
driver: virtio_net
version: 1.0.0
firmware-version: 
expansion-rom-version: 
bus-info: 0000:00:03.0
supports-statistics: no
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: no
```

#### 查看网卡队列数

``` 
# ethtool -l eth0
Channel parameters for eth0:
Pre-set maximums:
RX:		0
TX:		0
Other:		0
Combined:	1
Current hardware settings:
RX:		0
TX:		0
Other:		0
Combined:	1
```

该参数一般系统规格有关。队列数不少于cpu数时性能最高，但在队列数大于16之后，性能提升不会那么明显了

重新设置方法

```
# ethtool  -L  eth0   combined  1
```



#### 查看网卡队列长度

```
# ethtool -g eth0
Ring parameters for eth0:
Pre-set maximums:
RX:		256
RX Mini:	0
RX Jumbo:	0
TX:		256
Current hardware settings:
RX:		256
RX Mini:	0
RX Jumbo:	0
TX:		256
```

我们如果突发流量不大，有时也可以手动调小，这样pps会更高，缓存亲和性会更好。

如果突发流量大，那么最好保持最大值。

重新设置方法（云服务器不支持）：

```
# ethtool -G eth0 rx 512
# ethtool -G eth0 tx 512
```



#### 网卡RPS设置

```
# cat /sys/class/net/eth0/queues/rx-0/rps_cpus 
3

```

#### 网卡中断设置

查看系统上的中断是怎么分配在 CPU 上

```
# cat /proc/interrupts 
           CPU0       CPU1       
  0:         39          0   IO-APIC-edge      timer
  1:         10          0   IO-APIC-edge      i8042
  4:        248          0   IO-APIC-edge      serial
  6:          3          0   IO-APIC-edge      floppy
  8:          0          0   IO-APIC-edge      rtc0
  9:          0          0   IO-APIC-fasteoi   acpi
 11:        134          0   IO-APIC-fasteoi   uhci_hcd:usb1, virtio3
 12:         15          0   IO-APIC-edge      i8042
 14:          0          0   IO-APIC-edge      ata_piix
 15:   42154052          0   IO-APIC-edge      ata_piix
 24:          0          0   PCI-MSI-edge      virtio2-config
 25:     127093   21033281   PCI-MSI-edge      virtio2-req.0
 26:          0          0   PCI-MSI-edge      virtio1-config
 27:         14          0   PCI-MSI-edge      virtio1-virtqueues
 28:          7          0   PCI-MSI-edge      virtio0-config
 29:    1176809   54282321   PCI-MSI-edge      virtio0-input.0
 30:       7957   37167950   PCI-MSI-edge      virtio0-output.0
NMI:          0          0   Non-maskable interrupts
LOC: 1720236938 1552020569   Local timer interrupts
SPU:          0          0   Spurious interrupts
PMI:          0          0   Performance monitoring interrupts
IWI:   81611321   84475831   IRQ work interrupts
RTR:          0          0   APIC ICR read retries
RES: 1520614891 1513089982   Rescheduling interrupts
CAL:   29234740     608764   Function call interrupts
TLB:    3335323    3286296   TLB shootdowns
TRM:          0          0   Thermal event interrupts
THR:          0          0   Threshold APIC interrupts
DFR:          0          0   Deferred Error APIC interrupts
MCE:          0          0   Machine check exceptions
MCP:     143886     143886   Machine check polls
ERR:          0
MIS:          0
PIN:          0          0   Posted-interrupt notification event
NPI:          0          0   Nested posted-interrupt event
PIW:          0          0   Posted-interrupt wakeup event
```

可以看到中断为29的

```
29:    1176809   54282321   PCI-MSI-edge      virtio0-input.0
```

绑定中断到特定的CPU

```
# cat /proc/irq/29/smp_affinity
2
```

可以看到当前绑定为cpu1（即2号CPU）上，可以通过下面的方式修改

```
# echo 1 > /proc/irq/29/smp_affinity
```

在网络非常 heavy 的情况下，

1. 对于文件服务器、高流量 Web 服务器这样的应用来说，把不同的网卡 IRQ 均衡绑定到不同的 CPU 上将会减轻某个 CPU 的负担，提高多个 CPU 整体处理中断的能力;
2. 对于数据库服务器这样的应用来说，把磁盘控制器绑到一个 CPU、把网卡绑定到另一个 CPU 将会提高数据库的响应时间、优化性能。
3. 合理的根据自己的生产环境和应用的特点来平衡 IRQ 中断有助于提高系统的整体吞吐能力和性能。
   

## 内核tcp参数
内核参数名|说明
---|---
fs.file-max|最大能够打开的文件描写叙述符数量。注意是整个系统。在server中。我们知道每创建一个连接，系统就会打开一个文件描写叙述符，所以，文件描写叙述符打开的最大数量也决定了我们的最大连接数。select在高并发情况下被代替的原因也是文件描写叙述符打开的最大值，尽管它能够改动但一般不建议这么做
net.ipv4.tcp_max_syn_backlog|Tcp syn队列的最大长度，在进行系统调用connect时会发生Tcp的三次握手，server内核会为Tcp维护两个队列。Syn队列和Accept队列，Syn队列是指存放完毕第一次握手的连接。Accept队列是存放完毕整个Tcp三次握手的连接，改动net.ipv4.tcp_max_syn_backlog使之增大能够接受很多其它的网络连接。注意此參数过大可能遭遇到Syn flood攻击
net.ipv4.tcp_syncookies|改动此參数能够有效的防范syn flood攻击。原理：在Tcpserver收到Tcp Syn包并返回Tcp Syn+ack包时，不专门分配一个数据区。而是依据这个Syn包计算出一个cookie值。在收到Tcp ack包时，Tcpserver在依据那个cookie值检查这个Tcp ack包的合法性。假设合法，再分配专门的数据区进行处理未来的TCP连接。默认为0。1表示开启
net.ipv4.tcp_keepalive_time|Tcp keepalive心跳包机制。用于检測连接是否已断开。我们能够改动默认时间来间断心跳包发送的频率。keepalive通常是server对client进行发送查看client是否在线。由于server为client分配一定的资源。可是Tcp 的keepalive机制非常有争议。由于它们可耗费一定的带宽
net.ipv4.tcp_tw_reuse|大量处于time_wait状态是非常浪费资源的，它们占用server的描写叙述符等。改动此參数。同意重用处于time_wait的socket。默认为0，1表示开启
net.ipv4.tcp_tw_recycle|该參数表示高速回收处于time_wait的socket。默认为0，1表示开启
net.ipv4.tcp_fin_timeout|改动time_wait状的存在时间。默认的2MSL
net.ipv4.tcp_max_tw_buckets|所同意存在time_wait状态的最大数值，超过则立马被清除而且警告。
net.ipv4.ip_local_port_range|表示对外连接的端口范围
somaxconn|somaxconn參数决定Accept队列长度，在listen函数调用时backlog參数即决定Accept队列的长度，该參数太小也会限制最大并发连接数，由于同一时间完毕3次握手的连接数量太小，server处理连接速度也就越慢。server端调用accept函数实际上就是从已连接Accept队列中取走完毕三次握手的连接