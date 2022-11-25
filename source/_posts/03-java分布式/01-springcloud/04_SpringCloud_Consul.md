---
title: SpringCloud-Consul
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: Consul用 Go 语言开发，提供服务治理、配置中心、控制总线等功能的完整服务网格解决方案。
tags:
  - springcloud
  - 服务注册与发现
  - Consul
categories:
  - 分布式技术栈
abbrlink: 51c7125f
reprintPolicy: cc_by
date: 2022-01-16 21:42:11
coverImg:
img:
password:
---



## 1、Consul简介

### 1、是什么

- Consul 是一套开源的分布式服务发现和配置管理系统，由 HashiCorp 公司用 Go 语言开发。

- 提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。

- 它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持 HTTP 和 DNS 协议 支持跨数据中心的 WAN 集群 提供图形界面 跨平台，支持 Linux、Mac、Windows

  

### 2、能干嘛

#### 1、服务发现

提供HTTP和DNS两种发现方式。

#### 2、健康监测

支持多种方式，HTTP、TCP、Docker、Shell脚本定制化监控

#### 3、KV存储

Key、Value的存储方式

#### 4、多数据中心

Consul支持多数据中心

#### 5、可视化Web界面

### 3、官网文档

#### 1、下载地址

https://www.consul.io/downloads.html

#### 2、学习文档

https://www.springcloud.cc/spring-cloud-consul.html

## 2、安装并运行Consul



### 1、官网安装

https://www.consul.io/downloads

### 2、安装说明

下载完成后只有一个consul.exe文件，硬盘路径下双击运行，查看版本号信息

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117220306.png)

### 3、使用开发模式启动

（1）命令：`consul agent -dev`

（2）通过以下地址可以访问Consul的首页：`http://localhost:8500`

（3）效果：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117220604.png)

## 3、服务提供者

### 1、建model

cloud-providerconsul-payment8006



### 2、pom

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
    <artifactId>cloud-providerconsul-payment8006</artifactId>

    <dependencies>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
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



### 3、yml

```yml
###consul服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```



### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Payment8006 {
    public static void main(String[] args) {
        SpringApplication.run (Payment8006.class,args);
    }
}
```



### 5、业务类

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.UUID;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/payment/consul")
    public String paymentInfo() {
        return "springcloud with consul: " + serverPort + "\t\t" + UUID.randomUUID ( ).toString ( );
    }
}
```



### 5、测试



#### 1、访问

`http://localhost:8006/payment/consul`



#### 2、控制台

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117222004.png)



## 4、服务消费者

### 1、建model

cloud-consumerconsul-order82

### 2、pom

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
    <artifactId>cloud-consumerconsul-order82</artifactId>

    <dependencies>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
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



### 3、yml

```yml
###consul服务端口号
server:
  port: 82

spring:
  application:
    name: cloud-consumer-order
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}

```



### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderConsulMain82 {
    public static void main(String[] args) {
        SpringApplication.run (OrderConsulMain82.class,args);
    }
}

```



### 5、业务类

#### 1、Bean配置类

```java
package com.kk.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextBean {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate ( );
    }
}

```



#### 2、Controller

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class OrderConsulController {
    public static final String INVOKE_URL = "http://cloud-provider-payment"; //consul-provider-payment

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/consul")
    public String paymentInfo() {
        String result = restTemplate.getForObject (INVOKE_URL + "/payment/consul", String.class);
        System.out.println ("消费者调用支付服务(consule)--->result:" + result);
        return result;
    }
}

```



### 6、测试

#### 1、客户端

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117222922.png)

#### 2、访问

`http://localhost/consumer/payment/consul`



## 5、三个注册中心异同点

### 1、CAP



CAP理论关注粒度是数据，而不是整体系统设计的策略



- C:Consistency（强一致性）
- A:Availability（可用性）
- P:Partition tolerance（分区容错性）



### 2、CAP图

最多只能同时较好的满足两个。

-  CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，
  因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：
  CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
  CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
  AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117223429.png)

### 3、AP-CP架构图

#### 1、AP(Eureka)

AP架构

当网络分区出现后，为了保证可用性，系统B可以返回旧值，保证系统的可用性。
结论：违背了一致性C的要求，只满足可用性和分区容错，即AP



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117223509.png)

#### 2、CP(Zookeeper/Consul)

CP架构

当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性
结论：违背了可用性A的要求，只满足一致性和分区容错，即CP

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220117223549.png)