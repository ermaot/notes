## python 文化中的接口和协议
- 鸭子类型

```
In [1]: class Duck():
   ...:   def walk(self):
   ...:     print('I walk like a duck')
   ...:   def swim(self):
   ...:     print('i swim like a duck')
   ...:
   ...: class Person():
   ...:   def walk(self):
   ...:     print('this one walk like a duck')
   ...:   def swim(self):
   ...:     print('this man swim like a duck')
   ...:

In [2]: d = Duck()

In [3]: p = Person()

In [4]: def test(test):
   ...:     test.walk()
   ...:     test.swim()
   ...:

In [5]: test(d)
I walk like a duck
i swim like a duck

In [6]: test(p)
this one walk like a duck
this man swim like a duck

```
1. Person类拥有跟Duck类一样的方法，当有一个函数调用Duck类，并利用到了两个方法walk()和swim()。我们传入Person类也一样可以运行，函数并不会检查对象的类型是不是Duck，只要他拥有walk()和swim()方法，就可以正确的被调用。 
2. 如果一个对象实现了__getitem__方法，那python的解释器就会把它当做一个collection，就可以在这个对象上使用切片，获取子项等方法；如果一个对象实现了__iter__和next方法，python就会认为它是一个iterator，就可以在这个对象上通过循环来获取各个子项

