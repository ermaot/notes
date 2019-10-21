## 高效分析性能
## Julia集合的介绍
Julia集合是个有趣的CPU密集型问题，包含一个CPU密集型的组件和一个显式的输入集合
## 计算完整的Julia集合
## 计时的方法

```
from functools import wraps 
 
def timefn(fn): 
    @wraps(fn) 
    def measure_time(*args, **kwargs): 
        t1 = time.time()
        result = fn(*args, **kwargs) 
        t2 = time.time() 
        print ("@timefn:" + fn.func_name + " took " + str(t2 - t1) + " seconds") 
        return result 
    return measure_time
```
使用@wraps(fn) 保留fn函数的原属性


```
python -m timeit -n 5 -r 5 -s "import julia1" 
   "julia1.calc_pure_python(desired_width=1000, 
   max_iterations=300)" 
```
-n 5 循环5次<p>
-r 5 重复次数

Ipython内部有%timeit魔法函数

```
%timeit calc_pure_python(desired_width=1000, max_iterations=300) 
```

## 使用Unix的time命令计时

```
# /usr/bin/time -p python julia1_nopil.py 

或者

/usr/bin/time --verbose python julia1_nopil.py 
```
此处使用的是/usr/bin/time 而非直接使用time命令
## 使用CProfile模块
- cProfile 是一个标准库内建的分析工具。它钩入CPython的虚拟机来测量其每一个函数运行所花费的时间，但会引入一个巨大的开销
- cProfile是标准库内建的三个分析工具之一，另外两个是hotshot和profile。hotshot还处于实验阶段，profile则是原始的纯Python分析器

```
# python -m cProfile -s cumulative julia1_nopil.py
```
命令中的-s cumulative开关告诉cProfile对每个函数累计花费的时间进行排序
## 用runsnakerun对CProfile输出进行可视化
安装
```
//书上的pip install runsnake是安装不成功的
pip install SquareMap RunSnakeRun


```
使用

```
# python -m cProfile -o <outputfilename> <script-name> <options>

或者
import cProfile
command = """reactor.run()"""
cProfile.runctx( command, globals(), locals(), filename="OpenGLContext.profile" )

# python runsnake.py OpenGLContext.profile
或者
# python -m runsnakerun.runsnake OpenGLContext.profile
```

结果
![image](https://github.com/ermaot/notes/blob/master/python/python%E5%8E%9F%E7%90%86%E4%B8%8E%E4%BC%98%E5%8C%96/pic/python%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E6%96%B9%E6%B3%951.png)

还可以有类似的工具runsnakemem 

```
# pip install  meliae

from meliae import scanner
scanner.dump_all_objects( filename )

# runsnakemem <filename>
```
![image](https://github.com/ermaot/notes/blob/master/python/python%E5%8E%9F%E7%90%86%E4%B8%8E%E4%BC%98%E5%8C%96/pic/python%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E6%96%B9%E6%B3%952.png)
## 用line_profile进行逐行分析
创建测试py程序文件

```
import line_profiler
import sys

def test():
    for i in range(100):
        a = sum(range(1000000))
profile = line_profiler.LineProfiler(test)
profile.enable()
test()
profile.disable()
profile.print_stats(sys.stdout) 
```
或者直接

```
@profile
def test():
    for i in range(100):
        a = sum(range(1000000))
test()
```

按照行分析
```
# kernprof   -l -v  test.py    
Timer unit: 1e-06 s

Total time: 2.97179 s
File: test.py
Function: test at line 4

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     4                                           def test():
     5       101        531.0      5.3      0.0      for i in range(100):
     6       100    2971255.0  29712.5    100.0          a = sum(range(1000000))

Wrote profile results to test.py.lprof
Timer unit: 1e-06 s
```
或者按照函数分析，此时需要删除line_profiler代码

```
def test():
    for i in range(100):
        a = sum(range(1000000))

test()

```

```
# kernprof   -v  test.py    
Wrote profile results to test.py.prof
         207 function calls in 2.933 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    2.933    2.933 <string>:1(<module>)
        1    0.000    0.000    2.933    2.933 test.py:4(<module>)
        1    0.717    0.717    2.933    2.933 test.py:4(test)
        1    0.000    0.000    2.933    2.933 {execfile}
        1    0.000    0.000    0.000    0.000 {globals}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
      101    1.200    0.012    1.200    0.012 {range}
      100    1.016    0.010    1.016    0.010 {sum}
```


## 用memory_profile诊断内存的用量

```
# python -m memory_profiler  test.py
Filename: test.py

Line #    Mem usage    Increment   Line Contents
================================================
     3   18.715 MiB   18.715 MiB   @profile
     4                             def test():
     5   49.770 MiB    0.000 MiB       for i in range(100):
     6   49.770 MiB   23.426 MiB           a = sum(range(1000000))
```

```
# mprof run test.py
# mprof plot mprofile_20190824194402.dat
```

## 用heapy调查堆上的对象

## 用dowser实时画出变量的实例
## 用dis模块检查CPython字节码

```
import dis
def test():
    for i in range(100):
        a = sum(range(1000000))

dis.dis(test)
```

```
# python test.py
  6           0 SETUP_LOOP              38 (to 41)
              3 LOAD_GLOBAL              0 (range)
              6 LOAD_CONST               1 (100)
              9 CALL_FUNCTION            1
             12 GET_ITER            
        >>   13 FOR_ITER                24 (to 40)
             16 STORE_FAST               0 (i)

  7          19 LOAD_GLOBAL              1 (sum)
             22 LOAD_GLOBAL              0 (range)
             25 LOAD_CONST               2 (1000000)
             28 CALL_FUNCTION            1
             31 CALL_FUNCTION            1
             34 STORE_FAST               1 (a)
             37 JUMP_ABSOLUTE           13
        >>   40 POP_BLOCK           
        >>   41 LOAD_CONST               0 (None)
             44 RETURN_VALUE        
```

#### 不同的方法，不同的复杂度
## 在优化期间进行单元测试保持代码的正确性
#### No-op的@profile修饰器
## 确保性能分析成功的策略
- 关闭任何基于BIOS的加速器，因为它们只会混淆你的结果
- 在 BIOS 上禁用了 TurboBoost。 
- 禁用了操作系统改写SpeedStep（如果你有权限，你可以在你的BIOS中找到它）的能力
- 只使用主电源（从不使用电池电源）
- 运行实验时禁用后台工具如备份和 Dropbox。
- 多次运行实验来获得一个稳定的测量结果。
- 如果可能，降至run level1（UNIX）确保没有其他任务运行。
- 重启并重跑实验来二次验证结果