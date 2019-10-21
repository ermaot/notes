## 使用包组织代码
- 在Python中，每一个有效的Python文件（.py）都是模块。
- 每一个包含__init__.py文件的文件夹都被视作包，包让你可以使用文件夹来组织模块。
- 该__init__.py文件通常被称作构造文件，文件可以为空，也可以用来放置包的初始化代码。
- 当包或包内的模块被导入时，构造文件将被自动执行。
项目文件组织结构
```
sayhello/
|-- commands.py
|-- errors.py
|-- forms.py
|-- __init__.py
|-- models.py
|-- settings.py
|-- static
|-- templates
`-- views.py

```
#### 配置文件 settings.py
- 内容
```
import os
from sayhello import app
dev_db = 'sqlite:///' + os.path.join(os.path.dirname(app.root_path), 'data.db')
SECRET_KEY = os.getenv('SECRET_KEY', 'secret string')
SQLALCHEMY_TRACK_MODIFICATIONS = False
SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URI', dev_db)
```

```
app = Flask(__name__)
app.config.from_pyfile('settings.py')
```
#### 创建程序实例
sayhello/__init__.py：创建程序实例、初始化扩展
```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
app = Flask('sayhello')
#app=Flask(__name__.split('.')[0])
app.config.from_pyfile('settings.py')
app.jinja_env.trim_blocks = True
app.jinja_env.lstrip_blocks = True
db = SQLAlchemy(app)
from sayhello import views, errors, commands
```
- 从构造文件中导入变量时不需要注明构造文件的路径，只需要从包名称导入，比如导入在构造文件中定义的程序实例app可以使用from sayhello import app
- 给环境变量的FLASK_APP赋值

```
FLASK_APP=sayhello
```

## web程序开发流程
#### 前端开发
#### 后端开发
- 数据库建模
```
#sayhello.py

from datetime import datetime
from sayhello import db
class Message(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(200))
    name = db.Column(db.String(20))
　　timestamp = db.Column(db.DateTime, default=datetime.now, index=True)
```
 timestamp字段的默认值是datetime.now而不是datetime.now（）。前者是可调用的函数/方法对象（即名称），而后者是函数/方法调用（即动作）
 
- 创建表单类
```
#forms.py：问候表单
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, TextAreaField
from wtforms.validators import DataRequired, Length
class HelloForm(FlaskForm):
    name = StringField('Name', validators=[DataRequired(), Length(1, 20)])
    body = TextAreaField('Message', validators=[DataRequired(), Length(1, 200)])
    submit = SubmitField()
```

- 编写视图函数
```
from flask import flash, redirect, url_for, render_template
from sayhello import app, db
from sayhello.models import Message
from sayhello.forms import HelloForm
@app.route('/', methods=['GET', 'POST'])
def index():
    # 加载所有的记录
　　messages = Message.query.order_by(Message.timestamp.desc()).all()
    form = HelloForm()
    if form.validate_on_submit():
        name = form.name.data
        body = form.body.data
        message = Message(body=body, name=name)  # 实例化模型类，创建记录
        db.session.add(message)  # 添加记录到数据库会话
        db.session.commit()  # 提交会话
        flash('Your message have been sent to the world!')
        return redirect(url_for('index'))  # 重定向到index视图
    return render_template('index.html', form=form, messages=messages)
```
- 编写模板
templates/base.html
```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no"
    <title>{% block title %}Say Hello!{% endblock %}</title>
    <link rel="icon" href="{{ url_for('static', filename='favicon.ico') }}">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/bootstrap.min.css') }
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}" type=
</head>
<body>
<main class="container">
    <header>
        <h1 class="text-center display-4">
            <a href="{{ url_for('index') }}" class="text-success"><strong>Say Hello</stro
            <small style="font-size: 24px" class="text-muted">to the world</small>
        </h1>
    </header>
    {% for message in get_flashed_messages() %}
    <div class="alert alert-info">
        <button type="button" class="close" data-dismiss="alert">&times;</button>
        {{ message }}
    </div>
    {% endfor %}
    {% block content %}{% endblock %}
    <footer class="text-center">
        {% block footer %}
        ...
        <p><a id="bottom" href="#" title="Go Top">&uarr;</a></p>
        {% endblock %}
    </footer>
