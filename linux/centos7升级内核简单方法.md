借助elrepo。原文链[接CentOS7/6内核升级的简单方法：借助ELRepo，用yum命令更新内核](https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2019/02/25/centos-kernel-upgrade.html/)
## 安装elrepo

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org 
# for centos 7
rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

## 安装最新稳定版内核
安装elrepo包含的最新的稳定版内核：


```
yum --enablerepo=elrepo-kernel install kernel-ml

yum --enablerepo=elrepo-kernel install kernel-ml-devel    #如果要编译内核模块、eBPF程序等
```

可以用下面命令查看elrepo支持的内核：


```
yum --enablerepo=elrepo-kernel search kernel
```
## 更改grub配置使用最新的内核启动
配置grub2，下次启动时使用新安装的内核，用下面命令列出grub2中配置的内核：


```
# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.2.11-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
2 : CentOS Linux (0-rescue-735fbcd856b697dbc8d6deb1a11dd712) 7 (Core)
```

将CentOS Linux （4.20.12..) 设置为默认启动项，对应的序号是0：

```
grub2-set-default 0
grub2-mkconfig -o /boot/grub2/grub.cfg
```
## 重启机器

```
reboot
```

内核更新了：


```
# uname -r
5.2.11-1.el7.elrepo.x86_64
```
