- 我怎样让我的 Python代码作为低级代码来运行？ 
- JIT 编译器和 AOT 编译器的区别是什么？ 
- 编译后的 Python代码运行什么任务能够比本地 Python快？ 
- 为什么类型注解提升了编译后 Python代码的运行速度？ 
- 我该怎样使用 C或 Fortran为 Python编写模块？ 
- 我该怎样在 Python中使用 C或者 Fortran 的库？
## 可能获得哪种提升
- 使用特定编译方式，有可能得到至少一个数量级大小的速度提升
- 调用外部库（例如，正则表达式、字符串操作、调用数据库）的代码在编译后不可能表现出任何速度提升
- I/O密集型的程序同样不可能表现出明显的速度提升
- 如果Python代码集中于调用向量化的numpy例程，那么在编译后就不大可能运行得更快，只有当被编译的代码主要是 Python（并且可能主要是循环）时才会运行得更快
- ==编译后的代码不可能比手工精心编写的C例程运行得更快，但也不可能比它慢很多==
- 
![image](281AF33CC17E4FEA8DDC21980E39FE35)
## JIT和AOT编译器的对比
编译工具大体分为两类：提前编译工具（Cython、Shed Skin、Pythran）和“即时”编译工具（Numba、PyPy）
- 通过提前编译（AOT），你会创建一个为你的机器定制的静态库
- 通过即时编译，你不必提前做很多（如果有的话），你让编译器在使用时只逐步编译恰到好处的那部分代码。这就意味着你会有“冷启动”问题
## 为什么类型检查有助于代码更快运行
- Python是动态类型的，并且任意代码行都能够改变被引用对象的类型。这使得虚拟机难以在机器码层面优化代码的运行方式，因为它不知道哪种基础数据类型会用于将来的运算。
- CPU 密集型的代码区域内部，不改变变量类型的情况很常见可以做静态编译加快代码运行。 
## 使用C编译器

## Cython
#### 使用Cython编译纯python
创建文件cythonfn.pyx 

```
def calculate_z(maxiter, zs, cs):
    """Calculate output list using Julia update rule"""
    output = [0] * len(zs)
    for i in range(len(zs)):
        n = 0
        z = zs[i]
        c = cs[i]
        while n < maxiter and abs(z) < 2:
            z = z * z + c
            n += 1
        output[i] = n
    return output
```
创建文件setup.py

```
from distutils.core import setup
from distutils.extension import Extension
from Cython.Distutils import build_ext

setup(
        cmdclass = {'build_ext': build_ext},
        ext_modules = [Extension("calculate", ["cythonfn.pyx"])]
        )
```
执行

```
# python setup.py build_ext --inplace 
```
--inplace 参数让 Cython 在当前目录中构建编译模块， 而不是在一个独立的构建目录中。在构建完成后，我们会有两个难以卒读的中间文件 cythonfn.c 和calculate.so。 
#### Cython注解来分析代码
cython –a cythonfh.pyx 来产生输出文件cythonfn.html

```
# cython  -a cythonfn.pyx
/usr/lib64/python2.7/site-packages/Cython/Compiler/Main.py:369: FutureWarning: Cython directive 'language_level' not set, using 2 for now (Py2). This will change in a later release! File: /root/cythonfn.pyx
  tree = Parsing.p_module(s, pxd, full_module_name)
```

#### 增加一些类型注解
## shed skin
#### 构建扩展模块
#### 内存拷贝的开销
## Cython和numpy
#### 在一台机器上使用OpenMP来做并行解决方案
## Numba
## Pythran
## pypy
#### 垃圾收集的差异
#### 运行pypy并安装模块
## 什么时候使用每种工具
#### 其他即将出现的项目
#### 一个图像处理单元GPU注意点
#### 一个对未来编译器项目的展望
## 外部函数接口
#### ctypes
#### cffi
#### f2py
#### CPython模块