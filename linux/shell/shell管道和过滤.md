## 为何使用管道
1. 命令语法紧凑且使用简单
2. 通过管道将命令串联完成复杂任务
3. 管道输出的标准错误会聚集到一起

## 管道中的重定向

```
command < output.txt | command2|…… | commandN > output.txt

eg:
# mount | head -5 | column -t
sysfs       on  /sys                  type  sysfs       (rw,nosuid,nodev,noexec,relatime)
proc        on  /proc                 type  proc        (rw,nosuid,nodev,noexec,relatime)
devtmpfs    on  /dev                  type  devtmpfs    (rw,nosuid,size=3994256k,nr_inodes=998564,mode=755)
securityfs  on  /sys/kernel/security  type  securityfs  (rw,nosuid,nodev,noexec,relatime)
tmpfs       on  /dev/shm              type  tmpfs       (rw,nosuid,nodev)
```

## 过滤器
如果一个linux命令从标准输入接收它的输入数据，并在标准输出上产生它的输出数据，则可以成为过滤器

常用过滤器命令
命令|说明
---|---
awk|用于文本处理的解释性程序设计语言,通常被作为数据提取和报告的工具.
cut|用于将每个输入文件(如果没有指定文件则为标准输入)的每行的指定部分输出到标准输出.
grep|用于搜索一个或多个文件中匹配指定模式的行.
tar|用于归档文件的应用程序.
head|用于读取文件的开头部分(默认是10行).如果没有指定文件,则从标准输入读取.
paste|用于合并文件的行
sed|用于过滤和转换文本的流编辑器
sort|用于对文本文件的行进行排序.
split|用于将文件分割成块
strings|用于打印文件中可打印的字符串
tac|与cat命令的功能相反,用于倒序地显示文件或连接文件
tail|用于显示文件的结尾部分.
tee|用于从标准输入读取内容并写入到标准输出和文件.
tr|用于转换或删除字符
uniq|用于报告或忽略重复的行.
wc|用于打印文件中的总行数、单词数或字节数.

样例

```
# awk -F: '{print $1}' /etc/passwd | sort | head -5
adm
bin
chrony
daemon
dbus


# history | awk '{print $1,",",$2}' | sort -rn | uniq | head -5
1024 , history
1023 , history
1022 , history
1021 , history
1020 , awk


# grep "bin/bash" /etc/passwd | cut -d: -f1,6
root:/root
git:/home/git
java:/home/java
www:/home/www
test:/home/test

//tee命令从输入流输入，并同时可以输出到屏幕（标准输出），也同时重定向到一个文件中
# ls | tee list.txt
```
