python代码的质量建议

## 1. 利用数据交换，不使用中间变量

```
x,y = y,x
```

语义清晰；性能好

## 2. 利用延迟计算

## 3. 使用isinstance来判断类型，而不是type

对一个类的实例来type，显示的是<type 'instance'>，所有的类的实例的类型都相等（仅限于python2版本，python3已经不是如此）

## 4. 转浮点类型再除法

## 5.使用enumerate获取序号和元素

```
a = [1,2,3]
for i ,e in enumerate(a):
    print(i,e)
```

## 6. 勿用eval

## 7. is 与 ==的区别

is判断二者id，而==判断二者的值是否相等

由于python有小对象的复用机制，所以两个不同变量可能有相同的id

## 8.使用with打开和关闭资源

## 9 .finally可能存在的问题

try中发生异常，如果except 中没有对应的异常处理，会临时保存，等待 finally处理完毕后再抛出；如果finally中产生新异常，或者执行了return、 break语句，那么临时保存的异常会被丢失

## 10.字符串优先使用join而不是+

join会事先计算全部字符串的空间，然后再一次性分配，复杂度o(n);而+则是复制一次分配一次，复杂度为o(n)

## 11.注意可变对象与不可变对象，深拷贝与浅拷贝

对象初始化的时候，[]占位的参数如果不写，最好使用None为默认

## 12.列表，元组，字典，集合都可以推导

```
a = [1,2,3]
{i:str(i+1) for i in a}
tuple(str(i+1) for i in a)
{str(i+1) for i in a}
```



## 13. cPickle已经集成到pandas里了

## 14. sys.setrecursionlimit设置递归深度，sys.getrecursionlimit()获取递归深度

## 15. 赋值语句默认为局部变量，如果要操作全局变量则需要加global



