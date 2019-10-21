## 建表

```
create table t1(id int);
insert into t1 values(1)
```

## parse 解析
```
void mysql_parse(THD *thd, char *rawbuf, uint length,
 Parser_state *parser_state) {
 /* ...... */ /* 检查query_cache，如果结果存在于cache中，直接返回 */ if (query_cache_send_result_to_client(thd, rawbuf, length) <= 0) 
 {
 LEX *lex= thd->lex;
 	 
 	 /* 解析语句 */ bool err= parse_sql(thd, parser_state, NULL);
		
	 /* 整理语句格式，记录 general log */ /* ...... */ /* 执行语句 */
 error= mysql_execute_command(thd);
 /* 提交或回滚没结束的事务（事务可能在mysql_execute_command中提交，用trx_end_by_hint标记事务是否已经提交） */ if (!thd->trx_end_by_hint) 
 {
 if (!error && lex->ci_on_success)
 trans_commit(thd);
 
 if (error && lex->rb_on_fail)
 trans_rollback(thd);
 }
```

## 执行
#### mysql_execute_command()
```
 /* */ /* ...... */ case SQLCOM_INSERT:
 { 
 
 /* 检查权限 */ if ((res= insert_precheck(thd, all_tables)))
 break;

 /* 执行insert */
 res= mysql_insert(thd, all_tables, lex->field_list, lex->many_values,
 lex->update_list, lex->value_list,
 lex->duplicates, lex->ignore);

	/* 提交或者回滚事务 */ if (!res)
 {
 trans_commit_stmt(thd);
 trans_commit(thd);
 thd->trx_end_by_hint= TRUE;
 }
 else if (res)
 {
 trans_rollback_stmt(thd);
 trans_rollback(thd);
 thd->trx_end_by_hint= TRUE;
 }
```
## mysql_insert()
```
bool mysql_insert(THD *thd,TABLE_LIST *table_list,
 List<Item> &fields, /* insert 的字段 */ List<List_item> &values_list, /* insert 的值 */ List<Item> &update_fields,
 List<Item> &update_values,
 enum_duplicates duplic,
 bool ignore)
{ 
 /*对每条记录调用 write_record */ while ((values= its++))
 {
	if (lock_type == TL_WRITE_DELAYED)
 {
 LEX_STRING const st_query = { query, thd->query_length() };
 DEBUG_SYNC(thd, "before_write_delayed");
 /* insert delay */
 error= write_delayed(thd, table, st_query, log_on, &info);
 DEBUG_SYNC(thd, "after_write_delayed");
 query=0;
 }
 else /* normal insert */
 error= write_record(thd, table, &info, &update);
 }
 
 /*
 这里还有
 thd->binlog_query()写binlog
 my_ok()返回ok报文，ok报文中包含影响行数
 */ 
```
#### write_record
```
/*
 COPY_INFO *info 用来处理唯一键冲突，记录影响行数
 COPY_INFO *update 处理 INSERT ON DUPLICATE KEY UPDATE 相关信息
*/
int write_record(THD *thd, TABLE *table, COPY_INFO *info, COPY_INFO *update)
{
 if (duplicate_handling == DUP_REPLACE || duplicate_handling == DUP_UPDATE)
 {
 /* 处理 INSERT ON DUPLICATE KEY UPDATE 等复杂情况 */
 }
 /* 调用存储引擎的接口 */ else if ((error=table->file->ha_write_row(table->record[0])))
 {
 DEBUG_SYNC(thd, "write_row_noreplace");
 if (!ignore_errors ||
 table->file->is_fatal_error(error, HA_CHECK_DUP))
 goto err; 
 table->file->restore_auto_increment(prev_insert_id);
 goto ok_or_after_trg_err;
 }
}
```
#### ha_write_row、write_row
```
/* handler 是各个存储引擎的基类，这里我们使用InnoDB引擎*/ int handler::ha_write_row(uchar *buf)
{
 /* 指定log_event类型*/
 Log_func *log_func= Write_rows_log_event::binlog_row_logging_function;
 error= write_row(buf);
}
```
#### 入引擎层
这里是innodb引擎，handler对应ha_innobase 插入的表信息保存在handler中
```
int ha_innobase::write_row(
/*===================*/
 uchar* record) /*!< in: a row in MySQL format */
{
		error = row_insert_for_mysql((byte*) record, prebuilt);
}
```
----
```
UNIV_INTERN
dberr_t
row_insert_for_mysql( 
/*=================*/
 byte* mysql_rec, /*!< in: row in the MySQL format */
 row_prebuilt_t* prebuilt) /*!< in: prebuilt struct in MySQL
 handle */
{
		/*记录格式从MySQL转换成InnoDB*/
		row_mysql_convert_row_to_innobase(node->row, prebuilt, mysql_rec);
	
 thr->run_node = node;
 thr->prev_node = node;
		
		/*插入记录*/
 row_ins_step(thr);
}
```
---
```
UNIV_INTERN
que_thr_t*
row_ins_step(
/*=========*/
 que_thr_t* thr) /*!< in: query thread */
{
		/*给表加IX锁*/
		err = lock_table(0, node->table, LOCK_IX, thr);
		
		/*插入记录*/
		err = row_ins(node, thr);
}
```
#### 非主键的处理
InnoDB表是基于B+树的索引组织表，如果InnoDB表没有主键和唯一键，需要分配隐含的row_id组织聚集索引
```
static __attribute__((nonnull, warn_unused_result))
dberr_t
row_ins(
/*====*/
 ins_node_t* node, /*!< in: row insert node */
 que_thr_t* thr) /*!< in: query thread */
{
		if (node->state == INS_NODE_ALLOC_ROW_ID) {
				/*若innodb表没有主键和唯一键，用row_id组织索引*/
 		row_ins_alloc_row_id_step(node);
				
				/*获取row_id的索引*/
 node->index = dict_table_get_first_index(node->table);
 node->entry = UT_LIST_GET_FIRST(node->entry_list);
		}
		
		/*遍历所有索引，向每个索引中插入记录*/ while (node->index != NULL) {
 if (node->index->type != DICT_FTS) {
 /* 向索引中插入记录 */
 err = row_ins_index_entry_step(node, thr);

 if (err != DB_SUCCESS) {

 return(err);
 }
 } 
				
				/*获取下一个索引*/
 node->index = dict_table_get_next_index(node->index);
 node->entry = UT_LIST_GET_NEXT(tuple_list, node->entry);

 }
 }
}
```
#### 插入单个索引项
```
static __attribute__((nonnull, warn_unused_result))
dberr_t
row_ins_index_entry_step( 
/*=====================*/
 ins_node_t* node, /*!< in: row insert node */
 que_thr_t* thr) /*!< in: query thread */
{
 dberr_t err;

 /*给索引项赋值*/
 row_ins_index_entry_set_vals(node->index, node->entry, node->row);

		/*插入索引项*/
 err = row_ins_index_entry(node->index, node->entry, thr);

 return(err);
}
```
---
```
static
dberr_t
row_ins_index_entry( 
/*================*/
 dict_index_t* index, /*!< in: index */
 dtuple_t* entry, /*!< in/out: index entry to insert */
 que_thr_t* thr) /*!< in: query thread */ {

 if (dict_index_is_clust(index)) {
 		/* 插入聚集索引 */ return(row_ins_clust_index_entry(index, entry, thr, 0));
 } else {
 		/* 插入二级索引 */ return(row_ins_sec_index_entry(index, entry, thr));
 }
}
```
#### row_ins_clust_index_entry 和 row_ins_sec_index_entry 
函数结构类似，只分析插入聚集索引
```
UNIV_INTERN
dberr_t
row_ins_clust_index_entry(
/*======================*/
 dict_index_t* index, /*!< in: clustered index */
 dtuple_t* entry, /*!< in/out: index entry to insert */
 que_thr_t* thr, /*!< in: query thread */
 ulint n_ext) /*!< in: number of externally stored columns */
{
 if (UT_LIST_GET_FIRST(index->table->foreign_list)) {
 err = row_ins_check_foreign_constraints(
 index->table, index, entry, thr);
 if (err != DB_SUCCESS) {
 return(err);
 }
 }
 
 /* flush log，make checkpoint（如果需要） */
 log_free_check();

		/* 先尝试乐观插入，修改叶子节点 BTR_MODIFY_LEAF */
 err = row_ins_clust_index_entry_low(
 0, BTR_MODIFY_LEAF, index, n_uniq, entry, n_ext, thr, 
 &page_no, &modify_clock);
 
 if (err != DB_FAIL) {
 DEBUG_SYNC_C("row_ins_clust_index_entry_leaf_after");
 return(err);
 } 
		
		/* flush log，make checkpoint（如果需要） */
 log_free_check();

		/* 乐观插入失败，尝试悲观插入 BTR_MODIFY_TREE */ return(row_ins_clust_index_entry_low(
 0, BTR_MODIFY_TREE, index, n_uniq, entry, n_ext, thr,
 &page_no, &modify_clock));
```
