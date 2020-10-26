可以说，本质上，编程的一多半时间都是在处理字符串

下面列举一些字符串用法，总有你不知道的点

## 多行字符串

```
In [1]: a = ('select * '
     ...: 'from'
     ...: 'test where a="1"')

In [2]: a
Out[2]: 'select * fromtest where a="1"'
```

这比'''或者"""有一个好处，就是不会产生多余的换行

```
In [3]: a = '''select * '
     ...: 'from'
     ...: 'test where a="1"'''

In [4]: a
Out[4]: 'select * \'\n\'from\'\n\'test where a="1"'
```

## 判断是否是某个类型的变量

序号|方法名|说明
---|---|---
1|isalnum()|    
2|isdigit()|   
3|isnumeric()|   
4|istitle()|
5|isalpha()|   
6|isidentifier()|
7|isprintable()| 
8|isupper()|
9|isdecimal()|  
10|islower()|     
11|isspace()|

关于[isdigit、isdecimal、isnumeric的区别](https://www.cnblogs.com/jebeljebel/p/4006433.html)

```
In [1]: num1 = '1'

In [2]: num2 = "Ⅷ"

In [3]: num3 = "四"

In [4]: num1.isdigit(),num1.isdecimal(),num1.isnumeric()
Out[4]: (True, True, True)

In [5]: num2.isdigit(),num2.isdecimal(),num2.isnumeric()
Out[5]: (False, False, True)

In [6]: num3.isdigit(),num3.isdecimal(),num3.isnumeric()
Out[6]: (False, False, True)

In [21]: num3 = "四十"

In [22]: num3.isnumeric()
Out[22]: True
```

这样看，isdigit()和isdecimal()基本没有什么不同。但在下面的情况就不一样了

```
In [15]: num5 = b'1'

In [16]: num5.isdigit()
Out[16]: True

In [17]: num5.isdecimal()
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-17-5e9bb1f3ffa8> in <module>
----> 1 num5.isdecimal()

AttributeError: 'bytes' object has no attribute 'isdecimal'

In [18]: num5.isnumeric()
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-18-506873feb538> in <module>
----> 1 num5.isnumeric()

AttributeError: 'bytes' object has no attribute 'isnumeric'
```
UCD是Unicode字符数据库（Unicode Character DataBase）的缩写。
UCD由一些描述Unicode字符属性和内部关系的纯文本或html文件组成。
UCD中的文本文件大都是适合于程序分析的Unicode相关数据。其中的html文件解释了数据库的组织，数据的格式和含义。
```
In [1]: import unicodedata

In [2]: unicodedata.digit("2"),unicodedata.decimal("2") ,unicodedata.numeric("2")
Out[2]: (2, 2, 2.0)

In [3]: unicodedata.digit("2"),unicodedata.decimal("2") ,unicodedata.numeric("2")
Out[3]: (2, 2, 2.0)

In [4]: unicodedata.digit(b"3")
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-4-780162450bdd> in <module>
----> 1 unicodedata.digit(b"3")

TypeError: digit() argument 1 must be a unicode character, not bytes

In [5]: unicodedata.decimal(b"3")
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-5-6ed97b9b8695> in <module>
----> 1 unicodedata.decimal(b"3")

TypeError: decimal() argument 1 must be a unicode character, not bytes

In [6]: unicodedata.numeric(b"3")
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-6-6c9e228b2392> in <module>
----> 1 unicodedata.numeric(b"3")

TypeError: numeric() argument 1 must be a unicode character, not bytes

IIn [7]: unicodedata.digit("Ⅷ")
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-7-653408b4621c> in <module>
----> 1 unicodedata.digit("Ⅷ")

ValueError: not a digit

IIn [8]: unicodedata.decimal("Ⅷ")
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-8-870383e0dfad> in <module>
----> 1 unicodedata.decimal("Ⅷ")

ValueError: not a decimal

IIn [9]: unicodedata.numeric("Ⅷ")
Out[9]: 8.0

In [10]: unicodedata.digit("四")
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-10-3aec9f8a19f2> in <module>
----> 1 unicodedata.digit("四")

ValueError: not a digit

In [11]: unicodedata.decimal("四")
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-11-0d17bd624af2> in <module>
----> 1 unicodedata.decimal("四")

ValueError: not a decimal

In [12]: unicodedata.numeric("四")
Out[12]: 4.0

In [13]: unicodedata.numeric("壹")
Out[13]: 1.0

In [14]: unicodedata.numeric("肆")
Out[14]: 4.0

```

## 字符串的查找与替换
序号|方法名|说明
---|---|---
1|count(sub[,start[,end]])|    
2|find(sub[,start[,end]])|   
3|index(sub[,start[,end]])| 
4|rfind(sub[,start[,end]])|
5|rindex(sub[,start[,end]])|
6|replace(old,new[,count])|如果指定count，就替换count次，如果没有，就全部替换
7|split()|分拆字符串
8|title()|
9|capitalize()|
10|swapcase()|
11|lower()|
12|upper()|

```
In [1]: a="test is a test,not a exam"

In [2]: a.find("test")
Out[2]: 0

In [3]: a.find("ltest")
Out[3]: -1

In [4]: a.lfind("test")
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-4-cc9ada575631> in <module>
----> 1 a.lfind("test")

AttributeError: 'str' object has no attribute 'lfind'

In [5]: a.rfind("test")
Out[5]: 10

In [6]: a.rindex("test")
Out[6]: 10

In [7]: a.index("test")
Out[7]: 0

In [8]: "test" in a
Out[8]: True

In [9]: a.count("test")
Out[9]: 2

In [10]: a.count("test",1,2)
Out[10]: 0

In [11]: a.count("test",1)
Out[11]: 1

In [13]: b="Hello World"

In [14]: b.swapcase()
Out[14]: 'hELLO wORLD'

```

## 字符串的删减与填充
序号|方法名|说明
---|---|---
1|strip([chars])| 没有指定chars就删除空白符
2|lstrip([chars])| 空白符由string.whitespace定义
3|rstrip([chars])| 
4|center(width[,fillchar])|
5|ljust(width[,fillchar])|
6|rjust(width[,fillchar])|如果指定count，就替换count次，如果没有，就全部替换
7|zfill(width)|分拆字符串
8|expandtabs([tabsize])|tabsize默认为8

```
In [1]: import string

In [2]: string.whitespace
Out[2]: ' \t\n\r\x0b\x0c'
```

