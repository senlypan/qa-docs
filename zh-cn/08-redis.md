# Redis

![访问统计](https://visitor-badge.glitch.me/badge?page_id=08-redis&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-04-21

## 文档 

[Redis入门小册](http://redis.panshenlian.com/)

## QA

### 1、Redis为什么要持久化？
- Redis是内存数据库，宕机后数据会消失。
- Redis重启后快速恢复数据，要提供持久化机制
- Redis持久化是为了快速的恢复数据 **而不是为了存储数据**
- Redis有两种持久化方式：AOF（append only file），RDB（Redis DataBase）
- 注意：Redis持久化不保证数据的完整性。
- 当Redis用作DB时，DB数据要完整，所以一定要有一个完整的数据源（文件、mysql）
- 在系统启动时，从这个完整的数据源中将数据load到Redis中
- 数据量较小，不易改变，比如：字典库（xml、Table）
- 通过info命令可以查看关于持久化的信息
```conf
# Persistence
loading:0
rdb_changes_since_last_save:1
rdb_bgsave_in_progress:0
rdb_last_save_time:1589363051
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:1
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0
aof_current_size:58
aof_base_size:0
aof_pending_rewrite:0
aof_buffer_length:0
aof_rewrite_buffer_length:0
aof_pending_bio_fsync:0
aof_delayed_fsync:0
```

### 2、Redis在不同应用场景下如何选择持久化方案？

- 场景一：作为内存数据库
    - 开启 rdb+aof >>> 数据不容易丢，但性能低（RDB子进程fork持久化数据时间过长，主线程堵塞等问题）
    - 当然如果有原始数据源（mysql），每次启动时都从原始数据源（mysql）中初始化 ，则不用开启持久化 （数据量较小）
- 场景二：作为缓存服务器
    - 开启 rdb >>> 数据安全性一般，容易丢数据，但性能高
    - 开启 aof >>> 数据安全性高，丢数据概率低，每秒或每次命令保存一次
- 场景三：追求极致性能
    - 不开启持久化方案 >>> 必须有原始数据源（例如mysql），每次重启或故障后可以恢复数据