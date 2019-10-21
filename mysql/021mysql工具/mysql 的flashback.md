目前MySQL的flashback功能是利用binlog完成的，第一个实现该功能的是阿里云的彭立勋， 他在MySQL 5.5版本上就已实现，并将其提交给MariaDB。
https://www.cnblogs.com/waynechou/p/mysql_flashback_intro.html

美团点评又出了另一款闪回工具MyFlash，据说比 mysqlbinlog 具有更高效的闪回效果，项目地址：https://github.com/Meituan-Dianping/MyFlash