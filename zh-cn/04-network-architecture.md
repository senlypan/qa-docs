# 网络体系

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.03-network-architecture&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-03-26
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## The C10K problem

-  [The C10K problem](http://www.kegel.com/c10k.html).

## 协议层细化

- 传输层：QUIC/HTTP3.0
- 应用层：gRPC/WebSocket
- 安全协议：TLS 1.3/WireGuard

## 网络架构设计

- 软件定义网络（SDN）
- 服务网格数据平面（Envoy）
- 负载均衡算法（P2C/EWMA）