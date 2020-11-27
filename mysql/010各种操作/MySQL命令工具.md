MySQL还有自带了其他大量的工具程序，如

1. 针对离 线Innodb文件做checksum的innochecksum，
2. 转换mSQL C API函数的msql2mysql， 
3. dumpMyISAM全文索引的myisam_ftdump，
4. 分析处理slowlog的mysqldumpslow，
5. 查询mysql 相关开发包位置和include文件位置的mysql_config， 
6. 向MySQLAB报告bug的mysqlbug， 
7. 测试套件 mysqltest 和 mysql_client_test，
8. 批量修改表存储引擎类型的 mysql_convert_table_format，
9. 能从更新日志中提取给定匹配规则的query语句的 mysql_find_rows，
10. 更改MyIsam存储引擎表后缀名的mysql_fix_extensions，
11. 修复系统表 的mysql_fix_privilege_tables，
12. 查看数据库相关对象结构的mysqlshow，
13. MySQL升级工具 mysql_upgrade，
14. 通过给定匹配模式来kill客户端连接线程的mysql_zap，
15. 查看错误号信息 的perror，
16. 文本替换工具replace，
17. 如果您希望在MySQL 源代码的基础上做一些自己的修改，如修改MyISAM存储引擎的时候，可以利用myisamlog 来进行跟踪分析MyISAM的log。



本文摘取《MySQL性能调优与架构设计》