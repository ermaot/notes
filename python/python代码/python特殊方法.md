## 类的基础方法
目的|	所编写代码	|Python 实际调用
---|---|---
初始化一个实例|	x = MyClass()	|x.\_\_init\_\_()
字符串的“官方”表现形式	|repr(x)|	x.\_\_repr\_\_()
字符串的“非正式”值	|str(x)	|x.\_\_str\_\_()
字节数组的“非正式”值|	bytes(x)|	x.\_\_bytes\_\_()
格式化字符串的值	|format(x, format\_spec)|	x.\_\_format\_\_(format\_spec)

1. 对 \_\_init\_\_() 方法的调用发生在实例被创建 之后 。如果要控制实际创建进程，请使用 \_\_new\_\_() 方法。删除用\_\_del\_\_()方法
2. 按照约定， \_\_repr\_\_()方法所返回的字符串为合法的 Python 表达式。
3. 在调用 print(x) 的同时也调用了 \_\_str\_\_()方法。
4. 由于 bytes 类型的引入而从 Python 3 开始出现


## 行为方式与迭代器类似的类
目的|	所编写代码	|Python 实际调用
---|---|---
遍历某个序列|	iter(seq)|	seq.\_\_iter\_\_()
从迭代器中获取下一个值	|next(seq)|	seq.\_\_next\_\_()
按逆序创建一个迭代器|	reversed(seq)|	seq.\_\_reversed\_\_()

1. 无论何时创建迭代器都将调用\_\_iter\_\_()方法。这是用初始值对迭代器进行初始化的绝佳之处。
2. 无论何时从迭代器中获取下一个值都将调用 \_\_next\_\_()方法。
3. \_\_reversed\_\_()方法并不常用。它以一个现有序列为参数，并将该序列中所有元素从尾到头以逆序排列生成一个新的迭代器


## 计算属性
序号	|目的|	所编写代码	|Python 实际调用
---|---|---|---
①	|获取一个计算属性（无条件的）|	x.my\_property|	x.\_\_getattribute\_\_('my\_property')
②	|获取一个计算属性（后备）|	x.my\_property|	x.\_\_getattr\_\_('my\_property')
③|	设置某属性|	x.my\_property = value|	x.\_\_setattr\_\_('my\_property',value)
④|	删除某属性|	del x.my\_property	|x.\_\_delattr\_\_('my\_property')
⑤	|列出所有属性和方法|	dir(x)	|x.\_\_dir\_\_()

1. 如果某个类定义了 \_\_getattribute\_\_() 方法，在 每次引用属性或方法名称时Python 都调用它（特殊方法名称除外，因为那样将会导致讨厌的无限循环）。
2. 如果某个类定义了 \_\_getattr\_\_() 方法，Python 将只在正常的位置查询属性时才会调用它。如果实例 x 定义了属性color，x.color 将 不会 调用x.\_\_getattr\_\_('color')；而只会返回x.color已定义好的值。
3. 无论何时给属性赋值，都会调用 \_\_setattr\_\_()方法。
4. 无论何时删除一个属性，都将调用 \_\_delattr\_\_()方法。
5. 如果定义了 \_\_getattr\_\_() 或 \_\_getattribute\_\_() 方法， \_\_dir\_\_() 方法将非常有用。通常，调用 dir(x) 将只显示正常的属性和方法。如果 \_\_getattr()\_\_方法动态处理color 属性，dir(x) 将不会将 color 列为可用属性。可通过覆盖 \_\_dir\_\_() 方法允许将color 列为可用属性，对于想使用你的类但却不想深入其内部的人来说，该方法非常有益。

## 行为方式与函数类似的类
可以让类的实例变得可调用——就像函数可以调用一样——通过定义 \_\_call\_\_() 方法。

序号|目的|	所编写代码	|Python 实际调用
---|---|---|---
①|	像调用函数一样“调用”一个实例|	my\_instance()|	my\_instance.\_\_call\_\_()

