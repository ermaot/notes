标准库里的所有映射类型都是利用dict来实现的，，因此它们有个共同的限制，即只有可散列的数据类型才能用作这些映射里的键

```
如果一个对象是可散列的，那么在这个对象的生命周期中，它的散列值是不变的，而且这个对象需要实现 __hash__() 方法。另外可散列对象还要有__qe__()方法，这样才能跟其他键做比较。如果两个可散列对象是相等的，那么它们的散列值一定是一样的
```

```
In [1]: tt = (1, 2, (30, 40))

In [2]: hash(tt)
Out[2]: 8027212646858338501

In [3]: tl = (1, 2, [30, 40])

In [4]: hash(tl)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-4-b7b49ff1c689> in <module>
----> 1 hash(tl)

TypeError: unhashable type: 'list'

In [5]: tf = (1, 2, frozenset([30, 40]))

In [6]: hash(tf)
Out[6]: 985328935373711578
```

字典不管顺序

```
In [1]: a = dict(one=1, two=2, three=3)
   ...: b = {'one': 1, 'two': 2, 'three': 3}
   ...: c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
   ...: d = dict([('two', 2), ('one', 1), ('three', 3)])
   ...: e = dict({'three': 3, 'one': 1, 'two': 2})

In [2]: a
Out[2]: {'one': 1, 'two': 2, 'three': 3}

In [3]: b
Out[3]: {'one': 1, 'two': 2, 'three': 3}

In [4]: c
Out[4]: {'one': 1, 'two': 2, 'three': 3}

In [5]: d
Out[5]: {'two': 2, 'one': 1, 'three': 3}

In [6]: e
Out[6]: {'three': 3, 'one': 1, 'two': 2}

In [7]: a == b == c == d == e
Out[7]: True

```

## 字典推导

```
In [1]: li = [(1,2),(3,4),(4,5),(5,6)]

In [2]: test = {a:b for a,b  in li}

In [3]: test
Out[3]: {1: 2, 3: 4, 4: 5, 5: 6}

```

