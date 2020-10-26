Python 提供了在程序中与解释器交互的多种方法. 例如 eval 函数将一个字符串作为 Python 表达式求值. 你可以传递一串文本, 简单的表达式, 或者使用内建 Python 函数



## eval

```
In [1]: def cal(str):
    		result = eval(str)
    		print("result is :",result)   

In [2]: cal("1+1")
result is : 2

```

```
In [1]: print(eval("__import__('os').getcwd()"))
C:\Users\Administrator
```

## compile