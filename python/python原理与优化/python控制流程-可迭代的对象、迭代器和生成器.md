在 Python 中，所有集合都可以迭代。在Python语言内部，迭代器用于支持：
- for 循环
- 构建和扩展集合类型
- 逐行遍历文本文件
- 列表推导、字典推导和集合推导
- 元组拆包
- 调用函数时，使用 * 拆包实参
## sentence 类第一版：单词序列
将字符串拆分成单词列表
```
In [1]: import re
   ...: import reprlib
   ...: RE_WORD = re.compile('\w+')
   ...: class Sentence:
   ...:     def __init__(self, text):
   ...:         self.text = text
   ...:         self.words = RE_WORD.findall(text)
   ...:     def __getitem__(self, index):
   ...:         return self.words[index]
   ...:     def __len__(self):
   ...:         return len(self.words)
   ...:     def __repr__(self):
   ...:         return 'Sentence(%s)' % reprlib.repr(self.text)
   ...:

In [2]: s = Sentence("test the world")

In [3]: s[0]
Out[3]: 'test'

In [4]: len(s)
Out[4]: 3
```

#### 序列可迭代的原因：iter函数
内置的 iter 函数有以下作用：
1. 检查对象是否实现了\_\_iter\_\_方法，如果实现了就调用它，获取一个迭代器。
2. 如果没有实现\_\_iter\_\_方法，但是实现了\_\_getitem\_\_ 方法，Python会创建一个迭代器，尝试按顺序（从索引 0 开始）获取元素。
3. 如果尝试失败，Python抛出TypeError异常，通常会提示“C objectis not iterable”（C对象不可迭代），其中 C是目标对象所属的类
4. 从 Python3.4开始，检查对象x能否迭代，最准确的方法调用 iter(x) 函数，如果不可迭代，再处理 TypeError 异常。这比使用isinstance(x,abc.Iterable) 更准确，因为iter(x)函数会考虑到遗留的\_\_getitem\_\_ 方法，而 abc.Iterable 类则不考虑

## 可迭代的对象与迭代器的对比
- 使用 iter 内置函数可以获取迭代器的对象。
- 如果对象实现了能返回迭代器的 \_\_iter\_\_ 方法，那么对象就是可迭代的
- 序列都可以迭代
- 实现了 \_\_getitem\_\_ 方法，而且其参数是从零开始的索引，这种对象也可以迭代

```
In [6]: it = iter(s)

In [7]: it
Out[7]: <iterator at 0x19e2b384e0>

In [8]: next(it)
Out[8]: 'test'

In [9]: next(it)
Out[9]: 'the'

In [10]: next(it)
Out[10]: 'world'
```

可以不断使用next然后捕获异常的方法（next到最后元素会报StopIteration异常）

```
In [11]: s = 'ABC'
    ...: it = iter(s)
    ...: while True:
    ...:     try:
    ...:         print(next(it))
    ...:     except StopIteration:
    ...:         del it
    ...:         break
    ...:
A
B
C
```

- ==迭代器是这样的对象：实现了无参数的\_\_next\_\_方法，返回序列中的下一个元素；如果没有元素了，那么抛出 StopIteration异常。==
- Python中的迭代器还实现了\_\_iter\_\_方法，因此迭代器也可以迭代
## sentence类第二版：典型的迭代

```
class SentenceIterator:
    def __init__(self, words):
        self.words = words         
        self.index = 0  
    def __next__(self):
        try:
            word = self.words[self.index]  
        except IndexError:
            raise StopIteration() 
        self.index += 1  
        return word
    def __iter__(self):  
        return self
```

#### 把sentence变成迭代器：坏主意
- 可迭代的对象有个 \_\_iter\_\_ 方法，每次都实例化一个新的迭代器
- 而迭代器要实现 \_\_next\_\_方法，返回单个元素，此外还要实现\_\_iter\_\_方法，返回迭代器本身
## sentence类第三版：生成器函数

```
In [1]: import re
   ...: import reprlib
   ...: RE_WORD = re.compile('\w+')
   ...: class Sentence:
   ...:     def __init__(self, text):
   ...:         self.text = text
   ...:         self.words = RE_WORD.findall(text)
   ...:     def __repr__(self):
   ...:         return 'Sentence(%s)' % reprlib.repr(self.text)
   ...:     def __iter__(self):
   ...:         for word in self.words:
   ...:             yield word
   ...:         return
   ...:

In [2]: s = Sentence("test the world")

In [3]: for i in s:
   ...:     print(i)
   ...:
test
the
world

In [4]:

```

