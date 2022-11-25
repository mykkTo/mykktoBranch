---
title: Springcloud-Zookeeper(快速入门)
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: Springcloud-Zookeeper(快速入门)，全面版本后面会在大数据相关进行更新
tags:
  - springcloud
  - 服务注册与发现
  - Zookeeper
categories:
  - 分布式技术栈
abbrlink: c0125b8a
reprintPolicy: cc_by
date: 2022-01-15 23:01:12
coverImg:
img:
password:
---



## 安装参考【后面写】

https://blog.csdn.net/zhou_fan_xi/article/details/103275955



安装完，windown访问测试

`telnet 106.52.23.202 2181`



## 1、注册中心Zookeeper

### 基本概述

- zookeeper是一个分布式协调工具，可以实现注册中心功能

- 关闭Linux服务器防火墙后才能启动zookeeper服务器
- zookeeper服务器取代Eureka服务器，zk作为服务注册中心

## 2、服务提供者

### 1、建model

新建cloud-provider-payment8004

### 2、写pom

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
    <artifactId>cloud-provider-payment8004</artifactId>

    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.9版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
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



### 3、改yml

```yml
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004
#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 106.52.23.202:2181
```



### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient//该注解用于向使用consul或者zookeeper作为注册中心时注册服务
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run (PaymentMain8004.class,args);
    }
}
```



### 5、业务类

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.UUID;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/zk")
    public String paymentzk() {
        return "springcloud with zookeeper: " + serverPort + "\t" + UUID.randomUUID ( ).toString ( );
    }
}
```



### 6、启动8004注册进zookeeper

#### 1、liunx启动 zookeeper

```shell
#查看执行路径
[root@VM-0-13-centos bin]# pwd
/root/zookeeper-3.4.9/bin

#启动
[root@VM-0-13-centos bin]# ./zkServer.sh start
#停止
[root@VM-0-13-centos bin]# ./zkServer.sh stop
#重启
[root@VM-0-13-centos bin]# ./zkServer.sh restart

```





### 7、测试

### 1、访问服务

`http://localhost:8004/payment/zk`



### 2、查看服务端被是否被注册

```shell
#进入 zookeeper客户端
[root@VM-0-13-centos bin]# ./zkCli.sh
WatchedEvent state:SyncConnected type:None path:null

#查看序列
[zk: localhost:2181(CONNECTED) 0] ls /services/cloud-provider-payment
[a6293666-45c1-490e-adb8-bee63f121983]

#查看详细信息
[zk: localhost:2181(CONNECTED) 1] ls /services/cloud-provider-payment/a6293666-45c1-490e-adb8-bee63f121983
[]

[zk: localhost:2181(CONNECTED) 2] get  /services/cloud-provider-payment/a6293666-45c1-490e-adb8-bee63f121983
{"name":"cloud-provider-payment","id":"a6293666-45c1-490e-adb8-bee63f121983","address":"FYYX-2020GVNPLA","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1642423897607,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}

```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117211751.png)

### 8、思考

服务节点是临时节点还是持久节点？？

（1）当我关闭 8004节点后，zookeeper 依旧会持续发心跳，当有接收到反馈则还有序列号，没有则返回空[]



（2）然后再次启动，则返回一个新的序列，由此说明这是一个临时节点

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117213317.png)



## 3、服务消费者

### 1、建model

新建cloud-consumerzk-order81

### 2、写pom

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

    <artifactId>cloud-consumerzk-order81</artifactId>
    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.9版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
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



### 3、改yml

```yml
#81表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 81
#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-consumer-order
  cloud:
    zookeeper:
      connect-string: 106.52.23.202:2181
```



### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderZK81 {
    public static void main(String[] args) {
        SpringApplication.run (OrderZK81.class,args);
    }
}
```



### 5、业务类

#### 1、配置Bean

```java
package com.kk.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextBean {

    @Bean
    @LoadBalanced// 给予 RestTemplate 负载均衡的能力
    public RestTemplate getRestTemplate() {
        return new RestTemplate ( );
    }
}
```



#### 2、Controller

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class OrderZKController81 {

    public static final String INVOKE_URL = "http://cloud-provider-payment";

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping(value = "/consumer/payment/zk")
    public String paymentInfo() {
        String result = restTemplate.getForObject (INVOKE_URL + "/payment/zk", String.class);
        System.out.println ("消费者调用支付服务(zookeeper)--->result:" + result);
        return result;
    }
}
```





### 6、测试

#### 1、zookeeper

```shell
[zk: localhost:2181(CONNECTED) 21] ls /services
[cloud-provider-payment, cloud-consumer-order]
```



#### 2、访问

`http://localhost:81/consumer/payment/zk`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117215134.png)