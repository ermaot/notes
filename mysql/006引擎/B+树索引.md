## B+树
#### 概述
- B+树是为磁盘或其他直接存取辅助设备而设计的一种平衡查找树
- 在B+树中,所有的记录节点都按键值的大小顺序存放在同一层的叶子节点,各叶子节点指针进行连接
#### 插入
- 不管怎么变化,B+树总是会保持平衡。但是为了保持平衡对于新插入的键值可能需要做大量的拆分页(split) 操作
- B+树同样提供了类似于平衡二叉树的旋转(Rotation) 功能，以减少页拆分操作
- 通常情况下,左兄弟被首先检查用来做旋转操作
#### 删除
B+树使用填充因子(fill factor) 来控制树的删除变化,50% 是填充因子可设的最小值
## B+树索引
#### 索引的特点
B+树与索引之间的区别
![image](pic/B%2B%E6%A0%91%E7%B4%A2%E5%BC%951.png)
特点 | B+树|B+树索引
---|---|---
存储位置| 内存|磁盘
扇出率 |低| 高
并发控制 |可以不考虑 |需考虑
分裂方向 |不需要考虑 |向左、向右
- B+树高度较小，一般为3~4层，树的高度决定磁盘搜索次数
- B+树索引只能找到记录所在的页,但是并不能定位到记录在页中的具体位置(偏移量),这需要通过page directory 的二分查找得到具体的记录（内存中完成，效率较高）
#### 聚集索引
- 索引分聚集索引（clustered index）和辅助索引（secondary index）。聚集索引使用表的主键键值狗仔B+树，若无主键，innodb会自动构建一个6字节的隐式主键
- innodb是索引组织表（IOT），索引中的记录根据键值顺序存放（顺序指逻辑顺序而非物理顺序，物理顺序开销太大）
- 聚集索引的非叶子节点存放的是<键值，地址>对，地址指向下一层的指针，通过页在表空间的偏移量表示
#### 辅助索引
- 辅助索引（secondary index）又称二级索引或者非聚集索引（non-clustered index），也是B+树结构
- 辅助索引的叶子节点不保存记录中的所有列，而是<键值，（记录）地址>。记录的地址一般可以有以下两种形式：
1. 记录的物理地址，页号：槽号：偏移量
2. 记录的主键值
- MyISAM 无聚集索引。索引文件和数据文件分开存放。主键与其他索引的区别仅仅是主键是唯一的（unique）且非空（not null）
![MyISAM索引与数据记录之间的关系](pic/B%2B%E6%A0%91%E7%B4%A2%E5%BC%952.png)
- innodb存储引擎是索引组织表，辅助索引中记录地址存放的是主键的键值。若通过辅助索引查询完整记录，还要通过一次聚集索引来书签查找（bookmark lookup）
![innodb辅助索引和聚集索引的关系](pic/B%2B%E6%A0%91%E7%B4%A2%E5%BC%953.png)
- 不同辅助索引实现的对比

操作 | MyIsam  | innodb
---|---|---
读操作 |所有索引都是辅助索引,因此所有索引访问记录的开销基本都是相同的 | 通过辅助索引查询记录仅能得到主键值,<br>要查询完整记录还需要通过一次聚集索引查询 
写操作 |若记录发生改变,需要更新索引|仅当主键值发生改变时,需要更新辅助索引
B+树高度 |低 |聚集索引通常比辅助索引树的高度要高
范围查询| 低效 |聚集查询高效,辅助索引相对低效
#### 填充因子
- 存放记录数量与填充因子有关
- 填充因子受到插入的影响、如果顺序插入，页的填充因子较高（90%或更高）；若是乱序插入，填充因子一般为69%
- 自增主键为顺序插入，UUID主键则乱序插入；辅助索引大部分是无序
## innodb的B+树索引的实现
- 不论B+树索引是叶子节点还是非叶子节点，页、行记录格式与之前介绍相同；叶子节点包含隐藏系统列（事务ID列，回滚指针列，（6字节隐式主键列））
- 对于每一个B+树索引，innodb存储引擎内部会产生一个dict_tree_struct内存对象