</main>
<script type="text/javascript" src="{{ url_for('static', filename='js/jquery-3.2.1.slim.m
<script type="text/javascript" src="{{ url_for('static', filename='js/popper.min.js') }}"
<script type="text/javascript" src="{{ url_for('static', filename='js/bootstrap.min.js') 
<script type="text/javascript" src="{{ url_for('static', filename='js/script.js') }}"></s</body>
</html>
```
index.html：渲染表单和留言列表
```
{% extends 'base.html' %}
{% from 'macros.html' import form_field %}
{% block content %}
<div class="hello-form">
    <form method="post" action="{{request.full_path}}">
        {{ form.csrf_token }}
        <div class="form-group required">
            {{ form_field(form.name, class='form-control') }}
        </div>
        <div class="form-group required">
            {{ form_field(form.body, class='form-control') }}
        </div>
        {{ form.submit(class='btn btn-secondary') }}
    </form>
</div>
<h5>{{ messages|length }} messages
    <small class="float-right">
        <a href="#bottom" title="Go Bottom">&darr;</a>
    </small>
</h5>
<div class="list-group">
    {% for message in messages %}
        <a class="list-group-item list-group-item-action flex-column">
            <div class="d-flex w-100 justify-content-between">
                <h5 class="mb-1 text-success">{{ message.name }}
                    <small class="text-muted"> #{{ loop.revindex }}</small>                </h5>
                <small>
                    {{ message.timestamp.strftime('%Y/%m/%d %H:%M') }}
                </small>
            </div>
            <p class="mb-1">{{ message.body }}</p>
        </a>
    {% endfor %}
</div>
{% endblock %}
```

## 使用bootstrap-Flask简化页面编写
扩展Bootstrap-Flask内置了可以快速渲染Bootstrap样式HTML组件的宏，并提供了内置的Bootstrap资源，方便快速开发，使用它可以简化在Web程序里集使用Bootstrap的过程
- Bootstrap-Flask提供的包名称为flask_bootstrap，我们从这个包导入并实例化Bootstrap类，传入程序实例app，以完成扩展的初始化
```
from flask import Flask
from flask_bootstrap import Bootstrap
app = Flask(__name__)
bootstrap = Bootstrap(app)
```
- 加载资源文件
1. Bootstrap-Flask在模板中提供了一个bootstrap对象，提供了两个方法可以用来生成资源引用代码：用来加载CSS文件的bootstrap.load_css（）方法和用来加载JavaScript文件（包括Bootstrap、jQuery、Popper.js）的bootstrap.load_js（）方法。
2. Flask-Bootstrap默认从CDN（Content Delivery Network，内容分发网络）加载Bootstrap资源，同时也提供了内置的本地资源。
3. 如果想使用Bootstrap-Flask提供的本地资源，可以将配置变量BOOTSTRAP_SERVE_LOCAL设为True。另外，当FLASK_ENV环境变量设为development时，Bootstrap-Flask将自动使用本地资源
#### 使用Bootstrap-Flask提供的方法加载资源
```
<head>
    {{ bootstrap.load_css() }}
</head>
<body>
    ...
    {{ bootstrap.load_js() }}
