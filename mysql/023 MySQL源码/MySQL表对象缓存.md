本文来自MySQL运维内参

MySQL中，有很多的系统对象，比如表、视图、存储过程、存储函数等，每一种对象缓存方式不同，不是通过统一的方式管理

- 表对象缓存，是将某个表对象的字典信息缓存到内存中，提高表对象的访问效率
- 查询表的数据时，会先找到这个表缓存。表缓存是通过HASH表（源码table_def_cache）来管理的 
- MySQL是插件式数据库，每一个用户得到表对象之后需要将表实例化，所以并非所有用户都使用同一个缓存对象
- MySQL表对象缓存使用“共享私有化缓存”（源码TABLE_SHARE结构体）。打开表的时候，会将表的所有信息读入内存，包括表名、模式名、所有列信息、列默认值、表字符集、对应.frm文件路径、所属存储引擎、主键等
- 表信息通过TABLE_SHARE存储，所有用户可以共享，静态不允许修改，直到表从缓存中删除
- 表构造了TABLE_SHARE后被缓存，然后计算得到key，进入到table_def_cache，用户访问的时候，TABLE_SHARE会被实例化成TABLE对象（动态可操作的实例）
- **TABLE_SHARE服务于MySQL的server层，而TABLE对象服务于存储引擎层**
- 对表的操作完成后，在内存中保留下来，进入SHARE的free_tables链表中，下次访问的时候可以直接使用
- 表对象缓存有两个部分，**一部分是SHARE缓存(table_definition_cache控制)，另一部分是每个SHARE结构被实例化后的实例对象缓存**（table_open_cache控制）。
- MySQL管理表对象缓存空间大小的方法是计数，即系统SHARE总数不超多table_definition_cache（即参数table_definition_cache）
```
> show variables like "table_definition_cache";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| table_definition_cache | 2000  |
+------------------------+-------+
1 row in set (0.01 sec)

> show variables like "table_open_cache";
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| table_open_cache | 4000  |
+------------------+-------+
1 row in set (0.00 sec)
```
- 与表缓存相关的参数：table_open_cache和table_definition_cache
1. table_open_cache最小值400
```
//最大值2000，最小值400
default_value= min<ulong>(400+ table_cache_size/2, 2000)
```
2. table_cache_size计算，和innodb_open_files、max_connections有关系，很复杂，不写了^_^


## 优缺点总结
- 优点（==个人认为优点不是很大==）
1. 相比全字典缓存，降低DDL操作或者回滚导致的字典缓存维护工作代价
2. 有效利用了内存空间，按需载入，提高了内存利用率
- 缺点
1. 控制缓存空间大小是根据实例化表对象个数来计算，如果超过table_open_cache会根据算法淘汰不常用的实例化表对象。如果表定义很大，并发情况下就会建立多个实例化对象，可能将操作系统内存用光。对SHARE缓存也一样
2. 效率有一些代价。为了插件式引擎，中间加入了SHARE缓存，真正用到的时候还需要实例化，这样给CPU、内存带来一定程度上的压力
