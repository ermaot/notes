## 子类化内置类型很麻烦
基本上，内置类型的方法不会调用子类覆盖的方法。例如，dict 的子类覆盖的 \_\_getitem\_\_() 方法不会被内置类型的get() 方法调用

```
In [1]: class DoppelDict(dict):
   ...:     def __setitem__(self, key, value):
   ...:         super().__setitem__(key, [value] * 2)
   ...:

In [2]: dd = DoppelDict(one=1)

In [3]: dd
Out[3]: {'one': 1}

In [4]: dd['two'] = 2

In [5]: dd
Out[5]: {'one': 1, 'two': [2, 2]}

In [6]: dd.update(three=3)

In [7]: dd
Out[7]: {'one': 1, 'two': [2, 2], 'three': 3}
```
1.  继承自 dict 的 \_\_init\_\_方法显然忽略了我们覆盖的\_\_setitem\_\_方法：'one' 的值没有重复
2.  [] 运算符会调用我们覆盖的\_\_setitem\_\_方法，按预期那样工作：'two'对应的是两个重复的值，即 [2, 2]
3.  继承自 dict 的 update 方法也不使用我们覆盖的 \_\_setitem\_\_ 方法：'three' 的值没有重复。


```
In [1]: class AnswerDict(dict):
   ...:     def __getitem__(self, key):
   ...:         return 42
   ...:

In [2]: ad = AnswerDict(a='foo')

In [3]: ad['a']
Out[3]: 42

In [4]: d = {}

In [5]: d.update(ad)

In [6]: d['a']
Out[6]: 'foo'

In [7]: d
Out[7]: {'a': 'foo'}
```
可见dict.update 方法忽略了 AnswerDict.\_\_getitem\_\_ 方法

==如果不子类化 dict，而是子类化 collections.UserDict，问题便迎刃而解了==
## 多重继承和方法解析顺序

```
In [8]: class A:
   ...:     def ping(self):
   ...:         print('ping:', self)
   ...: class B(A):
   ...:     def pong(self):
   ...:         print('pong:', self)
   ...: class C(A):
   ...:     def pong(self):
   ...:         print('PONG:', self)
   ...: class D(B, C):
   ...:     def ping(self):
   ...:         super().ping()
   ...:         print('post-ping:', self)
   ...:     def pingpong(self):
   ...:         self.ping()
   ...:         super().ping()
   ...:         self.pong()
   ...:         super().pong()
   ...:         C.pong(self)
   ...:

In [9]: d = D()

In [10]: d.pingpong()
ping: <__main__.D object at 0x000000526428BCF8>
post-ping: <__main__.D object at 0x000000526428BCF8>
ping: <__main__.D object at 0x000000526428BCF8>
pong: <__main__.D object at 0x000000526428BCF8>
pong: <__main__.D object at 0x000000526428BCF8>
PONG: <__main__.D object at 0x000000526428BCF8>

In [11]: d.ping()
ping: <__main__.D object at 0x000000526428BCF8>
post-ping: <__main__.D object at 0x000000526428BCF8>

In [12]: d.pong()
pong: <__main__.D object at 0x000000526428BCF8>

In [13]: D.__mro__
Out[13]: (__main__.D, __main__.B, __main__.C, __main__.A, object)

```
内置的 super()函数会按照\_\_mro\_\_属性给出的顺序调用超类的方法

![左侧继承关系图；右侧类MRO图](pic/python%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84%E4%B9%A0%E6%83%AF%E7%94%A8%E6%B3%95-%E7%BB%A7%E6%89%BF%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B91.png)
## 多重继承的真实应用
Python 标准库中，最常使用多重继承的是 collections.abc 
![Tkinter GUI 类层次结构的 UML 简图](pic/python%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84%E4%B9%A0%E6%83%AF%E7%94%A8%E6%B3%95-%E7%BB%A7%E6%89%BF%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B92.png)
- Toplevel：表示 Tkinter 应用程序中顶层窗口的类。
- Widget：窗口中所有可见对象的超类。
- Button：普通的按钮小组件。
- Entry：单行可编辑文本字段。
- Text：多行可编辑文本字段

## 处理多重继承
#### Tkinter好的、不好的和令人厌恶的方面
## 一个现代示例：Django通用视图中的混入