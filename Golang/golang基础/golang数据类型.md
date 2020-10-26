## 一、array

- 数组是值类型，赋值和传参会复制整个数组，⽽不是指针。
- 数组⻓度必须是常量，且是类型的组成部分。[2]int 和 [3]int 是不同类型。
- ⽀持 "=="、"!=" 操作符，因为内存总是被初始化过的。
- 指针数组 [n]*T，数组指针 *[n]T

```
package main

import (
	"fmt"
	"reflect"
)

func main() {
	a := [3]int{1, 2}              // 未初始化元素值为 0。
	b := [...]int{1, 2, 3, 4}      // 通过初始化值确定数组⻓度。
	c := [5]int{2: 100, 4:200}     // 使⽤索引号初始化元素。
	d := [...]struct {
		name string
		age  uint8
	}{
		{"user1", 10},             // 可省略元素类型。
		{"user2", 20},             // 别忘了最后⼀⾏的逗号。
	}
	e := [2][3]int{{1, 2, 3}, {4, 5, 6}}
	f := [...][2]int{{1, 1}, {2, 2}, {3, 3}}    // 第 2 纬度不能⽤ "..."。
	fmt.Println("a value and type is",a,reflect.TypeOf(a))
	fmt.Println("b value and type is",b,reflect.TypeOf(b))
	fmt.Println("c value and type is",c,reflect.TypeOf(c))
	fmt.Println("d value and type is",d,reflect.TypeOf(d))
	fmt.Println("e value and type is",e,reflect.TypeOf(e))
	fmt.Println("f value and type is",f,reflect.TypeOf(f))
}
```

输出结果：

```
a value and type is [1 2 0] [3]int
b value and type is [1 2 3 4] [4]int
c value and type is [0 0 100 0 200] [5]int
d value and type is [{user1 10} {user2 20}] [2]struct { name string; age uint8 }
e value and type is [[1 2 3] [4 5 6]] [2][3]int
f value and type is [[1 1] [2 2] [3 3]] [3][2]int
```

知识点：

- 数组类型初始化有默认值
- 初始化时确定数组长度
- 可以使用索引初始化元素
- [3]int 和[4]int 不是一种类型
- 多维数组的定义与初始化

值拷⻉⾏为会造成性能问题，通常会建议使⽤ slice，或数组指针。



## 二、slice

slice 并不是数组或数组指针。它通过内部指针和相关属性引⽤数组⽚段，以实现变⻓⽅案

slice底层结构

```
runtime.h
struct Slice
{                        // must not move anything
    byte*    array;      // actual data
    uintgo   len;        // number of elements
    uintgo   cap;        // allocated number of elements
};
```

- 引⽤类型。但⾃⾝是结构体，值拷⻉传递。
- 属性 len 表⽰可⽤元素数量，读写操作不能超过该限制。
- 属性 cap 表⽰最⼤扩张容量，不能超出数组限制。
- 如果 slice == nil，那么 len、cap 结果都等于 0。

```
package main

import (
	"fmt"
)

func main() {
	data := [...]int{0, 1, 2, 3, 4, 5, 6,8:80}
	slice := data[1:4:5]
	fmt.Println(slice)
	slice[0],slice[1] = 20,30
	fmt.Println(slice)
	fmt.Println(data,len(data),cap(data))
	s1 := make([] int,6,8)
	fmt.Println(s1,len(s1),cap(s1))
	s1[2] = 9
	fmt.Println(*(&s1[2]))
	data2 := [][]int{
		[]int{1, 2, 3},
		[]int{100, 200},
		[]int{11, 22, 33, 44},
	}
	fmt.Println(data2[0][1],data2[1][0])

}
```

输出为

```
[1 2 3]
[20 30 3]
[0 20 30 3 4 5 6 0 80] 9 9
[0 0 0 0 0 0] 6 8
9
2 100
```

知识点：

- slice定义方法（直接定义法；可以指定序号初始化）
- make定义法（可同时指定len和cap，如果只有一个，就指len且cap=len）
- slice元素的修改方法
- slice修改后，原始数组也跟着一起变化，说明修改的是底层数组
- slice截取（开始，结束，以及最大长度cap），注意没有类似python的步长
- 多维数组定义与取值

##### append

append超过cap的时候，会自动创建新的slice并复制源数据到此处