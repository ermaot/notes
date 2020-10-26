mmap 模块提供了操作系统内存映射函数的接口

```
In [1]: import mmap
   ...: import os
   ...:
   ...: filename = "1.txt"
   ...:
   ...: file = open(filename, "r+")
   ...: size = os.path.getsize(filename)
   ...:
   ...: data = mmap.mmap(file.fileno(), size)
   ...:
   ...: print(data)
   ...: print(len(data), size)
   ...:
   ...: print(repr(data[:10]), repr(data[:10]))
   ...:
   ...: print(repr(data.read(10)), repr(data.read(10)))
<mmap.mmap object at 0x000000177BB587B0>
18 18
b'test1 test' b'test1 test'
b'test1 test' b' 2 test3'
In [6]: m = re.search(b"test",data)

In [7]: print(m.start(),m.group())
0 b'test'
```

在 Windows 下, 这个文件必须以既可读又可写的模式打开( `r+` , `w+` , 或 `a+` ), 否则 mmap 调用会失败

如果需要使用正则或者find等函数，子串需要为二进制模式（带b）

