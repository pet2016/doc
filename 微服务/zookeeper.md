
#### zab协议

zab 协议是为了zookeeper专门设计的一种支持崩溃恢复的原子广播协议

zab协议包含两种模式： 崩溃恢复和原子广播

- 原子广播

当集群有过半的follower完成了与leader的状态同步，那么整个集群进入消息广播模式
新加入服务器，服务器与leader通信同步数据，数据同步完成后参与到消息广播流程


消息广播流程：

1. 客户端发起请求
2. leader讲请求转成事务proposal，并为每个proposal分配一个事务ID(Zxid);
3. leader为每个follower单独分配一个fifo的队列，将proposal放入到队列
4. follower接收到proposal后，首先以事务日志的方式写入本地磁盘，写入成功后给leader反馈一个ack响应
5. leader接收到超过半数的follower反馈后，认为消息发送成功，发送commit消息
6. leader向所有的follower发送commit消息，同事自首也完成事务提交
7. follower接收到commit消息后也完成事务提交。

- 崩溃恢复

崩溃恢复通过选举实现，leader选举有两个场景
1. 集群启动时进行leader选举
2. leader崩溃后进行选举

重要的参数
1. myid：服务器ID,安装时配置，myid越大，被选举为leader的优先级越高；raft通过timeout实现
2. zxid：事务id，全局唯一并自增，zxid越大，标识事务是最新的
3. epoch：投票轮次，每投票一次，epoch增加1

另外，选举过程中节点的状态：
1. looking：竞选状态
2. following：随从状态，同步leader状态，参与leader选举的投票过程
3. observing：观察状态，同步leader状态，不参与leader选举投票过程
4. leading：领导者状态

选举流程：
1. 每个几点会发出一个投票，第一次都是投自己，投票信息（myid，zxid）
2. 收集各个服务器的投票
3. 处理投票并重新投票，处理逻辑：优先比较zxid，然后再比较myid
4. 统计投票，只要超过半数的服务器收到同样的投票信息，就可以确定leader
5. 改变服务器状态，进入正常的消息广播流程

#### 数据一致性

1. 选举拥有zxid最大值的节点作为新leader。因为proposal被提交前都必须得到超半数的follower ACK，只要合法follower节点正常，及必然会保存所有被commit的proposal；
2. 新的 leader将自己事务日志中proposal但未commit的消息处理
3. 新leader与follower建立新的FIFO队列，讲自己有其他follower没有的proposal发送给对方，再发送commit消息给对方，以保证follower保存所有的follower

zxip = epoch（32）+ counter（32） 64位数

选出新leader后，epoch 增加1， counter 变成 0。 counter是消息计算器

#### ZAB与Paxos的联系和区别
- 联系
1. 都存在一个类似Leader进程的角色，由其负责协调多个Follower进程的运行
2. Leader进程都会等待超过半数的Follower作出正确的反馈后，才会将一个提议进行提交（过半原则）
3. 在ZAB中，每个Proposal中都包含了一个epoch值，用来代表当前Leader周期，在Paxos中同样存在这样的一个表示，名字为 Ballot。
- 区别
1. Paxos算法中，新选举产生的主进程会进行两个阶段的工作；第一阶段称为读阶段：新的主进程和其他进程通信来收集主进程提出的提议，并将它们提交。第二阶段称为写阶段：当前主进程开始提出自己的提议。
2. ZAB协议在Paxos基础上添加了同步阶段，此时，新的Leader会确保存在过半的Follower已经提交了之前Leader周期中的所有事物Proposal。这一同步阶段的引入，能够有效保证，Leader在新的周期中提出事务Proposal之前，所有的进程都已经完成了对之前所有事务Proposal的提交。
总的来说，ZAB协议和Paxos算法的本质区别在于两者的设计目的不一样：ZAB协议主要用于构建一个高可用的分布式数据主备系统，而Paxos算法则用于构建一个分布式的一致性状态机系统。