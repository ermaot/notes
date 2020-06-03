postgresql的索引非常灵活，支持的类型很多。MySQL不支持函数索引等

## 表达式索引

postgresql支持函数索引，也可以是一个或者多个字段计算出来的标量表达式

```
# create table func_idx_test(a varchar(16));
CREATE TABLE

# create unique index idx_func on func_idx_test(lower(a));
CREATE INDEX
```

此时如果插入字符相同但仅仅是大小写不同的字符串，会导致索引重复，所以不能插入

```
# insert into func_idx_test values('test');
INSERT 0 1

# insert into func_idx_test values('Test');
ERROR:  duplicate key value violates unique constraint "idx_func"
DETAIL:  Key (lower(a::text))=(test) already exists.
```

## 部分索引

部分索引是指对表中部分的行做索引。通过部分索引的**谓词**把部分行筛选出来。

##### 设置一个部分索引以排除普通数值

```
# create table func_idx_test2(a  int);
CREATE TABLE
# create unique index partial_idx_func on func_idx_test2(a) where a>100 and a<200;
CREATE INDEX
```

##### 设置部分索引以排除不感兴趣的数值

当表中某一部分的数据占总数的一小部分并且经常使用，则可以只在该部分的数据创建索引改善性能



**postgresql支持带任意谓词的部分索引，只要涉及被索引的表的字段就行。

## GIST索引

GIST索引是generalized search tree的缩写，就是通用查找树，即一种平衡树结构的访问方法，是用户建立自定义索引的基础模板、B-Tree和许多其他的索引都可以用GIST索引实现。GIST即一个高级的变成接口，提供了一个框架，处理并发，WAL日志和搜索树结构处理任务，只需要实现者实现被访问的数据类型的回调函数即可。

## SP-GiST索引

SP-GiST是space-partitioned GiST索引的缩写，即空间分区的GiST索引，从9.2开始提供

## GIN索引

GIN索引即Generalized Inverted Index，广义倒排索引。GIN索引存储 了一系列的“key，位置列表”对。同一行的rowid会出现在多个位置列表中。