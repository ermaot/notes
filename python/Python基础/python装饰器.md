==装饰器是重要的，必须掌握==<br>
来自https://www.cnblogs.com/cicaday/p/python-decorator.html<br>
递进理解：<br>

```
def say_hello():
    print "hello!"
    
def say_goodbye():
    print "hello!"  # bug here

if __name__ == '__main__':
    say_hello()
    say_goodbye()
```
如果函数出错，需要打印函数名称，以定位问题，如何做？
#### 1. 菜鸟级

```
def say_hello():
    print "[DEBUG]: enter say_hello()"
    print "hello!"

def say_goodbye():
    print "[DEBUG]: enter say_goodbye()"
    print "hello!"

if __name__ == '__main__':
    say_hello()
    say_goodbye()
```
#### 2. 进阶菜鸟

```
def debug():
    import inspect
    caller_name = inspect.stack()[1][3]
    print "[DEBUG]: enter {}()".format(caller_name)   

def say_hello():
    debug()
    print "hello!"

def say_goodbye():
    debug()
    print "goodbye!"

if __name__ == '__main__':
    say_hello()
    say_goodbye()
```

### 3. 装饰器写法

```
def debug(func):
    def wrapper():
        print "[DEBUG]: enter {}()".format(func.__name__)
        return func()
    return wrapper

@debug
def say_hello():
    print "hello!"
```
- 装饰器本质上是一个Python函数
- 它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。
- 它经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景。
- 装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。
