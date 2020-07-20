**题目**：在表t1中存储如下内容，列2中的值用固定分隔符”，“进行分割:

c1|c2
---|---
1|a,b
2|c,d,e
3|f,g
请编写SQL，运行并得到如下结果：
c1c3
---|---
1|a
1|b
2|c
2|d
2|e
3|f
3|g

**规则**：**不得使用存储过程和自定义函数**，MySQL版本不限。
```
SELECT a.c1,SUBSTRING_INDEX(SUBSTRING_INDEX(a.c2,',',b.help_topic_id+1),',',-1) as c3   from jiang  a left join mysql.help_topic b  on b.help_topic_id < (LENGTH(a.c2)-LENGTH(REPLACE(a.c2,',',''))+1);

+------+------+
| c1   | c3   |
+------+------+
|    1 | a    |
|    1 | b    |
|    2 | c    |
|    2 | d    |
|    2 | e    |
|    3 | f    |
|    3 | g    |
+------+------+
7 rows in set (0.00 sec)

```
这里使用了mysql的help_topic
```
> desc mysql.help_topic;
+------------------+----------------------+------+-----+---------+-------+
| Field            | Type                 | Null | Key | Default | Extra |
+------------------+----------------------+------+-----+---------+-------+
| help_topic_id    | int(10) unsigned     | NO   | PRI | NULL    |       |
| name             | char(64)             | NO   | UNI | NULL    |       |
| help_category_id | smallint(5) unsigned | NO   |     | NULL    |       |
| description      | text                 | NO   |     | NULL    |       |
| example          | text                 | NO   |     | NULL    |       |
| url              | text                 | NO   |     | NULL    |       |
+------------------+----------------------+------+-----+---------+-------+
6 rows in set (0.02 sec)

```