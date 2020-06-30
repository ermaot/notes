## 简述
- B+树是1970年Rudolf Bayer教授在《Organization and Maintenance of Large Ordered Indices》一文中提出的
- 它采用多叉树结构，降低了索引结构的深度，避免传统二叉树结构中绝大部分的随机访问操作，从而有效减少了磁盘磁头的寻道次数，降低了外存访问延迟对性能的影响。
- 它保证树节点中键值对的有序性，从而控制search/insert/delete/update操作的时间复杂度在O(log(n))的范围内。
- 鉴于上述优势，B+树作为索引结构的构建模块，被广泛应用在大量数据库系统和存储系统中，其中就包括MySQL。

## B+树的数据结构及基础操作
![image](pic/B%2B%E6%A0%91%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6%E6%9C%BA%E5%88%B6.png)
一棵传统的B+树需要满足以下几点要求：
- 从根节点到叶节点的所有路径都具有相同的长度
- 所有数据信息都存储在叶节点上，非叶节点仅作为叶节点的索引存在
- 根结点至少拥有两个键值对
- 每个树节点最多拥有M个键值对
- 每个树节点（除了根节点）拥有至少M/2个键值对

一棵传统的B+需要支持以下操作：
- 单键值操作：Search/Insert/Update/Delete
- 范围操作：Range Search

## B+树并发控制机制的基本要求
正确的B+树并发控制机制需要满足以下几点要求：
- 正确的读操作：
1. R.1 不会读到一个处于中间状态的键值对：读操作访问中的键值对正在被另一个写操作修改
2. R.2 不会找不到一个存在的键值对：读操作正在访问某个树节点，这个树节点上的键值对同时被另一个写操作（分裂/合并操作）移动到另一个树节点，导致读操作没有找到目标键值对
- 正确的写操作：W.1 两个写操作不会同时修改同一个键值对
- 无死锁：D.1 不会出现死锁：两个或多个线程发生永久堵塞（等待），每个线程都在等待被其他线程占用并堵塞了的资源

不管B+树使用的是基于锁的并发机制还是Lock-Free的并发机制，都必须满足上述需求。

## B+树并发控制机制
简写|说明
---|---
SL (Shared Lock)|共享锁 — 加锁
SU (Shared Unlock) |共享锁 — 解锁
XL (Exclusive Lock) |互斥锁 — 加锁
XU (Exclusive Unlock)| 互斥锁 — 解锁
SXL (Shared Exclusive Lock) |共享互斥锁 — 加锁
SXU (Shared Exclusive Unlock)|共享互斥锁 — 解锁
R.1/R.2/W.1/D.1| 并发机制需要满足的正确性要求

- safe nodes：判断依据为该节点上的当前操作是否会影响祖先节点。以传统B+树为例：
1. 对于插入操作，当键值对的数量小于M时，插入操作不会触发分裂操作，该节点属于safe node；反之当键值对数量等于M时，该节点属于unsafe node；
2. 对于删除操作，当键值对的数量大于M/2时，不会触发合并操作，该节点属于safe node；反之当键值对数量等于M/2时，该节点属于unsafe node。
3. 对于MySQL而言，一个节点是否是安全节点取决于键值对的大小和页面剩余空间大小等多个因素，详细代码可查询MySQL5.7的btr_cur_will_modify_tree()函数

#### 基础并发控制机制（MySQL5.6）
MySQL5.6以及之前的版本采用了一种较为基础的并发机制：它采用了两种粒度的锁：
1. index粒度的S/X锁：用来控制对索引的树结构访问及修改操作的冲突
2. page粒度的S/X锁（本文等同于树节点粒度）：用来控制对数据页访问及修改操作的冲突。

##### 读操作
```
/* Algorithm1. 读操作 */
1.   SL(index)
2.   Travel down to the leaf node
3.   SL(leaf)
4.   SU(index)
5.   Read the leaf node
6.   SU(leaf)
```
- 读操作通过index的S锁，避免在访问到树结构的过程中树结构被其它写操作所修改，从而满足R.2的正确性要求
- 读操作到达叶节点后先申请叶节点页的锁，再释放index的锁，从而避免在访问具体的键值对信息时数据被其它写操作所修改，满足R.1的正确性要求
- 由于读操作在访问树结构的过程中对B+树加的是S锁，所以其它读操作可以并行访问树结构，减少了读-读操作之间的并发冲突

