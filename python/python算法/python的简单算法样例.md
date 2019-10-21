## 兔子数列

```
In [1]: def rabbit(n):
   ...:     if(n==1):
   ...:         return 1
   ...:     elif(n==2):
   ...:         return 1
   ...:     else:
   ...:         return rabbit(n-2)+ rabbit(n-1)
   ...:

In [2]: for i in range(1,15):
   ...:     print(rabbit(i))
   ...:
1
1
2
3
5
8
13
21
34
55
89
144
233
377
```
## 侏儒排序
算法过程：
1. 序列从头开始，如果当前值小于后面值，则下表加1
2. 如果当前值大于后面值，则交换，同时下标减一
3. 重复上述过程
该算法只需要一个循环，但由于会前进后退前进后退，所以平均复杂度类似于冒泡O(n^2)
```
In [1]: def gnomesort(list):
   ...:     i = 0
   ...:     while i < len(list):
   ...:         if( i ==0 or list[i-1] <= list[i]):
   ...:             i +=1
   ...:         else:
   ...:             list[i-1],list[i] = list[i],list[i-1]
   ...:             i -= 1
   ...:

In [2]: b = [3,2,1,5,6,9,4]

In [3]: gnomesort(b)

In [4]: b
Out[4]: [1, 2, 3, 4, 5, 6, 9]
```


## 归并排序
步骤说明
1. 把序列分为左右两边
2. 左右两边的对应元素做比较，大的放左边，小的放右边
3. 然后对左右两边的序列做上面1，2步的递归操作
```
In [1]: def mergesort(seq):
   ...:     mid = len(seq)//2
   ...:     left ,right = seq[:mid],seq[mid:]
   ...:     if len(left) > 1:left = mergesort(left)
   ...:     if len(right) > 1:right = mergesort(right)
   ...:     res = []
   ...:     while left and right:
   ...:         if left[-1] >= right[-1]:
   ...:             res.append(left.pop())
   ...:         else:
   ...:             res.append(right.pop())
   ...:     res.reverse()
   ...:     return (left or right ) + res
   ...:

In [2]: mergesort([5,6,4,3,9,12,10,3])
Out[2]: [3, 3, 4, 5, 6, 9, 10, 12]
```

## 一个已排序列表 ，插入一个元素仍保持顺序

```
In [1]: def find(seq,e):
   ...:     if(seq[0]>e):
   ...:         seq.insert(0,e)
   ...:         return seq
   ...:     if(seq[-1]<e):
   ...:         seq.append(e)
   ...:         return seq
   ...:     start,end  =0,len(seq)-1
   ...:     while True:
   ...:         i = (start + end) //2
   ...:         if(e>seq[i] and e >seq[i+1]):
   ...:             start = i
   ...:         elif(e<seq[i] and e<seq[i+1]):
   ...:             end = i
   ...:         elif(e>=seq[i] and e<=seq[i+1]):
   ...:             break
   ...:     left = seq[:i+1]
   ...:     left.append(e)
   ...:     left.extend(seq[i+1:])
   ...:     return left
   ...:

In [2]: print(find([1,2,3,4,5,6,7,8],9))
[1, 2, 3, 4, 5, 6, 7, 8, 9]

In [3]: print(find([1,2,3,4,5,6,7,8],1))
[1, 1, 2, 3, 4, 5, 6, 7, 8]

In [4]: print(find([1,2,3,4,5,6,7,8],6))
[1, 2, 3, 4, 5, 6, 6, 7, 8]
```
## 插入排序
递归版
```
In [1]: def insert_sort_r(seq,i):
   ...:     if(i==0):return
   ...:     insert_sort_r(seq,i-1)
   ...:     j=i
   ...:     while j>0 and seq[j-1] > seq[j]:
   ...:         seq[j-1],seq[j] = seq[j],seq[j-1]
   ...:         j -=1
   ...:

In [2]: a = [1,5,6,4,3,15,12,13]

In [3]: insert_sort_r(a,7)

In [4]: a
Out[4]: [1, 3, 4, 5, 6, 12, 13, 15]

In [5]:

```

迭代版   //感觉像冒泡排序？？

```
In [1]: def insert_sort(seq):
   ...:     for i in range(len(seq)):
   ...:         for j in range(i,0,-1):
   ...:             if seq[j-1] > seq[j]:
   ...:                 seq[j-1] ,seq[j] = seq[j],seq[j-1]
   ...:

In [2]: a = [56,54,23,87,19,4,0,12]

In [3]: insert_sort(a)

In [4]: a
Out[4]: [0, 4, 12, 19, 23, 54, 56, 87]

```