- 接口在python中的运作
1. 首先，Python语言没有interface关键字，而且除了抽象基类，每个类都有接口：类实现或继承的公开属性（方法或数据属性），包括特殊方法，如\_\_getitem\_\_ 或 \_\_add\_\_
2. 受保护的属性和私有属性不在接口中,虽然他们很容易被访问到类实现或继承的公开属性（方法或数据属性）
[原文链接](https://blog.csdn.net/zhchs2012/article/details/79273109)

访问私有属性和为私有属性直接赋值
```
In [1]: class test:
   ...:     def __init__(self,x,y):
   ...:         self.__x = x
   ...:         self.__y = y
   ...:     def getx(self):
   ...:         return self.__x
   ...:     def gety(self):
   ...:         return self.__y
   ...:

In [2]: a = test(1,2)

In [3]: a.getx(),a.gety()
Out[3]: (1, 2)

In [4]: a.__x = 3

In [5]: a.__dict__
Out[5]: {'_test__x': 1, '_test__y': 2, '__x': 3}

In [6]: a._test__x = 3

In [7]: a._test__x
Out[7]: 3

In [8]: a.__dict__
Out[8]: {'_test__x': 3, '_test__y': 2, '__x': 3}
```

## python喜欢序列

```
In [1]: class itertest:
   ...:     def __init__(self,x):
   ...:         self.list = x
   ...:     def __getitem__(self,position):
   ...:         return self.list[position]
   ...:     def __iter__(self):
   ...:         return (i for i in self.list)
   ...:

In [2]: a = itertest([1,2,3,4])

In [3]: a[1]
Out[3]: 2

In [4]: a[:]
Out[4]: [1, 2, 3, 4]

In [5]: for i in a:
   ...:     print(i)
   ...:
1
2
3
4
```
发现有__getitem__ 方法时，Python会调用它，传入从0开始的整数索引，尝试迭代对象，所以上述代码只需要如下即可运行：

```
In [1]: class itertest:
   ...:     def __init__(self,x):
   ...:         self.list = x
   ...:     def __getitem__(self,position):
   ...:         return self.list[position]
   ...:

In [2]: a = itertest([1,2,3,4])

In [3]: for i in a:
   ...:     print(i)
   ...:
1
2
3
4

In [4]: a[1:3]
Out[4]: [2, 3]
```

## 使用猴子补丁在运行时实现协议

```
In [1]: class itertest:
   ...:     def __init__(self,x):
   ...:         self.list = x
   ...:

In [2]: def __getitem__(itertest,position):
   ...:     return itertest.list[position]
   ...:


In [4]: itertest.__getitem__ = __getitem__

In [5]: a = itertest([1,2,3,4])

In [6]: for i in a:
   ...:     print(i)
   ...:
1
2
3
4
```

## alex martelli的水禽
## 定义抽象基类的子类
## 标准库中的抽象基类
1. 从 Python 2.6 开始，标准库提供了抽象基类。大多数抽象基类在collections.abc 模块中定义，且最常用
2. 其他地方也有，例如，numbers和 io包中有一些抽象基类

#### collections.abc模块中的抽象基类
ABC|	Inherits from	|Abstract Methods|	Mixin Methods
---|---|---|---
Container|	|	\_\_contains\_\_|	
Hashable|	|	\_\_hash\_\_|	
Iterable|	|	\_\_iter\_\_|	
Iterator|	Iterable|	\_\_next\_\_|	\_\_iter\_\_
Reversible|	Iterable|	\_\_reversed\_\_|	
Generator|	Iterator|	send,throw|	close,\_\_iter\_\_,\_\_next\_\_
Sized|	|	\_\_len\_\_|	
Callable|	|	\_\_call\_\_|	
Collection|	Sized,Iterable,Container|	\_\_contains\_\_,\_\_iter\_\_,\_\_len\_\_|	
Sequence|	Reversible,Collection|	\_\_getitem\_\_,\_\_len\_\_|	\_\_contains\_\_,\_\_iter\_\_,\_\_reversed\_\_,index, andcount
MutableSequence|	Sequence|	\_\_getitem\_\_,\_\_setitem\_\_,\_\_delitem\_\_,\_\_len\_\_,insert	InheritedSequencemethods andappend,reverse,extend,pop,remove, and\_\_iadd\_\_
ByteString|	Sequence|	\_\_getitem\_\_,\_\_len\_\_|	InheritedSequencemethods
Set|	Collection|	\_\_contains\_\_,\_\_iter\_\_,\_\_len\_\_|	\_\_le\_\_,\_\_lt\_\_,\_\_eq\_\_,\_\_ne\_\_,\_\_gt\_\_,\_\_ge\_\_,\_\_and\_\_,\_\_or\_\_,\_\_sub\_\_,\_\_xor\_\_, andisdisjoint
MutableSet|	Set|	\_\_contains\_\_,\_\_iter\_\_,\_\_len\_\_,add,discard|	InheritedSetmethods andclear,pop,remove,\_\_ior\_\_,\_\_iand\_\_,\_\_ixor\_\_, and\_\_isub\_\_
Mapping|	Collection|	\_\_getitem\_\_,\_\_iter\_\_,\_\_len\_\_|	\_\_contains\_\_,keys,items,values,get,\_\_eq\_\_, and\_\_ne\_\_
MutableMapping|	Mapping|	\_\_getitem\_\_,\_\_setitem\_\_,\_\_delitem\_\_,\_\_iter\_\_,\_\_len\_\_|	InheritedMappingmethods andpop,popitem,clear,update, andsetdefault
MappingView|	Sized|	|	\_\_len\_\_
ItemsView|	MappingView,Set|	|	\_\_contains\_\_,\_\_iter\_\_
KeysView|	MappingView,Set|	|	\_\_contains\_\_,\_\_iter\_\_
ValuesView|	MappingView,Collection|	|	\_\_contains\_\_,\_\_iter\_\_
Awaitable|	|	\_\_await\_\_|	
Coroutine|	Awaitable|	send,throw|	close
AsyncIterable|	|	\_\_aiter\_\_|	
AsyncIterator|	AsyncIterable|	\_\_anext\_\_|	\_\_aiter\_\_
AsyncGenerator|AsyncIterator|asend,athrow|aclose,\_\_aiter\_\_,\_\_anext\_\_

![image](https://github.com/ermaot/notes/blob/master/python/python%E5%8E%9F%E7%90%86%E4%B8%8E%E4%BC%98%E5%8C%96/pic/python%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84%E4%B9%A0%E6%83%AF%E7%94%A8%E6%B3%95-%E6%8E%A5%E5%8F%A3%EF%BC%9A%E4%BB%8E%E5%8D%8F%E8%AE%AE%E5%88%B0%E6%8A%BD%E8%B1%A1%E5%9F%BA%E7%B1%BB.png)
- Iterable、Container和Sized：各个集合应该继承这三个抽象基类，或者至少实现兼容的协议。Iterable 通过 \_\_iter\_\_ 方法支持迭代，Container 通过\_\_contains\_\_ 方法支持 in 运算符，Sized 通过 \_\_len\_\_ 方法支持len() 函数。
- Sequence、Mapping 和Set:　　这三个是主要的不可变集合类型，而且各自都有可变的子类。
- MappingView:　　在 Python 3 中，映射方法 .items()、.keys() 和 .values() 返回的对象分别是 ItemsView、KeysView和ValuesView的实例。前两个类还从Set类继承了丰富的接口
- Callable 和 Hashable:　　这两个抽象基类与集合没有太大的关系，只不过因为collections.abc 是标准库中定义抽象基类的第一个模块，而它们又太重要了，因此才把它们放到 collections.abc 模块中。

#### 抽象基类的数字塔
- 如果想检查一个数是不是整数，可以使用isinstance(x,numbers.Integral)，这样代码就能接受 int、bool（int的子类），或者外部库使用numbers抽象基类注册的其他类型。为了满足检查的需要，你或者你的 API的用户始终可以把兼容的类型注册为numbers.Integral 的虚拟子类。
- 如果一个值可能是浮点数类型，可以使用isinstance(x,numbers.Real)检查。这样代码就能接受bool、int、float、fractions.Fraction，或者外部库（如NumPy，它做了相应的注册）提供的非复数类型
## 定义并使用一个抽象基类

```
import abc
class Tombola(abc.ABC):  
##自己定义的抽象基类要继承 abc.ABC
    @abc.abstractmethod
    def load(self, iterable):  
        """从可迭代对象中添加元素。"""
## 抽象方法使用 @abstractmethod 装饰器标记，而且定义体中通常只有文档字符串
    @abc.abstractmethod
    def pick(self):  
     """随机删除元素，然后将其返回。
        如果实例为空，这个方法应该抛出`LookupError`。
        """
    def loaded(self):  
        """如果至少有一个元素，返回`True`，否则返回`False`。"""
        return bool(self.inspect())  
    def inspect(self):
        """返回一个有序元组，由当前元素构成。"""
        items = []
        while True:  
            try:
                items.append(self.pick())
            except LookupError:
                break
        self.load(items)  
        return tuple(sorted(items))
```

#### 抽象基类的句法详解
#### 定义Tombola抽象基类的子类
#### Tombola的虚拟子类
## Tombola子类的测试方法
## python使用的register方式
## 鹅的行为有可能像鸭子
