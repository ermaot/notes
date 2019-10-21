MySQL数据库的连接访问控制机制分为两个阶段：<p>
1. 第一个阶段,判断是否允许用户连接到数据库
2. 第二个阶段,成功建立连接以后,用户可以对哪些数据库及其数据库表进行有效权限的操作
## 账号认证
判断连接用户及其所在主机是否满足默认mysql数据库中user表所包含的相关认证信息
#### 账号定义
- MySQL中，用户账号是由user和host共同组成的
- mysql.user表中的host、user、password三个字段信息用作接入连接的认证参考
- host、user、password都不能为null，但默认会设置为空，即user可以匿名，无password，从任何主机连接
#### 身份审核
- 认证步骤
1. MySQL会首先调用sql/sql_acl.cc中的函数acl_check_host()来判断用户是否能够满足连接的主机名和IP地址
2. 通过host判断后，调用sql/password.c中的函数create_randowm_string()产生一个随机的字符串，来进行密码审核（密码认证步骤：server端先产生随机字符串，发送给登录客户端；客户端收到字符串，将输入的密码进行两次哈希处理，然后再与随机字符串重新生成新字符串，返回给server段；最后server段用本地保存的密码哈希处理后与客户端传来的字符串比较）
3. 然后MySQL通过sql/sql_parse.cc的函数check_user()去核实用户是否存在以及密码是否正确
4. 最后调用sql/sql_acl.cc中的函数acl_getroot()，用来合适用户资源信息；如果全部通过，调用acl_update_user()将THD类中的用户数据和资源结构更新
#### 具体优先原则
当服务器把user表读到内存后，根据host字段的具体程度对这三个字段进行排列，然后选择最先匹配的一条记录登录连接
## 权限控制
#### 系统权限表
在默认数据库mysql库中，包含了各个级别权限所需的6张权限表：user、db、host、tables_priv、columns_ppriv、procs_priv
1. user表：除了在前面账号认证一节中提到的,利用它来进行登录验证外,user表又被称为全局表,用来对所有数据库对象的操作进行限制.该表包括了MySQL数据库操作中所能够实现的所有权限,又被形象地称为超级用户表,默认情况下root用户具有所有权限
2.db表：db表是一个局部表,它是针对指定的某个数据库,允许合法用户执行该表中允许的权限,权限范围是该数据库中的所有对象.更多时候用户的权限都来自于db表所设置的权限
3. host表：被用作db表的辅助表.该表中host字段可以写入所有想要执行db表中规定权限的允许连接的主机,从而可以很方便地对多台允许连接的主机赋予权限.
4. tables_priv 和columns_priv 表：这两个表对数据库中的指定表和字段所应享有的权限进行定义,充分细化到某一个表中的所有字段或是某一个特定的字段.
5. procs_priv 表：该表是对某一个单独的存储程序或函数进行权限管理.
#### 权限审核
- 客户端通过认证后，MySQL会先调用sql/sql_parse.cc中的函数check_access()，去读取所有能使用表所具有的全局权限和db权限
- 执行一条sql语句的时候，MySQL会调用sql/sql_parse.cc中的函数check_table_access()以判断用户是否有权限访问被操作的表，同时将用户请求权限缓存到table list；然后sql/spl_acl.cc 中的函数check_grant()去检查table list中是否有满足要求的相应权限
#### 权限级别

MySQL给一个合法用户赋权按照5个表的顺序：user→db→table_priv→columns_priv→procs_priv
- 对所有数据库的所有表赋权的时候，用户的权限信息不会再被记录到db表中

## 安全部署
#### 服务器系统安全
1. 网络安全
2. 系统安全
#### 数据库系统安全
###### 默认用户管理
默认有4组合法用户，即@localhost  、@hostname、root@localhost、root@hostname。从本机不需要密码即可登录
1. 删除匿名用户

```
drop user ''@'localhost';
drop user ''@'$hostname';
```

2. 为超级用户设置密码
###### 以非root用户启动MySQL
root用户可以直接将/etc/passwd导入到数据库中
###### 登录主机域名解析
- 如果使用主机名登录，连接进程会通过DNS解析成IP再判定是否允许登录（可以虚假解析）。为此MySQL启动时，禁止DNS服务，只允许IP地址认证登录

```
mysqld_safe --skip-name-resolve
```
###### unix domain socket
MySQL通过unix系统提供的socket端口通信

```
mysqld_safe --skip-networking
mysql -uroot -p --socket=/$PATH_TO/mysql.sock
```
###### SSL 连接
要MySQL支持SSL，源码编译阶段需要添加特定的配置参数

```
mysqld --ssl-ca=cacert.pem --ssl-cert=server-cert.pem --ssl-key=server-key.pem
```
###### 跨越权限表
启动时可以绕过权限认证

```
mysqld_safe --skip-grant-table
```
可以通过编译选项禁止

```
./configure --disable-grant-options
```

###### SQL注入