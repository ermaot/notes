抖动可能是两个原因导致的，

1. 一个是Checkpoint机制不完善，这个暂时无法改进。步骤：Checkpoint刷新脏页–>InnoDB AIO队列–>操作系统IO队列–>存储设备
2. 一个是数据文件扩展。采用淘宝丁奇的方法，对MySQL增加预先扩展文件的功能，在测试前先将文件扩展至测试写满需要的大小，使测试过程中无需扩展文件

```
lter tablespace `trade/xxx` set extent_size=5000000; # 预先扩展数据文件
```

