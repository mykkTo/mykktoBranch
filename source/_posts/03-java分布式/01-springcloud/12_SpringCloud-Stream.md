---
title: SpringCloud-Stream消息驱动
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 'SpringCloud-Stream是一个构建消息驱动微服务的框架，屏蔽底层消息中间件的差异,降低切换成本，统一消息的编程模型'
tags:
  - springcloud
  - 消息驱动
  - stream
categories:
  - 分布式技术栈
abbrlink: e867710e
reprintPolicy: cc_by
date: 2022-02-19 22:52:16
coverImg:
img:
password:
---





## 友情链接

[rabitMQ安装（docker）](/posts/699f8954.html)



## 1、消息驱动概述

### 1、是什么

屏蔽底层消息中间件的差异,降低切换成本，统一消息的编程模型



官网：https://m.wang1314.com/doc/webapp/topic/20971999.html



### 2、设计思想

#### 1、标准MQ

- Message：生产者/消费者之间靠消息媒介传递信息内容
- MessageChannel（消息通道）：消息必须走特定的`通道`
- 消息通道里的消息如何被消费呢，谁负责`收发`处理
  - 消息通道MessageChannel的子接口SubscribableChannel，由MessageHandler消息处理器所订阅





#### 2、为什么用Cloud Stream

##### 1、概念

比方说我们用到了RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，
像RabbitMQ有exchange，kafka有Topic和Partitions分区，

这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的，`一大堆东西都要重新推倒重新做`，因为它跟我们的系统耦合了，这时候springcloud Stream给我们提供了一种解耦合的方式。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219223621.png)



##### 2、Stream中的消息通信方式遵循了发布-订阅模式

Topic主题进行广播：

- 在RabbitMQ就是Exchange
- 在Kakfa中就是Topic



##### 3、Spring Cloud Stream标准流程套路

- Binder：
  - 很方便的连接中间件，屏蔽差异
- Channel：
  - 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置
- Source和Sink：
  - 简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219225529.png)

#### 4、编码API和常用注解

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219225657.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220219225704.png)



## 2、案例说明

工程中新建三个子模块

- cloud-stream-rabbitmq-provider8801， 作为生产者进行发消息模块
- cloud-stream-rabbitmq-consumer8802，作为消息接收模块
- cloud-stream-rabbitmq-consumer8803  作为消息接收模块







## 3、消息驱动之生产者

### 1、新建Module

cloud-stream-rabbitmq-provider8801

### 2、pom

```xml
       <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
```



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

    <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!--基础配置-->
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
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

```yml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 101.34.180.133
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址

```



### 4、主启动

```java
package com.kk.springcloud;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args) {
        SpringApplication.run (StreamMQMain8801.class, args);
    }
}

```



### 5、业务类



#### 1、接口

```java
package com.kk.springcloud.service;

public interface IMessageProvider {
    String send();
}
```





#### 2、接口实现

```java
package com.kk.springcloud.service.impl;

import com.kk.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.MessageChannel;
import org.springframework.integration.support.MessageBuilder;
import javax.annotation.Resource;
import java.util.UUID;

@EnableBinding(Source.class) // 可以理解为是一个消息的发送管道的定义
public class MessageProviderImpl implements IMessageProvider {
    @Resource
    private MessageChannel output; // 消息的发送管道

    @Override
    public String send() {
        String serial = UUID.randomUUID ( ).toString ( );
        this.output.send (MessageBuilder.withPayload (serial).build ( )); // 创建并发送消息
        System.out.println ("***serial: " + serial);
        return serial;
    }
}

```



#### 3、controller

```java
package com.kk.springcloud.controller;

import com.kk.springcloud.service.IMessageProvider;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;

@RestController
public class SendMessageController {
    @Resource
    private IMessageProvider messageProvider;

    @GetMapping(value = "/sendMessage")
    public String sendMessage() {
        return messageProvider.send ( );
    }
}

```



### 6、测试

- 启动eureka，以及rabbitMq，以及8801
- 访问：`http://localhost:8801/sendMessage`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220220195046.png)

## 4、消息驱动之消费者

### 1、新建Module

cloud-stream-rabbitmq-consumer8802

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

    <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--基础配置-->
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
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 101.34.180.133
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址

```



### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8802 {
    public static void main(String[] args) {
        SpringApplication.run (StreamMQMain8802.class, args);
    }
}

```



### 5、业务类

```java
package com.kk.springcloud.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListener {
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println ("消费者1号，------->接收到的消息：" + message.getPayload ( ) + "\t port: " + serverPort);
    }
}

```



### 6、测试

url：`http://localhost:8801/sendMessage`



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220220225413.png)

## 5、分组消费与持久化

### 1、依照8802，clone出来一份运行8803

- cloud-stream-rabbitmq-consumer8803

- ```java
  @SpringBootApplication
  public class StreamMQMain8803 {
      public static void main(String[] args) {
          SpringApplication.run (StreamMQMain8803.class, args);
      }
  }
  ```

- ```java
  @Component
  @EnableBinding(Sink.class)
  public class ReceiveMessageListener {
      @Value("${server.port}")
      private String serverPort;
  
      @StreamListener(Sink.INPUT)
      public void input(Message<String> message) {
          System.out.println ("消费者2号，------->接收到的消息：" + message.getPayload ( ) + "\t port: " + serverPort);
      }
  }
  ```

  





### 2、启动

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220220231746.png)

### 3、运行后有两个问题

- 有重复消费问题
- 消息持久化问题

### 4、重复消费问题



目前是8802/8803同时都收到了，存在重复消费问题

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220220231944.png)



##### 如何解决：`分组和持久化属性group`

比如在如下场景中，订单系统我们做集群部署，都会从RabbitMQ中获取订单信息，
那`如果一个订单同时被两个服务获取到`，那么就会造成数据错误，我们得避免这种情况。
`这时我们就可以使用Stream中的消息分组来解决`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220220232152.png)



注意在Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。
`不同组是可以全面消费的(重复消费)，
同一组内会发生竞争关系，只有其中一个可以消费。`

### 5、分组

#### 1、原理

微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。
`不同的组是可以消费的，同一个组内会发生竞争关系，只有其中一个可以消费`。



#### 2、划分组(不同组)

- 8802/8803都变成不同组，group两个不同组（kkA,kkB）
- 8802修改YML

```yml
          group: kkA
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221221056.png)

- 8803修改YML

```yml
          group: kkB
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221221835.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221223137.png)





#### 3、结论(不同组)

不同组的还是重复消费

#### 4、划分组(不同组)

两边都配置为：kkA



#### 5、结论(不同组)

同一个组的多个微服务实例，每次只会有一个拿到。（解决重复消费问题）



### 6、持久化

目前两个消费者8802/8803都是归类于 kkA分组



1、下面要做的是将 8802分组配置注释

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221230211.png)



2、关闭8002/8003服务

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221232008.png)



3、8801先发送4条消息到rabbitmq

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221232637.png)



4、启动8802（无分组），没有任何打印



5、启动8803（有分组），接收离线数据

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221233738.png)

