本文来自：[https://bean-li.github.io/%E9%80%9A%E8%BF%87ipmitool%E8%8E%B7%E5%8F%96%E5%90%84%E5%85%83%E4%BB%B6%E7%9A%84%E6%B8%A9%E5%BA%A6%E4%BF%A1%E6%81%AF/](https://bean-li.github.io/通过ipmitool获取各元件的温度信息/)

ipmitool可以获取各个元件的温度信息，如何判断各个组件的温度信息，各个组件的温度信息是否OK，有没有温度过高或者过低的元件需要告警？

# 获取各个元件温度的方法

我们可以通过如下指令获取所有元件的温度信息和相关的状态：

```
root@node244:~# ipmitool sensor list 
CPU1 Temp        | 29.000     | degrees C  | ok    | 0.000     | 0.000     | 0.000     | 85.000    | 90.000    | 90.000    
CPU2 Temp        | 33.000     | degrees C  | nr    | 10.000    | 10.000    | 10.000    | 30.000    | 30.000    | 30.000    
PCH Temp         | 32.000     | degrees C  | ok    | 0.000     | 5.000     | 16.000    | 90.000    | 95.000    | 100.000   
System Temp      | 30.000     | degrees C  | ok    | -10.000   | -5.000    | 0.000     | 80.000    | 85.000    | 90.000    
Peripheral Temp  | 34.000     | degrees C  | ok    | -10.000   | -5.000    | 0.000     | 80.000    | 85.000    | 90.000    
Vcpu1VRM Temp    | 28.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 95.000    | 100.000   | 105.000   
Vcpu2VRM Temp    | 34.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 95.000    | 100.000   | 105.000   
VmemABVRM Temp   | 29.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 95.000    | 100.000   | 105.000   
VmemCDVRM Temp   | 28.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 95.000    | 100.000   | 105.000   
VmemEFVRM Temp   | 31.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 95.000    | 100.000   | 105.000   
VmemGHVRM Temp   | 30.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 95.000    | 100.000   | 105.000   
P1-DIMMA1 Temp   | 27.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 80.000    | 85.000    | 90.000    
P1-DIMMA2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P1-DIMMB1 Temp   | 27.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 80.000    | 85.000    | 90.000    
P1-DIMMB2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P1-DIMMC1 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P1-DIMMC2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P1-DIMMD1 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P1-DIMMD2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P2-DIMME1 Temp   | 29.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 80.000    | 85.000    | 90.000    
P2-DIMME2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P2-DIMMF1 Temp   | 30.000     | degrees C  | ok    | -5.000    | 0.000     | 5.000     | 80.000    | 85.000    | 90.000    
P2-DIMMF2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P2-DIMMG1 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P2-DIMMG2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P2-DIMMH1 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
P2-DIMMH2 Temp   | na         |            | na    | na        | na        | na        | na        | na        | na        
FAN1             | 4400.000   | RPM        | ok    | 300.000   | 500.000   | 700.000   | 25300.000 | 25400.000 | 25500.000 
FAN2             | 4300.000   | RPM        | ok    | 300.000   | 500.000   | 700.000   | 25300.000 | 25400.000 | 25500.000 
FAN3             | 4400.000   | RPM        | ok    | 300.000   | 500.000   | 700.000   | 25300.000 | 25400.000 | 25500.000 
FAN4             | na         |            | na    | na        | na        | na        | na        | na        | na        
FAN5             | na         |            | na    | na        | na        | na        | na        | na        | na        
FAN6             | na         |            | na    | na        | na        | na        | na        | na        | na        
FANA             | 4400.000   | RPM        | ok    | 300.000   | 500.000   | 700.000   | 25300.000 | 25400.000 | 25500.000 
FANB             | na         |            | na    | na        | na        | na        | na        | na        | na        
12V              | 12.315     | Volts      | ok    | 10.173    | 10.299    | 10.740    | 12.945    | 13.260    | 13.386    
5VCC             | 5.000      | Volts      | ok    | 4.246     | 4.298     | 4.480     | 5.390     | 5.546     | 5.598     
3.3VCC           | 3.316      | Volts      | ok    | 2.789     | 2.823     | 2.959     | 3.554     | 3.656     | 3.690     
VBAT             | 3.104      | Volts      | ok    | 2.376     | 2.480     | 2.584     | 3.494     | 3.598     | 3.676     
Vcpu1            | 1.800      | Volts      | ok    | 1.242     | 1.260     | 1.395     | 1.899     | 2.088     | 2.106     
Vcpu2            | 1.809      | Volts      | ok    | 1.242     | 1.260     | 1.395     | 1.899     | 2.088     | 2.106     
VDIMMAB          | 1.200      | Volts      | ok    | 0.948     | 0.975     | 1.047     | 1.344     | 1.425     | 1.443     
VDIMMCD          | 1.209      | Volts      | ok    | 0.948     | 0.975     | 1.047     | 1.344     | 1.425     | 1.443     
VDIMMEF          | 1.209      | Volts      | ok    | 0.948     | 0.975     | 1.047     | 1.344     | 1.425     | 1.443     
VDIMMGH          | 1.209      | Volts      | ok    | 0.948     | 0.975     | 1.047     | 1.344     | 1.425     | 1.443     
5VSB             | 4.974      | Volts      | ok    | 4.246     | 4.298     | 4.480     | 5.390     | 5.546     | 5.598     
3.3VSB           | 3.316      | Volts      | ok    | 2.789     | 2.823     | 2.959     | 3.554     | 3.656     | 3.690     
1.5V PCH         | 1.509      | Volts      | ok    | 1.320     | 1.347     | 1.401     | 1.644     | 1.671     | 1.698     
1.2V BMC         | 1.209      | Volts      | ok    | 1.020     | 1.047     | 1.092     | 1.344     | 1.371     | 1.398     
1.05V PCH        | 1.050      | Volts      | ok    | 0.870     | 0.897     | 0.942     | 1.194     | 1.221     | 1.248     
Chassis Intru    | 0x0        | discrete   | 0x0000| na        | na        | na        | na        | na        | na        
PS1 Status       | 0x1        | discrete   | 0x0100| na        | na        | na        | na        | na        | na        
PS2 Status       | 0x1        | discrete   | 0x0100| na        | na        | na        | na        | na        | na        
AOC_SAS Temp     | 60.000     | degrees C  | ok    | -11.000   | -8.000    | -5.000    | 100.000   | 105.000   | 110.000   
HDD Temp         | 29.000     | degrees C  | ok    | -11.000   | -8.000    | -5.000    | 50.000    | 55.000    | 60.000    
HDD Status       | 0x1        | discrete   | 0x01ff| na        | na        | na        | na        | na        | na    
```

一般来讲，第三列的值中有degree的，我们统计的是温度信息。

- 第一列： 传感器的名称，比如CPU1 Temp，
- 第二列: 该元件的当前温度值，注意有时候会是na，即取不到。
- 第四列： 温度的状态信息，ok表示温度正常，有时候该状态值为nr，为non-recovery，不可恢复的意思

一般来讲，常见的温度状态有以下5种：

- ok：温度正常
- nc： non-critical，温度偏高（或者偏低），但是并不太严重
- cr：critical，温度太高或者温度太低，很严重
- nr： non-recovery，温度太高或者温度太低，造成不可恢复的损伤。
- na：温度状态不明，比较少见。

注意ok –> nc –> cr –> nr 从正常，到越来越严重的温度问题。

# 如何触发温度告警

上一节我们介绍了nc cr 和nr三种状态，都说温度偏高或者温度偏低，那么

- 温度到什么程度状态会变成nc，
- 温度到什么程度会变成cr
- 温度到什么程度会变成nr

显然，各个元件的状态改变是有温度门限值的，我们可以通过如下方法查看：

```
root@node244:~# ipmitool sensor get "CPU1 Temp"
Locating sensor record...
Sensor ID              : CPU1 Temp (0x1)
 Entity ID             : 3.1 (Processor)
 Sensor Type (Threshold)  : Temperature (0x01)
 Sensor Reading        : 29 (+/- 0) degrees C
 Status                : ok
 Nominal Reading       : 40.000
 Normal Minimum        : -4.000
 Normal Maximum        : 89.000
 Upper non-recoverable : 90.000
 Upper critical        : 90.000
 Upper non-critical    : 85.000
 Lower non-recoverable : 0.000
 Lower critical        : 0.000
 Lower non-critical    : 0.000
 Positive Hysteresis   : 2.000
 Negative Hysteresis   : 2.000
 Minimum sensor range  : Unspecified
 Maximum sensor range  : Unspecified
 Event Message Control : Per-threshold
 Readable Thresholds   : lnr lcr lnc unc ucr unr 
 Settable Thresholds   : lnr lcr lnc unc ucr unr 
 Threshold Read Mask   : lnr lcr lnc unc ucr unr 
 Assertion Events      : 
 Assertions Enabled    : ucr+ 
 Deassertions Enabled  : ucr+ 
```

从上面的信息可以看出：

- Upper non-critical 85 度
- Upper critical 90 度
- Upper non-recovery 90 度
- Lower non-critical 0 度
- Lower critical 0 度
- Lower non-recoverable

有了门限值，是哪种状态就比较简单了。

- [0,85)之间是，状态ok
- [85,90) 状态为nc
- [90,) 状态为nr （因为cr的门限和nr的门限都是90，状态取nr）

低温的情况也是类似。

如何让温度状态告警呢，即变成nc或者cr或者nr状态呢？

ipmitool提供了方法来设置各个状态的门限值。

```
ipmitool -I open sensor thresh 'CPU2 Temp' upper 20 30 90
```

上述指令的意思是将CPU2 Temp元件的告警门限中的温度上限告警门限设置为20 30 和90.

以为CPU的温度是33度左右，我们可以通过如下指令，将状态变为nc：

```
root@node244:~# ipmitool -I open sensor thresh 'CPU2 Temp' upper 20 40 90
Locating sensor record 'CPU2 Temp'...
Setting sensor "CPU2 Temp" Upper Non-Critical threshold to 20.000
Setting sensor "CPU2 Temp" Upper Critical threshold to 40.000
Setting sensor "CPU2 Temp" Upper Non-Recoverable threshold to 90.000

root@node244:~# ipmitool sensor list
CPU2 Temp        | 33.000     | degrees C  | nc    | 10.000    | 10.000    | 10.000    | 20.000    | 40.000    | 90.000
```

33摄氏度，超过了20度，但是没要超过40度，因此状态是nc，即non-critical。

同样道理，我们将告警门限设置为 20 30 90的话，就会发现状态为cr，即critical：

```
root@node244:~# ipmitool -I open sensor thresh 'CPU2 Temp' upper 20 30 90
Locating sensor record 'CPU2 Temp'...
Setting sensor "CPU2 Temp" Upper Non-Critical threshold to 20.000
Setting sensor "CPU2 Temp" Upper Critical threshold to 30.000
Setting sensor "CPU2 Temp" Upper Non-Recoverable threshold to 90.000
root@node244:~# ipmitool sensor list 
CPU1 Temp        | 30.000     | degrees C  | ok    | 0.000     | 0.000     | 0.000     | 85.000    | 90.000    | 90.000    
CPU2 Temp        | 33.000     | degrees C  | cr    | 10.000    | 10.000    | 10.000    | 20.000    | 30.000    | 90.000    
```

同样道理，可以将状态变成nr，只需要设置门限为20 30 30 ，即可，不在赘述。