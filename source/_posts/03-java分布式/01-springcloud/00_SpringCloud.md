---
title: SpringCloud and Alibaba合集
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: SpringCloud
tags:
  - springcloud
  - springcloud-alibaba
  - 合集
categories:
  - 分布式技术栈
abbrlink: '58914314'
reprintPolicy: cc_by
date: 2022-01-11 00:42:15
coverImg:
img:
password:
---

## 案例代码仓库

https://gitee.com/TK_LIMR/springcloud2021To2021.git

## 分解的目录



### [1、项目构建and技术选型]()



### [2、Eureka服务注册与发现]()



### [4、Consul服务注册与发现]()

### [5、Ribbon负载均衡服务调用]()

### [6、OpenFeign服务接口调用]()

### [7、Hystrix断路器]()

### [8、zuul路由网关]()



### [9、Gateway新一代网关]()

### [10、Seata处理分布式事务]()



### [11、Sentinel实现熔断与限流]()

### [12、Nacos服务注册和配置中心]()

### [13、Sleuth分布式请求链路追踪]()

### [14、Stream消息驱动]()

### [15、Bus 消息总线]()

### [16、config分布式配置中心]()



## 食用技巧



### 1、同时启动多个Springboot



IDEA SpringBoot多个项目 开启 RunDashboard，

在项目根目录 .idea 文件夹  中   workspace.xml文件中加入

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220123123656.png)

```xml
<component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
  </component>
```



### 2、本地hosts配置

windown 10位置：`C:\Windows\System32\drivers\etc`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220131181324.png)