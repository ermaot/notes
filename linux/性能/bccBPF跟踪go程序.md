。如果你去搜索	Go	和	BPF，你会发现使用	BPF	接口的	Go	语言接口（例如，gobpf）。这不是我所探索的东西：我将使用	BPF	工具实现	Go	应用程序的性能分析和调试

## 一 动态链接方式

### 编写go程序

```
package main

import "fmt"

func main() {
        fmt.Println("Hello,BPF!")
}
```

### 安装gccgo

```
yum install gcc-go.x86_64
```

GCCGO则是GCC专门用来编译Golang语言的

### 编译文件

```
# gccgo -o hello -g hello.go 
```

-g表示编译链接信息，输出符号表

## 执行文件

```
./hello
```

### 跟踪程序

在另外的终端执行如下命令

```
# /usr/share/bcc/tools/funccount 'go:fmt.*'
Tracing 129 functions for "go:fmt.*"... Hit Ctrl-C to end.
^C
FUNC                                    COUNT
fmt.Println                                 1
fmt.padString.pN7_fmt.fmt                   1
fmt.printField.pN6_fmt.pp                   1
fmt.fmt_s.pN7_fmt.fmt                       1
fmt.Fprintln                                1
fmt.init.pN7_fmt.fmt                        1
fmt.put.pN9_fmt.cache                       1
fmt.fmtString.pN6_fmt.pp                    1
fmt.truncate.pN7_fmt.fmt                    1
fmt.free.pN6_fmt.pp                         1
fmt.doPrint.pN6_fmt.pp                      1
fmt..import                                 1
fmt.WriteByte.pN10_fmt.buffer               1
fmt.WriteString.pN10_fmt.buffer             1
fmt.get.pN9_fmt.cache                       1
fmt.clearflags.pN7_fmt.fmt                  2
Detaching...
```

查看hello文件的信息

```
# file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=0b5b305c9aa9d364c443c218a88e72c678e173c7, not stripped
# ldd hello
	linux-vdso.so.1 =>  (0x00007ffcbb980000)
	libgo.so.4 => /lib64/libgo.so.4 (0x00007fee464e8000)
	libm.so.6 => /lib64/libm.so.6 (0x00007fee461e6000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007fee45fd0000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fee45c03000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fee471fb000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fee459e7000)
```

可以看到时动态链接的可执行文件

## 二 静态链接方式

编译的时候，不适用gccgo，而使用go自带的工具

```
# go build hello.go 
```

```
# file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
# ldd hello
	不是动态可执行文件
```
同样执行文件

```
./hello
```

但由于是静态链接，所以跟踪程序方法稍有不同

```
# /usr/share/bcc/tools/funccount '/usr/share/systemtap/tapset/linux/hello:fmt*'
Tracing 41 functions for "/usr/share/systemtap/tapset/linux/hello:fmt*"... Hit Ctrl-C to end.
^C
FUNC                                    COUNT
fmt.(*fmt).padString                        2
fmt.(*fmt).truncateString                   2
fmt.(*fmt).fmtS                             2
fmt.newPrinter                              2
fmt.(*pp).free                              2
fmt.Fprintln                                2
fmt.(*pp).fmtString                         2
fmt.(*pp).printArg                          2
fmt.(*pp).doPrintln                         2
fmt.glob..func1                             2
fmt.init                                    2
Detaching...
```

## 三 使用funclatency

```
# /usr/share/bcc/tools/funclatency 'go:fmt.Println'
Tracing 1 functions for "go:fmt.Println"... Hit Ctrl-C to end.
^C

Function = [unknown] [19435]
     nsecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 0        |                                        |
         8 -> 15         : 0        |                                        |
        16 -> 31         : 0        |                                        |
        32 -> 63         : 0        |                                        |
        64 -> 127        : 0        |                                        |
       128 -> 255        : 0        |                                        |
       256 -> 511        : 0        |                                        |
       512 -> 1023       : 0        |                                        |
      1024 -> 2047       : 0        |                                        |
      2048 -> 4095       : 0        |                                        |
      4096 -> 8191       : 0        |                                        |
      8192 -> 16383      : 0        |                                        |
     16384 -> 32767      : 0        |                                        |
     32768 -> 65535      : 1        |****************************************|

Function = [unknown] [19431]
     nsecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 0        |                                        |
         8 -> 15         : 0        |                                        |
        16 -> 31         : 0        |                                        |
        32 -> 63         : 0        |                                        |
        64 -> 127        : 0        |                                        |
       128 -> 255        : 0        |                                        |
       256 -> 511        : 0        |                                        |
       512 -> 1023       : 0        |                                        |
      1024 -> 2047       : 0        |                                        |
      2048 -> 4095       : 0        |                                        |
      4096 -> 8191       : 0        |                                        |
      8192 -> 16383      : 0        |                                        |
     16384 -> 32767      : 0        |                                        |
     32768 -> 65535      : 0        |                                        |
     65536 -> 131071     : 1        |****************************************|
Detaching...
```

