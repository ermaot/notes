#### 阻止插入

使用可更新视图

```
create table  test(c1 VARCHAR2(10) DEFAULT '默认1',c2 VARCHAR2(10) DEFAULT '默认2',c3 VARCHAR2(10) DEFAULT '默认1',c4 date DEFAULT SYSDATE);

create view test_view as select c1,c2,c3 from test;
insert into test_view values('默认1','默认2','默认3');

select * from test_view;
C1	C2	C3
默认1	默认2	默认3
```

#### 复制表定义和数据

1. 复制表定义
```
create table test_dup as select * from test where 1=2;
```

2. 复制表数据

```
create table test_dup as select * from test;
```

mysql使用的方法比Oracle多了一个like，可以直接复制表结构.postgresql也有like，甚至还能继承表

```
create table test_dup like a;
create table test_dup as select * from test;
create table test_dup as select * from test;
```

#### insert all和insert first

insert all是Oracle特有的，可以支持条件插入、多表同时插入；MySQL和postgresql都没有。

1. 无条件insert

   ```
   insert all 
   into test1(a,b,c) values(1,2,3)
   into test2(a,b,c) values(1,3,4);
   ```

   

2. 有条件insert

```
insert all 
when id in (1,2,3) then into test(a,b,c) values(1,2,3)
when name in ('a','b','c') then into test2(a,b,c) values(1,3,4);
```

insert first当第一个表符合条件后，第二个就不再插入对应的行。