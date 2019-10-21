## 直接简单插入记录
CREATE TABLE t1 (a INT PRIMARY KEY, b INT NOT NULL) ENGINE=InnoDB;
insert into t1 values (4,2);
```
./storage/innobase/handler/ha_innodb.cc
ha_innobase::write_row
|–>row_insert_for_mysql 
 |–>转换记录格式row_mysql_convert_row_to_innobase
 |–>保存检查点savept = trx_savept_take(trx);
 |–>row_ins_step
  |–>加IX锁lock_table(0, node->table, LOCK_IX, thr)
    |–> row_ins     //轮询索引，向表中插入记录，这里只有聚集索引
     |–>row_ins_index_entry_step
      |–>构建索引记录(node->entry)row_ins_index_entry_set_vals
      |–>row_ins_index_entry(node->index, node->entry, 0, TRUE, thr)
       |–>检查外键约束row_ins_check_foreign_constraints
       |->使用如下两步尝试插入记录
         >>step1.row_ins_index_entry_low(BTR_MODIFY_LEAF, index, entry,n_ext, thr)/* Try first optimistic descent to the B-tree */
         >>step2，若step1失败,row_ins_index_entry_low(BTR_MODIFY_TREE, index, entry,n_ext, thr);/* Try then pessimistic descent to the B-tree */
           |–> row_ins_index_entry_low
            |–>确定search_mode
             1)如果是聚集索引，search_mode为传参BTR_MODIFY_LEAF或BTR_MODIFY_TREE
             2)当前事务的check_unique_secondary为false时(由变量unique_checks控制，默认为true，表示检查唯一索引约束)，search_mode = mode | BTR_INSERT | BTR_IGNORE_SEC_UNIQUE
             3)否则，search_mode = mode | BTR_INSERT
             |–>btr_cur_search_to_nth_level  查询索引树，将cursor移动到记录相应的位置（待分析）
             |–>检测是否有dupkey，主键索引调用row_ins_duplicate_error_in_clust，二级索引调用row_ins_scan_sec_index_for_duplicate
             |–>modify = row_ins_must_modify  (待分析)
              >>当modify!=0时，表明已经有一个足够长的common prefix，直接覆盖插入记录（待分析）
              >>当modify=0时
              1)当mode=BTR_MODIFY_LEAF时，调用btr_cur_optimistic_insert，尝试向一个索引page中插入记录，如果页面空闲空间太小，则返回失败
               |–>btr_cur_ins_lock_and_undo //检查是否有锁冲突，并插入undolog
                |–>lock_rec_insert_check_and_lock //检查锁冲突
                 |–>没有下一个记录的锁冲突，即lock_rec_get_first返回NULL，更新二级索引trx id，返回
                 |–>调用lock_rec_other_has_conflicting，检查是否有其他事务锁有下一个记录的LOCK_X, LOCK_GAP和LOCK_INSERT_INTENTION锁
                 |–>检测到有冲突lock_rec_enqueue_waiting
                  |–> 创建锁记录lock_rec_create,type_code为LOCK_X | LOCK_GAP|LOCK_INSERT_INTENTION
                  |–>检测死锁
                  |–>如果是聚集索引且不是Insert buffer
                   |–>写入undo信息 trx_undo_report_row_operation（待分析）
                   |–>设置聚集索引记录的trx id和回滚段指针，row_upd_index_entry_sys_field
                 |–>插入记录page_cur_tuple_insert
                 |–>更新该page的哈希索引btr_search_update_hash_on_insert(待分析)
                 |–>继承下一条记录的gap锁?(inherit为TRUE)，调lock_update_insert，这里不调用 (待分析) 
               2)否则
               a)调用buf_LRU_buf_pool_running_out，返回true，表示少于25%的buffer pool可用。根据注释，对于大事务，会将锁存储在buffer pool中（待证实
               b)调用btr_cur_pessimistic_insert（待分析）

```
drop table t1;
#### 简化版步骤
- 转换记录格式
- 保存检查点
- 插入
1. 加IX锁
2. 轮询索引，向表中插入记录，包括步骤：
- [x] 构建索引记录
- [x] 检查外键约束
- [x] row_ins_index_entry_low两步插入：先乐观插入 否则 悲观插入
## 插入记录on duplicate key
```
CREATE TABLE t1 (a INT PRIMARY KEY, b INT NOT NULL) ENGINE=InnoDB;
insert into t1 values (1,2);
insert into t1 values (1,5) on duplicate key update b = b+1;
```

```
ha_innobase::write_row->
row_insert_for_mysql
 –>row_ins_step
  –>row_ins->row_ins_index_entry
  |–>row_ins_index_entry_low
   |–>调用row_ins_duplicate_error_in_clust检测是否有dup key错误
    |–>对记录加锁
     >>对于REPLACE, LOAD DATAFILE REPLACE以及INSERT ON DUPLICATE KEY UPDATE操作，对记录加一个排他锁(LOCK_X)row_ins_set_exclusive_rec_lock
      a)聚集索引lock_clust_rec_read_check_and_lock
      b)二级索引lock_sec_rec_read_check_and_lock
     >>否则，加一个共享锁(LOCK_S)row_ins_set_shared_rec_lock
    |–>调用row_ins_dupl_error_with_rec检查物理记录和将要插入的记录是否相同

```
从Innodb层返回到Server层后，write_record函数根据返回的错误值，继续下面的逻辑


```
|–>判断是否是dup key错误
|–>if (table->file->extra(HA_EXTRA_FLUSH_CACHE)) // bug#52020相关点，稍后再议
|–>一堆乱七八糟的检查，构建记录等….
|–>row_update_for_mysql传递record到innodb更新数据
 |–>row_upd_step->row_upd
  |–>row_upd_clust_step
   |–>如果没有x锁，则尝试加锁lock_clust_rec_modify_check_and_lock
   |–>row_upd_clust_rec(待分析)
    |–>btr_cur_optimistic_update
```
