## 交集和差集
```
# comm newline.txt  newlineblank.txt 
		test1
		test2
		test3
test1
comm: file 1 is not in sorted order


// -1 -2 -3分别表示不显示1，2，3列
# comm newline.txt  newlineblank.txt  -1
	test1
	test2
	test3
comm: file 1 is not in sorted order

# comm newline.txt  newlineblank.txt  -1 -2
test1
test2
test3
comm: file 1 is not in sorted order

# comm newline.txt  newlineblank.txt  -1 -2 -3
comm: file 1 is not in sorted order

# comm newline.txt  newlineblank.txt   -3
test1
comm: file 1 is not in sorted order
```
## 文件权限和粘滞位
- setuid(S)出现在执行权限x的位置，是指允许用户以其拥有者的权限执行可执行文件
- 目录的rwx权限
1. 目录r权限允许读取目录中文件和子目录列表
2. 目录w允许目录中创建或者删除文件或目录
3. 目录x指明是否可以访问目录中的文件和子目录
- 权限位最后3个字符没有S权限（setuid和setgid）
- 如果目录设置了粘滞位，只有创建该目录的用户才能删除目录中的文件
- 如果没有执行权限只有粘滞位，用t表示；如果同时设置了，就用T


## 创建不可修改文件

```
chattr +i file
```
## 批量生成文件/文件夹

```
for i in {1..10}.txt
do touch $i
done
```
或者
touch -a 只更改文件访问时间；touch -m只更改内容修改时间；touch -d "Fri Jun 25 20:50:14 IST 1999" filename
```
# touch {1..10}.txt
# touch {10..20..2}.txt
# mkdir {10..20..2}.txt
```
## 挂载
==（这部分要重新看看）==
对挂载文件做了修改，可以用sync写入物理设备
```
sync
```

## 查找文件不同
diff
```
# diff newline.txt  newlineblank.txt 
4d3
< test1

# diff -u newline.txt  newlineblank.txt 
--- newline.txt	2019-08-30 10:55:23.999705033 +0800
+++ newlineblank.txt	2019-08-30 09:21:00.647550272 +0800
@@ -1,4 +1,3 @@
 test1
 test2
 test3
-test1

//可以对目录递归diff
# diff -Naur dir1 dir2
-N将却是文件视为空文件
-a将所有文件视为文本文件
-u生成一体化输出
-r递归遍历
```
patch
```
# diff -u newline.txt  newlineblank.txt  > version.patch
# patch -p1 newline.txt  < version.patch 
patching file newline.txt
```

## 列出目录
```
//方法1
# ls -d /*

//方法2
# ls -F | grep "/$"

//方法3
# ls | grep "^d"

//方法4
# find . -type d 
```
## pushd 和 popd
记录路径并便于切换

## wc统计

```
wc -l   //行数
wc -c   //字符数
wc -L   //最大长度的行的字符数
wc -w   //单词数
```
## 目录树

```
//按照模式tree
# tree -P "*.txt"

//排除某种模式
# tree -I "*.txt"

//打印大小
# tree -h -P "*.txt"

//特定深度
# tree -L 2
```
## 正则
符号|说明
---|---
星号*|匹配前面字符串或者正则表达式任意次（0次或者以上）
句点.|匹配除换行符外的任意一个字符
插入符号^|匹配一行的开始；或者否定正则表达式
美元符$|匹配一行的末尾
方括号[]|匹配方括号指定的字符集的一个字符<p>[abc]匹配a,b,c任意一个<p>[a-h]a~h中任意一个<p>[A-Z][a-z]任意一个大写或者小写字母<p>[^a-d]除a~d之外的所有字符
反斜线\|转义一个特殊字符
转义尖括号\<\>|标记单词边界。\<the\>匹配单词the，而不匹配them,there,other等

- 扩展正则
除以上外，还支持

符号|说明
---|---
问号?|匹配0个或者1个前面的字符
加号+|匹配1个或者多个
转义大括号\{\}|指示前面正则表达式的次数，如[0-9]\{5\}将匹配5位数字
圆括号()|包含正则表达式，或者用于提取字符串
竖线\||表示或，比如a(b|c)d表示"abd"或"acd"

- posix字符

posix字符|含义
---|---
[:alnum:]|匹配字母和数字等同于A-z,a-z,0-9
[:alpha:]|匹配字母字符，等同于于A-z,a-z
[:blank:]|匹配空格或者制表符
[:cntrl:]|匹配控制字符
[:digit:]|匹配十进制数字
[:graph:]|匹配ASCII码值范围33~126字符，类似[:print:]，但不包括空格
[:lower:]|匹配小写字母
[:upper:]|匹配大写字母
[:print:]|匹配ASCII码值范围32~126字符
[:space:]|匹配空白字符）空格和水平制表符
[:xdigit:]|匹配十六进制数字，等同于0~9,A~F,a~f


- gnu扩展

字符|含义
---|---
\w|任意单词组成的字符，相当[:alnum:]
\W|任意非单词组成字符，相当于^[:alnum:]
\B|匹配两个单词组成字符之间的空字符串
\b|单词边界
\s|空白
\S|非空白

## grep
- grep 搜索文件内容
- 高亮显示 --color
- 默认模式是通配符，正则表达式需要加-E 或者使用egrep
- 只显示匹配到的内容 -o
- 统计匹配到的行数-c
- 打印匹配结果所在的行-n
- 打印样式匹配所位于的字符或者字节偏移
- 递归搜索用-R（-r）
- 忽略大小写-i
- 指定多样式-e
- 搜索的时候只搜索某些文件--include
- 排除某些文件--exclude
- 静默输出-q（quiet）
- 打印匹配结果后的行-A
- 打印匹配结果前的行-B
- 打印匹配结果之前以及之后的行-C

## cut 按列切分

```
# cut -f 1,2,3,4 num2.txt 
1	2	3	4

//打印除了某些列之外的列
# cut -f 1,2,3,4 num2.txt  --complement
5

//可以看到不重复打印列
# cut -f 1,2,3,3,4 num.txt  -d " "
1 2 3 4

//打印一定范围的列。同理-b（字节），-c（字符）
# cut -f 1-4 num.txt  -d " "
1 2 3 4
# cut -f 1- num.txt  -d " "
1 2 3 4 5 6 7 8 9 10
# cut -f -4 num.txt  -d " "
1 2 3 4
# cut -c -5 num.txt  
1 2 3
# cut -b -4 num.txt  
1 2 

//定制输出的分隔符
# cut -f 1,2,3,4 num2.txt --output-delimiter ","
1,2,3,4

# cut -f 1-2,3-4 num2.txt --output-delimiter ","
1,2,3,4
```
