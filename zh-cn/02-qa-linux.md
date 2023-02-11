# Linux系统QA

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.02-qa-linux&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-03-26

## 一、常用工具

### 1、Linux编程手册
-  Linux Programmer's Manual
    -  use Linux Manual ,like '`man 2 socket`'.

### 2、Linux常用命令
- **2.1、线上问题定位**
    - **TOP**
        - 查看每个进程的情况，包括 CPU 利用率等
        - 在 Java 进程这一行里如果看到 CPU 利用率为 200% ，不用担心，这是当前机器所有核加在一起的 CPU 利用率
        - 常见 3 类情况：
            - 1、某个线程 CPU 利用率一直是 100% ，则说明是这个线程有可能存在 **死循环**，那么请记住这个 PID
            - 2、某个线程一直在 TOP 10 的位置，这说明这个线程有可能存在 **性能问题**
            - 3、CPU 利用率高的几个线程在不断变化，说明并不是由某一个线程导致 CPU 偏高
    - **jstat**
        - 在使用 TOP 命令查看进程情况时，如遇 CPU 利用率一直是 100%，除了存在死循环的可能性，也有可能是 **GC 造成**，可以使用 jstat 命令查看 GC 情况，确认是否 **持久代（P）** 或 **老年代（O）** 满了，产生 Full GC，导致 CPU 利用率持续飙高。
        - 用法：jstat -gcutil 9999 1000 5 
        - 说明：jstat -gcutil 端口 毫秒 回显次数
    - **jstack**
        - 除了使用 jstat 查看进程 GC 情况，还可以使用 jstack 命令把线程 dump 下来，看看究竟是哪个线程、执行什么代码造成的 CPU 利用率高
        - 例如，执行 jstack 把线程 dump 到文件 dump01 里头。
        - 命令：jstack 9999 > /mnt/local/data/panshenlian/java/dumpfile/dump01
        - 注意，dump 出来的线程 ID（nid）是十六进制的，而我们用 TOP 命令看到的线程 ID 是十进制的，所以要用 printf 命令转换一下进制。然后用十六进制的 ID 去 dump 里头找到对应的线程
        - 例如：`printf "%x\n" 31558` ，输出结果： `7b46` 
- **2.2、性能测试**
    - **netstat**
        - 使用 netstat 命令查询有多少台 **机器连接** 到这个端口上
        - netstat -nat | grep 9999 -c ， 输出结果如：`10`
        - -c 计算总数，可不加 -c 直接查看明细
    - **ps**
        - 使用 ps 命令查看 **线程数**
        - ps -eLf | grep java -c ，输出结果如：`97`
        - -c 计算总数，可不加 -c 直接查看明细
        - -e 代表列出所有进程
        - -l 代表长格式
        --f 代表完整的格式
    - **查看网络流量**
        - cat /proc/net/dev
    - **查看系统平均负载**
        - cat /proc/loadavg
    - **查看系统内存情况**
        - cat /proc/meminfo
    - **查看 CPU 的利用率**
        - cat /proc/stat
- **2.3、更多命令参考**
    - **CPU/Mem 资源使用状况**
        - 查看使用 `tsar --mem/cpu/io/net -n 1 -i 1`、`top` 等
    - **网络状况**
        - 使用 `lsof`、`netstat` 等
    - **磁盘使用状况**
        - 使用 `iostat`、`block_dump`、`inotifywait`、`df/du` 等
        - 例如 `df -h` 查看硬盘空间情况
        - 例如 `du -sh *` 查看硬盘空间具体占用情况
    - **内核日志**
        - 使用 `/var/log/messages`、`sudo dmesg` 等
    - **性能工具**
        - 使用 `strace`、`perf` 等
    - **IO压测**
        - 使用 `FIO` 等
