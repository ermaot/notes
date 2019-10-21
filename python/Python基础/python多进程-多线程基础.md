## 多线程模块thread
Python可以通过两个标准库(thread,threading)提供了对多线程的支持。thread模块已被废弃。用户可以使用threading模块代替。所以，在Python3中不能再使用”thread”模块。为了兼容性，Python3 将thread重命名为“_thread”。

```
>>> import thread
>>> from time import sleep, ctime
>>> loops = [4,2]
>>> def loop0():
...     print( 'start loop 0 at:', ctime())
...     sleep(4)
...     print ('loop 0 done at:', ctime())
... 
>>> def loop1():
...     print ('start loop 1 at:', ctime())
...     sleep(2)
...     print( 'loop 1 done at:', ctime())
... 
>>> def main():
...     print( 'start:', ctime())
...     thread.start_new_thread(loop0, ())
...     thread.start_new_thread(loop1, ())
...     sleep(6)
...     print( 'all end:', ctime())
... 
>>> if __name__ == '__main__':
...     main()
... 
('start:', 'Wed Aug 14 21:39:00 2019')
('start loop 1 at:', 'Wed Aug 14 21:39:00 2019')
('start loop 0 at:', 'Wed Aug 14 21:39:00 2019')
('loop 1 done at:', 'Wed Aug 14 21:39:02 2019')
('loop 0 done at:', 'Wed Aug 14 21:39:04 2019')
('all end:', 'Wed Aug 14 21:39:06 2019')
```

## 多线程模块threading

```
import threading
import time
 
class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        print("Starting " + self.name)
        # 获得锁，成功获得锁定后返回True
        # 可选的timeout参数不填时将一直阻塞直到获得锁定
        # 否则超时后将返回False
        threadLock.acquire()
        print_time(self.name, self.counter, 3)
        # 释放锁
        threadLock.release()
 
def print_time(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print ("%s: %s" % (threadName, time.ctime(time.time())))
        counter -= 1
 
threadLock = threading.Lock()
threads = []
 
# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)
 
# 开启新线程
thread1.start()
thread2.start()
 
# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)
 
# 等待所有线程完成
for t in threads:
    t.join()
print( "Exiting Main Thread")
```

## multiprocessing
由于 GIL 的存在，在CPU密集型的程序当中，使用多线程并不能有效地利用多核CPU的优势，因为一个解释器在同一时刻只会有一个线程在执行。所以，multiprocessing 模块可以充分的利用硬件的多处理器来进行工作。多进程相当于开启多个python解释器，每个解释器对应一个进程。