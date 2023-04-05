# 开发规范

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.22-development-standard&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-09-23
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## 一、手册篇

- [Java开发手册——嵩山版](https://developer.aliyun.com/topic/java20)
    - 手册涵盖编程规约、异常日志、单元测试、安全规约、MySQL数据库、工程规约、设计规约七大维度。


## 二、工具篇

- [Java 规约插件](https://github.com/alibaba/p3c)
    - 为了让开发者更加方便、快速的将规范推动并实行起来，阿里巴巴基于手册内容，研发了一套自动化的 IDE 检测插件（IDEA、Eclipse）， 该插件在扫描代码后，将不符合《手册》的代码按 Blocker/Critical/Major 三个等级显示在下方，甚至在 IDEA上，还基于 Inspection 机制提供了实时检测功能，编写代码的同时也能快速发现问题所在。