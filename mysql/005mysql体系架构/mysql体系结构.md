## mysql体系结构
参考来源：
- MYSQL技术内幕：姜承尧
- MySQL实战45讲：：丁奇（林晓斌）-极客时间
### mysql物理结构
1. 连接池
1. 管理服务于工具
1. SQL接口
1. 查询分析（语法分析，词法分析）
1. 优化器
1. 缓存
1. 存储引擎
1. 物理文件
![姜承尧-mysql物理结构](FF3703C24C6A4C51AC1540821DF67C24)
对照
### mysql执行流程
![丁奇-mysql结果与执行流程](https://github.com/ermaot/notes/blob/master/mysql/005mysql%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84/pic/mysql%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%842.jpg)
### 存储引擎
#### - innodb（==最常用==）
1. 支持事务，面向OLTP
1. 行锁（相对于MyISAM的表所）
1. 支持外键
2. 非锁定读
3. 数据放逻辑表空间
4. 可将表放独立ibd文件中（MySQL>4.1）
5. 支持裸设备
6. MVCC
7. SQL标准==4种隔离级别==，REPEATABLE为默认，==next-key locking==
8. 插入缓冲（insert buffer），二次写（double write），自适应哈希索引（adaptive hash index），预读（read ahead）
9. 聚集索引（表按主键顺序存储）
10.若无主键，自动6字节ROWID作为主键
#### - MyISAM
1. 不支持事务
1. 表锁（相对于innodb）
1. 支持全文索引
1. 缓存索引文件而非数据文件
1. 面向OLAP