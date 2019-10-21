PyUnit（unittest） 是 Python 自带的单元测试框架，用于编写和运行可重复的测试。PyUnit 是xUnit 体系的一个成员，xUnit 是众多测试框架的总称，PyUnit 主要用于进行白盒测试和回归测试。
## PyUnit（unittest） 
- PyUnit 是一个简单、易用的测试框架，其具有如下特征：
1. 使用断言方法判断期望值和实际值的差异，返回 bool 值。
2. 测试驱动设备可使用共同的初始化变量或实例。
3. 测试包结构便于组织和集成运行。

- PyUnit (unittest) 优点：
1. 可以使测试代码与产品代码分离。
2. 针对某一个类的测试代码只需要进行较少的改动，便可以应用于另一个类的测试。
3. PyUnit 开放源代码，可以进行二次开发，方便对 PyUnit 的扩展。


## 测试驱动开发（TDD）
测试驱动开发强调结果导向，也就是在开发某个功能之前，先定义好该功能的最终结果（测试用例关注函数的执行结果），然后再去开发该功能。就像建筑工人在砌墙之前，要先拉好一根笔直的绳子（作用相当于测试用例），然后再开始砌墙，这样砌出来的墙就会符合标准。所以说测试驱动开发确实是一种不错的开发方式

假如程序要开发满足 A 功能的 fun_a() 函数，采用测试驱动开发的步骤如下：
1. 为 fun_a() 函数编写测试用例，根据业务要求，使用大量不同的参数组合来执行 fun_a() 函数，并断言该函数的执行结果与业务期望的执行结果匹配。
2. 编写、修改 fun_a() 函数。
3. 运行 fun_a() 函数的测试用例，如果测试用例不能完全通过；则重复第 2 步和第 3 步，直到 fun_a() 的所有测试用例全部通过。

## PyUnit (unittest) 的用法
- 开发一个简单的 fk_math.py 程序，该程序包含两个函数，分别用于计算一元一次方程的解和二元一次方程的解

```
def one_equation(a , b):
    '''
    求一元一次方程a * x + b = 0的解
    参数a - 方程中变量的系数
    参数b - 方程中的常量
    返回 方程的解
    '''
    # 如果a = 0，则方程无法求解
    if a == 0:
        raise ValueError("参数错误")
    # 返回方程的解
    else:
        return -b / a  
#        return b / a
def two_equation(a , b , c):
    '''
    求一元二次方程a * x * x + b * x + c = 0的解
    参数a - 方程中变量二次幂的系数
    参数b - 方程中变量的系数
    参数c - 方程中的常量
    返回 方程的根
    '''
    # 如果a == 0，变成一元一次方程
    if a == 0:
        raise ValueError("参数错误")
    # 有理数范围内无解
    elif b * b - 4 * a * c < 0:
        raise ValueError("方程在有理数范围内无解")
    # 方程有唯一的解
    elif b * b - 4 * a * c == 0:
        # 使用数组返回方程的解
        return -b / (2 * a)
    # 方程有两个解
    else:
        r1 = (-b + (b * b - 4 * a * c) ** 0.5) / 2 / a
        r2 = (-b - (b * b - 4 * a * c) ** 0.5) / 2 / a
        # 方程的两个解
        return r1, r2
```
保存为fk_math.py

- unittest 要求单元测试类必须继承unittest.TestCase，该类中的测试方法需要满足如下要求：
1. 测试方法应该没有返回值。
2. 测试方法不应该有任何参数。
3. 测试方法应以test 开头。

编写测试用例

