本文参考https://www.cnblogs.com/sunny18/p/9007593.html
和
https://www.cnblogs.com/bibiafa/p/9230093.html
## 修改mysql提示符
- 在操作系统命令行中输入
```
mysql -u root --prompt 提示符 -p 回车
```
- 在mysql命令行中输入
```
mysql> prompt \u@mysql \r:\m:\s->
PROMPT set to '\u@mysql \r:\m:\s->'
root@mysql 04:27:21->
```
- 在my.cnf中配置
```
[mysql]
prompt=\\u@username \\r:\\m:\\s
```
## 提示符详解
参数|说明
---|---
\c | 执行的sql的序号
\D|当前日期的详细值
\d|当前所处数据库
\h|服务器主机
\l|当前分隔符
\m |当前时间的分钟
\n|  换行符
\O | 当前月份的三字母表示法 (Jan, Feb, …)
\o  |当前月份的数字表示
\P | am/pm
\p |当前TCP/IP端口或者socket文件
\R | 当前小时，24小时制
\r  |当前小时，12小时制
\S  |分号
\s  |当前时间的秒数
\t  |制表符
\U  | 包含了主机的用户名
\u |用户名
\v  |服务器版本
\w  |当前星期，三字符表示
\Y  |当前年，4位数
\y  |当前年，2位数
\_  |空格
\   |空格（斜线后有一个空格）
\'  |单引号
\"  |双引号
\\  |转义后的右斜线