# Linux系统QA

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.02-qa-linux&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-03-26
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


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

### 3、Linux常用任务

- 定时清理超过3天的日志文件

```shell
#!/bin/bash

anynowtime="date +'%Y-%m-%d %H:%M:%S'"
NOW="echo [\`$anynowtime\`][PID:$$]"

##### 可在脚本开始运行时调用，打印当时的时间戳及PID。
function job_start
{
    echo "`eval $NOW` job_start"
    echo "`eval $NOW` 查找超过3天的日志文件并清理...start"
    find /mnt/local/micro/*/logs/* -mtime +3 -name "*" -exec rm -rf {} \;
}

##### 可在脚本执行成功的逻辑分支处调用，打印当时的时间戳及PID。 
function job_success
{
    MSG="$*"
    echo "`eval $NOW` job_success:[$MSG]"
    echo "`eval $NOW` 查找超过3天的日志文件并清理...success"
    exit 0
}

##### 可在脚本执行失败的逻辑分支处调用，打印当时的时间戳及PID。
function job_fail
{
    MSG="$*"
    echo "`eval $NOW` job_fail:[$MSG]"
    echo "`eval $NOW` 查找超过3天的日志文件并清理...fail"
    exit 1
}

job_start
job_success

###### 作业平台中执行脚本成功和失败的标准只取决于脚本最后一条执行语句的返回值
###### 如果返回值为0，则认为此脚本执行成功，如果非0，则认为脚本执行失败
###### 可在此处开始编写您的脚本逻辑代码
```

- Nginx 下负载

```shell
#!/bin/bash

anynowtime="date +'%Y-%m-%d %H:%M:%S'"
NOW="echo [\`$anynowtime\`][PID:$$]"

##### 可在脚本开始运行时调用，打印当时的时间戳及PID。
function job_start
{
    echo "`eval $NOW` job_start"
    
    # 查找关键字并下线(注释掉)节点
    sed -i /mallmanage2/s@^\ @#@g /usr/local/nginx/conf/nginx.conf
    
    # 重新加载Nginx
    /usr/local/nginx/sbin/nginx -s reload
    
    sleep 10
}

##### 可在脚本执行成功的逻辑分支处调用，打印当时的时间戳及PID。 
function job_success
{
    MSG="$*"
    echo "`eval $NOW` job_success:[$MSG]"
    exit 0
}

##### 可在脚本执行失败的逻辑分支处调用，打印当时的时间戳及PID。
function job_fail
{
    MSG="$*"
    echo "`eval $NOW` job_fail:[$MSG]"
    exit 1
}

job_start

###### 作业平台中执行脚本成功和失败的标准只取决于脚本最后一条执行语句的返回值
###### 如果返回值为0，则认为此脚本执行成功，如果非0，则认为脚本执行失败
###### 可在此处开始编写您的脚本逻辑代码

```

- 停机/停服备份

```shell

#!/bin/bash

anynowtime="date +'%Y-%m-%d %H:%M:%S'"
NOW="echo [\`$anynowtime\`][PID:$$]"

##### 可在脚本开始运行时调用，打印当时的时间戳及PID。
function job_start
{
    echo "`eval $NOW` job_start"
    
    tomcat_home="/mnt/local/apache-tomcat-8.5.31"
    # 去除末尾斜杠/
    tomcat_home=${tomcat_home%*/}
    echo "Tomcat home: ${tomcat_home}"
    
    time=`date +%y-%m-%d`
    ts=`date +%Y-%m-%d_%H%M%S`
    
    # 创建程序代码备份目录
    if [ ! -d "${tomcat_home}/bak" ]; then
      mkdir -p ${tomcat_home}/bak
    fi
    
    if [ ! -d "${tomcat_home}/bak/$time" ]; then
      mkdir -p ${tomcat_home}/bak/$time
    fi
    
    # 备份程序代码
    find ${tomcat_home}/bak -mtime +0 -name "*" -exec rm {} -rf \;
    if [ ! -d "${tomcat_home}/bak/$time/webapps/" ]; then
      echo "Backup ${tomcat_home}/webapps to ${tomcat_home}/bak/$time/"
      cp -rf ${tomcat_home}/webapps ${tomcat_home}/bak/$time/
    fi
    
    # 关闭服务
    ps -ef | grep ${tomcat_home} | grep -v grep | cut -c 9-15 | xargs kill -s 9
    
    # 备份日志
    mkdir -p ${tomcat_home}/logs/${ts}
    mv  ${tomcat_home}/logs/*.* ${tomcat_home}/logs/${ts}
    
    
    # 清空部署目录
    rm -rf ${tomcat_home}/temp
    mkdir ${tomcat_home}/temp
    
    rm -rf ${tomcat_home}/webapps/*
    ls -l ${tomcat_home}/webapps/
    
    # 清理 heap dump 文件目录
    rm -rf /var/log/java/*
}

##### 可在脚本执行成功的逻辑分支处调用，打印当时的时间戳及PID。 
function job_success
{
    MSG="$*"
    echo "`eval $NOW` job_success:[$MSG]"
    exit 0
}

##### 可在脚本执行失败的逻辑分支处调用，打印当时的时间戳及PID。
function job_fail
{
    MSG="$*"
    echo "`eval $NOW` job_fail:[$MSG]"
    exit 1
}

job_start

###### 作业平台中执行脚本成功和失败的标准只取决于脚本最后一条执行语句的返回值
###### 如果返回值为0，则认为此脚本执行成功，如果非0，则认为脚本执行失败
###### 可在此处开始编写您的脚本逻辑代码

```

