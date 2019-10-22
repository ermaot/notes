## 简述
[参考链接](http://mysql.taobao.org/monthly/2018/03/02/?spm=a2c4e.10696291.0.0.168619a4MyG0Qk)
#### MySQL8.0之前的版本DDL是非原子的。
- 对于复合的DDL，比如DROP TABLE t1, t2;执行过程中如果遇到server crash，有可能出现表t1被DROP掉了，但是t2没有被DROP掉的情况。
- 即便是一条DDL，比如CREATE TABLE t1(a int);也可能在server crash的情况下导致建表不完整，有可能在建表失败的情况下遗留.frm或者.ibd文件

#### MySQL8.0以前的数据字典
metadata在Server layer是存储在MyISAM引擎的系统表里，对于事务型引擎Innodb则自己存储一份metadata
- metadata由于存储在Server layer以及存储引擎（这里特指Innodb)，两份系统表很容易造成数据的不一致。
- 两份系统表存储的信息有所不同，访问Server layer以及存储引擎需要使用不同API，这种设计导致了不能很好的统一对系统metadata的访问。另外两份API，也同时增加了代码的维护量。
- 由于Server layer的metadata存储在非事务引擎（MyISAM)里，所以在进行crash recovery的时候就不能维持原子性。
- DDL的非原子性使得Replication处理异常情况变得更加复杂。比如DROP TABLE t1, t2; 如果DROP t1成功，但是DROP t2失败，Replication就无法保证主备一致性了。
![ MySQL8.0前的数据字典](pic/MySQL8.0%20%E5%8E%9F%E5%AD%90DDL1.png)

#### MySQL8.0 的create table
![image](pic/MySQL8.0%20%E5%8E%9F%E5%AD%90DDL2.png)


#### MySQL8.0 的drop table
- 为了实现原子DDL的提交和回滚，InnoDB存储引擎引入了一个表DDL_LOG。
- 该表用来存储DDL执行期间InnoDB存储引擎需要对物理文件以及相关系统表操作的记录。
- 当DDL事务进行提交或者回滚之前，InnoDB存储引擎实际上不对物理文件或者相关系统表进行修改，只是记录相关的操作日志。
- 当DDL进行提交或者回滚操作的时候，InnoDB会对DDL_LOG表里的日志进行重放或者删除。
- DDL_LOG表作为一张日志记录表，它具有以下特点:
1. 不允许外部用户查询和修改,包括对该表进行DDL以及DML；
2. 对于DDL_LOG中的每一条记录都包含有trx_id（事务id），当DDL提交或者回滚完成的时候，post_ddl hook将会自动清除该表中的记录
3. 为了SERVER crash的时候DDL还能支持原子性，这个表的存储比较特殊，需要进行同步刷新。也就是只要写入数据就会进行持久化，不受innodb_flush_log_at_trx_commit的控制

