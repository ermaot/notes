postgresql一般有yum安装和编译安装。现在为了安装方便，提供了脚本的傻瓜式安装。

## yum安装

#### 查看安装源

```
# yum search postgresql
……
postgresql.i686 : PostgreSQL client programs
postgresql.x86_64 : PostgreSQL client programs
……
postgresql-contrib.x86_64 : Extension modules distributed with PostgreSQL
postgresql-debuginfo.x86_64 : Debug information for package postgresql
postgresql-devel.i686 : PostgreSQL development header files and libraries
postgresql-devel.x86_64 : PostgreSQL development header files and libraries
……
postgresql-libs.i686 : The shared libraries required for any PostgreSQL clients
postgresql-libs.x86_64 : The shared libraries required for any PostgreSQL clients
……
postgresql-server.x86_64 : The programs needed to create and run a PostgreSQL server
```

可以看到postgresql是客户端，postgresql-server是服务器，postgresql-contrib是相应的扩展

#### 安装

```
# yum install postgresql postgresql-server postgresql-contrib -y
```

安装之后，postgres可执行文件位于

```
# which postgres
/usr/bin/postgres
```



## 源码安装

#### 下载

进入https://www.postgresql.org/download 官网下载安装包

#### 查看编译选项

```
./configure --help
```

选项|说明
---|---
--prefix= PREFIX|指定安装目录，默认的安装目录为“/usr/ ocal/pgsql”
--includedir=DIR|指定C和C++的头文件目录，默认的安装目录为“ PREFIX/ Include。
--with- report= PORTNUM|指定初始化数据目录时的默认端口，这个值可以在安装之后进行修改需要重启数据库），修改它只在自行制作RPM包时有用，其他时候意义并不大。
--with- blocksize= BLOCKSIZE|指定数据文件的块大小，默认的是8kB，如果在OLAP场景下可以适当加这个值到32kB，以提高OLAP的性能，但在OLTP场景下建议使用&kB默认值。
--with- segsize= SEGSIZE|指定单个数据文件的大小，默认是1GB。
--with-wal- blocksize= BLOCKSIZE|指定WAL文件的块大小，默认是8kB。
--with- wal-segsize= SEGSIZE|指定单个WAL文件的大小，默认是16MB。

#### 安装

使用gmake 和gmake install



## 交互式脚本安装

9.5之后提供了交互式的脚本安装，但11版本之后不再提供。参考链接：https://www.postgresql.org/download/linux/redhat/

```
# ./postgresql-10.13-1-linux-x64.run 
----------------------------------------------------------------------------
Welcome to the PostgreSQL Setup Wizard.

----------------------------------------------------------------------------
Please specify the directory where PostgreSQL will be installed.

Installation Directory [/opt/PostgreSQL/10]: 

----------------------------------------------------------------------------
Select the components you want to install; clear the components you do not want 
to install. Click Next when you are ready to continue.

PostgreSQL Server [Y/n] :Y

pgAdmin 4 [Y/n] :Y

Stack Builder [Y/n] :Y

Command Line Tools [Y/n] :Y

Is the selection above correct? [Y/n]: Y

----------------------------------------------------------------------------
Please select a directory under which to store your data.
```

