# Python GIL全局解释器锁

Python 引进 GIL，可以最大程度上规避类似内存管理这样复杂的竞争风险问题。

## Python GIL底层实现原理

![GIL工作流程示意图](pic/python的GIL/2-1ZS012105L23.gif)
图 1 GIL 工作流程示意图


上面这张图，就是 GIL 在 Python 程序的工作示例。其中，Thread 1、2、3 轮流执行，每一个线程在开始执行时，都会锁住 GIL，以阻止别的线程执行；同样的，每一个线程执行完一段后，会释放 GIL，以允许别的线程开始利用资源。

读者可能会问，为什么 Python 线程会去主动释放 GIL 呢？毕竟，如果仅仅要求 Python 线程在开始执行时锁住 GIL，且永远不去释放 GIL，那别的线程就都没有运行的机会。其实，CPython 中还有另一个机制，叫做间隔式检查（check_interval），意思是 CPython 解释器会去轮询检查线程 GIL 的锁住情况，每隔一段时间，Python 解释器就会强制当前线程去释放 GIL，这样别的线程才能有执行的机会。

注意，不同版本的 Python，其间隔式检查的实现方式并不一样。早期的 Python 是 100 个刻度（大致对应了 1000 个字节码）；而 Python 3 以后，间隔时间大致为 15 毫秒。当然，我们不必细究具体多久会强制释放 GIL，读者只需要明白，CPython 解释器会在一个“合理”的时间范围内释放 GIL 就可以了。

整体来说，每一个 Python 线程都是类似这样循环的封装，来看下面这段代码：

```
for (;;) {    if (--ticker < 0) {        ticker = check_interval;           /* Give another thread a chance */        PyThread_release_lock(interpreter_lock);        /* Other threads may run now */           PyThread_acquire_lock(interpreter_lock, 1);    }    bytecode = *next_instr++;    switch (bytecode) {        /* execute the next instruction ... */    }}
```

从这段代码中可以看出，每个 Python 线程都会先检查 ticker 计数。只有在 ticker 大于 0 的情况下，线程才会去执行自己的代码。

## Python GIL不能绝对保证线程安全

注意，有了 GIL，并不意味着 Python 程序员就不用去考虑线程安全了，因为即便 GIL 仅允许一个 Python 线程执行，但别忘了 Python 还有 check interval 这样的抢占机制。

比如，运行如下代码：

```
import threadingn = 0def foo():    global n    n += 1threads = []for i in range(100):    t = threading.Thread(target=foo)    threads.append(t)for t in threads:    t.start()for t in threads:    t.join()print(n)
```

执行此代码会发现，其大部分时候会打印 100，但有时也会打印 99 或者 98，原因在于 n+=1 这一句代码让线程并不安全。如果去翻译 foo 这个函数的字节码就会发现，它实际上是由下面四行字节码组成：

\>>> import dis
\>>> dis.dis(foo)
LOAD_GLOBAL       0 (n)
LOAD_CONST        1 (1)
INPLACE_ADD
STORE_GLOBAL       0 (n)

而这四行字节码中间都是有可能被打断的！所以，千万别以为有了 GIL 程序就不会产生线程问题，我们仍然需要注意线程安全。