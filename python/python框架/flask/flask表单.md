在Web程序中，表单是和用户交互最常见的方式之一。用户注册、登录、撰写文章、编辑设置，无一不用到表单。表单涉及到的库有：
1. WTForms
2. Flask-WTF
3. Flask-CKEditor
## HTML表单
- 样例
```
<form method="post">
    <label for="username">Username</label><br>
    <input type="text" name="username" placeholder="Héctor Rivera"><br>
    <label for="password">Password</label><br>
    <input type="password" name="password" placeholder="19001130"><br>
    <input id="remember" name="remember" type="checkbox" checked>
    <label for="remember"><small>Remember me</small></label><br>
    <input type="submit" name="submit" value="Log in">
</form>
```
效果如下：
![image](pic/flask%E8%A1%A8%E5%8D%951.png)
- 除非比较简单的情况，一般不会在模板中直接使用HTML编写表单
## 使用flask-WTF处理表单
- 扩展Flask-WTF集成了WTForms
- Flask-WTF将表单数据解析、CSRF保护、文件上传等功能与Flask集成，另外还附加了reCAPTCHA支持
- reCAPTCHA（https://www.google.com/recaptcha/）是Google开发的免费验证码服务，在国内目前无法直接使用
- Flask-WTF默认为每个表单启用CSRF保护，它会为我们自动生成和验证CSRF令牌。
- ==默认情况下，Flask-WTF使用程序密钥来对CSRF令牌==
```
app.secret_key = 'secret string'
```

进行签名，所以我们需要为程序设置密钥
#### 定义WTForms表单类
```
from wtforms import Form, StringField, PasswordField, BooleanField, SubmitField, validators
from wtforms.validators import DataRequired, Length
class LoginForm(Form):
    username = StringField('Username', validators=[DataRequired()]) 
    password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
    remember = BooleanField('Remember me') 
    submit = SubmitField('Log in') 
```
#### 常用的WTForms字段
字段类|说明|对应的HTML表示
---|---|---
Booleanfield|复选框,值会被处理为True或 False|<input type="checkbox">
DateField|文本字段,值会被处理为 datetime.date对象|< cinput type="text">
Filefield|文件上传字段|<input type="file">
FloatField|浮点数字段,值会被处理为浮点型|<input type="text">
IntegerField|整数字段,值会被处理为整型|<input type="text">
RadioField|组单选按钮|<input type="radio">
SelectField|下拉列表|<select><option></option></select>
SelectMultipleField多选下拉列表|<select multiple><option></option></select>
Submitfield|提交按钮|<input type="submit">
StringField|文本字段|<input type="text">
HiddenField|隐藏文本字段|<input type="hidden">
PasswordField|密码文本字段|input type="password">
TextAreaField|多行文本字段|<textarea></textarea>
#### 实例化字段类常用参数
参数|说明
---|---
label|字段标签< label>的值,也就是渲染后显示在输入字段前的文字
render_Kw|个字典,用来设置对应的HTML< cinput>标签的属性<p>比如传{ placeholder:"YourName"},渲染后的HTML代码会将<Input>标签的 placeholder属性设为 Your Name
validators|一个列表,包含一系列验证器,会在表单提交后被逐一调用验证表单数据
default|字符串或可调用对象,用来为表单字段设置默认值
- 配置键WTF_CSRF_ENABLED用来设置是否开启CSRF保护，默认为True。Flask-WTF会自动在实例化表单类时添加一个包含CSRF令牌值的隐藏字段，字段名为csrf_token

#### 验证器（validator）
- 在WTForms中，验证器（validator）是一系列用于验证字段数据的类，我们在实例化字段类时使用validators关键字来指定附加的验证器列表。
- 验证器从wtforms.validators模块中导入

