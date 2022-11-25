---
title: SpringCloud-Eureka
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: SpringCloud-eureka一款比较早的服务注册框架，2.0x已经停止更新，但是大部分企业的老项目业务仍有在使用
tags:
  - springcloud
  - 服务注册与发现
  - Eureka
categories:
  - 分布式技术栈
abbrlink: 402d692e
reprintPolicy: cc_by
date: 2022-01-14 22:35:22
coverImg:
img:
password:
---



## 1、Eureka基础知识

### 1、什么是服务治理　

- Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务治理
-  在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

### 2、什么是服务注册

#### 1、概念

- Eureka采用了CS的设计架构，Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。
- 在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息 比如 服务地址通讯地址等以别名方式注册到注册中心上。另一方（消费者|服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))

#### 2、图解

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114212118.png)

### 3、Eureka两组件

#### 1、`EurekaServer`提供服务注册服务

各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

#### 2、`EurekaClient`通过注册中心进行访问

是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

## 2、单机Eureka构建步骤

### 1、eurekaServer端服务注册中心

#### 1、建model

cloud-eureka-server7001

#### 2、写pom

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
    <artifactId>cloud-eureka-server7001</artifactId>

    <dependencies>
        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--boot web actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般通用配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>

</project>

```



#### 3、改yml

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
    #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```



#### 4、主启动

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run (EurekaMain7001.class,args);
    }
}
```



#### 5、测试

1、访问：`http://localhost:7001/`



2、返回结果页面

![1642172826169](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1642172826169.png)

>No application available 没有服务被发现 O(∩_∩)O
>因为没有注册服务进来当然不可能有服务被发现

### 2、EurekaClient端provider-8001

#### 1、建model(不变)

cloud-provider-payment8001

#### 2、写pom（增加）

```xml
以前老版本，别再使用
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
 
 
现在新版本,当前使用
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



#### 3、改yml(增加)

```yml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114231354.png)

#### 4、主启动(添加)

```java
@EnableEurekaClient
```





#### 5、测试

（1）先要启动EurekaServer，再启动8001

（2）访问：`http://localhost:7001/`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114232422.png)



#### 6、自我保护机制

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114232802.png)

### 3、EurekaClient端consumer-80

#### 1、建model（不变）

#### 2、写pom（同上）

```xml
        <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
        <!--引入公共部分-->
        <dependency>
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
```



#### 3、改yml（同上）

```yml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```



#### 4、主启动（同上）

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run (PaymentMain8001.class, args);
    }
}
```



#### 5、测试

（1）先要启动EurekaServer，再启动80

（2）访问：`http://localhost/consumer/payment/get/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116095006.png)





## 3、集群Eureka构建步骤

### 1、Eureka集群原理说明

#### 1、图解

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116095142.png)

#### 2、问题：微服务RPC远程服务调用最核心的是什么 



#### 3、解决方案：

高可用，试想你的注册中心只有一个only one，会导致整个为服务环境不可用，所以

解决办法：搭建Eureka注册中心集群 ，实现负载均衡+故障容错



### 2、EurekaServer集群环境构建步骤



#### 1、建model

新建cloud-eureka-server7002，参考7001

#### 2、写pom

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
    <artifactId>cloud-eureka-server7002</artifactId>

    <dependencies>
        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--boot web actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般通用配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### 

#### 3、修配置

由于本地主机只有一台，为了模拟两台（多台)，则将 多个ip（域名eureka7001.com,eureka7002.com) 指向本地(127.0.0.1)

##### 1、找到对应的配置目录`C:\Windows\System32\drivers\etc`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116111940.png)

##### 2、写入配置文件

>127.0.0.1  eureka7001.com
>127.0.0.1  eureka7002.com

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116112237.png)



#### 4、改yml

##### 1、7001

```yml
server:
  port: 7001
  
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/

```



##### 2、7002

```yml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```



#### 5、主启动

```java
package com.kk.springcloud;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7002 {
    public static void main(String[] args) {
        SpringApplication.run (EurekaMain7002.class,args);
    }
}
```



### 3、支付8001发布到Eureka集群

#### 1、修改yml

主要修改一处：defaultZone:

```yml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/spring_cloud?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: a1b2c3

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.kk.springclond.entities
```



### 4、订单80发布到Eureka集群

#### 1、修改yml

主要修改一处：defaultZone:

```yml
server:
  port: 80

spring:
  application:
    name: cloud-consumer-service

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
```



### 5、测试流程

测试以上配置是否生效

#### 1、启动集群

先要启动EurekaServer，7001/7002服务

#### 2、启动生产者

再要启动服务提供者provider，8001

#### 3、启动消费者

再要启动消费者，80

#### 4、测试

`http://localhost/consumer/payment/get/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116121049.png)



### 6、支付8001集群构建【8002】

#### 1、方式一：

直接把整份8001 copy出一个新的，需改yml ，port即可【推荐】

```yml
server:
  port: 8002
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116131217.png)

#### 2、方式二：

（1）打开idea配置，取消勾选单一服务按钮

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116122129.png)



（2）修改yml

```yml
server:
  port: 8002
```

（3）启动

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116122301.png)



### 7、负载均衡

##### 1、订单80调用调整

在调用的时候不能写死成ip+端口，需要使用服务名：`cloud-payment-service`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116134417.png)



##### 2、@LoadBalanced注解

使用@LoadBalanced注解赋予RestTemplate负载均衡的能力

修改配置信息添加注解【80端口】

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116133032.png)

### 8、测试负载均衡

#### 1、启动顺序说明

（1）先要启动EurekaServer，7001/7002服务

（2）再要启动服务提供者provider，8001/8002服务

