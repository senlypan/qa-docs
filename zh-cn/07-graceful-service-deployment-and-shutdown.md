# Eurake / Nacos - 服务优雅上下线

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.07-graceful-service-deployment-and-shutdown&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2025-09-21
>
> 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）


## 一、Eureka 服务优雅上下线方式

### 1.1、退出服务 - Out of Service 

调用 Eureka 退出服务接口，下线指定服务，下线后服务不再接收新请求。

接口地址：`/eureka/apps/{ServerName}/{Instance}/status?value=OUT_OF_SERVICE`

### 1.2、 上线服务 - Up

调用 Eureka 上线服务接口，上线指定服务，上线后服务可接收新请求。

接口地址：`/eureka/apps/{ServerName}/{Instance}/status?value=UP`

### 1.3、注意事项

1、客户端存在 30 秒的拉新动作，可在服务上下线 30 秒后再进行新服务部署或停止。

## 二、Nacos 服务优雅上下线方式

### 2.1、Nacos 下线调用 URL 规范

```shell

http://192.168.10.56:8848/nacos/v1/ns/instance?namespaceId=${NAMESPACEID}&serviceName=${SERVICENAME}&groupName=${GROUPNAME}&ip=${PODIP}&port=${SERVICEPORT}&enabled=false

```

### 2.2、参数说明

```shell

namespaceId:${NAMESPACEID} 命名空间ID
serviceName:${SERVICENAME} 应用服务名称
groupName:${GROUPNAME}  分组名称
ip:${PODIP}  应用服务IP，在 k8s 中为 POD IP
port:${SERVICEPORT}  应用服务端口
enabled:false  可用状态


```

### 2.3、注意

由于 Nacos 遵循 RESTFUL 接口风格，故需使用 PUT 协议进行调用。

### 2.4、在 k8s （linux） 执行下线脚本示例

```shell

curl -x PUT 'http://192.168.10.56:8848/nacos/v1/ns/instance?namespaceId=dev&serviceName=odp-user-management&groupName=odp&ip=192.168.11.72&port=9203&enabled=false'


```