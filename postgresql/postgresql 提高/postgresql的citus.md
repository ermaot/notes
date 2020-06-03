citus能横向扩展多租户数据库，构建实时应用程序，是开源pg_shard的升级版本。

本文部分来自：https://blog.csdn.net/qingyafan/java/article/details/88068559

## Citus是什么？主要做什么？

Citus是PostgreSQL的一个插件，通过citus，你可以让多个PostgreSQL机器组成一个集群，利用这个集群，可以将一张大数据量的表自动水平分表，而无需担心分配的逻辑。和其他类似的基于PostgreSQL的分布式方案，比如GreenPlum，PostgreSQL-XL，PostgreSQL-XC相比，citus最大的不同在于citus是一个PostgreSQL扩展而不是一个独立的代码分支。 因此，citus可以用很小的代价和更快的速度紧跟PostgreSQL的版本演进；同时又能最大程度的保证数据库的稳定性和兼容性.具体citus可以做如下的事情：

1. 自动分片和分布数据。

你可以选择一个列，citus依据这个列将大数据表进行分片（sharding），然后将各个分片分配到各个worker节点；可以实现数据高可用。通过设置"citus.shard_replication_factor"控制每个分片的副本数量，每个分片的副本会被分配到不同的机器，如果包含该分片副本的某个机器宕机，数据还是查询的到，除非所有包含该副本的机器全部宕机，数据才会不可用。而且查询来了，citus也会对查询进行处理

2. 自动分割任务。

如果查询是针对的某条记录，citus会根据分布数据时记录的元数据，只到相应的分片（sharding）去查询，那么查询的数据量就降下来了，查询速度会快很多；

3. 并行查询。

## Citus优缺点
Citus适合做单表查询，且该单表数据量越大，Citus的优势就越明显。Citus集群相对于单机PostgreSQL，对SQL有一些支持不完善的地方。

## citus安装

1. yum源安装

```
# curl https://install.citusdata.com/community/rpm.sh  | sudo bash
# sudo yum install citus
```

2. 配置环境变量

```
# sudo su - postgres
# export PATH=$PATH:/usr/pgsql/bin
```

