# 论文《实用拜占庭容错（Practical Byzantine Fault Tolerance）》

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.07-papper-of-practical-byzantine-fault-tolerance&left_color=blue&right_color=red)

> 译者: 潘深练
>
> 创建: 2022-06-19
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


**论文《实用拜占庭容错（Practical Byzantine Fault Tolerance）》** 于 1999 年 2 月发表在第三届操作系统设计与实现研讨会论文集，发表地美国新奥尔良。

论文发表者是米格尔 · 卡斯特罗和芭芭拉 · 丽斯科夫，来自马赛诸塞州（邮编：02139）剑桥科技广场 545 号麻省理工学院计算机科学实验室，联系邮箱为 {castro,liskov}@lcs.mit.edu

译者注：芭芭拉 · 丽斯科夫（Barbara Liskov），1939年生，美国国家工程院院士、美国艺术与科学院院士和计算机机械协会 (ACM) 院士，2008年图灵奖得主。米格尔 · 卡斯特罗（Miguel Castro），是芭芭拉 · 丽斯科夫的学生。

## 摘要（Abstract）

本文提出了一种新的复制算法，该算法能够容忍拜占庭故障。我们认为，拜占庭容错算法在未来将变得越来越重要，因为恶意攻击和软件错误在不断增加，并可能导致有问题的节点表现出任意的行为。以前的算法是假设同步系统或太慢而无法在实践中使用，而本文介绍的算法是实用的：它可以在像互联网这样的异步环境中工作，并且包含了几个重要的优化，将以前算法的响应时间提高了一个数量级以上。我们使用该算法实现了一个拜占庭容错的 NFS 服务，并测量了其性能。结果表明我们的服务仅仅比标准的无复制的 NFS 慢 3%。

## 1 概要介绍（Introduction）

## 2 系统模型（System Model）

## 3 服务属性（Service Properties）

## 4 算法（The Algorithm）

### 4.1 客户端（The Client）

### 4.2 PBFT算法主线程（正常情况）（Normal-Case Operation）

### 4.3 垃圾回收（Garbage Collection）

### 4.4 视图变化（View Changes）

### 4.5 正确性（Correctness）

#### 4.5.1 安全性（Safety）

#### 4.5.2 活性（Liveness）

### 4.6 不确定性（Non-Determinism）

## 5 优化（Optimizations）

### 5.1 减少沟通（Reducing Communication）

### 5.2 密码学（Cryptography）

## 6 实现（Implementation）

### 6.1 副本库（The Replication Library）

### 6.2 拜占庭容错文件系统（BFS: A Byzantine-Fault-tolerant File System）

### 6.3 维护检查点（Maintaining Checkpoints）

### 6.4 计算检查点摘要（Computing Checkpoint Digests）

## 7 性能评估（Performance Evaluation）

### 7.1 实验设置（Experimental Setup）

### 7.2 微观基准（Micro-Benchmark）

### 7.3 安德鲁基准（Andrew Benchmark）

## 8 相关工作（Related Work）

## 9 结论（Conclusions）

### 致谢（Acknowledgments）

## 文献参考

- [mit.edu - Practical Byzantine Fault Tolerance](https://pmg.csail.mit.edu/papers/osdi99.pdf)

- [百度文库 - 拜占庭共识算法PBFT](https://wenku.baidu.com/view/5304eb6428160b4e767f5acfa1c7aa00b52a9dd4.html)


