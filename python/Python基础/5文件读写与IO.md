## 各种读写操作

```
//一次性读取全部行
In [1]: with open('1.txt', 'rt') as f:
   ...:     data = f.read()
   ...:

//按行读取
In [5]: with open('1.txt', 'rt') as f:
   ...:     for line in f:
   ...:         print(line,end='')
   ...:
abi2
ache

//覆盖写文件
In [6]: with open('1.txt', 'wt') as f:
   ...:     f.write("test")

//追加写文件
In [7]: with open('1.txt', 'at') as f:
   ...:     f.write("test")
   ...:
//二进制以wb，rb方式
//如果不存在才能写入，使用xb和st方式
//指定编码方式
with open('somefile.txt', 'rt',encoding='latin-1') as f:

//指定换行符
with open('somefile.txt', 'rt',newline='') as f:

//出现错误的处理方式
open('sample.txt', 'rt',encoding='ascii',errors='replace')
open('sample.txt', 'rt',encoding='ascii',errors='ignore')

//打印输出到文件中
In [1]: with open('1.txt', 'wt') as f:
   ...:     print("test2",file=f)
   ...:
 
 //打印到文件并指定输出的分隔符和换行符  
In [2]: with open('1.txt', 'wt') as f:
   ...:     print("test2","test3","test4",sep=',',end="\n",file=f)

In [5]: with open('1.txt', 'wt') as f:
   ...:     print(*[1,2,3,4],sep=".",file=f)
   ...:     print(','.join(['a','b','c','d']),sep=",",file=f)

//使用二进制写入
In [13]: with open('1.txt', 'wb') as f:
    ...:     f.write(b'HelloWorld')



```
## 读写压缩文件

```
import gzip
with gzip.open('somefile.gz', 'rt') as f:
    text = f.read()

import bz2
with bz2.open('somefile.bz2', 'rt') as f:
    text = f.read()

import gzip
with gzip.open('somefile.gz', 'wt') as f:
    f.write(text)

import bz2
with bz2.open('somefile.bz2', 'wt') as f:
    f.write(text)
```

## 内存映射一个二进制文件

```
In [1]: import os,mmap

In [2]: m = mmap.mmap(os.open("1.txt",os.O_RDWR),os.path.getsize("1.txt"),access=mmap.ACCESS_WRITE)

In [3]: m[:]
Out[3]: b'Hell.World'

In [4]: m[4:5]=b'o'

In [5]: m[:]
Out[5]: b'HelloWorld'

In [6]: m.close()
//默认情况下， memeory map() 函数打开的文件同时支持读和写操作，参数是access=mmap.ACCESS_WRITE，还可以是ACCESS_COPY，ACCESS_READ
```

mmap() 返回的 mmap 对象同样也可以作为一个上下文管理器来使用，这时候底层的文件会被自动关闭

```

```
操作系统仅仅为文件内容保留了一段虚拟内存。当你访问文件的不同区域时，这些区域的内容才根据需要被读取并映射到内存区域中。而那些从没被访问到的部分还是留在磁盘上


## 文件路径名的操作

```
In [1]: import os

In [2]: path = '/Users/beazley/Data/data.csv'

In [3]: os.path.basename(path)
Out[3]: 'data.csv'

In [4]: os.path.dirname(path)
Out[4]: '/Users/beazley/Data'

In [5]: path = '~/Data/data.csv'

In [6]: os.path.dirname(path)
Out[6]: '~/Data'

In [7]: os.path.basename(path)
Out[7]: 'data.csv'

In [8]: os.path.expanduser(path)
Out[8]: 'C:\\Users\\Administrator/Data/data.csv'

In [9]: os.path.splitext(path)
Out[9]: ('~/Data/data', '.csv')

```

## 测试文件是否存在

```
In [1]: import os

In [2]: os.path.exists("1.txt")
Out[2]: True

In [3]: os.path.exists("12.txt")
Out[3]: False

In [4]: os.path.isfile("1.txt")
Out[4]: True

In [5]: os.path.isdir("1.txt")
Out[5]: False

In [6]: os.path.islink("1.txt")
Out[6]: False

In [7]: os.path.realpath("1.txt")
Out[7]: 'C:\\Users\\Administrator\\1.txt'

In [8]: os.path.getsize("1.txt")
Out[8]: 10

In [9]: os.path.getmtime("1.txt")
Out[9]: 1565334825.137848
```

