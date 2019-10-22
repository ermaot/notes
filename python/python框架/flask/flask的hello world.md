## 通过pycharm创建一个新flask项目
![image](pic/flask%E7%9A%84hello%20world.png)
#### 最小flask程序
app.py
```
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()

```
#### 注册路由
在一个Web应用里，客户端和服务器上的Flask程序的交互可以简单概括为以下几步：
1. 用户在浏览器输入URL访问某个资源。
2. Flask接收用户请求并分析请求的URL。
3. 为这个URL找到对应的处理函数。
4. 执行函数并生成响应，返回给浏览器。
5. 浏览器接收并解析响应，将信息显示在页面中
大部分由flask完成，我们要做的只是==建立处理请求的函数，并为其定义对应的URL规则==
- 为函数附加app.route（）装饰器，并传入URL规则作为参数，就可以让URL与函数建立关联。这个过程我们称为注册路由（route）
- 路由负责管理URL和函数之间的映射，而这个函数则被称为视图函数（view function）
###### 样例：
```
@app.route('/')
def hello_world():
    return 'Hello World!'
```
###### 为视图绑定多个URL
```
@app.route('/hi')
@app.route('/hello')
def say_hello():
    return '<h1>Hello, Flask!</h1>'
```
###### 动态URL
```
@app.route('/greet/<name>')
def greet(name):
    return '<h1>Hello, %s!</h1>' % name
```
###### 路由参数指定类型
```
@app.route('/greet/<string:name>')
def greet(name):
    return '<h1>Hello, %s!</h1>' % name
```
- 未指定类型，则都是string
- 可以有int、float、uuid、path、string等几种类型

###### 路由指定默认值
```
@app.route('/greet', defaults={'name': 'Programmer'})
@app.route('/greet/<name>')
def greet(name):
    return '<h1>Hello, %s!</h1>' % name
```
等同于
```
@app.route('/greet')
@app.route('/greet/<name>')
def greet(name='Programmer'):
    return '<h1>Hello, %s!</h1>' % name
```


## 启动flask服务器
#### flask run
```
flask run
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

```
- 确保执行命令前激活了虚拟环境（pipenv shell），否则需要使用pipenv run flask run命令启动开发服务器
- flask --help
```
flask --help
Usage: flask [OPTIONS] COMMAND [ARGS]...

  A general utility script for Flask applications.

  Provides commands from Flask, extensions, and the application. Loads the
  application defined in the FLASK_APP environment variable, or from a
  wsgi.py file. Setting the FLASK_ENV environment variable to 'development'
  will enable debug mode.

    > set FLASK_APP=hello.py
    > set FLASK_ENV=development
    > flask run

Options:
  --version  Show the flask version
  --help     Show this message and exit.

Commands:
  routes  Show the routes for the app.
  run     Run a development server.
  shell   Run a shell in the app context.

```
#### python启动
```
# python app.py
```

#### 自动发现程序实例
- 旧的启动开发服务器的方式是使用app.run（）方法，目前已不推荐使用（deprecated）
- Flask会自动探测程序实例，自动探测存在下面这些规则：
1. 从当前目录寻找app.py和wsgi.py模块，并从中寻找名为app或application的程序实例
2. 从环境变量FLASK_APP对应的值寻找名为app或application的程序实例
- 如果主程序名为非app.py，则需要设置FLASK_APP环境变量
```
linux
# export FLASK_APP=hello
或者windows
 set FLASK_APP=hello
```

#### 管理环境变量
- 如果安装了python-dotenv，那么在使用flask  run或其他命令时会使用它自动从.flaskenv文件和.env文件中加载环境变量
- 在项目根目录下分别创建两个文件：.env和.flaskenv。.flaskenv用来存储和Flask相关的公开环境变量，比如FLASK_APP；而.env用来存储包含敏感信息的环境变量，比如后面我们会用来配置Email服务器的账户名与密码
- 在.flaskenv或.env文件中，环境变量使用键值对的形式定义，每行一个，以#开头
```
SOME_VAR=1
# 这是注释
FOO="BAR"
```
的为注释
#### 启动选项
- 使得服务器外部可见
```
# flask run --host=0.0.0.0
```
- 改变端口
```
# flask run --port=8000
```
- 执行flask run命令时的host和port选项也可以通过环境变量FLASK_RUN_HOST和FLASK_RUN_PORT设置

