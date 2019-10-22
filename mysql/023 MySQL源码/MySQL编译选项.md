## 编译选项
来自==622463 MySQL运维内参：MySQL、Galera、Inception核心原理与最佳实践==
与[mysql编译参数详解（./configure）](https://www.cnblogs.com/lisuyun/articles/4507715.html)
```
cmake .. -DBUILD_CONFIG=mysql_release \
-DINSTALL_LAYOUT=STANDALONE \
-DCMAKE_BUILD_TYPE=RelwithDebInfo \
-DENABLE_DTRACE=OFF
-DWITH_EMBEDDED_SERVER=OFF \
-DWITH_INNODB_MEMCACHED=ON \
-DWITH_SSL=bundled \
-DWITH_ZLIB=system 
-DWITH_PAM=ON \
-DCMAKE_INSTALL_PREFIX=/var/mysql/
-DINSTALL_PLUGINDIR="/var/mysql/lib/plugin" \
-DDEFAULT_CHARSET=utf8 \
-DEFAULT_COLLATION=utf8_general_ci \
-DWITH_EDITLINE=bundled \
-DFEATURE_SET=community \
-DCOMPILATION COMMENT="MySQL Server（GPL）"\
-DWITH_DEBUG=OFF \
-DWITH_BOOST=..
```
选项|说明
---|---
CMAKE_BUILD_TYPE|这是编译选项，用来指定编译的是 RELEASE版本还是 DEBUG版本，或者是 RelwithDebInfo版本的
CMAKE_INSTALL_PREFIX|这是用来指定安装路径的，指定的目录为MySQL安装之后的根目录
ENABLED_PROFIING|这个选项用来指定是否打开 profile开关，一般想要查看 MySQL执行信息时可以打开它
MYSQL_DATADIR|这个选项指定了在编译INSTALL工程时，生成的默认数据库路径，即在没有指定启动参数--defaults-file时的默认数据库路径
OPTIMIZER_TRACE|用来指明是否打开优化器 TRACE模块
WITH_ARCHIVE_STORAGE_ENGINE|表示是否支持 Archive存储引擎
WITH_INNOBASE_STORAGE_ENGINE|表示是否支持 InnoDB存储引擎
WITH_BLACKHOLE_STORAGE ENGINE|表示是否支持 BLACKHOLE存储引擎
WITH_EXAMPLE_STORAGE_ENGINE|支持Example存储引擎，这个存储引擎是一个示例，开发人员可以参考这个存储引擎来创建自己的存储引擎
WITH_FEDERATED_STORAGE_ENGINE|表示是否支持 Federated存储引擎
WITH_INNODB_MEMCACHED|表示是否支持 INNODB的 MEMCACHED 
WITH_PARTITION_STORAGE_ENGINE|表示是否支持 partition存储引擎
WITH_PERFSCHEMA_STORAGE_ENGINE|表示是否支持信息模式存储引擎
prefix=PREFIX|指定程序安装路径；
enable-assembler|使用汇编模式；（文档说明：compiling in x86 (and sparc) versions of common string operations, which should result in more performance.  汇编x86的普通操作符，可以提高性能）
enable-local-infile|启用对LOAD DATA LOCAL INFILE语法的支持(默认不支持)；
enable-profiling|Build a version with query profiling code (req.community-features)
enable-thread-safe-client|使用编译客户端；(让客户端支持线程的意思)
with-big-tables|启用32位平台对4G大表的支持；
with-charset=CHARSET|指定字符集；
with-collation=|默认collation；
with-extra-charsets=CHARSET,CHARSET,...|指定附加的字符集；
with-fast-mutexes|Compile with fast mutexes
with-readline|
with-ssl|启用SSL的支持；
with-server-suffix=|添加字符串到版本信息；
with-embedded-server|编译embedded-server，构建嵌入式MySQL库；
with-pthread|强制使用pthread类库；
with-mysqld-user=|指定mysqld守护进程的用户；
with-mysqld-ldflags=|静态编译MySQL服务器端；（静态链接提高13%性能）
with-client-ldflags=|静态编译MySQL客户端；（静态链接提高13%性能）
with-plugins=PLUGIN|PLUGIN 等等等（MySQL服务器端支持的存储引擎组件(默认为空)，可选值较多：<p>partition：MySQL Partitioning Support<p>daemon_example：This is an example plugin daemon<p>ftexample：Simple full-text parser plugin<p>archive：Archive Storage Engine<p>blackhole：Basic Write-only Read-never tables<p>csv：Stores tables in text CSV format，强制安装<p>example：Example for Storage Engines for developers<p>federated：Connects to tables on remote MySQL servers<p>heap：Volatile memory based tables，强制安装<p>ibmdb2i：IBM DB2 for i Storage Engine<p>innobase：Transactional Tables using InnoDB<p>innodb_plugin：Transactional Tables using InnoDB<p>myisam：Traditional non-transactional MySQL tables，强制安装<p>myisammrg：Merge multiple MySQL tables into one，强制安装<p>ndbcluster：High Availability Clustered tables<p>
with-plugin-PLUGIN|强制指定的插件链接至MySQL服务器；
with-zlib-dir=|向MySQL提供一个自定义的压缩类库地址；
without-server|仅安装MySQL客户端；
without-query-cache|不要编译查询缓存；
without-geometry|不要编译geometry-related部分；
without-debug|编译为产品版，放弃debugging代码；
without-ndb-debug|禁用special ndb debug特性；



```
make -j 24          //-j并行编译，提高速度
```
