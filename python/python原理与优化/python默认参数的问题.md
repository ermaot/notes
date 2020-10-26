要注意一点，默认参数是在函数之间共享的。这会在调用函数时使用默认参数造成问题

```
In [125]: def append_test(a,listitem=[]):
     ...:     listitem.append(a)
     ...:     return(listitem)
     ...:

In [126]: append_test(1)
Out[126]: [1]

In [127]: append_test(2)
Out[127]: [1, 2]

In [128]: append_test.__defaults__
Out[128]: ([1, 2],)
```

可以看到执行第二次append_test的时候，出现了[1,2]的结果。理论上我们默认参数是[]，不应该是括号中的内容才对吗？

实际上，因为函数有一个\_\_defaults\_\_（python2中是func_defaults)，保存了该函数当前的默认值。也就是说它每一次调用，都可能会被修改，导致下一次调用不一致。

遇到这种情况，需要使用None对象作为占位符

对于对象，查看下面的代码：

```
In [1]: class Test():
   ...:     def __init__(self,name,item=[]):
   ...:         self.name = name
   ...:         self.item = item
   ...:     def set_attr(self,listitem=[]):
   ...:         self.item.append(listitem)
   ...:

In [2]: a=Test('a')

In [3]: b=Test('b')

In [4]: a.set_attr([1])

In [5]: b.item
Out[5]: [[1]]

In [6]: a.__init__.__defaults__
Out[6]: ([[1]],)

In [7]: b.__init__.__defaults__
Out[7]: ([[1]],)
```

可以看到b没有设置属性值，却已经有了值。原因就在于，初始化的时候，对象a和对象b公用了默认值。如果想消除这种影响，可以如下写：

```
In [1]: class Test():
   ...:     def __init__(self,name,item=[]):
   ...:         self.name = name
   ...:         self.item = item
   ...:     def set_attr(self,listitem=[]):
   ...:         self.item.append(listitem)
   ...:

In [5]: a=Test('a',[])

In [6]: b=Test('b',[])

In [7]: a.set_attr([1])

In [8]: b.item
Out[8]: []
```



