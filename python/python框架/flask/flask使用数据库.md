本章新涉及的Python库如下所示：
- SQLAlchemy（1.2.7）
- Flask-SQLAlchemy（2.3.2）
- Alembic（0.9.9）
- Flask-Migrate（2.1.1）

## 数据库分类
- NoSQL
1. 文档存储（document store）：文档存储使用的文档类似SQL数据库中的记录，文档使用类JSON格式来表示数据。常见的文档存储DBMS有MongoDB、CouchDB等
```
{
    id: 1,
    name: "Nick",
    sex: "Male"
    occupation: "Journalist"
}
```
2. 键值对存储（key-value store）：。常见的键值对DBMS有Redis、Riak等，其中Redis不仅可以管理键值对数据库，还可
以作为缓存后端（cache backend）和消息代理（message broker）
3. 有列存储（column store，又被称为宽列式存储）
4. 图存储（graph store）等
- SQL
## ORM
- 在Web应用里使用原生SQL语句操作数据库主要存在下面两类问题：
1. 手动编写SQL语句比较乏味，而且视图函数中加入太多SQL语句会降低代码的易读性。另外还会容易出现安全问题，比如SQL注入。
2. 常见的开发模式是在开发时使用简单的SQLite，而在部署时切换到MySQL等更健壮的DBMS。但是对于不同的DBMS，我们需要使用不同的Python接口库，这让DBMS的切换变得不太容易
- ORM把底层的SQL数据实体转化成高层的Python对象，甚至不需要了解SQL，只需要通过Python代码即可完成数据库操作，ORM主要实现了三层映射关系：
1. 表→Python类
2. 字段（列）→类属性
3. 记录（行）→类实例
- ORM 优点
1. 便于使用
2. 灵活性好。你既能使用高层对象来操作数据库，又支持执行原生SQL语句。
3. 提升效率。从高层对象转换成原生SQL会牺牲一些性能，但这微不足道的性能牺牲换取的是巨大的效率提升。
4. 可移植性好。ORM通常支持多种DBMS，包括MySQL、PostgreSQL、Oracle、SQLite等。你可以随意更换DBMS，只需要稍微改动少量配置

## 使用Flask-SQLAlchemy管理数据库
#### 安装与初始化
安装
```
pipenv install flask-sqlalchemy
```
初始化
```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
app = Flask(__name__)
db = SQLAlchemy(app)
```
#### 连接数据库
一些常用的DBMS及其数据库URI格式示例
DBMS|URI
---|---
PostgreSQL|postgresql://username:password@host/databasename
MySQL|mysql://username:password@host/databasename
Oracle|oracle://username:password@host:port/sidname
SQLite(UNIX)|sqlite:////absolute/path/to/foo.db
SQLite(Windows)|sqlite:///absolute\\path\\to\\foo.db或者sqlite:///absolute\path\to\foo.db'
SQlite(内存型)|sqlite:///或sqlite:///:memory:

- 在Flask-SQLAlchemy中，数据库的URI通过配置变量SQLALCHEMY_DATABASE_URI设置，默认为SQLite内存型数据库
```
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL', 'sqlite:///' + os.path.curdir)
```
- Flask-SQLAlchemy建议你设置SQLALCHEMY_TRACK_MODIFICATIONS配置变量，这个配置变量决
定是否追踪对象的修改，这用于Flask-SQLAlchemy的事件通知系统。默认值为None，如果没有特殊需要，我们可以把它设为False来关闭警告信息
```
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
```
#### 定义数据库模型
- 样例
```
class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.Text)
```
SQLAlchemy常用的字段类型
字段|说明
---|---
Integer|整数
string|字符串,可选参数 length可以用来设置最大长度
Text|较长的 Unicode文本
Date|日期,存储 Python的 datetime, date对象
Time|时间,存储 Python的 datetime.time对象
DateTime|时间和日期,存储 Python的 datetime对象
Interval|时间间隔,存储 Python的 datetime. timedelta对象
Float|浮点数
Boolean|布尔值
PickleType|存储 Pickle列化的 Python对象
LargeBinary|存储任意二进制数据