#### DDL_LOG操作的实现
通过Log_DDL类实现
```
class Log_DDL {
 public:
  /** Constructor */
  Log_DDL();

  /** Deconstructor */
  ~Log_DDL() {}

  /* 记录对于Btree的操作 */
  dberr_t write_free_tree_log(trx_t *trx, const dict_index_t *index,
                              bool is_drop_table);

  /* 记录删除ibd文件的操作 */
  dberr_t write_delete_space_log(trx_t *trx, const dict_table_t *table,
                                   space_id_t space_id, const char *file_path,
                                 bool is_drop, bool dict_locked);

  /* 记录重命名ibd文件的操作 */
  dberr_t write_rename_space_log(space_id_t space_id, const char *old_file_path,
                                 const char *new_file_path);

  /* 记录DROP TABLE操作 */
  dberr_t write_drop_log(trx_t *trx, const table_id_t table_id);

  /* 记录Rename操作 */
    dberr_t write_rename_table_log(dict_table_t *table, const char *old_name,const char *new_name);

  /* 记录删除表缓冲记录的操作 */
  dberr_t write_remove_cache_log(trx_t *trx, dict_table_t *table);

  /** 对DDL_LOG中的记录进行重放的操作。当SERVER层对原子DDL需要进行提交的时候， InnoDB会对DDL_LOG表中的记录进行重放来完成DDL对物理文件操作。*/
  dberr_t replay(DDL_Record &record);

  /** DDL提交或者回滚的时候，InnoDB存储引擎会调用该函数完成DDL的实际操作。如果 DDL事务成功提交，重放所有日志文件完成物理文件的实际操作并清除日志记录。
     如果回滚，则只需要清除掉DDL_LOG表中对应的日志记录即可。*/
  dberr_t post_ddl(THD *thd);

  /* SERVER启动的时候，会扫描DDL_LOG表，并重放所有的日志记录。*/
  dberr_t recover();
    /** Is it in ddl recovery in server startup.
  @return true if it's in ddl recover */
  static bool is_in_recovery() { return (s_in_recovery); }

 private:
  /* 下面相关的函数是真正操作DDL_LOG表的接口函数，是用来辅助实现上面的write**函数以及replay函数的。*/
  dberr_t insert_free_tree_log(trx_t *trx, const dict_index_t *index,
                               uint64_t id, ulint thread_id);

  void replay_free_tree_log(space_id_t space_id, page_no_t page_no,
                            ulint index_id);

  dberr_t insert_delete_space_log(trx_t *trx, uint64_t id, ulint thread_id,
                                  space_id_t space_id, const char *file_path,
                                  bool dict_locked);

  void replay_delete_space_log(space_id_t space_id, const char *file_path);

  dberr_t insert_rename_space_log(uint64_t id, ulint thread_id,
                                  space_id_t space_id,
                                  const char *old_file_path,
                                  const char *new_file_path);
  void replay_rename_space_log(space_id_t space_id, const char *old_file_path,
                               const char *new_file_path);

  dberr_t insert_drop_log(trx_t *trx, uint64_t id, ulint thread_id,
                          const table_id_t table_id);

  void replay_drop_log(const table_id_t table_id);

  dberr_t insert_rename_table_log(uint64_t id, ulint thread_id,
                                  table_id_t table_id, const char *old_name,
                                  const char *new_name);

  void replay_rename_table_log(table_id_t table_id, const char *old_name,
                               const char *new_name);

  dberr_t insert_remove_cache_log(uint64_t id, ulint thread_id,
                                  table_id_t table_id, const char *table_name);

  void replay_remove_cache_log(table_id_t table_id, const char *table_name);

  /** Delete log record by id
  @param[in]  trx   transaction instance
  @param[in]  id    log id
  @param[in]  dict_locked true if dict_sys mutex is held,
                                  otherwise false
  @return DB_SUCCESS or error */
  dberr_t delete_by_id(trx_t *trx, uint64_t id, bool dict_locked);

  /** Scan, replay and delete log records by thread id
  @param[in]  thread_id thread id
  @return DB_SUCCESS or error */
  dberr_t replay_by_thread_id(ulint thread_id);

  /** Delete the log records present in the list.
  @param[in]  records   DDL_Records where the IDs are got
  @return DB_SUCCESS or error. */
  dberr_t delete_by_ids(DDL_Records &records);

  /** Scan, replay and delete all log records
  @return DB_SUCCESS or error */
  dberr_t replay_all();

  /** Get next autoinc counter by increasing 1 for innodb_ddl_log
    @return new next counter */
  inline uint64_t next_id();

  /** Check if we need to skip ddl log for a table.
  @param[in]  table dict table
  @param[in]  thd mysql thread
  @return true if should skip, otherwise false */
  inline bool skip(const dict_table_t *table, THD *thd);

 private:
  /** Whether in recover(replay) ddl log in startup. */
  static bool s_in_recovery;
};
```
记录操作的类型有：
1. 记录对于Btree的操作
2. 记录删除ibd文件的操作
3. 记录重命名ibd文件的操作
4. 记录DROP TABLE操作
5. 记录Rename操作
6. 记录删除表缓冲记录的操作


#### drop table流程
![drop table流程](pic/MySQL8.0%20%E5%8E%9F%E5%AD%90DDL3.png)
- 对于不支持原子DDL的存储引擎，Handler::ha_delete_table MySQL8.0的执行方式和之前版本没有太大的区别，都是直接删除物理文件，然后清理系统表
- 对于InnoDB存储引擎而言，Handler::ha_delete_table并不会进行实际物理文件的修改，而只是记录相关的操作到DDL_LOG table
- 当DDL事务提交或者回滚的时候，会调用post_ddl进行日志回放

#### innobase_basic_ddl::delete_impl函数的源码