***数据结构 dict_tree_struct 的说明***

变量名|类型 |说明
---|---|---
type| ulint |索引的类型,有效值为:<br>DICT_CLUSTED 聚集索引<br>DICT_UNIQUE辅助索引,但含有唯-一约束<br>DICT_UNIVERSAL普通辅助索引<br>DICT_IBUF insert buffer
索id |dulint |索引id
space |ulint| 索引所在表空间
page| ulint| root页的编号( 偏移量)
pad[64]| byte| 将下一个变量lock放在独立的cache  line中,<br>减少多核CPU下并发的资源竞争 
lock| rw_lock_t |读/写锁,控制索引并发
tree_indexes| UT_LIST_BASE_NODE<br>_T(dict_index_t) |链表, 类型为dict_index_t
magic_n | ulint| 调试模式使用
#### 相关latch
- 对于B+树索引的并发控制通过两部分的latch进行控制
1. 首先每个页上都有一个读写锁,这保存在数据结构buf_block_struct中, 可以对每个叶子节点加s-latch或者x-latch用于并发控制
2. 每个索引树的内存对象dict_tree_struct中还有一个读/写锁，可理解为非叶子节点的读/写锁,或者理解为整棵B+树索引的读/写锁。不能访问非叶子节点，即不可访问该B+树
#### 整理
- innodb 的页是一个无序堆，页中的记录无序存放，记录通过record header 的next record指向进行串联
- 删除记录时，记录所占用的空间会释放到链表PAGE_FREE中；分配空间时，首先查找PAGE_FREE链表的第一个空间，满足则分配，若不满足则从PAGE_HEAP_TOP分配
- Innodb有页的整理功能（reorganize）功能。对页进行DML操作，若空间不足，首先进行整理操作，若还是不足，才进行分裂
- 函数btr_page_reorganize_low用来完成页的整理操作
1. 操作开始前对需整理的页加上x-latch
2. 该操作仅需要记录重做日志的类型,对其他操作,如记录和锁信息的移动产生的重做日志不需要进行记录
3. 整理会改变记录的heap_no，所以还需要调用函数 lock_move_reorganize_ page 更新记录上锁的信息
#### 分裂
- 插入操作会导致B+树分裂，innodb 对 B+ 树索引分裂策略进行优化，符合磁盘IOPS较低的特性
- innodb 不再总是将中间记录作为分裂点，而根据插入情况进行判断
#### 合并
## 查找
- 函数 btr_cur_search_to_nth_level 用来查找指定的记录<br>
***btr_cur_search_to_nth_level参数说明***

参数 | 说 明
---|---
index |根据哪个索引进行查询
level |查询到树高度为level时就结束,查询叶子节点中的记录时,level为0
mode |和第8章中介绍页的查询模式相同,有效值为<br>PAGE_CUR_LE<br>PAGE_CUR_L<br>PAGE_CUR_GE<br>PAGE_CUR_G
latch_mode| 对页和内存索引对象加何种latch进行并发控制,有效值为:<br>BTR_SEARCH_LEAF<br>BTR_SEARCH_TREE<br>BTR_NO_LATCHES<br>BTR_MODIFY_TREE<br>BTR_CONT_MODIFY_TREE<br>BTR_SEARCH_PREV<br>BTR_MODIFY_PREV 
cursor |查询得到的记录用btr_cur_t结果保存
has_ search_ latch |是否已经对btr_ search_ latch加上了s-latch,有效值为:<br>0:没有对btr_search_latch加上任何latch保护<br>RW_ S_ LATCH: 已经对btr_ search_ latch 加上latch保护
#### mode
- 查找模式有4种PAGE_CUR_LE、PAGE_CUR_L、PAGE_CUR_GE、PAGE_CUR_G，但实际使用的时候只有PAGE_CUR_LE、PAGE_CUR_GE两种。因为实际在页中，存在多版本记录，也就是存在delete flag不同的相同主键记录
- 非叶子节点搜索，使用的是PAGE_CUR_L。如果查询得到的结果是Supremum，则会自动搜索下一页以便得到正确的数据。使用唯一值查找记录的时候，使用PAGE_CUR_GE，而非PAGE_CUR_LE（此时会锁定Supremum，而如果插入新的Supremum的值，则新的插入语句会被阻塞）
#### latch_mode
- 参数latch_mode表示需要对页和索引内存对象加何种latch进行保护,该参数的有效值说明如下表所示：