- 启动服务(mall)

```shell
#!/bin/bash

anynowtime="date +'%Y-%m-%d %H:%M:%S'"
NOW="echo [\`$anynowtime\`][PID:$$]"

##### 可在脚本开始运行时调用，打印当时的时间戳及PID。
function job_start
{
    echo "`eval $NOW` job_start"
    
    port="8081"                                              
    ip=`/sbin/ifconfig|sed -n '/inet addr/s/^[^:]*:\([0-9.]\{7,15\}\) .*/\1/p' | head -1`
    
    tomcat_home="/mnt/local/apache-tomcat-8.5.31"
    tomcat_home=${tomcat_home%*/} # 去除末尾斜杠/
    
    log "Tomcat home: ${tomcat_home}"
    log "IP: ${ip}"
    
    time=`date +%y-%m-%d`
    ts=`date +%Y-%m-%d_%H%M%S`


    #如果没有war包回退上个版本，重启退出
    if [ ! -f "${tomcat_home}/webapps/mall.war" ]
    then
        log "没有war包回退上个版本，重启退..."
        ps -ef | grep ${tomcat_home} | grep -v grep | cut -c 9-15 | xargs kill -s 9
        rm -rf ${tomcat_home}/webapps/*
        cp -rf ${tomcat_home}/bak/$time/webapps/mall.war ${tomcat_home}/webapps/
        ${tomcat_home}/bin/startup.sh
        log "no mall.war"
        sleep 150
        exit 1
    fi

    log "启动Tomcat..."
    ${tomcat_home}/bin/startup.sh
    ls -l ${tomcat_home}/webapps/
    ps -ef | grep /mnt/local/mall/
    log "150秒后将进行健康检查..."
    sleep 150
    
    
    #健康检查
    int=1                                                                                 
    while (( $int<=20 ))                                                                                                                
    do
        int=$(($int+1))
        str=`curl "http://$ip:$port/mall/health.check" 2> /dev/null`
        sleep 5
        if [[ $str=="OK" ]];then  
                log "健康检查通过"
                sleep 5
                break
        fi
    done
    
    #如果健康检查超过100秒还没有通过，回退版本并退出
    str=`curl "http://$ip:$port/mall/health.check" 2> /dev/null`
    if [[ $str=="OK" ]];then 
        log "健康检查通过，发版成功"                                                                                    
        sleep 5
        exit 0
    else
        log "健康检查未通过"
        
        if [ -f "${tomcat_home}/bak/$time/webapps/mall.war" ]; then
            ps -ef | grep ${tomcat_home} | grep -v grep | cut -c 9-15 | xargs kill -s 9
            rm -rf ${tomcat_home}/webapps/*
            
            log "恢复备份的war包[${tomcat_home}/bak/$time/webapps/mall.war]..."
            cp -rf ${tomcat_home}/bak/$time/webapps/mall.war ${tomcat_home}/webapps/
            log "重启备份的服务,流程结束"
            ${tomcat_home}/bin/startup.sh
            sleep 150
            exit 1
        fi
    fi
}

##### 可在脚本执行成功的逻辑分支处调用，打印当时的时间戳及PID。 
function job_success
{
    MSG="$*"
    echo "`eval $NOW` job_success:[$MSG]"
    exit 0
}

##### 可在脚本执行失败的逻辑分支处调用，打印当时的时间戳及PID。
function job_fail
{
    MSG="$*"
    echo "`eval $NOW` job_fail:[$MSG]"
    exit 1
}

function log
{
    echo "`eval $NOW`:$*"
}


job_start

###### 作业平台中执行脚本成功和失败的标准只取决于脚本最后一条执行语句的返回值
###### 如果返回值为0，则认为此脚本执行成功，如果非0，则认为脚本执行失败
###### 可在此处开始编写您的脚本逻辑代码


```

