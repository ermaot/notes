本文参考：https://my.oschina.net/actiontechoss/blog/3066374

https://blog.csdn.net/h2453532874/article/details/94475741

https://www.cnblogs.com/lshan/p/13064505.html

https://www.cnblogs.com/xyj179/p/11451593.html

JSON是一种轻量级的数据交互的格式，大部分NO SQL数据库的存储都用JSON。MySQL从5.7开始支持JSON格式的数据存储，并且新增了很多JSON相关函数。MySQL 8.0 又带来了一个新的把JSON转换为TABLE的函数JSON_TABLE，实现了JSON到表的转换



## json_keys

```
> set @ytt='{"name":[{"a":"ytt","b":"action"}, {"a":"dble","b":"shard"},{"a":"mysql","b":"oracle"}]}';
Query OK, 0 rows affected (0.00 sec)

> select json_keys(@ytt);
+-----------------+
| json_keys(@ytt) |
+-----------------+
| ["name"]        |
+-----------------+
1 row in set (0.00 sec)

> select json_keys(@ytt,'$.name[0]');
+-----------------------------+
| json_keys(@ytt,'$.name[0]') |
+-----------------------------+
| ["a", "b"]                  |
+-----------------------------+
1 row in set (0.00 sec)
```



## json_table

```
> select * from json_table(@ytt,'$.name[*]' columns (f1 varchar(10) path '$.a', f2 varchar(10) path '$.b')) as tt;
+-------+--------+
| f1    | f2     |
+-------+--------+
| ytt   | action |
| dble  | shard  |
| mysql | oracle |
+-------+--------+
3 rows in set (0.00 sec)


```

```
set @json_str1 = ' {
    "query_block": {
      "select_id": 1,
      "cost_info": {
        "query_cost": "1.00"
    },
    "table": {
      "table_name": "bigtable",
      "access_type": "const",
      "possible_keys": [
        "id"
    ],
     "key": "id",
     "used_key_parts": [
      "id"
    ],
     "key_length": "8",
     "ref": [
      "const"
    ],
     "rows_examined_per_scan": 1,
     "rows_produced_per_join": 1,
     "filtered": "100.00",
     "cost_info": {
       "read_cost": "0.00",
       "eval_cost": "0.20",
       "prefix_cost": "0.00",
       "data_read_per_join": "176"
   },
     "used_columns": [
       "id",
       "log_time",
       "str1",
       "str2"
     ]
   }
  }
}';

-------------------1-------------------
> select json_keys(@json_str1) as 'first_object';
+-----------------+
| first_object    |
+-----------------+
| ["query_block"] |
+-----------------+
1 row in set (0.00 sec)

-------------------2-------------------
> select json_keys(@json_str1,'$.query_block') as 'secont_object';
+-------------------------------------+
| secont_object                       |
+-------------------------------------+
| ["table", "cost_info", "select_id"] |
+-------------------------------------+
1 row in set (0.00 sec)

-------------------3-------------------
> select json_keys(@json_str1,'$.query_block.table') as 'third_object'\G
*************************** 1. row ***************************
third_object: ["key", "ref", "filtered", "cost_info", "key_length", "table_name", "access_type", "used_columns", "possible_keys", "used_key_parts", "rows_examined_per_scan", "rows_produced_per_join"]
1 row in set (0.00 sec)

-------------------4-------------------
> select json_extract(@json_str1,'$.query_block.table.cost_info') as 'forth_object'\G
*************************** 1. row ***************************
forth_object: {"eval_cost": "0.20", "read_cost": "0.00", "prefix_cost": "0.00", "data_read_per_join": "176"}
1 row in set (0.00 sec)

-------------------5-------------------
SELECT * FROM JSON_TABLE(@json_str1,
         "$.query_block"
         COLUMNS(
           rowid FOR ORDINALITY,
       NESTED PATH '$.table' 
       COLUMNS (
           a1_1 varchar(100) PATH '$.key',
           a1_2 varchar(100) PATH '$.ref[0]',
           a1_3 varchar(100) PATH '$.filtered',
           nested path '$.cost_info' 
           columns (
                a2_1 varchar(100) PATH '$.eval_cost' ,
                a2_2 varchar(100) PATH '$.read_cost',
                a2_3 varchar(100) PATH '$.prefix_cost',
                a2_4 varchar(100) PATH '$.data_read_per_join'       )
       )
      )
    ) AS tt;
```



## json_extract

```
> select  JSON_EXTRACT('{"keywords": ["我", "想", "赚钱"]}','$.keywords') ;
+---------------------------------------------------------------------+
| JSON_EXTRACT('{"keywords": ["我", "想", "赚钱"]}','$.keywords')     |
+---------------------------------------------------------------------+
| ["我", "想", "赚钱"]                                                |
+---------------------------------------------------------------------+
1 row in set (0.00 sec)
```



## json_contains

