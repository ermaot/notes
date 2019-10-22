Raft的总体目标是将log完全一样地复制到集群中的所有机器上，用来创建所谓的Replicated State Machine(多副本状态机，就是具有多个copy的应用程序)

## replicated state machine思想
- 设想你有一个程序或应用，你想让它非常可靠，其中一个办法就是，在多个机器上同时运行这同一个程序，并且保证完全一样地运行
- 执行过程
1. 如果一个客户端，想执行一个command（命令，指令，指的是具体的某个操作请求），那么它可以请求其中一台state machine，这台machine就把这个命令，如command X，记录到自己的本地日志log中
2. 把command X传递给其他所有machines
3. 其他machine也在各自的log中记录下这条命令
4. 一旦这条command X被safely replicated(安全地复制)到所有machine的所有log中，那么这个command X就可以被传递给各个machine开始执行
5. 一旦有machine执行完成命令X，就会把结果返回给客户端
- 说明
1. 所有机器上都部署相同的machine应用
2. 因为log是复制过去的，安全地复制保证了所有机器上的所有machine的log都完全一样
3. 所有机器上的所有machine都按照log中的相同顺序执行命令
4. log是相同的，log中相同位置的命令也是相同的
5. 所有机器上的所有machine都一样，执行的命令也一样，输出的结果也必定完全一样。
6. 这里还有个隐含条件，就是machine应用必须是deterministic的，即确定性的，由输入必定能推出唯一的输出，不能带有随机性模糊性

- Consensus Module（一致性模块，machine里的一个协调控制模块）的工作职责是管理这些logs，确保被合理地复制，并决定什么时候将这些logs中的command提交给state machine执行（其实consensus module就是一个协调模块，相当于整个系统的大脑，是整个算法的控制核心）
- 为了保证效率，只要求大多数机器在线，并能互相通信即可
- 机器可以crash，可以停止运行，可以暂停运行再过段时间恢复，但要求运行的时候必须正常（意思是你可以罢工，但你在岗的时候干活儿必须正确），不能处理Byzantine failures（拜占庭将军问题，指的是恶意篡改数据这类行为）
- 允许网络中断、消息丢失、消息延迟、消息传递乱序、网络分化等，不能处理的就是恶意篡改数据这类，比如受到恶意网络攻击、或者你恶意篡改磁盘中的log日志等这类非正常行为

## 分布式一致性算法
- 实现一致性算法有两种可能方式：
1. 对称式，或无leader：这种方式中所有server的角色完全一样，权利也一样，在任意时刻的行为也基本一样，所有server都是对等的；客户端可以请求任意一台server，将command写入log中，并复制到其他server
2. 非对称式，或leader-based：在任意时间各个server都是不平等的，某个时刻只有一个server是leader，管理集群中的所有操作；其他server都是被统治的，只能简单按照leader的旨意干活；在这类系统中，客户端只能与leader通信，也只有leader能与其他server交流

- raft采用的就是==第二种leader-based的==方式，并且把一致性问题分解为两个不同问题：
1. 在有leader的情况下集群如何正常运作
2. leader挂掉之后如何选举更换leader。
- raft采用的这种leader-based方式的优势是使得正常运作过程非常简单，raft算法的所有复杂性其实都来自于leader变更

## raft详解
#### raft的6个问题
- leader选举：如何从多个server中挑选一台作为leader；当leader挂掉之后，如何感知到，并挑选出新的leader来替换旧leader
- 正常运行（最基本的log复制过程）：leader从客户端收到请求后如何将log复制到其他机器
- leader变更过程中的安全性和一致性：leader变更是raft中最难的，也是最关键的；首先会讲下safety到底意味着什么，以及如何保证safety；接着讲新leader上任后如何处理log，使得整个系统恢复一致性状态
- neutralize 旧leader：这是leader变更中的另一个问题，就是旧leader并没有真的死掉，死灰复燃，重新恢复之后，我们该如何处理
- 客户端交互：客户端如何与整个系统交互？关键点是请求过程中server挂掉了client怎么办？如何实现linearizable semantics（线性化语义），即每个客户端命令只能执行一次（once and exactly once，防止出现多次执行出问题，类似于幂等性概念）
- 配置变更：如何在集群中新增、或删除机器

