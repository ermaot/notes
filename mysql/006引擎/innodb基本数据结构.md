## 相关文件
- dyn0dyn.* 、 fut0*.* 、 ha0ha.* 、hash0hash.* 、mem0.*、 ut0.*共8164行
![与基本算法相关的文件](B38FC769F72B43DBB8D4A60AFD62E258)
## 内存管理系统
- 此处的内存管理不是管理缓存池中的页，而是动态生成的数据库对象
#### 内存管理
###### 内存层次结构
- innodb并未直接使用malloc和free
- innodb使用的是内存堆（memory heap）方式，一次性分配大内存，之后的内存请求可以在innodb内部完成
- 并非按需分配，减少了频繁malloc 和free的性能开销（动态分配，dynamic allocation）
- innodb可以从缓冲池中分配内存建立内存堆，这样更快生成内存页（大小16KB）：缓冲池分配（buffer allocation）
- innodb内存结构分三层，从下到上依次为：系统内存 → （通用内存池，缓冲内存池） → 各种用途的内存堆对象
- 从通用内存池分配，使用mem_area_alloc；从缓冲内存池分配，使用buf_frame_alloc
![内存层次结构示意图](C278E696DD334D03BAAF82883DC42BE2)
###### 内存堆
- 内存堆是innodb最基本也是最重要的内存对象
- 内存堆其实就是一系列相连的内存块
- 内存堆相当于一个栈：通过不断增加内存块以增大内存堆的空间；从栈顶释放内存块
- innodb使用 mem_block_t 表示内存堆中每次从操作系统或者缓冲池中分配的内存块，每个内存块都包含 mem_block_info_t 的元数据信息
#### 通用内存池（==此处有很多笔记需要补充==）
![伙伴系统分配方式](F16121900B4440B585C7834DDE83AF06)
## 哈希表
#### 哈希算法
- 哈希算法复杂度为O(1)
- 哈希表也称散列表，从直接寻址表改造而来
- 哈希算法：利用哈希函数，根据关键字k算出槽(slot)所在位置
- 可能产生冲突（Collision），采用链接法（chaining），即把散列到同一个槽中的元素用链表连接起来
- 哈希算法很多，有加法散列、乘法散列、==除法散列==、全域散列，保证最小冲突
#### 数据结构
- hash_table_struct

变量 | 类型 | 说明
---|---|---
n_cells | ulint | 哈希表中槽的数量
array | hash_cell_t *|  哈希表
n_mutexes | ulint|  保护哈希表段的互斥量个数
mutexes | mutexes_t *|  互斥量
heaps | mem_heap_t  **|  用来分配哈希段中各元胞哈希链的内存堆数组
heap | mem_heap_t *|  用来分配哈希段中各元胞哈希链的内存堆
magi_n | ulint |  魔数，用于调试

- 哈希表创建的时候，必须制定数量，该数量是略大于哈希表元胞个数的素数，方便平均散列（从源码上看，该数尽量离2的幂次远）<p>
==ut_find_prime(n)== 函数寻找略大于n的素数
- 并发控制：缓冲池哈希表的并发由buf_pool_struct互斥量控制，自适应哈希索引由全局读写锁btr_search_latch控制
- 哈希链（该结构中并未定义）:保存在哈希对象中，或者自己定义。缓冲池哈希链在缓冲池的页结构（buf_block_struct）中
- 对不同对象，哈希计算方法稍有不同。基本过程：先对对象fold（buf_page_address_fold,rec_fold），然后使用 hash_calc_hash 映射到槽中
- 哈希表对外提供的接口

函数名 | 说明
---|---
HASH_INSERT(TYPE, NAME, TABLE, FOLD, DATA) | 往哈希表table中插入类型为TYPE的新对象data
HASH_DELETE(TYPE, NAME, TABLE, FOLD, DATA) | 
HASH_GET_FIRST(TABLE, HASH_VAL) |
HASH_GET_NEXT(NAME, DATA) | 
HASH_SEARCH(NAME, TABLE, FOLD, DATA, TEST) |  查找哈希表中名为FOLD的对象，并通过TEST函数检测冲突
HASH_DELETE_AND_COMPACT(TYPE, NAME, TABLE, NODE) | 
HASH_MIGRATE(OLD_TABLE, NEW_TABLE, NODE_TYPE, PTR_NAME, FOLD_FUNC)| 
HASH_GET_N_NODES(TYPE, NAME, TABLE, N) | 获取哈希表中节点数目<p>（==源码中未发现该函数，是否已改名==）
HASH_SEARCH_ALL(NAME, TABLE, TYPE, DATA, ASSERTION, TEST) |  

## 双链表
#### 内存双链表
- 通常意义上的双链表，广泛应用于缓冲池、事务处理、锁、文件系统等模块
- 在ut0lst.h中宏定义，与数据类型无关
- 链表有两个部分：链表基节点 UT_LIST_BASE_NODE_T(TYPE) 与 节点链表 UT_LIST_NODE_T(TYPE)
![双链表示意图](A3819AFEC7C84726BAE82DE8FD783A85)
#### 磁盘双链表
- 主要用于表空间管理、事务处理、B树模块，用于建立保存在磁盘的数据之间的关系
- 先读到内存，然后通过块内的数据偏移得到数据
- 与内存双链表对应的结构是 flst_base_node_t 和 flst_node_t
## 其他数据结构和算法
#### 动态数组
- 存储空间能自动增加的一种数据结构
- 由 dyn_array_t 定义 主要用于mtr模块
- 内存堆分配为上一次的两倍，而动态数组分配 dyn_block_t 大小
- dyn_array_create创建动态数组，但并不立刻分配空间，只初始化第一个块
#### 排序
- innodb是合并排序，函数是 UT_SORT_FUNCTION_BODY(SORT_FUN, ARR, AUX_ARR, LOW, HIGH, CMP_FUN)