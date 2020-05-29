## 查看procedure  和function 

1. 全部查看

```
select * from mysql.proc ;
```

2. 查看function 

```
show procedure status;
```

或者

```
select `name` from mysql.proc where db = 'your_db_name' and `type` = 'FUNCTION'
```

3. 查看procedure  

```
 show procedure status; 
```

或者

```
select `name` from mysql.proc where db = 'your_db_name' and `type` = 'PROCEDURE'
```

## 查看视图和表

```
> show table status where comment ='view';
```

或者

```
SELECT * from information_schema.VIEWS   
SELECT * from information_schema.TABLES 
```

## 查看触发器

```
> SHOW TRIGGERS from test like "%test%";
```

或者

```
select * from information_schema.triggers;
```