- 归档

```shell
#!/bin/bash

anynowtime="date +'%Y-%m-%d %H:%M:%S'"
NOW="echo [\`$anynowtime\`][PID:$$]"

##### 可在脚本开始运行时调用，打印当时的时间戳及PID。
function job_start
{
    echo "`eval $NOW` job_start"
    
    ts=`date +%Y-%m-%d_%H%M%S`
    archive_dir=/mnt/local/archive/${SERVICE_NAME}
    
    # 创建备份目录
    mkdir -p ${archive_dir}/$ts
    
    # 备份上一版本的jar
    cd ${archive_dir} || return
    log "备份上一版本..."
    mv ${archive_dir}/*.jar ${archive_dir}/$ts/

    
    # 下载Jenkins归档jar包
    URL=http://47.94.172.159:20020/nexus/repository/raw-local/bk/paas/9/micro/${SERVICE_NAME}/1.0.0/${SERVICE_NAME}-1.0.0-release.jar
    log "从Jenkins下载归档jar包...${URL}"
    wget ${URL} -O ${archive_dir}/${SERVICE_NAME}.jar
    
    log "归档中转完成"
}

##### 可在脚本执行成功的逻辑分支处调用，打印当时的时间戳及PID。 
function job_success
{
    MSG="$*"
    echo "`eval $NOW` job_success:[$MSG]"
    exit 0
}

##### 可在脚本执行失败的逻辑分支处调用，打印当时的时间戳及PID。
function job_fail
{
    MSG="$*"
    echo "`eval $NOW` job_fail:[$MSG]"
    exit 1
}

function log
{
    echo "`eval $NOW`:$*"
}

job_start

###### 作业平台中执行脚本成功和失败的标准只取决于脚本最后一条执行语句的返回值
###### 如果返回值为0，则认为此脚本执行成功，如果非0，则认为脚本执行失败
###### 可在此处开始编写您的脚本逻辑代码
```

- 更新jar并重启

```shell
#!/bin/bash

anynowtime="date +'%Y-%m-%d %H:%M:%S'"
NOW="echo [\`$anynowtime\`][PID:$$]"

##### 可在脚本开始运行时调用，打印当时的时间戳及PID。
function job_start
{
    echo "`eval $NOW` job_start"
      
    # 去除首尾空白     
    SERVICE_NAME=`echo ${SERVICE_NAME}`
    
    ip=`/sbin/ifconfig|sed -n '/inet addr/s/^[^:]*:\([0-9.]\{7,15\}\) .*/\1/p' | head -1`
    service_dir=/mnt/local/micro/${SERVICE_NAME}
    ts=`date +%Y-%m-%d_%H%M%S`
    
    #备份
    if [ ! -d "${service_dir}/bak" ]; then
        mkdir -p ${service_dir}/bak
    fi
    cd ${service_dir} || return
    log "备份..."
    mv -f ${service_dir}/*.jar ${service_dir}/bak/

    
    # 下载
    log "下载..."
    wget http://${ARCHIVE_SERVER_IP}:6666/archive/${SERVICE_NAME}/${SERVICE_NAME}.jar -O  ${service_dir}/${SERVICE_NAME}.jar
    
    # 重启
    log "重启..."
    supervisorctl stop ${SERVICE_NAME}
    supervisorctl start ${SERVICE_NAME}
    #PID=supervisorctl status | grep ${SERVICE_NAME} | awk '{print $4}' | sed  's/,*$//g'
    #kill $PID
    
    sleep 120

}

##### 可在脚本执行成功的逻辑分支处调用，打印当时的时间戳及PID。 
function job_success
{
    MSG="$*"
    echo "`eval $NOW` job_success:[$MSG]"
    exit 0
}

##### 可在脚本执行失败的逻辑分支处调用，打印当时的时间戳及PID。
function job_fail
{
    MSG="$*"
    echo "`eval $NOW` job_fail:[$MSG]"
    exit 1
}

function log
{
    echo "`eval $NOW`:$*"
}


job_start

###### 作业平台中执行脚本成功和失败的标准只取决于脚本最后一条执行语句的返回值
###### 如果返回值为0，则认为此脚本执行成功，如果非0，则认为脚本执行失败
###### 可在此处开始编写您的脚本逻辑代码

```