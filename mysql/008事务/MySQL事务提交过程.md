## 入口
无论对于dml语句【insert，update，delete等】还是dcl语句【commit，rollback】，mysql提供了公共接口mysql_execute_command
```
mysql_execute_command
{
   switch (command)
   {
       case SQLCOM_INSERT:
                mysql_insert();
                break;
  
       case SQLCOM_UPDATE:
                mysql_update();
                break;
  
       case SQLCOM_DELETE:
                mysql_delete();
                break;
       ...... 
   }
    
   if thd->is_error()  //语句执行错误
     trans_rollback_stmt(thd);
  else
    trans_commit_stmt(thd); 
}
```
## 语句级提交
语句级提交，对于==非自动模式提交情况下==，主要作两件事情
1. 释放autoinc锁，这个锁主要用来处理多个事务互斥地获取自增列值，因此，无论最后该语句提交或是回滚，该资源都是需要而且可以立马放掉的。
2. 标识语句在事务中位置，方便语句级回滚

commit源码
```
mysql_execute_command
    trans_commit
ha_commit_trans(thd, FALSE);
{
    TC_LOG_DUMMY:ha_commit_low
        ha_commit_low()    
            innobase_commit
            {
                //获取innodb层对应的事务结构
                trx = check_trx_exists(thd);
 
                if(单个语句，且非自动提交)
                {
                     //释放自增列占用的autoinc锁资源
                     lock_unlock_table_autoinc(trx);
                     //标识sql语句在事务中的位置，方便语句级回滚
                     trx_mark_sql_stat_end(trx);
                }
                else 事务提交
                {
                     innobase_commit_low()
                     {   
                        trx_commit_for_mysql(); 
                            trx_commit(trx);  
                     }
 
　　　　　　　　　　　　//确定事务对应的redo日志是否落盘【根据flush_log_at_trx_commit参数，确定redo日志落盘方式】
                     trx_commit_complete_for_mysql(trx);
　　　　　　　　　　　　　　trx_flush_log_if_needed_low(trx->commit_lsn);
　　　　　　　　　　　　　　　　log_write_up_to(lsn); 
                }
            }
}
trx_commit
    trx_commit_low
        {
            trx_write_serialisation_history
            {
                trx_undo_update_cleanup //供purge线程处理，清理回滚页
            }
            trx_commit_in_memory
            {
                lock_trx_release_locks //释放锁资源
                trx_flush_log_if_needed(lsn) //刷日志
                trx_roll_savepoints_free //释放savepoints
            }
        }
```

提交过程中，主要做了4件事情，
1. 清理undo段信息。对于innodb存储引擎的更新操作来说，undo段需要purge，这里的purge主要职能是，真正删除物理记录。在执行delete或update操作时，实际旧记录没有真正删除，只是在记录上打了一个标记，而是在事务提交后，purge线程真正删除，释放物理页空间。因此，提交过程中会将undo信息加入purge列表，供purge线程处理。
2. 释放锁资源，mysql通过锁互斥机制保证不同事务不同时操作一条记录，事务执行后才会真正释放所有锁资源，并唤醒等待其锁资源的其他事务；
3. 刷redo日志，前面我们说到，mysql实现事务一致性和持久性的机制。通过redo日志落盘操作，保证了即使修改的数据页没有即使更新到磁盘，只要日志是完成了，就能保证数据库的完整性和一致性；
4. 清理保存点列表，每个语句实际都会有一个savepoint(保存点)，保存点作用是为了可以回滚到事务的任何一个语句执行前的状态，由于事务都已经提交了，所以保存点列表可以被清理了。