```
import unittest
from fk_math import *
class TestFkMath(unittest.TestCase):
    # 测试一元一次方程的求解
    def test_one_equation(self):
        # 断言该方程求解应该为-1.8
        self.assertEqual(one_equation(5 , 9) , -1.8)
        # 断言该方程求解应该为-2.5
        self.assertTrue(one_equation(4 , 10) == -2.5 , .00001)
        # 断言该方程求解应该为27/4
        self.assertTrue(one_equation(4 , -27) == 27 / 4)
        # 断言当a == 0时的情况，断言引发ValueError
        with self.assertRaises(ValueError):
            one_equation(0 , 9)
    # 测试一元二次方程的求解
    def test_two_equation(self):
        r1, r2 = two_equation(1 , -3 , 2)
        self.assertCountEqual((r1, r2), (1.0, 2.0), '求解出错')
        r1, r2 = two_equation(2 , -7 , 6)
        self.assertCountEqual((r1, r2), (1.5, 2.0), '求解出错')
        # 断言只有一个解的情形
        r = two_equation(1 , -4 , 4)
        self.assertEqual(r, 2.0, '求解出错')
        # 断言当a == 0时的情况，断言引发ValueError
        with self.assertRaises(ValueError):
            two_equation(0, 9, 3)
        # 断言引发ValueError
        with self.assertRaises(ValueError):
            two_equation(4, 2, 3)

if __name__ == '__main__':
    unittest.main()
```
保存为TestFkMath.py

- 执行

```
//执行某个具体的用例
python -m unittest TestFkMath.py

或者
//当前目录下所有的测试用例
py -m unittest
```



- 补充内容
1. unittest.TestCase 内置了大量 assertXxx 方法来执行断言，其中最常用的断言方法如表所示

断言方法|	检查条件
---|---
assertEqual(a, b)	|a == b
assertNotEqual(a, b)	|a != b
assertTrue(x)	|bool(x) is True
assertFalse(x)	|bool(x) is False
assertIs(a, b)	|a is b
assertIsNot(a, b)	|a is not b
assertIsNone(x)|	x is None
assertIsNotNone(x)	|x is not None
assertIn(a, b)|	a in b
assertNotIn(a, b)|	a not in b
assertlsInstance(a, b)	|isinstance(a, b)
assertNotIsInstance(a, b)	|not isinstance(a, b)

2. 程序要对异常、错误、警告和日志进行断言判断，TestCase 提供了如表所示的断言方法

断言方法|	检查条件
---|---
assertRaises(exc, fun, *args, **kwds)	|fun(*args, **kwds) 引发 exc 异常
assertRaisesRegex(exc, r, fun, *args, **kwds)	fun(*args, **kwds) |引发 exc 异常，且异常信息匹配 r 正则表达式
assertWarns(warn, fun, *args, **kwds)	fun(*args, **kwds) |引发 warn 警告
assertWamsRegex(warn, r, fun, *args, **kwds)	fun(*args, **kwds) |引发 warn 警告，且警告信息匹配 r 正则表达式
assertLogs(logger, level)	|With 语句块使用日志器生成 level 级别的日志

3. 用于完成某种特定检查的断言方法
断言方法	|检查条件
---|---
assertAlmostEqual(a, b)	|round(a-b, 7) == 0
assertNotAlmostEqual(a, b)	|round(a-b, 7) != 0
assertGreater(a, b)|	a > b
assertGreaterEqual(a, b)|	a >= b
assertLess(a, b)|	a < b
assertLessEqual(a, b)|	a <= b
assertRegex(s, r)	|r.search(s)
assertNotRegex(s, r)	|not r.search(s)
assertCountEqual(a, b)	|a、b 两个序列包含的元素相同，不管元素出现的顺序如何

4. 当测试用例使用 assertEqual() 判断两个对象是否相等时，如果被判断的类型是字符串、序列、列表、元组、集合、字典，则程序会自动改为使用下表所示的断言方法进行判断。换而言之，表中所示的断言方法其实没有必要使用，unittest 模块会自动应用它们

断言方法|	用于比较的类型
---|---
assertMultiLineEqual(a, b)	|字符串(string)
assertSequenceEqual(a, b)	|序列(sequence)
assertListEqual(a, b)	|列表(list)
assertTupleEqual(a, b)	|元组(tuple)
assertSetEqual(a, b)	|集合(set 或 frozenset)
assertDictEqual(a, b)	|字典(dict)



