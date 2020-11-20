Linux内核里有一个kprobe机制，这是一个动态的收集debug信息的工具，systemtap就是基于kprobe机制的一个调试工具。systemtap的使用简单，而且可以自己定义脚本，对开发人员、系统管理员来说是一个深入分析性能的利器。

## 一、安装准备

systemtap的安装需要准备一台测试机器，先在测试机器上安装kernel-debuginfo、kernel-debuginfo-common、kernel-level、systemtap-runtime、gcc等相关包，然后在测试机器上编译测试脚本，编译测试完成之后将编译好的模块放在生产机器上，生产机器只需要安装systemtap-runtime包即可

CentOS的debuginfo相关安装包可以在http://debuginfo.centos.org/里找到。
**注意，安装kernel-debuginfo时，相应的内核版本一定要一致！**

## 二、安装kernel相关包

首先要确定系统内核版本，然后下载相对应的kernel-debuginfo包。

```
# uname -a
Linux systemtap.example.com 2.6.32-504.el6.x86_64 #1 SMP
```

然后安装2.6内科的相关kernel包

## 三、安装Systemtap软件包

```
yum -y install  gcc systemtap 
```

## 四、编译Systemtap脚本

Systemtap软件包本身已经提供了一些脚本，这些脚本涉及I/O、内存、网络等，并且几乎可以直接使用。下面以profiling中的topsys.stp为例

```
# cd /usr/share/systemtap/examples/profiling/
```

查看内容topsys.stp

```
#!/usr/bin/stap
#
# This script continuously lists the top 20 systemcalls in the interval 
# 5 seconds
#

global syscalls_count

probe syscall_any {
  syscalls_count[name] <<< 1
}

function print_systop () {
  printf ("%25s %10s\n", "SYSCALL", "COUNT")
  foreach (syscall in syscalls_count- limit 20) {
    printf("%25s %10d\n", syscall, @count(syscalls_count[syscall]))
  }
  delete syscalls_count
}

probe timer.s(5) {
  print_systop ()
  printf("--------------------------------------------------------------\n")
}

global prom_arr

probe prometheus {
  foreach (syscall in syscalls_count- limit 20)
    prom_arr[syscall] = @count(syscalls_count[syscall])

  @prometheus_dump_array1(prom_arr, "top_syscall_count", "name")
  delete prom_arr
}

```

测试脚本是否通过

```
stap -v topsys.stp
```

如果出现pass 5：starting run，则说明通过，可以编译

```
# stap -v -p4 -m topsys.ko topsys.stp
```

然后拷贝到所需运行的机器（该机器至少需要安装systemtap-runtime）

## 五、运行

```
# staprun topsys.ko
```

