 本文链接：https://blog.csdn.net/edyf123/article/details/81026139 

在mysql关闭时，参数innodb_fast_shutdown 影响着表的存储引擎为innodb的行为。参数为0,1,2三个值。

0，代表当MYSQL关闭时，Innodb需要完成所有full purge和merge insert buffer操作，这需要花费时间来完成。如果做Innodb plugin升级，通常需要将这个参数调为0,，然后在关闭数据库
1， 是参数的默认值，不需要完成full purge和merge insert buffer操作，但是在缓冲池的一些数据脏页还是会刷新到磁盘。
2   表示 不需要完成full purge和merge insert buffer操作 ，也不将缓冲池中的数据脏页写回磁盘，。而是将日志都写入日志文件。这样不会有任何事物丢失，但是mysql在下次启动时，会执行恢复操作（recovery）

如果在上次关闭innodb的时候是在innodb_fast_shutdown=2或是mysql crash这种情况，那么它会利用redo log重做那些已经提交了的事务。
接下来的操作过程是：
(1). Rollback uncompleted transitions 取消那些没有提交的事务
(2). Purge all 清除无用的undo页
(3). Merge insert buffer 合并插入缓冲
