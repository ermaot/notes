## MySQL删除表
分两个过程：
- buffer pool页面清除。只需要将页面从flush队列删除即可
1. 删除线程首先根据表的space id，从buffer pool中持有每一个buffer pool实例的锁并遍历该实例
2. 在flush list中找到属于被删除表的页面
3. 然后将页面从flush list中删除，并且将oldest_modification设置为0，表示失效（可重新使用）
4. 如果buffer pool很大，或者是buffer pool有很多需要被flush的页面，那么遍历扫描就会很花时间导致响应buffer pool实例被阻塞
- 删除ibd磁盘文件
#### 代码过程：
1. 通过buf_pool_mutex_enter(buf_pool)函数持有buffer pool mutext
2. 通过buf_pool_list_mutex_enter(buf_pool)函数持有buffer pool中的flush list mutex
3. 扫描flush list表，删除被删表的脏页；如果cpu和mutex时间过长，则调用buf_flush_try_yield函数释放cpu、flush list mutex和buffer pool mutex，并用os_thread_yield()函数强行上下文切换，然后重新持有buffer pool mutex和flush list mutex
4. 释放flush list mutex，然后释放buffer pool mutex
#### 删除大文件替代办法
硬链接法
1. 创建ibd文件的硬链接
2. 删除表（同时删除了ibd文件），这一步很快。因为此时只减少了一个文件的硬链接而已
3. 删除硬链接文件
#### galera cluster
galera cluster 有一个wsrep_on参数可以控制本地操作是否需要复制到其他节点，如果使用硬链接方法删除文件需要将该配置设置为off，再处理

## MySQL常见死锁
