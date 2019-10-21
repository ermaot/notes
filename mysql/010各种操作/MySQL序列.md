## 查询序列值
- 返回本次连接服务器后最新创建的auto_increment值
- 如果有多个序列的表进行插入，可能会串
- truncate之后序列值会重置
- last_insert_id是针对于会话的，也就是不同会话即使操作同一个表，last_insert_id也可能不同
```
> select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                1 |
+------------------+
1 row in set (0.00 sec)
```
## 重新序列化
1. 删除自增列，再重建（两个语句）

```
alter table insect drop id;
alter table insect add id int unsigned not null auto_increment first,add primary key(id);
```

2. 删除自增列，再重建（一个语句）
MySQL允许在一个alter table进行多个操作（此处为一个原子操作）
```
alter table insect drop id,add id int unsigned not null auto_increment first
```
## 序列顶部数值再使用

```
alter tabler test auto_increment = 1
```
## 设定自增初始值

```
CREATE TABLE t(id INT UNSIGNED NOT NULL AUTO_INCREMENT, PRIMARY KEY (id) )AUTO_INCREMENT = 100;
```
