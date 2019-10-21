## 列表推导

```
>>> [i for i in range(10) if i%2 ==0]
[0, 2, 4, 6, 8]

```
知识点：1.列表推导;2.enumerate;
```
>>> test = ['one','two','three']
>>> list(enumerate(test))
[(0, 'one'), (1, 'two'), (2, 'three')]      
>>> ["%d:%s"%(i,enum) for i ,enum in enumerate(test)]                               
['0:one', '1:two', '2:three']

```

## 生成器和迭代器

```
>>> i = iter("abc")
>>> i.next()
'a'
>>> i.next()
'b'
>>> i.next()
'c'
>>> i.next()
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-10-e590fe0d22f8> in <module>()
----> 1 i.next()

StopIteration: 

```
### yield可以暂停一个函数并返回中间结果

```
In [9]: def fab():
   ...:     a,b = 1,2
   ...:     while True:
   ...:         yield b
   ...:         a,b = b,a+b
   ...:         

In [10]: f = fab()

In [11]: f.next()
Out[11]: 2

In [12]: f.next()
Out[12]: 3

In [13]: f.next()
Out[13]: 5

```
### power函数返回的是==generator==
#### - yield用法
```
In [18]: def power(values):
    ...:     for value in values:
    ...:         print 'powering %s' % value
    ...:         yield value
    ...:         

In [19]: def adder(values):
    ...:     for value in values:
    ...:         print 'adding to %s' % value
    ...:         if value%2 == 0:
    ...:             yield value + 3
    ...:         else:
    ...:             yield value + 2
    ...:             
In [20]: elements = [1,4,7,9,12,19]

In [21]: res = adder(power(elements))

In [22]: res.next()
powering 1
adding to 1
Out[22]: 3

In [23]: res.next()
powering 4
adding to 4
Out[23]: 7

In [24]: res.next()
powering 7
adding to 7
Out[24]: 9

In [31]: type(power(elements))
Out[31]: generator

```
#### - send用法
send会添加throw 和close两个函数
```
In [2]: def psy():
   ...:     print("please tell me your problems")
   ...:     while True:
   ...:         answer = (yield)
   ...:         if answer is not None:
   ...:             if answer.endswith('?'):
   ...:                 print("Dont't ask too much questions")
   ...:             elif 'good' in answer:
   ...:                 print("A that's good ,go on")
   ...:             elif 'bad' in answer:
   ...:                 print("Don't be so negative")
   ...:                 

In [3]: free = psy()

In [4]: free.next()
please tell me your problems

In [5]: free.send('I feel so bad')
Don't be so negative

In [6]: free.send("Why I shouldn't")

In [7]: free.send("Why I shouldn't?")
Dont't ask too much questions

```
#### - 协程

```
In [16]: def coroutine1():
    ...:     for i in range(3):
    ...:         print("c1")
    ...:         yield i
    ...:         

In [17]: def coroutine2():
    ...:     for i in range(3):
    ...:         print("c2")
    ...:         yield i
In [19]: multitask.add(coroutine1())

In [20]: multitask.add(coroutine2())

In [21]: multitask.run()
c1
c2
c1
c2
c1
c2

```
#### - 生成器表达式
1.和常规生成器一样，每次输出一个元素<br>
2.和列表推导一样，不会事先进行计算<br>
3.比普通写法减少代码总量<br>
4.比列表推导减少内存占用
```
In [22]: iter = (x**2 for x in range(10) if x%2 ==0)

In [23]: for el in iter:
    ...:     print el
    ...:     
0
4
16
36
64

```
#### - itertools模块
1. 获取第五个元素
1. 少于五个则结束
2. 第二个和第三个参数代表起终条件
```
In [35]: def stating_at_five():
    ...:     value = raw_input().strip()
    ...:     while value != '':
    ...:         for el in itertools.islice(value.split(),4,None):
    ...:             yield el
    ...:         value = raw_input().strip()
    ...:         

In [36]: iter = stating_at_five()

In [37]: iter.next()
one two three four five six
Out[37]: 'five'

In [38]: iter.next()
Out[38]: 'six'

In [39]: iter.next()
one two
one two three four five six
Out[39]: 'five'

In [40]: iter.next()
Out[40]: 'six'

```
#### - tee往返式迭代器

#### - groupby -- uniq迭代器

```
In [1]: from itertools import groupby
In [2]: def compress(data):
   ...:     return ((len(list(group)),name) for name,group in groupby(data))
   ...: 
In [3]: def decompress(data):
   ...:     return (car * size for size ,car in data)
   ...: 
In [6]: list(compress('get uuuuuuuuuuuuuuuuup'))
Out[6]: [(1, 'g'), (1, 'e'), (1, 't'), (1, ' '), (17, 'u'), (1, 'p')]
In [12]: compressed = compress('get uuuuuuuuuuuuuuuuup')

In [14]: ''.join(decompress(compressed))
Out[14]: 'get uuuuuuuuuuuuuuuuup'

```

#### - chains(*iterables)

```
In [28]: test = chain(iter('AB'), iter('CDE'), iter('F'))

In [29]: for el in test:
    ...:     print el
    ...:     
A
B
C
D
E
F

```

#### - count([n])
创建一个迭代器，生成从n开始的连续整数,n默认为0
```
In [33]: cn = count(3)

In [34]: print cn.next()
3
```

#### - cycle(iterable)

```
In [48]: test = iter("abc")

In [49]: for i in cycle(test):
    ...:     print i
a
In [51]: for i in cycle(test):
    ...:     print i
    ...:     time.sleep(2)
    ...:     
b
c
b
c

```

#### - dropwhile(predicate,iterable)
#### - ifilter(predicate,iterable)
#### - ifilterfalse(predicate,iterable)
#### - imap(function,*iterables)
#### - izip(*iterables)
#### - repeat(object[,times])
#### - starmap(function,iterable)
#### - takewhile(predicate,iterable)


### 装饰器

```
In [52]: class WhatFor(object):
    ...:     @classmethod
    ...:     def it(cls):
    ...:         print("work with %s"%cls)
    ...:     @staticmethod
    ...:     def uncommon():
    ...:         print("I could be a global function")
    ...:         

In [53]: this_is = WhatFor()

In [54]: this_is.it()
work with <class '__main__.WhatFor'>

In [55]: this_is.uncommon()
I could be a global function

```
