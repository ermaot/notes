## 客户端工具

工具|解释
---|---
clusterdb|cluster命令的封装。对物理文件重新排序，一定情况下可以节省磁盘空间，加快搜索速度
reindexdb|reindex的封装，物理索引文件发生损坏或者索引膨胀，reindex可以对指定表重建索引并删除旧索引
vacuumdb|vacuum、vacuum freeze、vacuum full、vacuum analyze这几个命令的封装
vacuumlo|清理数据库中的为引用的大对象
createdb、dropdb|create database和drop database的封装
createuser、dropuser|create user和drop user的封装
pg_basebackup|取得一个正在运行中的postgresql实例的基础备份
pg_dump和pg_dumpall|数据库转储方式备份
pg_restore|从pg_dump命令创建的非文本格式的备份中恢复
ecpg|用于C程序的 PostgreSQL嵌人式SQL预处理器。它将SQL调用替换为特殊函数调用，把带有嵌入式SQL语句的C程序转换为普通C代码。输出文件可以被任何C编译器工具处理。
oid2name|解析一个 PostgreSQL数据目录中的OID和文件结点
pgbench|是运行基准测试的工具，平常我们可以用它模拟简单的压力测试。
pg_config|获取当前安装的 PostgreSQL应用程序的配置参数。
pg_receivexlog|可以从一个运行中的实例获取事务日志的流。
pg_recvlogical|控制逻辑解码复制槽以及来自这种复制槽的流数据。
psql|是连接 PostgreSQL数据库的客户端命令行工具，是使用频率非常高的工具

## 服务器工具

工具|说明
---|---
initdb|用来创建新的数据库目录。
pg_archivecleanup|清理 PostgreSQL WAL归档文件的工具。
pg_controldata|显示数据库服务器的控制信息，例如目录版本、预写日志和检查点的信息。
pg_ctl|初始化、启动、停止、控制数据库服务器的工具。
pg_resetwal|可以清除预写日志并且有选择地重置存储在 pg_control文件中的一些控制信息。当服务器由于控制文件损坏， pg_resetwal可以作为最后的手段。
pg_rewind|是在 master、 slave角色发生切换时，将原 master通过同步模式恢复，避免重做基础备份的工具。
pg_test_fsync|可以通过一个快速的测试，了解系统使用哪一种预写日志的同步方法（ wal sync method）最快，还可以在发生IO问题时提供诊断信息。
pg_test_timing|是一种度量系统计时开销以及确认系统时间绝不会回退的工具。
pg_upgrade|是 PostgreSQL的升级工具，在版本升级的章节会详细讲解。
pg_waldump|用来将预写日志解析为可读的格式。
postgres是| PostgreSQL的服务器程序。
postmaster|可以从bin目录中看到，是指向 postgres服务器程序的一个软链接