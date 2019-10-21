## 内置数据类型
#### 存储类型
- 容器序列：　list、tuple和collections.deque这些序列能存放不同类型的数据
- 扁平序列：str、bytes、bytearray、memoryview 和 array.array，这类序列只能容纳一种类型
#### 是否可变
- 可变序列：　list、bytearray、array.array、collections.deque 和memoryview
- 不可变：　tuple、str 和 bytes

## 变量推导
python2：变量泄漏
```
In [1]: x = 'my precious'

In [2]: dummy = [x for x in 'ABC']

In [3]: x
Out[3]: 'C'
```
//ord()函数
```
In [4]:  dummy = [ord(x) for x in 'ABC']

In [5]: dummy
Out[5]: [65, 66, 67]
```

python3：无变量泄漏

```
In [1]:  x = 'my precious'

In [2]:  dummy = [x for x in 'ABC']

In [3]: x
Out[3]: 'my precious'
```
#### 列表推导同filter和map的比较
filter 和 map合起来能做的事情，列表推导也可以做，而且还不需要借助难以理解和阅读的 lambda 表达式
```
InIn [6]:  symbols = '$€¤'

In [7]:  beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]

In [8]:

In [8]: beyond_ascii
Out[8]: [8364, 164]

In [9]: beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))

In [10]: beyond_ascii
Out[10]: [8364, 164]

In [11]: beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]
```
#### 笛卡尔积（双重推导）

```
In [1]: colors = ['black', 'white']
   ...: sizes = ['S', 'M', 'L']
   ...: tshirts = [(color, size) for color in colors for size in sizes]

In [2]: tshirts
Out[2]:
[('black', 'S'),
 ('black', 'M'),
 ('black', 'L'),
 ('white', 'S'),
 ('white', 'M'),
 ('white', 'L')]
 
In [3]: tshirts = [[color, size] for color in colors for size in sizes]

In [4]: tshirts
Out[4]:
[['black', 'S'],
 ['black', 'M'],
 ['black', 'L'],
 ['white', 'S'],
 ['white', 'M'],
 ['white', 'L']]
```

## 生成器推导
 
```
In [1]: symbols = '$€¤'
//必须有tuple，而list推导不需要list
In [2]: tuple(ord(symbol) for symbol in symbols)
Out[2]: (36, 8364, 164)

In [3]: import array

In [4]: array.array('I', (ord(symbol) for symbol in symbols))
Out[4]: array('I', [36, 8364, 164])
```
生成器表达式逐个产出元素，而不是一次性生成全部元素

```
In [1]: a = [1,2,3]

In [2]: b = [4,5,6]

In [3]: for i in ((c,s) for c in a for s in b):
   ...:     print(i)
   ...:
(1, 4)
(1, 5)
(1, 6)
(2, 4)
(2, 5)
(2, 6)
(3, 4)
(3, 5)
(3, 6)

In [4]: for i in ('%d,%d'%(c,s) for c in a for s in b):
   ...:     print(i)
   ...:
1,4
1,5
1,6
2,4
2,5
2,6
3,4
3,5
3,6
```

## 元组与记录

#### 把元组用作记录；并定制排序方式
```
In [1]: lax_coordinates = (33.9425, -118.408056)
   ...: city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
   ...: traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ('ESP', 'XDA205856')]

In [2]: for passport in sorted(traveler_ids):
   ...:     print('%s/%s' % passport)
   ...:
BRA/CE342567
ESP/XDA205856
USA/31195855

In [3]: for passport in sorted(traveler_ids,key=lambda c :c[1]):
   ...:     print('%s/%s' % passport)
   ...:
USA/31195855
BRA/CE342567
ESP/XDA205856
```

#### 元组拆包

```
In [4]: lax_coordinates = (33.9425, -118.408056)

In [5]: latitude, longitude = lax_coordinates

In [6]: latitude
Out[6]: 33.9425

In [7]: longitude
Out[7]: -118.408056
```
#### * 的用法
- 以用 * 运算符把一个可迭代对象拆开作为函数的参数

