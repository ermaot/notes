## 历史
- Undo log是MVCC多版本控制的核心模块，一直以来undo log都存储在ibdata系统表空间中
- 从5.6开始，用户可以把undo log存储到独立的tablespace中，并拆分成多个Undo log文件
- 5.6及5.6之前的版本都无法缩小文件的大小，而长时间未提交事务导致大量undo 空间的浪费的例子
- ==5.7的undo log的truncate操作是基于独立undo 表空间来实现的==
## 相关参数
- 在能够使用该特性之前，需要先打开独立undo表空间，注意现在只能在install db的时候才能开启，因为他在初始化阶段是写死占用了最小的几个space id的

## undo相关的几个参数
```
MySQL 8
> show variables like "%undo%";
+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| innodb_max_undo_log_size | 1073741824 |
| innodb_undo_directory    | ./         |
| innodb_undo_log_encrypt  | OFF        |
| innodb_undo_log_truncate | ON         |
| innodb_undo_tablespaces  | 2          |
+--------------------------+------------+
5 rows in set (0.00 sec)

> show global  variables like "%innodb_purge_rseg_truncate_frequency%";
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| innodb_purge_rseg_truncate_frequency | 128   |
+--------------------------------------+-------+
1 row in set (0.00 sec)


```


参数名|说明
---|---
innodb_undo_directory| Undo文件的存储目录
innodb_undo_tablespaces| undo tablespace的个数，至少大于等于2，因为在truncate一个undo log文件时，要保证另外一个是可用的，这样就无需停止业务了.
innodb_undo_logs|undo回滚段的数量， 至少大于等于35，默认128
innodb_max_undo_log_size|控制最大undo tablespace文件的大小，超过这个阈值时才会去尝试truncate. truncate后的大小默认为10M（1G），该值必须设置为G、M的整数倍
innodb_undo_log_truncate|打开/关闭该特性
innodb_purge_rseg_truncate_frequency|用于控制purge回滚段的频度

## 源码过程
新的类undo::Truncate被引入，来管理table space truncate的过程，挂在purge_sys->undo_trunc中

#### 标记需要truncate的undo tablespace
- 这个动作由purge的协调线程发起的，默认情况下每做128次purge后，会调用函数trx_purge_truncate进行清理操作，truncate对应的调用栈为：

```
trx_purge_truncate
|—>trx_purge_truncate_history
|—>trx_purge_mark_undo_for_truncate
|—>trx_purge_initiate_truncate
```
- trx_purge_mark_undo_for_truncate 是标记truncate undo表空间的入口函数，主要包括如下步骤
1. step1: 检查是否开启truncate参数，或者已经有table space已经被标记为truncate
2. step2:检查是否可以进行安全的truncate，也就是上面说的innodb_undo_tablespaces>=2,  innodb_undo_logs>=35
3. step3:从上次扫描的tablespace开始遍历(round-robin)，哪些tablespace可以标记为可truncate，如果发现一个需要truncate的 tablespace，则标记其为需要truncate，并从遍历中break出来:

```
undo_trunc->mark(space_id);
undo::Truncate::add_space_to_trunc_list(space_id);
```

4. step4: 遍历被选中的tablespace中的回滚段，将其设置为不可分配，判断条件为：


```
917                 if (rseg != NULL && !trx_sys_is_noredo_rseg_slot(rseg->id)) {
918                         if (rseg->space
919                                 == undo_trunc->get_marked_space_id()) {
920
921                                 /* Once set this rseg will not be allocated
922                                 to new booting transaction but we will wait
923                                 for existing active transaction to finish. */
924                                 rseg->skip_allocation = true;
925                                 undo_trunc->add_rseg_to_trunc(rseg);
926                         }
927                 }
```


#### truncate operation
- 在标记需要truncate的tablespace后，需要先检查需要删除的回滚段是否是可释放的。也就是没有任何活跃的事务会应用到其中的Undo log
入口函数：trx_purge_initiate_truncate
1. step 1: 检查回滚段是否可释放，如果不可以(有活跃事务可能使用undo做MVCC)，直接返回
2. step 2:做一次redo checkpoint，因为如果随后发生crash，可能针对该undo tablespace的redo 就会无效了，因为文件被truncate了。
log_make_checkpoint_at(LSN_MAX, TRUE);  这会刷新所有脏页和redo log.
3. step 3.开始之前…

```
1100                 undo_trunc->start_logging(
1101                         undo_trunc->get_marked_space_id());
```
创建一个命名为undo_<space_id>_trunc.log的文件，如果crash重启发现该文件，则表明truncate tablespace可能没有完成，需要重做.
4. step 4: 清理对应的purge queue，无需继续做Purge 操作

```
trx_purge_cleanse_purge_queue(undo_trunc);
```
5. step5 : 执行真正的truncate
```
bool    success = trx_undo_truncate_tablespace(undo_trunc);
```
入口函数：trx_undo_truncate_tablespace
..truncate文件
success = fil_truncate_tablespace(
space_id, SRV_UNDO_TABLESPACE_SIZE_IN_PAGES);

文件先被truncate到0，再重新设置到10M，相当于一个新建的undo tablespace.

…重新初始化undo log tablespace的头，这个过程不记录redo log.
fsp_header_init(space_id, SRV_UNDO_TABLESPACE_SIZE_IN_PAGES, &mtr);

…重新初始化该tablespace内的回滚段头
6. step 6：在完成truncate后，再做一次checkpoint
7. step 7: 完成后

```
undo_trunc->done_logging(undo_trunc->get_marked_space_id());
```
删除undo_<space_id>_trunc.log

8. step 8: 清理操作
```
undo_trunc->reset();
undo::Truncate::clear_trunc_list();
```
## 过程概览
- undo 超出innodb_max_undo_log_size设置的表空间标记为truncate。选择用于truncate的undo 表空间是以循环方式执行的，以避免每次都truncate相同的undo 表空间。
- 驻留在所选undo tablespace中的回滚段将处于非活动状态，以便不会给他们分配新事务。允许完成当前使用回滚段的现有事务。
- 该清除系统释放那些不需要的回滚段。
- 释放undo表空间中的所有回滚段后，将允许truncate操作，并将undo表空间truncate为其初始大小。undo表空间文件的初始大小取决于 innodb_page_size值。对于默认的16k InnoDB页面大小，初始undo表空间文件大小为10MiB。对于4k，8k，32k和64k页面大小，初始undo表空间文件大小分别为7MiB，8MiB，20MiB和40MiB。
- 重新激活回滚段，以便将它们分配给新事务。

原文链接：https://blog.csdn.net/wanbin6470398/article/details/81633418

## 性能影响
当truncate undo表空间时，该表空间中的回滚段暂时停用。其他undo 表空间剩余的活动回滚段承担整个系统负载的责任，这可能会导致性能略有下降。性能下降程度取决于许多因素，包括：
- undo 表空间数量
- undo log数量
- undo表空间大小
- I/O速度
- 现有的长事务
- 系统负载
