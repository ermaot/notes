https://blog.csdn.net/he172073675/article/details/80453667

版本			select version();
ip，连接数		select substring_index(host,':',1) as ip , count(*) from information_schema.processlist group by ip;
运行时长（秒）	show global status like 'uptime';	mysql已经工作的秒数
当前用户		select user();
线程			show status like 'threads%';
连接线程		show status like 'threads_connected';
活动线程		show status like 'threads_running';
线程缓存		show variables like 'thread_cache_size'; 
最大线程数		show variables like 'max_connections';
响应的连接数		show global status like 'max_used_connections';   
事务			select * from information_schema.innodb_trx
慢查询			show variables like '%slow%'; 	show global status like '%slow%';	slow_queries（慢查询次数）
打开文件的数目		show status like 'open_files%';
总共能打开的文件数量	show variables like 'open_files_limit';
当前打开的表数		show global status like 'open_tables';
总共能打开的表数	show variables like 'table_open_cache';

-- 检查是否有死进程：
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX; 
-- 查看正在锁的事务 
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
-- 查看等待锁的事务 
 SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS
