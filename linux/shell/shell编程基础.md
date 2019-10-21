## 一些基础知识点
- shell脚本的shebang

指定脚本的解释程序，如果没有写，则为bash，一般可以写做

```
#!/bin/bash
```
- shell的注释

使用#

- 脚本的权限

```
chmod +x    test.sh
```
## bash的参数扩展
参数是一个存储数值的实体,并由名称、数字或特定符号所引用.
1. 被名称引用的参数称作变量.
2. 被数字引用的参数称作位置参数.
3. 被特定符号引用的参数具有特殊的含义和用途,被作为Bash的特殊内部变量引用


```
#test="test"
#echo $test
#echo $testR
#echo ${test}R
#echo $test-R

位置参数，如果位置大于9，也需要大括号
echo $1
echo ${10}
```
间接扩展

```
#test1=test2
#test2="test2v"
#echo $test1
test2
#echo ${!test1}
test2v
```

大小写参数扩展//bash4.0之后的特性

```
跟着,首字符小写；跟着,,全部字符小写
跟着^首字符大写；跟着^^全部字符大写（~具有一样的效果）
#echo ${test1}
test2
#echo ${test1,}
test2
#echo ${test1^}
Test2
#echo ${test1^^}
TEST2
#echo ${test1~}
Test2
#echo ${test1~~}
TEST2
```

变量名扩展

```
#test1="test1v"
#test2="test2v"
#test3="test3v"
//以某种模式开头的变量
#echo ${!test*}
test1 test2 test3
//以某种模式开头的变量
#echo ${!test@}
test1 test2 test3
```
//#表示从开头匹配指定模式最短字符串，##表示从开头匹配指定模式最长字符串
```
#echo ${test1}
test1v
#echo ${test1#*t}
est1v
#echo ${test1#*t*}
est1v
#echo ${test1#t*}
est1v
#echo ${test1##t*}

#echo ${test1##*t}
1v
```

//%表示从末尾匹配指定模式最短字符串，%%表示从末尾匹配指定模式最长字符串
```
#echo ${test1}
test1v
#echo ${test1%*t*}
tes
#echo ${test1%%*t*}
```

//提取文件名和后缀名
```
#testfile="test.txt"
#echo $testfile
test.txt

#echo ${testfile#*.}
txt
#echo ${testfile%.*}
test
```
//提取文件名和目录名
```
#filename="/tmp/file.txt"
#echo $filename
/tmp/file.txt
#echo ${filename%/*}
/tmp
#echo ${filename##*/}
file.txt

```
字符串搜索与替换，/表示替换一次，//表示替换所有，如果没有指定匹配模式，就代表全部替换
```
#testre="ababcdef1234"
#echo ${testre/ab/abc}
abcabcdef1234
#echo ${testre//ab/abc}
abcabccdef1234
#echo ${testre//ab/}
cdef1234
```

字符串长度

```
#echo ${#testre}
12
```
截取字符串

```
#echo ${testre:4}
cdef1234
#echo ${testre:4:5}
cdef1
```
默认值与未定义默认值

```
//如果变量未定义或者为null
#echo ${testre:-test}
ababcdef1234
#unset testre
#echo ${testre:-test}
test
//如果变量未定义
#echo ${testre-test}
test
```
指定默认值

```
与上面很类似，但同时赋值给变量
#unset testre
#echo ${testre:=test}
test
#echo ${testre}
test
```
使用替代值

```
如果参数 PARAMETER是未定义的,或其值为空时,这种模式将不扩展任何内容.如果参数PARAMETER是定义的,且其值不为空,这种模式将扩展WORD,而不是扩展为参数 PARAMETER的值.
#unset testre
#echo ${testre:+test}

#echo ${testre+test}

#testre="testd"
#echo ${testre+test}
test
#echo ${testre:+test}
test
```


## bash内部变量
系统变量|含义
---|---
BASH_VERSION|保存Bash实例的版本
DISPLAY|设置 X display名字
EDITOR|设置默认的文本编辑器
HISTFILE|保存命令历史的文件名
HISTFILESIZE|命令历史文件所能包含的最大行数
HISTSIZE|记录在命令历史中的命令数
HOME|当前用户的主目录
HOSTNAME|你的计算机的主机名
IFS|定义 Shell的内部字段分隔符,一般是空格符、制表符和换行符
PATH|搜索命令的路径.它是以冒号分隔的目录列表<p>Linux下的标准命令之所以能在Shell命令行下的任何路径直接使用<p>就是因为这些标准命令所在目录的路径定义在了PATH变量中<p>Shells会在PATH环境变量指定的全部路径中搜索任何匹配的可执行文件
PSI|你的提示符设定
PWD|当前工作目录.由cd命令设置
SHELL|设置登录 Shell的路径
TERM|设置你的登录终端的类型
TMOUT|用于设置 Shell内建命令read的默认超时时间,单位为秒<p>在交互式的Shell中此变量的值作为发出命令后等待用户输入的秒数,如果没有用户输入将会自动退出
BASH|
$OSTYPE|
#SECONDS|脚本已运行秒数
$UID|当前用户的id值
```
# echo $BASH_VERSION,$DISPLAY,$EDITOR,$HISTFILE,$HISTFILESIZE,$HOME,$IFS,$PATH,$PS1,$PWD,$SHELL,$TERM,$TIMEOUT
4.2.46(2)-release,,,/root/.bash_history,1000,/root, ,/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin,[\u@\h \W]$,/root,/bin/bash,xterm,
```


