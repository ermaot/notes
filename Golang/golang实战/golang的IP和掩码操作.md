## 一、处理IP和掩码

```
package main

import (
	"flag"
	"fmt"
	"net"
	"os"
)
var ipaddr  string
var mask int


func init(){
	flag.StringVar(&ipaddr, "i", "", "ip地址,默认为空")
	flag.IntVar(&mask, "m", 32, "子网掩码,默认为32")
}

func getmask(mask uint16) (byte,byte,byte,byte){
	var a,b,c,d uint16 = 0,0,0,0
	if mask <= 8{
		a = mask
	}else if mask <= 16 {
		a,b = 8 ,mask-8
	}else if mask <= 24 {
		a,b,c = 8,8,mask-16
	}	else if mask<=32{
		a,b,c,d = 8,8,8,mask-24
	}
	return (1<<a-1)<<(8-a),(1<<b-1)<<(8-b),(1<<c-1)<<(8-c),(1<<d-1)<<(8-d)
}
func main() {
	flag.Parse()
	addr := net.ParseIP(ipaddr)
	if addr == nil {
		fmt.Println("Invalid addr")
		os.Exit(0)
	} else {
		fmt.Println("The address is",addr.String())
	}
	if (mask >32) || (mask <0){
		fmt.Println("Invalid mask")
		os.Exit(0)
	}
	mask1 := addr.DefaultMask()
	ones1 ,bits1 := mask1.Size()
	network1 := addr.Mask(mask1)
	fmt.Println(ipaddr,mask1.String(),ones1,bits1,network1)
	mask2 := net.IPv4Mask(getmask(uint16(mask)))
	ones2 ,bits2 := mask2.Size()
	network2 := addr.Mask(mask2)
	fmt.Println(ipaddr,mask2.String(),ones2,bits2,network2)
	os.Exit(0)

}

```

保存为ipaddr.go，编译后，运行，输出：

```
> .\ipaddr.exe -i 192.168.0.1 -m 8
The address is 192.168.0.1
192.168.0.1 ffffff00 24 32 192.168.0.0
192.168.0.1 ff000000 8 32 192.0.0.0
```

知识点：

- Parse解析参数，使用-i来指定ip,-m来指定掩码
- DefaultMask()可以获取默认子网掩码，得到的类型是IPMask
- mask.Size()返回两个参数，一个是mask有多少位1，一个是mask共有多少位（IPv4一般都是32）
- net.IPv4Mask()有4个参数，类型为byte，分别是四个点分位
- addr.Mask(mask)返回的是经过掩码遮蔽之后所得到的网络名