#### 生成器的工作原理

```
In [1]: def gen():
   ...:     yield 1
   ...:     yield 2
   ...:     yield 3
   ...:

In [2]: gen
Out[2]: <function __main__.gen()>

In [3]: gen()
Out[3]: <generator object gen at 0x0000008FCE1D2830>

In [4]: for i in gen():print(i)
1
2
3

In [5]: g = gen()

In [6]: next(g)
Out[6]: 1

In [7]: next(g)
Out[7]: 2

In [8]: next(g)
Out[8]: 3

In [9]: next(g)
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-9-e734f8aca5ac> in <module>
----> 1 next(g)

StopIteration:

```
-  只要 Python 函数中包含关键字 yield，该函数就是生成器函数
-  生成器是迭代器，会生成传给 yield 关键字的表达式的值
## sentence类第四版：惰性实现

```
In [1]: import re
   ...: import reprlib
   ...: RE_WORD = re.compile('\w+')
   ...: class Sentence:
   ...:     def __init__(self, text):
   ...:         self.text = text
   ...:     def __repr__(self):
   ...:         return 'Sentence(%s)' % reprlib.repr(self.text)
   ...:     def __iter__(self):
   ...:         for match in RE_WORD.finditer(self.text):
   ...:             yield match.group()
   ...:

In [2]: s = Sentence("test the world")

In [3]: for i in s:print(i)
test
the
world
```
re.finditer 函数是 re.findall函数的惰性版本，返回的不是列表，而是一个生成器，按需生成 re.MatchObject实例。如果有很多匹配，re.finditer 函数能节省大量内存
## sentence类第五版：生成器表达式

```
In [1]: import re
   ...: import reprlib
   ...: RE_WORD = re.compile('\w+')
   ...: class Sentence:
   ...:     def __init__(self, text):
   ...:       self.text = text
   ...:     def __repr__(self):
   ...:       return 'Sentence(%s)' % reprlib.repr(self.text)
   ...:     def __iter__(self):
   ...:       return (match.group() for match in RE_WORD.finditer(self.text))
   ...:

In [2]: s1 = Sentence("test the world")

In [3]: for i in s1:print(i)
test
the
world
```
生成器表达式可以理解为列表推导的惰性版本：不会迫切地构建列表，而是返回一个生成器，按需惰性生成元素
## 何时会用生成器表达式
如果生成器表达式要分成多行写，倾向于定义生成器函数，以便提高可读性。此外，生成器
函数有名称，因此可以重用
## 另一个示例：等差数列生成器

```
In [1]: class infinite:
   ...:     def __init__(self,start,step,end=None):
   ...:         self.begin,self.step,self.end = start,step,end
   ...:     def __iter__(self):
   ...:         result = self.begin
   ...:         forever = self.end is None
   ...:         index = 0
   ...:         while forever or result < self.end:
   ...:             yield result
   ...:             index += 1
   ...:             result = self.begin + self.step * index
   ...:

In [2]: s = infinite(1,1,20)

In [3]: for i in s:print(i)
```

#### 使用itertools模块生成等差数列
Python 3.4 中的 itertools模块提供了19个生成器函数，结合起来使用能实现很多有趣的用法

```
import itertools
gen = itertools.count(1, .5)


```



```
gen = itertools.takewhile(lambda n: n < 3, itertools.count(1, .5))
```

## 标准库中的生成器函数
模块| 函数 |说明
---|---|---
itertools|compress(it,selector_it)|并行处理两个可迭代的对象；如果 selector_it中的元素是真值，产出 it 中对应的元素
itertools|dropwhile(predicate,it)|处理 it，跳过 predicate 的计算结果为真值的元素，然后产出剩下的各个元素（不再进一步检查）
（内置）| filter(predicate, it)|把 it 中的各个元素传给 predicate，如果predicate(item) 返回真值，那么产出对应的元素；如果 predicate 是 None，那么只产出真值元素
itertools|filterfalse(predicate,it)|与 filter 函数的作用类似，不过 predicate 的逻辑是相反的：predicate 返回假值时产出对应的元素
itertools|islice(it, stop) 或islice(it, start,stop, step=1)|产出 it 的切片，作用类似于 s[:stop] 或s[start:stop:step]，不过 it 可以是任何可迭代的对象，而且这个函数实现的是惰性操作
itertools|takewhile(predicate,it)|predicate 返回真值时产出对应的元素，然后立即停止，不再继续检查



