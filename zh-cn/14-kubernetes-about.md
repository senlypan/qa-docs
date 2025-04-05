# Kubernetes 专题 

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.14-kubernetes-about&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-08-31
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## 一、Kubernetes 基础篇

`待补充`

## 二、Kubernetes 网络篇

### 1、veth设备

- 本机和 docker 容器内网络通信的基础

详细见 [《轻松理解 Docker 网络虚拟化基础之 veth 设备！》](https://mp.weixin.qq.com/s/sSQFINJ8RO8Nc4XtcyQIjQ)

### 2、bridge

- 本机的两个 docker 之间怎么通信

详细见 [《聊聊 Linux 上软件实现的“交换机” - Bridge！》](https://mp.weixin.qq.com/s/JnKz1fUgZmGdvfxOm2ehZg)

### 3、网络命名空间

- docker 之间的网络隔离

详细见 [《动手实验+源码分析，彻底弄懂 Linux 网络命名空间》](https://mp.weixin.qq.com/s/lscMpc5BWAEzjgYw6H0wBw)

### 4、手工模拟 docker 容器网络

- 实践内容

详细见 [《手工模拟实现 Docker 容器网络！》](https://mp.weixin.qq.com/s/Arcz3RWe_o0Ijw6uPWKdVw)

### 5、iptables

- 用来识别哪些流量应该转发到本机的容器，哪些应该转发到其他机器，为 k8s 网络打个基础，因为 k8s 是集群管理

详细见 [《来，今天飞哥带你理解 iptables 原理！》](https://mp.weixin.qq.com/s/O084fYzUFk7jAzJ2DDeADg)

### 6、k8s 网络原理

- 介绍整体k8s网络的架构

详细见 [《深入理解K8S网络原理上》](https://mp.weixin.qq.com/s/8muM-UAl7KyFAIsdfTebyQ)、[《深入理解K8S网络原理下》](https://mp.weixin.qq.com/s/-DcJ7N_gqhaPWsIT1NMKIw)
 