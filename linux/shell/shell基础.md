## shell 登录相关文件
级别|文件名|说明
---|---|---
系统全局|/etc/profile|系统级初始化文件，定义了一些环境变量，由登录shell调用
系统全局|/etc/bash.bashrc 或 /etc/bashrc |名称根据linux发行版而异，每个交互式shell的系统级启动脚本，定义了一些函数和别名
系统全局|/etc/bash.logout|系统级的登录shell清理脚本，退出shell时执行；部分linux发行版默认没有此文件
用户级|$HOME/.bash_profile<p> $HOME/.bash_login<p>$HOME/.profile<p>| 用户个人初始化脚本，用户登录时执行。三个只有一个会被执行，按照顺序查找，执行找到的第一个
用户级|$HOME/.bash_logout|用户退出shell的清理脚本
用户级|$HOME/.inputrc|用户个人的由readline使用的启动脚本，定义了处理某些情况下的键盘映射

## bash启动脚本
文件名|说明
---|---
 /etc/profile|当用户在运行级别3登录系统时首先运行.
/etc/profile.d|当/etc/profile运行时,会调用该目录下的一些脚本.
$HOME/.bash_profile<p>$HOME/bash_login<p>$HOME/profile|在/etc/profile运行后,第一个存在的被运行.
$HOME/.bashrc|上述脚本的中一个运行后即调用此脚本.
/etc/bashrc<p>/etc/bash.bashrc|由 $HOME/bashrc调用运行.


- 当一个个交互式的非登录 Shell启动时,Bash将读取并运行如下脚本

文件名|说明
---|---
$HOME/.bashrc|如果此文件存在即被运行.
/etc/bashrc|将被 $HOME/.bashrc调用运行
/etc/profile.d|此目录下的脚本将被/etc/bash或 /etc/bash.bashrc调用运行

## bash启动脚本主要设置的环境
1. 设置环境变量PATH和PS1
2. 通过变量 EDITOR 设置默认的文本编辑器
3. 设置默认的 umask(文件或目录的权限属性);
4. 覆盖或移除不想要的变量或别名
5. 设置别名;
6. 加载函数.

## 定制属于自己的启动脚本
#### 启动脚本

```
//.bash_profile

//.bashrc

//用户相关环境变量

//一些变量定义与获取
```

#### 退出脚本

```
//.bash_logout

//清理历史文件(比如mysql的历史)
# /bin/rm $HOMR/.mysql_history

//其他操作
/bin/backup.sh 
```

## 有效的shell

```
# cat /etc/shells 
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
```

```
# which bash
/usr/bin/bash
```
## shell变量
shell变量分为系统变量和用户自定义变量
#### 系统变量
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

```
# echo $BASH_VERSION,$DISPLAY,$EDITOR,$HISTFILE,$HISTFILESIZE,$HOME,$IFS,$PATH,$PS1,$PWD,$SHELL,$TERM,$TIMEOUT
4.2.46(2)-release,,,/root/.bash_history,1000,/root, ,/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin,[\u@\h \W]$,/root,/bin/bash,xterm,
```
- env 或者printenv

```
# env
XDG_SESSION_ID=31471
HOSTNAME=izm5edbv563hlvcbf71ophz
TERM=xterm
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=212.64.81.71 58980 22
SSH_TTY=/dev/pts/2
QT_GRAPHICSSYSTEM_CHECKED=1
USER=root
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:
MAIL=/var/spool/mail/root
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
PWD=/root
LANG=en_US.UTF-8
QT_GRAPHICSSYSTEM=native
HISTCONTROL=ignoredups
SHLVL=1
HOME=/root
LOGNAME=root
SSH_CONNECTION=212.64.81.71 58980 172.31.180.77 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/0
_=/usr/bin/env
```

## shell变量赋值
- =符号即可，但有需要注意的地方

```
# test="test"
# echo $test
test

# test=test
# echo $test
test


```
= 左右不要有空格，否则会报command not found错误（也要看具体的shell）

```
//等号左侧有空格
# test1 =test
-bash: test1: command not found

//等号右侧有空格
# test1= test
# echo $test1
test
```
- 赋值默认是字符串赋值

```
# var=1
# var2=$var+1
# echo $var2
1+1

```
如果想算数表达式赋值，则

```
# let var2=$var+1
# echo $var2
2
```
把变量赋给另外变量

```
# var3=$var2
# echo $var3
2
```

命令的执行结果赋给变量（``或者$()）

```
# var=`pwd`
# echo $var
/root

# var=$(pwd)
# echo $var
/root
```
接受控制台输入

```
# echo "Enter a variable";read var4
Enter a variable
test4
# echo $var4
test4
```

#### 变量命名规则
1. 字母或者下划线开头
2. 大小写敏感

#### echo 与printf
printf类似于C语言的printf，格式是 printf <格式控制>  <变量>

```
# printf "eg:%s\n" "$var"
eg:test
```
格式控制

分类符|描述
---|---
%b|打印相关参数并解释其中带有反斜杠"\"的特殊字符
%q|以shell引用的格式打印相关参数,使其可以在标准输入中重用
%d|以带符号十进制数的格式打印相关参数
%i|与%d相同
%o|以无符号八进制数的格式打印相关参数
%u|以无符号十进制数的格式打印相关参数
%x|以无符号小写十六进制数的格式打印相关参数
%X|与%x相同,只是十六进制数为大写
%f|以浮点数的格式解析并打印相关参数
%e|以双精度浮点数<N>±e<N>的格式打印相关参数
%E|与%e相同,只是用大写字母E
%g|以%f或%e的格式打印相关参数
%G|以%f或%E的格式打印相关参数
%c|以字符的格式打印相关参数,并且只打印参数中的第一个字符
%S|以字符串的格式打印相关参数
%n|指定打印的字符个数
%%|表示打印一个字符“%”


