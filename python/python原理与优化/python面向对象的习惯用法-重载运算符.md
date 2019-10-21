## 运算符重载基础
- 运算符重载如果使用得当，API会变得好用，代码会变得易于阅读。
- Python 施加了一些限制，做好了灵活性、可用性和安全性方面的平衡：
1. 不能重载内置类型的运算符
2. 不能新建运算符，只能重载现有的
3. 某些运算符不能重载：is、and、or和not（不过位运算符&、| 和 ~ 可以）

 ==，这个运算符由\_\_eq\_\_ 方法支持
## 一元运算符

```
def __abs__(self):
    return math.sqrt(sum(x * x for x in self))
def __neg__(self):
    return Vector(-x for x in self)  
def __pos__(self):
    return Vector(self) 
```

#### x 与 +x在何时不等
- Decimal运算，会根据上下文精度调整

```
In [1]: import decimal

In [2]: ctx = decimal.getcontext()

In [3]: ctx.prec = 30

In [4]: one_third = decimal.Decimal('1') / decimal.Decimal('3')

In [5]: one_third
Out[5]: Decimal('0.333333333333333333333333333333')

In [6]: one_third == +one_third
Out[6]: True

//此时28是Decimal的默认精度
In [7]: ctx.prec = 28

In [8]: one_third == +one_third
Out[8]: False

In [9]: +one_third
Out[9]: Decimal('0.3333333333333333333333333333')
```
Decimal会根据上下文的设置选择特定精度，使用了+就对原来的值做了精度调整
-  collections.Counter

```
In [1]: from collections  import Counter

In [2]: ct = Counter('abracadabra')

In [3]: ct
Out[3]: Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

In [4]: ct['r'] = -3

In [5]: ct['d'] = 0

In [6]: ct
Out[6]: Counter({'a': 5, 'b': 2, 'r': -3, 'c': 1, 'd': 0})

In [7]: +ct
Out[7]: Counter({'a': 5, 'b': 2, 'c': 1})
```
对Counter对象执行+运算，会去掉非正值
## 重载向量加法运算符
## 重载标量乘法运算符
## 众多比较运算符
正向方法返回NotImplemented的话，调用反向方法
分组 |中缀运算符 |正向方法调用| 反向方法调用 |后备机制
---|---|---|---|---
相等性 |a == b |a.\_\_eq\_\_(b)| b.\_\_eq\_\_(a) |返回 id(a) == id(b)
 相等性| a != b |a.\_\_ne\_\_(b) |b.\_\_ne\_\_(a) |返回 not (a == b)
排序 |a > b |a.\_\_gt\_\_(b)| b.\_\_lt\_\_(a) |抛出 TypeError
 排序 |a < b |a.\_\_lt\_\_(b) |b.\_\_gt\_\_(a) |抛出 TypeError
 排序 |a >= b| a.\_\_ge\_\_(b) |b.\_\_le\_\_(a) |抛出 TypeError
 排序 |a <= b |a.\_\_le\_\_(b) |b.\_\_ge\_\_(a)| 抛出T ypeError
## 增量赋值运算符