configparser库可以帮助解析配置文件。在python2系列，该库名称Configparser

In [1]: import configparser

In [2]: conf = configparser.ConfigParser()

In [3]: conf.read('config.ini')
Out[3]: ['config.ini']

In [4]: conf.get('test','para1')
Out[4]: 'test1'

