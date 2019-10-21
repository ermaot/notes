## ls
#### 单独ls

```
#ls
1  2  3  test  test2
```
#### ls -l

```
#ls -l
total 8
-rw-r--r-- 1 root root    0 Aug 26 16:48 1
-rw-r--r-- 1 root root    0 Aug 26 16:48 2
-rw-r--r-- 1 root root    0 Aug 26 16:48 3
drwxr-xr-x 2 root root 4096 Aug 26 16:48 test
drwxr-xr-x 2 root root 4096 Aug 26 16:48 test2
```
第一段：第一个字符：
符号|文件类型
---|---
- | 普通文件
d|目录
s|套接字
l|链接文件

第二段：9个字符（rw-），表示权限（属主权限，同组权限，其他组权限）<p>
第三段：数字表示文件链接数，有多少个文件链接到这个文件上<p>
第四段和第五段：所有者和用户组<p>
第六段：文件大小<p>
第七段：文件最近一次被修改时间<p>
第八段：文件名<p>

#### ls -F

```
#ls -F
1  12@  13  2  3  test/  test2/
```
/ 表示目录；无字符表示普通文件；@表示链接文件；*表示可执行文件

####其他选项
-d 表示列出文件夹具体信息；R参数表示递归显示文件夹内容；r表示逆序；t表示按修改时间排序；-s表示按文件大小排序；-a表示列出所有文件，包括隐藏文件和文件夹；-A表示只列出隐藏

```
#ls -ld test
drwxr-xr-x 2 root root 4096 Aug 26 16:48 test


#ls -R
.:
1  12  13  2  3  test  test2

./test:
1

./test2:
2

#ls  -t
test2  test  1  13  12  3  2
#ls -rt
2  3  12  13  1  test  test2


#ls -s
total 8
0 1  0 12  0 13  0 2  0 3  4 test  4 test2
#ls -ls
total 8
0 -rwxrwxrwx 2 root root    0 Aug 26 17:02 1
0 lrwxrwxrwx 1 root root    1 Aug 26 16:54 12 -> 1
0 -rwxrwxrwx 2 root root    0 Aug 26 17:02 13
0 -rw-r--r-- 1 root root    0 Aug 26 16:48 2
0 -rw-r--r-- 1 root root    0 Aug 26 16:48 3
4 drwxr-xr-x 2 root root 4096 Aug 26 17:03 test
4 drwxr-xr-x 2 root root 4096 Aug 26 17:03 test2
#ls -sr
total 8
4 test2  4 test  0 3  0 2  0 13  0 12  0 1

#ls -a
.  ..  1  12  13  2  3  test  test2

#ls -A
1  12  13  2  3  test  test2
```

## cat
cat命令可以查看文件内容或者连接文件、创建一个或者多个文件和重定向输出到终端或者文件

- 查看文件;-n显示行号；-b显示非空白行的行号；-e末尾显示结束符

```
#cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

#cat -n /etc/passwd 
     1	root:x:0:0:root:/root:/bin/bash
     2	bin:x:1:1:bin:/bin:/sbin/nologin
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin
     4	adm:x:3:4:adm:/var/adm:/sbin/nologin
     
#cat -b test.txt 
     1	test1
     2	test2

     3	test4
     
#cat -e test.txt 
test1$
test2$
$
test4$

//合并文件到新文件中
#cat test.txt test2.txt > test3.txt
```

## less 、more
- more命令用于一页一页翻阅文件

```
#more -5 test3.txt 
test1
test2

test4
test1
--More--(65%)
```

- less向前和向后翻都支持，且不需要一次性加载全部文件，速度快<p>

使用方法 | 说明
---|---
/ |向前搜索关键字，n 下一个匹配，N上一个匹配
? |向后搜索关键字，n 下一个匹配，N上一个匹配
ctrl+F|向前一个窗口
ctrl+B|向后一个窗口
ctrl+D|向前半个窗口
ctrl+U|向后半个窗口
G|跳转到文件开头或者末尾
q或者ZZ|退出


