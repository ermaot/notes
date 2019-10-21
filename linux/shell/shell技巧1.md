## 一些小细节
- shebang：指定执行的shell，文件头部写
```
//如果是bash，则如下写
#!/bin/bash
```

- 打开一个终端，会执行一组命令来定义如提示文本、颜色等设置，来自于~/.bashrc
- bash还维护了一个历史命令文件~/.bash_history
- bash中命令是通过分号或者换行符分隔
- 使用单引号的echo，bash不会对单引号的变量求值($var)，按照原样显示
```
# test="test@"
# echo '$test'
$test
# echo "$test"
test@
```
- !在单引号和双引号显示也不同
```
# echo '$test!'
$test!
//实际上执行了history命令中最后一条命令
# echo "$test!"
echo "$test"test1" > "test2" && echo True
> 
```

- printf使用技巧
```
# printf "%-5s %-10s %-4s\n" No Name Mark
No    Name       Mark
# printf "%-5s %-10s %-4.2f\n" 1 Sarath 80.3456 
1     Sarath     80.35
```
格式控制|说明
---|---
%s|字符串
%c|字符
%d|整数
%f|浮点数

%-5s：-表示左对齐（默认为右对齐）；5代表宽度；s代表字符串输出

%-4.2f：.2指定保留2个小数位

- echo的 -e参数代表转义
```
# echo -e "1\t2\t3"
1	2	3
```
- echo -e可以打印彩色字符
1. 前景色
颜色码：重置=0,黑色=30,红色=31,绿色=32,黄色=33,蓝色=34,洋红=35,青色=36,白色=37
```
# echo -e "\e[1;31m This is red text;\e[0m"
 This is red text;
```

2. 背景色
颜色码是:重置=0,黑色=40,红色=41,绿色=42,黄色=43,蓝色=44,洋红=45,青色=46,白色=47
```
# echo -e "\033[1;41m This is red text;\e[0m"
 This is red text;
```
- 环境变量
```
//cat /proc/$pid/environ
# cat /proc/835/environ 
USER=rootLOGNAME=rootHOME=/rootPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/binMAIL=/var/mail/rootSHELL=/bin/bashSSH_CLIENT=212.64.81.71 65210 22SSH_CONNECTION=212.64.81.71 65210 172.31.180.77 22SSH_TTY=/dev/pts/2TERM=xtermXDG_SESSION_ID=32915XDG_RUNTIME_DIR=/run/user/0[
```
```
export PATH=$PATH:/usr/local/test
或者
# PATH=$PATH:/usr/local/test
#export PATH
```
- 获取字符串长度
```
# var="qwertyyuiop"
# echo ${#var}
11
```
- 识别当前shell版本
```
# echo $SHELL
/bin/bash
或者
# echo $0
-bash
```
- 判断当前用户是否是root(UID是否是0)
```
echo $UID
```

- 修改提示字符串（修改$PS1变量）
```
# echo $PS1
[\u@\h \W]\$
```


符号|解释
---|---
\u|当前用户的用户名
\h|主机名.
\w|当前工作目录的全路径.
\n|新的一行.
\$|如果当前用户UID是0,则显示符号"#",否则显示符号"$"

## shell的计算
- let

let不需要加$
```
# let a=1
# let a=a+1
# echo $a
2
```
- []
```
# a=1
# b=$[a+1]              //可以用# b=$[$a+1]
# echo $b

```
-$(())
```
# a=1
# b=$((a+1))            //可以用# b=$(($a+1))
# echo $b
2

```

