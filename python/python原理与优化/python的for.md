**Python 语法**[1] 中对 `for` 的定义是:

```
for_stmt: 'for' exprlist 'in' testlist ':' suite ['else' ':' suite]
```

其中 `exprlist` 指分配目标. 这意味着对可迭代对象中的**每一项都会执行**类似 `{exprlist} = {next_value}` 的操作.

## 对循环的值赋值

```
for i in range(4):
    print(i)
    i = 10
```

这个输出结果是什么？如果按照C语言的语法，在执行for循环的时候，修改了i的值，则会导致循环结束。但实际结果是

```
0
1
2
3

```

因为按照对for的定义，每一次循环都会对i赋值，导致i=10这句没有作用



## for的副作用

```
In [2]: some_str = 'whatthefuck'

In [3]: sum_dict = {}

In [4]: for i ,sum_dict[i] in enumerate(some_str):
   ...:     pass
   ...:     

In [5]: sum_dict
Out[5]: 
{0: 'w',
 1: 'h',
 2: 'a',
 3: 't',
 4: 't',
 5: 'h',
 6: 'e',
 7: 'f',
 8: 'u',
 9: 'c',
 10: 'k'}

```

我们在for循环体里，只有一个pass，似乎什么也没有做。但输出结果说明，for修改了sum_dict。

根据for的定义，实际上for可以转化成这种语句：

```
>>> i, some_dict[i] = (0, 'w')
>>> i, some_dict[i] = (1, 'h')
>>> i, some_dict[i] = (2, 'a')
```