```
> create table test_json(a json);
Query OK, 0 rows affected (0.03 sec)
> insert into test_json values('{"keywords":["我","想","赚钱"]}');
Query OK, 1 row affected (0.01 sec)
> select * from test_json;
+----------------------------------------+
| a                                      |
+----------------------------------------+
| {"keywords": ["我", "想", "赚钱"]}     |
+----------------------------------------+
1 row in set (0.00 sec)

> SELECT * FROM test_json  WHERE a->"$.keywords" like "%我%";
+---------------------------------------------+
| a                                           |
+---------------------------------------------+
| {"keywords": ["我", "想", "赚钱"]}          |
+---------------------------------------------+
1 rows in set (0.00 sec)

> SELECT * FROM test_json  WHERE JSON_CONTAINS(JSON_ARRAY("我", "想", "赚钱"),a->'$.keywords');
+----------------------------------------+
| a                                      |
+----------------------------------------+
| {"keywords": ["我", "想", "赚钱"]}     |
+----------------------------------------+
1 rows in set (0.00 sec)

> SELECT * FROM test_json  WHERE a like "%我%";
+---------------------------------------------+
| a                                           |
+---------------------------------------------+
| {"keywords": ["我", "想", "赚钱"]}          |
+---------------------------------------------+
1 rows in set (0.00 sec)
```



## json_set  JSON_INSERT JSON_REPLACE

```

JSON_SET(json_doc, path, val[, path, val] ...)
JSON_INSERT(json_doc, path, val[, path, val] ...)
JSON_REPLACE(json_doc, path, val[, path, val] ...)
```

JSON_SET() 替换已经存在的值，增加不存在的值。
JSON_INSERT() 新增不存在的值。
JSON_REPLACE() 替换/修改已经存在的值。

```
################### set #####################
> SELECT JSON_SET('{"a": 1, "b": 2}', '$.c', 3) AS 'Result';
+--------------------------+
| Result                   |
+--------------------------+
| {"a": 1, "b": 2, "c": 3} |
+--------------------------+
1 row in set (0.00 sec)

> SELECT JSON_SET('{"a": 1, "b": 2}', '$.a', 3) AS 'Result';
+------------------+
| Result           |
+------------------+
| {"a": 3, "b": 2} |
+------------------+
1 row in set (0.00 sec)


################### replace #####################
> SELECT JSON_REPLACE('{"a": 1, "b": 2}', '$.c', 3) AS 'Result';
+------------------+
| Result           |
+------------------+
| {"a": 1, "b": 2} |
+------------------+
1 row in set (0.00 sec)

> SELECT JSON_REPLACE('{"a": 1, "b": 2}', '$.a', 3) AS 'Result';
+------------------+
| Result           |
+------------------+
| {"a": 3, "b": 2} |
+------------------+
1 row in set (0.00 sec)

################### replace #####################
> SELECT JSON_inSErT('{"a": 1, "b": 2}', '$.a', 3) AS 'Result';
+------------------+
| Result           |
+------------------+
| {"a": 1, "b": 2} |
+------------------+
1 row in set (0.00 sec)

> SELECT JSON_inSErT('{"a": 1, "b": 2}', '$.c', 3) AS 'Result';
+--------------------------+
| Result                   |
+--------------------------+
| {"a": 1, "b": 2, "c": 3} |
+--------------------------+
1 row in set (0.00 sec)



```

## JSON_OBJECTAGG(key,value)

两个列名或表达式作为参数，第一个用作键，第二个用作值，并返回包含键值对的JSON对象。如果结果不包含任何行，或者出现错误，则返回NULL。如果任何键名称为NULL或参数数量不等于2，则会发生错误。

## JSON_ARRAYAGG(col or expr)

将结果集聚合为单个JSON数组，其元素由参数列的值组成。此数组中元素的顺序未定义。该函数作用于计算为单个值的列或表达式。异常返回NULL。

## json_array 创建json数组
json_object 创建json对象
json_quote 将json转成json字符串类型
查询json 
json_contains 判断是否包含某个json值
json_contains_path 判断某个路径下是否包json值
json_extract 提取json值
column->path    json_extract的简洁写法，MySQL 5.7.9开始支持
column->>path   json_unquote(column -> path)的简洁写法
json_keys 提取json中的键值为json数组
json_search 按给定字符串关键字搜索json，返回匹配的路径
修改json 
json_append 废弃，MySQL 5.7.9开始改名为json_array_append
json_array_append 末尾添加数组元素，如果原有值是数值或json对 象，则转成数组后，再添加元素
json_array_insert 插入数组元素
json_insert 插入值（插入新值，但不替换已经存在的旧值）
json_merge 合并json数组或对象
json_remove 删除json数据
json_replace 替换值（只替换已经存在的旧值）
json_set 设置值（替换旧值，并插入不存在的新值）
json_unquote 去除json字符串的引号，将值转成string类型
返回json属性 
json_depth 返回json文档的最大深度
json_length 返回json文档的长度
json_type 返回json值得类型
json_valid 判断是否为合法json文档