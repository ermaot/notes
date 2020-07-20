本文参考：

https://baijiahao.baidu.com/s?id=1644815785057748879&wfr=spider&for=pc

https://blog.csdn.net/sinat_42483341/article/details/106908586

## show profile简介

1. show profile可以显示某一个具体的SQL各个步骤所花费的时间，非常实用
2. SHOW PROFILES将来会被Performance Schema替换掉，8.0目前还在支持中

## show profile使用方法

#### 1. 查看如何使用profile

```
 help profile
Name: 'SHOW PROFILE'
Description:
Syntax:
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY n]
    [LIMIT row_count [OFFSET offset]]

type:
    ALL
  | BLOCK IO
  | CONTEXT SWITCHES
  | CPU
  | IPC
  | MEMORY
  | PAGE FAULTS
  | SOURCE
  | SWAPS

The SHOW PROFILE and SHOW PROFILES statements display profiling
information that indicates resource usage for statements executed
during the course of the current session.

Profiling is controlled by the profiling session variable, which has a
default value of 0 (OFF). Profiling is enabled by setting profiling to
1 or ON:
```



#### #### 2.查看是否支持profile，并打开

```
>  select @@have_profiling;
+------------------+
| @@have_profiling |
+------------------+
| YES              |
+------------------+
1 row in set (0.00 sec)

-------以下是MySQL8的提示---------
> show warnings;
+---------+------+---------------------------------------------------------------------------+
| Level   | Code | Message                                                                   |
+---------+------+---------------------------------------------------------------------------+
| Warning | 1287 | '@@have_profiling' is deprecated and will be removed in a future release. |
+---------+------+---------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

打开profile

```
> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set (0.00 sec)

> set profiling=on;
Query OK, 0 rows affected (0.00 sec)

> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           1 |
+-------------+
1 row in set (0.00 sec)
```

#### 3.查看所需要跟踪的语句

```
> show profiles;
+----------+------------+------------------------------+
| Query_ID | Duration   | Query                        |
+----------+------------+------------------------------+
|        1 | 0.00022904 | select * from test.warehouse |
+----------+------------+------------------------------+
1 row in set (0.00 sec)
```

#### 4.查看profile

```
> show profile for query 1;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000063 |
| checking permissions | 0.000006 |
| Opening tables       | 0.000014 |
| After opening tables | 0.000004 |
| System lock          | 0.000004 |
| Table lock           | 0.000002 |
| After table lock     | 0.000005 |
| init                 | 0.000014 |
| optimizing           | 0.000006 |
| statistics           | 0.000024 |
| preparing            | 0.000008 |
| executing            | 0.000003 |
| Sending data         | 0.000048 |
| end                  | 0.000004 |
| query end            | 0.000006 |
| closing tables       | 0.000006 |
| freeing items        | 0.000004 |
| updating status      | 0.000007 |
| cleaning up          | 0.000003 |
+----------------------+----------+
19 rows in set (0.00 sec)

> show profile cpu ,memory,block io for query 1;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000063 | 0.000035 |   0.000028 |            0 |             0 |
| checking permissions | 0.000006 | 0.000003 |   0.000003 |            0 |             0 |
| Opening tables       | 0.000014 | 0.000008 |   0.000006 |            0 |             0 |
| After opening tables | 0.000004 | 0.000002 |   0.000002 |            0 |             0 |
| System lock          | 0.000004 | 0.000002 |   0.000001 |            0 |             0 |
| Table lock           | 0.000002 | 0.000001 |   0.000001 |            0 |             0 |
| After table lock     | 0.000005 | 0.000003 |   0.000003 |            0 |             0 |
| init                 | 0.000014 | 0.000008 |   0.000006 |            0 |             0 |
| optimizing           | 0.000006 | 0.000003 |   0.000002 |            0 |             0 |
| statistics           | 0.000024 | 0.000013 |   0.000011 |            0 |             0 |
| preparing            | 0.000008 | 0.000005 |   0.000003 |            0 |             0 |
| executing            | 0.000003 | 0.000001 |   0.000002 |            0 |             0 |
| Sending data         | 0.000048 | 0.000027 |   0.000021 |            0 |             0 |
| end                  | 0.000004 | 0.000002 |   0.000002 |            0 |             0 |
| query end            | 0.000006 | 0.000003 |   0.000002 |            0 |             0 |
| closing tables       | 0.000006 | 0.000003 |   0.000003 |            0 |             0 |
| freeing items        | 0.000004 | 0.000003 |   0.000002 |            0 |             0 |
| updating status      | 0.000007 | 0.000003 |   0.000003 |            0 |             0 |
| cleaning up          | 0.000003 | 0.000002 |   0.000001 |            0 |             0 |
+----------------------+----------+----------+------------+--------------+---------------+
19 rows in set (0.00 sec)
```

#### 5. 查看上一条SQL的profile

```
> show profile;
+-----------------+----------+
| Status          | Duration |
+-----------------+----------+
| starting        | 0.000032 |
| freeing items   | 0.000004 |
| updating status | 0.000007 |
| cleaning up     | 0.000002 |
+-----------------+----------+
4 rows in set (0.00 sec)
```

**show profile自身不会产生Profiling。**