#### raft的server角色
系统中的任意一个server，只能是以下3种状态中的一种：
1. leader：同一时刻至多只能由一个server处于leader状态，处理所有的客户端交互、日志复制等；（同一时刻至多一个，意思是要么一个leader，要么没leader
2. follower：大部分时候集群中的绝大部分server都是follower的状态，这些server是完全被动的状态，即不能主动发出请求，只能响应来自其他server的请求；（意思就是不能主动问别人，只能别人问了你回答；don't ask me! I ask you, you answer only!）
3. candidate：这是一个从follower到leader之间的中间状态；只是leader选举过程的一个临时状态；
正常情况下的集群状态应该是：1==个leader，其他所有都是follower==

#### raft遵循的性质
- Election Safty ：每一个任期内只能有一个领导人
- Leader Append-Only：leader只能追加日志条目，不能重写或者删除日志条目
- Log Matching：如果两个日志条目的index和term都相同，则两个如果日志中，两个条目及它们之前的日志条目也完全相同
- Leader Completeness：如果一条日志被commited过，那么大于该日志条目任期的日志都应该包含这个点
- State Machine Safety：如果一个server将某个特定index的日志条目交由状态机处理了，那么对于其他server，交由状态及处理的log中相同index的日志条目应该相同

##### 如何保证Election Safty
raft中，只要candidate获得多数投票，就可以成为领导人。follower在投票的时候遵循两个条件：
- 先到先得：cadidate的term大于follower的term，或者两个term相等，但cadidate的index大于follower的index
- 对于选举结果：如果票被瓜分，产生两个leader，这次选举失效，进行下一轮的选举；只有一个leader的情况是被允许的

如何保证在有限的时间内确定出一个候选人，而不会总是出现票被瓜分的情况？raft使用随机选举超时(randomize election timeouts)，使得每个server的timeout不一样。发起新一轮选举的时候，有些server还不是voter，或者一些符合条件的candidate还没有参加下一轮。

##### 如何保证Log Matching
Leader在进行AppendEntry RPCs的时候，这个消息中会携带preLogIndex和preLogTerm这两个信息，follower收到消息的时候，首先判断它最新日志条目的index和term是否和rpc中的一样，如果一样，才会append.


##### 如何保证Leader Completeness
这段有点长

##### raft协议中有一个约定，不能提交之前任期内log entry作为commit点
为何？

##### cluster membership changes
raft的解决方法就是two phase approach，引入一个过度配置，称为==共同一致状态==
- leader收到更新配置请求的时候，产生一个（old,new）entry，并append进日志通过rpc让follower追加
- 这条日志如果顺利，将这条日志commit产生new entry, append到日志通过rpc让follower追加
- 如果顺利commit，从而完成新配置的生成
#####  log过长或日志回放时间过长怎么办
- 需要log compaction
- raft采用的方法是写时复制的snapshot(写是复制在linux中可以通过fork来完成)
- 写时复制主要是处于性能考虑的，如果state machine数据太多，snapshot将会耗费大量的时间，也许会导致系统可用性大大降低