## 行为方式与序列类似的类
序号|	目的|	所编写代码	Python |实际调用
---|---|---|---
①|	序列的长度	|len(seq)	|seq.\_\_len\_\_()
②	|了解某序列是否包含特定的值|	x in seq|	seq.\_\_contains\_\_(x)

## 行为方式与字典类似的类

序号|	目的	|所编写代码|	Python 实际调用
---|---|---|---
①	|通过键来获取值	|x[key]	|x.\_\_getitem\_\_(key)
②|	通过键来设置值|	x[key] = value	|x.\_\_setitem\_\_(key, value)
③	|删除一个键值对|	del x[key]	|x.\_\_delitem\_\_(key)
④	|为缺失键提供默认值|	x[nonexistent\_key]	|x.\_\_missing\_\_(nonexistent\_key)

## 可比较的类
序号|	目的	|所编写代码|	Python 实际调用
---|---|---|---
①	|相等	|x == y	|x.\_\_eq\_\_(y)
②	|不相等|	x != y	|x.\_\_ne\_\_(y)
③|	小于|	x < y	|x.\_\_lt\_\_(y)
④|	小于或等于|	x <= y	|x.\_\_le\_\_(y)
⑤	|大于|	x > y	|x.\_\_gt\_\_(y)
⑥|	大于或等于|	x >= y|	x.\_\_ge\_\_(y)
⑦|	布尔上上下文环境中的真值|	if x:	|x.\_\_bool\_\_()

## 可序列化的类
序号|	目的	|所编写代码|	Python 实际调用
---|---|---|---
①	|自定义对象的复制|	copy.copy(x)|	x.\_\_copy\_\_()
②|	自定义对象的深度复制|	copy.deepcopy(x)|	x.\_\_deepcopy\_\_()
③	|在 pickling 之前获取对象的状态	|pickle.dump(x, file)	|x.\_\_getstate\_\_()
④	|序列化某对象	|pickle.dump(x, file)	|x.\_\_reduce\_\_()
⑤	|序列化某对象（新 pickling 协议）|	pickle.dump(x, file, protocol\_version)	|x.\_\_reduce\_ex\_\_(protocol\_version)
⑥|	控制 unpickling 过程中对象的创建方式|	x = pickle.load(file)|	x.\_\_getnewargs\_\_()
⑦|	在 unpickling 之后还原对象的状态|	x = pickle.load(file)|	x.\_\_setstate\_\_()


## 可在 with 语块中使用的类
with 语块定义了 运行时刻上下文环境；在执行with语句时将“进入”该上下文环境，而执行该语块中的最后一条语句将“退出”该上下文环境。



序号|	目的	|所编写代码|	Python 实际调用
---|---|---|---
①|	在进入 with 语块时进行一些特别操作|	with x:	|x.\_\_enter\_\_()
②|	在退出 with 语块时进行一些特别操作|	with x:	|x.\_\_exit\_\_()


## 特殊方法
序号|	目的	|所编写代码|	Python 实际调用
---|---|---|---
①	|类构造器	|x = MyClass()	|x.__new__()
②	|类析构器|	del x	|x.__del__()
③	|只定义特定集合的某些属性|	 	|x.__slots__()
④	|自定义散列值|	hash(x)	|x.__hash__()
⑤	|获取某个属性的值	|x.color	|type(x).__dict__['color'].__get__(x, type(x))
⑥	|设置某个属性的值|	x.color = 'PapayaWhip'	|type(x).__dict__['color'].__set__(x, 'PapayaWhip')
⑦	|删除某个属性	del |x.color|	type(x).__dict__['color'].__del__(x)
⑧	|控制某个对象是否是该对象的实例|your class	|isinstance(x, MyClass)	MyClass.__instancecheck__(x)
⑨	|控制某个类是否是该类的子类	|issubclass(C, MyClass)|	MyClass.__subclasscheck__(C)
⑩	|控制某个类是否是该抽象基类的子类|	issubclass(C, MyABC)	|MyABC.__subclasshook__(C)


## 跟运算符相关的特殊方法
类别|方法名|运算符
---|---|---
一元运算符|__neg__ | -
一元运算符|__pos__ |+
一元运算符|__abs__ |abs()