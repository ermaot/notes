## gdb简介
gdb命令行调试工具非常强大，是linux下调试的神器。可以完成四个方面的功能
1. 启动程序，按照自定义的要求运行程序
2. 设置断点，在所需位置停止
3. 在断点出检查程序所发生的事情
4. 动态改变程序的执行环境

## gdb初试
写一段简单的代码sum.c，用gdb调试看看
```
#include <stdio.h>
int func(int n)
{
    int sum=0,i;
    for(i = 0; i < n; i++)
    {
        sum += i;
    }
    return sum;
}
int main()
{
    int i;
    long result = 0;
    for(i = 1; i <= 100;i++)
    {
        result += i;
    }
    printf("result[1-100] = %d\n",result);
    printf("result[1-250]=%d\n",func(250));
}
```
1. 编译sum.c
```
gcc -g sum.c -o sum
```
2. gdb调试
```
(gdb) l 0                                                   //显示第0行代码
1	#include <stdio.h>
2	int func(int n)
3	{
4	    int sum=0,i;
5	    for(i = 0; i < n; i++)
6	    {
7	        sum += i;
8	    }
9	    return sum;
10	}
(gdb)                                                      //回车，代表与上一条命令相同
11	int main()
12	{
13	    int i;
14	    long result = 0;
15	    for(i = 1; i <= 100;i++)
16	    {
17	        result += i;
18	    }
19	    printf("result[1-100] = %d\n",result);
20	    printf("result[1-250] = %d\n",func(250));
(gdb) 
21	}
(gdb) break 16                                    //第16行插入断点
Breakpoint 3 at 0x400564: file sum.c, line 16.
(gdb) b func                                        //func函数处插入断点，b是break的简写
Breakpoint 4 at 0x400524: file sum.c, line 4.
(gdb) run                                            //运行程序，直到断点处  
Starting program: /root/sum 

Breakpoint 3, main () at sum.c:17
17	        result += i;
(gdb) n                                              //单条语句执行
15	    for(i = 1; i <= 100;i++)
(gdb) continue                                   //执行到下一个断点
Continuing.

Breakpoint 3, main () at sum.c:17
17	        result += i;
(gdb) print i                                        //打印变量的值
$5 = 2
(gdb) p result                                    //打印变量的值，p是print的简写
$6 = 1                                                //

```
bt可以查看堆栈，finish退出函数，quit退出gdb

## gdb使用
#### gdb的启动方法
1. gdb <program>
program是程序文件，一般在当前目录下
2. gdb <program> core
gdb同时调试程序和coredump文件
3. gdb <program> <pid>
gdb附着到已经运行的进程上调试

#### gdb命令
命令|缩写|说明
---|---|---
breakpoint|b|设置断点
continue|c|继续执行至下一个断点，直到程序终止。如果给出了参数n且从断点处继续执行，则只有在第n次通过后才在该断点停止
next|n|不进入子程序调用，执行下一行。如果给出了参数n，则执行n次，或直到程序因其他原因停止
step|s|进入子程序调用，执行下一行。如果给出了参数n，则执行n次，或直到程序因其他原因停止
finish| fin|一直执行，直到当前子例行程序返回
backtrace| bt|打印堆栈轨迹。对于全参数，还会输出各个堆栈框架中的局部变量的值
enable|ena|启用被禁用的断点
disable|disa|禁用断点
info breakpoints| i b|打印断点列表及各断点详细信息
info line| i li|如果省略参数则打印当前正在调试的行的信息，否则打印特定行的信息
print|p|打印被指定为参数的变量或表达式的值
list|l|根据参数的值打印源代码
info local |i lo|显示当前框架的局部变量的值
up|u|p选择并打印当前框架的上层框架的局部变量。参数会指出向上移动的框架的数量默认值为1
down |down|选择并打印当前框架的下层框架的局部变量。参数会指出向下移动的框架的数量默认值为1
info threads| i th|列出正在运行的所有线程及各线程的一些信息
thread |thr|切换到指定线程
info registers| i r|打印处理器寄存器的内容
disassemble| disas|反汇编指定代码段。如果没有给出参数，则反汇编当前堆栈框架pc寄存器外围的可执行代码