#### 2、访问

`http://localhost/consumer/payment/get/1`

#### 3、结果

（1）达到负载效果



（2）8001/8002端口交替出现



#### 4、小结

Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还有负载功能了



## 4、actuator微服务信息完善

### 1、主机名称:服务名称修改

#### 1、当前问题

含有主机名名称

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116135403.png)

#### 2、修改8001

```yml
  instance:
    instance-id: payment8001
```



完整 yml

```yml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/spring_cloud?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: a1b2c3

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
  instance:
    instance-id: payment8001

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.kk.springclond.entities
```

#### 3、效果

其实就是取别名

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116140426.png)

### 2、访问信息有IP信息提示

#### 1、问题：

没有IP提示

#### 2、修改8001

```yml
    prefer-ip-address: true     #访问路径可以显示IP地址
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116140654.png)

#### 3、效果

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116140925.png)

## 5、服务发现Discovery

### 1、概述

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息



### 2、修改8001的Controller

#### 1、增加

```java
    @Resource
    private DiscoveryClient discoveryClient;

    @Value("${server.port}")
    private String serverPort;

    /**
     * 查看注册服务的信息
     * @return
     */
    @GetMapping(value = "/payment/discovery")
    public Object discovery() {
        // 目前已经注册的微服务列表
        List<String> services = discoveryClient.getServices ( );
        for (String element : services) {
            log.info (element);
        }

        // 根据名称获取的微服务实例（就像类和对象的关系）
        List<ServiceInstance> instances = discoveryClient.getInstances ("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance element : instances) {
            log.info (element.getServiceId ( ) + "\t" + element.getHost ( ) + "\t" + element.getPort ( ) + "\t"
                    + element.getUri ( ));
        }
        return this.discoveryClient;
    }
```



#### 2、全部

```java
package com.kk.springcloud.controller;

import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import com.kk.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.*;
import javax.annotation.Resource;
import java.util.List;

@RestController
@Slf4j
/*
 * @Description:
 * @Author:         阿K
 * @CreateDate:     2022/1/16 18:11
 * @Param:
 * @Return:
 **/
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Resource
    private DiscoveryClient discoveryClient;

    @Value("${server.port}")
    private String serverPort;

    /**
     * 查看注册服务的信息
     *
     * @return
     */
    @GetMapping(value = "/payment/discovery")
    public Object discovery() {
        // 目前已经注册的微服务列表
        List<String> services = discoveryClient.getServices ( );
        for (String element : services) {
            log.info (element);
        }

        // 根据名称获取的微服务实例（就像类和对象的关系）
        List<ServiceInstance> instances = discoveryClient.getInstances ("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance element : instances) {
            log.info (element.getServiceId ( ) + "\t" + element.getHost ( ) + "\t" + element.getPort ( ) + "\t"
                    + element.getUri ( ));
        }
        return this.discoveryClient;
    }


    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment) {
        int result = paymentService.create (payment);
        log.info ("*****插入结果1：" + result + "111");
        if (result > 0) {  //成功
            return new CommonResult (200, "插入数据库成功", result);
        } else {
            return new CommonResult (444, "插入数据库失败", null);
        }
    }

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id) {
        Payment payment = paymentService.getPaymentById (id);
        log.info ("*****查询结果：" + payment);
        if (payment != null) {  //说明有数据，能查询成功
            return new CommonResult (200, "查询成功8081", payment);
        } else {
            return new CommonResult (444, "没有对应记录，查询ID：" + id, null);
        }
    }
}

```



### 3、8001主启动

#### 1、增加

```java
@EnableDiscoveryClient
```

#### 2、全部

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient// 此注解后期若是不用 Eureka，将被下方注解所代替
@EnableDiscoveryClient// 服务注册发现
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run (PaymentMain8001.class, args);
    }
}
```



### 4、测试



#### 1、启动顺序

先要启动EurekaServer，再启动8001主启动类，需要稍等一会儿

#### 2、访问

`http://localhost:8001/payment/discovery`



## 6、Eureka自我保护

### 1、故障现象

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，
Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。



如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式：
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. 
RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE 

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116182547.png)

### 2、导致原因



此举属于CAP里面的AP分支



一句话：某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存



#### 1、问题一：

**为什么会产生Eureka自我保护机制？**

为了防止EurekaClient可以正常运行，但是 与 EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除



#### 2、问题二：

**什么是自我保护模式？**

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116182942.png)

### 3、怎么禁止自我保护

#### 1、注册中心7001

（1）出厂默认，自我保护机制是开启的

`eureka.server.enable-self-preservation=true`

（2）使用eureka.server.enable-self-preservation = false 可以禁用自我保护模式

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116183216.png)

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/
  server:
    #关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000

```

（3）效果

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116183418.png)

#### 2、生产者8001

##### 1、默认

（1）单位为秒(默认是30秒)

`eureka.instance.lease-renewal-interval-in-seconds=30`

（2）单位为秒(默认是90秒)

`eureka.instance.lease-expiration-duration-in-seconds=90`



##### 2、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116204552.png)



```yml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/spring_cloud?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: a1b2c3

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
  instance:
    instance-id: payment8001
    prefer-ip-address: true     #访问路径可以显示IP地址
    #心跳检测与续约时间
    #开发时设置小些，保证服务关闭后注册中心能即使剔除服务
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.kk.springclond.entities
```



##### 3、测试

（1）先启动7001，在启动8001

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116205055.png)

（2）再关闭8001，发现服务已经被删除了

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116205204.png)





## 7、Eureka停更

``https://github.com/Netflix/eureka/wiki`



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220116205552.png)



技术选型可以考虑 zookeeper、nacos(alibaba)