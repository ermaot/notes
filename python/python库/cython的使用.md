## 安装

```
pip install cython
```

## 代码（python版）

```
In [1]: def sum(n):
    ...:     total = 0
    ...:     for i in range(1,n+1):
    ...:         total += i
    ...:     return total
In [2]: %timeit sum(1000)
54.2 µs ± 607 ns per loop (mean ± std. dev. of 7 runs, 10000 loops each)
```

## 代码（cython版）

```
In [3]: %%cython
    ...: def sumc(int n):
    ...:     total = 0
    ...:     for i in range(1,n+1):
    ...:         total += i
    ...:     return total
    ...:     

In [4]: %timeit sumc(1000)
33.2 µs ± 1.21 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)

```

