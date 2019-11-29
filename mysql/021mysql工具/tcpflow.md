本文来自 http://wubx.net/tcpflow-dump-sql/ 和 [http://wubx.net/%e7%94%a8tcpdump%e6%8a%93%e5%8f%96mysql%e6%89%a7%e8%a1%8c%e7%9a%84sql/](http://wubx.net/用tcpdump抓取mysql执行的sql/) 



##  tcpdump抓取SQL
```
#!/bin/bash
tcpdump -i eth0 -s 0 -l -w – dst port 3306 | strings | perl -e ‘
#!/bin/bash
while(<>) { chomp; next if /^[^ ]+[ ]*$/;
if(/^(SELECT|UPDATE|DELETE|INSERT|SET|COMMIT|ROLLBACK|CREATE|DROP|ALTER|CALL)/i) {
if (defined $q) { print “$q\n”; }
$q=$_;
} else {
$_ =~ s/^[ \t]+//; $q.=” $_”;
}
}’
```

## tcpflow抓取SQL
```
#mkdir flow
#cd flow
#tcpflow –i eth0 dst MasterIP and port 3306
等待一会 Ctrl+c

#cd ..
# find flow –print0 |xargs -0 extract_queries –u >slow
#mysqldumpslow -s c slow >stats
```
- 不足之处：
1. 不能真正把SQL的执行时间记录下来，因为这个只是网络IO的流量抓取
2. 同时这个也不能把真正的连接数据库的用户抓下来




