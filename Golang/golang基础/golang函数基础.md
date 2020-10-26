## 一、函数定义

不⽀持 嵌套 (nested)、重载 (overload) 和 默认参数 (default parameter)。

- ⽆需声明原型。
- ⽀持不定⻓变参。
- ⽀持多返回值。
- ⽀持命名返回参数。
- ⽀持匿名函数和闭包。

```
package main

import "fmt"

func test(x, y int, s string) (int, string) {      // 类型相同的相邻参数可合并。
	n := x + y                                     // 多返回值必须⽤括号。
	return n, fmt.Sprintf(s, n)
}
func main() {
	a,b := test(1,2,"%v")
	fmt.Println(a,b)
	fmt
}
```

输出：

```
3 3
```

知识点：

- 函数的定义方法（关键字 func 定义函数，左⼤括号依旧不能另起⼀⾏）
- 函数调用方法
- fmt.Sprintf()的使用方法(Sprintf() 是把格式化字符串输出到指定的字符串中，可以用一个变量来接受，然后打印)

### 延展知识点：Printf()、Sprintf()、Fprintf() 函数的区别用法是什么？

　　都是输出格式化字符串，只是输出到的目标不一样：

- Printf() 是把格式化字符串输出到标准到标准输出（一般是屏幕，可以重定向）

- Printf() 是和标准输出文件（stdout）关联的，Fprintf 则没有这个限制

- Sprintf() 是把格式化字符串输出到指定的字符串中，可以用一个变量来接受，然后在打印

- Fprintf() 是把格式字符串输出到指定的文件设备中，所以参数比Printf 多一个文件指针*File主要用于文件操作，Fprintf() 是格式化输出到一个 Stream ,通常是一个文件

下表格出了常用的一些格式化样式中的动词及功能。
动词                   |	                              功能   
---|---
%v|按值的本来值输出
%+v	 |  在 %v 的基础上，对结构体字段名和值进行展开       
%#v|输出 Go 语言语法格式的值
%T|输出 Go 语言语法格式的类型和值
%%|输出 %% 本体
%b|整型以二进制方式显示
%o|整型以八进制方式显示
%d|整型以十进制方式显示
%x|整型以 十六进制显示
%X|整型以十六进制、字母大写方式显示
%U|Unicode 字符
%f|浮点数
%p|指针，十六进制方式显示

### 变参的使用

```
package main

import "fmt"
func test(s string, n ...int) string {
	var x int
	for _, i := range n {
		x += i
	}
	return fmt.Sprintf(s, x)
}
func main() {
	println(test("sum: %d", 1, 2, 3))
	s := []int{1, 2, 3}
    println(test("sum: %d", s...))
}
```

输出：

```
sum: 6
sum: 6
```

变参本质上就是 slice。只能有⼀个，且必须是最后⼀个。