## unittest 单元测试框架解析

#### 样例
```
# 将要被测试的类
classWidget:
    def __init__(self, size = (40, 40)):
        self._size = size
    def getSize(self):
        return self._size
    def resize(self, width, height):
        if width < 0 or height < 0:
            raise ValueError, "illegal size"
        self._size = (width, height)
    def dispose(self):
        pass
```

```
from widget import Widget
import unittest
# 执行测试的类
classWidgetTestCase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget()
    def testSize(self):
        self.assertEqual(self.widget.getSize(), (40, 40))
    def tearDown(self):
        self.widget = None
# 构造测试集
def suite():
    suite = unittest.TestSuite()
    suite.addTest(WidgetTestCase("testSize"))
    return suite

# 测试
if __name__ == "__main__":
    unittest.main(defaultTest = 'suite')
```
1. 用 import 语句引入 unittest 模块让所有执行测试的类都继承于 TestCase 类，可以将 TestCase 看成是对特定类进行测试的方法的集合
2. setUp()方法中进行测试前的初始化工作，tearDown()方法中执行测试后的清除工作。setUp()和 tearDown()都是 TestCase 类中定义的方法
3. 在 testSize()中调用 assertEqual()方法，对 Widget 类中getSize()方法的返回值和预期值进行比较，确保两者是相等的，assertEqual()也是 TestCase 类中定义的方法。
4. 提供名为 suite()的全局方法， PyUnit 在执行测试的过程调用 suit()方法来确定有多少个测试用例需要被执行，可以将 TestSuite 看成是包含所有测试用例的一个容器。


#### 框架分析
- 编写测试用例
采用 PyUnit 提供的动态方法，只编写一个测试类来完成对整个软件模块的测试，这样对象的初始化工作可以在 setUp()方法中完成，而资源的释放则可以在 tearDown()方法中完成

- 组织用例集
完整的单元测试很少只执行一个测试用例，开发人员通常都需要编写多个测试用例才能对某一软件功能进行比较完整的测试，这些相关的测试用例称为一个测试用例集，在PyUnit中是用 TestSuite 类来表示的。可以在单元测试代码中定义一个名为suite()的全局函数，并将其作为整个单元测试的入口，PyUnit 通过调用它来完成整个测试过程

```
def suite():
    suite = unittest.TestSuite()
    suite.addTest(WidgetTestCase("testSize"))
    suite.addTest(WidgetTestCase("testResize"))
    return suite

```
如果用于测试的类中所有的测试方法都以 test 开头，Python 程序员甚至可以用 PyUnit 模块提供的makeSuite()方法来构造一个。

```
def suite():
    return unittest.makeSuite(WidgetTestCase, "test")
```

#### 运行测试集
PyUnit 使用 TestRunner类作为测试用例的基本执行环境，来驱动整个单元测试过程。Python 开发人员在进行单元测试时一般不直接使用 TestRunner 类，而是使用其子类TextTestRunner来完成测试，并将测试结果以文本方式显示出来：

```
runner = unittest.TextTestRunner()
runner.run(suite)
```
对 widget.py 被测试类，下面通过 PyUnit 编写完整的单元测试用例：


```
//text_runner.py
from widget import Widget
import unittest
# 执行测试的类
classWidgetTestCase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget()
    def tearDown(self):
        self.widget.dispose()
        self.widget = None
    def testSize(self):
        self.assertEqual(self.widget.getSize(), (40, 40))
    def testResize(self):
        self.widget.resize(100, 100)
        self.assertEqual(self.widget.getSize(), (100, 100))
# 测试
if __name__ == "__main__":
# 构造测试集
    suite = unittest.TestSuite()
    suite.addTest(WidgetTestCase("testSize"))
    suite.addTest(WidgetTestCase("testResize"))
# 执行测试
    runner = unittest.TextTestRunner()
    runner.run(suite)
```
PyUnit 模块中定义了一个名为main的全局方法，使用它可以很方便地将一个单元测试模块变成可以直接运行的测试脚本，main()方法使用TestLoader类来搜索所有包含在该模块中的测试方法，并自动执行它们。如果Python程序员能够按照约定（以test开头）来命名所有的测试方法，那就只需要在测试模块的最后加入如下几行代码即可

