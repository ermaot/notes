## 安装pip 和 pipenv
1. 如果pip尚未安装，下载 get-pip.py  [下载链接](https://bootstrap.pypa.io/get-pip.py)，然后执行安装即可

```
PyPI（https://pypi.org）是一个Python包的在线仓库，截至2018年5月，共有13万多个包
```

2. 安装完毕pip后，安装pipenv
pipenv是pip、Pipfile和Virtualenv的结合体，让包安装、包依赖管理和虚拟环境管理更加方便
```
# pip install pipenv

# pipenv --version
pipenv, version 2018.11.26
```
## 创建虚拟运行环境

```
# pipenv install
Creating a virtualenv for this project...
Pipfile: /docs/pyenv/Pipfile
……………………
✔ Successfully created virtual environment! 
Virtualenv location: /root/.local/share/virtualenvs/pyenv-s6Us8EjH
……………………
```
- 默认情况下，Pipenv会统一管理所有虚拟环境
- 在Windows系统中，虚拟环境文件夹会在C：\Users\Administrator\.virtualenvs\目录下创建
- Linux或macOS会在~/.local/share/virtualenvs/目录下创建
- 如果你想在项目目录内创建虚拟环境文件夹，可以设置环境变量PIPENV_VENV_IN_PROJECT，这时名为.venv的虚拟环境文件夹将在项目根目录被创建
- 虚拟环境文件夹的目录名称的形式为“当前项目目录名+一串随机字符”，比如helloflask-5Pa0ZfZw
#### 进入环境
```
# pipenv shell
Launching subshell in virtual environment...
 . /root/.local/share/virtualenvs/pyenv-s6Us8EjH/bin/activate
[root@VM_112_34_centos pyenv]#  . /root/.local/share/virtualenvs/pyenv-s6Us8EjH/bin/activate
(pyenv) [root@VM_112_34_centos pyenv]# 

```
- 当执行pipenv shell或pipenv run命令时，Pipenv会自动从项目目录下的.env文件中加载环境变量

#### pipenv run
- 除了显式地激活虚拟环境，Pipenv还提供了一个pipenv run命令，这个命令允许你不显式激活虚拟环境即可在当前项目的虚拟环境中执行命令，比如
- ，pipenv run是更推荐的做法，因为这个命令可以让你在执行操作时不用关心自己是否激活了虚拟环境
```
#  pipenv run python hello.py
```
#### 管理依赖
- 在创建虚拟环境时，如果项目根目录下没有Pipfile文件，pipenv install命令还会在项目文件夹根目录下创建Pipfile和Pipfile.lock文件
- 前者用来记录项目依赖包列表，而后者记录了固定版本的详细依赖包列表。
- 当我们使用Pipenv安装/删除/更新依赖包时，Pipfile以及Pipfile.lock会自动更新
- 使用pip list 查看依赖列表
```
# pip list
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Package    Version
---------- -------
pip        19.2.3 
setuptools 41.2.0 
wheel      0.33.6 

```
- 在项目路径下使用pipenv graph查看依赖情况
```
# pipenv graph
Flask==1.1.1
  - click [required: >=5.1, installed: 7.0]
  - itsdangerous [required: >=0.24, installed: 1.1.0]
  - Jinja2 [required: >=2.10.1, installed: 2.10.1]
    - MarkupSafe [required: >=0.23, installed: 1.1.1]
  - Werkzeug [required: >=0.15, installed: 0.15.6]

```
## 安装flask
```
# pipenv install flask
Installing flask...
? Installation Succeeded 
Pipfile.lock (c3d3b3) out of date, updating to (dfae9f)...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
? Success! 
Updated Pipfile.lock (c3d3b3)!
Installing dependencies from Pipfile.lock (c3d3b3)...
  ??   ???????????????????????????????? 6/6 ? 00:00:18

```
- Pipenv会自动帮我们管理虚拟环境，所以在执行pipenv install安装Python包时，无论是否激活虚拟环境，包都会安装到虚拟环境中。
- 后面我们都将使用Pipenv安装包，这相当于在激活虚拟环境的情况下使用pip安装包。只有需要在全局环境下安装/更新/删除包，我们才会使用pip

#### 使用 pipenv update flask更新flask
```
# pipenv update flask
```
## 安装pycharm