有效值 |说明
---|---
BTR_SEARCH_LEAF |对索引内存对象加s-latch,对查询的页加s-latch<br>当查询到叶子节点时释放索引内存对象的S- latch
BTR_MODIFY_LEAF |对索引内存对象加s-latch,对查询的页加x-latch<br>当查询到叶子节点时释放索引内存对象的s-latch
BTR_NO_LATCHES| 对索引内存对象加s-latch,对查询的页不加任何latch保护
BTR_MODIFY_TREE |对索引内存对象加x-latch,对查询的页加x-latch
BTR_CONT_MODIFY_TREE |函数开始前已经对索引内存对象加x-latch,对查询的页加x-latch
BTR_SEARCH_PREV| 对查询的页加s-latch
BTR_MODIFY_PREV| 对查询的页加x-latch

- 若函数btr_cur_search_to_nth_level最终定位到的页不是叶子节点,即level大于0,那么对页加x-latch,同时不释放索引内存对象上的latch保护，否则根据latch_mode对叶子节点加上latch保护,并根据latch_mode选择是否释放索引对象上的latch保护
- InnoDB存储引擎总是首先对索引内存对象加上s-latch保护,然后进行页的操作,若操作INSERT、UPDATE、DELETE不会导致非叶子节点发生变化,即不会发生分裂、合并、树的高度变化,则其立即释放索引内存对象的s-latch保护,称这种方式为==乐观方式==。否则,立即释放索引内存对象以及页上的latch,并对内存索引对象和页加x-latch保护,这种方式称为==悲观方式==,并发在此处受到了限制。
#### cursor
- cursor是一个 btr_cur_struct 数据结构对象

```
struct btr_cur_struct {
	dict_index_t*	index;		
	page_cur_t	page_cur;	
	page_t*		left_page;		
	que_thr_t*	thr;			
	/* The following fields are used in btr_cur_search... to pass	information: */
	ulint		flag;		
	ulint		tree_height;
	ulint		up_match;	
	ulint		up_bytes;	
	ulint		low_match;
	ulint		low_bytes;	
	ulint		n_fields;
	ulint		n_bytes;	
	ulint		fold;		
	btr_path_t*	path_arr;	
};
```
- btr_cur_struct 变量说明

变量 |说明
---|---
index |此次查询使用的索引
page_cur| 查询得到的结果
left_page| 查询得到记录所在页的左兄弟页
thr |线程队列
flag |使用何种查询得到记录结果,有效值为:<br>BTR_CUR_HASH使用自适应哈希查询得到记录<br>BTR_CUR_HASH_FAIL使用自适应哈希查询记录失败<br>之后使用B+树索引查询成功<br>BTR_CUR_BINARY直接使用B+树索引查询得到结果<br>BTR_CUR_INSERT_TO_IBUF 使用插入缓存进行记录的插入
tree_height |树的高度
up_match|
up_bytes|
low_match|函数page_cur_search_With_match 返回的结果
low_bytes|
n_fields|==在自适应哈希小节中进行介绍==
n_ bytes|
fold |哈希fold
path |查询得到记录的路径,并保存每个路径上的查询信息
==此处还有内容，稍后完善==
## dml操作
#### 插入
- innodb 有悲观和乐观两个阶段。
- 首先调用乐观插入（btr_cur_optimistic_insert），如果会导致页分裂，则调用悲观插入（btr_cur_pessimistic_insert）
- 调用乐观插入，如果记录不能插入到页中，就对页整理；若还是不行，再悲观插入
- btr_cur_optimistic_insert 参数说明

