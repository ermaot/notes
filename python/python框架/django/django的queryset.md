## queryset简述

1. queryset是懒加载的
2. queryset是链式调用的，即执行方法后返回的还是这个对象



## queryset方法

#### 链式方法

方法|说明
---|---
all接口|相当于 SELECT *  FRoM table_name语句，用于查询所有数据。
filter接口|根据条件过滤数据，常用的条件基本上是字段等于、不等于、大于、小于。<br>还有能改成产生LIKE查询的：Model.objects.filter( content\_\_contains = "条件 )
exclude接口|同 filter，只是相反的逻辑。
reverse接口|把 Query set中的结果倒序排列。
distinct接口|用来进行去重查询，产生 SELECT DISTINCT这样的SQL查询。
none接口|返回空的 query_set

#### 非链式方法
方法|说明
---|---
get|Post.objects.get(id=1)。如果记录不存在会抛出异常，所以一般配合try使用
create|直接创建一个model对象
get_or_create|根据条件查找，如果没有找到就创建
update_or_create|同get_or_create
count|返回queryset的记录条数
latest|返回最新的一条记录
earliest|返回最早的一条记录
first|从当前queryset记录中获取第一条
last|从当前queryset获取最后一条
exists|返回True或者False
bulk_create|同create，用来批量查询数据
in_bulk|批量查询。根据id_list中的id 去返回数据库中和id相对应的数据返回的是一个dict类型{ id : id对应的数据 }。Post.objects.in_bulk([1,2,3])
update|会触发django的signal
delete|会触发django的signal
values|明确知道返回某个字段的值。title_list = Post.objects.filter(catagory_id).values('title')。返回的是dict类型的queryset<QuerySet [{'title':XXX},]
value_list| 返回的是tuple的QuerySet。如果只是一个字段，可以加flat=True参数

#### 进阶接口
方法|说明
---|---
defer|把不需要展示的字段做延迟加载（可能产生N+1问题）
only|与defer相反
select_related|
prefetch_related|

#### 常用字段查询
使用方法类似于
```
Post.object.filter(content__contains='查询条件')
```
方法|说明
---|---
contains|包含，相似查询
icontains|包含，忽略大小写
exact| 
iexact| 
in| 
gt| 
gte| 
lt| 
lte| 
startswith|
istartswith|
endswith|
range|Post.objects.filter(created_time__range=('2018-05-01','2018-06-01'))

#### 进阶查询

##### F表达式
```
post = Post.objects.get(id=1)
post.pv = post.pv+1
post.save()
```
上面的代码在多线程的情况下会有并发问题
应该使用下面
```
from django.modles import F
post = Post.objects.get(id=1)
post.pv = F('pv')+1
post.save()
```
这样会产生update ……set pv=pv+1这样的原子操作语句

##### Q表达式
```
from django.modles import Q
Post.objects.filter(Q(id=1)|Q(id=2))
Post.objects.filter(Q(id=1)&Q(id=2))
```

#### Count
```
from django.modles import Count
categories = Category.objects.annotate(posts_count = Count('post'))
print(categories[0].posts_count)
```
##### Sum
```
from django.modles import Sum
Post.objects.aggrerate(all_pv=Sum('pv'))
```
##### 其他
Avg、Min、Max