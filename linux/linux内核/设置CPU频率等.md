https://sunpma.com/115.html



cpufrequtils是查看cpu当前频率以及修改频率、选择cpu、选择cpu运行方式的。注意，只支持某些可调节频率的cpu

```
# yum install cpufrequtils
```

##### 查看cpu类型、当前频率、支持频率、运行模式等

```
cpufreq-info
```

##### 调整CPU频率

```
cpufreq-set -c CPU号 -f 要设置频率
# cpufreq-set -c 0 -f 2.4GHz
```

#### 调整cpu频率上下限

```
cpufreq-set -d 频率下限 
cpufreq-set -u 频率上限
```

#### 调整cpu运行模式

```
cpufreq-set -g
```

这里，模式就是执行cpufreq-info后看到的所支持的模式

比如我的支持以下几种：powersave, userspace, ondemand, conservative, performance

```
powersave    是无论如何都只会保持最低频率的所谓“省电”模式；
userspace    是自定义频率时的模式，这个是当你设定特定频率时自动转变的；
ondemand     默认模式。一有cpu计算量的任务，就会立即达到最大频率运行，等执行完毕就立即回到最低频率；
conservative 保守模式，会自动在频率上下限调整，和ondemand的区别在于它会按需分配频率
performance  顾名思义只注重效率，无论如何一直保持以最大频率运行。

```

```
编辑文件，如果不存在就创建一个
vi /etc/default/cpufrequtils
添加如下规则
GOVERNOR="performance"
重启软件使其生效
systemctl restart cpufrequtils
```

#### 关闭CPU

```
echo 0 > /sys/devices/system/cpu/cpu3/online
```

#### 打开CPU

```
echo 1 > /sys/devices/system/cpu/cpu3/online
```

#### 查看是否关闭

```
#cat /proc/cpuinfo
```

如果没有该CPU则说明已关闭

```
#lscpu
```

如果看到online cpulist中没有该CPU则说明关闭

```
# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   1
Off-line CPU(s) list:  0

```

