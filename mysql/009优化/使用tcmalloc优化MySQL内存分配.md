本文来自http://www.penglixun.com/tech/database/opt_mysql_using_tcmalloc.html

http://www.penglixun.com/tech/database/static_compile_mysql_with_tcmalloc.html

## 动态库

1. 先安装libunwind，然后安装tcmalloc包。
2. 按默认方式编译和安装成功后在 mysqld_safe 中加入
3. LD_PRELOAD=”/usr/local/lib/libtcmalloc.so”
4. 重启MySQL。

## 静态编译

Google开源的tcmalloc则改进了malloc的一些效率问题，在大量malloc和free时，操作系统的内存曲线明显比Linux下malloc函数要平稳，在大并发情况下，提升程序稳定性和性能。
一般网上都是把tcmalloc动态库加到mysqld_safe中启动，下面是静态编译方法。

#### 编译tcmalloc先要编译libunwind

```
wget http://download.savannah.gnu.org/releases/libunwind/libunwind-0.99.tar.gz
tar zxvf libunwind-0.99.tar.gz

CHOST=”x86_64-pc-linux-gnu” \
CFLAGS=” -O3 -fPIC \
-fomit-frame-pointer \
-pipe \
-march=nocona \
-mfpmath=sse \
-m128bit-long-double \
-mmmx \
-msse \
-msse2 \
-maccumulate-outgoing-args \
-m64 \
-ftree-loop-linear \
-fprefetch-loop-arrays \
-freg-struct-return \
-fgcse-sm \
-fgcse-las \
-frename-registers \
-fforce-addr \
-fivopts \
-ftree-vectorize \
-ftracer \
-frename-registers \
-minline-all-stringops \
-fbranch-target-load-optimize2″ \
CXXFLAGS=”${CFLAGS}” \
./configure && make && make install
```

#### 然后编译tcmalloc