## binlog结构

```
### UPDATE `test`.`test`
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='test1' /* VARSTRING(48) meta=48 nullable=1 is_null=0 */
### SET
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='test1' /* VARSTRING(48) meta=48 nullable=1 is_null=0 */
```

#### 坑1

可以发现，binlog中并没有记录列名，只记录了顺序。

单机的时候问题不大，但是在双Master的结构下，这问题可就大了，假设Master A <--> Master B正在相互复制，在Master B做一个DDL改变了字段顺序，那么从B DDL完成开始，到A接受执行完这个DDL为止，之间A产生的这张表的数据复制到B都是错误的！因为MySQL只按字段顺序应用binlog。

如何避免这个问题呢？有两个办法，最保险的只加字段不删字段，加字段总是在表末尾，一句话：不改变字段的顺序。另一个办法就是改变字段顺序的DDL只在提供服务的主机上执行，如果双Master都提供服务，这就不行了，只能在末尾添加字段。

#### 坑2

mysqlbinlog这个工具的作者真的比较懒，-d参数过滤数据库的时候，只有Statement方式记录的SQL能被过滤，所有按Row方式记录的SQL都没有被过滤！也就是说，假设你的数据库是基于Row方式记录binlog的，你想通过mysqlbinlog -d db1来过滤出db1的SQL，这是不靠谱的，所有Row方式记录的SQL全部被解出来了，你要是到数据库去应用这些解出来的SQL，你就准备悲剧了，各种Duplicate Key



#### 坑3

Row方式记录的SQL都是Base64格式，解出来还是一样，传到数据库去执行的话，MySQL还会把这些BASE64的字串解成SQL，再去应用，所以数据库Load可能会增长的很高