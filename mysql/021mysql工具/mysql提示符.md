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
```
\c  A counter that increments for each statement you issue
\D  The full current date
\d The default database
\h The server host
\l The current delimiter (new in 5.1.12)
\m  Minutes of the current time
\n  A newline character
\O  The current month in three-letter format (Jan, Feb, …)
\o  The current month in numeric format
\P  am/pm
\p The current TCP/IP port or socket file
\R  The current time, in 24-hour military time (0–23)
\r  The current time, standard 12-hour time (1–12)
\S  Semicolon
\s  Seconds of the current time
\t  A tab character
\U   

\u Your user name
\v  The server version
\w  The current day of the week in three-letter format (Mon, Tue, …)
\Y  The current year, four digits
\y  The current year, two digits
\_  A space
\   A space (a space follows the backslash)
\'  Single quote
\"  Double quote
\\  A literal “\” backslash character
\x 
x, for any “x” not listed above
```