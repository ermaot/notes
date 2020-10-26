## 一、GOLANG 关键字

golang共25个关键字，如下：

序号|关键字|说明
---|---|
1|var|变量的声明
2|const| 常量的声明
3|package|导入
4| import:|导入
5|func|用于定义函数和方法
6|return |用于从函数返回
7|defer|在函数退出之前执行
8|go |用于并行
9|select |用于选择不同类型的通讯
10|interface|用于定义接口
11|struct| 用于定义抽象数据类型
12|break|流程控制
13|case|流程控制
14|continue|流程控制
15|for|流程控制
16|fallthrough|流程控制
17|else|流程控制
18|if|流程控制
19|switch|流程控制
20|goto|流程控制
21|default| 流程控制
22|chan|用于channel通讯
23|type|用于声明自定义类型
24|map|用于声明map类型数据
25|range用于读取slice、map、channel数据

## 二、变量赋值、初始化

1. 使用var定义变量

如果使用var定义，可以自动初始化
类型|自动初始化值
---|---
int,int8,int16,int32|0
string|""
bool|false
byte|0
float32,float64|0.0
complex64,complex128|
uintptr|
map |nil 
channel |nil 
interface| nil 接⼝
function| nil
```
package main
import "fmt"
func main() {
	var i int
	var j string
	var k bool
	fmt.Println(i,j,k)	
}
```
输出结果为0    false
中间其实有一个空字符串，但没有打印出来
2. 如果提供初始化值，则可以不写类型（golang自动类型推断。）
```
package main
import "fmt"
func main() {
	var f = 1.6
	var s = "golang is fun"
	fmt.Println(f)
	fmt.Println(s)
}
```
输出结果为
```
1.6
golang is fun
```
3. 函数内部可以通过:=定义变量
```
package main

import "fmt"
var s2 = "test"
//a := "variable outside function"
func main() {
	var f = 1.6
	var s = "golang is fun"
	fmt.Println(f)
	fmt.Println(s)
	fmt.Println(s2)
	//fmt.Println(a)
}
```

需要注意”函数内部“四个字。当不在函数内部的时候，该变量不被承认。可以参考注释部分的代码，编辑器会提示错误

4. 可以一次定义多个同类型变量（不初始化）；一次定义多个不同类型变量（需初始化）

## 三、常量

常量必须是编译期可以确定的量。这包含两种可能：1.直接量2.可以编译期计算得到的值
```
package main

import "fmt"

func main() {
	 const (
	 	s = "abc"
	 	y
	 	a = len(s)
	 	b
	 )
	 fmt.Println(s,y,a,b)
}
```
输出结果为
```
abc abc 3 3
```
有如下知识点：
1. s  是直接量
2. y 和 b 没有直接赋值，延续上一个变量的值
3. a 的量是通过s计算得到

#### 枚举量

iota定义常量组。从0开始自增；可以多个iota单独自增；如果被打断，需要显式恢复

```
package main

import "fmt"

func main() {
	 const (
	 	a,b = iota,iota
	 	c,d
	 	e,f = 4,5
	 	g,h
	 	i= iota
	 )
	
	fmt.Println(a,b,c,d,e,f,g,h,i)
}
```

输出结果为

```
0 0 1 1 4 5 4 5 4
```

这里有以下知识点：

1. iota初始值为0
2. 可以使用多个iota单独自增
3. 打断后需要显式恢复
4. 恢复的值，要考虑间隔的数量（恢复的i值，不是从打断处恢复，而是加上了间断的行数）
5. 打断后的值，延续上一行的值（g和h延续e和f的值）

## 四、数据类型

#### 1. 基本类型如下：