## 获得目录列表并过滤

```
In [1]: import os

In [2]: os.listdir(".")
Out[2]:

In [3]: import glob

In [4]: pyfiles = glob.glob('somedir/*.py')

In [5]: pyfiles
Out[5]: []
```

## 忽略文件名编码

```
In [1]: import sys

In [2]: sys.getfilesystemencoding()
Out[2]: 'utf-8'

In [9]:os.listdir(b'.')                         
Out[2]: [b'\xe3\x80\x8c\xe5\xbc\x80\xe5\xa7\x8b\xe3\x80\x8d\xe8\x8f\x9c\xe5\x8d\x95',]
```
## 打印不合法的文件名

```
repr()
```
## 增加或改变已打开文件的编码
给一个以二进制模式打开的文件添加 Unicode 编码/解码方式

```
In [1]: import io

In [2]: io.TextIOWrapper(open("1.txt",'rb'),encoding='utf-8').read()
Out[2]: 'test'
```
想修改一个已经打开的文本模式的文件的编码方式，可以先使用 detach()方法移除掉已存在的文本编码层，并使用新的编码方式代替
```
In [1]: import sys,io

In [2]: sys.stdout.encoding
Out[2]: 'utf-8'

In [3]: sys.stdout = io.TextIOWrapper(sys.stdout.detach(),encoding='gbk')
```
## 将在写入文本文件

```
In [1]: import sys

In [2]: sys.stdout.buffer.write(b'Hello\n')
Hello
Out[2]: 6
```


## 操作临时文件

```
In [3]: from tempfile import TemporaryFile

In [4]: from tempfile import NamedTemporaryFile

In [11]: with TemporaryFile('w+t') as f:
    ...:     f.write('HelloWorld\n')
    ...:     f.seek(0)                      //这一步不可少。表示回到文件起始处
    ...:     data = f.read()

//已命名的临时文件
In [13]: with NamedTemporaryFile('w+t') as f:
    ...:     print('filenameis:',f.name)
    ...:
filenameis: C:\Users\ADMINI~1\AppData\Local\Temp\2\tmpf7sfd5mq

//已命名的临时文件夹
In [14]: from tempfile import TemporaryDirectory

In [15]: with TemporaryDirectory() as dirname:
    ...:     print('dirnameis:',dirname)
    ...:
dirnameis: C:\Users\ADMINI~1\AppData\Local\Temp\2\tmphr5uy6uy

```

## 与串口通信

```
import serial
ser = serial.Serial('/dev/tty.usbmodem641', baudrate=9600,bytesize=8,parity='N',stopbits=1)
ser.write(b'G1X50Y50\r\n')
resp = ser.readline()
```

## 序列化python对象

```
In [16]: class test():
    ...:     pass
    ...:

In [17]: import pickle

In [18]:

In [18]:

In [18]: data = test()

In [19]: f = open('somefile', 'wb')
    ...: pickle.dump(data,f)
    ...: s = pickle.dumps(data)

In [20]:

In [20]: s
Out[20]: b'\x80\x03c__main__\ntest\nq\x00)\x81q\x01.'

In [21]: print(s)
b'\x80\x03c__main__\ntest\nq\x00)\x81q\x01.'

In [22]: f = open('somefile', 'rb')
    ...: data = pickle.load(f)

In [23]: data = pickle.loads(s)

In [24]: data
Out[24]: <__main__.test at 0x6195f8c4a8>



In [1]: import pickle
   ...: f = open('somedata', 'wb')
   ...: pickle.dump([1, 2, 3, 4],f)
   ...: pickle.dump('hello',f)
   ...: pickle.dump({'Apple', 'Pear', 'Banana'},f)
   ...: f.close()
   ...: f = open('somedata', 'rb')
   ...: pickle.load(f)
Out[1]: [1, 2, 3, 4]

In [2]:

In [2]: pickle.load(f)
Out[2]: 'hello'

In [3]: pickle.load(f)
Out[3]: {'Apple', 'Banana', 'Pear'}

In [4]:
```