```
In [1]: def testiter(a):
   ...:     return  a.lower() in 'abcd'
   ...:

In [2]: import itertools

In [3]: list(filter(testiter,'oped'))
Out[3]: ['d']

In [4]: list(itertools.filterfalse(testiter,'oped'))
Out[4]: ['o', 'p', 'e']

//把seq的第一个元素传入func,如果func返回真,则返回删除第一个元素后其余元素的序列, 如果func返回假,则返回所有元素, 不删除
In [5]: list(itertools.dropwhile(testiter,'oped'))
Out[5]: ['o', 'p', 'e', 'd']

In [6]: list(itertools.takewhile(testiter,'oped'))
Out[6]: []

In [7]: list(itertools.islice('Aardvark', 1, 7, 2))
Out[7]: ['a', 'd', 'a']

```


模块| 函数 |说明
---|---|---
itertools|accumulate(it,[func])|产出累积的总和；如果提供了 func，那么把前两个元素传给它，然后把计算结果和下一个元素传给它，以此类推，最后产出结果
（内置） |enumerate(iterable,start=0)|产出由两个元素组成的元组，结构是 (index,item)，其中 index 从 start 开始计数，item 则从iterable 中获取
（内置） |map(func, it1,[it2, ..., itN])|把 it 中的各个元素传给func，产出结果；如果传入N 个可迭代的对象，那么 func 必须能接受 N 个参数，而且要并行处理各个可迭代的对象
itertools |starmap(func, it)|把 it 中的各个元素传给 func，产出结果；输入的可迭代对象应该产出可迭代的元素 iit，然后以func(*iit) 这种形式调用 func


```
In [1]: sample = [5, 4, 2, 8, 7, 6, 3, 0, 9, 1]

In [2]: import itertools,operator

In [3]: list(itertools.accumulate(sample))
Out[3]: [5, 9, 11, 19, 26, 32, 35, 35, 44, 45]

In [4]: list(itertools.accumulate(sample, min))
Out[4]: [5, 4, 2, 2, 2, 2, 2, 0, 0, 0]

In [5]: list(itertools.accumulate(sample, max))
Out[5]: [5, 5, 5, 8, 8, 8, 8, 8, 9, 9]

In [6]: list(itertools.accumulate(range(1, 11), operator.mul))
Out[6]: [1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]

In [7]: list(itertools.accumulate(sample, operator.mul))
Out[7]: [5, 20, 40, 320, 2240, 13440, 40320, 0, 0, 0]
```
#### 用于映射的生成器函数

```
In [1]: list(enumerate('albatroz', 1))
Out[1]:
[(1, 'a'),
 (2, 'l'),
 (3, 'b'),
 (4, 'a'),
 (5, 't'),
 (6, 'r'),
 (7, 'o'),
 (8, 'z')]

In [2]: import operator

In [3]: list(map(operator.mul, range(11), range(11)))
Out[3]: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

In [4]: list(map(operator.mul, range(11), [2, 4, 8]))
Out[4]: [0, 4, 16]

In [5]: list(map(lambda a, b: (a, b), range(11), [2, 4, 8]))
Out[5]: [(0, 2), (1, 4), (2, 8)]

In [6]: import itertools

In [7]: list(itertools.starmap(operator.mul, enumerate('albatroz', 1)))
Out[7]: ['a', 'll', 'bbb', 'aaaa', 'ttttt', 'rrrrrr', 'ooooooo', 'zzzzzzzz']

In [8]: sample = [5, 4, 2, 8, 7, 6, 3, 0, 9, 1]

In [9]: list(itertools.starmap(lambda a, b: b/a,enumerate(itertools.accumulate(sample), 1)))
Out[9]:
[5.0,
 4.5,
 3.6666666666666665,
 4.75,
 5.2,
 5.333333333333333,
 5.0,
 4.375,
 4.888888888888889,
 4.5]
```

## python3.3中新出现的句法：yield from
## 可迭代的归约函数
## 深入分析iter函数
## 案例分析：在数据库转换工具中使用生成器
## 把生成器当成协程
