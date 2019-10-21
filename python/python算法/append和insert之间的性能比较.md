## append性能

```
In [1]: import time

In [2]: def test_append(count):
   ...:     start = time.clock()
   ...:     nums = []
   ...:     for i in range(count):
   ...:         nums.append(i)
   ...:     nums.reverse()
   ...:     end = time.clock()
   ...:     print(end-start)
   ...:

In [3]: def test_insert(count):
   ...:     start = time.clock()
   ...:     nums = []
   ...:     for i in range(count):
   ...:         nums.insert(0,i)
   ...:     end = time.clock()
   ...:     print(end-start)
   ...:

In [4]: test_insert(10**5)
5.043418088053091

In [5]: test_append(10**5)
0.009924166339580154

In [6]: test_append(10**4)
0.000852622330491215

In [7]: test_insert(10**4)
0.043740475395615874

```
可以看到当数据量为1000时，insert耗时大约是append的5倍；当数据量为10000时，insert耗时大约是append的50倍；当数据量是100000时，insert耗时大约是append的500倍


```
In [24]: def test_insert(count):
    ...:      start = time.clock()
    ...:      nums = []
    ...:      for i in range(count):
    ...:          nums.insert(i-1,i)
    ...:      nums.reverse()
    ...:      end = time.clock()
    ...:      print(end-start)
    ...:

In [25]: test_insert(10**5)
0.03260470255298742

In [26]: test_insert(10**4)
0.0020343367662576384
```
这样不管数据量多少，insert大约是append耗时的3倍了

测量与分析

```
In [39]: import cProfile

In [40]: cProfile.run("test_insert(10000)")
         10005 function calls in 0.089 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.046    0.046    0.089    0.089 <ipython-input-35-e60b2ac0c72a>:1(test_insert)
        1    0.000    0.000    0.089    0.089 <string>:1(<module>)
        1    0.000    0.000    0.089    0.089 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    10000    0.043    0.000    0.043    0.000 {method 'insert' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'reverse' of 'list' objects}
        
        

In [42]: cProfile.run("test_append(10000)")
0.08218502630916191
         10028 function calls in 0.086 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.042    0.042    0.086    0.086 <ipython-input-41-fded3ba17786>:1(test_append)
        1    0.000    0.000    0.086    0.086 <string>:1(<module>)
        2    0.000    0.000    0.004    0.002 ansitowin32.py:160(write)
        2    0.000    0.000    0.004    0.002 ansitowin32.py:177(write_and_convert)
        2    0.000    0.000    0.004    0.002 ansitowin32.py:193(write_plain_text)
        2    0.000    0.000    0.000    0.000 ansitowin32.py:245(convert_osc)
        2    0.000    0.000    0.004    0.002 ansitowin32.py:40(write)
        1    0.000    0.000    0.086    0.086 {built-in method builtins.exec}
        2    0.000    0.000    0.000    0.000 {built-in method builtins.len}
        1    0.000    0.000    0.004    0.004 {built-in method builtins.print}
        2    0.000    0.000    0.000    0.000 {built-in method time.clock}
    10000    0.040    0.000    0.040    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        4    0.000    0.000    0.000    0.000 {method 'finditer' of '_sre.SRE_Pattern' objects}
        2    0.004    0.002    0.004    0.002 {method 'flush' of '_io.TextIOWrapper' objects}
        1    0.000    0.000    0.000    0.000 {method 'reverse' of 'list' objects}
        2    0.000    0.000    0.000    0.000 {method 'write' of '_io.TextIOWrapper' objects}
```
