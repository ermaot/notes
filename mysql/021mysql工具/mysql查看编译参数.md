参考文章

https://blog.csdn.net/ylqmf/article/details/7793532



查看已编译的mysql的编译参数，可使用mysqlbug工具

```
# mysqlbug
Finding system information for a MySQL bug report
test -x 
Could not find a text editor. (tried emacs)
You can change editor by setting the environment variable VISUAL.
If your shell is a bourne shell (sh) do
VISUAL=your_editors_name; export VISUAL
If your shell is a C shell (csh) do
setenv VISUAL your_editors_name
```

第一次执行发现不成功，按照提示来做

```
VISUAL=vim; export VISUAL
```

再次执行

```
# mysqlbug
……
……
Finding system information for a MySQL bug report
test -x /usr/bin/vim
Using editor /usr/bin/vim
You can change editor by setting the environment variable VISUAL.
If your shell is a bourne shell (sh) do
VISUAL=your_editors_name; export VISUAL
If your shell is a C shell (csh) do
setenv VISUAL your_editors_name
File not changed, no bug report submitted.
The raw bug report exists in /tmp/failed-mysql-bugreport
If you use this remember that the first lines of the report are now a lie..
```

可以看到编译选项，同时，内容也会保存到/tmp/failed-mysql-bugreport

```
​```
>MySQL support: [none | licence | email support | extended email support ]
>Synopsis:      <synopsis of the problem (one line)>
>Severity:      <[ non-critical | serious | critical ] (one line)>
>Priority:      <[ low | medium | high ] (one line)>
>Category:      mysql
>Class:         <[ sw-bug | doc-bug | change-request | support ] (one line)>
>Release:       mysql-5.5.56 (MariaDB Server)

>C compiler:    cc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-16)

>C++ compiler:  c++ (GCC) 4.8.5 20150623 (Red Hat 4.8.5-16)

>Environment:
        <machine, os, target, libraries (multiple lines)>
System: Linux VM_112_34_centos 5.2.11-1.el7.elrepo.x86_64 #1 SMP Thu Aug 29 08:10:52 EDT 2019 x86_64 x86_64 x86_64 GNU/Linux
Architecture: x86_64

Some paths:  /usr/bin/perl /usr/bin/make /usr/bin/gmake /usr/bin/gcc /usr/bin/cc
GCC: 使用内建 specs。
COLLECT_GCC=/usr/bin/gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
Compilation info (call): CC='/usr/bin/cc'  CFLAGS=''  CXX='/usr/bin/c++'  CXXFLAGS=''  LDFLAGS=''  ASFLAGS=''
Compilation info (used): CC='/usr/bin/cc'  CFLAGS=''  CXX='/usr/bin/c++'  CXXFLAGS=''  LDFLAGS=''  ASFLAGS=''


Perl: This is perl 5, version 16, subversion 3 (v5.16.3) built for x86_64-linux-thread-multi
```

