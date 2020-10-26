单例模式，是设计模式中常用的一种模式。该模式的主要目的是确保**某一个类只有一个实例存在**。比如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个 AppConfig 的类来读取配置文件的信息。如果在程序运行期间，有很多地方都需要使用配置文件的内容，也就是说，很多地方都需要创建 AppConfig 对象的实例，这就导致系统中存在多个 AppConfig 的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似 AppConfig 这样的类，我们希望在程序运行期间只存在一个实例对象。

## 使用模块

**Python 的模块就是天然的单例模式**，因为模块在第一次导入时，会生成 `.pyc` 文件，当第二次导入时，就会直接加载 `.pyc` 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。

```
class Singleton(object):
    def foo(self):
        pass
singleton = Singleton()
```

将上述代码保存在文件 `mysingleton.py` 中，要使用时，直接在其他文件中导入此文件中的对象，这个对象即是单例模式的对象

```
from **** import singleton
```

## 使用装饰器

```
In [38]: def Singleton(cls):
    ...:     _instance = {}
    ...:
    ...:     def _singleton(*args, **kargs):
    ...:         if cls not in _instance:
    ...:             _instance[cls] = cls(*args, **kargs)
    ...:         return _instance[cls]
    ...:
    ...:     return _singleton
    ...:
    ...:
    ...: @Singleton
    ...: class A(object):
    ...:     a = 1
    ...:
    ...:     def __init__(self, x=0):
    ...:         self.x = x
    ...:

In [39]: a1 = A(2)

In [40]: a2 = A(3)

In [41]: a1.x
Out[41]: 2

In [42]: a2.x
Out[42]: 2

In [43]: a2.a
Out[43]: 1

In [44]: a1.a
Out[44]: 1

In [50]: id(a1)
Out[50]: 356755004888

In [51]: id(a2)
Out[51]: 356755004888
```

可以看到虽然定义了两个对象a1和a2，实际上是同一个。



## 基于\_\_new\_\_方法实现

当我们实例化一个对象时，是**先执行了类的\_\_new\_\_方法**（我们没写时，默认调用object.\_\_new\_\_），**实例化对象**；然后**再执行类的\_\_init\_\_方法**，对这个对象进行初始化，所有我们可以基于这个，实现单例模式

```
In [53]: import threading

In [54]: class Singleton(object):
    ...:     _instance_lock = threading.Lock()
    ...:
    ...:     def __init__(self):
    ...:         pass
    ...:
    ...:
    ...:     def __new__(cls, *args, **kwargs):
    ...:         if not hasattr(Singleton, "_instance"):
    ...:             with Singleton._instance_lock:
    ...:                 if not hasattr(Singleton, "_instance"):
    ...:                     Singleton._instance = object.__new__(cls)
    ...:         return Singleton._instance
    ...:

In [55]: obj1 = Singleton()

In [56]: obj2 = Singleton()

In [57]: id(obj1)
Out[57]: 356759305240

In [58]: id(obj2)
Out[58]: 356759305240
```

这种方法加锁，保证了并发的正确性。但注意

1. 如果子类重写了**\_\_new\_\_方法**，有可能会覆盖或者干扰Singleton的**\_\_new\_\_方法**的执行。
2. 如果子类有\_\_init\_\_方法，每次实例化该Singleton的时候，\_\_init\_\_都会被调用。这显然是不应该的，\_\_init\_\_()应该只能在初始化的时候被调用一次。