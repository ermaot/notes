## PG起源
- PostgreSQL是由PostgreSQL社区全球志愿者开发团队开发的开源对象-关系型数据库.它源于 UCBerkeley大学1977年的Ingres计划,这个项目是由著名的数据库科学家Michael Stonebraker(2015年图灵奖获得者)领导的
- 在1994年,两个 UC Berkeley大学的研究生 Andrew Y和Jol!chen增加了一个SQL语言解释器来替代早先的基于 Ingres的QUEL系统,建立了Postgres95.为了反映数据库的新SQL查询语言特性, Postgres95在1996年重命名为 PostgreSQL,并第一次发行了以PostgreSQL命名的6.0版本
- 在2005年, PostgreSQL发行了以原生方式运行在Windows系统下的8.0版本.
- 2010年PostgreSQL9.0的发行,PostgreSQL进入了黄金发展阶段,目前, PostgreSQL最新的稳定版是 PostgreSQL10.
- ==PostgreSQL是目前可免费获得的最高级的开源数据库.它非常稳定可靠==
## PG特点
- PostgreSQL几乎支持多种操作系统,包括各种 Linux发行版及多种UNX、类UNX系统以及 Windows系统,例如AIⅨX、BSD、 HP-UX、 SGIIRIX、 Mac os x、 Solaris、Tru64.
- 它有丰富的编程接口,如C、C++、Go、Java、Perl、 Python、Ruby、Tcl和开放数据库连接(ODBC)的编程接口
- 支持广泛的数据类型,数组、json、 json及几何类型,还可以使用SQL命令 CREATE TYPE创建自定义类型.
- 支持大部分的SQL标准,可以支持复杂SQL查询、支持SQL子查询、 Window function,
- 有非常丰富的统计函数和统计语法支持;支持主键、外键、触发器、视图、物化视图,还可以用多种语言来编写存储过程,例如C、Java、 python、R语言等.
- 持并行计算和基于MVCC的多版本并发控制,支持同步、半同步、异步的流复制
- 支持逻辑复制和订阅, Hot Standby,支持多种数据源的外部表( Foreign data wrappers),可以将其他数据源当作自己的数据表使用,例如 Oracle、 MySQL、 Informix、 SQLite、MS SQL Server等.

## 许可证
postgresql使用postgresql licence生命，类似于BSD和MIT许可证

## PG安装
#### yum安装
