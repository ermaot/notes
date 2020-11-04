## 一、勿将size_t与真实类型混用

```
unsigned int x;
size_t  y;
x = y;
```

由于size_t在不同的平台上，可能代表很不同的类型（unsigned int、 unsigned long int或者 unsigned long long int），所以可能会导致数据截断

## 二、数据比较时类型转换带来的陷阱

```
#include<stdio.h>
int main(void){
    int a = -1;
    unsigned int b = 100;
    if(a > b){
    printf("a=%d",a);
    }
}
```

这个程序会输出什么？会输出a=-1。因为a是有符号数，而b是无符号数，在比较的时候会发生数据转换导致a成为一个很大的值，所以会输出a

## 三、整数溢出

```
#include<stdio.h>
int main(void){
        unsigned int a = 1073741824;
        int b;
        printf("%d",a);
        b = a * sizeof(int);
        printf("%d",b);
}
```

输出结果为0