# redis查询key的内存大小

通过redis-rdb-tools工具进行查询。

```
pip install rdbtools
```

可能会出现

```
WARNING: python-lzf package NOT detected. Parsing dump file will be very slow unless you install it. To install, run the following command:

pip install python-lzf

```

安装python-lzf，然后可以查看键的大小

```
# rdb -c memory /var/lib/redis/dump.rdb
database,type,key,size_in_bytes,encoding,num_elements,len_largest_element,expiry
0,string,btmp,14384,string,12501,12501,
0,sortedset,test,69,ziplist,2,1,
0,string,a,56,string,2,2,

```