##### 写操作
###### 悲观写
```
/* Algorithm2. 悲观写操作 */
1.   XL(index)
2.   Travel down to the leaf node
3.   XL(leaf)   /* lock prev/curr/next leaves */
4.   Modify the tree structure 
5.   XU(index)  
6.   Modify the leaf node 
7.   XU(leaf)
```
- 每个写操作首先对B+树加X锁（step1），从而阻止了其它读写操作在这个写操作执行过程中访问B+树，避免它们访问到一个错误的中间状态。
- 然后，它遍历树结构直到对应的叶节点（step2），并对叶节点的page加X锁（step3）
- 接着，它判断该操作是否会引发split/merge等修改树结构的操作。如果是，它就修改整个树结构（step4）后再释放index的锁（step5）。
- 最后，它在修改叶节点的内容（step6）后释放了叶节点的X锁（step7）。
- 写操作在访问树结构的过程中**对B+树加的是X锁，会堵塞其它的读/写操作**

##### 乐观写
```
/* Algorithm3. 乐观写操作 */
1.   SL(index)
2.   Travel down to the leaf node
3.   XL(leaf)
4.   SU(index)
5.   Modify the leaf node
6.   XU(leaf)
```
- 每一个树节点页可以容纳大量的键值对信息，所以B+树的写操作在多数情况下并不会触发split/merge等修改树结构的操作，所以假设大部分写操作并不会修改树结构
- B+树往往优先执行乐观写操作，只有乐观写操作失败才会执行悲观写操作
- MySQL5.6采用的是“从上到下，从左到右”的加锁顺序，不会出现两个线程加锁顺序成环的现象，所以不会出现死锁的情况

#### 只锁住被修改的分支
是否存在一种并发机制，它只锁住B+树中被修改的分支，而不是锁住整个树结构呢？**是**

MySQL5.7修改树结构时，写操作不再只是粗暴地对整个索引结构加锁，而**只对修改的分支加锁**

##### 读操作
```
/* Algorithm4. 读操作 */
1.   current <= root
2.   SL(current) 
3.   While current is not leaf do {
4.     SL(current->son)
5.     SU(current)
6.     current <= current->son
7.   }
8.   Read the leaf node 
9.   SU(current)
```
- 读操作从根节点出发，首先持有根节点的S锁（step1-2）。
- 在（step3-7）的过程中，读操作先获得子节点的S锁，再释放父节点的S锁，这个过程反复执行直到找到某个叶节点
- 最后，它在读取叶节点的内容（step8）后释放了叶节点的S锁（step9）。
- 因为读操作在持有子节点的锁后才释放父节点的锁，所以不会读到一个正在修改的树节点，不会在定位到某个子节点后子节点的键值对被移动到其它节点，因此能满足R.1/R.2的正确性要求

##### 写操作
```
/* Algorithm5. 写操作 */
1.   current <= root
2.   XL(current)
3.   While current is not leaf do {
4.      XL(current->son)
5.      current <= current->son
6.      If current is safe do {
7.         /* Unlocked ancestors on stack. */
8.         XU(locked ancestors)
9.      }     
10.  }
11.  /* Already lock the modified branch. */
12.  Modify the leaf and upper nodes 
13.  XU(current) and XU(locked ancestors) 
```
- 写操作同样从根节点出发，首先持有根节点的X锁（step1-2）。
- 在step3到step10的过程中，写操作先获得子节点的X锁，然后判断子节点是否是一个安全节点（操作会引起该节点的分裂/合并等修改树结构的操作）。如果子节点是安全节点，写操作立即释放祖先节点（可能包含多个节点）的X锁，否则就会暂时保持父节点的锁，这个过程反复执行直到找到某个叶节点。
- 当到达了叶节点后，写操作就已经持有了修改分支上所有树节点的X锁，从而避免其它读/写操作访问该分支（step11），满足W.1的正确性要求。
- 最后，它在修改这个分支的内容（step12）后释放了分支的锁（step13）
- 死锁问题，同样采用的是“从上到下”的加锁顺序，满足D.1的正确性要求

#### SX锁
- Algorithm 6与在Algorithm 5十分相似，主要的区别在于，Algorithm 6在step2，4，8中使用SX锁取代了X锁。到达某个叶节点后，它再将修改分支上的SX锁升级为X锁
- 在写操作将影响分支上的锁升级为X锁前，所有读操作都可以访问被这个写操作访问过的非叶节点，从而减少了线程之间的冲突。
- 由于SX锁的存在，不会出现多个写操作修改同一个分支的情况，从而满足了W.1的正确性要求


