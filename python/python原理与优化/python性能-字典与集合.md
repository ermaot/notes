## 字典和集合如何工作
- 字典和集合使用散列表来获得O(1)的查询和插入，因为使用了hash索引

#### 插入和获取
- 新插入数据的位置取决于键的散列值以及该值如何跟其他对象比较
- 插入数据时，需要计算==键的散列值并掩码==来得到一个有效的数组索引
#### 删除
当一个值从散列表中被删除时，不能写null到内存桶，因为我们已经用 NULL来作为嗅探散列碰撞的终止值。所以，必须写一个特殊的值来表示该桶虽空，但其后可能还有别的因散列碰撞而插入的值
#### 改变大小
- 研究显示一个不超过三分之二满的表在具有最佳空间节约的同时依然具有不错的散列碰撞避免率
- 如果需要改变大小，需要分配一个更大的表（也就是在内存中预留更多的桶），将掩码调整为适合新的表，旧表中的所有元素被重新插入新表。这需要重新计算索引。代价巨大。
- 字典或集合默认的最小长度是8（也就是说，即使你只保存 3个值，Python仍然会分配8个元素）。每次改变大小时，桶的个数增加到原来的4倍，直至达到50000个元素，之后每次增加到原来的 2 倍
#### 散列函数和熵
## 字典和命名空间
- Python在命名空间的管理上过度使用了字典来进行查询
- 每当 Python 访问一个变量、函数或模块时：
1. 首先，Python查找locals()数组，其内保存了所有本地变量的条目
2. 如果它不在本地变量里，那么会搜索globals()字典
3. 如果对象也不在那里， 则搜索__builtin__对象


```
In [1]: import math 

In [2]: from math import sin

In [3]: def test1(x):
   ...:     return math.sin(x)
   ...: 

In [4]: def test2(x):
   ...:     return sin(x)
   ...: 

In [5]: def test3(x,sin=math.sin):
   ...:     return sin(x)
   ...: 


In [7]: import dis


//查询 math 库来调用sin,一次查找math模块，一次在模块中查找sin 函数
In [8]: dis.dis(test1)
  2           0 LOAD_GLOBAL              0 (math)
              3 LOAD_ATTR                1 (sin)
              6 LOAD_FAST                0 (x)
              9 CALL_FUNCTION            1
             12 RETURN_VALUE        


//test2 从 math模块显式导入了sin函数，因此该函数可在全局命名空间中被直接访问,可以避免查询math模块以及后续的属性查询
In [9]: dis.dis(test2)
  2           0 LOAD_GLOBAL              0 (sin)
              3 LOAD_FAST                0 (x)
              6 CALL_FUNCTION            1
              9 RETURN_VALUE        

//test3 定义了sin函数为一个参数关键字，其默认值是 math 模块的 sin函数的引用,本地变量无须字典查询；它们被保存在一个十分微小的数组中，具有很快的查询速度
In [10]: dis.dis(test3)
  2           0 LOAD_FAST                1 (sin)
              3 LOAD_FAST                0 (x)
              6 CALL_FUNCTION            1
              9 RETURN_VALUE        
```

我们可以看到只需在循环前将 sin 函数本地化就能获
得 9.4%的速度提升
```
In [1]: from math import sin 

In [2]: def tight_loop_slow(iterations): 
   ...:         """ 
   ...:         >>> %timeit tight_loop_slow(10000000) 
   ...:         1 loops, best of 3: 2.21 s per loop 
   ...:         """ 
   ...:         result = 0 
   ...:         for i in xrange(iterations): 
   ...:                 # this call to sin requires a global lookup 
   ...:                 result += sin(i) 
   ...:          

In [3]: def tight_loop_fast(iterations): 
   ...:         """ 
   ...:         >>> %timeit tight_loop_fast(10000000) 
   ...:         1 loops, best of 3: 2.02 s per loop 
   ...:         """ 
   ...:         result = 0 
   ...:         local_sin = sin 
   ...:         for i in xrange(iterations): 
   ...:                 # this call to local_sin requires a local lookup 
   ...:                 result += local_sin(i)
   ...:         

In [4]: %timeit tight_loop_slow(10000000)
1 loops, best of 3: 1.43 s per loop

In [5]: %timeit tight_loop_fast(10000000) 
1 loops, best of 3: 1.4 s per loop
```
