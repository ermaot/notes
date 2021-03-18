mlock锁住内存是为了防止这段内存被操作系统swap掉。此操作风险高，仅超级用户可以执行。

看家族成员：

```
#include <sys/mman.h>
int mlock(const void *addr, size_t len);
int munlock(const void *addr, size_t len);
int mlockall(int flags);
int munlockall(void);
```

系统调用 mlock 家族允许程序在物理内存上锁住它的部分或全部地址空间。这将阻止Linux 将这个内存页调度到交换空间（swap space），即使该程序已有一段时间没有访问这段空间。

一个严格时间相关的程序可能会希望锁住物理内存，因为内存页面调出调入的时间延迟可能太长或过于不可预知。安全性要求较高的应用程序可能希望防止敏感数据被换出到交换文件中，因为这样在程序结束后，攻击者可能从交换文件中恢复出这些数据。

锁定一个内存区间只需简单将指向区间开始的指针及区间长度作为参数调用 mlock。linux 分配内存到页(page)且每次只能锁定整页内存，被指定的区间涉及到的每个内存页都将被锁定。getpagesize 函数返回系统的分页大小，在 x86 Linux 系统上，这个值是 4KB

示例程序：（锁定1M内存。由于如果只分配不写入，系统是不会触发COW，所以还要写入才能真正mlock）

```
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include <sys/mman.h>

int main()
{
        const int alloc_size =  1024 * 1024;
        char* memory = malloc (alloc_size);
        mlock (memory, alloc_size);
        size_t i;
        size_t page_size = getpagesize ();
        for (i = 0; i < alloc_size; i += page_size)
        memory[i] = 0;
        sleep(10);
        munlock(memory, alloc_size);
        free(memory);
        return 0;
}
```

编译

```
gcc mlock.c  -o mlock
```

先查看mlock内存

```
# cat /proc/meminfo | grep -i mlock
Mlocked:            7936 kB

```

执行

```
# ./mlock
```

同时查看lock住的内存

```
# cat /proc/meminfo | grep -i mlock
Mlocked:            8964 kB

```

等于8964-7936 = 1028，与1M很接近

mlock执行完毕之后再查看mlock住的内存

```
# cat /proc/meminfo | grep -i mlock
Mlocked:            7936 kB

```

可以看到释放掉了