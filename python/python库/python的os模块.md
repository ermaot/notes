这个模块中的大部分函数通过对应平台相关模块实现, 比如 posix 和 nt.os 模块会在第一次导入的时候自动加载合的执行模块

## 处理文件

序号|函数名|说明
---|---|---
1|os.rename(oldname,newname)|重新命名文件
2|os.remove(filename)|删除文件
3|os.path.isfile()|检验给出的路径是否是一个文件
4|os.stat("1.csv")|获取文件属性

```
In [1]: import os

In [2]: os.stat("1.csv")
Out[2]: os.stat_result(st_mode=33206, st_ino=6473924464703115, st_dev=240536266, st_nlink=1, st_uid=0, st_gid=0, st_size=12, st_atime=1565342546, st_mtime=1565342546, st_ctime=1565342545)

In [3]: type(os.stat("1.csv"))
Out[3]: os.stat_result

In [4]: mode, ino, dev, nlink, uid, gid, size, atime, mtime, ctime = os.stat("1.csv")

In [5]: print(mode, ino, dev, nlink, uid, gid, size, atime, mtime, ctime)
33206 6473924464703115 240536266 1 0 0 12 1565342546 1565342546 1565342545

In [6]: print(os.stat("1.txt").st_mode,os.stat("1.txt").st_ino)
33206 9851624184877293

In [7]: print(os.stat("1.txt")[stat.ST_CTIME])
1602390669
```
有一个库，叫stat，可以借助它操作各种文件属性
```
In [1]: import time,stat,os

In [2]: st = os.stat("1.txt")

In [3]: st.st_mode
Out[3]: 33206

In [4]: stat.S_ISREG(st.st_mode)
Out[4]: True

In [5]: stat.S_ISLNK(st.st_mode)
Out[5]: False
```
## 处理目录
序号|函数名|说明
---|---|---
1|os.listdir(foldername)|列出目录下的所有文件，包括隐藏文件
2|os.chdir(foldername)|改变工作目录
3|os.pardir|父目录，一般是".."
4|os.curdir|当前目录，一般是"."
5|os.path.isdir()|检验给出的路径是否是一个目录
6|os.removedirs()|删除多个目录
7|os.getcwd()|得到当前工作目录
8|os.path.isabs()|判断是否是绝对路径
9|os.path.exists()|检验给出的路径是否真地存在
10|os.path.split()|返回一个路径的目录名和文件名
11|os.path.splitext()|分离扩展名
12|os.path.dirname()|获取路径名
13|os.path.basename()|获取文件名
14|os.makedirs（r“c：\python\test”）|创建多级目录
15|os.mkdir（“test”）|创建单个目录

## 处理进程

…………