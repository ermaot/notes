- 模板引擎的作用就是读取并执行模板中的特殊语法标记，并根据传入的数据将变量替换为实际值，输出最终的HTML页面，这个过程被称为渲染（rendering）。
- Flask默认使用的模板引擎是Jinja2，它是一个功能齐全的Python模板引擎，除了设置变量，还允许我们在模板中添加if判断，执行for迭代，调用函数等，以各种方式控制模板的输出。
- 对于Jinja2来说，模板可以是任何格式的纯文本文件，比如HTML、XML、CSV、LaTeX等
## 模板基本用法
#### 模板样例
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>{{ user.username }}'s Watchlist</title>
</head>
<body>
<a href="{{ url_for('index') }}">&larr; Return</a>
<h2>{{ user.username }}</h2>
{% if user.bio %}
    <i>{{ user.bio }}</i>
{% else %}
    <i>This user has not provided a bio.</i>
{% endif %}
{# 下面是电影清单（这是注释） #}
<h5>{{ user.username }}'s Watchlist ({{ movies|length }}):</h5>
<ul>
    {% for movie in movies %}
        <li>{{ movie.name }} - {{ movie.year }}</li>
    {% endfor %}
</ul>
</body>
</html>
```
- &larr; 是一个HTML实体
- [链接](https://dev.w3.org/html5/html-author/charref)可以查看所有HTML实体
- 基本语法
1. if判断
```
{% ... %}
```
2. 表达式
```
{{ ... }}
```
3. 注释
```
{# ... #}
```
4. 支持使用.获取变量属性，比如user.usename等同于user['username']
5. jinja并不支持所有Python语法，应该适度使用模板，仅把和输出控制有关的逻辑操作放到模板中
6. Jinja2允许你在模板中使用大部分Python对象，比如字符串、列表、字典、元组、整型、浮点型、布尔值。它支持基本的运算符号（+、-、*、/等）、比较符号（比如==、！=等）、逻辑符号（and、or、not和括号）以及in、is、None和布尔值（True、False）
7. Jinja2提供了多种控制结构来控制模板的输出，其中for和if是最常用的两种
8. 常用的Jinja2 for循环特殊变量

变量名|说明
---|---
loop.index|当前迭代数(从1开始计数)
loop.index0|当前迭代数(从0开始计数)
loop.revindex|当前反向迭代数(从1开始计数)
loop.revindex0|当前反向迭代数(从0开始计数)
loop.first|如果是第一个元素,则为True
loop.last|如果是最后一个元素,则为True
loop.previte|上一个迭代的条目
loop.nextitem|下一个迭代的条目
loop.length|序列包含的元素数量

更加详细的for，查看[链接](https://jinja.palletsprojects.com/en/2.10.x/templates/#for)

#### 模板渲染
```
from flask import Flask, render_template
...
@app.route('/watchlist')
def watchlist():
    return render_template('watchlist.html', user=user, movies=movies)
```
- 除了render_template（）函数，Flask还提供了一个render_template_string（）函数用来渲染模板字符串
- 传入Jinja2中的变量值可以是字符串、列表和字典，也可以是函数、类和类实例
- 你想传入函数在模板中调用，那么需要传入函数对象本身，而不是函数调用（函数的返回值），所以仅写出函数名称即可。当把函数传入模板后，我们可以像在Python脚本中一样通过添加括号的方式调用，你也可以在括号中传入参数
## 模板辅助工具
#### 上下文
- 模板上下文包含了很多变量，其中包括我们调用render_template（）函数时手动传入的变量以及Flask默认传入的变量
- 除了渲染时传入变量，也可以在模板中定义变量，使用set标签
```
{% set navigation = [('/', 'Home'), ('/about', 'About')] %}
```
- 你也可以将一部分模板数据定义为变量，使用set和endset标签声明开始和结束
```
{% set navigation %}
    <li><a href="/">Home</a>
    <li><a href="/about">About</a>
{% endset %}
```
- 内置上下文变量
标准模板全局变量

变量|说明
---|---
config|当前的配置对象
request|当前的请求对象,在已激活的请求环境下可用
session|当前的会话对象,在已激活的请求环境下可用
g|与请求绑定的全局变量,在已激活的请求环境下可用
- 自定义上下文
1. Flask提供了一个app.context_processor装饰器，可以用来注册模板上下文处理函数，它可
以完成统一传入变量的工作。模板上下文处理函数需要返回一个包含变量键值对的字典
```
@app.context_processor
def inject_foo():
    foo = 'I am foo.'
    return dict(foo=foo)  # 等同于return {'foo': foo}
```
2. 当我们调用render_template（）函数渲染任意一个模板时，所有使用app.context_processor装饰器注册的模板上下文处理函数（包括Flask内置的上下文处理函数）都会被执行，这些函数的返回值会被添加到模板中，因此我们可以在模板中直接使用foo变量
3. 传函数
```
def inject_foo():
    foo = 'I am foo.'
    return dict(foo=foo)
app.context_processor(inject_foo)
```
```
app.context_processor(lambda: dict(foo='I am foo.'))
```
#### 内置全局函数
- Jinja2内置模板全局函数

函数|说明
---|---
range([start, ]stop, step])|和 Python中的 range用法相同
lipsum(n=5, html=True,min=20,max=100)|生成随机文本( lorem ipsum),可以在测试时用来填充页面.默认生成5段HTML文本,每段包含20-100个单词
dict( *items)|和 Python中的 dict()用法相同
- flask内置全局函数

函数|说明
---|---
url_for()|用于生成url
get_flashed_messages()|用于获取flash消息的函数

- Flask除了把g、session、config、request对象注册为上下文变量，也将它们设为全局变量，可以全局使用

#### 自定义全局函数
- 可以使用app.template_global装饰器直接将函数注册为模板全局函数
```
@app.template_global()
def bar():
    return 'I am bar.'
```

#### 过滤器
- 首字母大写，类似于name.title（）
```
{{ name|title }}
```
- 获取字符串长度，类似于len()
```
{{ movies|length }}
```
- 内置过滤器列表

过滤器|说明
---|---
default( (value, default_value=u", boolean= False)|设置默认值,默认值作为参数传入,别名为d
escape(s)|转义HTML文本,别名为e
first(seq)|返回序列的第一个元素
last(seq)|返回序列的最后一个元素
length(object)|返回变量的长度
random(seq)|返回序列中的随机元素
safe(value)|将变量值标记为安全,避免转义
trim(value)|清除变量值前后的空格
max( value, case sensitive= False, attribute==None)|返回序列中的最大值
min(value, case sensitive= False, attribute= None)|返回序列中的最小值
unique(value, case sensitive=False. attribute= None)|返回序列中的不重复的值
striptags(value)|清除变量值内的HTML标签
urie( (value, trim url limit=None, nofollow=False, target=None, rel-None)|将URL文本转换为可单击的HTML链接
wordcount(s)|计算单词数量
tojson(value, indent=None)|将变量值转换为JSON格式
truncate(s, length=255. killwords=False,end="...",leeway=None)截断字符串,常用于显示文章摘要, length参数设置截断的长度, killwords参数设置是否截断单词,end参数设置结尾的符号
- 更多过滤器查看[链接](http://jinja.pocoo.org/docs/2.10/templates/#builtin-filters)
- 在使用过滤器时，列表中过滤器函数的第一个参数表示被过滤的变量值（value）或字符串（s），即竖线符号左侧的值，其他的参数可以通过添加括号传入
```
<h1>Hello, {{ name|default('陌生人')|title }}!</h1>
```
#### 自定义过滤器
注册自定义过滤器
```
from flask import Markup
@app.template_filter()
def musical(s):
    return s + Markup(' &#9835;')
```
使用
```
{{ name|musical }}
```
#### 测试器
- 在Jinja2中，测试器（Test）是一些用来测试变量或表达式，返回布尔值（True或False）的特殊函数
```
{% if age is number %}　　
{{ age * 365 }}
{% else %}
　　无效的数字。
{% endif %}
```
测试器|说明
---|---
callable(object)|判断对象是否可被调用
defined(value)|判断变量是否已定义
undefined (value)|判断变量是否未定义
none(value)|判断变量是否为None
number(value)|判断变量是否是数字
string( value)|判断变量是否是字符串
sequence(value)|判断变量是否是序列,比如字符串、列表、元组
iterable(value)|判断变量是否可迭代
mapping(value)|判断变量是否是匹配对象,比如字典
sameas(value, other)|判断变量与 other是否指向相同的内存地址

- 全部内置测试器查看[链接](http://jinja.pocoo.org/docs/2.10/templates/#list-of-builtin-tests)
- 自定义测试器：以使用Flask提供的app.template_test（）装饰器来注册一个自定义测试器
```
@app.template_test()
def baz(n):
    if n == 'baz':
        return True
    return False
```

#### 模板环境对象
- 在Jinja2中，渲染行为由jinja2.Enviroment类控制，所有的配置选项、上下文变量、全局函数、过滤器和测试器都存储在Enviroment实例上。
- 当与Flask结合后，我们并不单独创建Enviroment对象，而是使用Flask创建的Enviroment对象，它存储在app.jinja_env属性上
- 添加自定义全局对象
```
def bar():
    return 'I am bar.'
foo = 'I am foo.'
app.jinja_env.globals['bar'] = bar
app.jinja_env.globals['foo'] = foo
```
和app.template_global（）装饰器不同，直接操作globals字典允许传入任意Python对象，而不仅仅是函数
- 添加自定义过滤器
```
def smiling(s):
    return s + ' :)'
app.jinja_env.filters['smiling'] = smiling
```
- 添加自定义测试器
```
def baz(n):
    if n == 'baz':
        return True
    return False
app.jinja_env.tests['baz'] = baz
```
- 查看[链接](http://jinja.pocoo.org/docs/latest/api/#jinja2.Environment)Enviroment类的所有属性及用法说明。

## 模板结构组织
#### 局部模板
- 我们还会用到另一类非独立模板，这类模板通常被称为局部模板或次模板，因为它们仅包含部分代码，所以我们不会在视图函数中直接渲染它，而是插入到其他独立模板中
- 当多个独立模板中都会使用同一块HTML代码时，我们可以把这部分代码抽离出来，存储到局部模板中
- 为了和普通模板区分开，局部模板的命名通常以一个下划线开始
```
{% include '_banner.html' %}
```
#### 宏
- 宏（macro）是Jinja2提供的一个非常有用的特性，它类似Python中的函数。
- 使用宏可以把一部分模板代码封装到宏里，使用传递的参数来构建内容，最后返回构建后的内容。
- 在功能上，它和局部模板类似，都是为了方便代码块的重用
- 为了便于管理，我们可以把宏存储在单独的文件中，这个文件通常命名为macros.html或_macors.html。
- 在创建宏时，我们使用macro和endmacro标签声明宏的开始和结束
定义宏
```
{% macro qux(amount=1) %}
    {% if amount == 1 %}
        I am qux.
    {% elif amount > 1 %}
        We are quxs.
    {% endif %}
{% endmacro %}
```
使用宏
```
{% from 'macros.html' import qux %}
...
{{ qux(amount=5) }}
```
#### 模板继承
- Jinja2的模板继承允许你定义一个基模板，把网页上的导航栏、页脚等通用内容放在基模板中，而每一个继承基模板的子模板在被渲染时都会自动包含这些部分
- 基模板存储了程序页面的固定部分，通常被命名为base.html或layout.html
- 当子模板继承基模板后，子模板会自动包含基模板的内容和结构。为了能够让子模板方便地覆盖或插入内容到基模板中，我们需要在基模板中定义块（block），在子模板中可以通过定义同名的块来执行继承操作
- 块的开始和结束分别使用block和endblock标签声明，而且块之间可以嵌套
```
<!DOCTYPE html>
<html>
<head>
    {% block head %}
        <meta charset="utf-8">
        <title>{% block title %}Template - HelloFlask{% endblock %}</title>
　　　　{% block styles %}{% endblock %}
    {% endblock %}
</head>
<body>
<nav>
    <ul><li><a href="{{ url_for('index') }}">Home</a></li></ul>
</nav>
<main>
    {% block content %}{% endblock %}
</main>
<footer>
    {% block footer %}
        ...
    {% endblock %}
</footer>
{% block scripts %}{% endblock %}
</body>
</html>
```
- 。在这个基模板中，我们创建了六个块：head、title、styles、content、footer和scripts，分别用来划分不同的代码
- 使用extends标签声明扩展基模板
```
{% extends 'base.html' %}
{% from 'macros.html' import qux %}
{% block content %}
{% set name='baz' %}
<h1>Template</h1>
<ul>
    <li><a href="{{ url_for('watchlist') }}">Watchlist</a></li>
    <li>Filter: {{ foo|musical }}</li>
    <li>Global: {{ bar() }}</li>
    <li>Test: {% if name is baz %}I am baz.{% endif %}</li>
    <li>Macro: {{ qux(amount=5) }}</li>
</ul>
{% endblock %}
```
- extends必须是子模板的第一个标签
- 在子模板中，我们可以对父模板中的块执行两种操作
1. 覆盖内容：当在子模板里创建同名的块时，会使用子块的内容覆盖父块的内容
2. 追加内容：如果想要向基模板中的块追加内容，需要使用Jinja2提供的super（）函数进行声明，这会向父块添加内容
```
{% block styles %}
{{ super() }}
<style>
    .foo {
        color: red;
    }
</style>
{% endblock %}
```


## 模板进阶实战
#### 空白控制
- 如果想在渲染时自动去掉空行，可以在定界符内侧添加减号。比如，{%-endfor%}会移除该语句前的空白，同理，在右边的定界符内侧添加减号将移除该语句后的空白
```
{% if user.bio -%}
    <i>{{ user.bio }}</i>
{% else -%}    <i>This user has not provided a bio.</i>
{%- endif %}
```
- 我们也可以使用模板环境对象提供的trim_blocks和lstrip_blocks属性设置，前者用来删除Jinja2语句后的第一个空行，后者则用来删除Jinja2语句所在行之前的空格和制表符（tabs）
```
app.jinja_env.trim_blocks = True
app.jinja_env.lstrip_blocks = True
```
- 宏内的空白控制行为不受trim_blocks和lstrip_blocks属性控制，我们需要手动设置
#### 加载静态文件
- 为了在HTML文件中引用静态文件，我们需要使用url_for（）函数获取静态文件的URL。Flask内置了用于获取静态文件的视图函数，端点值为static
```
<img src="{{ url_for('static', filename='avatar.jpg') }}" width="50">
```
- 如果你想使用其他文件夹来存储静态文件，可以在实例化Flask类时使用static_folder参数指定，静态文件的URL路径中的static也会自动跟随文件夹名称变化。
- 在实例化Flask类时使用static_url_path参数则可以自定义静态文件的URL路径
#### 添加Favicon
- favicon.ico文件指的是Favicon（favorite icon，收藏夹头像/网站头像），又称为shortcut icon、tab icon、website icon或是bookmarkicon。顾名思义，这是一个在浏览器标签页、地址栏和书签收藏夹等处显示的小图标，作为网站的特殊标记
- 要想为Web项目添加Favicon，你要先有一个Favicon文件，并放置==到static目录下==
```
<link rel="icon" type="image/x-icon" href="{{ url_for('static', filename 'favicon.ico') } }">
```
#### 使用CSS框架
- CSS框架内置了大量可以直接使用的CSS样式类和JavaScript函数，使用它们可以非常快速地让程序页面变得美观和易用，同时我们也可以定义自己的CSS文件来进行补充和调整
- Bootstrap所依赖的jQuery（https://jquery.com/）和Popper.js（https://popper.js.org/）需要单独下载，这三个JavaScript文件在引入时要按照jQuery→Popper.js→Boostrap的顺序引入
#### 使用宏加载静态资源
　template/templates/macros.html
```
{% macro static_file(type, filename_or_url, local=True) %}
    {% if local %}
        {% set filename_or_url = url_for('static', filename=filename_or_url) %}
    {% endif %}
    {% if type == 'css' %}
        <link rel="stylesheet" href="{{ filename_or_url }}" type="text/css">
    {% elif type == 'js' %}
        <script type="text/javascript" src="{{ filename_or_url }}"></script>
    {% elif type == 'icon' %}
        <link rel="icon" href="{{ filename_or_url }}">
    {% endif %}
{% endmacro %}
```
使用宏加载静态资源

```
static_file('css', 'css/bootstrap.min.css')
```
#### 消息闪现
- Flask提供了一个非常有用的flash（）函数，它可以用来“闪现”需要显示给用户的消息
- 使用功能flash（）函数发送的消息会存储在session中，我们需要在模板中使用全局函数get_flashed_messages（）获取消息并将其显示出来
- 样例
```
from flask import Flask, render_template, flash
app = Flask(__name__)
app.secret_key = 'secret string'
@app.route('/flash')
def just_flash():
    flash('I am flash, who is looking for me?')
    return redirect(url_for('index'))
```
- Jinja2内部使用Unicode，所以你需要向模板传递Unicode对象或只包含ASCII字符的字符串。
- 在Python 2.x中，如果字符串包含中文（或任何非ASCII字符），那么需要在字符串前添加u前缀
```
<main>
    {% for message in get_flashed_messages() %}
        <div class="alert">{{ message }}</div>
    {% endfor %}
    {% block content %}{% endblock %}
</main>
```
#### 自定义错误页面
- template/templates/errors/404.html：404页面模板
```
{% extends 'base.html' %}
{% block title %}404 - Page Not Found{% endblock %}
{% block content %}
<h1>Page Not Found</h1>
<p>You are lost...</p>
{% endblock %}
```
- 错误处理函数需要附加app.errorhandler（）装饰器，并传入错误状态码作为参数
```
from flask import Flask, render_template
...
@app.errorhandler(404)
def page_not_found(e):
    return render_template('errors/404.html'), 404
```