参数 | 说明
---|---
flags |有效值为:<br>0<br>**BTR_NO_LOCKING_FLAG**:若含有BTR_NO_LOCKING_FLAG标志位,<br>则表示插入后不需要对查询的记录加上一个锁。例如对于插入缓存<br>不会对其进行并发的读取操作,因此不需要对插入的记录进行锁保护<br>**BTR_KEEP_SYS_FLAG**:若不含有该标志位时,更新记录的隐藏列roll ptr<br>例如当插入的是辅助索引或者是非叶子节点时,记录不含有隐藏列,因此需要设置该标志位<br>**BTR_NO_UNDO_LOG_FLAG**:若flag含有此标志位,表示不需要记录undo log<br>例如回滚时不需要再次产生undo log
cursor |curosr指向插入前通过函数btr_cur_search_to_nth_Level定位到待插入记录的前一条记录<br>其查询mode为PAGE_CUR_LE, latch_mode 为BTR_MODIFY_LEAF entry 
rec |当插入成功,返回插入后的记录
big_rec |当插入成功,并且插入的记录转化为大记录,返回大记录格式
thr |查询线程
mt r |页的日志

1. 插入记录为逻辑记录，先转化为物理记录，然后计算页是否有足够的空间容纳，若无则返回DB_FAIL（innodb要求插入操作保留1/16页大小预留空间，防止update导致页分裂）
2. 若插入的元组转化为大记录格式，则需要将行溢出页部分保存在变量big_rec中，调用btr_cur_optimistic_insert完成后，再插入行溢出页列的部分
3. 逻辑记录转化为物理记录后，还需要通过函数btr_cur_ins_lock_and_undo检查锁的信息，并成为undo日志。若锁检测到下一个记录已经被其他事务持有，则函数返回DB_WAIT_LOCK
4. 通过之前的检查后，确保记录可以插入页中。先调用page模块的page_cur_insert_rec_low进行记录的插入，若失败，则调用btr_page_reorganize整理页，然后再btr_cur_optimistic_insert
5. 更新页的自适应哈希索引
6. 若插入操作需要更新内存中页锁信息或flag不包含BTR_NO_LOCKING_FLAG标志位，则调用函数lock_update_insert
7. 若插入的对象是插入缓冲（插入缓冲本身也是B+ 树），更新对应缓冲位图页的信息
8. btr_cur_optimistic_insert完成后，会释放页的x-latch。若big_rec不为NULL，表示有行溢出数据，此时需要对索引内存对象加x-latch，再对页加x-latch，之后调用btr_store_big_rec_extern_fileds将溢出的列数据存放到行溢出页中
- btr_cur_pessimistic_insert 函数参数与乐观插入btr_cur_optimistic_insert完全相同，不同如下：
1. 索引内存对象持有x-latch，页持有x-latch，开销较大，并发受限
2. 悲观插入前，依然会尝试乐观插入（此时依然持有内存索引对象的x-latch）
3. 确定需要悲观操作，预留一些区的空间，以保证即使索引树结构发生变化但依然有足够的磁盘空间保证操作完成。插入完成后释放该空间（源码显示至少预留3个区大小的空间）
4. 调用btr_root_raise_and_inset 和btr_page_split_and_insert完成分裂和插入操作
- InnoDB存储引擎中对于非叶子节点的插入都是悲观插入.非叶子节点的插入由函数btr_insert_on_non_leaf_level 完成
#### 非主键更新
- 非主键更新也分为乐观更新和悲观更新
- 乐观更新有原地更新（update in place）和普通的乐观更新
- 原地更新是指更新记录中各个列在更新过程中都没有发生改变（而非列大小之和没有改变）；包含extern属性的列不能原地更新
- 一般辅助索引都不能是原地更新，而是delete mark + insert操作；若辅助索引列的字符集发生改变但排序规则没变，可以就地更新
- 函数btr_cur_optimistic_update进行乐观更新,前提是主键列没有发生改变。此外,乐观更新过程中仅持有记录所在页的x-latch。该函数的参数如表所示

