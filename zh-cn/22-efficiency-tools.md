# 效率工具与工作方法

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.22-efficiency-tools&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-11-02

## 一、效率工具

### 1.1、开发机

- 窗口操作使用 `Screen`、`tmux` 保持链接不中断

- SSH远端包括 `iTerm2`、`Bash`、`Zsh`、`Fish`

- 编辑器包括 `VSCode`、`Vim` 

- 调查问题 Debug 使用 `gdb`、`pdb`

- 内存泄漏：`tcmalloc`

- 代码扫描 `静态 cppcheck` 和 `动态代码扫描 asan`


### 1.2、Linux

- Cpu/Mem 资源查看使用 `tsar --mem/cpu/io/net -n 1 -i 1`、`top` 等

- 网络使用 `lsof`、`netstat`

- 磁盘使用 `iostat`、`block_dump`、`inotifywait`、`df/du`

- 内核日志包括 `/var/log/messages`、`sudo dmesg`

- 性能工具 `strace`、`perf`

- IO压测 `FIO`


### 1.3、文档类

- 语雀的在线 UML 图/流程图/里程碑方便多人共同编辑等

- Teambition 的项目管理甘特图

- Aone 的需求管理和缺陷管理

- 离线工具诸如 processOn / Xmind思维导图 / draw.io 流程图 / OneNote




## 二、工作方法

 
### 2.1、SMART 原则

- S：Specific，具体的

- M：Measurable，可以衡量的

- A：Attainable，可以达到的
 
- R：Relevant，具有一定的相关性

- T：Time-bound，有明确的截止期限

### 2.2、论文学习方法

推荐先阅读 **大数据** 经典系列（例如Google 新/老三驾马车），对于 **存储** 领域同学，推荐 Fast 论文

- Motivation: 解决了一个什么样的问题？为什么要做这个问题？

- Trade-off: 优势和劣势是什么？带来了哪些挑战？

- 适用场景: 没有任何技术是普适的，业务场景，技术场景

- 系统实现: 组成部分和关键实现，核心思想和核心组件，灵魂在哪里？

- 底层原理: 其底层的关键基础技术，基于这个基础还有哪些工作？

- Related Works: 这个问题上还有什么其他的工作？相关系统对比？不同的实现、不同的侧重、不同的思路？

### 2.3、TDD

**Test-diven Development** 测试驱动开发，阿里 **石超** 推荐，“自从看了TDD这本书，我就爱上了写UT”，当时听完这句话驱动了我的好奇心，TDD到底一个什么神奇的方法？

后来发现在《软件测试》《Google ：Building Secure & Reliable Systems》《重构》 《重构与模式》《敏捷软件开发》《程序员的职业素养》……国外泰斗级程序员大叔的书里，全部都推荐了 TDD。

TDD 不是万能药，主要思维模式是，**先想清楚系统的行为表现，再下手编码，测试想清楚了，开发的 API /系统表现就清晰了，API/函数/方法语义就明确了**。

