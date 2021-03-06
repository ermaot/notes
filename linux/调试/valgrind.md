https://www.oschina.net/translate/valgrind-memcheck

系统编程中一个重要的方面就是有效地处理与内存相关的问题。你的工作越接近系统，你就需要面对越多的内存问题。有时这些问题非常琐碎，而更多时候它会演变成一个调试内存问题的恶梦。所以，在实践中会用到很多工具来调试内存问题。

在本文中，我们将讨论最流行的开源内存管理框架 VALGRIND。

> 摘自 Valgrind.org：
>
> Valgrind是用于构建动态分析工具的探测框架。它包括一个工具集，每个工具执行某种类型的调试、分析或类似的任务，以帮助完善你的程序。Valgrind的架构是模块化的，所以可以容易地创建新的工具而又不会扰乱现有的结构。

许多有用的工具被作为标准而提供。

1. Memcheck是一个内存错误检测器。它有助于使你的程序，尤其是那些用C和C++写的程序，更加准确。
2. Cachegrind是一个缓存和分支预测分析器。它有助于使你的程序运行更快。
3. Callgrind是一个调用图缓存生成分析器。它与Cachegrind的功能有重叠，但也收集Cachegrind不收集的一些信息。
4. Helgrind是一个线程错误检测器。它有助于使你的多线程程序更加准确。
5. DRD也是一个线程错误检测器。它和Helgrind相似，但使用不同的分析技术，所以可能找到不同的问题。
6. Massif是一个堆分析器。它有助于使你的程序使用更少的内存。
7. DHAT是另一种不同的堆分析器。它有助于理解块的生命期、块的使用和布局的低效等问题。
8. SGcheck是一个实验工具，用来检测堆和全局数组的溢出。它的功能和Memcheck互补：SGcheck找到Memcheck无法找到的问题，反之亦然。
9. BBV是个实验性质的SimPoint基本块矢量生成器。它对于进行计算机架构的研究和开发很有用处。

也有一些对大多数用户没有用的小工具：Lackey是演示仪器基础的示例工具；Nulgrind是一个最小化的Valgrind工具，不做分析或者操作，仅用于测试目的。

 

### 使用 Valgrind Memcheck

memcheck工具的使用方式如下:

```
valgrind --tool=memcheck ./a.out
```

从上面的命令可以清楚的看到, 主要的命令是valgrind，而我们想使用的工具是通过'-tool'选项来指定的. 上面的‘a.out’指的是我们想使用memcheck运行的可执行文件.

该工具可以检测下列与内存相关的问题 :

- 未释放内存的使用
- 对释放后内存的读/写
- 对已分配内存块尾部的读/写
- 内存泄露
- 不匹配的使用malloc/new/new[] 和 free/delete/delete[]
- 重复释放内存

注意: 上面列出的并不很全面，但却包含了能被该工具检测到的很多普遍的问题.

让我们一个一个地对上面的场景进行讨论:

注意: 下面讨论的所有测试代码都应该使用gcc并且加上-g选项(用来在memcheck的输出中生成行号)进行编译. 就想我们之前讨论过的 [C程序被编译成可执行文件](http://www.thegeekstuff.com/2011/10/c-program-to-an-executable/), 它需要经历四个不同的阶段.



### 1. 使用未初始化的内存

Code :

```
#include <stdio.h>
#include <stdlib.h> 

int main(void)
{
    char *p; 

    char c = *p; 

    printf("\n [%c]\n",c); 

    return 0;
}
```

在上面的代码中，我们尝试使用未初始化的指针 ‘p’.

让我们运行Memcheck来看下结果.

```
$ valgrind --tool=memcheck ./val
==2862== Memcheck, a memory error detector
==2862== Copyright (C) 2002-2009, and GNU GPL'd, by Julian Seward et al.
==2862== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==2862== Command: ./val
==2862==
==2862== Use of uninitialised value of size 8
==2862==    at 0x400530: main (valgrind.c:8)
==2862==

[#]
==2862==
==2862== HEAP SUMMARY:
==2862==     in use at exit: 0 bytes in 0 blocks
==2862==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==2862==
==2862== All heap blocks were freed -- no leaks are possible
==2862==
==2862== For counts of detected and suppressed errors, rerun with: -v
==2862== Use --track-origins=yes to see where uninitialized values come from
==2862== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
```

从上面的输出可以看到,Valgrind检测到了未初始化的变量，然后给出了警告(上面加粗的几行(译者注：貌似上面没有加粗的)).

### 2. 在内存被释放后进行读/写

Code :

```
#include <stdio.h>
#include <stdlib.h> 

int main(void)
{
    char *p = malloc(1);
    *p = 'a'; 

    char c = *p; 

    printf("\n [%c]\n",c); 

    free(p);
    c = *p;
    return 0;
}
```

上面的代码中，我们有一个释放了内存的指针 ‘p’ 然后我们又尝试利用指针获取值.

让我们运行memcheck来看一下Valgrind对这种情况是如何反应的.

```
$ valgrind --tool=memcheck ./val
==2849== Memcheck, a memory error detector
==2849== Copyright (C) 2002-2009, and GNU GPL'd, by Julian Seward et al.
==2849== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==2849== Command: ./val
==2849== 

 [a]
==2849== Invalid read of size 1
==2849==    at 0x400603: main (valgrind.c:30)
==2849==  Address 0x51b0040 is 0 bytes inside a block of size 1 free'd
==2849==    at 0x4C270BD: free (vg_replace_malloc.c:366)
==2849==    by 0x4005FE: main (valgrind.c:29)
==2849==
==2849==
==2849== HEAP SUMMARY:
==2849==     in use at exit: 0 bytes in 0 blocks
==2849==   total heap usage: 1 allocs, 1 frees, 1 bytes allocated
==2849==
==2849== All heap blocks were freed -- no leaks are possible
==2849==
==2849== For counts of detected and suppressed errors, rerun with: -v
==2849== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
```

从上面的输出内容可以看到，Valgrind检测到了无效的读取操作然后输出了警告 ‘Invalid read of size 1′.

另注,[使用gdb来调试c程序](http://www.thegeekstuff.com/2010/03/debug-c-program-using-gdb/).

### 3. 从已分配内存块的尾部进行读/写

Code :

```
#include <stdio.h>
#include <stdlib.h> 

int main(void)
{
    char *p = malloc(1);
    *p = 'a'; 

    char c = *(p+1); 

    printf("\n [%c]\n",c); 

    free(p);
    return 0;
}
```

在上面的代码中，我们已经为‘p’分配了一个字节的内存,但我们在将值读取到 ‘c’中的时候使用的是地址p+1.

现在我们使用Valgrind运行上面的代码 :

```
$ valgrind --tool=memcheck ./val
==2835== Memcheck, a memory error detector
==2835== Copyright (C) 2002-2009, and GNU GPL'd, by Julian Seward et al.
==2835== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==2835== Command: ./val
==2835==
==2835== Invalid read of size 1
==2835==    at 0x4005D9: main (valgrind.c:25)
==2835==  Address 0x51b0041 is 0 bytes after a block of size 1 alloc'd
==2835==    at 0x4C274A8: malloc (vg_replace_malloc.c:236)
==2835==    by 0x4005C5: main (valgrind.c:22)
==2835== 

 []
==2835==
==2835== HEAP SUMMARY:
==2835==     in use at exit: 0 bytes in 0 blocks
==2835==   total heap usage: 1 allocs, 1 frees, 1 bytes allocated
==2835==
==2835== All heap blocks were freed -- no leaks are possible
==2835==
==2835== For counts of detected and suppressed errors, rerun with: -v
==2835== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
```

同样，该工具在这种情况下也检测到了无效的读取操作.

### 4. 内存泄露

Code:

```
#include <stdio.h>
#include <stdlib.h> 

int main(void)
{
    char *p = malloc(1);
    *p = 'a'; 

    char c = *p; 

    printf("\n [%c]\n",c); 

    return 0;
}
```

在这次的代码中, 我们申请了一个字节但是没有将它释放.现在让我们运行Valgrind看看会发生什么：

```
$ valgrind --tool=memcheck --leak-check=full ./val
==2888== Memcheck, a memory error detector
==2888== Copyright (C) 2002-2009, and GNU GPL'd, by Julian Seward et al.
==2888== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==2888== Command: ./val
==2888== 

 [a]
==2888==
==2888== HEAP SUMMARY:
==2888==     in use at exit: 1 bytes in 1 blocks
==2888==   total heap usage: 1 allocs, 0 frees, 1 bytes allocated
==2888==
==2888== 1 bytes in 1 blocks are definitely lost in loss record 1 of 1
==2888==    at 0x4C274A8: malloc (vg_replace_malloc.c:236)
==2888==    by 0x400575: main (valgrind.c:6)
==2888==
==2888== LEAK SUMMARY:
==2888==    definitely lost: 1 bytes in 1 blocks
==2888==    indirectly lost: 0 bytes in 0 blocks
==2888==      possibly lost: 0 bytes in 0 blocks
==2888==    still reachable: 0 bytes in 0 blocks
==2888==         suppressed: 0 bytes in 0 blocks
==2888==
==2888== For counts of detected and suppressed errors, rerun with: -v
==2888== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
```

输出行(上面加粗的部分)显示，该工具能够检测到内存的泄露.

注意: 在这里我们增加了一个选项‘–leak-check=full’来得到内存泄露的详细细节.



 

### 5. 不匹配地使用malloc/new/new[] 和 free/delete/delete[]

Code:

```
#include <stdio.h>
#include <stdlib.h>
#include<iostream> 

int main(void)
{
    char *p = (char*)malloc(1);
    *p = 'a'; 

    char c = *p; 

    printf("\n [%c]\n",c);
    delete p;
    return 0;
}
```

上面的代码中，我们使用了malloc()来分配内存，但是使用了delete操作符来删除内存.

注意 : 使用g++来编译上面的代码，因为delete操作符是在C++中引进的，而要编译C++需要使用g++.

让我们运行来看一下 :

```
$ valgrind --tool=memcheck --leak-check=full ./val
==2972== Memcheck, a memory error detector
==2972== Copyright (C) 2002-2009, and GNU GPL'd, by Julian Seward et al.
==2972== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==2972== Command: ./val
==2972== 

 [a]
==2972== Mismatched free() / delete / delete []
==2972==    at 0x4C26DCF: operator delete(void*) (vg_replace_malloc.c:387)
==2972==    by 0x40080B: main (valgrind.c:13)
==2972==  Address 0x595e040 is 0 bytes inside a block of size 1 alloc'd
==2972==    at 0x4C274A8: malloc (vg_replace_malloc.c:236)
==2972==    by 0x4007D5: main (valgrind.c:7)
==2972==
==2972==
==2972== HEAP SUMMARY:
==2972==     in use at exit: 0 bytes in 0 blocks
==2972==   total heap usage: 1 allocs, 1 frees, 1 bytes allocated
==2972==
==2972== All heap blocks were freed -- no leaks are possible
==2972==
==2972== For counts of detected and suppressed errors, rerun with: -v
==2972== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
```

从上面的输出可以看到 (加粗的行), Valgrind清楚的说明了‘不匹配的使用了free() / delete / delete []‘

你可以尝试在测试代码中使用'new'和'free'进行组合来看看Valgrind给出的结果是什么.



### 6. 两次释放内存

Code :

```
#include <stdio.h>
#include <stdlib.h> 

int main(void)
{
    char *p = (char*)malloc(1);
    *p = 'a'; 

    char c = *p;
    printf("\n [%c]\n",c);
    free(p);
    free(p);
    return 0;
}
```

在上面的代码中, 我们两次释放了'p'指向的内存. 现在让我们运行memcheck :

```
$ valgrind --tool=memcheck --leak-check=full ./val
==3167== Memcheck, a memory error detector
==3167== Copyright (C) 2002-2009, and GNU GPL'd, by Julian Seward et al.
==3167== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==3167== Command: ./val
==3167== 

 [a]
==3167== Invalid free() / delete / delete[]
==3167==    at 0x4C270BD: free (vg_replace_malloc.c:366)
==3167==    by 0x40060A: main (valgrind.c:12)
==3167==  Address 0x51b0040 is 0 bytes inside a block of size 1 free'd
==3167==    at 0x4C270BD: free (vg_replace_malloc.c:366)
==3167==    by 0x4005FE: main (valgrind.c:11)
==3167==
==3167==
==3167== HEAP SUMMARY:
==3167==     in use at exit: 0 bytes in 0 blocks
==3167==   total heap usage: 1 allocs, 2 frees, 1 bytes allocated
==3167==
==3167== All heap blocks were freed -- no leaks are possible
==3167==
==3167== For counts of detected and suppressed errors, rerun with: -v
==3167== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
```

从上面的输出可以看到(加粗的行), 该功能检测到我们对同一个指针调用了两次释放内存操作.

在本文中，我们把注意力放在了内存管理框架Valgrind，然后使用memcheck(Valgrind框架提供的)工具来了解它是如何降低需要经常操作内存的程序员的负担的. 该工具能够检测到很多手动检测不到的与内存相关的问题