#### 加锁分析
- 频繁加锁操作在多核处理器上会产生Coherence Cache Miss过高的问题
- 当系统中存在大量cache coherence miss时，势必会提高处理器/总线的资源消耗，产生较高的延迟开销，这在内存数据库/内存系统中更是一个不可忽略的问题
- 自顶向下的加锁策略，在安全地获取到子节点的锁后释放父节点的锁。然而我们很容易发现，这种加锁方式依然是十分悲观的

#### Blink树：简单有效的多线程B+树
- **Blink树假设访问树节点的读写操作是原子性的**，读操作不会读到写操作修改到一半的状态（即已满足R.1正确性要求），但写操作之间修改同一份数据时会出现冲突
- **Blink树提出为每一个树节点配置一个右指针**，它的出现使得在无锁状态下自顶向下的访问策略+自底向上的加锁策略成为可能

##### 读操作

```
/* Algorithm7. 读操作 */
1.   current <= root
2.   While current is not leaf do {
3.      current <= scanNode(current, v)
4.      current <= current->son
5.   }
6.   /* Keep move right if necessary. */
7.   /* Deal with the leaf node. */
/*  scanNode函数 */
8.  Func scanNode(node* t, int v) {
9.     If t->next->key[0] <= v do 
10.       t <= scanNode(t->next, v)
11.   return t;
12. }
```
- Blink树规定树的分裂操作顺序必然是从左至右，因此目标键值对只有可能被分裂到子节点的右兄弟节点
- Blink树提出为每一个树节点配置一个右指针，通过该指针访问右节点
- 读操作会判断子节点的右兄弟节点的最小值是否大于它正在查找的目标键值，如果不是说明目标键值对在右兄弟节点或者更右边的节点，指针就会往右走，直到找到某个右兄弟节点的最小值大于目标键值


##### 删除操作
当发生删除操作时，它采用index粒度的X锁，堵塞其它读/写操作

##### 写操作

```
/* Algorithm8. 写操作 */
1.   current <= root                                  
2.   While current is not leaf do {             
3.      current <= scannode(current, v)     
4.      stack <= current                           
5.      current <= current->son                 
6.   }                                                          
7.   XL(current)   /* lock the current leaf */ 
8.   moveRight(current)                             
9.   DoInsertion:                                        
10.  If current is safe do                                       
11.    insert(current) and XU(current)              
12.  else {
13.    allocate(next)
14.    shift(next) + link(next)
15.    modify(current)
16.    oldnode <= current
17.    current <= pop(stack)
18.    XL(current)
19.    moveRight(current) 
20.    XU(oldnode)
21.    goto DoInsertion; 
22. } 
```
- 写操作使用和读操作类似的方式定位到目标叶节点current并加锁（step1-8）。
- 为了支持自底向上加锁，写操作遍历过程中将访问到的树节点压入栈stack中。如果叶节点是安全节点，直接插入后释放锁就可以了（step10-11）。如果叶节点不是安全节点，就分配一个新的next节点，将叶节点的数据移动到next节点，修改current节点并将右指针指向next节点（step13-15）。
- 然后，写操作从栈中弹出上一层的父节点并加锁（step16-18）。由于父节点也可能被分裂，所以也需要通过moveRight函数移动到正确的上一层节点（step19），然后重复上述的DoInsertion过程
- 。moveRight与scanNode相似，主要的区别在于前者是在加锁状态下向右走，拿到右节点的锁后可释放当前结点的锁。
- 写操作通过树节点粒度的锁，避免了多个写操作同时修改同一个树节点，满足W.1的正确性要求。
- 对于死锁，由于Blink树只支持“自左向右，自底向上”加锁的策略，所以不会出现死锁的问题

##### Blink树的问题
Blink树有效减少了加锁频率，但是它依然存在两个问题：
1. 不实际的假设：读写树节点的操作是原子性的；
2. 删除操作竟然需要锁住整个索引结构，效率太差。

#### OLFIT树

```
/* Algorithm9. 树节点的写操作 */
1.   XL(current)
2.   Update the node content
3.   INCREASE(version)
4.   XU(current)
```
---

```
/* Algorithm10. 树节点的读操作 */
1.   RECORD(version)
2.   Read the node content
3.   If node is lock, go to step1
4.   If version differs, go to step1
```

