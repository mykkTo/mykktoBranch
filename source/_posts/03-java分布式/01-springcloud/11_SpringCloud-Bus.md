---
title: SpringCloud-Bus分布式节点链接
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架
tags:
  - springcloud
  - bus
  - 节点链接
categories:
  - 分布式技术栈
abbrlink: 699f8954
reprintPolicy: cc_by
date: 2022-02-06 11:49:12
coverImg:
img:
password:
---



## 1、概述

### 1、是什么

Bus支持两种消息代理：RabbitMQ 和 Kafka



Spring Cloud Bus 配合 Spring Cloud Config 使用可以实现配置的动态刷新。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220208124710.png)



Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架，
`它整合了Java的事件处理机制和消息中间件的功能`。

Spring Clud Bus目前支持RabbitMQ和Kafka。

### 2、能干嘛

Spring Cloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道。



### 3、为何被称为总线



#### 什么是总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。



#### 基本原理

ConfigClient实例都监听MQ中同一个topic(默认是springCloudBus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置。



## 2、RabbitMQ环境配置

### 1、安装

安装docker：https://www.jianshu.com/p/f554c85b25c1

#### 1、拉取

`docker pull  rabbitmq:3.7.7-management`

#### 2、创建容器并启动

`docker run -d --hostname localhost --name myrabbit -p 15672:15672 -p 5672:5672 rabbitmq:3.7.7-management`



- -d 后台运行容器；

- --name 指定容器名；

- -p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；

- -v 映射目录或文件；

- --hostname  主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；

- -e 指定环境变量；
  - 默认的用户名：`guest`；
  - 默认用户名的密码：`guest`）

#### 3、访问

http://localhost:15672/  （换成自己服务器的IP）

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220208150738.png)



## 3、SpringCloud Bus 动态刷新全局广播

### 1、演示广播效果，增加复杂度

以3355为模板再制作一个3366

#### 1、建model

cloud-config-client-3366

#### 2、pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud2021to2022</artifactId>
        <groupId>com.kk</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-config-client-3366-2</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```





#### 3、bootstrap.yml

```yml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址
      
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/

```



#### 4、主启动

```java
package com.kk.springcloud;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class ConfigClientMain3366 {
    public static void main(String[] args) {
        SpringApplication.run (ConfigClientMain3366.class,args);
    }
}

```



#### 5、业务类

```java
package com.kk.springcloud.controller;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class ConfigClientController {
    @Value("${server.port}")
    private String serverPort;

    @Value("${mytest.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String configInfo() {
        return "serverPort: " + serverPort + "\t\n\n configInfo: " + configInfo;
    }
}

```



### 2、给cloud-config-center-3344配置中心`服务端`添加消息总线支持

#### 1、pom

```xml
<!--添加消息总线RabbitMQ支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```





#### 2、yml

```yml
##rabbitmq相关配置,暴露bus刷新配置的端点
management:
  endpoints: #暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'
```





#### 



### 3、给cloud-config-client-3355`客户端`添加消息总线支持

#### 1、pom

```xml
        <!--添加消息总线RabbitMQ支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```



#### 2、bootstrap.yml

```yml
#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```



```yml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址
#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
  rabbitmq:
    host: 106.xx.xx.xx
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/

# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"

```



#### 

### 4、给cloud-config-client-3366`客户端`添加消息总线支持

#### 同上



### 5、测试

##### 1、修改Github上配置文件增加版本号

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219122545.png)

##### 2、发送 post 请求

`curl -X POST "http://localhost:3344/actuator/bus-refresh`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219122714.png)



##### 3、配置中心

`http://config-3344.com:3344/config-dev.yml`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219185957.png)



同步更新了配置信息



##### 4、客户端

`http://localhost:3355/configInfo`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219190101.png)



发现并没有更新，运行指定 定点试试：`curl -X POST "http://localhost:3355/actuator/bus-refresh`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219190442.png)



运行后发现就更新了，很奇怪哦

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219190516.png)



##### 5、结论

一次修改，广播通知，处处生效（目前只有服务端是这样的，客户端只能手动，问题原因未知）



## 4、SpringCloud Bus 动态刷新定点(局部)通知

### 1、概念

指定具体某一个实例生效而不是全部 

### 2、公式

http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}



/bus/refresh请求不再发送到具体的服务实例上，而是发给config server并
通过destination参数类指定需要更新配置的服务或实例

### 3、案例

只刷新 3355



`curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"`





### 





## 参考：

https://blog.csdn.net/haoweng4800/article/details/102946846

https://www.cnblogs.com/huanshilang/p/12585877.html

https://www.jianshu.com/p/efac7bd8941f