## raft 的 term
- 时间被划分成一个个的term（类似于zookeeper中的epoch，指的就是任期、时期、年代，某一个leader的统治时期），
- 每个term都有一个number，这个number必须单向递增且从未被用过；
- 每个term时期，分两部分：
1. 为这个term选举leader的过程，
2. leader在这个term的剩余时间内作为leader管理整个系统
- raft保证一个term内只有一个server可以被选举成leader；但有些term内可能没有选出leader
- raft中的每个server必须保存current term值（当前年代、当前任期号），这个值是当前server所认为的（best guess，为什么说是guess，是因为有时候比如server断网又恢复了，它其实是不知道当前term的，只能猜测现在还处于之前的term中，而这个猜测不一定是对的）当前系统所处于的term
- term值必须被可靠地存储在磁盘中，以保证server宕机重启之后该值不丢失
- term的作用非常重要，其核心作用是让raft能够及时识别过期信息
- 我们的系统认为来自于==更大term的信息一定是更准确的==，总是采纳来自于最新term的信息

## 各个server之间如何交互
- raft中所有server之间的通信都是RPC调用
- 只有两种类型的RPC调用
1. 第一种是RequestVote，用于选举leader
2. 第二种是AppendEntries，用于normal operations中leader向其他机器复制log

## raft算法各个过程
#### 启动
- 当整个系统启动的时候，所有的server都是follower状态
1. follower是完全被动的，它不会主动尝试联系其他server，只能被动响应来自其他server的信息
2. follower为了保持在自身的follower状态，它必须要相信集群中存在一个leader
3. follower唯一可能的通信方式就是接收来自leader或candidate的请求
- leader为了保持自身的权威，必须不停地向集群中其他所有server发送心跳包，在raft中，心跳信息非常简单，就是不带数据的AppendEntries RPC请求
- 如果过了一段时间，某个follow还一直没收到任何RPC请求，那么它就会认为集群中已经没有可用的leader了，它就会发起选举过程，争取自己当leader；follow的等待时间，就叫election timeout（选举超时），一般是100-500ms

#### 选举
##### 选举过程
- 当一个server开始竞选leader，第一件事就是增大current term值（全局单向递增）
- 为了进行竞选，这个follower必须先转变到candidate状态；在candidate状态中，该server只有一个目标，就是争取自己当leader；为了当leader，它必须要争取到大多数投票
- 先投自己一票
- 发送RequestVote RPC请求到其他所有server，如果请求发送出去之后没有响应，就会不断重试，一直发，直到出现下面三种情况之一：
1. 该candidate得到大多数server的投票：这是我们最希望出现的情况，也是绝大部分情况下会出现的情况；一旦投票过半，则该candidate立即变成leader状态，且同时立即向其他所有follower发送心跳包
2. 该candidate收到了来自leader的RPC请求：这说明有其他candidate在同时跟自己竞选leader，并且已经竞选成功，则该candidate立即回到follower状态，接收来自leader的请求，并被动响应
3. 过了election timeout的时间，以上两种情况都没发生即回到最初步骤中，增加当前current time值，开始新一轮的竞选

##### 选举原则
- safety：安全性，意思是说在任意一个给定的term内，最多只允许一个server获胜成为leader。为了保证这一点，需要两个条件：
1. 任意一个server在一个term内只能投出一票，投给最先到请求到它的candidate；同时持久化投票信息持久保存到磁盘上，保证即使该server投完票后宕机，也不会在同一个term内给第二个candidate投票了。
2. 只有获得大多数投票才能获胜
- liveness：为了保证系统能向前运行，我们要确保不能一直都是无leader状态，必须要能最终选出一个leader，确保不要总是出现splited vote（投票分散）
1. raft的解决办法很简单，不要让所有server的election timeout都相同，而是在T到2T之间随机选择超时时间（T就是election timeout，这个值通常要比系统中最快的机器的超时时间短）
2. 每个server每次都用随机方法计算出超时时间
3. 这种办法在T远大于broadcast time（传播时间，指的是server发起投票、收到投票所花时间）的情况下效果尤其明显