#### Masstree，B+树优化机制的集大成者
Masstree融合了大量B+树的优化策略，包括单线程场景下和多线程场景下的
##### 单线程场景
- 采用了B+树 + Tri树的混合数据结构。
- 在传统B+树中，为了支持变长键值这一场景，B+树要么在树节点中预留很大的键值空间，然而这会导致存储空间的浪费。还有一种方式就是B+树采用定长指针指向变长键值，这节约了存储空间，负面效果就是存在大量指针访问，可能导致处理器缓存命中率的严重降低，影响索引结构的性能。
- Masstree提出B+树和Tri树的混合数据结构，将变长键值分为多个固长的部分，固长部分通过B+树组织，多个固长部分间的关系通过Tri树组织，取得了在空间利用率+性能两者间的平衡。
- 基于int类型的比较。Masstree将变长键值划分成多个固长部分，每个固长部分可以通过int类型表示，而不是char类型。由于处理器处理int类型比较操作的速度远远快于char数组的比较，因此Masstree通过int类型的比较进一步加速了查找过程。
- 预取指令。对于B+树从根节点到叶节点的遍历操作，绝大部分延迟是访存延迟（对于基于外存的B+树，则是外存延迟）造成的，所以Masstree通过预取指令减少访存延迟

##### 多线程场景
- 双向链表：Masstree通过维护双向链表，可以在树节点粒度的锁基础上实现并发删除操作。
- 消除不必要的版本号变化，减少重做开销
1. 在OLFIT树中，任何一个树节点的操作都会导致树节点的版本号发生变化，这会导致同时访问该节点的读操作重做。
2. 然而，有一部分树节点的写操作并不会导致读操作读到一个错误的状态，所以不需要改变版本号：对于更新操作，在8B范围内的更新操作是原子性的；超过8B范围的更新操作也可以做成原子性的，即用指针指向超过8B范围的数据，更新操作只需要修改8B的指针就可以了；
3. 对于插入操作，传统B+树的插入操作往往会导致键值对的重排序，这需要通过版本号的变化通知读操作可能读到不一致的状态。而在Masstree中，它通过8B的permutation维护树节点键值对的有序性，避免传统B+树中键值对排序的操作，但是每个树节点最多只能容纳15个键值对



#### Mysql5.7的B+树并发控制机制
从MySQL5.6版本升级到5.7版本的过程中，B+树的并发机制发生了比较大的变化，主要包括以下几点：
1. 引入了SX锁
2. 写操作尽可能只锁住修改分支，减少加锁的范围

##### 读操作
```
/* Algorithm12. 读操作 */
1.   SL(index)
2.   While current is not leaf do {
3.      SL(non-leaf)
4.   }
5.   SL(leaf)
6.   SU(non-leaf)
7.   SU(index)
8.   Read the leaf node
9.   SU(leaf)
```
- 每个读操作首先对树结构加S锁（step1），其次访问树结构直到对应的叶节点（step2-4）。这里与5.6不同之处在于，读操作对经过的所有叶节点加S锁。
- 接着，它对叶节点的page加S锁（step5）后释放了索引结构和非叶节点的S锁（step6-7）。
- 最后，它访问叶节点的内容（step8），释放了叶节点的锁（step9）。
- 显而易见，读操作能满足R.1/R.2的正确性要求

##### 写操作
- 乐观写
```
/* Algorithm13. 乐观写操作 */
1.   SL(index)
2.   While current is not leaf do {
3.      SL(non-leaf)
4.   }
5.   XL(leaf)
6.   SU(non-leaf)
7.   SU(index)
8.   Modify the leaf node
9.   XU(leaf)
```

- 悲观写
```
/* Algorithm14. 悲观写操作 */
1.   SX(index) 
2.   While current is not leaf do {
3.      XL(modified non-leaf)
4.   }
5.   XL(leaf)      /* lock prev/curr/next leaf */
6.   Modify the tree structure 
7.   XU(non-leaf)
8.   SXU(index) 
9.   Modify the leaf node 
10.  XU(leaf)
```

- MySQL中的索引结构已经不再是一棵普通的B+树，它需要支持spatial index这样更加复杂的索引结构，需要在history list过大的时候优先支持purging线程，存在需要锁住左-当前节点-右节点这样的情况，所以它依赖索引的S/SX/X锁来避免有两个写操作同时修改树结构。
- 它还需要支持类似modify_prev/search_prev等相对复杂的回溯操作，所以需要对非叶节点加锁，避免被其它操作所修改。
- 并且，一个实例可能存在多个聚集索引和二级索引，MySQL中B+树考虑的情况变得十分复杂