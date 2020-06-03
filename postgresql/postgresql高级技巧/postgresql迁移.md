## 迁移步骤

从Oracle迁移到postgresql，主要步骤如下：

1. 项目准备等前期工作：了解目的，评估风险，制定方案，组建团队
2. 数据库对象迁移：表、索引、序列、存储过程、触发器等，数据库不同对象的定义就不一样，需要重新定义代码改造
3. 应用代码改造：SQL语法的差异导致应用层实现会不同。功能不一样，会采用不同的实现方案
4. 数据迁移测试：验证迁移后数据准确性，完整性
5. 功能测试：对新系统进行功能测试
6. 性能测试：对系统做性能测试，看是否性能满足需求
7. 生产交割：做割接演练，记录数据迁移测试时间，停服时间，以及迁移步骤是否顺利

## 数据库对象迁移

#### 数据类型对应的适配

数据类型|Oracle|postgresql
---|---|---
字符类型|varchar、varchar2、N varchar、N varchar2|character varying、text、varchar
字符类型|char、nchar|character 
字符类型|clob、nlocb、long|text
数字类型|number|numeric
数字类型|float|real、double precision
时间类型|date|date或timestamp
时间类型|timestamp|timestamp
二进制类型|blob|bytea
- Oracle默认转换成大写，postgresql默认对象名称为小写
- postgresql不要用双引号

#### 存储过程

postgresql没有存储过程概念，可使用函数代替

## 应用代码

主要包括SQL代码改造和应用代码改造

#### SQL改造

1. 标准SQL，postgresql和Oracle差别不大
2. Oracle限制返回结果集记录数，使用rownum，而postgresql使用limit
3. Oracle的序列和postgresql的序列用法不同

```
# oracle
select seq_1.currval from dual;

#postgresql
select currval('seq_1')
```



4. oracle的子查询可以不用别名，而postgresql必须用

```
# oracle
select * from (select * from table_name);

#postgresql
select * from (select * from table_name) as b;
```

5. Oracle使用start with ……connect by递归，而postgresql使用CTE递归

#### 函数差异

1. Oracle使用sysdate、current_date获取当前日期，current_timestamp获取时间戳；postgresql兼容current_date和current_timestamp，同时now()函数可以取当前时间戳

```
# oracle
select sysdate,current_date from dual;

# postgresql
# select current_date,current_timestamp,now();
 current_date |       current_timestamp       |              now              
--------------+-------------------------------+-------------------------------
 2020-06-03   | 2020-06-03 16:32:02.016873+08 | 2020-06-03 16:32:02.016873+08
(1 row)
```

2. Oracle使用INSTR函数查找字符串在另一字符串的位置；postgresql使用position函数

## 数据迁移测试

数据迁移三种方式

1. Oracle数据按照格式转为文件，然后导入到postgresql
2. postgresql中安装oracle_fdw扩展，然后直接访问
3. ETL数据抽取工具

## 功能测试和数据测试

## 生产交割