验证器|说明
---|---
Data Required(message=None)|验证数据是否有效
Email(message=None)|验证 Email地址
EqualTo(fieldname, message=None)|验证两个字段值是否相同
InputRequired( message=None)|验证是否有数据
Length(min=-1, max=-1, message=None)|验证输入值长度是否在给定范围内
Number range(min=None, max=None, message=None)|验证输入数字是否在给定范围内
Optional(strip whitespace=True)|允许输入值为空,并跳过其他验证
Regexp(regex, flags=0, message=None)|使用正则表达式验证输入值
URL(require tld=True, message=None)|验证URL
AnyOf(values, message=None, values formatter=None)|确保输入值在可选值列表中
Noneof(values, message=None, values formatter= None)|确保输入值不在可选值列表中

可以添加提示信息
```
name = StringField('Your Name', validators=[DataRequired(message=u'名字不能为！')]) 
```
#### flask-wtf生成html
- 创建response上下文
```
>>> from werkzeug.test import EnvironBuilder
>>> ctx = app.request_context(EnvironBuilder('/','http://localhost/').get_environ())
>>> ctx.push()

```
- 创建类
```
from wtforms import Form, StringField, PasswordField, BooleanField, SubmitField, validators
from wtforms.validators import DataRequired, Length
from flask_wtf import FlaskForm
class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
    remember = BooleanField('Remember me')
    submit = SubmitField('Log in')
```
- 使用类
```
>>> form = LoginForm()
>>> form.username()
u'<input id="username" name="username" required type="text" value="">'
>>> form.submit()
u'<input id="submit" name="submit" type="submit" value="Log in">'
>>> form.username.label
Label('username', 'Username')
>>> form.submit.label
Label('submit', 'Log in')

```
- 使用render_kw属性
```
username = StringField('Username', render_kw={'placeholder': 'Your Username'})
```
将class的定义改成
```
class LoginForm(FlaskForm):
    username = username = StringField('Username', render_kw={'placeholder': 'Your Username'})
    password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
    remember = BooleanField('Remember me')
    submit = SubmitField('Log in')
```
执行
```
>>> class LoginForm(FlaskForm):
...     username = username = StringField('Username', render_kw={'placeholder': 'Your Username'})
...     password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
...     remember = BooleanField('Remember me')
...     submit = SubmitField('Log in')
...
>>> form = LoginForm()
>>> form.username()
u'<input id="username" name="username" placeholder="Your Username" type="text" value="">'

```

- 在调用字段时传入
```
>>> form.username(style='width: 200px;', class_='bar')
u'<input class="bar" id="username" name="username" placeholder="Your Username" style="width: 200px;" type="text" value="">'

```
#### 在模板中渲染表单
- 创建模板basic.html
```
<form method="post">
    {{ form.csrf_token }} <!-- 渲染CSRF令牌隐藏字段 -->
    {{ form.username.label }}{{ form.username }}<br>
    {{ form.password.label }}{{ form.password }}<br>
    {{ form.remember }}{{ form.remember.label }}<br>
    {{ form.submit }}<br>
</form>
```
- 引入forms.py并编写路由
```
from forms import LoginForm
………………
@app.route('/basic')
def basic():
    form = LoginForm()
    return render_template('basic.html', form=form)
```
- 访问http://127.0.0.1:5000/basic
![image](pic/flask%E8%A1%A8%E5%8D%952.png)
渲染的html如下：
```
<form method="post">
    <input id="csrf_token" name="csrf_token" type="hidden" value="IjMxY2JjNzVkMTVmZWFlODAwZDY2ZGE3NTliN2RjMzYxYzU4YzRjNjYi.XYLrVg.sSh8GvjWanwrqW3Ihku2VtPsMR8"> <!-- 渲染CSRF令牌隐藏字段 -->
    <label for="username">Username</label><input id="username" name="username" required="" type="text" value=""><br>
    <label for="password">Password</label><input id="password" name="password" required="" type="password" value=""><br>
    <input id="remember" name="remember" type="checkbox" value="y"><label for="remember">Remember me</label><br>
    <input id="submit" name="submit" type="submit" value="Log in"><br>
</form>
```
- 我们调用了form.csrf_token属性渲染Flask-WTF为表单类自动创建的CSRF令牌字段。form.csrf_token字段包含了自动生成的CSRF令牌值，在提交表单后会自动被验证，为了确保表单通过验证，我们必须在表单中手动渲染这个字段

