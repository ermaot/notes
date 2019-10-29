## 一致性

- 数据库和分布式系统中都有一致性概念
- 数据库中的ACID，C是Consistency，这个C主要强调应用逻辑的一致性，比如应用定义的约束，包括外键等。
- 分布式系统的CAP以及一致性协议，也称为一致性。
1. 前者主要强调，读是否能读到最新，以及并发场景下操作执行的时序关系，主要包括线性一致性(linearizability)，顺序一致性(sequential consistency)，因果一致性(causal consistency)等

2. 后者主要强调“共识”，分布式中的多个节点对某个事情(选主，事务提交)达成一致，常见的共识算法包括paxos协议，raft协议等 

#### 线性一致性
线性一致性要求：
- 第一，“写后读”，这里写和读是两个操作，如果写操作在完成之后，读才开始，读要能读到最新的数据，而且保证以后也能读操作也都能读到这个最新的数据
- 第二，所有操作的时序与真实物理时间一致，即使不相关的两个操作，如果执行有先后顺序，线性一致性要求最终执行的结果也需要满足这个先后顺序
- 第三，如果两个操作是并发的(比如读A没有结束时，写B开始了)，那么这个并发时序不确定，但从最终执行的结果来看，要确保所有线程(进程，节点)看到的执行序列是一致的

#### 顺序一致性(sequential consistency)
- 对于单个线程，操作的顺序仍然要保留
- 对于多个线程(进程，节点)，执行的事件的先后顺序与物理时钟顺序不保证。但是要求，从执行结果来看，所有线程(进程，节点)看到的执行序列是一样的

#### 因果一致性(causal consistency)
相对于顺序一致性，弱化了不相关操作是否需要保序

#### 可串行化
- 隔离级别是纯粹数据库领域的概念与分布式系统并没有交集
- Serializable，是说并发场景下，多个并发事务最终执行的序列与某个串行执行的序列相同(无事务并发，事务的执行没有重叠)
- 数据库系统中实现可串行化调度也是解决冲突串行化问题。主要有两种：
1. 一种是基于S2PL(Strict 2 Phrase Locking)，事务操作过程中，对读加读锁，对写加写锁，事务提交时，才将锁释放，为了避免幻读，还需要实现间隙锁等；
2. 另外一种，是基于Snapshot的SSI隔离级别，这种实现与S2PL的主要区别在于，读仍然采用快照读，不加锁，读写不互斥

#### 线性一致性VS可串行化
- 讨论线性一致性时，我们讨论的**粒度是一个操作**，操作是否满足先后关系；而讨论隔离级别时，**粒度是一个事务**，事务是否与某个串行执行的结果相同
- 基于S2PL协议实现的可串行化，可以做到线性一致性兼得；而SSI由于是快照读，导致读不能读到最新，所以不满足线性一致性的
- 数据库要实现“线性一致性”，需要保证事务操作按全局时钟的先后顺序。对于写而言，通过一个统一的地方分配时间戳，显然先后执行的事务分配的时间戳也满足先后关系。这里实际上需要一个统一“全局时间源”，也就是业内常用的**TSO(TimeStampOracle)方案**
- 结合S2PL可以兼得可串行化+线性一致性，也就是实现Strict Serializable

#### External Consistency(外部一致性)
- External Consistency相比linearizability，主要是约束了非相关并发事务的提交顺序与物理时钟要保持一致，因为linearizability并不约束并发执行的操作

## MySQL测试
#### 概述
- MySQL自动测试框架是一个以MySQL框架和内部引擎为测试对象的工具。主要执行脚本在发布路径的mysql-test目录下。
- 自动测试框架的主要测试步骤，是通过执行一个case，将该case的输出结果，与标准的输出结果作diff

