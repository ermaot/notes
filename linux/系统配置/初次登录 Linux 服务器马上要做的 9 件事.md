## 1、第一次接触

当我登录到一台服务器时，我做的第一件事就是检查它是否拥有我将要运行的测试所需的操作系统、内核和硬件架构。我经常会检查一台服务器已经运行了多久。虽然这对测试系统来说并不重要，因为它会被多次重启，但我还是发现这些信息很有帮助。

使用下面的命令来获取这些信息。我主要使用 Red Hat Linux 进行测试，所以如果你使用其他 Linux 发行版，请在文件名中使用 `*-release` 而不是 `redhat-release`：

```text
cat /etc/redhat-release
uname -a
hostnamectl
uptime
```

## 2、有人登录在上面吗？

一旦我知道这台机器符合我的测试需求，我需要确保没有其他人同时登录该系统运行他们自己的测试。虽然考虑到整备系统会帮我处理好这个问题，这种可能性很小，但偶尔检查一下还是有好处的 —— 尤其是当我第一次登录服务器的时候。我还会检查是否有其他用户（除了 root）可以访问系统。

使用下面的命令来查找这些信息。最后一条命令是查找 `/etc/passwd` 文件中具有 shell 访问权限的用户；它会跳过文件中没有 shell 访问权限或 shell 设置为 `nologin` 的其他服务：

```text
who
who -Hu
grep sh$ /etc/passwd
```

## 3、物理机还是虚拟机

现在我有了属于自己的机器，我需要确定它是一台物理机还是一台虚拟机（VM）。如果是我自己整备的这台机器，我可以确定这是我要求的东西。但是，如果你使用的是一台不是你自己整备的机器，你应该检查该机器是物理机还是虚拟机。

使用以下命令来识别这些信息。如果是物理系统，你会看到供应商的名称（如 HP、IBM 等）以及服务器的品牌和型号；而在虚拟机中，你应该看到 KVM、VirtualBox 等，这取决于创建虚拟机时使用了什么虚拟化软件：

```text
dmidecode -s system-manufacturer
dmidecode -s system-product-name
lshw -c system | grep product | head -1
cat /sys/class/dmi/id/product_name
cat /sys/class/dmi/id/sys_vendor
```

## 4、硬件

因为我经常测试连接到 Linux 机器的硬件，所以我通常使用物理服务器，而不是虚拟机。在物理机器上，我的下一步是确定服务器的硬件能力 —— 例如，运行的是什么类型的 CPU，它有多少个核心，启用了哪些标志，以及有多少内存可用于运行测试。如果我正在运行网络测试，我会检查连接到服务器的以太网或其他网络设备的类型和容量。

使用以下命令来显示连接到 Linux 服务器的硬件。其中一些命令在新的操作系统版本中可能会被废弃，但你仍然可以从 yum 存储库中安装它们或切换到它们的等效新命令：

```text
lscpu or cat /proc/cpuinfo
lsmem or cat /proc/meminfo
ifconfig -a
ethtool <devname>
lshw
lspci
dmidecode
```

## 5、已安装的软件

测试软件总是需要安装额外的依赖包、库等。然而，在安装任何东西之前，我都会检查已经安装了什么（包括是什么版本），以及配置了哪些存储库，这样我就知道软件来自哪里，并可以调试任何软件包安装问题。

使用下面的命令来确定安装了什么软件：

```text
rpm -qa
rpm -qa | grep <pkgname>
rpm -qi <pkgname>
yum repolist
yum repoinfo
yum install <pkgname>
ls -l /etc/yum.repos.d/
```

## 6、运行的进程和服务

检查了安装的软件之后，自然就会检查系统上有哪些进程在运行。当在系统上运行性能测试时，这一点至关重要 —— 如果一个正在运行的进程、守护进程、测试软件等占用了大部分 CPU/RAM，那么在运行测试之前停止该进程是有意义的。这也可以检查测试所需的进程或守护进程是否已经启动并运行。例如，如果测试需要 `httpd` 运行，那么即使安装了软件包，启动守护进程的服务也可能没有运行。

使用以下命令来识别系统上正在运行的进程和已启用的服务：

```text
pstree -pa 1
ps -ef
ps auxf
systemctl
```

## 7、网络连接

如今的机器网络化程度很高，它们需要与网络上的其他机器或服务进行通信。我会识别服务器上哪些端口是开放的，是否有到测试机器的任何网络连接，是否启用了防火墙，如果启用了，是否屏蔽了任何端口，以及机器与哪些 DNS 服务器对话。

使用以下命令来识别网络服务相关信息。如果一个过时的命令不可用，请从 yum 存储库中安装它或使用等效的新命令：

```text
netstat -tulpn
netstat -anp
lsof -i
ss
iptables -L -n
cat /etc/resolv.conf
```

## 8、内核

在进行系统测试时，我发现了解内核相关的信息是很有帮助的，比如内核版本和加载了哪些内核模块。我还会列出任何[可调整的内核参数](https://link.zhihu.com/?target=https%3A//www.oreilly.com/library/view/red-hat-enterprise/9781785283550/ch10s05.html)以及它们的设置，并检查启动运行中的内核时使用的选项。

使用以下命令来识别这些信息：

```text
uname -r
cat /proc/cmdline
lsmod
modinfo <module>
sysctl -a
cat /boot/grub2/grub.cfg
```

## 9、日志

现在，我已经对服务器有了很好的了解，包括安装了哪些软件，运行了哪些进程。还有一件事我无法逃避，那就是日志文件 —— 我需要知道在哪里可以查看不断更新的信息。

使用以下命令查看系统的日志：

```text
dmesg
tail -f /var/log/messages
journalctl
```

## 接下来的步骤

虽然命令和实用程序会发生变化，但它们所显示的基本信息大致不变。在你专注于掌握哪些命令之前，你需要对你要寻找的信息以及它属于什么类别有一个宏观的看法。

由于 Linux 将大部分信息保存在文件中，这些命令基本上是从文件中读取信息，并以一种易于理解的方式呈现出来。下一步的好做法是找出每个命令用来获取信息显示的文件。一个提示：寻找这些信息的方法是 `strace` 命令。



转载自https://zhuanlan.zhihu.com/p/339852180