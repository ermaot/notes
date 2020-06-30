本文参考：

https://blogread.cn/it/article/3495?f=

https://www.cnblogs.com/luozhiyun/p/11216287.html

## 一致性读

InnoDB是支持MVCC多版本一致性读的，因此和其他实现了MVCC的系统如Oracle，PostgreSQL一样，读不会阻塞写，写也不会阻塞读。虽然同样是MVCC，各家的实现是不太一样的。Oracle通过在block头部的事务列表，和记录中的锁标志位，加上回滚段，个人认为实现上是最优雅的方式。 而PostgreSQL更是将多个版本的数据都放在表中，而没有单独的回滚段，导致的一个结果是回滚非常快，却付出了查询性能降低的代价。InnoDB的实现更像Oracle，同样是将数据的旧版本存放在单独的回滚段中，但是也有不同。



## 原理简述

1. InnoDB表会有三个隐藏字段，6字节的DB_ROW_ID，6字节的DB_TX_ID，7字节的DB_ROLL_PTR（指向对应回滚段的地址），一致性读主要跟后两者有关系。InnoDB内部维护了一个递增的tx id counter
2. InnoDB每个事务在开始的时候，会将当前系统中的活跃事务列表（trx_sys->trx_list）创建一个副本（read view）
3. 然后一致性读去比较记录的tx id的时候，并不是根据当前事务的tx id，而是根据read view最早一个事务的tx id（read view->up_limit_id）来做比较的，这样就能确保在事务B之前没有提交的所有事务的变更，B事务都是看不到的。当然，这里还有个小问题要处理一下，就是当前事务自身的变更还是需要看到的。