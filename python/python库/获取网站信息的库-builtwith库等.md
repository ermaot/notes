## 识别网站所用技术
构建网站所使用的技术类型也会对我们如何爬取产生影响。 有一个十分有用的工具可以检查网站构建的技术类型一builtwi th 模块。 该模块的安装方法如下。
```
pip  install  builtwith 
```
该模块将URL作为参数，下载该URL并对其进行分析，然后返回该网站使用的技术。 下面是使用该模块的一个例子

```
In [20]: import builtwith

In [21]: builtwith.parse("https://www.baidu.com")
Out[21]: {}

In [22]: builtwith.parse("http://www.univebook.cn")
Out[22]:
{'web-servers': ['Nginx'],
 'web-frameworks': ['Twitter Bootstrap'],
 'javascript-frameworks': ['jQuery']}
```

可以看到builtwith库能力有限，不一定能获取全部信息

## whois

```
pip  install  python-whois
```

```
In [1]: import whois

In [2]: whois.whois("www.baidu.com")
Out[2]:
{'domain_name': ['BAIDU.COM', 'baidu.com'],
 'registrar': 'MarkMonitor, Inc.',
 'whois_server': 'whois.markmonitor.com',
 'referral_url': None,
 'updated_date': [datetime.datetime(2019, 5, 9, 4, 30, 46),
  datetime.datetime(2019, 5, 8, 20, 59, 33)],
 'creation_date': [datetime.datetime(1999, 10, 11, 11, 5, 17),
  datetime.datetime(1999, 10, 11, 4, 5, 17)],
 'expiration_date': [datetime.datetime(2026, 10, 11, 11, 5, 17),
  datetime.datetime(2026, 10, 11, 0, 0)],
 'name_servers': ['NS1.BAIDU.COM',
  'NS2.BAIDU.COM',
  'NS3.BAIDU.COM',
  'NS4.BAIDU.COM',
  'NS7.BAIDU.COM',
  'ns1.baidu.com',
  'ns7.baidu.com',
  'ns4.baidu.com',
  'ns3.baidu.com',
  'ns2.baidu.com'],
 'status': ['clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited',
  'clientTransferProhibited https://icann.org/epp#clientTransferProhibited',
  'clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited',
  'serverDeleteProhibited https://icann.org/epp#serverDeleteProhibited',
  'serverTransferProhibited https://icann.org/epp#serverTransferProhibited',
  'serverUpdateProhibited https://icann.org/epp#serverUpdateProhibited',
  'clientUpdateProhibited (https://www.icann.org/epp#clientUpdateProhibited)',
  'clientTransferProhibited (https://www.icann.org/epp#clientTransferProhibited)',
  'clientDeleteProhibited (https://www.icann.org/epp#clientDeleteProhibited)',
  'serverUpdateProhibited (https://www.icann.org/epp#serverUpdateProhibited)',
  'serverTransferProhibited (https://www.icann.org/epp#serverTransferProhibited)',
  'serverDeleteProhibited (https://www.icann.org/epp#serverDeleteProhibited)'],
 'emails': ['abusecomplaints@markmonitor.com', 'whoisrequest@markmonitor.com'],
 'dnssec': 'unsigned',
 'name': None,
 'org': 'Beijing Baidu Netcom Science Technology Co., Ltd.',
 'address': None,
 'city': None,
 'state': 'Beijing',
 'zipcode': None,
 'country': 'CN'}
```

## robots.txt解析

使用Python自带 的robotparser模块