#### normal operation
##### log概念
log：
- 每个server都会保存一份自己私有的log：leader有，各个followers也都有；这些log保存在各自的机器上，只供自己操作
- log由entry组成，每个entry都有一个index，用来标记该entry在log中的位置；（就是数组与元素的关系）
- 每个entry内包含两个东西：
1. command：即可以在state machine上执行的具体指令；指令的格式是由客户端和state machine协商制定的，consensus module毫不关心，你可以把它想象成是某个方法和对应的参数；
2. term number：标识该条entry在哪个leader的term内被创建
- log必须被持久可靠地保存，即log必须保存在磁盘等可靠的存储介质中
- server每次收到有log的变更请求，server必须操作完成，并确保log被安全保存之后才再返回请求
- 如果某个entry在集群中大多数server中都有，那么我们认为这条entry是committed的（已提交的）可以被安全地交给state machine去执行
- raft保证entry是持久存储的，并最终被集群中的所有机器都执行

##### 具体过程
- 客户端向leader发送command请求
- leader将command存入自己的log中
- leader向并发所有follower发送AppendEntries RPC请求，并等待接收响应结果
- 一旦leader接收到足够多（过半）的响应，即认为这条entry已经被committed，则认为可以安全地执行这条command了
1. leader一旦认为某条entry已committed，则将对应的command传给它的state machine执行，执行完成之后返回结果给client；
2. leader一旦认为某条entry已committed，则会通知其他所有follower，最终集群中所有机器都知道这条entry已经被提交了
3. 一旦followers知道这条entry已经被提交了，也会将对应的command传递给自己的state machine执行

#### leader changes
步骤：
1. 新leader上任后，各个server的log状态很可能是不一致的；因为旧leader可能只完成了部分server的log复制就挂掉了
2. raft中新leader上任后，并不会立即对不一致的旧log进行clean up，而仍然是正常开始normal operation
3. clean up是在normal operation的过程中进行的；原因在于：新leader上任后，可能有些server仍然是宕机状态，新leader没有办法立即对其进行clean up（因为那些server宕机或网络不通，无法进行通讯），只能等到这些server恢复正常后再进行clean up，而新leader不知道这些server什么时候能恢复正常，如果傻傻地苦苦等待，很可能会导致整个系统无法运行。所以，我们设计的系统，必须要确保新leader上任后要立即开始normal operation，而在normal operation的过程中要确保最终所有log一致
4. raft的做法是：总是认为leader的log是对的！
5. raft认为leader的log总是对的，总是包含所有重要的信息，因此leader的重任就是最终使得所有follower的日志与之完全相同。（皇帝总是对的，皇帝总是试图广播自己的想法，让其他所有人的想法与之最终一致）
6. 新leader在clean up的过程中也有可能宕机，再有新的leader上任，它也可能还没完成重任就又宕机了，长时间如此，最终会导致各个server的log混乱不堪

##### safety requirement原则
- 实现replicated logs的系统遵守的最基本的safety requirement原则
1. Once a log entry has been applied to a state machine, no other state machine must apply a different value for that log entry;即一旦某条log entry已经被某一个state machine执行，则其他任何state machine也必须执行这同一条log entry
2. 所有state machine都必须按照相同的顺序，执行完全相同的entry；（执行entry指的就是执行entry中的command）
- raft采取了更严苛的准则，即如下的safety property：
1. 如果某个leader发现某条log entry已经被提交，则这条entry必须存在于所有后续的leader中。这意味着，一旦某个leader即位，则在它的整个term时期内，它一定含有所有的已提交的log entries
- raft能够保证的safety requirement有：
1. leaders永远不会修改自己log中的entries，只能添加，而一旦下台，其entry是有可能被修改的。这条性质叫AppendOnlyProperty
2）只有leaders中的log才能被提交，某条entry即使过半server都存在了，但在leader中不存在，也是不能被提交的
3）entries在被提交给state machine执行之前，必须已被提交，即只有已被提交的entry才能被执行；

