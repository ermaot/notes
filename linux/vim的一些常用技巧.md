#### 默认显示行号

修改vim配置文件（我使用的是centos7.5），增加set nu

```
# vim /etc/vimrc 
```

#### 默认不增加注释

有时候在上下文有注释的情况下，新增一行会自动添加注释，这个有时候会增加麻烦。可以修改vim配置文件，增加set paste即可

#### 查看内核

```
# cat /proc/version 
Linux version 3.10.0-862.11.6.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) ) #1 SMP Tue Aug 14 21:49:04 UTC 2018
# uname -r
3.10.0-862.11.6.el7.x86_64
```

#### 查看发行版

```
# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
# lsb_release 
LSB Version:	:core-4.1-amd64:core-4.1-noarch
# cat /etc/issue
\S
Kernel \r on an \m
```