- bc
```
# echo "4*0.56"  | bc
2.24

//赋值
# test=$(echo "4*0.56"  | bc)
# echo $test

//设定精度范围
# echo "scale=2;2/3" | bc
.66

//这个有点奇怪
# echo "scale=2;4*0.9999" | bc
3.9996

# echo "scale=4;sqrt(2)" | bc
1.4142

# echo "scale=10;1.234^10" | bc
8.1875053535
```
## 数组与相关数组
关联数组相当于python的字典
```
# array=(1 2 3 4 5 6)
# echo ${array[0]}
1
# echo ${array[@]}
1 2 3 4 5 6
# echo ${array[*]}
1 2 3 4 5 6
# array[6]=7
# echo ${array[*]}
1 2 3 4 5 6 7

//可以看到shell的数组是弱类型
# array[7]="ttest"
# echo ${array[*]}
1 2 3 4 5 6 7 ttest

//数组名后不接序号，默认为0
# echo ${#array}
1
//数组长度
# echo ${#array[*]}
8
```
关联数组
```
# declare -A array_a
# array_a["test1"]=1
# array_a["test2"]=2
# array_a["test3"]=3
# echo ${array_a["test3"]}
3
# echo ${array_a["test2"]}
2
# echo ${array_a["test1"]}
1
```
列出数组索引（对关联数组和普通数组同样适用）
```
# echo ${!array_a[@]}
test1 test2 test3
# echo ${!array[*]}
0 1 2 3 4 5 6 7
```
## 获取终端信息
```
//打印当前终端的列数和行数
# tput cols
179
# tput lines
22

//打印当前终端名
# tput longname
xterm terminal emulator (X Window System)

//将光标移动到方位（100，100）
# tput cup 100 100


```

设置终端背景色（取值为0~7），为10的时候与0相同

![image](13DA6658DEE34D82A8D05E139C59E9DA)


```
# tput bold             //设置粗体
# tput sgr0             //恢复正常设置


# tput smul             //下划线开始
# tput rmul             //下划线结束


```

用stty不回显密码（-echo禁止将输出发送到终端）

```
#!/bin/sh
#Filename: password. sh
echo -e "Enter password:
stty -echo
read password
stty echo
echo
echo Password read
```


## date命令

```
# date
Thu Aug 29 16:42:09 CST 2019

//从1970年1月1日0点开始的秒数
# date +%s
1567068141
# date --date "Thu Jun 29 16:42:09 CST 2018" +%s
1530261729

# date --date "Thu Jun 29 16:42:09 CST 2018" +%A
Friday
# date --date "Thu Jun 29 16:42:09 CST 2018" +%a
Fri


# date "+%d %B %Y"
29 August 2019
```
日期内容 |格式  
---|---
星期 |    %a(例如:Sat)        
星期 |    %A(例如: Saturday)  
月 |      %b(例如:Nov)        
月 |      %B(例如: November)  
日 |      %d(例如:31)         
固定格式日期(mm/ddy) |%D(例如:10/1810)    
年 |      %y(例如:10)         
年 |      %Y(例如:2010)       
小时 |    %I或%H(例如:08)     
分钟 |    %M(例如:33)         
秒 |      %S(例如:10)         
纳秒 |    %N(例如:695208515)  
UNX纪元时( 以秒为单位)|%s(例如:1290049486)

## 调试脚本

```
# bash -x  script.sh
# sh -x  script.sh
# set -x
# set +x
# set -v
# set +v
```
如果在脚本开头加#!/bin/bash -xv 可以直接开启调试功能


## fork炸弹

```
:(){:|:&};:
```

```
:()
{
:|: &
}
;
:
```


* 第 1 行说明下面要定义一个函数，函数名为冒号，没有可选参数。
* 第 2 行表示函数体开始。
* 第 3 行是函数体真正要做的事情，首先它递归调用本函数，然后利用管道调用一个新进程（它要做的事情也是递归调用本函数），并将其放到后台执行。
* 第 4 行表示函数体结束。
* 第 5 行并不会执行什么操作，在命令行中用来分隔两个命令用。从总体来看，它表明这段程序包含两个部分，首先定义了一个函数，然后调用这个函数。
* 第 6 行表示调用本函数。

==可以使用/etc/security/limits.conf来限制可生成的最大进程数避开==

## 导出函数
函数也能像环境变量一样用export导出,如此一来,函数的作用域就可以扩展到子进程中,如下:

```
export -f fname
```

## $?
获取命令的返回值

## 保留换行符

```
# cat newline.txt 
test1
test2
test3

//丢失了换行符
# echo $(cat newline.txt)
test1 test2 test3

# echo "$(cat newline.txt)"
test1
test2
test3
```