```
#echo $BASH
/bin/bash

#echo $OSTYPE
linux-gnu

#echo $SECONDS
576255

#echo $UID
0
```
## bash的位置参数
Bash中的位置参数是由除0以外的一个或多个数字表示的参数。位置参数是当Shell或Shell的函数被引用时由Shell或Shell函数的参数赋值,并且可以使用Bash的内部命令set来重新赋值.位置参数N可以被引用为${N},或当N只含有个数字时被引用为$N

```
#set 1 2 3 four five six seven eight nine ten
#echo "$1 $2 $3 $4 $5 $6 $7 $8 $9 ${10}"
1 2 3 four five six seven eight nine ten
```
#### 特殊参数
*
```
$*  表示从1开始的所有位置参数组成的字符串
```
@
```
$@  表示从1开始的所有位置参数组成的列表
```

#

```
$#  表示参数的个数
```

?

```
$?  表示前台上一个命令的执行状态
```
0表示成功，1~255表示失败

$

```
$$  表示当前shell的进程号
```
!

```
$!表示最近一次执行的后台命令的进程号
```

0

```
$0  表示脚本本身的名称
```
_

```
$_  表示开始运行的shell或者shell脚本的路径
```
## declare

```
# declare           //显示当前所有的变量
```
参数|说明|样例
---|---|---
-r|只读变量|declare  -i var=1
-i|把变量定义为整数|#declare -i number<p>如果number被赋值为字符串，则值为0；如果赋值为float型，则值取整
-x|通过环境输出到后续命令
-p|显示指定变量的属性和值|#declare -p number<p>declare -i number="4"

## bash的数组
声明
```
#declare -a ARRAY
#declare -ar LINUX=('ubuntu' 'centos' 'redhat')
#declare  LINUX1=('ubuntu' 'centos' 'redhat')
#echo ${LINUX[0]}
ubuntu
#echo ${LINUX1[0]}
ubuntu
```
引用

```
#declare  LINUX1=('ubuntu' 'centos' 'redhat')
#echo ${LINUX1[0]}
ubuntu
#echo ${LINUX1[@]}
ubuntu centos redhat
#echo ${LINUX1[*]}
ubuntu centos redhat
//如果不带序号，默认为0号值
#echo ${LINUX1}
ubuntu
```
取消

```
unset   /readonly不可unset
#unset LINUX1
-bash: unset: LINUX1: cannot unset: readonly variable
```

## 运算符
类似于C语言，含义与结合性都是一样的
操作符|说明|样例
---|---|---
id++    id-- | 变量后自增/自减
++id    --id|变量前自增/自减
+  -        |单目负号和正号
~       !   |逻辑取反和按位取反
**          |求幂
+  -        |加减
<<  >>      |移位操作
<= , >=  ,< ,  > ,  =   !=|比较操作
&   ^   \||按位与、按位异或、按位或
&&  ||      |逻辑与、逻辑或
expr?expr:expr |    三目运算符
expr,expr       |逗号运算符
= *= /= %= += -= <<= >>= &= |= ^=|赋值运算

##算数扩展与let

```
#var=5
# echo $var
5

//这种情况下变量加$不加$效果一样
# echo $((var+5))
10
# echo $(($var+5))
10

# x=17
# y=2
# echo $((x%y))
1
# echo $((17%3))
2

//比较，为假返回0，为真返回1
# echo $((17>3))
1
```

使用let

```
# let i=i+5
# echo $i
5

//支持空格
# let "i = i +5"
# echo $i
10
```

## expr

```
# expr 6 + 8
14

# expr "6 \* 8"
6 \* 8

# expr 6 \* 8
48

# expr 2\>5
2>5

# expr 2 \> 5
0
```

## 退出脚本
- 每一个命令执行完都有退出状态，成功会为0，不成功会有状态码。可以通过$?查看上一条命令的退出状态码

- 可以通过exit N给脚本一个退出状态码

## 脚本调试技巧
#### 打印调试信息
-x选项；-v选项；-xv选项合用
```
# bash -x test.sh
+ echo /usr/bin/bash
/usr/bin/bash
+ echo test.sh
test.sh
+ echo 14962
14962
+ echo

+ echo 0
0

或者set -x  当前会话一直打开调试信息
set +x  当前会话关闭


```

#### 脚本变量
- $LINENO：当前脚本的行数
- $FUNCNAME:包含了当前执行调用堆栈中的所有shell函数名称的数组变量，${FUNCNAME[0]}代表当前正在执行的shell函数名称，，${FUNCNAME[1]}代表调用，${FUNCNAME[0]}的函数的名称，依次类推
- $PS4：bash -x的+就是$PS4，但可以重新定义

```
export PS4='+{$LINENO:${FUNCNAME[0]}}'
```

## shell编程风格
1. 每行不多于80字符
2. 保持一致的缩进深度，程序结构的缩进和逻辑嵌套的深度一致。
3. 每一个代码块之间留一个空行，提高可读性
4. 每个脚本文件必须有一个文件头注释，提供文件名和它的内容
5. 任何一个不简短且不显而易见的函数都要注释
6. 不显而易见的以及重要的代码部分都需要注释