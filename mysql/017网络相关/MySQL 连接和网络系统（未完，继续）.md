## MySQL 网络交互过程
分为以下阶段：
1. 握手阶段（连接阶段，客户端与服务器端交互）
2. 从服务器到客户端：握手初始化包
3. 客户端到服务器：客户端认证包
4. 服务器端到客户端：OK包、错误（error）包
5. 客户端到服务器端：命令包
6. 服务器到客户端：OK包、错误（error）包、结果集包
7. 客户端到服务器段：断开请求
8. tcp四次挥手断开连接
![MySQL 网络交互过程](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F1.png)
[MySQL协议分析](https://www.cnblogs.com/davygeek/p/5647175.html)<p>
[MySQL网络协议分析](https://segmentfault.com/a/1190000012166738?utm_source=tag-newest)
## 协议和操作系统协议栈
- 有时候mysql net 几个包放同一个tcp/ip包中，有时候一个mysql net包横跨几个tcp/ip包
- mysql调用net_serv.cc中的my_net_write写到缓冲中
 MAX_PACKET_LENGTH = 256*256*256 -1
```
my_bool
 my_net_write(NET *net,const uchar *packet,size_t len)
 {
   uchar buff[NET_HEADER_SIZE];
   if (unlikely(!net->vio)) /* nowhere to write */
     return 0;
   /*
     Big packets are handled by splitting them in packets of MAX_PACKET_LENGTH
     length. The last packet is always a packet that is < MAX_PACKET_LENGTH.
     (The last packet may even have a length of 0)
   */
   while (len >= MAX_PACKET_LENGTH)
   {
     const ulong z_size = MAX_PACKET_LENGTH;
     int3store(buff, z_size);
     buff[3]= (uchar) net->pkt_nr++;
     if (net_write_buff(net, buff, NET_HEADER_SIZE) ||
     net_write_buff(net, packet, z_size))
       return 1;
     packet += z_size;
     len-=     z_size;
   }
   /* Write last packet */
   int3store(buff,len);
   buff[3]= (uchar) net->pkt_nr++;
   if (net_write_buff(net, buff, NET_HEADER_SIZE))
     return 1;
 #ifndef DEBUG_DATA_PACKETS
   DBUG_DUMP("packet_header", buff, NET_HEADER_SIZE);
 #endif
   return test(net_write_buff(net,packet,len));
 }
 
```
- 如果数据包需要马上发给客户端，则sql/net_serv.cc中的net_flush会被调用

```
 
 my_bool net_flush(NET *net)
 {
   my_bool error= 0;
   DBUG_ENTER("net_flush");
   if (net->buff != net->write_pos)
   {
     error=test(net_real_write(net, net->buff,
                   (size_t) (net->write_pos - net->buff)));
     net->write_pos=net->buff;
   }
   /* Sync packet number if using compression */
   if (net->compress)
     net->pkt_nr=net->compress_pkt_nr;
   DBUG_RETURN(error);
 }
 
```
## 网络包格式
- MySQL NET 数据包分为压缩与非压缩，在双方==握手阶段==协商决定
#### null结尾字符串和带长度标识字符串
- null结尾字符串：字符串以ASCII码“\0”标识null
- 带长度标识字符串（Length coded string）有两个部分：长度定义部分和内容部分

第一字节值 |后续字节数| 说明
---|---|---
0~250|0|这个值说明了Length Coded String后面有多少字节数据<p>这种方式限制了后续数据最多有250个字节
251|0| 列值为NULL,仅仅用于行数据包
252|0|后面两个字节的值,说明Length Coded String后面有多少字节数据
253|3|后面三个字节的值,说明Length Coded String后面有多少字节数据
254| 8|后面八个字节的值,说明Length Coded String后面有多少字节数据

#### 网络包头部格式
所有的MysQL NET数据包都含有一个4字节长度的包头，如下所示：

字节数| 名称| 描述
---|---|---
3|MySQL NET数据包长度|数据包长度记录了数据包头部之后的数据长度.根据2^24, 我们可以得出一个包最大长度为16MB
1|数据包序列号|和大部分网络包-一样,TCP/IP 无法保证收到的顺序,所以 需要应用层次定义顺序以保证数据按预期收到

- 早期mysql（<4.0），规定数据包长度只有3字节，所以最大长度为16M
- 4.0后，如果数据包超过MAX_PACKET_LENGTH(2^24-1)包被分割成（datalength/MAX_PACKET_LENGTH）+1个不超过MAX_PACKET_LENGTH大小的包
- 如果服务器采用压缩包方式，则数据包头含有额外三个字节用于记录被压缩包的数据长度
- 压缩使用zlib/compress.c的compress()函数


#### 客户端认证请求包
- 客户端认证请求包的格式

字节数 |字段名|说明
---|---|---
4| 客户端标志|与服务器端能力描述有相同作用<p>也是用位图的形式表述客户端能够接受的连接选项<p>所有选项都可以在include/mysq. com.h中找到,以CLIENT开头<p>分为客户端部分和扩展部分，分别为2字节
4| 包最大长度(Max packet size)|一个包能包容的最大长度
1 |客户端字符集ID|MySQL 客户端所使用的字符集ID
23| 填充字符[0x00)|是以Null结尾的字符串
N |用户名|存放试图登录的用户名
1+N| 密码加密字段(scramble. buf)|是带长度标识的字符串(Length Coded String)
N |数据库名(可选)|数据库的名称字段
客户端标志示例
![客户端标志示例](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F2.png)

- libmysq/libmysql.c::mysql_real_connect()定义了MySQL客户端的认证过程
- sql/sql_connect.c::check_connections()中定义了服务器端认证用户的过程


#### 命令包
命令包格式
字节数|字段名
---|---
1|命令
N|命令参数

- show global status like "com_%"

```
> show global status like "com_%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Com_admin_commands         | 0     |
| Com_alter_db               | 0     |
| Com_alter_db_upgrade       | 0     |
| Com_alter_event            | 0     |
| Com_alter_function         | 0     |
| Com_alter_procedure        | 0     |
| Com_alter_server           | 0     |
| Com_alter_table            | 0     |
| Com_alter_tablespace       | 0     |
| Com_analyze                | 0     |
| Com_assign_to_keycache     | 0     |
| Com_begin                  | 0     |
| Com_binlog                 | 0     |
| Com_call_procedure         | 0     |
| Com_change_db              | 0     |
| Com_change_master          | 0     |
| Com_check                  | 0     |
| Com_checksum               | 0     |
| Com_commit                 | 483   |
| Com_create_db              | 0     |
| Com_create_event           | 0     |
| Com_create_function        | 0     |
| Com_create_index           | 0     |
| Com_create_procedure       | 0     |
| Com_create_server          | 0     |
| Com_create_table           | 0     |
| Com_create_trigger         | 0     |
| Com_create_udf             | 0     |
| Com_create_user            | 0     |
| Com_create_view            | 0     |
| Com_dealloc_sql            | 0     |
| Com_delete                 | 0     |
| Com_delete_multi           | 0     |
| Com_do                     | 0     |
| Com_drop_db                | 0     |
| Com_drop_event             | 0     |
| Com_drop_function          | 0     |
| Com_drop_index             | 0     |
| Com_drop_procedure         | 0     |
| Com_drop_server            | 0     |
| Com_drop_table             | 0     |
| Com_drop_trigger           | 0     |
| Com_drop_user              | 0     |
| Com_drop_view              | 0     |
| Com_empty_query            | 0     |
| Com_execute_sql            | 0     |
| Com_flush                  | 0     |
| Com_grant                  | 0     |
| Com_ha_close               | 0     |
| Com_ha_open                | 0     |
| Com_ha_read                | 0     |
| Com_help                   | 0     |
| Com_insert                 | 0     |
| Com_insert_select          | 0     |
| Com_install_plugin         | 0     |
| Com_kill                   | 0     |
| Com_load                   | 0     |
| Com_lock_tables            | 0     |
| Com_optimize               | 0     |
| Com_preload_keys           | 0     |
| Com_prepare_sql            | 0     |
| Com_purge                  | 0     |
| Com_purge_before_date      | 0     |
| Com_release_savepoint      | 0     |
| Com_rename_table           | 0     |
| Com_rename_user            | 0     |
| Com_repair                 | 0     |
| Com_replace                | 0     |
| Com_replace_select         | 0     |
| Com_reset                  | 0     |
| Com_resignal               | 0     |
| Com_revoke                 | 0     |
| Com_revoke_all             | 0     |
| Com_rollback               | 0     |
| Com_rollback_to_savepoint  | 0     |
| Com_savepoint              | 0     |
| Com_select                 | 12709 |
| Com_set_option             | 5644  |
| Com_show_authors           | 0     |
| Com_show_binlog_events     | 0     |
| Com_show_binlogs           | 0     |
| Com_show_charsets          | 1     |
| Com_show_client_statistics | 0     |
| Com_show_collations        | 1     |
| Com_show_contributors      | 0     |
| Com_show_create_db         | 0     |
| Com_show_create_event      | 0     |
| Com_show_create_func       | 0     |
| Com_show_create_proc       | 0     |
| Com_show_create_table      | 0     |
| Com_show_create_trigger    | 0     |
| Com_show_databases         | 2     |
| Com_show_engine_logs       | 0     |
| Com_show_engine_mutex      | 0     |
| Com_show_engine_status     | 0     |
| Com_show_errors            | 0     |
| Com_show_events            | 0     |
| Com_show_fields            | 24    |
| Com_show_function_status   | 0     |
| Com_show_grants            | 0     |
| Com_show_index_statistics  | 0     |
| Com_show_keys              | 0     |
| Com_show_master_status     | 0     |
| Com_show_open_tables       | 0     |
| Com_show_plugins           | 0     |
| Com_show_privileges        | 0     |
| Com_show_procedure_status  | 0     |
| Com_show_processlist       | 0     |
| Com_show_profile           | 0     |
| Com_show_profiles          | 0     |
| Com_show_relaylog_events   | 0     |
| Com_show_slave_hosts       | 0     |
| Com_show_slave_status      | 0     |
| Com_show_status            | 2     |
| Com_show_storage_engines   | 0     |
| Com_show_table_statistics  | 0     |
| Com_show_table_status      | 0     |
| Com_show_tables            | 1     |
| Com_show_triggers          | 0     |
| Com_show_user_statistics   | 0     |
| Com_show_variables         | 4     |
| Com_show_warnings          | 0     |
| Com_signal                 | 0     |
| Com_slave_start            | 0     |
| Com_slave_stop             | 0     |
| Com_stmt_close             | 0     |
| Com_stmt_execute           | 0     |
| Com_stmt_fetch             | 0     |
| Com_stmt_prepare           | 0     |
| Com_stmt_reprepare         | 0     |
| Com_stmt_reset             | 0     |
| Com_stmt_send_long_data    | 0     |
| Com_truncate               | 0     |
| Com_uninstall_plugin       | 0     |
| Com_unlock_tables          | 0     |
| Com_update                 | 483   |
| Com_update_multi           | 0     |
| Com_xa_commit              | 0     |
| Com_xa_end                 | 0     |
| Com_xa_prepare             | 0     |
| Com_xa_recover             | 0     |
| Com_xa_rollback            | 0     |
| Com_xa_start               | 0     |
| Compression                | OFF   |
+----------------------------+-------+
```
#### 握手初始化包
- 握手初始化包格式

字节数| 字段名|说明
---|---|---
1 | 协议版本号|系统协议版本号在include/mysql_version.h的PROTOCOL_VERSION常量中定义
n=strlen(server version)+1 |服务器信息|也可以在include/mysql_version.h文件中找到。MYSQL_SERVER_VERSION常量中保留着服务器版本信息
4 |线程ID（原文写的是进程id，有误）|MySQL服务器为此连接分配的线程号
8|密码验证1|与密码验证2-起共同用做密码验证
1|0x00填充位|
2|服务器能力描述|MySQL用比特位图的形式表述服务器能够接受的连接选项。所有选项都可以在include/mysql_com.h中找到,以CLIENT开头
1|服务器字符集|
2| 服务器状态|
13 |0x00填充位|
13| 密码验证2|
![握手包示例](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F3.png)

#### 结果包
对于客户端的认证包或者命令包的请求，服务器段都将发送一个回应包，可能是OK包、ERROR包、命令的结果集包等

结果包的类型 |FIELD_COUNT字段内容| 例包
---|---|---
OK包 |0x00|
ERROR包 |0xff|
结果集包|1-250 |Select * from table1的包
属性包|1-250| Select 1+1返回的包
行数据包| 1-250|
EOF包| 0xfe|
###### OK包
-  MySQL服务器成功执行了一个命令后,它将回复OK包，OK包适合于不用返回大量结果集的情况，承载少许信息量。OK包常常是对以下各种客户端命令的回复：
1. COM_PING
2. COM_QUERY(此处Query是更加广泛意义上的查询,包含SQL语句中的INSERT、UPDATE和DELETE等)
3. COM_REFRESH
4. COM_REGISTER_SLAVE

字节数 |字段|说明
---|---|---
1 |FIELD_COUNT|意指该包是0K包,总为零
1~9 |命令影响的行数|命令DELETE、INSERT 和UPDATE影响的行数
1~9 |插入ID|如果该操作引起任何AUTO INCREMENT的作用,那么插入ID会记录最后一个AUTO INCREMENT的结果
2| 服务器状态|
2| 警告数量|
N |消息|执行语句后,在返回的结果后有一段总结性信息，如2 rows affected

![OK包示例](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F4.png)
- 使用sql/protocol.cc中的net_send_ok()函数

###### ERROR包
一旦MySQL服务器处理命令出错,或者用户的认证信息有问题,那么MySQL会传递ERROR包，格式如下：
字节数 |字段|说明
---|---|---
1| 其值总为0xff|
2 |错误号|在include/mysqld error.h 中定义了内部错误号:<p>#define ER_ERROR_FIRST 1000<p>#define ER_HASHCHK 1000<p>#define ER_NISAMCHK 1001<p>#define ER_NO 1002<p>#define ER YES 1003<p>#define ER CANT_CREATE_FILE 1004<p>#define ER_CANT_CREATE_TABLE 1005<p>#define ER_CANT_CREATE_DB 1006<p>#define ER_DB_CREATE_EXISTS 1007<p>#define ER_DB_DROP_EXISTS 1008<p>
1 |SQL状态标识符,总是'#'|用于区分状态出资4.1以及以后版本
5 |SQL状态|每个错误号对应了一个错误状态。<p>该对应任务由mysql_errno_to_sqlstate()函数完成。<p>SQL状态在include/sql state.h 中有所记录:<p>ER_DUP_KEY . , "23000",""<p>ER_OUTOFMEMORY ,"HY001","S1001"
N| 消息|发生错误的原因，长度512字节<p>发送error的函数net_send_error_packet()在sql/protocol.cc中

###### 结果集包
- 客户端90%语句是查询语句，查询语句中 90%返回一个结果集，结果基本都超过一个单元格。
- 结果集包结构
![结果集包结构](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F5.png)
1. 结果集包头部
字节数|字段名|说明
---|---|---
1~9|FIELD_COUNT|记录返回集的列数
1~9|附属字段|可选，show columns才用到
![结果集包头](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F6.png)
2. 列包（属性包）

列包格式
字节数|字段名|说明
---|---|---
N(带长度标识字符串)| 分类|分类字段.MySQL内部使用.在5.0和5.1中均为def
N(带长度标识字符串)| 数据库|数据库描述,即数据库名称
N(带长度标识字符串)| 表|数据库表描述,即数据库表名
N(带长度标识字符串] |原表|as左侧的表名，即原表名
N(带长度标识字符串] |列名称|as右侧的列名，即别名
N(带长度标识字符串) |原列名称|as左侧的列名
1| 填充|
2 |字符集编号|每类字符集均由字符集ID表示
4 |长度|列的长度,即定义列时使用的显示长度.Varchar(5)的长度总是为5,不管这个变量中存了多少数据
1| 类型|列数据类型.MySQL的列可以多达20多种类型<p>include/mysqL com.h中定义了所有类型的编号<p>为了向后兼容,采用了宏定义:<p>#define CLIENT_MULTI_QUERIES CLIENT_MULTI_STATEMENTS<p>#define FIELD_TYPE_DECIMAL MYSQL_TYPE_,DECIMAL<p>#define FIELD_TYPE_NEWDECIMAL MYSQL_TYPE_NEWDECIMAL<p>
2 |标志位(flags)|列的其他可能定义,如primary key、zerofill 等<p>#define NOT_ NULL_ FLAG 1 /*字段不能为空(NULL) */<p>#define PRI_ KEY_ FLAG 2 /*这个字段是主键*/<p>#define UNIQUE_ KEY_ FLAG 4 1*这个字段是唯一键*/<p>#define MULTIPLE_ KEY_ FLAG 8 /*这个字段是索引的一部分*/<p>#define BLOB_ FLAG 16 /*字段是blob类型*/<p>#define UNSIGNED_ PLAG 32 /*字段是unsigned的*/<p>#define ZEROFILL_ ,FLAG 64 /*字段采用零值填充*/<p>#define BINARY_ FLAG 128 /*字段是二进制的*/<p>
1| 十进制|对于Decimal或Numeric类型的数据,该字段记录了小数点后面的位数
2| 填充(总是0x00)
N(带长度标识字符串]| 默认值|仅用于数据表定义.当用户执行show columns from tablea 时,并不返回这个字段的任何值
![列包格式](pic/MySQL%20%E8%BF%9E%E6%8E%A5%E5%92%8C%E7%BD%91%E7%BB%9C%E7%B3%BB%E7%BB%9F7.png)
整个包有FIELD_COUNT个列包
3. 行包

MySQL发送完列包之后，发送行包

4. EOF 包
EOF是End Of File的缩写，在MySQL的网络环境下表示某种类型的包传输已经结束,所以EOF包出现在列(属性)包之后,也出现在行数据包之后

 EOF 包的包格式
字节数 |字段|说明
---|---|---
1| FIELD_COUNT|总是=0xfe
2 |警告数量|在传递完所有数据包时,显示所有警告(Warning)的数量
2 |状态标志符|包含各类服务器的状态,如SERVER STATUS MORE RESULTS

MySQL服务器调用write_eof_packet()将数据写到网络输出缓冲区,然后再调用send_eof()函数发送该包