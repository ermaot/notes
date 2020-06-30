## 使用方法

show table status支持模糊匹配

```
show table status ;
show table status like "test_isa";
show table status like "%test_isam%";
```

## 字段解释

```
> show table status like "%%test_isam%"\G
*************************** 1. row ***************************
           Name: test_isam
         Engine: MyISAM
        Version: 10
     Row_format: Fixed
           Rows: 100
 Avg_row_length: 9
    Data_length: 900
Max_data_length: 2533274790395903
   Index_length: 1024
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2020-06-28 14:47:28
    Update_time: 2020-06-28 14:47:53
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)

```

字段|解释
---|---
Name|表名称
Engine:|表的存储引擎
Version:|版本
Row_format|行格式。对于MyISAM引擎，这可能是Dynamic，Fixed或Compressed。动态行的行长度可变，例如Varchar或Blob类型字段。固定行是指行长度不变，例如Char和Integer类型字段。
Rows|表中的行数。对于非事务性表，这个值是精确的，对于事务性引擎，这个值通常是估算的。
Avg_row_length|平均每行包括的字节数
Data_length|整个表的数据量(单位：字节)
Max_data_length|表可以容纳的最大数据量
Index_length|索引占用磁盘的空间大小
Data_free|对于MyISAM引擎，标识已分配，但现在未使用的空间，并且包含了已被删除行的空间。
Auto_increment|下一个Auto_increment的值
Create_time|表的创建时间
Update_time|表的最近更新时间
Check_time|使用 check table 或myisamchk工具检查表的最近时间
Collation|表的默认字符集和字符排序规则
Checksum|如果启用，则对整个表的内容计算时的校验和
Create_options|指表创建时的其他所有选项
Comment|包含了其他额外信息，对于MyISAM引擎，包含了注释徐标新，如果表使用的是innodb引擎 ，将现实表的剩余空间。如果是一个视图，注释里面包含了VIEW字样

## 等价使用方法

```
> > select * from information_schema.tables where table_schema = "perf" and TABLE_NAME='test_isam'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: perf
     TABLE_NAME: test_isam
     TABLE_TYPE: BASE TABLE
         ENGINE: MyISAM
        VERSION: 10
     ROW_FORMAT: Fixed
     TABLE_ROWS: 0
 AVG_ROW_LENGTH: 0
    DATA_LENGTH: 0
MAX_DATA_LENGTH: 2533274790395903
   INDEX_LENGTH: 1024
      DATA_FREE: 0
 AUTO_INCREMENT: NULL
    CREATE_TIME: 2020-06-28 14:47:28
    UPDATE_TIME: 2020-06-28 15:40:36
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS: 
  TABLE_COMMENT: 
1 row in set (0.00 sec)
```

