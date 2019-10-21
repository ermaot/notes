## 换国内源
### 1. 直接使用国内源

```
#pip install web.py -i http://pypi.douban.com/simple
#pip install web.py -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
```

### 2. 配置文件
如果想配置成默认的源，方法如下：<br>
需要创建或修改配置文件（一般都是创建），<br>
linux的文件==在~/.pip/pip.conf==，<br>
windows==在%HOMEPATH%\pip\pip.ini==，<br>
修改内容为：
```
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host=pypi.douban.com
```

```
# pip --help
 
Usage:   
  pip <command> [options]
 
Commands:
  install                     安装包.
  uninstall                   卸载包.
  freeze                      按着一定格式输出已安装包列表
  list                        列出已安装包.
  show                        显示包详细信息.
  search                      搜索包，类似yum里的search.
  wheel                       Build wheels from your requirements.
  zip                         不推荐. Zip individual packages.
  unzip                       不推荐. Unzip individual packages.
  bundle                      不推荐. Create pybundles.
  help                        当前帮助.
 
General Options:
  -h, --help                  显示帮助.
  -v, --verbose               更多的输出，最多可以使用3次
  -V, --version               现实版本信息然后退出.
  -q, --quiet                 最少的输出.
  --log-file <path>           覆盖的方式记录verbose错误日志，默认文件：/root/.pip/pip.log
  --log <path>                不覆盖记录verbose输出的日志.
  --proxy <proxy>             Specify a proxy in the form [user:passwd@]proxy.server:port.
  --timeout <sec>             连接超时时间 (默认15秒).
  --exists-action <action>    Default action when a path already exists: (s)witch, (i)gnore, (w)ipe, (b)ackup.
  --cert <path>               证书.
```
