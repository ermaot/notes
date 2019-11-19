本文来自https://yq.aliyun.com/articles/41056
## 简介
- 除了Performance Schema外，在MySQL 5.6中还提供了一个新的information_schema表来监控Innodb的内部运行状态——INNODB_METRICS；该表维护了一组计数器，用户可以通过这些计数器，来监控Innodb内部运行是否健康
- MySQL5.6.12版本中，共有210个计数器;mysql8.0.17-debug中，有299个
```
> select count(*) from INNODB_METRICS;
+----------+
| count(*) |
+----------+
|      299 |
+----------+
1 row in set (0.00 sec)
```

## 打开或者关闭
- 1.打开计数器：
```
mysql> set global innodb_monitor_enable = ‘adaptive_hash_%';
```

- 2.关闭计数器：
```
mysql> set global innodb_monitor_disable = ‘adaptive_hash_%';

Query OK, 0 rows affected (0.00 sec)
```
- 3.重置AHI所有列的值：
```
mysql> set global innodb_monitor_reset_all = “adaptive_hash_%”;
Query OK, 0 rows affected (0.00 sec)
```
- 4.只重置COUNTER的值：
```
mysql> set global innodb_monitor_reset = “adaptive_hash_%”;
Query OK, 0 rows affected (0.00 sec)
```
- 5.根据模块名打开：
```
mysql> set global innodb_monitor_enable = module_adaptive_hash;
Query OK, 0 rows affected (0.00 sec)
```
- 6.打开所有计数器：
```
mysql>  set global innodb_monitor_enable = all;
Query OK, 0 rows affected (0.00 sec)
```
- 7.关闭所有计数器：
```
mysql> set global innodb_monitor_disable =  all;
Query OK, 0 rows affected (0.00 sec)
```