本文来自[Postgresql - 将表/数据缓存到内存中（预热） - pg_prewarm](https://blog.csdn.net/chuckchen1222/article/details/81064596)

- 预热功能，使用pg_prewarm函数，方便的将数据缓存至内存中。
- 这个功能不是自带的，是存在在扩展包中，所以要使用前需要先添加扩展。

## 函数简介

#### 函数体

pg_prewarm(regclass, mode text default 'buffer', fork text default 'main', first_block int8 default null, last_block int8 default null) RETURNS int8

- 第一个参数是预热的relation。
- 第二个参数是要使用预热的方法
- 第三个参数是relation fork被预热
- 第四个参数是预热的第一个块号
- 第五个参数是预热的最后一个块号
- 返回值是prewarm块的数量。

#### 与人方法

预热方法有三种：

1. 对操作系统发出异步prefetch请求
2. 读取块的请求范围，但可能会较慢
3. 缓冲区将请求的块范围（执行的查询）读入数据库缓冲区缓存中。

当使用预取或读取时，或使用PostgreSQL在使用缓冲器时可能会导致较低编号的块被释放，因为较高编号的块被读入。预热数据也没有对缓存驱逐的特殊保护，因此其他系统活动可能会在读取后不久将新的预热块驱逐出去；反之，预热也可能从高速缓存中驱逐其他数据。由于这些原因，**预热通常在启动时最有用，当缓存大部分为空时**。


## 操作过程
1. 创建extension
```
mytest=# create extension pg_prewarm ;
CREATE EXTENSION

```
2. 我们需要借助pg_buffercache 来查看内存中的变化
```
mytest=# create extension pg_buffercache ;
CREATE EXTENSION
```
3. 重启一下pg
```
service postgresql-10 restart
```
4. 查看内存信息
```
mytest=# select count(*) from pg_buffercache where relfilenode = (select relfilenode from pg_class where relname = 'test01');
count
-------
0
(1 row)

```
5. pg_prewarm
```
mytest=# select pg_prewarm('test01','buffer','main') ;
pg_prewarm
------------
2041
(1 row)

mytest=# select count(*) from pg_buffercache where relfilenode = (select relfilenode from pg_class where relname = 'test01');
count
-------
2041
(1 row)

说明表已经被缓存到内存中。

```

