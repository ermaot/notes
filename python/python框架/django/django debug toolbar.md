## 简介

Django Debug Toolbar是Django开发中必备利器，可以帮助开发者快速了解项目的整体信息以及每个页面包括sql信息，http相关信息。本篇将详细讲解如何django-debug-toolbar的使用。

## 项目集成Django Debug Toolba

我们去**Django Debug Toolba**r官网，跟随官方文档一起学习[Django Debug Toolba官方文档](https://django-debug-toolbar.readthedocs.io/en/latest/)

### 1 .安装

![这里写图片描述](pic/django debug toolbar/20180915161222373)
如图，使用pip命令直接安装即可，（**注意：一般我们会用virtualenvwrapper创建虚拟创建开发环境，那么切记一定要先是workon命令切换到当前的开发环境再安装。**）

```
pip install django-debug-toolbar
```

### 2.把debug_toolbar添加进INSTALLED_APPS

![这里写图片描述](pic/django debug toolbar/20180915161956839)

**注意：**如上图官方文档说的，一定要把**debug_toolbar**放在**django.contrib.staticfiles**，当然不要理解为紧跟着**django.contrib.staticfiles**后面，只要在后面即可。
![这里写图片描述](pic/django debug toolbar/20180915162415596)

### 3.urls中配置

![这里写图片描述](pic/django debug toolbar/20180915162607334)

如上图所示，在项目url中配置
path中，地址为`__debug__/`，视图为`include(debug_toolbar.urls)`
完整的配置为：
![这里写图片描述](pic/django debug toolbar/2018091516300394)

### 4 Middleware中间件的配置

官方文档截图：
![这里写图片描述](pic/django debug toolbar/20180915163238693)
在settings中的**MIDDLEWARE**配置’**debug_toolbar.middleware.DebugToolbarMiddleware’**,我们要把django-debug-toolbar这个中间件**尽可能配置到最前面**，但是，必须要要放在**处理编码和响应内容的中间件后面**，比如我们要是使用了`GZipMiddleware`，就要把`DebugToolbarMiddleware`放在`GZipMiddleware`后面。
如下图，我没有使用到**处理编码和响应内容的中间件后面**，所以直接放在了最前面
![这里写图片描述](pic/django debug toolbar/20180915163803957)

### 5.配置IP地址

官方文档截图：
![这里写图片描述](pic/django debug toolbar/20180915164301730)
我们需要在**settings.py文件中**配置INTERNAL_IPS，只有访问这里面配置的ip地址时， Debug Toolbar才是展示出来。因为我们一般都是本地开发，所以，直接配置为127.0.0.1就可以了
如下我的配置：
![这里写图片描述](pic/django debug toolbar/20180915164721658)

### 配置完成，界面显示django-debug-toolbar面板

现在，运行羡慕，我们Django Debug Toolbar界面就会出现了
![这里写图片描述](pic/django debug toolbar/20180915165559676)
我们网页的右上角就会出现一个DjDt的图标，点击展开
![这里写图片描述](pic/django debug toolbar/2018091516573061)

右边则为页面的信息，比如我们点开versions,就是Django的版本和相关信息
![这里写图片描述](pic/django debug toolbar/20180915170000397)

### django-debug-toolbar面板介绍

- **Versions** ：代表是哪个django版本
- **Timer** : 用来计时的，判断加载当前页面总共花的时间
- **Settings** : 读取django中的配置信息
- **Headers** : 当前请求头和响应头信息
- **Request**: 当前请求的想信息（视图函数，Cookie信息，Session信息等）
- **SQL**:查看当前界面执行的SQL语句
- **StaticFiles**：当前界面加载的静态文件
- **Templates**:当前界面用的模板
- **Cache**：缓存信息
- **Signals**：信号
- **Logging**：当前界面日志信息
- **Redirects**：当前界面的重定向信息

### 自定义自己的django-debug-toolbar右侧面板

右侧面板太多，有些面板信息不需要，比如Versions django版本这个面板我们不需要显示在这里那么怎么办：
官网截图：
![这里写图片描述](pic/django debug toolbar/20180915171853409)

我们在**settings.py**去使用`DEBUG_TOOLBAR_PANELS`配置我们需要显示的面板即可

```
DEBUG_TOOLBAR_PANELS = [
    # 代表是哪个django版本
    'debug_toolbar.panels.versions.VersionsPanel',
    # 用来计时的，判断加载当前页面总共花的时间
    'debug_toolbar.panels.timer.TimerPanel',
    # 读取django中的配置信息
    'debug_toolbar.panels.settings.SettingsPanel',
    # 看到当前请求头和响应头信息
    'debug_toolbar.panels.headers.HeadersPanel',
    # 当前请求的想信息（视图函数，Cookie信息，Session信息等）
    'debug_toolbar.panels.request.RequestPanel',
    # 查看SQL语句
    'debug_toolbar.panels.sql.SQLPanel',
    # 静态文件
    'debug_toolbar.panels.staticfiles.StaticFilesPanel',
    # 模板文件
    'debug_toolbar.panels.templates.TemplatesPanel',
    # 缓存
    'debug_toolbar.panels.cache.CachePanel',
    # 信号
    'debug_toolbar.panels.signals.SignalsPanel',
    # 日志
    'debug_toolbar.panels.logging.LoggingPanel',
    # 重定向
    'debug_toolbar.panels.redirects.RedirectsPanel',
]1234567891011121314151617181920212223242526
```

比如我只需要**TimerPanel**、**RequestPanel**、**HeadersPanel**、**SQLPanel**这四个面板；我们就可以配置：
![这里写图片描述](pic/django debug toolbar/20180915172421370)
配置好后，重新运行我们的项目,则只显示这个4个面板：
![这里写图片描述](pic/django debug toolbar/20180915172536277)



摘抄自：https://blog.csdn.net/cn_1937/article/details/82715983