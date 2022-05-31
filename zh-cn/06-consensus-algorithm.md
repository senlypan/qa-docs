# 共识算法

> 作者: 潘深练
>
> 创建: 2022-05-12

## 一、什么是一致性

![#001](../_media/images/06-consensus-algorithm/001-CAP.jpg)

### CAP

对于一个分布式系统，不能同时满足一下三点：

- 一致性（Consistency）
- 可用性（Availability）
- 分区容错性（Partition Tolerance）

![#002](../_media/images/06-consensus-algorithm/002-consensus.jpg)

### 一致性模型

#### 弱一致性

- 最终一致性
    - DNS (Domain Name System)
    - Gossip (Cassandra 的通信协议)

#### 强一致性

- 同步
- Paxos
- Raft (multi-paxos)
- ZAB (multi-paxos)

### 明确问题

- **数据不能存在单点上**。分布式系统对容错（fault tolorence）的一般解决方案是状态机复制（state machine replication），所以主要针对状态机复制（state machine replication）讨论应用之上的共识（consensus）算法。**需要注意的是**， paxos 其实是一个共识算法，系统的最终一致性，不仅需要达成共识，**还会取决于 client 的行为**。


## 二、 延伸

### Quorum机制

### Lease机制

### NWR协议





## 三、概念

### 主观下线

### 哨兵的选举




## 四、实践

### Zookeeper

### etcd






## 参考阅读

1、[The Part-Time Parliament](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf) Lamport in 1990.
2、[Paxos Made Simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf) Lamport in 2001.
3、[Lamport about paxos](http://lamport.azurewebsites.net/pubs/pubs.html#lamport-paxos)
4、[Paxos Made Moderately Complex](https://paxos.systems/)
5、[In Search of an Understandable Consensus Algorithm](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14)
6、[Paxos intro in《凤凰架构》](http://icyfenix.cn/distribution/consensus/paxos.html)
7、[Implementing Replicated Logs with Paxos](https://ongardie.net/static/raft/userstudy/paxos.pdf)
8、[Zookeeper：分布式系统入门到实战](https://www.youtube.com/watch?v=BhosKsE8up8)（Youtube）