## 处理表单数据
表单数据的处理涉及很多内容，除去表单提交不说，从获取数据到保存数据大致会经历以下步骤：
1. 解析请求，获取表单数据。
2. 对数据进行必要的转换，比如将勾选框的值转换成Python的布尔值。
3. 验证数据是否符合要求，同时验证CSRF令牌。
4. 如果验证未通过则需要生成错误消息，并在模板中显示错误消息。
5. 如果通过验证，就把数据保存到数据库或做进一步处理

#### 提交表单
- 当<form>标签声明的表单中类型为submit的提交字段被单击时，就会创建一个提交表单的HTTP请求，请求中包含表单各个字段的数据。
- 表单的提交行为主要由三个属性控制

HTML表单中控制提交行为的属性
属性|默认值|说明
---|---|---
action|当前URL,即页面对应的URL|表单提交时发送请求的目标URL
method|get|提交表单的HTTP请求方法,目前仅支持使用GET和POST方法
enctype|application/x-www-form-urlencoded|表单数据的编码类型当表单中包含文件上传字段时,<p>需要设为 multipart/form-data,还可以设为纯文本类型text/plain

- form标签的action属性用来指定表单被提交的目标URL，默认为当前URL，也就是渲染该模板的路由所在的URL。如果你要把表单数据发送到其他URL，可以自定义这个属性值
- 当使用GET方法提交表单数据时，表单的数据会以查询字符串的形式附加在请求的URL里
```
http://localhost:5000/basic?username=greyli&password=12345
```
- GET方式仅适用于长度不超过3000个字符，且不包含敏感信息的表单
- 出于安全的考虑，我们一般使用POST方法提交表单。使用POST方法时，按照默认的编码类型，表单数据会被存储在请求主体中
```
POST /basic HTTP/1.0
...
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
username=greyli&password=12345
```
#### 验证表单数据
- 表单的验证通常分为以下两种形式：客户端验证和服务器端验证
1. 客户端验证
```
<input type="text" name="username" required>
```

```
{{ form.username(required='') }}
```
可以考虑使用各种JavaScript表单验证库，比如jQuery Validation Plugin（https://jqueryvalidation.org/）、Parsley.js（http://parsleyjs.org/）以及可与Bootstrap集成的BootstrapValidator（http://1000hz.github.io/bootstrap-validator/，目前仅支持Bootstrap3版本）
2. 服务器端验证（server side validation）是指用户把输入的数据提交到服务器端，在服务器端对数据进行验证
- WTForms验证机制
1. WTForms验证表单字段的方式是在实例化表单类时传入表单数据，然后对表单实例调用validate（）方法
2. 这会逐个对字段调用字段实例化时定义的验证器，返回表示验证结果的布尔值。
3. 如果验证失败，就把错误消息存储到表单实例的errors属性对应的字典中
- 样例
```
## 设定request环境
from werkzeug.test import EnvironBuilder
ctx = app.request_context(EnvironBuilder('/','http://localhost/').get_environ())
ctx.push()

## 编写类
from wtforms import Form, StringField, PasswordField, BooleanField
from wtforms.validators import DataRequired, Length
class LoginForm(Form):                                             
	username = StringField('Username', validators=[DataRequired()])
	password = PasswordField('Password', validators=[DataRequired(), Length(4, 128)])

## 使用类
>>> form = LoginForm(username='', password='123')
>>> form.data
{'username': '', 'password': '123'}
>>> form.validate()
False
>>> form.errors
{'username': [u'This field is required.'], 'password': [u'Field must be between 4 and 128 characters long.']}


>>> form2 = LoginForm(username='greyli', password='123456')
>>> form2.data
{'username': 'greyli', 'password': '123456'}
>>> form2.validate()
True
>>> form2.errors
{}

```
- 使用POST方法提交的表单，其数据会被Flask解析为一个字典，可以通过请求对象的form属性获取（request.form）
- 使用GET方法提交的表单的数据同样会被解析为字典，不过要通过请求对象的args属性获取（request.args）

