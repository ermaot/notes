## tcprstat分析服务的响应速度利器
tcprstat是percona用来监测mysql响应时间的。不过对于任何运行在TCP协议上的响应时间，都可以用。
- 示例
```
$sudo tcprstat -p 3306 -t 1 -n 5
timestamp   count   max min avg med stddev  95_max  95_avg  95_std  99_max  99_avg  99_std
1283261499  1870    559009  39  883 153 13306   1267    201 150 6792    323 685
1283261500  1865    25704   29  578 142 2755    889 175 107 23630   333 1331
1283261501  1887    26908   33  583 148 2761    714 176 94  23391   339 1340
1283261502  2015    304965  35  624 151 7204    564 171 79  8615    237 507
1283261503  1650    289087  35  462 146 7133    834 184 120 3565    244 358
```
- 你也可以读取tcpdump的文件进行分析
```
$sudo tcpdump -i eth0 -nn port 80  -w ./tcpdump.log 
$sudo tcprstat -l 10.234.9.103 -t 2 -n 5 -r ./tcpdump.log 
timestamp       count   max     min     avg     med     stddev  95_max  95_avg  95_std  99_max  99_avg  99_std
1403180482      2       28017   26717   27367   28017   650     26717   26717   0       26717   26717   0
1403180484      0       0       0       0       0       0       0       0       0
```
#### 安装
1. 下载http://github.com/downloads/Lowercases/tcprstat/tcprstat-static.v0.3.1.x86_64
2. 移动到/usr/bin并给与执行权限

#### 参数说明
```
命令行参数    简短形式   类型      描述                    默认值
--format    -f        字符串     输出格式化字符串  ”%T\t%n\t%M\t%m\t%a\t%h\t%S\t%95M\t%95a\t%95S\t%99M\t%99a\t%99S\n” 
--help                          显示帮助信息
--interval  -t        数字      监控多少秒输出一次统计     10
--iterations  -n      数字      共输出几次统计信息         1
--local       -l      字符串    本级ip地址列表
--port        -p      数字      服务端口
--read        -r      字符串    pcap文件路径
--version                      显示版本信息
--no-header           字符串    输出不显示头信息
--header              字符串    指定输出的头信息
```