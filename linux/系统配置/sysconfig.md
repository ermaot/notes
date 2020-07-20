# Linux /etc/sysconfig的系列配置

本文参考：https://blog.csdn.net/weixin_41632560/article/details/82899626
https://blog.csdn.net/The__Apollo/article/details/60143553

序号|文件|说明
---|---|---
1|/etc/sysconfig/amd|此文件的内容是为启用amd守护进程提供它的各种参数，这些参数允许此进程自动挂载或卸载文件系统
2|/etc/sysconfig/apmd|高级电源管理所使用的配置文件
3|/etc/sysconfig/arpwatch|此文件为arpwatch守护进程的配置文件，在系统启动时arpwatch守护进程会使用此文件中的某些区段。arpwatch进程主要用来维护一张MAC地址与IP地址的对应表。
4|/etc/sysconfig/authconfig|此文件中的内容为主机在进行认证时使用。在此文件中可能有如下所示的配置项：USEMD5=<value>：<value＞可以是如下值中的一个：yes-使用MD5方式认证；no-不使用MD5方式认证。USEKERBEROS=<value>:<value>可以是如下值中的一个：yes-使用kerberos认证 no-不使用kerberos认证 USELDAPAUTH=<value>:<value>可以是如下值中的一个：  yes-使用LDAP认证 no-不使用LDAP认证
5|/etc/sysconfig/autofs|此文件用来定义自动挂载设备时使用的选项，包括NFS文件系统、CD－ROMS、DISKTTES或其它媒体，文件内容如下所列：<br>LOCALOPTIONS＝"<value>":<value>为字符串格式，用来定义自动挂载规则，缺省为空。DAEMONOPTIONS＝"<value>":<value>值用来设置卸载设备前的时间长度（以秒为单位），缺省为60S。<br>UNDERSCORETODOT=<value>:<value>为二进制会下值，用来控制是否将下载线转换为点，例如：auto_home转换为auto.home,缺省为1，即允许。<br>DISABLE＿DIRECT＝<value>：<value>值为二进制，用来设置是否可以禁止直接连接支持，缺省为1，即允许
6|/etc/sysconfig/clock|此文件用来控制解释从系统硬件时钟读取的值，其下确值如下：<br>UTC＝<value>:<value>可以是下列值中的一个：<br>true或yes-硬件时钟设为universal格式<br>false或no-硬件时钟为本地时间<br>ARC＝<value>:<value>为下列值：<br>true或yes-设置为此值时硬件时钟只用于HRC或HLPHABIOS系统<br>false或no-设置为此值时硬件时钟只用于unix系统<br>SRM＝<value>:<value>为下列值：<br>yes或true－设置为此值时系统时间从1900年开始，此值只有于SRm-based ALPHA系统<br>no或false-设置为此值是为普通UNIX使用<br>ZONE＝＜filename>:<filename>为/usr/share/zoneinfo下的时区文件，如　zone="america/newyork"。
7|/etc/sysconfig/desktop|当使用运行级别5时，此文件为新用户指定桌面和运行显示管理器。其内容可能如下所示：<br>DESKTOP＝"<value>":<value>可以为下列值之一：<br>GNOME-使用GNOME桌面环境<br>KDE－使用KDE桌面环境<br>DISPLAYANAGER＝"<value>":<value>可以为以下值之一：<br>GNOME－使用GNOME显示管理器<br>KDE－使用KDE显示管理器<br>XDM－使用XDM显示管理器
8|/etc/sysconfig/devlabel|此文件为设备标签配置文件，不必通过手工编辑设置，可以使用/sbin/devlabel命令来修改其中指定项的值来设置此文件
9|/etc/sysconfig/dhcpd|此文件内容提供一些区段给dhcpd守护进程在系统引导时使用，dhcpd守护进程使用DHCP及BOOTP协议为主机自动分配IP地址
10|/etc/sysconfig/exim|此文件允许发送信息给一个或多个客户，如果网络需要可以路由此信息。
11|/etc/sysconfig/firstboot|此文件为firstboot守护进程的配置文件
12|/etc/sysconfig/gpm|此文件通过其内容中的一些段提供给gpm守护进程在系统引导时使用。gpm进程为鼠标服务，此文件内容包括一些与鼠标相关的如鼠标键数、接口信息等
13|/etc/sysconfig/harddisks|此文件中的内容为系统中已安装的硬盘的参数。其内容如下所列：<br>USE_DMA=1：设置硬盘是否使用DMA，值为1使用，0不使用。<br>MULTIPLE_IO=16：当此项设置值为16时，允许每一次I/O中断读取多个扇区。设置此值可以减少30%－50%的系统开销，但要小心使用此项，缺省为禁止。<br>EIDE＿32BIT＝3：设置此项值为3时找开（E）IDE32位I/0支持，缺省禁止。<br>LOOKAHEAD＝1：设置此项为1时允许驱动read-lookahead方式工作，缺省禁止。<br>EXTRA＿PARAMS＝SPECIFITS：指定EXTRA的参数，缺省为无（没有参数）
14|/etc/sysconfig/hwconf|此文件内容列出系统中所有KUDZU检测出来的硬件列表，此文件不应手动编辑，如果对此文件中的内容进行了改变，相应设备就会立即增加或删除
15|/etc/sysconfig/i18n|此文件内容用来设置缺省语言、其它支持的语言和缺省的系统字体，例如：<br>LANG="en_us,UTF-8"<br>SUPPORTED="en_us,UTF-8!en_us:en"<br>SYSFONT="latareycreb-sun16"
16|/etc/sysconfig/init|此文件中的内容用来在系统引导期间控制显示和其它功能。它的内容可以为以下所示：<br>BOOTUP=<value>:<value>可以为以下值之一：<br>color-设置为此值时，当设备初始化成功或失败时显示不同的颜色。<br>verbise-设置为此值时为一种旧的显示样式，提供一些信息比如成功或失败的信息<br>RES_COL=<value>:<value＞的内容为以数字方式表示的屏幕显示的信息的列数，缺省为60。<br>MOVE_TO_COL=<value>:<value>设置的值为移动光标时移动的列数。<br>ECHO_EN:此命令设定通过用echo -en命令光标移动的行数。<br>SETCOLOR＿SUCCESS＝＜value>:<value>设置的值用来设定echo -en命令成功时显示的颜色，缺省为绿色。<br>SETCOLOR＿TAILURE＝<value>:<value>设置的值用来设定echo -en命令错误时显示的颜色，缺省为红色。<br>SETCOLOR＿WARNING＝<value>:<value>设置的值用来设定echo -en命令警告时的颜色，缺省为黄色。<br>SETCOLOR＿NORMAL＝<value>:<value>设置的会下用来设定echo -en命令一般模式时的颜色为“NORMAL”。<br>LOGLEVEL=<value>:<value>设置内核初始化日志记录层次，缺省为3，设置为8时设置记录所有方面的日志信息，包括debugging信息，设置为1时只记录kernel panics信息，syslogd守护进程在启动时会读取此文件。<br>PROMPT＝<value>:<value>为下列值中之一：<br>yes－设置为此值时允许通过按键来进行交互模式显示启动<br>no－设置为此值时不允许通过按键来进行交互
17|/etc/sysconfig/ip6tables-config|此文件中保存的内容用来给内核在ip6tables服务启动或设置IPV6过滤规则时使用。不要直接编辑此文件中的内容，除非非常熟悉ip6tables的结构和规则。规则可以通过使用/sbin/ip6tables命令来创建
18|/etc/sysconfig/iptables-config|同上，适用于IPV4
19|/etc/sysconfig/autofsck|当系统出现问题的时候，由该文件决定是否在启动后使用fsck来检查硬盘
20|/etc/sysconfig/keyboard|设定键盘的形式
21|/etc/sysconfig/kudzu|设置开机后检查新设备的方式
22|/etc/sysconfig/network|网络设置
23|/etc/sysconfig/network-scripts|