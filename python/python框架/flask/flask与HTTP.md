## flask请求过程
![flask请求过程](CDE40724B8BA468397CDAF27D9566080)

## flask的HTTP原理
#### request
- 请求解析和响应封装实际上大部分是由Werkzeug完成的，Flask子类化Werkzeug的请求（Request）和响应（Response）对象并添加了和程序相关的特定功能
- request属性
比如http://helloflask.com/hello?name=Grey

属性| 值
---|---
path|u"/hello"
full_path|u"/hello?name=Grey"
host|u"helloflask.com"
host_url|u"http://helloflask.com/"
base_url|u"http://helloflask.com/hello"
url|u"http://helloflask.com/hello?name=Grey"
url_root|u"http://helloflask.com/"
args|Werkzeug的 ImmutableMultiDict对象.存储解析后的查询字符串,可通过字典方式获取键值.如果你想获取未解析的原生查询字符串,可以使用 query_string属性
blueprint|当前蓝本的名称
cookies|一个包含所有随请求提交的 cookies的字典
data|包含字符串形式的请求数据
endpoint|与当前请求相匹配的端点值
files|Werkzeug的 Multidict对象,包含所有上传文件,可以使用字典的形式获取文件.使用的键为文件 Input标签中的name属性值,对应的值为 Werkzeug的FileStorage对象,可以调用 saved方法并传入保存路径来保存文件
form|Werkzeug的 ImmutableMulti Dict对象.与files类似,包含解析后的表单数据表单字段值通过 Input标签的name属性值作为键获取
values|Werkzeug的 Combined MultiDict对象,结合了args和form属性的值
get_data(cache=True, as_text=False, parse_from_data= False)|获取请求中的数据,默认读取为字节字符串( bytestring),将 as text设为True则返回值将是解码后的 unicode字符串
get_json(self, force=False,silent=False, cache=True)|作为JSON解析并返回数据,如果MIME类型不是JsON,返回None(除非force设为True);解析出错则抛出 Werkzeug提供的 BadRequest异常(如果未开启调试模式,则返回400错误响应,后面会详细介绍),如果 silent设为True则返回None; cache设置是否缓存解析后的JSON数据
headers|一个 Werkzeug的 Environ Headers对象,包含首部字段,可以以字典的形式操作
is_json|通过MIME类型判断是否为JsON数据,返回布尔值
json|包含解析后的JSON数据,内部调用 get son0,可通过字典的方式获取键值
method|请求的HTTP方法
referrer|请求发起的源URL.即 referer
scheme|请求的URL模式(http或htps)
user_agent|用户代理( User Agent,UA),包含了用户的客户端类型,操作系统类型等信息

## flask处理http请求
#### 路由匹配
- 程序实例中存储了一个路由表（app.url_map），其中定义了URL规则和视图函数的映射关系
```
Map([<Rule '/' (HEAD, OPTIONS, GET) -> hello_world>,
 <Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
 <Rule '/hello/<name>' (HEAD, OPTIONS, GET) -> greet>])

```
- 当请求发来后，Flask会根据请求报文中的URL（path部分）来尝试与这个表中的所有URL规则进行匹配，调用匹配成功的视图函数。
- 如果没有找到匹配的URL规则，说明程序中没有处理这个URL的视图函数，Flask会自动返回404错误响应（Not Found，表示资源未找到）
- flask routes命令可以查看程序中定义的所有路由，这个列表由app.url_map解析得到
#### 设置监听HTTP方法
- GET是最常用的HTTP方法，所以视图函数默认监听的方法类型就是GET，HEAD、OPTIONS方法的请求由Flask处理
- 而像DELETE、PUT等方法一般不会在程序中实现，在构建Web API时才会用到这些方法
```
@app.route('/hello', methods=['GET', 'POST'])
def hello():
    return '<h1>Hello, Flask!</h1>'
```
- 当某个请求的方法不符合要求时，Flask会自动返回一个405错误响应（Method Not Allowed，表示请求方法不允许）

#### 请求参数类型
类型|说明
---|---
string|不包含斜线的字符串(默认值)
int|整型
float|浮点数
path|包含斜线的字符串。static路由的URL规则中的 filename变量就使用了这个转换器
any|匹配一系列给定值中的一个元素，类似于枚举类型
UUID|字符串
```
@app.route('/colors/<any(blue, white, red):color>')
def three_colors(color):
    return '<p>Love is patient and kind.<p>'
```
#### 请求钩子
钩子|说明
---|---
before_first_request|注册一个函数,在处理第一个请求前运行
before_request|注册一个函数,在处理每个请求前运行
after_request|注册一个函数,如果没有未处理的异常抛出,会在每个请求结束后运行
teardown_request|注册一个函数,即使有未处理的异常抛出,会在每个请求结束后运行.如果发生异常,会传入异常对象作为参数到注册的函数中
after_this_request|在视图函数内注册一个函数,会在这个请求结束后运行

样例：
```
from flask import Flask,session,redirect,after_this_request
import click
from flask import request
app = Flask(__name__)

##after_request
@app.after_request
def process_response(response):
    print(2222)
    return response

## before_request
@app.before_request
def process_request():
    if request.path=="/login":
        return None
    if not session.get("user_info"):
        return None
    return None

@app.route("/")
def index():
    name = request.args.get("name","Flask")
    ##注意写法和用法，在“视图函数内”，且不需要加app
    @after_this_request
    def test(response):
        print(3333)
        return response
    return "<h1>Hello World%s</h1>"%name

@app.cli.command()
def hello():
    click.echo('Hello, Human!')
```

