## 链表的python实现
- 单向链表
```
In [1]: class Node:
   ...:     def __init__(self,value,next=None):
   ...:         self.value = value
   ...:         self.next = next
   ...:

In [2]: L = Node('a',Node('b',Node('c',Node('d',))))

In [3]: L.next.next.next.value
Out[3]: 'd'
```

- 双向链表

```
In [6]: class Node_twoway:
   ...:     def __init__(self,value,next=None,prev = None):
   ...:         self.value = value
   ...:         self.next = next
   ...:         self.prev = prev
   ...:

In [7]: L_T1 = Node_twoway('a')

In [8]: L_T2 = Node_twoway('b')

In [9]: L_T3 = Node_twoway('c')

In [10]: L_T4 = Node_twoway('d')

In [11]: L_T1.next = L_T2

In [12]: L_T2.next = L_T3

In [13]: L_T2.prev = L_T1

In [14]: L_T3.next = L_T4

In [15]: L_T3.prev = L_T2

In [16]: L_T4.prev = L_T3

In [17]: L_T1.next.next.next.value
Out[17]: 'd'

In [19]: L_T2.prev.value
Out[19]: 'a'
```

## 邻接图

```
In [44]: a ,b,c,d,e,f,g,h=range(8)


In [46]: n=[
    ...: [b,c,d,e,f],
    ...: [c,e],
    ...: [d],
    ...: [e],
    ...: [f],
    ...: [c,g,h],
    ...: [f,h],
    ...: [f,g]
    ...: ]
```
## 加权邻接图

```
In [1]: a ,b,c,d,e,f,g,h=range(8)

In [2]: N=[
   ...: {b:2,c:1,d:3,e:9,f:4},
   ...: {c:4,e:3},
   ...: {d:8},
   ...: {e:7},
   ...: {f:5},
   ...: {c:2,g:2,h:2},
   ...: {f:1,h:6},
   ...: {f:9,g:8}
   ...: ]
   
   //另一种表示法
N={
'a':set('bedf'),
'b':set('ce'),
'c':set('d'),
'd':set('e'),
'e':set('f'),
'f':set('cgh'),
'g':set('fh'),
'h':set('fg')
}
```
## 邻接矩阵


## 二叉树

```
In [3]: class tree:
   ...:     def __init__(self,left,right):
   ...:         self.left = left
   ...:         self.right = right
   ...:

In [4]: t = tree(tree('a','b'),tree('c','d'))

//上面是书上样例，但没有专门的数值项，所以个人认为下一种更好
In [9]: class Tree:
   ...:     def __init__(self,value,left=None,right=None):
   ...:         self.value = value
   ...:         self.left = left
   ...:         self.right = right
   ...:

In [10]: t1 = Tree(1,Tree(2),Tree(3))

In [11]: t1.value
Out[11]: 1

In [12]: t1.left.value
Out[12]: 2
```
