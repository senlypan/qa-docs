# Spring

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.05-spring&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-04-18
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## 文档 

- 👉 [潘深练 - 《一文读懂Spring本质》](http://spring.panshenlian.com/#/zh-cn/02-spring-core) 

## QA

### 1、多数据源与分布式事务的关系思考？

- 首先明确，多数据源切换 与 分布式事务 是两件事。
    - 1、 确实在spring框架下，可以通过 AbstractRoutingDataSource + threadlocal 去做数据源切换，其中核心通过 threadLocal 绑定线程上下文，通过AbstractRoutingDataSource查找对应数据源。
    - 2、 而如果涉及到多个数据源之间要保持数据一致性，或与三方接口要保持数据一致性，就涉及分布式事务，而分布式事务，自然思路是记录 undo 和 redo ，本质是模拟了 mysql 处理单机事务时对日志的运用。通过 undo 做撤销/回滚操作，通过 redo 做重做/恢复操作。
    - 3、需要明确边界，事务都是数据库来保证，而 spring 框架保证的是事务传播，事务传播是 spring 框架的概念，负责维护线程与事务的关系，简而言之，spring支持在线程间或方法嵌套时，使用者是否要共用一个事务，或开启新事务。spring 框架只不过通过包装数据库事务使用开发者用起来更方便。 spring 在你声明事务的数据库操作代码里，前插 start transaction 后插 commit，以保障这些语句在一个session里执行（从数据库层面理解起来也是一个connection），如果不声明事务 实际上spring整合orm框架后 一个数据库操作的方法就是一个session。切数据源后需要多个数据源间操作的数据库语句前后加 start 和 commit，但这会造成另一个问题，两个事务之间怎么保证原子性？ 从这里开始演化成了分布式事务的场景。
    - 推荐阅读
        - [Spring是如何支持多数据源?](https://mp.weixin.qq.com/s/uR4FyEJKiU_75eGb4ynKiQ)
        - [dynamic-datasource-spring-boot-starter](https://mp.weixin.qq.com/s/uR4FyEJKiU_75eGb4ynKiQ) 