转义字符表
转义符|描述
---|---
\\"|打印双引号
\NNN|用八进制的值表示一个ASCⅡ字符,例如101,即65,表示字符"A"
\\|打印一个反斜杠"\\"
\a|发出告警音
\b|删除前一个字符
\f|换页符,在某些实现中会清屏,有些会换行
\n|换行
\r|从行头开始,和换行不一样,仍在本行
\t|Tab键
\v|竖直tab,和\f相似,不同机器显示有所不同,通常会引起换行VERTICAL TAB or CTRL-K
\xHH|用十六进制的值表示一个ASCI字符,例如x41,即65,表示字符"A"


## 变量引用
- 最好用双引号将变量名括起来，防止变量中的特殊字符被解释成其他含义，也可防止多个单词分离

```
//把变量分成3份传入
# for var in $LIST;do echo $var;done
one
two
three

//把变量当整体
# for var in "$LIST";do echo $var;done
one two three
```

## export
export -fnp [变量或者函数名称]=[变量设置值]
- -f选项表示 export一个函数
- -n选项表示将 export属性从指定变量或函数上移除
- -p选项打印当前Shel所有输出的变量,与单独执行 export命令结果相同.

## 删除变量
unset删除响应的变量或者函数

```
unset [-fv]  [变量或者函数名称]
-f 删除一个已定义的函数
-v 删除一个变量
```

## shell历史命令
- bash将历史命令保存在缓冲区或者默认文件~/.bash_history中
- 保存命令的多少由环境变量HISTSIZE定义
- 使用ctrl + r然后输入关键字可以搜索历史命令
- history命令可以显示全部的历史命令
- !!可以直接执行上一条命令
- !接着关键字可以搜索命令并执行
- history执行后，找到历史命令的序号，用!加上序号即可执行历史命令

```
# history

 1124  var=$(pwd)
 1125  history
#！1125
```

## shell扩展
shell扩展有8种：==大括号扩展、波浪号扩展==、参数和变量扩展、==命令替换==、算数扩展、进程替换、单词拆分、==文件名扩展==
#### 大括号扩展

```
# echo {0..10}
0 1 2 3 4 5 6 7 8 9 10
# echo {5..-3}
5 4 3 2 1 0 -1 -2 -3
# echo {g..a}
g f e d c b a
# echo {g..a}{1..5}
g1 g2 g3 g4 g5 f1 f2 f3 f4 f5 e1 e2 e3 e4 e5 d1 d2 d3 d4 d5 c1 c2 c3 c4 c5 b1 b2 b3 b4 b5 a1 a2 a3 a4 a5
```
嵌套大括号

```
# echo a{{1,2,3}a,{a,b,c}1}g
a1ag a2ag a3ag aa1g ab1g ac1g

# echo a{{1..3}a,{a..c}1}g
a1ag a2ag a3ag aa1g ab1g ac1g

//一次同时创建3个文件夹
# mkdir {dir1,dir2,dir3}
# ls -d dir*
dir1  dir2  dir3
```
bash4.0后可以设置步长

```
# echo {1..10..2}
1 3 5 7 9

# echo {a..z..5}
a f k p u z

# echo {0001..10..2}
0001 0003 0005 0007 0009
```

#### 波浪号扩展

```
//进入家目录
# cd ~

//进入test用户的主目录
# cd ~test


```
#### 命令扩展

```
`command`
$(COMMAND)
```

#### 文件名扩展
bash没有设置-f选项，支持文件名扩展
- *：匹配任何字符串，包括空字符串
- ?:匹配任意单字符
- [...]：匹配方括号内的任意字符

```
# ls *.zip
mysql-8.0.15.zip  sakila-db.zip

# ls -d dir?
dir1  dir2  dir3

# ls -d dir[123]
dir1  dir2  dir3

# ls -d buil?
build
```

## 创建和使用别名
在~/.bashrc文件中，可以设置命令的别名

```
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
```
使用unalias删除别名

```
# alias ll='ls -l'
# unalias ll
```
临时使用非alias

```
\aliasname
```
## bash提示符

```
$echo $PS1
[\u@\h \W]$

```
符号|解释
---|---
\u|当前用户的用户名
\h|主机名.
\w|当前工作目录的全路径.
\n|新的一行.
\$|如果当前用户UID是0,则显示符号"#",否则显示符号"$"


```
[root@vm ~]#
root是u，\h是主机名,\W是工作路径，与[\u@\h \W]$一一对应
```

## shell选项

```
#set -o
allexport      	off
braceexpand    	on
emacs          	on
errexit        	off
errtrace       	off
functrace      	off
hashall        	on
histexpand     	on
history        	on
ignoreeof      	off
interactive-comments	on
keyword        	off
monitor        	on
noclobber      	off
noexec         	off
noglob         	off
nolog          	off
notify         	off
nounset        	off
onecmd         	off
physical       	off
pipefail       	off
posix          	off
privileged     	off
verbose        	off
vi             	off
xtrace         	off
```
set -x 打开调试选项

- shopt

```
shopt -s feature-name               //打开一个选项
shopt -u feature-name               //关闭一个选项
```
作用|命令
---|---
#纠正目录拼写|shopt -g-s aspell
#当终端窗口大小改变时,确保显示得到更新|shopt -g -s checkwinsize
#开启扩展模式匹配特性|shopt -g -s extglob
退出时追加而不是重启命令历史|shopt -s histappend
#使Bash尝试保存历史记录中多行命令的所有行|shopt -g -s cmdhist
#得到后台任务结束的及时通知|set -o notify