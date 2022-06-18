# 分布式锁

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.08-distributed-locking&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-05-11

## 分布式锁的本质

Martin Kleppmann 分析了 Redis 的 RedLock 算法写了一篇文章 [《How to do distributed locking》](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html) ，详细说明了 Redis 分布式锁基于时间（过期）的设计，往往会因为 STW 或机器时钟不稳定等问题 hang 住从而致使锁超期释放。

于是乎推荐用 zk 这类基于目录顺序和临时标签自增去搞，不像 redis 那样依赖过期时间，而是基于心跳去搞，但是依然会由于分布式的根本问题，那就是网络不可靠，同样会导致锁超时释放，没能达到分布式锁的目的。

而 redission 看门狗不断续期，其实也就是在保持心跳，但也还是不能从根本上解决假死问题，只要发生假死，心跳都会发不出去，即使是 zk 也会误认为机器已经挂了。因此根本的问题核心出在分布式的基本问题上：**网络不可靠**。

其中 zk 之所以会被认为比 redis 可靠，其实是因为它的持久化机制，假如 redis 配上共识算法，配上集群，配上 aof，也能达到 ZK 同样的效果。

