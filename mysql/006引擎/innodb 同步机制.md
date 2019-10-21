## 相关文件
- 主要文件：sync0*.*.

文件名称 1 | 代码行数 | 说明
---|---|---
sync0rw.cc | 497|读/写锁的实现
sync0rw.h  | 471|
sync0sync.c | 1273|
sync0sync.cc | 260|互斥量的实现
Sync0type.h | 16|相关定义

## 基础知识
- innodb 中mutex对象使用mutual exclusion实现互斥操作
- linux下可以使用spin lock 、semephore 、monitor 、sequencer 互斥操作
- innodb 封装mutex数据结构，与spin lock 相似，并扩充优化
- UNIV_ SYNC_ DEBUG模式下其可以进行AB-BA死锁检测
- 通过WaitArray来解决高并发情况下的Thread Thrashing（线程颠簸？） 问题
#### memory model
- memory model 决定了如何访问内存，以及并发情况下cpu之间的相互影响
- memory model 不包括虚拟地址转换（virtual address translation）
- memory model 关注的是cpu 与内存之间的数据传输
- 最简单的模型是顺序内存模型（sequence memory model），即 strong ordering
- 两种waek ordering：total store memory model， partial store memory model
#### mutual exclusion
- 顺序内存模型在多个cpu访问情况下，访问顺序依然不确定，导致race condition（竞争条件）
- 指令序列称为临界区（critical section），操作的数据称为临界资源（critical resources）
- 同一时间，只能一个cpu访问临界区，叫互斥
#### atomic read-modify-write operation
- 原子的 read-modify-write 操作，称为原子总线操作
- 目前cpu都支持 TAS （test-and-set）指令，即从内存取出一个字节或者一个word，与0比较，然后无条件设为1；操作都是原子操作
- 一旦有TAS操作，cpu和IO都不能使用总线；可基于TAS构造spin lock 、 semephore等
- swap_atomic 执行 swap-atomic 硬件指令，可以用来构造 TAS ：首先将寄存器中的值设置为1，然后执行atomic swap，最后和寄存器中的值比较
#### spin lock
- 最简单最广泛的互斥结构
- 用来互斥的代码较少，可快速执行完毕，是 short term mutual exclusion
- spin lock（自旋锁） 使用word（4字节实现），如果对象已经加锁，则在while循环中等待锁释放
- spin lock还有try_lock方法，若对象已经加锁则直接返回
- unlock 使用 memory barrier 技术强制让cpu按照之前定义的顺序提交
#### 死锁
- 两个cpu在等待对方持有的锁，称为AB-BA类型的死锁
- 为了避免AB-BA型死锁，必须按照同样的顺序（==此处描述不准确==）对资源加锁和解锁，即 nested lock
## innodb同步机制
- innodb 有两种同步机制可选：mutex（完全互斥） 和 rw-lock（可给临界资源加 s-latch 或者 x-latch ）
#### mutext
- innodb 自己实现了互斥的数据结构 mutex_struct
- TAS 返回1的时候，先自旋（减少对内存访问，减少占用总线带宽）；如果仍不能得到mutex，则放入wait array
- 不直接放入等待队列，是为了减少上下文切换（context switch）
- 单核cpu自旋无意义；多核cpu可以减少上下文切换次数
#### rw-lock
#### wait array
#### 死锁检测
## 小结