btr_cur_optimistic_update参数说明
参数| 说 明
---|---
flags |同乐观插入相同
cursor|指向需要更新的记录
update |保存更新操作发生变化列的新值
cmpl_ info |辅助索引的更新信息,将在undo章节进行详细介绍thr 查询线程
mtr |页的日志
#### 主键更新
- innodb主键更新步骤：
1. 设置原主键记录的delete flag为1
2. 插入新主键记录的值
3. purge线程判断是否有其他事务引用原主键记录；若无，对记录彻底删除
- 该方式将更新操作变成了插入操作
- 列属性为extern时，大部分数据在溢出页中。若没有更新extern，新记录需要继承溢出页数据。purge进程清理时，原主键不拥有该溢出页或者undo时该页被记录拥有，则溢出页都不能被删除
#### 删除
- innodb删除操作步骤
1. 更新记录的delete flag（删除操作的事务中完成），即delete mark操作
2. purge线程删除记录（后台的purge线程中完成）。彻底删除，记录的空间会放入到页PAGE_FREE链表中
- 辅助索引上没有隐式锁，因此不需要隐式和显式的锁转化
- 聚集索引伪删除操作产生undo日志
- delete mark操作会产生重做日志（聚集索引和辅助索引），格式如下：
![delete mark重做日志格式](pic/B%2B%E6%A0%91%E7%B4%A2%E5%BC%954.png)
****表示压缩方式存储***
## 持久游标
- btr0cur模块用于对B+树索引进行增删改查操作，产生对应的redo和undo，并处理可能发生的B+树分裂和合并的情况
- 上层不直接调用btr0cur中的查询函数，而通过持久游标（persistent cursor）对象处理查询并调用btr0cur中的查询函数
- 将查询得到的记录信息保存在持久游标中

```
struct btr_pcur_struct{
btr_cur_t btr_cur; /* a B-tree cursor */
ulint latch_mode; /*同函数btr_cur_search_to_nth_level*/
ulint old_stored;/*表示是否有记录存放于持久游标中，有效值为：
                    BTR_PCUR_OLD_STORED
                    BTR_PCUR_OLD_NOT_STORED*/
rec_t* old_rec; /*持久游标保存查询得到的记录*/
ulint rel_pos;/*持久游标定位到的位置，有效值为：
                BTR_PCUR_AFTER_LAST_IN_TREE定位到的页没有记录,伪定位到伪记录inf imum
                BTR_PCUR_BEFORE_FIRST_IN_TREE定位到的页没有记录,且定位到伪记录sup remum
                BTR_PCUR_ON定位到用户记录,old_rec保存指向的用户记录
                BTR_PCUR_BEFORE定位到infimum记录, old_rec保存第一个用户记录
                BTR_PCUR_AFTER定位到记录supremum, old_rec保存最后一个用户记录*/
dulint modify_clock;/**/
ulint pos_state;/*标记位置信息,有效值为:
                BTR_PCUR_IS_POSITIONED
                BTR_PCUR_WAS_POSITIONED
                BTR_PCUR_NOT__POSITIONED*/
mtr_t* mtr;/**/
byte* old_rec_buf;/**/
ulint buf_size;/**/
};
```
- 进行select、update、delete操作，首先定位到第一条记录，然后扫描下一条记录（fetch next record）。持久游标用户保存每次查询到的记录，待扫描下一条记录时需要对上一条记录进行恢复，若页没有发生改变则可直接使用old_rec定位，否则需要根据old_rec索引键值重新定位记录再进行查询
- 对于唯一键值的等值查询且锁模式部位lock_x时，innodb不需要持久游标保存查询得到的记录
## 自适应哈希索引
#### 实现原理
- 自适应哈希索引是指，哈希索引的创建和维护都是由存储引擎完成，用户不能自己手动干预
- 并非整张表的数据进行索引，而是热点也的数据
- 哈希索引根据记录进行索引，而B+树根据页进行索引
- 自适应哈希索引在内存中创建，数据库重启之后需要重新创建
#### 创建哈希索引
#### 哈希索引的维护
#### 自适应哈希索引的优缺点
