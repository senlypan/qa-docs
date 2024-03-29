# Zookeeper分布式过程协同技术

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.07-zookeeper&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-09-27
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## 一、概述

### 1.1、基本定义

Zookeeper 从设计模式角度来理解：是一个基于 **观察者模式** 设计的分布式服务管理框架，它 **负责存储和管理大家都关心的数据** ，然后 **接受观察者的注册**，一旦这些数据的状态发生变化，Zookeeper 就将 **负责通知已经在 Zookeeper 上注册的那些观察者** 做出相应的反应。

又或者可以简单理解为： 

**Zookeeper = 文件系统 + 通知机制**

### 1.2、特点

- **强一致**（CP）
   - 全局数据一致，每个节点保存一份相同副本
- **有序性**
   - 更新请求，顺序进行，来自同一个客户端的更新请求，按其发送顺序依次执行
- **实时性**
   - 在一定时间范围内，客户端能读到最新的数据
- **主从集群**（Leader、follower observer）
   - 集群中只要有半数以上节点存活，Zookeeper 集群就能正常服务

## 二、优势

`待补充`


## 三、应用

- [zookeeper实现分布式锁的两种方式](https://cloud.tencent.com/developer/article/2039944)
