## 让MySQL搜索、排序时区分大小写
#### 在SQL中强制
```
SELECT `field` FROM `table` WHERE BINARY `username` = ‘xxxxxx’
```
#### 建表时强制
在建表时，添加BINARY属性即可。如果使用PHPMyAdmin建表，直接在属性中选择‘binary’ 即可。