```
In [8]: divmod(20, 8)
Out[8]: (2, 4)

In [9]: t = (20, 8)

In [10]: divmod(*t)
Out[10]: (2, 4)

In [11]: divmod((20,8))
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-11-705803c5bc7e> in <module>
----> 1 divmod((20,8))

TypeError: divmod expected 2 arguments, got 1
```
- 用*来处理剩下的元素
Python 中，函数用 *args 来获取不确定数量的参数
```
In [1]:  a, b, *rest = range(5)

In [2]: a,b,rest
Out[2]: (0, 1, [2, 3, 4])

In [3]:  a, b, *rest = range(3)

In [4]: a,b,rest
Out[4]: (0, 1, [2])

In [5]:  a, b, *rest = range(2)

In [6]: a,b,rest
Out[6]: (0, 1, [])
```

os.path.split()
```
In [12]: import os

In [13]:  os.path.split('/home/luciano/.ssh/idrsa.pub')
Out[13]: ('/home/luciano/.ssh', 'idrsa.pub')
```

os.path.dirname 和os.path.basename
```
In [17]: os.path.dirname("/home/luciano/.ssh/idrsa.pub")
Out[17]: '/home/luciano/.ssh'

In [18]: os.path.basename("/home/luciano/.ssh/idrsa.pub")
Out[18]: 'idrsa.pub'
```

==如果做的是国际化软件，那么_可能就不是一个理想的占位符，因为它也是gettext.gettext函数的常用别名==


#### 嵌套元组和列表拆包
嵌套元组拆包
```
In [1]: a = [(1,2,(3,4)),(5,6,(7,8))]

In [2]: for i,j,(k,l) in a:
   ...:     print(i,j,k,l)
   ...:
1 2 3 4
5 6 7 8
```
嵌套列表拆包

```
In [3]: b = [[1,2,[3,4]],[5,6,[7,8]]]

In [4]: for i,j,[k,l] in b:
   ...:     print(i,j,k,l)
   ...:
1 2 3 4
5 6 7 8
```

#### 命名元组

```
In [1]: from collections import namedtuple

In [2]: City = namedtuple('City', 'name country population coordinates')

In [3]: tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))

In [4]: tokyo
Out[4]: City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722, 139.691667))

In [5]: tokyo.name,tokyo.country,tokyo.population,tokyo.coordinates
Out[5]: ('Tokyo', 'JP', 36.933, (35.689722, 139.691667))

In [6]: City._fields
Out[6]: ('name', 'country', 'population', 'coordinates')
```

#### 列表或元组的方法和属性 比较
方法|列表|元组|说明 
---|---|---|---
s.\_\_add\_\_(s2)   | √ | √   |s + s2，拼接 
s.\_\_iadd\_\_(s2) |  √ |     |s += s2，就地拼接 
s.append(e)  |    √ |     |在尾部添加一个新元素 
s.clear()       | √  |   | 删除所有元素 
s.\_\_contains\_\_(e)| √| √  | s 是否包含 e 
s.copy()      |   √  |    |列表的浅复制 
s.count(e)|√ | √  | e 在 s  中出现的次数 
s.\_\_delitem\_\_(p) |√  |    |把位于 p  的元素删除 
s.extend(it)   |  √   |   |把可迭代对象 it 追加给 s 
s.\_\_getitem\_\_(p) |√|  √  | s[p]，获取位置 p  的元素 
s.\_\_getnewargs\_\_() |  |   √|  在 pickle  中支持更加优化的序列化 
s.index(e)        |  √  | √ |  在 s  中找到元素 e 第一次出现的位置 
s.insert(p, e)    |  √   |    |在位置 p 之前插入元素e 
s.\_\_iter\_\_()     |   √ |  √ |  获取 s  的迭代器 
s.\_\_len\_\_()       |  √  | √ |  len(s)，元素的数量 
s.\_\_mul\_\_(n)      |  √  | √ |  s * n，n 个 s  的重复拼接 
s.\_\_imul\_\_(n)     |  √    |  | s *= n，就地重复拼接 
s.\_\_rmul\_\_(n)      | √  | √ |  n * s，反向拼接  
s.pop([p])         | √    |    |删除最后或者是 （可选的）位于 p  的元素，并返回它 的值 
s.remove(e)     |   √|     |  删除 s  中的第一次出现的 e 
s.reverse()        | √     ||  就地把 s  的元素倒序排列 
s.\_\_reversed\_\_()  |  √ ||返回 s  的倒序迭代器 
s.\_\_setitem\_\_(p,   e)  |   √   ||    s[p] = e，把元素 e 放在位置p，替代已经在那个位置的元素 
s.sort([key],   [reverse]) |√   ||就地对 s  中的元素进行排序，可选的参数有键 （key） 和是否倒序 （reverse） 