## flask的http响应

#### 响应报文
- 响应报文主要由协议版本、状态码（status code）、原因短语（reason phrase）、响应首部和响应主体组成

组成说明|响应报文内容
---|---
报文首部:状态行(协议、状态码、原因短语)|Http/1.1 200 Ok
报文首部:各种首部字段|Content-Type: text/html; charset=utf-8<p>Content-Length: 22<p>Server: Werkzeug/0. 12.2 Python/2.7. 13<p>Date: Thu, 03 Aug 201705: 05: 54 GMT
空行 |
报文主体|<p>Hello, Human</p>

常见状态码
类型|状态码|原因短语(用于解释状态码)|说明
---|---|---|---
成功|200|OK|请求被正常处理
成功|201|Created|请求被处理,并创建了一个新资源
成功|204|No Content|青求处理成功,但无内容返回
重定向|301|Moved Permanently|永久重定向
重定向|302|found|临时性重定向
重定向|304|Not modified|请求的资源未被修改,重定向到缓存的资源
客户端错误|400|Bad Request|表示请求无效,即请求报文中存在错误
客户端错误|401|unauthorized|类似403,表示请求的资源需要获取授权信息,在浏览器中会弹出认证弹窗
客户端错误|403|Forbidden|表示请求的资源被服务器拒绝访问
客户端错误|404|Not Found|表示服务器上无法找到请求的资源或URL无效
服务器端错误|500|Internal Server Error|服务器内部发生错误

#### 改变响应状态码
```
@app.route('/hello/<name>')
def greet(name):
    print(app.url_map)
    return 'Hello %s!' % name ,201
```
```
//这么写有点问题
@app.route('/hello/<name>')
def greet(name):
    # print(app.url_map)
    return 'Hello %s!' % name ,302, {'Location', 'http://www.example.com'}
```

#### 重定向
```
from flask import Flask, redirect
# ...
@app.route('/hello')
def hello():
    return redirect('http://www.example.com')
```
```
from flask import Flask, redirect, url_for 
...
@app.route('/hi')
def hi():
    ...
    return redirect(url_for('hello'))  # 重定向到/hello
@app.route('/hello')
def hello():
    ...
```
redirect可以加code参数
```
return redirect('http://{0}/{1}'.format('www.newweb.net', page_name), code=301)
```

#### 404
```
from flask import Flask, abort
...
@app.route('/404')
def not_found():
    abort(404)
```
abort不需要加return

## flask 上下文
#### 上下文全局变量
- Flask会在每个请求产生后自动激活当前请求的上下文，激活请求上下文后，request被临时设为全局可访问
- 当每个请求结束后，Flask就销毁对应的请求上下文
- 请求对象在各自的线程内是全局的。Flask通过本地线程（thread local）技术将请求对象在特定的线程和请求中全局可访问
flask的上下文变量

变量名|上下文类别|说明
---|---|---
current_app|程序上下文|指向处理请求的当前程序实例
g|程序上下文|替代 Python的全局变量用法,确保仅在当前请求中可用<p>用于存储全局数据,每次请求都会重设
reques|请求上下文|封装客户端发出的请求报文数据
session|请求上下文|用于记住请求之间的数据,通过签名的 Cookie实现

- 程序也会有多个程序实例的情况，为了能获取对应的程序实例，而不是固定的某一个程序实例，我们就需要使用current_app变量
- g存储在程序上下文中，而程序上下文会随着每一个请求的进入而激活，随着每一个请求的处理完毕而销毁，所以每次请求都会重设这个值
- g也支持使用类似字典的get（）、pop（）以及setdefault（）方法进行操作
```
from flask import g
@app.before_request
@app.route('/hello')
def get_name():
    g.name = request.args.get('name')
    return 'Hello %s!' % g.name
```

#### 激活上下文
- 在下面这些情况下，Flask会自动帮我们激活程序上下文：
1. 当我们使用flask run命令启动程序时。
2. 使用旧的app.run（）方法启动程序时。
3. 执行使用@app.cli.command（）装饰器注册的flask命令时。
4. 使用flask shell命令启动Python Shell时
- 当请求进入时，Flask会自动激活请求上下文，这时我们可以使用request和session变量。
- 另外，当请求上下文被激活时，程序上下文也被自动激活。
- 当请求处理完毕后，请求上下文和程序上下文也会自动销毁，也就是说，在请求处理时这两者拥有相同的生命周期
- 依赖于上下文的有url_for（）、jsonify（）等函数
- 手动激活上下文
1. 程序上下文对象使用app.app_context（）获取
```
>>> from app import app
>>> from flask import current_app
>>> with app.app_context():
...     current_app.name

```
2. 显式地使用push（）方法推送（激活）上下文，在执行完相关操作时使用pop（）方法销毁上下文
```
>>> from app import app
>>> from flask import current_app
>>> app_ctx = app.app_context()
>>> app_ctx.push()
>>> current_app.name
'app'
>>> app_ctx.pop()

```
3. 请求上下文可以通过test_request_context（）方法临时创建
```
>>> from app import app
>>> from flask import request
>>> with app.test_request_context('/hello'):
...     request.method
'GET'
```

#### 上下文钩子
Flask为上下文提供了一个teardown_appcontext钩子，使用它注册的回调函数会在程序上下文被销毁时调用，而且通常也会在请求上下文被销毁时调用
```
@app.teardown_appcontext
def teardown_db(exception):
    ...
    db.close()
```
