## 2PC
- MySQL的事务提交逻辑主要在函数ha_commit_trans中完成。
- 事务的提交涉及到binlog及具体的存储的引擎的事务提交，所以MySQL用2PC来保证的事务的完整性。
- MySQL的2PC过程如下：

![image](https://github.com/ermaot/notes/blob/master/mysql/008%E4%BA%8B%E5%8A%A1/pic/Innodb%E4%B8%A4%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A41.png)

## 事务提交过程
- 先调用binglog_hton和innobase_hton的prepare方法完成第一阶段，binlog_hton的papare方法实际上什么也没做，innodb的prepare将事务状态设为TRX_PREPARED，并将redo log刷磁盘 (innobase_xa_prepare à trx_prepare_for_mysql à trx_prepare_off_kernel)。
- 如果事务涉及的所有存储引擎的prepare都执行成功，则调用TC_LOG_BINLOG::log_xid将SQL语句写到binlog，此时，事务已经铁定要提交了。否则，调用ha_rollback_trans回滚事务，而SQL语句实际上也不会写到binlog。
- 最后，调用引擎的commit完成事务的提交。实际上binlog_hton->commit什么也不会做(因为(2)已经将binlog写入磁盘)，innobase_hton->commit则清除undo信息，刷redo日志，将事务设为TRX_NOT_STARTED状态(innobase_commit à innobase_commit_low à trx_commit_for_mysql à trx_commit_off_kernel)。
```
//ha_innodb.cc
static int innobase_commit(
/*============*/
/* out: 0 */
THD* thd, /* in: MySQL thread handle of the user for whom
the transaction should be committed */
bool all) /* in: TRUE - commit transaction
FALSE - the current SQL statement ended */
{
...
trx->mysql_log_file_name = mysql_bin_log.get_log_fname();
trx->mysql_log_offset = (ib_longlong)mysql_bin_log.get_log_file()->pos_in_file;
...
}
```
- 函数innobase_commit提交事务，先得到当前的binlog的位置，然后再写入事务系统PAGE(trx_commit_off_kernel à trx_sys_update_mysql_binlog_offset)。InnoDB将MySQL binlog的位置记录到trx system header中
```
//trx0sys.h
/* The offset of the MySQL binlog offset info in the trx system header */
#define TRX_SYS_MYSQL_LOG_INFO (UNIV_PAGE_SIZE - 1000)
#define TRX_SYS_MYSQL_LOG_MAGIC_N_FLD 0 /* magic number which shows
if we have valid data in the
MySQL binlog info; the value
is ..._MAGIC_N if yes */
#define TRX_SYS_MYSQL_LOG_OFFSET_HIGH 4 /* high 4 bytes of the offset
within that file */
#define TRX_SYS_MYSQL_LOG_OFFSET_LOW 8 /* low 4 bytes of the offset
within that file */
#define TRX_SYS_MYSQL_LOG_NAME 12 /* MySQL log file name */
```
## 事务恢复流程
- Innodb在恢复的时候，不同状态的事务，会进行不同的处理(见trx_rollback_or_clean_all_without_sess函数)：
1. 对于TRX_COMMITTED_IN_MEMORY的事务，清除回滚段，然后将事务设为TRX_NOT_STARTED；
2. 对于TRX_NOT_STARTED的事务，表示事务已经提交，跳过；
3. 对于TRX_PREPARED的事务，要根据binlog来决定事务的命运，暂时跳过;
4. 对于TRX_ACTIVE的事务，回滚。

- MySQL在打开binlog时，会检查binlog的状态(TC_LOG_BINLOG::open)。如果binlog没有正常关闭(LOG_EVENT_BINLOG_IN_USE_F为1)，则进行恢复操作，基本流程如下
![image](https://github.com/ermaot/notes/blob/master/mysql/008%E4%BA%8B%E5%8A%A1/pic/Innodb%E4%B8%A4%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A42.png)
1. 扫描binlog，读取XID_EVENT事务，得到所有已经提交的XA事务列表(实际上事务在innodb可能处于prepare或者commit)；
2. 对每个XA事务，调用handlerton::recover，检查存储引擎是否存在处于prepare状态的该事务(见innobase_xa_recover)，也就是检查该XA事务在存储引擎中的状态；
3. 如果存在处于prepare状态的该XA事务，则调用handlerton::commit_by_xid提交事务
4. 否则，调用handlerton::rollback_by_xid回滚XA事务。

## 几个参数
- sync_binlog
Mysql在提交事务时调用MYSQL_LOG::write完成写binlog，并根据sync_binlog决定是否进行刷盘。默认值是0，即不刷盘，从而把控制权让给OS。如果设为1，则每次提交事务，就会进行一次刷盘；这对性能有影响(5.6已经支持binlog group)，所以很多人将其设置为100。
```
bool MYSQL_LOG::flush_and_sync()
{
int err=0, fd=log_file.file;
safe_mutex_assert_owner(&LOCK_log);
if (flush_io_cache(&log_file))
return 1;
if (++sync_binlog_counter >= sync_binlog_period && sync_binlog_period)
{
sync_binlog_counter= 0;
err=my_sync(fd, MYF(MY_WME));
}
return err;
}
```
- innodb_flush_log_at_trx_commit

![image](https://github.com/ermaot/notes/blob/master/mysql/008%E4%BA%8B%E5%8A%A1/pic/Innodb%E4%B8%A4%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A43.png)
- innodb_support_xa
用于控制innodb是否支持XA事务的2PC，默认是TRUE。如果关闭，则innodb在prepare阶段就什么也不做；这可能会导致binlog的顺序与innodb提交的顺序不一致(比如A事务比B事务先写binlog，但是在innodb内部却可能A事务比B事务后提交)，这会导致在恢复或者slave产生不同的数据。
```
int  innobase_xa_prepare(
/*================*/
/* out: 0 or error number */
THD* thd, /* in: handle to the MySQL thread of the user
whose XA transaction should be prepared */
bool all) /* in: TRUE - commit transaction
FALSE - the current SQL statement ended */
{
…
if (!thd->variables.innodb_support_xa) {
return(0);
}
```

## 参数讨论
- 上面3个参数三者都设置为1(TRUE)，数据才能真正安全。
- sync_binlog非1，可能导致binlog丢失(OS挂掉)，从而与innodb层面的数据不一致。
- innodb_flush_log_at_trx_commit非1，可能会导致innodb层面的数据丢失(OS挂掉)，从而与binlog不一致。

