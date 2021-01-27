

## MySql的flush用法

MySQL的FLUSH(清除或者重新加载内部缓存） FLUSH flush_option [,flush_option]，如果你想要清除一些MySQL使用内部缓存，你应该使用FLUSH命令。为了执行FLUSH，你必须有**reload**权限。
flush_option 可以是下列参数：

选项|说明
---|---
HOSTS  | ==这个用的最多==，主要是用来清空主机缓存表。<br/>如果你的某些主机改变IP数字，或如果你得到错误消息Host ... is blocked，你应该清空主机表。<br/>当在连接MySQL服务器时，对一台给定的主机有多于 max_connect_errors个错误连续不断地发生，MySQL为了安全的需要将会阻止该主机进一步的连接请求。清空主机表允许主机再尝试连接。 
LOGS   | 关闭当前的二进制日志文件并创建一个新文件，新的二进制日志文件的名字在当前的二进制文件的编号上加1。 <br>mysql中flush logs操作会生成一个新的binlog文件。<br/>如果在==从库==执行flush logs 不仅会生成一个新的binlog文件，而且会生成一个新的relaylog文件。<br/>flush logs 还影响slow log和general log,当删除slow log或者general log,然后执行flush logs，此时会再重新生成一个新的slow log或者general log 
PRIVILEGES | 这个也是经常使用的，每当重新赋权后，为了以防万一，让新权限立即生效，目的是从数据库授权表中重新装载权限到缓存中。 
TABLES |   关闭所有打开的表，同时该操作将会清空查询缓存中的内容。
FLUSH TABLES WITH READ LOCK |关闭所有打开的表，同时对于所有数据库中的表都加一个读锁，直到显式地执行unlock tables，该操作常常用于数据备份的时候。<br/>解锁的语句就是unlock tables。<br/>FLUSH TABLES WITH READ LOCK对于数据库是全局的表锁定，如果只想锁定几个表，可以用LOCK TABLES tbl_name [AS alias] {READ [LOCAL] 
STATUS |   重置大多数状态变量到0。
MASTER  | 重置二进制日志文件的索引文件为空，创建一个新的二进制日志文件。<br/>不过这个已经==不推荐使用==，改成==reset master== 了。 
QUERY CACHE |重整查询缓存，消除其中的碎片，提高性能，但是并不影响查询缓存中现有的数据<br/>这点和Flush table 和Reset Query  Cache（将会清空查询缓存的内容）不一样的。 
SLAVE  | 让从数据库忘记主数据库的复制位置，同时也会删除已经下载下来的relay log<br/>与Master一样，已经==不推荐使用==，改成==Reset Slave==了。 

一般来讲，Flush操作都会记录在二进制日志文件中，但是FLUSH LOGS、FLUSH MASTER、FLUSH SLAVE、FLUSH TABLES WITH READ LOCK不会记录，因此上述操作如果记录在二进制日志文件中话，会对从数据库造成影响。

注意：Reset操作其实扮演的是一个Flush操作的增强版的角色。





参考：https://blog.csdn.net/w892824196/article/details/80641850