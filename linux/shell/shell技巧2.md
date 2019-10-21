常用命令
## cat
```
# cat newline.txt   newline2.txt 
test1
test2
test3
test1
test2
test3

//- 符号视作来自标准输入
# echo test | cat - newline.txt 
test
test1
test2
test3

//压缩连续的空白行
# cat  blank.txt 
test1

test2



test3



# cat -s blank.txt 
test1

test2

test3

//也可以用tr完全删除空行
# cat  blank.txt  | tr -s "\n"
test1
test2
test3

# cat  -n blank.txt 
     1	test1
     2	
     3	test2
     4	
     5	
     6	
     7	test3
     8	
     9	
# cat  -n blank.txt  | cat -T
     1^Itest1
     2^I
     3^Itest2
     4^I
     5^I
     6^I
     7^Itest3
     8^I
     9^I
```

## 录制终端会话
-t 表示将时序数据导入标准错误 2>则用于将stderr重定向到timeing.log
```
# script -t 2>timeing.log -a output.session
Script started, file is output.session

#exit
# scriptreplay timeing.log output.session
```
#### terminal广播

```
//terminal 1
# mkfifo scriptfifo
//terminal2
# cat scriptfifo
//terminal1
# script -f scriptfifo

结束需要在terminal1 中exit
```
## find
```
//or 和and
# find . \( -name "*.txt" -o -name "*fifo" \)  -print
# find . \( -name "*.txt" -a -name "*new*" \)  -print

//指定路径，相当于把整个路径名一起匹配
# find /home/ -path "*root" -print
/home/www/.cache/yarn/v1/npm-node-sass-4.9.0-d1b8aa855d98ed684d6848db929a20771cc2ae52/test/fixtures/cwd-include-path/root

//正则。似乎有点不是太好用
find -regex

//否定
# find . ! -name "*.txt"

//目录深度
# find . -maxdepth 2 -mindepth 1 -type f 

//比某时间新
# find . -type f -newer 2.txt

//删除匹配的文件
# find . -type f -newer 2.txt -delete

//权限位
# find . -type f -newer 2.txt -perm 644

//直接操作
# find . -type f -newer 2.txt -exec chown test:test {} \;
```

## xargs
xargs将接收到的数据重新格式化，再将其作为参数提供给其他命令
```
// 多行转单行
# cat newline.txt | xargs 
test1 test2 test3

//单行转多行
# cat num.txt | xargs  -n 4
1 2 3 4
5 6 7 8
9 10

//分隔符分隔
# echo "test1test2test2" | xargs -d "t"
 es 1 es 2 es 2
 
//find 和xargs混用的注意事项
find的输出结果的分隔符可能是'\n' 或者' '，所以find末尾最好加-print0
```

## tr
```
# echo "Test 123 test" | tr "a-z" "A-Z"
TEST 123 TEST

# echo "Test 123 test" | tr "a-z" "A-Z" | tr '0-9' "e-m"
TEST fgh TEST

//删除某个集合
# echo "Test 123 test" | tr -d "a-z"
T 123 

//删除非某个集合
# echo "Test 123 test" | tr -d -c "a-z"
esttest
# echo "Test 123 test" | tr -d -c [abc1]
1

//算加法
# echo "1 2 3 4 5 6" | echo $[$(tr " " "+")]
21
```
## md5sum
```
# md5sum 2.txt  2 newline.txt 
d41d8cd98f00b204e9800998ecf8427e  2.txt
d41d8cd98f00b204e9800998ecf8427e  2
c19ac63da7949b15179f42093cbf95b8  newline.txt
```
## 排序、去重

```
//按数字排序
# sort -n newline.txt 
test1
test1
test2
test3

//逆序
# sort -r newline.txt 
test3
test2
test1
test1

//按月份排序
# sort -M mon.txt | paste  -d "\t"  mon.txt -
January	January
December	July
July	December

//对两个文件排序并合并，合并的结果不需要排序
sort -m 

//指定按照哪一个关键字排序
# sort -k 2 col.txt 
test4	test1
test3	test2
test2	test3
test1	test4
```
## 分隔文件和数据

```
# dd if=/dev/zero bs=10k count=1 of=data.file

//按dao分割
# split -b 10k data.file
或者
# split -b 10k data.file -d -a 4
# split -b 10k data.file -d -a 4

//按行分割
# split -l 1 newline2.txt -d -a 4 split_file
```
csplit可以根据指定条件和字符串分割
```
# csplit server.log /SERVER/ -n 2 -s {*} -f server -b "%02d.log"
```
1. /SERVER/用来匹配某一行，支持正则/[REGEX]/
2. {*}表示匹配后重复分割，直到文件末尾。{整数}表示执行次数
3. -s静默模式
4. -f分割后文件名前缀
5. -b指定后缀格式


## 提取字符串
```
//非贪婪模式
# test="shell.shell.linux.linux"
# echo ${test%.*}
shell.shell.linux
# echo ${test#*.}
shell.linux.linux

//贪婪模式
# echo ${test%%.*}
shell
# echo ${test##*.}
linux
```
## 批量重命名和移动
rename版本如果是perl版本的，那rename命令是支持正则的；因此在perl版本的rename下，执行上述命令就可以成功，如果rename版本是c版本的，那么上述命令是不成功的

```
rename *txt *TXT
```

## 交互式脚本
使用read或者expect