```
/**
  该函数用来实现InnoDB存储引擎端，执行DROP TABLE语句时所采取的一些列步骤。让我们
  根据源码来分析一下InnoDB为了支持原子DDL所做的修改。
  innobase_basic_ddl类实现了InnoDB在create table，drop table，rename table的时候
  需要进行的操作。这里我们重点分析drop table的操作。
*/
template <typename Table>
int innobase_basic_ddl::delete_impl(THD *thd, const char *name,
                                    const Table *dd_tab,
                                    enum enum_sql_command sqlcom) {
  dberr_t error = DB_SUCCESS;
  char norm_name[FN_REFLEN];

  DBUG_EXECUTE_IF("test_normalize_table_name_low",
                  test_normalize_table_name_low(););
  DBUG_EXECUTE_IF("test_ut_format_name", test_ut_format_name(););

  /* Strangely, MySQL passes the table name without the '.frm'
  extension, in contrast to ::create */
  normalize_table_name(norm_name, name);

  innodb_session_t *&priv = thd_to_innodb_session(thd);
  /* 根据表名查找对应的InnoDB表结构 */
  dict_table_t *handler = priv->lookup_table_handler(norm_name);

  /* 释放索引上的cache */
  if (handler != NULL) {
    for (dict_index_t *index = UT_LIST_GET_FIRST(handler->indexes);
         index != NULL && index->last_ins_cur;
         index = UT_LIST_GET_NEXT(indexes, index)) {
      /* last_ins_cur and last_sel_cur are allocated
      together,therfore only checking last_ins_cur
      before releasing mtr */
      index->last_ins_cur->release();
      index->last_sel_cur->release();
       } else if (srv_read_only_mode ||
             srv_force_recovery >= SRV_FORCE_NO_UNDO_LOG_SCAN) {
    return (HA_ERR_TABLE_READONLY);
  }

  trx_t *trx = check_trx_exists(thd);

  TrxInInnoDB trx_in_innodb(trx);

  ulint name_len = strlen(name);

  ut_a(name_len < 1000);

  /* Either the transaction is already flagged as a locking transaction
  or it hasn't been started yet. */

  ut_a(!trx_is_started(trx) || trx->will_lock > 0);

  /* We are doing a DDL operation. */
  ++trx->will_lock;

  bool file_per_table = false;
  if (dd_tab != nullptr && dd_tab->is_persistent()) {
    dict_table_t *tab;

    dd::cache::Dictionary_client *client = dd::get_dd_client(thd);
    dd::cache::Dictionary_client::Auto_releaser releaser(client);
    /* 打开系统表来获取表定义内容 */
        int err = dd_table_open_on_dd_obj(
        client, dd_tab->table(),
        (!dd_table_is_partitioned(dd_tab->table())
             ? nullptr
             : reinterpret_cast<const dd::Partition *>(dd_tab)),
        norm_name, tab, thd);

    if (err == 0 && tab != nullptr) {
      /* 这里会检查表是否可以被换出缓冲。为了避免重复打开使用表，这里优化不淘汰正在或者即将被使用的表 */
      if (tab->can_be_evicted && dd_table_is_partitioned(dd_tab->table())) {
        mutex_enter(&dict_sys->mutex);
        dict_table_ddl_acquire(tab);
        mutex_exit(&dict_sys->mutex);
      }

      file_per_table = dict_table_is_file_per_table(tab);
      dd_table_close(tab, thd, nullptr, false);
    }
  }
  /* 该函数负责将执行DROP TABLE的操作写入DDL_LOG table中。 */
  error = row_drop_table_for_mysql(norm_name, trx, sqlcom, true, handler);

  if (handler != nullptr && error == DB_SUCCESS) {
    priv->unregister_table_handler(norm_name);
  }
    if (error == DB_SUCCESS && file_per_table) {
    dd::Object_id dd_space_id = dd_first_index(dd_tab)->tablespace_id();
    dd::cache::Dictionary_client *client = dd::get_dd_client(thd);
    dd::cache::Dictionary_client::Auto_releaser releaser(client);

    if (dd_drop_tablespace(client, thd, dd_space_id) != 0) {
      error = DB_ERROR;
    }
  }

  return (convert_error_code_to_mysql(error, 0, NULL));
}
```
#### post_ddl的源码
```
dberr_t Log_DDL::post_ddl(THD *thd) {
  if (skip(nullptr, thd)) {
    return (DB_SUCCESS);
  }

  if (srv_read_only_mode || srv_force_recovery >= SRV_FORCE_NO_UNDO_LOG_SCAN) {
    return (DB_SUCCESS);
  }

  DEBUG_SYNC(thd, "innodb_ddl_log_before_enter");

  DBUG_EXECUTE_IF("ddl_log_before_post_ddl", DBUG_SUICIDE(););

  /* If srv_force_recovery > 0, DROP TABLE is allowed, and here only
  DELETE and DROP log can be replayed. */

  ulint thread_id = thd_get_thread_id(thd);

  if (srv_print_ddl_logs) {
    ib::info(ER_IB_MSG_660)
        << "DDL log post ddl : begin for thread id : " << thread_id;
  }
  
  thread_local_ddl_log_replay = true;

  /* 这里是回放函数。当DDL回滚的时候，由于所有对DDL_LOG表的操作都是在事务中进行的，
     当事务回滚的时候，所有DDL进行的操作记录都将被回滚掉，也就是说该函数调用基本是进去走一趟就出来了。 */
  replay_by_thread_id(thread_id);

  thread_local_ddl_log_replay = false;

  if (srv_print_ddl_logs) {
    ib::info(ER_IB_MSG_661)
        << "DDL log post ddl : end for thread id : " << thread_id;
  }

  return (DB_SUCCESS);
}
```

原子DDL是MySQL8.0引入的非常重要的一个特性，可以期待以后==事务DDL==的出现