---
title: SpringCloud-OpenFeign远程调用服务
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: SpringCloud-OpenFeign远程调用服务
tags:
  - netflix
  - springcloud
  - 远程调用服务
  - openfeign
categories:
  - 分布式技术栈
abbrlink: a723ac51
reprintPolicy: cc_by
date: 2022-01-21 21:32:13
coverImg:
img:
password:
---

## 1、概述

### 1、是什么



https://github.com/spring-cloud/spring-cloud-openfeign

Feign是一个声明式的Web服务客户端，让编写Web服务客户端变得非常容易，`只需创建一个接口并在接口上添加注解即可`



### 2、能干嘛

#### 1、 Feign能干什么

Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，`往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。`所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，`我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)`，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。





#### 2、Feign集成了Ribbon

利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，`通过feign只需要定义服务绑定接口且以声明式的方法`，优雅而简单的实现了服务调用



### 3、区别



#### 1、feign

Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端
Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```





#### 2、openfeign

OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```





## 2、OpenFeign使用步骤





### 1、建model

新建cloud-consumer-feign-order80

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
    <artifactId>cloud-consumer-feign-order80</artifactId>

    <dependencies>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础通用配置-->
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
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/


```



### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run (OrderFeignMain80.class,args);
    }
}
```



### 5、业务类

#### 1、新建PaymentFeignService接口并新增注解@FeignClient

```java
package com.kk.springcloud.service;

import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}

```



#### 2、控制层Controller

```java
package com.kk.springcloud.controller;

import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import com.kk.springcloud.service.PaymentFeignService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;

@RestController
public class OrderFeignController {
    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById (id);
    }
}

```



### 6、测试

#### 1、启动顺序

- 启动 7001/7002 eureka集群
- 启动 8001/8002 生产者集群
- 启动 OpenFeign:80  消费者



#### 2、访问

`http://localhost/consumer/payment/get/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220121235624.png)

#### 3、Feign自带负载均衡配置项



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220121235624.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220121235644.png)

### 7、小结

![](https://v1.mykkto.cn/image/blog/2022/springcloud/sdjghj5hjyhj.png)



## 3、OpenFeign超时控制

### 1、模拟超时

超时设置，故意设置超时演示出错情况

##### 1、服务提供方`8001`故意写暂停程序

`com.kk.springcloud.controller.PaymentController;`

```java
    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeOut() {
        System.out.println ("*****paymentFeignTimeOut from port: " + serverPort);
        //暂停几秒钟线程
        try {
            TimeUnit.SECONDS.sleep (3);
        } catch (InterruptedException e) {
            e.printStackTrace ( );
        }
        return serverPort;
    }
```



##### 2、服务消费方`80`添加超时方法PaymentFeignService

`com.kk.springcloud.service.PaymentFeignService`

```java
    @GetMapping(value = "/payment/feign/timeout")
    String paymentFeignTimeOut();
```



##### 3、服务消费方80添加超时方法OrderFeignController

`com.kk.springcloud.controller.OrderFeignController`

```java

    @GetMapping(value = "/consumer/payment/feign/timeout")
    public String paymentFeignTimeOut() {
        return paymentFeignService.paymentFeignTimeOut ( );
    }
```



##### 4、测试

url：`http://localhost/consumer/payment/feign/timeout`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220123131209.png)



##### 5、小结



OpenFeign默认等待1秒钟，超过后报错 



### 2、是什么

#### 1、概述

默认Feign客户端只等待一秒钟，但是服务端处理需要超过1秒钟，导致Feign客户端不想等待了，直接返回报错。
为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制。





#### 2、默认支持Ribbon

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220123150839.png)



#### 3、超时时间配置

```yml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220123150435.png)





## 4、OpenFeign日志打印功能

#### 1、是什么

Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。
说白了就是`对Feign接口的调用情况进行监控和输出`





#### 2、日志级别

> NONE：默认的，不显示任何日志；
>
> BASIC：仅记录请求方法、URL、响应状态码及执行时间；
>
> HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
>
> FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。



#### 3、配置日志 bean

```java
package com.kk.springcloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenFeignConfig {
    @Configuration
    public class FeignConfig {
        @Bean
        Logger.Level feignLoggerLevel() {
            return Logger.Level.FULL;
        }
    }
}

```



#### 4、yml中配置

```yml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.kk.springcloud.service.PaymentFeignService: debug
```

```yml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/

#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000

logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.kk.springcloud.service.PaymentFeignService: debug

```



#### 5、后台启动查看

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220123151946.png)

