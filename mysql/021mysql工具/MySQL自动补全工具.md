## MySQL自带工具

注意：在命令行下只有切换到数据库之后，才能补全表名，对于命令是不能补全的。

1、my.conf增加如下配置：

[mysql]
#no-auto-rehash
auto-rehash         #添加auto-rehash
重启mysql服务，然后用客户端连接即可。

2、命令行增加参数

mysql -u root -p --auto-rehash

## MYCLI
centos/RH系安装：

```
sudo yum install pip  
sudo pip install mycli
```
Debian/Ubuntu 安装 MyCLI ：


```
sudo apt-get update
sudo apt-get install mycli
```

- MyCLI 是一个易于使用的命令行客户端，可用于受欢迎的数据库管理系统 MySQL、MariaDB 和 Percona，支持自动补全和语法高亮
- 它是使用 prompt_toolkit 库写的，需要 Python 2.7、3.3、3.4、3.5 和 3.6 的支持
- MyCLI 还支持通过 SSL 安全连接到 MySQL 服务器。

#### MyCLI 的特性
- 当你第一次使用它的时候，将会自动创建一个文件 ~/.myclirc。
- 当输入 SQL 的关键词和数据库中的表、视图和列时，支持自动补全。
- 默认情况下也支持智能补全，能根据上下文的相关性提供补全建议。
比如：
SELECT * FROM <Tab> - 这将显示出数据库中的表名。
SELECT * FROM users WHERE <Tab> - 这将简单的显示出列名称。

- 通过使用 Pygents 支持语法高亮
- 支持 SSL 连接
- 提供多行查询支持
- 它可以将每一个查询和输出记录到一个文件中（默认情况下禁用）。
- 允许保存收藏一个查询（使用 \fs 别名 保存一个查询，并可使用 \f 别名 运行它）。
- 支持 SQL 语句执行和表查询计时
- 以更吸引人的方式打印表格数据

![mycli自动补全](https://github.com/ermaot/notes/blob/master/mysql/021mysql%E5%B7%A5%E5%85%B7/pic/MySQL%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8%E5%B7%A5%E5%85%B7.png)