## 常见的映射方法
dict、defaultdict 和 OrderedDict 的常见方法，后面两个数据类型是 dict 的变种，位于 collections 模块内
函数|dict	|defaultdict|	OrderedDict	|说明	
---|---|---|---|---
d.clear()	|√	|√	|√	|移除所有元素	
d.\_\_contains\_\_(k)|√|√	|√|	检查k 是否在 d  中 
d.copy()	|√  |√	|√	|浅复制 
d.\_\_copy\_\_()	||√	||用于支持 copy.copy 
d.default\_factory|	|√	||在 \_\_missing\_\_ 函数中被调用的 函数，用以给未找到的元素设置值* 
d.\_\_delitem\_\_(k)|√ |√	|√|	del d[k]，移除键为 k  的元素 
d.fromkeys(it,	[initial])	|√  |√	|√|	将迭代器 it 里的元素设置为映射里的键，如果有 initial 参数，就把它作为这些键对应的值 （默认是 None）	
d.get(k,  [default])	|√	|√	|√	|返回键 k 对应的值，如果字典里没有键 k，则返回 None 或者default	
d.\_\_getitem\_\_(k) |√	|√	|√	|让字典 d 能用 d[k]  的形式返回键 k 对应的值 
d.items()	|√	|√	|√	|返回 d 里所有的键值对 
d.\_\_iter\_\_()	|√	|√	|√|	获取键的迭代器 
d.keys()	|√	|√	|√	|获取所有的键	
d.\_\_len\_\_()	|√	|√	|√	|可以用 len(d)  的形式得到字典里键值对的数量	
d.\_\_missing\_\_(k)|	|√||	当 \_\_getitem\_\_ 找不到对应键的  时候，这个方法会被调用	
d.move\_to\_end(k, [last])	|||√	|把键为 k  的元素移动到最靠前或 者最靠后的位置 （last  的默认值是 True）	
d.pop(k, [defaul]|√	|√	|√	|返回键 k 所对应的值，然后移除这个键值对。如果没有这个键，返回 None 或者 defaul	
d.popitem()	|√	|√	|√|	随机返回一个键值对并从字典里 # 移除它 
d.\_\_reversed\_\_()	|||√	|返回倒序的键的迭代器	
d.setdefault(k, [default])	|√	|√	|√	|若字典里有键k，则把它对应的值 ， 设置为 default，然后返回这个 值；若无，则让 d[k] =default，然后返回 default 
d.\_\_setitem\_\_(k,	v)	|√  |√	|√	|实现 d[k] = v 操作，把 k 对应的值设为v 
d.update(m, [**kargs])	|√  |√	|√	|m 可以是映射或者键值对迭代器，用来更新 d 里对应的条目 
d.values()	|√  |√	|√	|返回字典里的所有值 

- default_factory 并不是一个方法，而是一个可调用对象（callable），它的值在defaultdict 初始化的时候由用户设定。
- OrderedDict.popitem()会移除字典里最先插入的元素（先进先出）；同时这个方法还有一
个可选的 last 参数，若为真，则会移除最后插入的元素（后进先出）

## enumerate

```
In [1]: seasons = ['Spring', 'Summer', 'Fall', 'Winter']

In [2]: list(enumerate(seasons))
Out[2]: [(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]

In [3]: list(enumerate(seasons, start=1))
Out[3]: [(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]

In [4]: list(enumerate(seasons, 1))
Out[4]: [(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]

In [5]: for i in enumerate(seasons, 1):
   ...:     print(i)
   ...:
(1, 'Spring')
(2, 'Summer')
(3, 'Fall')
(4, 'Winter')

In [6]: print(enumerate(seasons))
<enumerate object at 0x00000030DB487708>
```

## setdefault

```
In [68]: d = {}

In [69]: d.setdefault('a',[]).append(1)

In [70]: d
Out[70]: {'a': [1]}

In [71]: d.setdefault('a',[]).append(2)
    ...: d.setdefault('b',[]).append(4)

In [72]: d
Out[72]: {'a': [1, 2], 'b': [4]}
```


## 映射的弹性键查询
#### defaultdict
特殊方法 \_\_missing\_\_它会在defaultdict 遇到找不到的键的时候调用default_factory，创造某个值

```
In [1]: import collections

In [2]: d = collections.defaultdict(list)

In [3]: d
Out[3]: defaultdict(list, {})

In [4]: d['a'].append(1)

In [5]: d
Out[5]: defaultdict(list, {'a': [1]})

In [6]: d['a'].append(2)

In [7]: d
Out[7]: defaultdict(list, {'a': [1, 2]})

In [8]: d['a']
Out[8]: [1, 2]

//如果打印一个不存在的值，会自动赋值一个空值
In [9]: d[0]
Out[9]: []

In [10]: d
Out[10]: defaultdict(list, {'a': [1, 2], 0: []})

In [11]: d[1]
Out[11]: []

In [12]: d
Out[12]: defaultdict(list, {'a': [1, 2], 0: [], 1: []})
```

#### \_\_missing\_\_函数

```
定义StrKeyDict0类，继承自dict
class StrKeyDict0(dict):  
    def __missing__(self, key):
        if isinstance(key, str):            
            raise KeyError(key)
        return self[str(key)]  
    def get(self, key, default=None):
        try:
            return self[key]  
        except KeyError:
            return default  
    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()  
```

#### 字典的变种
- collections.OrderedDict：这个类型在添加键的时候会保持顺序;OrderedDict 的 popitem 方法默认删除并返回的是字典里的最后一个元素;但是如果像 my_odict.popitem(last=False) 这样调用它，那么它删除并返回第一个被添加进去的元素

```
In [1]: from collections import OrderedDict

In [2]: d = OrderedDict()

In [3]: d['a'] = 1

In [4]: d['b'] = 2

In [5]: d['c'] = 3

In [6]: d['d'] = 4

In [7]: d
Out[7]: OrderedDict([('a', 1), ('b', 2), ('c', 3), ('d', 4)])


In [9]: d.pop('a')
Out[9]: 1

In [10]: d
Out[10]: OrderedDict([('b', 2), ('c', 3), ('d', 4)])

In [11]: d.popitem()
Out[11]: ('d', 4)

In [12]: d
Out[12]: OrderedDict([('b', 2), ('c', 3)])

In [13]: d.popitem(last=False)
Out[13]: ('b', 2)

In [14]: d
Out[14]: OrderedDict([('c', 3)])

```
- collections.ChainMap：将多个映射合并为单个映射

```
In [1]: a = {'x': 1, 'z': 3}
   ...: b = {'y': 2, 'z': 4}

In [2]: import collections
   ...: c = collections.ChainMap(a, b)

In [3]: c
Out[3]: ChainMap({'x': 1, 'z': 3}, {'y': 2, 'z': 4})

//如果相同值，则使用第一个
In [4]: print(c['x'],c['y'],c['z'])
1 2 3

//如果原值改变，则映射值也会改变
In [4]: a['x'] = 2

In [5]: c
Out[5]: ChainMap({'x': 2, 'z': 3}, {'y': 2, 'z': 4})

//对映射值操作，会反映到原值上
In [6]: del c['x']

In [7]: a
Out[7]: {'z': 3}

//如果修改已经存在的字典值，则会导致原值也修改


In [12]: c['z']=4

In [13]: a
Out[13]: {'z': 4, 'y': 3}

//但要注意这里，如果第一个字典里没有的值复制，不是改变y的映射值，而是新增一个；同时修改了a的值
In [8]: c['y'] = 3

In [9]: b
Out[9]: {'y': 2, 'z': 4}

In [10]: c
Out[10]: ChainMap({'z': 3, 'y': 3}, {'y': 2, 'z': 4})

In [11]: a
Out[11]: {'z': 3, 'y': 3}
```
如果是使用a.update(b)这样的方法，也能得到字典合并的效果，但其实是复制了原来的字典而合并得到一个新的字典，并非类似于chainmap的映射


- collections.Counter：　这个映射类型会给键准备一个整数计数器

```
In [2]: import collections

In [3]: ct = collections.Counter('abracadabra')

In [4]: ct
Out[4]: Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

In [5]: ct.update('tdsadf')

In [6]: ct
Out[6]: Counter({'a': 6, 'b': 2, 'r': 2, 'c': 1, 'd': 3, 't': 1, 's': 1, 'f': 1})

In [7]: ct.most_common(2)
Out[7]: [('a', 6), ('d', 3)]

In [8]: ct + collections.Counter('abracadabra')
Out[8]: Counter({'a': 11, 'b': 4, 'r': 4, 'c': 2, 'd': 4, 't': 1, 's': 1, 'f': 1})

// 这里的减号，是删除所有的相同值，而不是上一次的值
In [9]: ct - collections.Counter('abracadabra')
Out[9]: Counter({'a': 1, 'd': 2, 't': 1, 's': 1, 'f': 1})
```
Counter 实现了 + 和 - 运算符用来合并记录，还有most_common([n]) 这类很有用的方法

####  不可变映射 MappingProxyType
MappingProxyType 只在 Python 3.3 后才有
```
In [1]: from types import MappingProxyType

In [2]: d = {1:'A'}

In [3]: d_proxy = MappingProxyType(d)

In [4]: d_proxy
Out[4]: mappingproxy({1: 'A'})

//通过映射修改是不允许的
In [5]: d_proxy[2] = 'x'
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-5-bc17a9a62754> in <module>
----> 1 d_proxy[2] = 'x'

TypeError: 'mappingproxy' object does not support item assignment

//可以通过原值修改
In [6]: d[2] = 'B'

In [7]: d_proxy[2]
Out[7]: 'B'
```

## 集合
集合的本质是许多唯一对象的聚集。因此，集合可以用于去重
```
In [1]: l = ['spam', 'spam', 'eggs', 'spam']

//集合去重
In [2]: set(l)
Out[2]: {'eggs', 'spam'}

In [3]: list(set(l))
Out[3]: ['eggs', 'spam']

```
求交集

```
found = len(set(needles) & set(haystack))
或者
found = len(set(needles).intersection(haystack))

In [1]: l = ['a','b','c','d']

In [2]: m = ['b','c','e']

In [3]: intersection = list(set(l) & set(m))

In [4]: intersection
Out[4]: ['b', 'c']

In [5]: intersection = list(set(l).intersection(set(m)))

In [6]: intersection
Out[6]: ['b', 'c']
```
求合集

```
In [8]: union = list(set(l) | set(m))

In [9]: union
Out[9]: ['c', 'b', 'e', 'a', 'd']

In [10]: union = list(set(l).union(set(m)))

In [11]: union
Out[11]: ['c', 'b', 'e', 'a', 'd']
```

#### 集合字面量

```
In [1]: s = {1}

In [2]: type(s)
Out[2]: set

In [3]: s
Out[3]: {1}

In [4]: s.pop()
Out[4]: 1

In [5]: s
Out[5]: set()

//{}不能用来赋值空set
In [6]: empty = {}

In [7]: empty
Out[7]: {}

In [8]: empty = set()

In [9]: empty
Out[9]: set()

```
1. 像 {1, 2, 3} 这种字面量句法相比于构造方法（set([1,2,3])）要更快且更易读。
2. 后者的速度要慢一些，因为Python必须先从set这个名字来查询构造方法，然后新建一个列表，最后再把这个列表传入到构造方法里。
3. 如果是像{1,2,3}这样的字面量，Python会利用一个专门的叫作BUILD_SET的字节码来创建集合
4. 

#### name

```
In [10]: from unicodedata import name

In [11]: {chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i),'')}
Out[11]:
{'#',
 '$',
 '%',
 '+',
 '<',
 '=',
 '>',
 '¢',
 '£',
 '¤',
 '¥',
 '§',
 '©',
 '¬',
 '®',
 '°',
 '±',
 'µ',
 '¶',
 '×',
 '÷'}
```

#### 集合操作
数学符号|Python运算符|方法 |描述
---|---|---|---
s∩z|s&z|s.\_\_and\_\_(z) |s 和 z 的交集
s∩z|z&s|s.\_\_rand\_\_(z) |反向 & 操作
s∩z|z&s|s.intersection(it, ...)|把可迭代的 it 和其他所有参数转化为集合，然后求它们与 s的交集
s∩z|s &=z|s.\_\_iand\_\_(z)|把 s 更新为 s 和 z 的交集
s∩z|s&=z|s.intersection\_update(it, ...)|把可迭代的 it 和其他所有参数转化为集合，然后求得它们与s 的交集，然后把 s 更新成这个交集
s∪z |s\|z |s.\_\_or\_\_(z)| s 和 z 的并集
s∪z |z \| s|s.\_\_ror\_\_(z)| \| 的反向操作
s∪z |z \| s | s.union(it, ...)|把可迭代的 it 和其他所有参数转化为集合，然后求它们和 s的并集
s∪z |s \|= z| s.\_\_ior\_\_(z) |把 s 更新为 s 和 z 的并集
s∪z |s \|= z|s.update(it, ...)|把可迭代的 it 和其他所有参数转化为集合，然后求它们和 s的并集，并把 s 更新成这个并集
S \ Z |s - z| s.\_\_sub\_\_(z)|s 和 z 的差集，或者叫作相对补集
S \ Z |z - s|s.\_\_rsub\_\_(z) |- 的反向操作
S \ Z |z - s|z.difference(it, ...)|把可迭代的 it 和其他所有参数转化为集合，然后求它们和 s的差集
S \ Z |s -= z|s.\_\_isub\_\_(z) |把 s 更新为它与 z 的差集
S \ Z |s -= z|s.difference\_update(it, ...)|把可迭代的 it 和其他所有参数转化为集合，求它们和 s 的差集，然后把 s 更新成这个差集
S △ Z |s ^ z|s.symmetric\_difference(it)|求 s 和 set(it) 的对称差集
S △ Z |s ^ z| s.\_\_xor\_\_(z) |求 s 和 z 的对称差集
S △ Z |z ^ s|s.\_\_rxor\_\_(z) |^ 的反向操作
S △ Z |z ^= z |s.symmetric\_difference\_update(it,...)|把可迭代的 it 和其他所有参数转化为集合，然后求它们和 s的对称差集，最后把 s 更新成该结果
S △ Z |s ^= z |s.\_\_ixor\_\_(z) |把 s 更新成它与 z 的对称差集

并集
```
In [1]: a = {1,2,3,4,5,6}

In [2]: b = {4,5,6,7,8}

In [3]: a & b
Out[3]: {4, 5, 6}

In [4]: a.__and__(b)
Out[4]: {4, 5, 6}

In [5]: b & a
Out[5]: {4, 5, 6}

In [6]: a.__rand__(b)
Out[6]: {4, 5, 6}

In [7]: a.intersection(b)
Out[7]: {4, 5, 6}

//就地更新
In [8]: a.__iand__(b)
Out[8]: {4, 5, 6}

In [9]: a
Out[9]: {4, 5, 6}

In [10]: a = {1,2,3,4,5,6}

In [11]: a.intersection_update(b)

In [12]: a
Out[12]: {4, 5, 6}
```

交集

```
In [1]: a = {1,2,3,4,5,6}

In [2]: b = {4,5,6,7,8}

In [3]: a | b
Out[3]: {1, 2, 3, 4, 5, 6, 7, 8}

In [4]: a.__or__(b)
Out[4]: {1, 2, 3, 4, 5, 6, 7, 8}

In [5]: a.__ror__(b)
Out[5]: {1, 2, 3, 4, 5, 6, 7, 8}

In [6]: a.union(b)
Out[6]: {1, 2, 3, 4, 5, 6, 7, 8}

In [7]: a.__ior__(b)
Out[7]: {1, 2, 3, 4, 5, 6, 7, 8}

In [8]: a
Out[8]: {1, 2, 3, 4, 5, 6, 7, 8}

In [9]: a = {1,2,3,4,5,6}

In [10]: a.update(b)

In [11]: a
Out[11]: {1, 2, 3, 4, 5, 6, 7, 8}

```

差集

```
In [1]: a = {1,2,3,4,5,6}

In [2]: b = {4,5,6,7,8}

In [3]: a - b
Out[3]: {1, 2, 3}

In [4]: a.__sub__(b)
Out[4]: {1, 2, 3}

In [5]: a.__rsub__(b)
Out[5]: {7, 8}

In [6]: a.difference(b)
Out[6]: {1, 2, 3}

In [7]: a -= b

In [8]: a
Out[8]: {1, 2, 3}

In [9]: a = {1,2,3,4,5,6}

In [10]: a.__isub__(b)
Out[10]: {1, 2, 3}

In [11]: a
Out[11]: {1, 2, 3}

In [12]: a = {1,2,3,4,5,6}

In [13]: a.difference_update(b)

In [14]: a
Out[14]: {1, 2, 3}
```

对称差集

```
In [1]: a = {1,2,3,4,5,6}

In [2]: b = {4,5,6,7,8}

In [3]: a.symmetric_difference(b)
Out[3]: {1, 2, 3, 7, 8}

In [4]: a ^ b
Out[4]: {1, 2, 3, 7, 8}

In [5]: b ^ a
Out[5]: {1, 2, 3, 7, 8}

In [6]: a.__xor__(b)
Out[6]: {1, 2, 3, 7, 8}

In [7]: a.__rxor__(b)
Out[7]: {1, 2, 3, 7, 8}

In [8]: a.symmetric_difference_update(b)

In [9]: a
Out[9]: {1, 2, 3, 7, 8}

In [10]: a = {1,2,3,4,5,6}

In [11]: a.__ixor__(b)
Out[11]: {1, 2, 3, 7, 8}

In [12]: a
Out[12]: {1, 2, 3, 7, 8}
```

#### 集合的比较运算符，返回值是布尔类型
数学符号|Python 运算符| 方法 |描述
---|---|---|---
 ''| ''|s.isdisjoint(z)|查看 s 和 z 是否不相交（没有共同元素）
e ∈ S| e in s |s.\_\_contains\_\_(e) |元素 e 是否属于 s
S ⊆ Z s <= z||s.\_\_le\_\_(z) |s 是否为 z 的子集
S ⊆ Z s <= z||s.issubset(it)|把可迭代的 it 转化为集合，然后查看s 是否为它的子集
S ⊂ Z |s < z |s.\_\_lt\_\_(z) |s 是否为 z 的真子集
S ⊇ Z s >= z||s.\_\_ge\_\_(z) |s 是否为 z 的父集
S ⊇ Z s >= z||s.issuperset(it)|把可迭代的 it 转化为集合，然后查看s 是否为它的父集
S ⊃ Z| s > z |s.\_\_gt\_\_(z) |s 是否为 z 的真父集


```
In [1]: a = {1,2,3,4,5,6}

In [2]: b = {4,5,6,7,8}

In [3]: a.isdisjoint(b)
Out[3]: False

In [4]: 1 in a
Out[4]: True

In [5]: a.__contains__(1)
Out[5]: True

In [6]: c = {1,2,3}

In [7]: a.__le__(c)
Out[7]: False


In [9]: c.__le__(a)
Out[9]: True

In [10]: c.issubset(a)
Out[10]: True

In [11]: a.__ge__(c)
Out[11]: True

In [12]: a.issuperset(c)
Out[12]: True

In [13]: a.__gt__(c)
Out[13]: True

In [14]: d = {1,2,3}

In [15]: d.__gt__(c)
Out[15]: False

```

#### 集合类型的其他方法
''|set| frozenset |''
---|---|---|---
s.add(e) |√ |  |把元素 e 添加到 s 中
s.clear()|√ ||  移除掉 s 中的所有元素
s.copy() |√|√ |对 s 浅复制
s.discard(e)|√ ||  如果 s 里有 e 这个元素的话，把它移除
s.\_\_iter\_\_()|√|√| 返回 s 的迭代器
s.\_\_len\_\_()|√ |√| len(s)
s.pop() |√  ||从 s 中移除一个元素并返回它的值，若 s 为空，则抛出 KeyError 异常
s.remove(e) |√  ||从 s 中移除 e 元素，若 e 元素不存在，则抛出KeyError 异常


```
In [1]: a = {1,2,3,4,5,6,7}

In [2]: b = frozenset([1,2,3,4])

In [3]: a.add(8)

In [4]: a
Out[4]: {1, 2, 3, 4, 5, 6, 7, 8}

In [5]: c = a.copy()

In [6]: c
Out[6]: {1, 2, 3, 4, 5, 6, 7, 8}

In [7]: a.discard(9)

In [8]: a
Out[8]: {1, 2, 3, 4, 5, 6, 7, 8}

In [9]: a.discard(7)

In [10]: a
Out[10]: {1, 2, 3, 4, 5, 6, 8}


In [12]: a.__len__()
Out[12]: 7

In [13]: len(a)
Out[13]: 7

In [14]: d = b.copy()

In [15]: d
Out[15]: frozenset({1, 2, 3, 4})

In [16]: len(d)
Out[16]: 4
```