## 切片
#### 为什么切片和区间会忽略最后一个元素
1. 这个习惯符合 Python、C 和其他语言里以 0 作为起始下标的传统
2. 当只有最后一个位置信息时，我们也可以快速看出切片和区间里有几个元素：range(3) 和 my_list[:3] 都返回 3 个元素
3. 当起止位置信息都可见时，我们可以快速计算出切片和区间的长度，用后一个数减去第一个下标（stop - start）即可
4. 这样做也让我们可以利用任意一个下标来把序列分割成不重叠的两部分，只要写成 my_list[:x] 和 my_list[x:] 就可以

#### 对对象切片
s[a:b:c] 的形式对 s 在 a 和 b之间以c为间隔取值

```
In [1]: s = 'bicycle'

In [2]: s[::3]
Out[2]: 'bye'

In [3]: s[::-1]
Out[3]: 'elcycib'

In [4]: s[::-2]
Out[4]: 'eccb'
```
seq[start:stop:step] 进行求值的时候，Python 会调用seq.__getitem__(slice(start, stop, step))


#### 多维切片

#### 给切片赋值

```
In [1]: l = list(range(10))

In [2]: l
Out[2]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [3]: l[2]=3

In [4]: l
Out[4]: [0, 1, 3, 3, 4, 5, 6, 7, 8, 9]

In [5]: l[2:4] = [1,2]

In [6]: l
Out[6]: [0, 1, 1, 2, 4, 5, 6, 7, 8, 9]

In [7]: del l[3:6]

In [8]: l
Out[8]: [0, 1, 1, 6, 7, 8, 9]
```
#### 对序列使用+和*

```
In [1]: s1 = [1,2,3]

In [2]: s2 = [4,5,6]

In [3]: s1 + s2
Out[3]: [1, 2, 3, 4, 5, 6]

In [4]: s1 *3
Out[4]: [1, 2, 3, 1, 2, 3, 1, 2, 3]

In [5]: s = 'abc'

In [6]: s*3
Out[6]: 'abcabcabc'
```
注意，如果类似于下面c，c的3个元素其实都是同一个的副本
```
In [1]: a = [1,2,3]

In [2]: b = a*3

In [3]: b
Out[3]: [1, 2, 3, 1, 2, 3, 1, 2, 3]

In [4]: c = [a]*3

In [5]: c
Out[5]: [[1, 2, 3], [1, 2, 3], [1, 2, 3]]

In [6]: c[0][0] = 4

In [7]: c
Out[7]: [[4, 2, 3], [4, 2, 3], [4, 2, 3]]

```

#### 序列的增量赋值
- += 背后的特殊方法是\_\_iadd\_\_（用于“就地加法”）。但是如果一个类没有实现这个方法的话，Python 会退一步调用 \_\_add\_\_ 
- 时对可变序列（例如 list、bytearray 和array.array）来说，\_\_iadd\_\_后值会就地改动
- *=   -=  /= 也类似

```
In [1]: l = [1,2,3]

In [2]: l
Out[2]: [1, 2, 3]

In [3]: id(l)
Out[3]: 300200495624

In [4]: l *= 2

In [5]: l
Out[5]: [1, 2, 3, 1, 2, 3]

In [6]: id(l)
Out[6]: 300200495624

In [7]: l = l*2

In [8]: l
Out[8]: [1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]

In [9]: id(l)
Out[9]: 300206987848
```
#### 元组不可变吗？
元组本身不可变，但元组里面的元素如果是列表或者字典等可变元素的集合，则集合内容可变