## raft如何在不同的阶段保障一致性
#### 1.数据到达 Leader 节点前
这个阶段 Leader 挂掉不影响一致性，不多说。
![image](https://github.com/ermaot/notes/blob/master/mysql/013%E5%88%86%E5%B8%83%E5%BC%8F/pic/RAFT%E7%AE%97%E6%B3%951.png)

#### 2.数据到达 Leader 节点，但未复制到 Follower 节点
- 这个阶段 Leader 挂掉，数据属于未提交状态，Client 不会收到 Ack 会认为超时失败可安全发起重试。
- Follower 节点上没有该数据，重新选主后 Client 重试重新提交可成功。
- 原来的 Leader 节点恢复后作为 Follower 加入集群重新从当前任期的新 Leader 处同步数据，强制保持和 Leader 数据一致
![image](https://github.com/ermaot/notes/blob/master/mysql/013%E5%88%86%E5%B8%83%E5%BC%8F/pic/RAFT%E7%AE%97%E6%B3%952.png)


#### 3.数据到达 Leader 节点，成功复制到 Follower 所有节点，但还未向 Leader 响应接收
- 这个阶段 Leader 挂掉，数据在 Follower 节点处于未提交状态（Uncommitted）但保持一致
- 重新选出 Leader 后可完成数据提交
- 此时 Client 由于不知到底提交成功没有，可重试提交
- 针对这种情况 Raft 要求 RPC 请求实现幂等性，也就是要实现内部去重机制
![image](https://github.com/ermaot/notes/blob/master/mysql/013%E5%88%86%E5%B8%83%E5%BC%8F/pic/RAFT%E7%AE%97%E6%B3%953.png)

##### 4.数据到达 Leader 节点，成功复制到 Follower 部分节点，但还未向 Leader 响应接收
- 这个阶段 Leader 挂掉，数据在 Follower 节点处于未提交状态（Uncommitted）且不一致
- ==Raft 协议要求投票只能投给拥有最新数据的节点==。所以拥有最新数据的节点会被选为 Leader 再强制同步数据到 Follower，数据不会丢失并最终一致
![image](https://github.com/ermaot/notes/blob/master/mysql/013%E5%88%86%E5%B8%83%E5%BC%8F/pic/RAFT%E7%AE%97%E6%B3%954.png)

##### 5.数据到达 Leader 节点，成功复制到 Follower 所有或多数节点，数据在 Leader 处于已提交状态，但在 Follower 处于未提交状态
这个阶段 Leader 挂掉，重新选出新 Leader 后的处理流程和阶段 3 一样。
![image](https://github.com/ermaot/notes/blob/master/mysql/013%E5%88%86%E5%B8%83%E5%BC%8F/pic/RAFT%E7%AE%97%E6%B3%955.png)

##### 6.数据到达 Leader 节点，成功复制到 Follower 所有或多数节点，数据在所有节点都处于已提交状态，但还未响应 Client
这个阶段 Leader 挂掉，Cluster 内部数据其实已经是一致的，Client 重复重试基于幂等策略对一致性无影响。
![image](https://github.com/ermaot/notes/blob/master/mysql/013%E5%88%86%E5%B8%83%E5%BC%8F/pic/RAFT%E7%AE%97%E6%B3%956.png)

##### 7.网络分区导致的脑裂情况，出现双 Leader
- 网络分区将原先的 Leader 节点和 Follower 节点分隔开，Follower 收不到 Leader 的心跳将发起选举产生新的 Leader。这时就产生了双 Leader
- 原先的 Leader 独自在一个区，向它提交数据不可能复制到多数节点所以永远提交不成功。
- 向新的 Leader 提交数据可以提交成功，网络恢复后旧的 Leader 发现集群中有更新任期（Term）的新 Leader 则自动降级为 Follower 并从新 Leader 处同步数据达成集群数据一致

##### 8？
如果集群中网络问题导致出现两个leader，一部分follower可以与leader1通信，并能提交，一部分与leader2通信，也可以提交，这种情况怎么办？