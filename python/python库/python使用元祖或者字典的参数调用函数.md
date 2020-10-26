Python允许你实时地创建函数参数列表. 只要把所有的参数放入一个元组中，然后通过内建的 apply 函数调用函数.

```
def function(a, b): 
    print a, b 
 
apply(function, ("whither", "canada?")) 
apply(function, (1, 2 + 3))
```

该函数在python3中已经失效

