## 对象表示形式
每门面向对象的语言至少都有一种获取对象的字符串表示形式的标准方式。Python提供了两种方式：
1. \_\_repr\_\_:repr():以便于开发者理解的方式返回对象的字符串表示形式
2. \_\_str\_\_:str():以便于用户理解的方式返回对象的字符串表示形式
\_\_bytes\_\_ 方法与 \_\_str\_\_ 方法类似：
1. bytes() 函数调用它获取对象的字节序列表示形式。
2. \_\_format\_\_ 方法会被内置的format()函数和str.format()方法调用，使用特殊的格式代码显示对象的字符串表示形式
3. \_\_repr\_\_、\_\_str\_\_ 和 \_\_format\_\_ 都必须返回 Unicode 字符串（str类型）。只有\_\_bytes\_\_ 方法应该返回字节序列（bytes 类型）

```
from array import array
import math
class Vector2d:
    typecode = 'd' 
    def __init__(self, x, y):
        self.x = float(x)   
        self.y = float(y)
    def __iter__(self):
        return (i for i in (self.x, self.y))  
    def __repr__(self):
        class_name = type(self).__name__
        return '{}({%r}, {%r})'.format(class_name, *self)  
    def __str__(self):
        return str(tuple(self))  
    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +bytes(array(self.typecode, self)))  
    def __eq__(self, other):
        return tuple(self) == tuple(other)  
    def __abs__(self):
        return math.hypot(self.x, self.y)  
    def __bool__(self):
        return bool(abs(self)) 
```

## 再谈向量类
## 备选构造方法
## classmethod 和 staticmethod

```
In [1]: class Demo:
   ...:     @classmethod
   ...:     def klassmeth(*args):
   ...:         return args
   ...:     @staticmethod
   ...:     def statmeth(*args):
   ...:         return args
   ...:

In [2]: Demo.klassmeth()
Out[2]: (__main__.Demo,)

In [3]: Demo.klassmeth('spam')
Out[3]: (__main__.Demo, 'spam')

In [4]: Demo.statmeth()
Out[4]: ()

In [5]: Demo.statmeth('spam')
Out[5]: ('spam',)
```

## 格式化显示
内置的 format() 函数和str.format()方法把各个类型的格式化方式委托给相应的.\_\_format\_\_(format_spec) 方法。format_spec 是格式说明符，它是：
1. format(my_obj, format_spec) 的第二个参数，或者
2. str.format() 方法的格式字符串，{} 里代换字段中冒号后面的部分

```
In [1]: brl = 1/2.43

In [2]: brl
Out[2]: 0.4115226337448559

In [3]: format(brl, '0.4f')
Out[3]: '0.4115'

In [4]: '1 BRL = {rate:0.2f} USD'.format(rate=brl)
Out[4]: '1 BRL = 0.41 USD'
```


```
In [1]: format(42, 'b')
Out[1]: '101010'

In [2]: format(2/3, '.1%')
Out[2]: '66.7%'
```
```
def __format__(self, fmt_spec=''):
    components = (format(c, fmt_spec) for c in self) 
    return '({}, {})'.format(*components)
```
## 可散列的Vector2d
1. 把 Vector2d 实例变成可散列的，必须使用 \_\_hash\_\_ 方法和\_\_eq\_\_ 方法
2. 且散列值不应该变化
## python的私有属性和受保护属性
## 使用__slots__类节省空间
在类中定义 \_\_slots\_\_ 属性的目的是告诉解释器：“这个类中的所有实
例属性都在这儿了！”这样，Python 会在各个实例中使用类似元组的结
构存储实例变量，从而避免使用消耗内存的 \_\_dict\_\_ 属性
- 如果使用得当，\_\_slots\_\_ 能显著节省内存，不过有几点要注
意。
1. 每个子类都要定义\_\_slots\_\_属性，因为解释器会忽略继承的\_\_slots\_\_ 属性。
2. 实例只能拥有\_\_slots\_\_中列出的属性，除非把 '\_\_dict\_\_'加入\_\_slots\_\_中（这样做就失去了节省内存的功效）
3. 如果不把 '\_\_weakref\_\_'加入__slots__，实例就不能作为弱引用的目标
## 覆盖类属性