#### 样例
- 在mysql-test/t目录下创建自己的测试文件mytest.test
```
use test;
create table t(c int) engine=InnoDB ;
insert into t values(1);
select c from t;
drop table t;
```
- 在mysql-test/r目录下创建自己的测试文件mytest.result
```
use test;
create table t(c int) engine=InnoDB ;
insert into t values(1);
select c from t;
c
1
drop table t;
```
- 执行
```
# ./mtr mytest
Logging: /root/mysql-8.0.17/mysql-test/mysql-test-run.pl  mytest
MySQL Version 8.0.17
Checking supported features
 - Binaries are debug compiled
Using 'all' suites
Collecting tests
Checking leftover processes
Removing old var directory
Creating var directory '/root/mysql_debug/mysql-test/var'
Installing system database
Using parallel: 1

==============================================================================
                  TEST NAME                       RESULT  TIME (ms) COMMENT
------------------------------------------------------------------------------
worker[1] mysql-test-run: WARNING: running this script as _root_ will cause some tests to be skipped
[100%] main.mytest                               [ pass ]    346
------------------------------------------------------------------------------
The servers were restarted 0 times
The servers were reinitialized 0 times
Spent 0.346 of 29 seconds executing testcases

Completed: All 1 tests were successful.
```
- 说明
1. mysql-test/mtr这个文件，是一个perl脚本。同目录下还有 mysql-test-run 和mysql-test-run.pl这两个文件，这三个文件一模一样。
2. 每个case会启动一个mysql服务，默认端口为13000。如果这个case涉及到需要启动多个服务（比如主从），则端口从13000递增。
3. ./mtr的参数只需要指明测试case的前缀即可。当你执行./mtr testname会自动到t/目录下搜索 testname.test文件来执行。当然你也可以执行./mtr mytest.test， 效果相同。
4. mytest.test第一行是必须的，当你需要使用InnoDB引擎的时候。可以看到mtr能够解释source语法，等效于将目标文件的内容全部拷贝到当前位置。Mysql-test/include目录下有很多这样的文件，他们提供了类似函数的功能，以简化每个case的代码。注意souce前面的 –， 大多数的非SQL语句都要求加 。
5. mytest.test最后一行是删除这个创建的表。因为每个case都要求不要受别的case影响，也不要影响别的case，因此自己在case中创建的表要删除。
6. mtr会将mytest.test的执行结果与r/mytest.result作diff。 若完全相同，则表示测试结果正常。注意到我们例子中的mytest.result. 其中不仅包括了这个case的输出，也包括了输入的所有内容。实际上有效输出只有5、6两行。如果希望输出文件中只有执行结果，可以在第一行后面加入 — disable_query_log

#### 批量执行
- ./mtr
1. 会执行所有的case。包括t/目录下所有以.test为后缀的文件，也包括 suits目录下的所有以.test为后缀的文件。
2. 只要任何一个case执行失败（包括内部执行失败和与result校验失败）都会导致整个执行计划退出。
3. –force，加入这个参数，mtr会忽略错误并继续执行下一个case直到所有的case执行结束。

