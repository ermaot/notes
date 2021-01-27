本文来自：https://bean-li.github.io/systemtap-check-memory-leak/

## 为何需要systemtap检测内存泄漏

C/C++程序，Memory Leak是非常讨厌也是非常具有挑战的话题。简单的程序，靠code review就可以搞定，但是复杂的体系庞大的程序，很难通过code review找到对应的点，通过手段缩小范围是十分必要的。

有Valgrind的工具可以跑程序，检查内存泄漏，但是更多地用于SuperLab自测。客户现场出了内存泄漏，需要的是一种不改变程序，尽可能保留宝贵的现场的定位手段。SystemTap提供出一种动态追踪的手段，当内存泄漏正在发生时，可以通过SystemTap追踪内存的分配（malloc）和释放（free），同时记录下用户程序的调用堆栈，统计一段时间，以分配内存的堆栈作为key，而以分配出去但是尚未free的内存总量作为value，观察对内存增长贡献最大的代码路径。

我们希望有一个一致的手段，当内存泄漏发生时，我们运行追踪脚本，统计出最可疑的调用堆栈路径。

之所以琢磨这件事情，是因为我们的ceph-mds程序在某种的情况下发生了内存泄漏，驻留集内存RSS不断增长。希望通过手段，快速找到对应的调用堆栈。



### 安装glibc以及符号表

```
yum install kernel-devel glibc.x86_64 glibc-debuginfo.x86_64 
```

查看安装环境信息

```
# uname -r
5.2.11-1.el7.elrepo.x86_64
# stap -V
Systemtap translator/driver (version 4.0/0.168/0.176, rpm 4.0-11.el7)
Copyright (C) 2005-2018 Red Hat, Inc. and others
This is free software; see the source for copying conditions.
tested kernel versions: 2.6.18 ... 4.19-rc7
enabled features: AVAHI BOOST_STRING_REF DYNINST BPF JAVA PYTHON2 LIBRPM LIBSQLITE3 LIBVIRT LIBXML2 NLS NSS READLINE
```

使用stap -L测试内核符号和glibc的符号表

```
# stap -L  'kernel.function("printk")'
# stap -L  'process("/lib64/libc.so.6").function("malloc")'
```

如果没有报错，就说明安装成功。否则需要使用debuginfo-install安装所需符号表

### 测试程序

```
#  cat t.c
#include <stdlib.h>
 
void fun() {
  malloc(1000);
}
 
int main(int argc, char *argv[]) {
  fun();
  return 0;
}
 
 
# cat m.stp
probe process("/lib/x86_64-linux-gnu/libc.so.6").function("malloc") {
	if (target()== pid()) {
		print_ubacktrace();
		exit();
	}
}
probe begin {
	println("~");
}
```

### 编译并链接测试

```
gcc  -g t.c
# stap -L 'process("./a.out").function("*")'
process("/root/a.out").function("__do_global_dtors_aux")
process("/root/a.out").function("__libc_csu_fini")
process("/root/a.out").function("__libc_csu_init")
process("/root/a.out").function("_fini")
process("/root/a.out").function("_init")
process("/root/a.out").function("_start")
process("/root/a.out").function("deregister_tm_clones")
process("/root/a.out").function("frame_dummy")
process("/root/a.out").function("fun@/root/t.c:3")
process("/root/a.out").function("main@/root/t.c:7") $argc:int $argv:char**
process("/root/a.out").function("register_tm_clones")

```

### 测试

```
stap m.stp -c ./a.out
```

然后使用addr2line找出代码所在

```
addr2line -e ./a.out 0x4004a6   
```

