字符串连接，有+和join两种方式，但哪种效率高呢？下面来测试以下``

```
In [81]: def join_str(str):
    ...:     return ''.join(str)
    ...:

In [82]: def plus_str(str):
    ...:     result = ''
    ...:     for i in str:
    ...:         result = result +i
    ...:     return result
    ...:
```

结果如下：

```
In [84]: str = ["Hello World" for i in range(10)]

In [86]: %timeit join_str(str)
290 ns ± 5.06 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)

In [87]: %timeit plus_str(str)
950 ns ± 9.66 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)

In [88]: str = ["Hello World" for i in range(100)]

In [89]: %timeit join_str(str)
1.08 µs ± 13.2 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)

In [90]: %timeit plus_str(str)
15.1 µs ± 96 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)

In [91]: str = ["Hello World" for i in range(1000)]

In [92]: %timeit join_str(str)
8.13 µs ± 144 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)

In [93]: %timeit plus_str(str)
189 µs ± 3.45 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)

```

可见随着字符串的越来越长，二者之间性能差距越来越大。为什么呢？

因为字符串是不可变对象，对字符串+操作，都会申请一块新的内存空间，将每次+的结果重新复制到该内存空间中，时间复杂度可以认为是O(n^2)

而join操作，会事先计算空间大小，一次性申请内存，并将内容复制到该内存中，时间复杂度为O(n)