- ./mtr –suite=funcs_1
1. Suits目录下有多个目录，是一些测试的套餐。此命令单独执行 suits/funcs_1目录下的所有case。（其他目录不执行）
2. t/目录下的所有文件组成了默认的套餐main。 因此 ./mtr –suite=main则只执行t/*.test.
- ./mtr  –do-test=events
1. 执行所有以 events为前缀的case（搜索范围为t/和所有的suite）。
2. –do-test的参数支持正则表达式，上述命令等效于./mtr –do-test=events.*
3. 如果想测试所有的包括innodb的case，可以用 ./mtr –do-test=.*innodb.*

#### 其他说明
##### 文件说明
1. 目录下的mtr文件即为mysql-test-run的缩写，文件内容相同
2. 输入和输出分别放在不同的文件中，执行结果与输出文件内容作对比。输入文件都在t目录下，输出文件在r目录下。对应的输入输出文件仅后缀名不同，分别为 *.test 和 *.result
3. t目录下的*.opt文件是指在这个测试中，mysql必须以opt文件的内容作为测试参数启动。
4. *.sh文件是在执行启动mysql-server之前必须提前执行的脚本。disabled.def中定义了不执行的testfile
5. r目录下， 若一个执行输出结果和testname.result文件不同，会生成一个testname.reject文件。 该文件在下次执行成功之后被删除
6. mysql-test/var/log/文件夹下，如果用例执行失败，会生成testname.log文件。该文件在下次执行成功之后被删除
7. include目录是一些头文件，这些文件在 t/*.test 文件中使用，用source 命令引入
8. lib目录是一些库函数，被mtr脚本调用
9. 有些测试需要用到一些标准数据，存在std_data目录下
10. Suite目录也是一些测试用例，每个目录下包含一套，./mtr –suite=funcs_1执行suits/funcs_1目录下的所有case
11. 每个testfile是一个测试用例，多个测试用例之间可能有关联。 一个file中任何一个非预期的失败都会导致整个test停止（使用force参数则可继续执行）
12. Mtr所在的目录路径中不能有空格
13. Test文件名由字母数字、下划线、中划线组成，但只能以字母数字打头

##### 执行说明
1. Mtr会启动mysql，有些case下可能重启server，为了使用不同的参数启动（这个如何实现？）
2. 调用bin/mysqltest读case并发送給mysql-server
3. 注意如果服务端输出了未过滤的warning或error，则会也会导致test退出
4. Mtr实际调用mysqltest作测试。 –result-file文件用于指定预定义的输出，用于与实际输出作对比。若同时指定了 –recored参数，则表示这个输出数据不是用来对比的，而是要求将这个输出结果写入到指定的这个文件中
5. 执行./mtr时另外启动了一个mysql server，默认端口13000，如果这个case涉及到需要启动多个服务（比如主从），则端口从13000递增
6. Mtr允许并行执行，端口会从13000开始使用。但需要特别指定不同的—vardir指定不同的日志目录。但并行执行的case可能导致写同一个testname.reject.
7. 使用–parallel=auto可以多线程执行case
8. Mtr对比结果使用简单的diff，因此自己编写的测试case不应该因为执行时间不同而导致结果不同。当然框架在执行diff之前，允许自定义处理规则对得到的result作处理，来应对一些变化
9. ./mtr –record test_name 会将输出结果存入文件 r/test_name.result
10. 自己在脚本中创建的库、表等信息，要删除，否则会出warnning
11. Mtr默认使用的是mysql-test/var/my.cnf文件，需要替换该文件为自定义的配置文件
12. 若要使用innodb引擎，必须明确在testname.test文件头加– source include/have_innodb.inc
13. — sleep 10 等待10s
14. 默认情况下， r/testname.result中会包含原语句和执行结果，若不想输出原语句，需要自t/restname.test文件头使用 -disable_query_log
15. 每个单独的case会重启服务并要求之前的数据是清空的
16. 如果要测试出错语句，必须在testname.test文件中，在会出错的语句之前加入 –error 错误号
```
比如重复创建表，语句如下
create table t(c int) engine=InnoDB ;
– error 1050
create table t(c int) engine=InnoDB ;
这样在testname.result中输出
create table t(c int) engine=InnoDB ;
  ERROR 42S01: Table ‘t’ already exists
则能够正常通过
也可使用 ER_TABLE_EXISTS_TABLE （宏定义为1050）
也可使用 –error S42S01 （注意需要加前缀S），但S系列的可能一个错误号对应多种错误
```
17. –enable_info 在testname.test文件头增加这个命令，在结果中会多输出影响行数。 对应的关闭命令为 –disable_info
18. enable_metadata 可以显示更多信息 
19. disable_result_log不输出执行结果
20. 有些case的输出结果可能包含时间因素的影响，导致无法重复验证。–replace_column 可以解决部分情况
```
–replace_column 1 XXXXX
Select a, b from t;
输出结果会是
XXXXX     b.value
即将第一列固定替换为xxxxx。 注意，每个replace_column的影响范围仅局限于下一行的第一个select语句。
```
21. mtr –mysqld=–skip-innodb –mysqld=–key_buffer_size=16384 用这种将参数启动传递給mysql server 。 每个选项必须有一个—mysqld打头，不能连在一起写，即使引号包含多个也不行
22. mysql-test-run.pl –combination=–skip-innodb   –combination=–innodb,–innodb-file-per-table。这个命令是参数传给多个test case， 第一个参数传 skip-innodb， 第二个参数传 –innodb, innodb-file-per-table。  若所有启动参数中combination只出现一次，则无效
23. skip-core-file 强行控制server不要core
24. 如果需要在server启动前执行一些脚本，可以写在 t/testname.sh文件中，由mtr自动执行
25. Mysql-stress-test.pl用于压力测试，注意默认的my.cnf中的参数，比如innodb_buffer_pool_size只有128M
26. 在一个case中重启server
```
–exec echo “wait” > $MYSQL_TMP_DIR/mysqld.1.expect
–shutdown_server 10
–source include/wait_until_disconnected.inc
# Do something while server is down
–enable_reconnect
–exec echo “restart” > $MYSQL_TMP_DIR/mysqld.1.expect
–source include/wait_until_connected_again.inc
```
27. 循环语句语法
```
let $1= 1000;
while ($1)
{
 # execute your statements here
 dec $1;
}
```
28. mysql_client_test是一个单独的case，里面包含了多个case， 但并不是写成脚本，而是直接调用，可以直接从源码中看到里面调用的各个语句。源码位置testclients/mysql_client_test.cc
29. Case完成后，mtr会检测server的错误日志，如果里面包含Error或Warning，则会认为case fail
30. 如果要忽略整个验证server日志的过程，可以在文件头增加 –nowarnings
31. 如果要指定忽略某些行，允许使用正则表达式，比如 call mtr.add_suppression(“The table ‘t[0-9]*’ is full”); 能够忽略 The table ‘t12′ is full 这样的warning
32. call语句也是输入语句的一部分，因此会输出到结果内容中，除非
```
–disable_query_log
call mtr.add_suppression(“The table ‘t[0-9]*’ is full”);
–enable_query_log
```

##### 参考链接
http://dev.mysql.com/doc/mysqltest/2.0/en/mysqltest-commands.html



## 测试分布式系统的线性一致性

#### 专门测试

来自 https://www.jianshu.com/p/bddfce1494d6 

我们实际如何做正确操作的测试呢？在最简单的软件里面，我们可以使用输入输出测试，譬如 `assert(expected_output == f(input))`，我们也可以在分布式系统上面使用一个类似的方法，譬如，对于 key-value store，当多个 client 开始执行操作的时候，我们可以有如下的测试

```
for client_id = 0..10 {
    spawn thread {
        for i = 0..1000 {
            value = rand()
            kvstore.put(client_id, value)
            assert(kvstore.get(client_id) == value)
        }
    }
}
wait for threads
```

- 在 [Jepsen](https://link.jianshu.com?t=http://jepsen.io/) 里面，有一个一致性验证工具 [Knossos](https://link.jianshu.com?t=https://github.com/jepsen-io/knossos)，但在测试一些分布式 key-value store 的时候，Knossos 并不能很好的工作，它可能只能适用于一些少的并发 clients，以及只有几百的事件的历史。

- 为了解决 Knossos 的问题，作者开发了 [Procupine](https://link.jianshu.com?t=https://github.com/anishathalye/porcupine)，一个用 Go 写的更快的线性一致性验证工具。Porcupine 使用一个用 Go 开发的执行规范去验证历史是否是线性的。根据实际测试的情况，Porcupine 比 Knossos 快很多倍

#### 错误注入

  

## jepsen



## porcupine

 来自：https://www.jianshu.com/p/9aedd234ef62

Porcupine 的使用非常简单，就两个验证函数，CheckOperations 和 CheckEvents 

#### Event

 Event 的定义如下： 

```
type Event struct {
    Kind  EventKind
    Value interface{}
    Id    uint
}
```

- Kind 是 event 的类型，就两种，Call 和 Return，因为我们的任何操作，其实就是先发起一个 request，也就是 Call，然后收到 response，也就是 Return。

- Value 是一个 interface，也就是我们自己实际操作的 request 或者 response。

- Id 就是这个 event 的标识，request 和 response 必须用相同的 Id。假设我们实现一个简单的 Key-Value store，就只会对一个 key 操作

- 定义 request 和 response 如下

```
type Request struct {
// 0 for read, 1 for put
Op int
// put value
Value int
}

type Resposne struct {
// for read value
Value int
// for put ok
Ok bool
}
```

  

 一个 event history 可能如下： 

```
Event {EventKind: CallEvent, Value: Request { Op: 0 }, Id: 1 }
Event {EventKind: CallEvent, Value: Request { Op: 1, Value: 10 }, Id: 2 } 
Event {EventKind: ReturnEvent, Value: Response { Value: 5 }, Id: 1 }
Event {EventKind: ReturnEvent, Value: Response { Ok: true }, Id: 2 }
```

 **将对应的 event list 扔给 CheckEvents 函数**，就能知道这个 history 是否是线性一致性的了 

## Model

 Model 的定义如下: 

```
type Model struct {
    // Initial state of the system.
    Init func() interface{}
    // Step function for the system. Returns whether or not the system
    // could take this step with the given inputs and outputs and also
    // returns the new state. This should not mutate the existing state.
    Step func(state interface{}, input interface{}, output interface{}) (bool, interface{})
}
```

-  去掉了一些无关的 field function，主要关注两个，Init 和 Step 

-  Init 是整个系统初始的状态 

-  Step 其实就是一个状态机变更，它会根据当前 event 的 input 和 output 尝试从一个 state 变更到另一个 state。返回 true 就标明变更成功，进入一个新的状态了 

  ```
  func Step(state interface{}, input interface{}, output interface{}) (bool, interface{})
          st := state.(int)
          inp := input.(Request)
          out := output.(Response)
          // for read
          if inp == 0 {
              return st == out.Value, st
          }
          
          // for write
          if !out.ok {
              return true, st
          }
          return out.ok, inp.value
  ```

  

## Timeout

-  在实际的分布式环境中，一个操作，我们有三种状态，成功，失败，超时 
-  如果我们碰到了一个是 timeout 的 response，我们应该暂存起来，然后在整个 history 的最后，才把这个 response 加进去，这样 Porcupine 才能正确处理 