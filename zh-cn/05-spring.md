# Spring

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.05-spring&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-04-18

## 文档 

[一文读懂Spring本质](http://spring.panshenlian.com/#/zh-cn/02-spring-core)
 

## QA

### 1、多数据源与分布式事务的关系思考？

- 首先明确，多数据源切换 与 分布式事务 是两件事。
    - 1、 确实在spring框架下，可以通过 AbstractRoutingDataSource + threadlocal 去做数据源切换，其中核心通过 threadLocal 绑定线程上下文，通过AbstractRoutingDataSource查找对应数据源。
    - 2、 而如果涉及到多个数据源之间要保持数据一致性，或与三方接口要保持数据一致性，就涉及分布式事务，而分布式事务，自然思路是记录 undo 和 redo ，本质是模拟了 mysql 处理单机事务时对日志的运用。通过 undo 做撤销/回滚操作，通过 redo 做重做/恢复操作。
