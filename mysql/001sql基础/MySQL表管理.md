## 克隆表
```
create table clone like sample ;				//复制/克隆表，但不复制外键定义，data directory 和index directory；仅仅是参照表定义而创建一个空表
insert into new_table select * from sample ;	//插入表的数据
insert into new_table( a ,b ) select c ,d from sample ;
```
## 保存查询结果到表
```
create table new_table select * from sample ;	//创建新表，包含了之前所有的数据
insert into new_table( a ,b ) select c ,d from sample ;
create table new_table( a int ,b int ,c int) select * from sample;		//new_table 表的列数和 sample 可以不一致
```

## 创建临时表

```
create temporary table tmp select * from sample ;
```

## 查看表的存储引擎

```
select engine from information_schema.tables where table_schema = 'temp' and table_name = 'sample' ;
// 此处有一个有意思的现象
select engine, * from information_schema.tables limit 1;        //非法
select *,engine from information_schema.tables limit 1;         //合法
show create table new_table ;
show table status like '%url%';
```
## 修改存储引擎

```
alter table new_table engine = innodb ;
```



## 本文来自《mysql cookbook》