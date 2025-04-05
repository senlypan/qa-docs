# 服务治理

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.08-service-governance&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2025-04-05
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## 服务治理生命周期核心阶段



### A-服务设计

- 服务契约​​
    - API 定义（如 OpenAPI/Swagger）、接口兼容性设计。
- 服务拆分​​
    - 领域划分（DDD）、微服务边界设计。
- 治理策略预置​
    - 熔断规则、流量控制规则的初始配置。
​


### B-治理基础设施

> 提供服务的​​动态寻址​​能力，是流量路由（如负载均衡）的前提。

- 服务发现
    - 将服务元数据（IP、端口、健康状态）注册到服务发现中心（如 Nacos/Eureka）。
    - 例如：Nacos 同时支持服务注册发现和健康检查，是微服务的“通讯录”。

> 实现配置与代码分离，支持​​动态生效​​，是治理规则（如限流阈值）的存储底座。

- 配置中心
    - 通过配置中心（如 Nacos/Spring Cloud Config）加载运行时参数（如数据库连接、超时时间）。
    - 例如：通过 Nacos 修改超时时间，无需重启服务即可生效。



### C-流量治理策略

> 解决 ​​“流量如何分配”​​ 问题（如根据用户身份路由到不同服务版本）

- 负载均衡
    - Ribbon/LoadBalancer
- 路由
    - Spring Cloud Gateway/Istio VirtualService
​- 限流
    - Sentinel/Resilience4j
​- 熔断 
    - Hystrix/Sentinel



### ​D-​运行时容错机制

> 解决 ​​“出错时如何恢复”​​ 问题（如熔断后返回兜底数据）。

- 重试
    - Spring Retry
​- 降级
    - Hystrix/Sentinel
- 故障隔离
    - 舱壁模式



### E-服务观测

> 解决 ​​“系统是否健康”​​ 问题（如通过指标发现慢接口）。

- 链路追踪
    - Zipkin/SkyWalking
- 日志聚合
    - ELK/Grafana Loki
- 指标监控 
    - Prometheus/Micrometer



### ​​F-安全治理

- 认证鉴权
    - Spring Security, OAuth2, JWT
- 数据加密 
    - 证书, 对称加密, 非对称加密



### ​​G-自动化运维​

- 版本管理 
    - 灰度发布（如 Spring Cloud Gateway 路由权重）、蓝绿部署。
- 服务下线 
    - 优雅关闭（Spring Actuator /shutdown）、流量迁移。
- 弹性扩缩容
    - Kubernetes HPA, Chaos Mesh
​- 自愈
    - Kubernetes HPA, Chaos Mesh