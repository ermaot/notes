## 一、不完整代码

```

package main

import (
  // 需在此处添加代码。[1]
  "fmt"
)

var name string

func init() {
  // 需在此处添加代码。[2]
}

func main() {
  // 需在此处添加代码。[3]
  fmt.Printf("Hello, %s!\n", name)
}

```

不修改，只增加代码，要实现”根据运行程序时给定的参数问候某人”的功能，如何做？

## 二、原型1

```
package main

import (
	"flag"
	"fmt"
)

var name string

func init() {
	flag.StringVar(&name, "name", "everyone", "The greeting object.")
}

func main() {
	flag.Parse()
	fmt.Printf("Hello, %s!\n", name)
}
```

输出结果

```
> go run .\eg_var_type.go --help
Usage of C:\Users\ADMINI~1\AppData\Local\Temp\2\go-build310676846\b001\exe\eg_var_type.exe:
  -name string
        The greeting object. (default "everyone")
> go run .\eg_var_type.go -name="robert"
Hello, robert!
```

知识点：

1. flag.StringVar()的使用：接收4个参数

- 第 1 个参数是用于存储该命令参数值的地址，具体到这里就是在前面声明的变量name的地址了，由表达式&name表示。
- 第 2 个参数是为了指定该命令参数的名称，这里是name。
- 第 3 个参数是为了指定在未追加该命令参数时的默认值，这里是everyone。
- 第 4 个函数参数，即是该命令参数的简短说明了，这在打印命令说明时会用到

2. flag.Parse()的使用：

   - 用于真正解析命令参数，并把它们的值赋给相应的变量
   - 对该函数的调用必须在所有命令参数存储载体的声明（这里是对变量name的声明）和设置（这里是在[2]处对flag.StringVar函数的调用）之后，并且在读取任何命令参数值之前进行。
   - 最好把flag.Parse()放在main函数的函数体的第一行

3. Usage of D:\go\GOPATH\test\eg_var_type.exe:的意思是，go run命令构建上述命令源码文件时临时生成的可执行文件的完整路径

   

## 三、原型2

```
package main

import (
	"flag"
	"fmt"
	"os"
)

var name string

func init() {
	flag.StringVar(&name, "name", "everyone", "The greeting object.")
}

func main() {
	flag.Usage = func() {
		fmt.Fprintf(os.Stderr, "Usage of %s:\n", "question")
		flag.PrintDefaults()
	}
	flag.Parse()
	fmt.Printf("Hello, %s!\n", name)
}
```

输出结果:

```
> go run .\eg_var_type.go -name="robert"
Hello, robert!
> go run .\eg_var_type.go --help
Usage of question:
  -name string
        The greeting object. (default "everyone")
```

## 四、原型3

```
package main

import (
	"flag"
	"fmt"
	"os"
)

var name string

func init() {
	flag.CommandLine = flag.NewFlagSet("", flag.ExitOnError)
	flag.CommandLine.Usage = func() {
		fmt.Fprintf(os.Stderr, "Usage of %s:\n", "question")
		flag.PrintDefaults()
	}
	flag.StringVar(&name, "name", "everyone", "The greeting object.")

}

func main() {

	//flag.Usage = func() {
	//	fmt.Fprintf(os.Stderr, "Usage of %s:\n", "question")
	//	flag.PrintDefaults()
	//}
	flag.Parse()
	fmt.Printf("Hello, %s!\n", name)
}
```

输出同上

- 我们在调用flag包中的一些函数（比如StringVar、Parse等等）的时候，实际上是在调用flag.CommandLine变量的对应方法。
- flag.CommandLine相当于默认情况下的命令参数容器。通过对flag.CommandLine重新赋值，我们可以更深层次地定制当前命令源码文件的参数使用说明
- flag.ExitOnError的含义是，告诉命令参数容器，当命令后跟--help或者参数设置的不正确的时候，在打印命令参数使用说明后以状态码2结束当前程序

## 五、另外一个参数获取的程序例子

```
package main

import (
	"flag"
	"fmt"
)

var username , password , host  string
var port int

func init(){
	flag.StringVar(&username, "u", "", "用户名,默认为空")
	flag.StringVar(&password, "p", "", "密码,默认为空")
	flag.StringVar(&host, "h", "127.0.0.1", "主机名,默认 127.0.0.1")
	flag.IntVar(&port, "P", 3306, "端口号,默认为3306")

}

func main() {
	flag.Parse()
	fmt.Printf("username=%v password=%v host=%v port=%v", username, password, host, port)
}
```