| 类型          | 长度   | 默认值   | 说明                               |
| ------------- | ------ | -------- | ---------------------------------- |
| bool          | 1      | false    |                                    |
| byte          | 1      | 0        | uint8                              |
| rune          | 4      | 0        | Unicode Code Point, int32          |
| int, uint     | 4或 8  | 0        | 32 或 64 位，根据平台不同                 |
| int8, uint8   | 1      | 0        | -128 ~ 127, 0 ~ 255                |
| int16, uint16 | 2      | 0        | -32768 ~ 32767, 0 ~ 65535          |
| int32, uint32 | 4      | 0        | -21亿 ~ 21 亿, 0 ~ 42 亿           |
| int64, uint64 | 8      | 0        |                                    |
| float32       | 4      | 0.0      |                                    |
| float64       | 8      | 0.0      |                                    |
| complex64     | 8      |          |                                    |
| complex128    | 16     |          |                                    |
| uintptr       | 4 或 8 |          | 以存储指针的 uint32 或 uint64 整数 |
| array         |  |          | 值类型 |
| struct        |  |          | 值类型 |
| string        |        | ""       | UTF-8 字符串                       |
| slice         || nil    | 引用类型 |
| map           | |nil    | 引用类型 |
| channel       || nil    | 引用类型 |
| interface     | |nil    | 接口     |
| function      | |nil    | 函数     |

支持⼋进制、⼗六进制，以及科学记数法。标准库 math 定义了各数字类型取值范围。

```
package main

import (
	"fmt"
	"math"
)

func main() {
	a, b, c, d := 071, 0x1F, 1e9, math.MinInt16
	fmt.Println(math.MaxInt16,math.MaxFloat64,math.MaxInt32)

}
```

输出结果为

```
57 31 1e+09 -32768
1.7976931348623157e+308 2147483647
```

知识点：

1. a为8进制
2. b为16进制
3. c为科学计数
4. math标准库的使用

#### 2.引用类型

引⽤类型包括 slice、map 和 channel。它们有复杂的内部结构，除了申请内存外，还需要初始化相关属性

```
package main

import "fmt"

func main() {
	a := []int{0, 0, 0}     // 提供初始化表达式。
	a[1] = 10
	b := make([]int, 3)     // makeslice
	b[0] = 10
	c := new([10]int )
	c[1] =10
	fmt.Println(a[0],b[0],c[1])
}
```

输出

```
0 10 10
```

知识点：

1. 各种初始化方式，make和new
2. new返回的是指针，可以用c := new([]int )来产生长度(length)为0，容量(cap)为0的数组的地址
3. 初始化时，int值内存默认为0（变量b）

####  3. 类型转换

不支持隐式转换，就算是从低精度向高精度也不行

#### 4. 字符串

- 默认值是空字符串 ""。
- ⽤索引号访问某字节，如 s[i]。
- 不能⽤序号获取字节元素指针，&s[i] ⾮法。
- 不可变类型，⽆法修改字节数组。
- 字节数组尾部不包含 NULL。
- 支持``来存原始字符串
```
package main

import "fmt"

func main() {
	str1 := "abcdefghij"
	str2 := `abcd
efg
hij`
	str3 := str1 + str2
	fmt.Println(str1, string(str1[0]), str1[:3], str1[5:], str1[1:4], str2, str3)
	fmt.Println(&str1,&str2, &str3)
}
```
输出为：
```
abcdefghij a abc fghij bcd abcd
efg
hij abcdefghijabcd
efg
hij
0xc0000841e0 0xc0000841f0 0xc000084200
```
这里有知识点：

- 字符串声明与初始化
- ``在字符串定义中的作用
- 字符串的切片
- println打印单字符时是ascii码，需要转成string才能显示成字符串
- 字符串的连接用法
- 字符串的不变性与连接操作内存的变化

如果需要修改字符串，需先转化成[]rune或者[]byte，再修改完成后转成string

遍历string，也有byte和rune两种方式
```
package main

func main() {
	s := "abcd"
	bs := []byte(s)
	bs[1] = 'B'
	println(string(bs))
	u := "电脑"
	us := []rune(u)
	us[1] = '话'
	println(string(us))
	u2 := "电脑"
	us2 := []byte(u2)
	us2[1] = '1' //us2[1] = '话'会溢出
	println(string(us2))
}
```

输出结果如下:

```
aBcd
电话
�1�脑
a,b,c,æ,±,,å,­,,
a,b,c,汉,字,
```

知识点：

- byte对字符串的转换
- rune对字符串的转换
- 字符串操作的溢出
- 字符串byte遍历
- 字符串rune遍历