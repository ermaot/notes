参考文章 https://blog.csdn.net/u014470361/article/details/81193023

在阅读mysql源码的时候，看到这种定义（来自my_compiler.h）

```
#if defined(__cplusplus)
inline bool likely(bool expr) { return expr; }
inline bool unlikely(bool expr) { return expr; }
#else
#define likely(x) (x)
#define unlikely(x) (x)
#endif
```

你会看到这是一个inline函数，而且都是直接返回表达式的值。那这样写有什么用呢？直接写表达式不好吗？

而在linux内核代码中，也存在类似现象

比如/include/linux/compiler.h */中有代码段

```
# define likely(x)  __builtin_expect(!!(x), 1)
# define unlikely(x)    __builtin_expect(!!(x), 0)
```

1. __builtin_expect是gcc中提供的一个预处理命令，可以直接引用而不需要加头文件。
2. __builtin_expect(!!(x), 1)表示`x == 1`这种情况是“经常发生的”或是“很可能发生的”
3. __builtin_expect(!!(x), 0)表示`x == 0这种情况是“经常发生的”或是“很可能发生的”
4. !!(x)将x（不管是逻辑值还是非逻辑值）转化成逻辑值



```
下文本段来自https://blog.csdn.net/qq_25077833/java/article/details/53344249
```

/kernel/shed.c中有一段：

使用likely后 ，执行if后面语句的可能性大些，==编译器将if{}里的内容编译到前面。==

使用unlikely后，执行else后面语句的可能性大些，==编译器将else{}里的内容编译到前面。==

这样便有利于cpu预取，提高预取指令的正确率，因而可提高效率。




#### *总之，likely与unlikely互换或不用都不会影响程序的正确性。但可能会影响程序的效率*