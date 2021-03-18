对一般的编程语言来说，if-else语句成对出现，if语句比else要多，也就是出现了if可以不出现else，一个else只能针对一个if。但python则不一样，else的语法比较特殊，一个else可以针对于多个if。也就是else在循环正常结束或者循环条件不成立时被执行。举个例子：

```
def prime(n):
	for i in range(2,n):
		for j in range(2,i):
			if i%j == 0:
				break
		else:
			print("%d is a prime number"%i)
```

该else表示，第二个for执行结束，并且不满足i%j==0时打印print

else如果放在如下位置：

```
def prime(n):
    for i in range(2,n):
        for j in range(2,i):
            if i%j == 0:
                break
    else:
        print("%d is a prime number"%i)
```

如果放此处，则只会打印最后一个素数

python的异常处理还有一种try-except-else-finally形式。比如打开数据库-执行语句-如果正常结束则commit-如果异常就rollback这个过程，如果只有try-except-finally，则正常结束这个过程只能写try里如果异常则try后面的commit代码不执行，但如果有else，则commit可以写进else中，显得更加清晰

