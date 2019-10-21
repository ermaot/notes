## 把函数视作对象
Python 函数是对象

```
In [1]: def factorial(n):
   ...:     '''returns n!'''
   ...:     return 1 if n < 2 else n * factorial(n-1)
   ...:

In [2]: factorial.__doc__
Out[2]: 'returns n!'

In [3]: type(factorial)
Out[3]: function

In [4]: help(factorial)
Help on function factorial in module __main__:

factorial(n)
    returns n!


In [5]: fact = factorial

In [6]: fact(5)
Out[6]: 120

In [7]: fact(12)
Out[7]: 479001600

In [8]: fact
Out[8]: <function __main__.factorial(n)>

In [9]: map(factorial, range(11))
Out[9]: <map at 0x5401425d30>

In [10]: list(map(factorial, range(11)))
Out[10]: [1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]

```
map函数：Python 2.x 返回列表；Python 3.x 返回迭代器

## 高阶函数
根据单词长度给一个列表排序和根据反向拼写给一个单词列表排序

```
In [1]: a = ['ab','cde','fghij','kl']

//根据单词长度给一个列表排序
In [2]: sorted(a,key=len)
Out[2]: ['ab', 'kl', 'cde', 'fghij']

In [3]: def reverse(word):
   ...:     return word[::-1]
   ...:

//根据反向拼写给一个单词列表排序
In [4]: sorted(a,key=reverse)
Out[4]: ['ab', 'cde', 'fghij', 'kl']
```
在函数式编程范式中，最为人熟知的高阶函数有map、filter、reduce 和 apply（已经弃之不用）
#### map 、filter、reduce现代替代品
使用列表推导与map实现之间的区别
```
In [1]: def factorial(n):
   ...:     '''returns n!'''
   ...:     return 1 if n < 2 else n * factorial(n-1)
   ...:

In [2]: list(map(factorial, range(6)))
Out[2]: [1, 1, 2, 6, 24, 120]

In [3]: [factorial(n) for n in range(6)]
Out[3]: [1, 1, 2, 6, 24, 120]

In [4]: list(map(factorial, filter(lambda n: n % 2, range(6))))
Out[4]: [1, 6, 120]

In [5]: [factorial(n) for n in range(6) if n % 2]
Out[5]: [1, 6, 120]
```
- 在 Python 3 中，map 和 filter 返回生成器（一种迭代器）
- 在 Python 2 中，reduce 是内置函数，但是在 Python 3 中放到functools 模块里


```
In [1]: from functools import reduce

In [2]: from operator import add

In [3]: reduce(add, range(100))
Out[3]: 4950

In [4]: sum(range(100))
Out[4]: 4950

```
sum 和 reduce 的通用思想是把某个操作连续应用到序列的元素上，累计之前的结果，把一系列值归约成一个值

- all 和 any 也是内置的归约函数。
1. all(iterable)：　　如果 iterable 的每个元素都是真值，返回 True；all([]) 返回True。
2. any(iterable)：　　只要 iterable 中有元素是真值，就返回 True；any([]) 返回False

```
In [1]: if all([True  for i in range(10) if i%2 ==0]):
   ...:     print("True")
   ...:
True

In [2]: if any([True  for i in range(10) if i%2 ==0]):
   ...:     print("True")
   ...:
True
```

## 匿名函数
- lambda 关键字在 Python 表达式内创建匿名函数
- 除了作为参数传给高阶函数之外，Python很少使用匿名函数。由于句法上的限制，非平凡的 lambda 表达式要么难以阅读，要么无法写出

```
In [1]: fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']

In [2]: sorted(fruits, key=lambda word: word[::-1])
Out[2]: ['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']

```

## 可调用对象
## 用户定义的可调用类型
## 函数内省

