## case接受一个参数

```

#!/bin/bash
#testcase for paramter 
#

case "$1" in
    test)
        echo "parameter $1 "
        ;;
    start)
        echo "parameter $1 "
        ;;
    *)
        echo "parameter $1"
        ;;
esac
```

## shift命令处理命令行参数
- shift 命令用于传参时将传递的参数变量左移
- 格式为shift [n]
- 如果n=0，不变，如果n=1，后面的参数依次前移；如果1<n<$#，会丢弃前n-1个参数，第n个参数变成第一个；如果n>$#，不变

```
#!/bin/bash
## testcase for shit
while [ -n "$1" ]
do
    echo "current parameter is : $1"
    shift
done
```

## 使用for循环处理参数

```
#!/bin/bash
index=1
echo "list parameters with \$*:"
##此处$*不能加引号，否则会将参数列表当成一个字符串合并；也可以用*@，可以不用引号
for arg in $*
do
    echo "arg #$index = $arg"
    let index+=1
done
```
运行

```
#sh para_for.sh  test1 test2
list parameters with $*:
arg #1 = test1
arg #2 = test2
```


## 读取脚本名
$0


## 选项处理
不推荐这样处理，因为不够灵活，不能处理不同参数写一起的情况
```

#!/bin/bash
#此处的while如果只用一层[]会报错
while [[ -n "$1" && -n "$2"  ]]
do
    opt=$1
    para=$2
    case $1 in
        -e|-E)
            echo "the e para is $2"
            shift 1
            shift 1
    ;;
         -p|-P)
            echo "the p para is $2"
            shift 1
            shift 1
    ;;
    esac
done
```


## getopts 和getopt


## 获取用户输入
read