</body>
```
#### 快捷渲染表单
- Bootstrap-Flask内置了两个用于渲染WTForms表单类的宏，一个是与我们第4章创建的form_field宏类似的render_field（）宏，另一个是用来快速渲染整个表单的render_form（）宏。这两个宏都会自动渲染错误消息，渲染表单的验证状态样式
- 使用宏在index.html模板中渲染问候表单
```
{% extends 'base.html' %}
{% from 'bootstrap/form.html' import render_form %}
{% block content %}
    <div class="hello-form">
        {{ render_form(form),action=request.full_path}}
    </div>
{% endblock %}
```
1. 它将会自动为你创建一个<form>标签，然后在标签内依次渲染包括CSRF令牌在内的所有字段。
2. 除了渲染表单字段，它还会根据表单的验证状态来渲染表单状态和错误消息。一般情况下，你只需要传入表单类实例作为参数。
3. render_form（）宏常用参数

参数|默认值|说明
---|---|---
method |'post'|表单的 method属性
extra_classes| None|额外添加的类属性
role |'form'|表单的role属性
form_type|'basic'|Bootstrap表单的样式，可以是 basic、 inline或 horizontal
button_map|{}|一个匹配按钮字段name属性到 Bootstrap按钮样式类型的字段<p>可用的样式类型有 info、 primary、 secondary、 danger、 warning、 success y light、dark<p>默认为 secondary，即" btn btn- secondary"
id|''|表单的id属性
action|''|表单提交的目标URL，默认提交到当前URL

Bootstrap-Flask内置的常用宏

宏|所在模板路径|说明
---|---|---
render_field()| bootstrap/form.html|渲染单个 WTForms表单字段
render_form()| bootstrap/form.html|渲染整个 WTForms表单类
render_pager()| bootstrap/pagination.html|渲染一个基础分页导航，仅包含上一页、下一页按钮
render_pagination() |bootstrap/pagination.html|渲染一个标准分页导航部件
render_nav_item() |bootstrap/nav.html|渲染导航链接
render_breadcrumb_item()|bootstrap/nav.html|渲染面包屑链接
## 使用Flask-Moment本地化日期和时间
datetime模块的datetime.utcnow（）方法用来生成当前的UTC（Coordinated Universal Time，协调世界时间），而UTC格式时间就是不包含时区信息的纯正时间，便于与客户端协调
```
from datetime import datetime
...
class Message(db.Model):
    ...
    timestamp = db.Column(db.DateTime, default=datetime.utcnow)
```
- 扩展Flask-Moment简化了在Flask项目中使用Moment.js的过程，集成了常用的时间和日期处理函数
- 安装
```
# pipenv install flask-moment
```
- 实例化
```
from flask_moment import Moment
app = Flask(__name__)
...
moment = Moment(app)
```
- Flask-Moment在模板中提供了moment对象
1. 这个对象提供两个方法来加载资源：moment.include_moment（）方法用来加载Moment.js的Javascript资源；moment.include_jquery（）用来加载jQuery。
2. 这两个方法默认从CDN加载资源，传入local_js参数可以指定本地资源URL
```
...
{{ moment.include_moment(local_js=url_for('static', filename='js/moment-with-locales.min.js')}}
</body>
```
- Flask-Moment默认以英文显示时间，我们可以传入区域字符串到locale（）方法来更改显示语言
```
...
{{ moment.locale('zh-cn') }}
</body>
```
- 更合理的方式是根据用户浏览器或计算机的语言来设置语言，我们可以在locale（）方法中将auto_detect参
数设为True
```
...
{{ moment.locale(auto_detect=True) }}
</body>
```
- 渲染时间日期
```
{{ moment(timestamp).format('格式字符串') }}
```
Moment.js内置格式化字符串
格式字符串|输出示例
---|---
L|2017-07-26
LL|2017年7月26日
LLL|2017年7月26日早上8点00分
LLLL|2017年7月26日星期三早上8点00分
LT|早上8点00分
LTS|早上8点0分0秒|
l11|2017年8月10日03：23
1111|2017年8月10日星期四03：23

- 除了输出普通的时间日期，Moments.js还支持输出相对时间。比如相对于当前时间的“三分钟前”“一个月前”等
```
<small>{{ moment(message.timestamp).fromNow(refresh=True) }}</small>
```
## 使用Faker生成虚拟数据
- Faker内置了20多类虚拟数据，包括姓名、地址、网络账号、信用卡、时间、职位、公司名称、Python数据等
- 要生成虚拟数据，首先要实例化Faker类，创建一个fake对象作为虚拟数据生成器
```
>>> from faker import Faker
>>> fake = Faker()
或者指定中文
>>> fake = Faker('zh_CN'))
```


## 使用Flask-Debug Toolbar调试程序
扩展Flask-DebugToolbar提供了一系列调试功能，可以用来查看请求的SQL语句、配置选项、资源加载情况等信息
- 安装
```
pipenv install flask-debugtoolbar
```
- 实例化
```
from flask import Flask
...
from flask_debugtoolbar import DebugToolbarExtension
app = Flask(__name__)
...
toolbar = DebugToolbarExtension(app)
```

## Flask配置的两种组织形式