```
In [1]: def factorial(n):
   ...:     '''returns n!'''
   ...:     return 1 if n < 2 else n * factorial(n-1)
   ...:

In [2]: dir(factorial)
Out[2]:
['__annotations__',
 '__call__',
 '__class__',
 '__closure__',
 '__code__',
 '__defaults__',
 '__delattr__',
 '__dict__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__get__',
 '__getattribute__',
 '__globals__',
 '__gt__',
 '__hash__',
 '__init__',
 '__init_subclass__',
 '__kwdefaults__',
 '__le__',
 '__lt__',
 '__module__',
 '__name__',
 '__ne__',
 '__new__',
 '__qualname__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__']

```
名称 |类型| 说明
---|---|---
\_\_annotations\_\_ dict 参数和返回值的注解
\_\_call\_\_|method-wrapper|实现 () 运算符；即可调用对象协议
\_\_closure\_\_ |tuple| 函数闭包，即自由变量的绑定（通常是 None）
\_\_code\_\_ |code |编译成字节码的函数元数据和函数定义体
\_\_defaults\_\_ |tuple |形式参数的默认值
\_\_get\_\_|method-wrapper|实现只读描述符协议




## 从定位参数到仅限关键字参数
仅限关键字参数是 Python3新增的特性。若想指定仅限关键字参数，要把它们放到前面有 * 的参数后面。如果不想支持数量不定的定位参数，但是想支持仅限关键字参数，在签名中放一个 *，如下所示

```
In [1]: def add(a,*,b):
   ...:     print(a,b)
   ...:

In [2]: add(1,b=1)
1 1

In [3]: add(1,1)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-3-2e8e00ead54c> in <module>
----> 1 add(1,1)

TypeError: add() takes 1 positional argument but 2 were given

```
## 获取关于参数的信息

```
In [1]: def add(a=1,*,b=12):
   ...:     print(a,b)
   ...:

In [2]: add.__defaults__
Out[2]: (1,)

In [3]: add()
1 12

In [4]: add(1)
1 12

In [5]: add(2)
2 12

In [8]: add.__code__.co_varnames
Out[8]: ('a', 'b')
```

## 函数注解

```
In [1]: def add(a:'int'=1,*,b:'int>0'=8)->None:
   ...:     print(a,b)
   ...:

In [2]: add.__annotations__
Out[2]: {'a': 'int', 'b': 'int>0', 'return': None}

In [3]: add.__defaults__
Out[3]: (1,)
```

## 支持函数式编程的包



#### operator模块
```
In [1]: from functools import reduce
   ...: def fact(n):
   ...:     return reduce(lambda a, b: a*b, range(1, n+1))
   ...:

In [2]: fact(10)
Out[2]: 3628800

In [3]: from operator import mul

In [4]: def fact2(n):
   ...:     return reduce(mul, range(1, n+1))
   ...:

In [5]: fact2(10)
Out[5]: 3628800
```

- itemgetter 和attrgetter

```
In [1]: from operator import itemgetter

In [2]: a = [('a','f','h'),('b','e','g'),('c','d','i')]

In [3]: for i in sorted(a,key=itemgetter(0)):
   ...:     print(i)
   ...:
('a', 'f', 'h')
('b', 'e', 'g')
('c', 'd', 'i')

In [4]: for i in sorted(a,key=itemgetter(1)):
   ...:     print(i)
   ...:
('c', 'd', 'i')
('b', 'e', 'g')
('a', 'f', 'h')

In [5]: for i in sorted(a,key=itemgetter(2)):
   ...:     print(i)
   ...:
('b', 'e', 'g')
('a', 'f', 'h')
('c', 'd', 'i')
```

- methodcaller
methodcaller 创建的函数会在对象上调用参数指定的方法
```
In [1]: from operator import methodcaller

In [2]: s = 'The time has come'

In [3]: upcase = methodcaller('upper')

In [4]: upcase(s)
Out[4]: 'THE TIME HAS COME'

In [5]: hiphenate = methodcaller('replace', ' ', '-')

In [6]: hiphenate(s)
Out[6]: 'The-time-has-come'
```

#### 使用functions.partial冻结参数
functools.partial 这个高阶函数用于部分应用一个函数。部分应用是指，基于一个函数创建一个新的可调用对象，把原函数的某些参数固定。

```
In [1]: from operator import mul

In [2]: from functools import partial

In [3]: triple = partial(mul, 3)

In [4]: list(map(triple, range(1, 10)))
Out[4]: [3, 6, 9, 12, 15, 18, 21, 24, 27]
```
