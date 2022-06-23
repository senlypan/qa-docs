# 并发编程

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.04-concurrent-programming&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-04-18

## 并发编程基础 

- [Java并发编程聚合文档](http://concurrent-programming.panshenlian.com/#/zh-cn/00-Java-Concurrency-and-Multithreading-Tutorial)

- [ThreadLocal夺命11连问](https://mp.weixin.qq.com/s/xssF-ckUsXI7tY74zix-GQ)

## QA

### 1、应用容器的线程池，不想自己主动 remove 维护，容器会自动回收吗

- 首先回答：不会。由于：
    - 业务层面把线程变量存储到 Threadlocal 中 （实质是在 ThreadLocalMap），ThreadLocal 只是作为 key ，并且 ThreadLocalMap 中的 key 为 Threadlocal 的弱引用，当一个对象只存在弱引用时，key 会在下一次 GC 的时候被清除掉。
    - ![](../_media/images/04-concurrent-programming/04-concurrent-programming-001.png)
    - 但是，ThreadLocalMap 对应的 value 是强引用，不会被 gc 清理，这样一来就会导致内存泄露。
    - 所以需要在代码中主动进行 remove 清除，避免：
        - A. 由于ThreadLocalMap.Entry的Value没及时remove，导致的内存堆积、甚至泄露；
        - B. 由于线程池场景下线程复用，导致不同客户端情况出现数据脏读；
    - 参考文章：[《CSDN - ThreadLocal:内存泄漏问题及Java的对应处理办法》](https://blog.csdn.net/cold___play/article/details/105936714)
 