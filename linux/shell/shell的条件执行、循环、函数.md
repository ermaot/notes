## 条件执行
#### test语句
test命令可用于
1. 文件属性测试
2. 字符串测试
3. 算数测试

```
# test -d "/tmp";echo $?
0

//注意[]要留空格
#  [ "abc" == "ced" ] ;echo $?
1

# test 7 -gt 3 && echo True
True
```
操作符|描述
---|---
-e <file>|如果<file>存在则为真
-f <file>|如果<file>存在且是一个常规文件则为真
-d <file>|如果<file>存在且是一个目录则为真
-c <file>|如果<file>存在且是一个特殊字符文件则为真
-b <file>|如果<file>存在且是一个特殊块文件则为真
-p <file>|如果<file>存在且是一个命名管道则为真
-S <file>|如果<file>存在且是一个套接字文件则为真
-L <file>|如果<file>存在且是一个符号链接文件则为真
-h <file>|如果<file>存在且是一个符号链接文件则为真
-g <file>|如果<file>存在且设置了sgid件则为真
-u <file>|如果<file>存在且设置了sgid件则为真
-r <file>|如果<file>存在且可读则为真
-w <file>|如果<file>存在且可写则为真
-s <file>|如果<file>存在且不为空则为真
-t <fd>|如果文件描述符<fd>以打开且引用了一个终端则为真
<file1> -nt <file2>|如果<file1> 比<file2> 新则为真（mtime）
<file1> -ot <file2>|如果<file1> 比<file2> 旧则为真（mtime）
<file1> -ef <file2>|如果<file1>有硬链接到<file2> 则为真

#### 字符串测试操作表
操作符|描述
---|---
-z <STRING>|如果<STRING>为空则为真
-n <STRING>|如果<STRING>不为空则为真
<STRING1> = <STRING2> |如果 <STRING1> 与<STRING2>相同则为真
<STRING1> ！= <STRING2> |如果 <STRING1> 与<STRING2>不相同则为真
<STRING1> < <STRING2> |如果 <STRING1> 在<STRING2>字典顺序之前则为真（ASCII码）
<STRING1> > <STRING2> |如果 <STRING1> 在<STRING2>字典顺序之后则为真（ASCII码）

```
# test -n "" || echo True
True
# test -z "" && echo True
True
#  [ "test1" \< "test2" ] && echo True
True
```
#### 算数测试
操作符|描述
---|---
<INTEGER1> -eq <INTEGER2> |euqal
<INTEGER1> -ne <INTEGER2> |not equal
<INTEGER1> -le <INTEGER2> |less or equal
<INTEGER1> -ge <INTEGER2> |greater or equal
<INTEGER1> -lt <INTEGER2> |less than
<INTEGER1> -gt <INTEGER2> |greater than

```
# [ 5 -lt 10 ] && echo True
True

```

#### [[]] 和[]
[[]] | [] |样例
---|---|---
>|\>|[[a>b]]   <p>[ a\>b ]
<|\<|[[a<b]]   <p>[ a\<b ]

#### read交互获取密码

```
//将读取的密码赋值给pass
# read -sp "enter a password:" pass
enter a password:
# echo $pass
1234
```

#### if then   elif then fi fi

```

if [ 1 -gt 2 ]
then echo "1 greater then 2"
    elif [ 2 -le 3 ]
    then  echo "1 less then 2"
    else echo "ok"
fi
```

## 条件执行
逻辑与：只有当前一个命令执行成功时才执行后一个命令<p>
逻辑或：只有当前一个命令执行失败时才执行后一个命令
#### 逻辑与&&
只有全部条件为真，整个逻辑才是真

#### 逻辑或||
command1 || command2，只有conmmand1返回非0（执行失败）才运行command2

#### 逻辑非!

```
# if [ ! -d test.txt ];then echo "ok";fi
ok
```

#### case语句实例
case EXPRESSION in 
PATTERN1 )
    COMMANDS
;;
PATTERN2 )
    COMMANDS
;;
PATTERN3 | PATTERN4 )
    COMMANDS
;;
esac


## 循环
#### for循环

```
# for i in test1 test2 test3;do echo $i;done
test1
test2
test3

# vars=(1 2 3)
# for i in ${vars[@]};do echo $i;done
1
2
3

# for i in $(ls);do echo $i;done
```

#### while

```
while [ condition ]
do
.....
done
```

```
while IFS= read -r line
do
....
done < "/path/file"

```

```
# while :;do echo "test  test2" ;sleep 3;done
test  test2
```
#### until
until在循环条件为假的情况下才持续执行；until下的循环体至少执行一次
```
until [ condition ]
do
command
done
```

#### select

#### 循环控制
- break
- continue

## 函数定义
#### 函数语法

```
function_name()
{
    commands
    [ return N ]
}
//可以省略圆括号()
```
可以使用unset -f 取消函数的定义

## 参数

```
name(){
    arg1=$1
    arg2=$2
    command on $arg1
}
```
#### 本地变量
1. shell变量一般都是全局的
2. local只能用在函数体内部

#### 从函数文件中调用函数

```
//加载文件里的函数
. /path/to/your/functions.sh
或者
source /path/to/your/functions.sh
```
