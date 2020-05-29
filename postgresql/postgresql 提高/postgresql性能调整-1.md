## 预读参数

预读是使用了局部性原理，认为“当前读到的数据，相邻的数据也很可能会读到”，这样预读一些数据，可以将随机读取转化为顺序读取，提高IO效率。

查看预读参数

```
# blockdev --getra /dev/vda1
8192
```

一些机器默认为256，本机为8192.对于一些新的机器，可以设置为16384.不过该值没有特别的设置方法，需要在具体环境中多次测试

设置预读参数

```
# /sbin/blockdev --setra 16384 /dev/vda1
# /sbin/blockdev --getra /dev/vda1
```

或者使用下面的方法

```
# cat  /sys/block/vda/queue/read_ahead_kb 
8192
# echo 16384 >   /sys/block/vda/queue/read_ahead_kb
# cat  /sys/block/vda/queue/read_ahead_kb 
16384
```



## swap

可以使用swapoff然后swapon来清理swap的内容，让swap“干净”

## 透明大页