- 设置运行环境
开发环境（development enviroment）和生产环境（production enviroment）
```
写入.flaskenv中
FLASK_ENV=development
```
或者
```
set FLASK_ENV=development
然后执行：
flask run
 * Environment: development
 * Debug mode: on
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 272-343-756
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)


```
- 以通过FLASK_DEBUG环境变量设置，设为1则开启，设为0则关闭


## 调试
#### 调试器
- Werkzeug提供的调试器非常强大
1. 调试器允许你在错误页面上执行Python代码
2. 单击错误信息右侧的命令行图标，会弹出窗口要求输入PIN码，也就是在启动服务器时命令行窗口打印出的调试器PIN码（Debugger PIN）。
3. 输入PIN码后，我们可以单击错误堆栈的某个节点右侧的命令行界面图标，这会打开一个包含代码执行上下文信息的Python Shell，我们可以利用它来进行调试
#### 重载器
- 重载器的作用就是监测文件变动，然后重新启动开发服务器
- 默认会使用Werkzeug内置的stat重载器，它的缺点是耗电较严重，而且准确性一般
- 使用watchdog，只在开发时才会用到，所以我们在安装命令后添加了一个--dev选项。在Pipfile文件中，这个包会被添加到dev-packages部分
```
# pipenv install watchdog --dev
```
```
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]
watchdog = "*"

[packages]
flask = "*"

[requires]
python_version = "2.7"

```

- 按下Crtl+F5或Shift+F5执行硬重载（hard reload）

## flask shell
- 使用flask shell进入flask的交互界面
- ，执行这个命令前我们要确保程序实例可以被正常找到
```
> flask shell
Python 2.7.12 (v2.7.12:d33e0cf91556, Jun 27 2016, 15:19:22) [MSC v.1500 32 bit (Intel)] on win32
App: app [development]
Instance: d:\project\docsreal\instance

```
```
>>> app
<Flask 'app'>
>>> app.name
'app'

```
## flask扩展
此Flask官方推荐以flask_something形式导入扩展

## 项目配置
- 在Flask中，配置变量就是一些大写形式的Python变量，你也可以称之为配置参数或配置键。使用统一的配置变量可以避免在程序中以硬编码（hard coded）的形式设置程序
- Flask提供的配置，扩展提供的配置，程序特定的配置
- ，这些配置变量都==通过Flask对象的app.config属性作为统一的接口来设置和获取==，它指向的Config类实际上是字典的子类，所以你可以像操作其他字典一样操作它
```
app.config['ADMIN_NAME'] = 'Peter'
```
- 用update（）方法则可以一次加载多个值
```
app.config.update(
    TESTING=True,
    SECRET_KEY='_5#yF4Q8z\n\xec]/'
)
```
- 还可以把配置变量存储在单独的Python脚本、JSON格式的文件或是Python类中，config对象提供了相应的方法来导入配置

## url
- 使用Flask提供的url_for（）函数获取URL，增加灵活性
```
……
@app.route('/')
def hello_world():
    return 'Hello World!' + url_for('greet',name='Jack')
# 此处会在web上输出Hello World!/hello/Jack


@app.route('/hello/<name>')
def greet(name):
    return 'Hello %s!' % name
……
```
- 我们使用url_for（）函数生成的URL是相对URL（即内部URL）
- 如果你想要生成供外部使用的绝对URL，将_external参数设为True
```
url_for('.static',_external=True,filename='pic/test.png')
```
## flask 命令
- 我们可以自定义flask命令
- 创建任意一个函数，并为其添加app.cli.command（）装饰器，我们就可以注册一个flask命令
```
@app.cli.command()
def hello():
    click.echo('Hello, Human!')
    print('Hello, Human2!')
```
- 自定义命令使用
```
>flask hello
Hello, Human!
Hello, Human2!

```
```
>flask hello --help
Usage: flask hello [OPTIONS]

Options:
  --help  Show this message and exit.

```
## 模板与静态文件
- 默认情况下，模板文件存放在项目根目录中的templates文件夹中，
- 静态文件存放在static文件夹下
- 这两个文件夹需要和包含程序实例的模块处于同一个目录下

## Flask与MVC架构
- Werkzeug中URL匹配的实现主要参考了Routes（一个URL匹配库），再往前追溯，Routes的实现又参考了Rubyon Rails
- MVC架构最初是用来设计桌面程序的，后来也被用于Web程序，应用了这种架构的Web框架有Django、Ruby on Rails等。
- 在MVC架构中，程序被分为三个组件：数据处理（Model）、用户界面（View）、交互逻辑（Controller）

