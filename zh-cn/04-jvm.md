# JVM虚拟机

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.04-jvm&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-04-18

## 文档 

[走近JVM](http://jvm.panshenlian.com/#/zh-cn/02-jvm)

## QA

### JVM 日志错误一般核查办法？

- 直接获取答案：百度、谷歌、咨询系统原有维护人员
- 观察监控异常：从机器维度，jvm 维度，接口流量、程序日志维度。对比近期出现的波动
- 对比正常应用机器：拿异常机器和正常机器做对比，差异点就有可能导致问题的原因
- 分析最近的出现的各类变更，包括代码、配置、上线记录、人工操作等

 