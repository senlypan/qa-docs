# 数据结构

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.03-data-structure&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-06-07

## 集合性能概览

Collection | Ordering | Random Access | Key-Value | Duplicate Elements | Null Element | Thread Safety 
----                 | ----  | ----  | ----  | ----  | ----  | ---- 
ArrayList            | ✔️ | ✔️ | ❌ | ✔️ | ✔️ | ❌ 
LinkedList           | ✔️ | ❌ | ❌ | ✔️ | ✔️ | ❌ 
HashSet              | ❌ | ❌ | ❌ | ❌ | ✔️ | ❌  
TreeSet              | ✔️ | ❌ | ❌ | ❌ | ❌ | ❌ 
HashMap              | ❌ | ✔️ | ✔️ | ❌ | ✔️ | ❌ 
TreeMap              | ✔️ | ✔️ | ✔️ | ❌ | ❌ | ❌ 
Vector               | ✔️ | ✔️ | ❌ | ✔️ | ✔️ | ✔️ 
HashTable            | ❌ | ✔️ | ✔️ | ❌ | ❌ | ✔️ 
Properties           | ❌ | ✔️ | ✔️ | ❌ | ❌ | ✔️ 
Stack                | ✔️ | ❌ | ❌ | ✔️ | ✔️ | ✔️ 
CopyOnWriteArrayList | ✔️ | ✔️ | ❌ | ✔️ | ✔️ | ✔️ 
ConcurrentHashMap    | ❌ | ✔️ | ✔️ | ❌ | ❌ | ✔️ 
CopyOnWriteArraySet  | ❌ | ❌ | ❌ | ❌ | ✔️ | ✔️ 

## 数据结构

### CopyOnWriteArrayList 应用场景

首先了解 CopyonwriteArraylist 的一个诞生背景，主要由于 ArrayList 线程不安全（并发场景下的读写冲突问题），而 Vector 虽线程安全锁粒度却又太粗（synchronized），所以在 JDK8 的 JUC 出了个 CopyonwriteArraylist，既能保证线程安全又减小了锁粒度，提高读写性能，唯二不足的就是 copy-on-write 的通病：**占用内存**（写时拷贝副本）、**数据最终一致性**（写时volatile导致读不及时）。

分享了一些实际场景：

- hase etcd里面的数据多版本
- 多副本之间的数据拷贝和最终一致
- 业务数据不同客户读取到的不同数据
- 协同编辑
- linux进程的内存共享
- 业务数据和数据分析的 星型模型

CopyOnWriteArrayList(免锁容器)的好处之一是当多个迭代器同时遍历和修改这个列表时，不会抛出 ConcurrentModificationException。在 CopyOnWriteArrayList 中，写入将导致创建整个底层数组的副本，而源数组将保留在原地，使得复制的数组在被修改时，读取操作可以安全地执行。

1. 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致 young gc 或者 full gc；
2. 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，读取到数据可能还是旧的,虽然 CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求。

CopyOnWriteArrayList 透露的思想：

1. 读写分离，读和写分开
2. 最终一致性
3. 使用另外开辟空间的思路，来解决并发冲突

## QA

### How to synchronize ArrayList in Java ?

- 1、Collections.synchronizedList() - method - returns synchronized list 在查询时需要显式使用同步块，例如 synchronized(list){ todo ...}.
- 2、copyOnWriteArrayList - class - Thread Safety variant of ArrayList.

