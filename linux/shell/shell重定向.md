linux中，一切皆文件，硬件也是如此
- 0——标准输入——键盘：从文件（默认是键盘）读取输入
- 1——标准输出——屏幕：发送数据到文件（默认是屏幕）
- 2——标准错误——屏幕：发送所有错误信息到一个文件（默认是屏幕）
上述3个数字是标准的POSIX字符，也称为==文件描述符==

![标准输入输出流](https://github.com/ermaot/notes/blob/master/linux/shell/pic/shell%E9%87%8D%E5%AE%9A%E5%90%91.png)

## 标准输入
在Shel运行任何命令之前,它先尝试打开文件进行读取.如果打开文件失败,Shell将以一个错误退出并不运行命令.如果打开文件成功,Shell使用打开的文件的文件描述符作为命令的标准输入文件描述符.标准输入具有如下特点:
1. 它是默认的输入方法,它被所有命令使用来读取输入.
2. 它用数字0表示
3. 它也被称作 stdin.
4. 默认的标准输入设备是键盘.

操作符"<"是输入重定向操作符,其语法如下所示:
```
command  < input_file

eg:
#cat < test2.txt 
#sort < text3.txt
```
## 标准输出
标准输出具有如下特点:
1. 它被命令用来写入或显示命令自身的输出
2. 它用数字1表示.
3. 它也被称作 stdout
4. 默认的标准输出设备是屏幕.

操作符">"是输出重定向操作符,它的语法如下所示:
```
command  > input_file
```
## 标准错误
标准错误具有如下特点
1. 它是默认的错误输出方法,它被用于写入所有系统错误信息
2. 它用数字2表示.
3. 它也被称为 stderr
4. 默认的标准输出设备是屏幕或显示器.

操作符"2>"是标准错误重定向操作符,其语法如下所示:
```
command  2> input_file
```

## 重定向
#### 文件重定向
- 文件重定向是更改一个文件描述符以指向一个文件
- 操作符 ">"开始一个输出重定向，告诉bash标准输出（stout）应当指向一个文件
- 操作符 >> 表示追加
- 操作符“<” 表示输入
```
#tr a-z A-Z < text3.txt 
```

```
#read line1 < text3.txt 
#echo $line1
test1


#if true ;then read line1;read line2;fi < text3.txt
#echo $line1
test1
#echo $line2
test2
```

- bash有一种重定向类型是here-documents

<<
```
#tr a-z A-Z << END
> one two three
> four five six
> END
ONE TWO THREE
FOUR FIVE SIX

#tr a-z A-E << END
> ab cd ef gh
> END
AB CD EE EE
```

<<<

```
#tr a-z A-Z <<< end
END
[root@izm5edbv563hlvcbf71ophz dir1]#
```
#### 创建空文件

```
# > file
```
#### 丢弃输出

```
command  > /dev/null

//丢弃标准错误输出
command 2> /dev/null
//丢弃全部输出
command &> /dev/null
```

## 标准错误和标准输出重定向

```
command &> filename
command >& filename
command > filename 2>&1
command 2>&1 >filename
```

追加重定向

```
command >> filename 2>&1
command 2>&1 >>filename
```

## 标准输入与输出重定向

```
command < input-file > outputfile

<imput-file command > outputfile
```
样例
```
#tr a-z A-Z < text3.txt  > text32.txt
#< text3.txt tr a-z A-Z   > text32.txt
```
## 文件描述符
- exec命令允许我们操作文件描述符
- exec 2 > filename 可以将我们后面所有的命令的错误输出都到filename中

```
#exec 3<text3.txt 
//只能使用一次
#grep test <& 3
//关闭文件描述符
exec 3<&-
```

## 指定用于输出的文件描述符

```
exec [4] >file
```

```
# exec 4>logout.txt
# date >&4
# cat logout.txt 
Thu Aug 29 11:15:09 CST 2019
```
## 关闭文件描述符

```
//关闭输入文件描述符
exec 3<&-

//关闭输出文件描述符
exec 3>&-
```

## 注意

```
command 2>&-  //丢弃错误输出，但并未所有情况都能正常运行，所以不建议，而应该用 2>/dev/null
```


## 打开可读写的文件描述符

```
exec [n] <> filename
```
