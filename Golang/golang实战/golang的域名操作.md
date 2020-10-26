```
package main

import (
	"flag"
	"fmt"
	"net"
	"os"
)
var domainp  string
func init(){
	flag.StringVar(&domainp, "d", "", "dns地址,默认为空")
}

func main() {	
	flag.Parse()

	if domainp != ""{
		domainip ,err:= net.LookupIP(domainp)
		if err ==nil {
			fmt.Println(domainip)
		}else {
			fmt.Println("Invalid ipaddr")
		}
		cname, err:=net.LookupCNAME(domainp)
		if err ==nil {
			fmt.Println(cname)
		}else {
			fmt.Println("Invalid ipaddr")
		}

		os.Exit(0)
	}

}

```

保存为domainp.go,编译后执行，输出结果

```
> .\domain.exe  -d www.baidu.com
[180.101.49.11 180.101.49.12]
www.a.shifen.com.
```



知识点：

- net.LookupIP()可以根据域名查找IP，返回的是一个IP数组
- LookupCNAME()根据域名查找cname