Centos7 gcc版本默认4.8.3，Red Hat 为了软件的稳定和版本支持，yum 上版本也是4.8.3，所以无法使用yum进行软件更新，所以使用scl。

scl软件集(Software Collections),是为了给 RHEL/CentOS 用户提供一种以方便、安全地安装和使用应用程序和运行时环境的多个（而且可能是更新的）版本的方式，同时避免把系统搞乱。

使用scl升级gcc步骤：

## 一、安装scl源

```
# yum install centos-release-scl scl-utils-build
```

## 二、列出scl有哪些源可以用

```
# yum list all --enablerepo='centos-sclo-rh' | grep gcc
gcc.x86_64                                 4.8.5-44.el7           @os           
gcc-c++.x86_64                             4.8.5-44.el7           @os           
libgcc.x86_64                              4.8.5-44.el7           @os           
compat-gcc-32-debuginfo.x86_64             3.2.3-72.el7           debug         
compat-gcc-34-debuginfo.x86_64             3.4.6-32.el7           debug         
compat-gcc-44.x86_64                       4.4.7-8.el7            os            
compat-gcc-44-c++.x86_64                   4.4.7-8.el7            os            
compat-gcc-44-debuginfo.x86_64             4.4.7-8.el7            debug         
compat-gcc-44-gfortran.x86_64              4.4.7-8.el7            os            
devtoolset-3-gcc-debuginfo.x86_64          4.9.2-6.el7            centos-sclo-rh-debuginfo
devtoolset-4-gcc-debuginfo.x86_64          5.3.1-6.1.el7          centos-sclo-rh-debuginfo
devtoolset-6-gcc-debuginfo.x86_64          6.3.1-3.1.el7          centos-sclo-rh-debuginfo
devtoolset-7-gcc.x86_64                    7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-gcc-c++.x86_64                7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-gcc-debuginfo.x86_64          7.3.1-5.16.el7         centos-sclo-rh-debuginfo
devtoolset-7-gcc-gdb-plugin.x86_64         7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-gcc-gfortran.x86_64           7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-gcc-plugin-devel.x86_64       7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-libgccjit.x86_64              7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-libgccjit-devel.x86_64        7.3.1-5.16.el7         centos-sclo-rh
devtoolset-7-libgccjit-docs.x86_64         7.3.1-5.16.el7         centos-sclo-rh
devtoolset-8-gcc.x86_64                    8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-gcc-c++.x86_64                8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-gcc-debuginfo.x86_64          8.3.1-3.2.el7          centos-sclo-rh-debuginfo
devtoolset-8-gcc-gdb-plugin.x86_64         8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-gcc-gfortran.x86_64           8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-gcc-plugin-devel.x86_64       8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-libgccjit.x86_64              8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-libgccjit-devel.x86_64        8.3.1-3.2.el7          centos-sclo-rh
devtoolset-8-libgccjit-docs.x86_64         8.3.1-3.2.el7          centos-sclo-rh
devtoolset-9-gcc.x86_64                    9.3.1-2.el7            centos-sclo-rh
devtoolset-9-gcc-c++.x86_64                9.3.1-2.el7            centos-sclo-rh
devtoolset-9-gcc-debuginfo.x86_64          9.3.1-2.el7            centos-sclo-rh-debuginfo
devtoolset-9-gcc-gdb-plugin.x86_64         9.3.1-2.el7            centos-sclo-rh
devtoolset-9-gcc-gfortran.x86_64           9.3.1-2.el7            centos-sclo-rh
devtoolset-9-gcc-plugin-devel.x86_64       9.3.1-2.el7            centos-sclo-rh
devtoolset-9-libgccjit.x86_64              9.3.1-2.el7            centos-sclo-rh
devtoolset-9-libgccjit-devel.x86_64        9.3.1-2.el7            centos-sclo-rh
devtoolset-9-libgccjit-docs.x86_64         9.3.1-2.el7            centos-sclo-rh
gcc-base-debuginfo.x86_64                  4.8.5-44.el7           debug         
gcc-debuginfo.x86_64                       4.8.5-44.el7           debug         
gcc-gfortran.x86_64                        4.8.5-44.el7           os            
gcc-gnat.x86_64                            4.8.5-44.el7           os            
gcc-go.x86_64                              4.8.5-44.el7           os            
gcc-libraries-debuginfo.x86_64             8.3.1-2.1.1.el7        debug         
gcc-objc.x86_64                            4.8.5-44.el7           os            
gcc-objc++.x86_64                          4.8.5-44.el7           os            
gcc-plugin-devel.x86_64                    4.8.5-44.el7           os            
libgcc.i686                                4.8.5-44.el7           os            
relaxngcc.noarch                           1.12-6.el7             os            
relaxngcc-javadoc.noarch                   1.12-6.el7             os            

```

## 三、安装7版本的gcc、gcc-c++、gdb
```
yum install devtoolset-7-gcc devtoolset-7-gcc devtoolset-7-gcc-debuginfo devtoolset-7-gcc-gdb-plugin
```

## 四、查看从 SCL 中安装的包的列表：

```
scl --list 或 scl -l
```

## 五、切换版本

切换前查看gcc版本

切换版本： 

```
scl enable devtoolset-4 bash
```

## 六、scl常用命令

```
scl --list 或scl -l``scl --help 或 scl -h``scl enable <scl-package-name> <command> #使用scl来执行command命令``scl enable devtoolset-4 bash #使用scl创建一个scl包的bash会话环境``exit #退出当前scl bash环境，恢复成系统bash环境
```





参考：https://www.cnblogs.com/dj0325/p/8481092.html