```
if __name__ == "__main__":
    unittest.main()
```

## 批量执行测试用例
进入文件夹，然后遍历用例，分别执行，同时将测试结果输出到日志文件中


## 测试报告
#### 生成 HTMLTestRunner 测试报告
- 安装HTMLTestRunner
1. 方法1：下 HTMLTestRunner.py，放入python安装目录的Lib下
2. 方法2：下载[wheel安装](https://pypi.org/project/html-testRunner/#modal-close)
```
import HtmlTestRunner
```

- 使用方法
1. 用例写入到一个文件
```
from selenium import webdriver 
from selenium.webdriver.common.by import By 
from selenium.webdriver.common.keys import Keys 
from selenium.webdriver.support.ui import Select 
from selenium.common.exceptions import NoSuchElementException 
import unittest, time, re 
import HTMLTestRunner #引入HTMLTestRunner包 
class Baidu(unittest.TestCase): 
    def setUp(self): 
        self.driver = webdriver.Firefox() 
        self.driver.implicitly_wait(30) 
        self.base_url = "http://www.baidu.com/" 
        self.verificationErrors = [] 
        self.accept_next_alert = True 
    #百度搜索用例 
    def test_baidu_search(self): 
        driver = self.driver 
        driver.get(self.base_url + "/") 
        driver.find_element_by_id("kw").send_keys("selenium webdriver") 
        driver.find_element_by_id("su").click() 
        time.sleep(2) 
        driver.close() 
    #百度设置用例 
    def test_baidu_set(self): 
        driver = self.driver 
        #进入搜索设置页 
        driver.get(self.base_url + "/gaoji/preferences.html") 
        #设置每页搜索结果为100条 
        m=driver.find_element_by_name("NR") 
        m.find_element_by_xpath("//option[@value='100']").click() 
        time.sleep(2) 
        #保存设置的信息 
        driver.find_element_by_xpath("/html/body/form/div/input").click() 
        time.sleep(2) 
        driver.switch_to_alert().accept() 
    def tearDown(self): 
        self.driver.quit() 
        self.assertEqual([], self.verificationErrors) 
if __name__ == "__main__": 
    #定义一个单元测试容器 
    testunit=unittest.TestSuite() 
    #将测试用例加入到测试容器中 
    testunit.addTest(Baidu("test_baidu_search")) 
    testunit.addTest(Baidu("test_baidu_set")) 
    #定义个报告存放路径，支持相对路径 
    filename = 'D:\\selenium_python\\report\\result.html' 
    fp = file(filename, 'wb') 
    #定义测试报告 
    runner =HTMLTestRunner.HTMLTestRunner( stream=fp, title=u'百度搜索测试报告', description=u'用例执行情况：') 
    #运行测试用例 
    runner.run(testunit) 
```

2. 用例分别写入到单独的文件

```
#coding=utf-8
import unittest
#这里需要导入测试文件
import baidu,youdao
testunit=unittest.TestSuite()
#将测试用例加入到测试容器(套件)中
//makeSuite 用于生产 testsuit 对象的实例，把所有的测试用例组装成TestSuite，最后把 TestSuite 传给TestRunner进行执行
testunit.addTest(unittest.makeSuite(baidu.Baidu))
testunit.addTest(unittest.makeSuite(youdao.Youdao))
#执行测试套件
runner = unittest.TextTestRunner()
runner.run(testunit)
```

3. 添加注释

```
……
#百度搜索用例
def test_baidu_search(self):
    u"""百度搜索"""
    driver = self.driver
    driver.get(self.base_url + "/")
……
```
