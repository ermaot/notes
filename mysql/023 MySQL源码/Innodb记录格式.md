## 显示记录格式
```
> show table status like "%test%"\G
*************************** 1. row ***************************
           Name: test0906
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 2
 Avg_row_length: 8192
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2019-09-06 17:02:05
    Update_time: 2019-09-06 17:01:32
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.01 sec)
```
Row_format显示的是存储格式，MySQL5.0引入compat格式，用得比较多

## 源码分析格式
MySQL有3个层次的存储格式
- server层的格式：与引擎无关，适用于所有引擎。操作数据时必然要经过的一种格式，row模式下的binlog存储所使用的格式
- 索引元组格式：Innodb存储引擎在存取记录时一种中间状态。它是Innodb在内存中的一种元组格式，==不同索引对应的元组不同==（？）
- 物理存储格式。这种格式与索引元组一一对应。每次存取，都从server层格式转换为索引元祖格式，再转换为页面上的索引记录物理格式。
![compat格式](pic/Innodb%E8%AE%B0%E5%BD%95%E6%A0%BC%E5%BC%8F.png)
1. 首部是变长字段长度列表，最大为2字节（所以mysqlvarchar最大长度为2^16-1 = 65535）
2. NULL标志位，所用字节为byte
3. 记录头

名称 | 大小（位） | 描述
---|--- | ---
（） | 1 | 未知
（） | 1 | 未知
delete_flag | 1 | 该行是否已经被删除
min_rec_flag | 1 | 最小的记录标志
n_owned | 4 | 该记录拥有的记录数
heap_no | 13 | 索引堆中该记录的排序记录
recored_type | 3 | 记录类型：000=普通 001=B+树节点指针 010=infimum 011=supermum 1xx=保留
next_recorder | 16 | 页中下一条记录的相对未知
合计 | 40 | 即5个字节

隐藏列：两个（事务ID列（6个字节）和回滚指针列（7个字节））<p>