## head 、tail  显示头部和尾部
head
```
//查看开头的5行
head -n 5

//查看除最后5行的所有行
head -n -5

//查看前N个字节的内容
head -c 10
```

tail

```
//显示最后n行
tail -n 5

//即时打印末尾
tail -f -n 5

//--pid 与-f合用，进程退出时tail退出
tail -f  filename --pid=5678
```

## file：查看文件类型

```
#file text3.txt
text3.txt: ASCII text

//显示MIME型信息
#file -i text3.txt
text3.txt: text/plain; charset=us-ascii

//无空白填充
#file -N *
1: empty
12: symbolic link to `1'
13: empty
2: empty
3: empty
test: directory
test2: directory
test2.txt: ASCII text
test3.txt: ASCII text
test.txt: ASCII text
text3.txt: ASCII text

//有空白填充
# file *
1:         empty
12:        symbolic link to `1'
13:        empty
2:         empty
3:         empty
test:      directory
test2:     directory
test2.txt: ASCII text
test3.txt: ASCII text
test.txt:  ASCII text
text3.txt: ASCII text
```

## wc：统计文件信息


```
#wc test2.txt 
 4  3 19 test2.txt 
 行数 单词数 字符数  文件名
```
-l 统计行数
```
#wc -l text3.txt 
64 text3.txt
```

-w 统计单词数
```
#wc -w text3.txt 
48 text3.txt
```

-c 统计文件字节数
```
#wc -c text3.txt 
304 text3.txt
```

-L 统计最大行长度
```
#wc -L text3.txt 
5 text3.txt
```

## find

```
//在某路径下搜索文件名
# find . -name "test*"      //# find /etc -name "test*"
./test3.txt
./test.txt
./test
./test2
./test2.txt

//按照类型搜索
# find . -name "test*"  -type d
./test
./test2

//按照权限搜索
# find . -name "test*"  -type f -perm 644
./test3.txt
./test.txt
./test2.txt

//查找权限为a+w的文件
# find . -name "test*"  -type f -perm /a+w
./test3.txt
./test.txt
./test2.txt

//查找空文件
# find .   -type f -empty
./3
./13
./1
./test/1
./test2/2
./2

// -user 

// -mtime 3    3天前修改过
// -mtime +30   30天前修改过
//-mtime -3   3天内修改过
//-atime 60  60分钟内访问过
// -size   50M   大小50M
// -size +50M  -size 100M   大于50M小于100M
```
## touch

```
// -a 只改变访问时间
// -c 不创建新文件；如果文件存在，修改时间戳；如果文件不存在则不创建
// -m 修改文件的修改时间，访问时间不变
```

## mkdir

```
// 可以使用相对路径也可以使用绝对路径
//-p可以递归创建，即中间文件夹不存在的时候一同创建
// -m 可以设置创建目录的权限
```

## cp

```
//创建一个文件的副本
cp file.txt newfile.txt

//复制到一个新的文件夹下
cp file.txt /tmp

//使用*
cp * /tmp

// -p 选项保留源文件的额所有者、用户组、权限、修改和访问时间等一些扩展属性
cp -P filename /tmp

//-R 或者 -r 递归复制
cp -R * /tmp

//归档模式复制
cp -a * /tmp
-a ，归档模式，相当于-dpR，-d保留软链接，-p  -R同上
```

## ln

```
ln -s               //创建软链接
ln                  //硬链接

ln  --backup        //如果文件已经存在，则备份
ln -f               //强制覆盖
```

## mv
## rm
## ls -l 查看权限
## chmod修改权限
## chown、chgrp
## setuid 和 setgid
## sort
## uniq
## tr：替换或者删除字符
## grep：查找字符串
## diff比较两个文件
## hostname
## w、who：列出系统登录的用户
## uptime
## date
## id