#### 在视图函数中验证表单
- 
```
from flask import request
...
@app.route('/basic', methods=['GET', 'POST'])
def basic():
    form = LoginForm()  # GET + POST
    if request.method == 'POST' and form.validate():
        ...  # 处理POST请求
    return render_template('forms/basic.html', form=form)  # 处理GET请求
```
- Flask-WTF提供的validate_on_submit（）方法合并了提交和验证这两个操作
```
from flask import Flask, render_template, redirect, url_for, flash
...
@app.route('/basic', methods=['GET', 'POST'])
def basic():
    form = LoginForm()
    if form.validate_on_submit():
        username = form.username.data
        flash('Welcome home, %s!' % username)
        return redirect(url_for('login'))
    return render_template('basic.html', form=form)
```
在浏览器中，当单击F5刷新/重载时的默认行为是发送上一个请求。如果上一个请求是POST请求，那么就会弹出一个确认窗口，询问用户是否再次提交表单。为了避免出现这个容易让人产生困惑的提示，我们尽量不要让提交表单的POST请求作为最后一个请求。这就是为什么我们在处理表单后返回一个重定向响应，这会让浏览器重新发送一个新的GET请求到重定向的目标URL

#### 在模板中渲染错误消息
- 对于验证未通过的字段，WTForms会把错误消息添加到表单类的errors属性（匹配作为表单字段的类属性到对应的错误消息列表的字典）中
- 可直接通过字段名来获取对应字段的错误消息列表，即“form.字段名.errors”

## 表单进阶
#### 设置错误消息语言
#### 使用宏渲染表单
- 在模板中渲染表单时，我们有大量的工作要做：调用字段属性，获取<input>定义;调用对应的label属性，获取<label>定义;·渲染错误消息。可以使用宏
```
{% macro form_field(field) %}
    {{ field.label }}<br>
    {{ field(**kwargs) }}<br>
    {% if field.errors %}
        {% for error in field.errors %}
            <small class="error">{{ error }}</small><br>
        {% endfor %}
    {% endif %}
{% endmacro %}
```

```
{% from 'macros.html' import form_field %}
...
<form method="post">
    {{ form.csrf_token }}
    {{ form_field(form.username)}}<br>
    {{ form_field(form.password) }}<br>
    ...
</form>
```
- 这类工作可以交给扩展来完成

#### 自定义验证器
- 行内验证器
```
from wtforms import IntegerField, SubmitField
from wtforms.validators import ValidationError
class FortyTwoForm(FlaskForm):
    answer = IntegerField('The Number')
    submit = SubmitField()
    def validate_answer(form, field):
        if field.data != 42:
            raise ValidationError('Must be 42.')
```
- 全局验证器
```
from wtforms.validators import ValidationError
def is_42(form, field):
    if field.data != 42:
        raise ValidationError('Must be 42')
class FortyTwoForm(FlaskForm):
    answer = IntegerField('The Number', validators=[is_42])
    submit = SubmitField()
```
工厂函数形式的全局验证器示例
```
from wtforms.validators import ValidationError
def is_42(message=None):
    if message is None:
        message = 'Must be 42.'
    def _is_42(form, field):
        if field.data != 42:
            raise ValidationError(message)
    return _is_42
class FortyTwoForm(FlaskForm):
    answer = IntegerField('The Number', validators=[is_42()])
    submit = SubmitField()
```
#### 文件上传
- 创建上传表单
Flask-WTF提供FileField类，它继承WTForms提供的上传字段FileField，添加对Flask的集成
```
from flask wtf.file import FileField, FileRequired, FileAllowed
class UploadForm(FlaskForm):
    photo = FileField('Upload Image', validators=[FileRequired(), FileAllowed(['j
    submit = SubmitField()
```
验证器|说明
---|---
FileRequired( message=None)|验证是否包含文件对象
FileAllowed(upload_set,message=None)|用来验证文件类型,upload_set参数用来传入包含允许的文件后缀名列表