字典：
```
In [1]: a = ({'a':{'c':1}},{'b':2})

In [2]: a[0]['a']['c'] =2

In [3]: a
Out[3]: ({'a': {'c': 2}}, {'b': 2})
```
列表：注意列表吊诡的事情 ：
1. 元组里的列表里的元素可变
2. 元组里的列表不能直接用=赋值改变，同时抛出异常
3. 元组里的列表可以通过+=就地更新，同时抛出异常

```
In [1]: a = (1,2,[3,4])

In [2]: a
Out[2]: (1, 2, [3, 4])

In [3]: a[2][0] = 5

In [4]: a
Out[4]: (1, 2, [5, 4])

In [5]: a[2] = [6,7]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-5-07c7f94b1b91> in <module>
----> 1 a[2] = [6,7]

TypeError: 'tuple' object does not support item assignment

In [6]: a
Out[6]: (1, 2, [5, 4])

In [7]: a[2] += [6,7]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-7-f6bb232814a0> in <module>
----> 1 a[2] += [6,7]

TypeError: 'tuple' object does not support item assignment

In [8]: a
Out[8]: (1, 2, [5, 4, 6, 7])
```
查看字节码
```
import dis

dis.dis('a[2]+=[6]')
     0 LOAD_NAME                0 (a)
     2 LOAD_CONST               0 (2)
     4 DUP_TOP_TWO
     6 BINARY_SUBSCR                    //将值存入栈顶（TOS
     8 LOAD_CONST               1 (6)
    10 BUILD_LIST               1
    12 INPLACE_ADD                      //完成+=
    14 ROT_THREE
    16 STORE_SUBSCR
    18 LOAD_CONST               2 (None) //a[2]赋值失败
    20 RETURN_VALUE
```

#### 排序
list.sort 方法会就地排序列表，也就是说不会把原列表复制一份，所以这个方法的返回值是 None ；sorted会返回新列，原列表不变

```
In [1]: a = [3,5,2,6]

In [2]: id(a)
Out[2]: 9410995208

In [3]: sorted(a)
Out[3]: [2, 3, 5, 6]

In [4]: id(a)
Out[4]: 9410995208

In [5]: a
Out[5]: [3, 5, 2, 6]

In [6]: list.sort(a)

In [7]: a
Out[7]: [2, 3, 5, 6]

In [8]: id(a)
Out[8]: 9410995208

```

```
In [1]: a = ['a','test','ok']

In [2]: list.sort(a,reverse=True,key=len)

In [3]: a
Out[3]: ['test', 'ok', 'a']


```

```
In [1]: b = ['a','TEst','oK']

In [2]: list.sort(b,reverse=True,key=str.lower)

In [3]: b
Out[3]: ['TEst', 'oK', 'a']

In [4]: list.sort(b,reverse=True,key=lambda c:c.upper())

In [5]: b
Out[5]: ['TEst', 'oK', 'a']

In [6]: sorted(b,reverse=True,key=lambda c:c.upper())
Out[6]: ['TEst', 'oK', 'a']

In [7]: sorted(b,reverse=True,key=str.lower)
Out[7]: ['TEst', 'oK', 'a']

```

## array

```
In [1]: from array import array
   ...: from random import random
   ...: floats = array('d', (random() for i in range(10**7)))

In [2]: floats[-1]
Out[2]: 0.8314705043706755

In [3]: with open('floats.bin', 'wb') as f:
   ...:     floats.tofile(f)
   ...:

In [4]: floats2 = array('d')

In [6]: with open('floats.bin', 'rb') as f:
   ...:     floats2.fromfile(f,10**7)
   ...:

In [7]: floats2[-1]
Out[7]: 0.8314705043706755

In [8]: floats2 == floats
Out[8]: True
```