1. 使用HTML5中的accept属性也可以在客户端实现简单的类型过滤
2. 扩展Flask-Uploads（https://github.com/maxcountryman/flask-uploads）内置了在Flask中实现文件上传的便利功能
3. Flask-WTF提供的FileAllowed（）也支持传入Flask-Uploads中的上传集对象（Upload_Set）作为upload_set参数的值
4. 有Flask-Transfer（https://github.com/justanr/Flask-Transfer）
5. 通过设置Flask内置的配置变量MAX_CONTENT_LENGTH，我们可以限制请求报文的最大长
度，单位为字节
```
app.config['MAX_CONTENT_LENGTH'] = 3 * 1024 * 1024
```
- 渲染上传表单
```
<form method="post" enctype="multipart/form-data">
    {{ form.csrf_token }}
    {{ form_field(form.photo) }}
    {{ form.submit }}
</form>
```

- 处理上传文件
1. 当包含上传文件字段的表单提交后，上传的文件需要在请求对象的files属性（request.files）中获取
```
ImmutableMultiDict([('photo', <FileStorage: u'0f913b0ff95.JPG' ('image/jpeg')>)])
```
2. 上传的文件会被Flask解析为Werkzeug中的FileStorage对象（werkzeug.datastructures.FileStorage）。当手动处理时，我们需要使用文件上传字段的name属性值作为键获取对应的文件对象
```
request.files.get('photo')
```
3. 用Flask-WTF时，它会自动帮我们获取对应的文件对象
```
import os
app.config['UPLOAD_PATH'] = os.path.join(app.root_path, 'uploads')
@app.route('/upload', methods=['GET', 'POST'])
def upload():
    form = UploadForm()
    if form.validate_on_submit():
        f = form.photo.data
        filename = random_filename(f.filename)
        f.save(os.path.join(app.config['UPLOAD_PATH'], filename))
        flash('Upload success.')
        session['filenames'] = [filename]
        return redirect(url_for('show_images'))
    return render_template('upload.html', form=form)
```
- 当表单通过验证后，我们通过form.photo.data获取存储上传文件的FileStorage对象
- 文件名
```
#原文件名
filename = f.filename

#安全文件名，secure_filename（）会过滤掉文件名中的非ASCII字符
>>> from werkzeug import secure_filename
>>> secure_filename('avatar!@#//#\\%$^&.jpg')
'avatar.jpg'
>>> secure_filename('avatar头像.jpg')
'avatar.jpg'

#重命名
def random_filename(filename):
    ext = os.path.splitext(filename)[1]
    new_filename = uuid.uuid4().hex + ext
    return new_filename
```
- 在UUID的标准中，UUID分为5个版本，每个版本使用不同的生成方法并且适用于不同的场景。uuid4（）方法对应的是第4个版本：不接收参数而生成随机UUID
- uploads文件夹，用于保存上传后的文件。指向这个文件夹的绝对路径存储在自定义配置变量UPLOAD_PATH中
```
app.config['UPLOAD_PATH'] = os.path.join(app.root_path, 'uploads')
```
- 对FileStorage对象调用save（）方法即可保存
```
f.save(os.path.join(app.config['UPLOAD_PATH'], filename))
```
- 显示上传后的图片
```
@app.route('/uploads/<path:filename>')
def get_file(filename):
    return send_from_directory(app.config['UPLOAD_PATH'], filename)
```
#### 使用Flask-CKEditor集成富文本编辑器
- 富文本编辑器即WYSIWYG（What You See Is What You Get，所见即所得）编辑器，类似于我们经常使用的文本编辑软件。它提供一系列按钮和下拉列表来为文本设置格式，编辑状态的文本样式即最终呈现出来的样式
- CKEditor（http://ckeditor.com/）是一个开源的富文本编辑器，它包含丰富的配置选项，而且有大量第三方插件支持
- 扩展Flask-CKEditor简化了在Flask程序中使用CKEditor的过程
```
pipenv install flask-ckeditor
```
- 实例化Flask-CKEditor提供的CKEditor类，传入程序实例
```
from flask_ckeditor import CKEditor
ckeditor = CKEditor(app)
```
#### 配置富文本编辑器
- Flask-CKEditor常用配置
完整配置见：[Flask-CKEditor文档的配置部分](https://flask-ckeditor.readthedocs.io/en/latest/configuration.html)

配置键|默认值|说明
---|---|---
CKEDITOR_SERVE_LOCAL|False|设为True会使用内置的本地资源
CKEDITOR_PKG_TYPE|'standard'|CKEditor包类型,可选值为basic、standard和full
CKEDITOR_LANGUAGE|''|界面语言,传人ISO639格式的语言码
CKEDITOR_HEIGHT|''|编辑器高度
CKEDITOR_WIDTH|''|编辑器宽度
```
app.config['CKEDITOR_SERVE_LOCAL'] = True
```
1. CKEDITOR_SERVE_LOCAL和CKEDITOR_PKG_TYPE配置变量仅限于使用Flask-CKEditor提供的方法加载资源时有效，手动引入资源时可以忽略。
2. 配置变量CKEDITOR_LANGUAGE用来固定界面的显示语言（简体中文和繁体中文对应的配置分别为zh-cn和zh），如果不设置，默认CKEditor会自动探测用户浏览器的语言偏好，然后匹配对应的语言，匹配失败则默认使用英文
3. Flask-CKEditor内置的资源已经包含了常用插件，可以通过Flask-CKEditor提供的示例程序（https://github.com/greyli/flask-ckeditor/tree/master/examples）来了解这些功能的具体实现

- 渲染富文本编辑器
富文本编辑器在HTML中通过文本区域字段表示，即<textarea></textarea>。Flask-CKEditor通过包装WTForms提供的TextAreaField字段类型实现了一个CKEditorField字段类
```
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField
from wtforms.validators import DataRequired, Length
from flask_ckeditor import CKEditorField  # 从flask_ckeditor包导入
class RichTextForm(FlaskForm):
    title = StringField('Title', validators=[DataRequired(), Length(1, 50)])
    body = CKEditorField('Body', validators=[DataRequired()])
    submit = SubmitField('Publish')
```
表单
```
{% extends 'base.html' %}
{% from 'macros.html' import form_field %}
{% block content %}
<h1>Integrate CKEditor with Flask-CKEditor</h1>
<form method="post">
    {{ form.csrf_token }}
    {{ form_field(form.title) }}
    {{ form_field(form.body) }}
    {{ form.submit }}
</form>
{% endblock %}
{% block scripts %}
{{ super() }}
{{ ckeditor.load() }}
{% endblock %}
```
#### 单个表单多个提交按钮
- form/forms.py：包含两个提交按钮的表单
```
class NewPostForm(FlaskForm):
    title = StringField('Title', validators=[DataRequired(), Length(1, 50)])
    body = TextAreaField('Body', validators=[DataRequired()])
    save = SubmitField('Save')  # 保存按钮
    publish = SubmitField('Publish')  # 发布按钮
```
- form/app.py：判断被单击的提交按钮
```
@app.route('/two-submits', methods=['GET', 'POST'])
def two_submits():
    form = NewPostForm()
    if form.validate_on_submit():
        if form.save.data:  # 保存按钮被单击
            # save it...
            flash('You click the "Save" button.')
        elif form.publish.data:  # 发布按钮被单击
            # publish it...
            flash('You click the "Publish" button.')
        return redirect(url_for('index'))
    return render_template('2submit.html', form=form)
```
