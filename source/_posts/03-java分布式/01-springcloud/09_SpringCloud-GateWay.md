---
title: SpringCloud-Gaywayl路由网关
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 阿里开源的新一代网关 Gayway ，比起 netflix 的 zuul性能优越很多！
tags:
  - springcloud
  - alibaba
  - 路由网关
  - gateway
categories:
  - 分布式技术栈
abbrlink: faa2de28
reprintPolicy: cc_by
date: 2022-02-02 09:22:13
coverImg:
img:
password:
---



## 一、概述简介



### 1、官网

#### 1、上一代zuul 1.X

https://github.com/Netflix/zuul/wiki

#### 2、Gateway

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

### 2、是什么

#### 1、概述

Gateway是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot 2和 Project Reactor等技术。
Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能， 例如：熔断、限流、重试等



#### 2、一句话

SpringCloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202021557805.png)



### 3、能干嘛

- 反向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控
- ......

### 4、网关在微服务的位置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202021601043.png)

### 5、GateWay优于Zuul的地方

#### 1、我们为什么选择Gateway？

##### 1、neflix不太靠谱，zuul2.0一直跳票，迟迟不发布



##### 2、SpringCloud Gateway具有如下特性

- 基于Spring Framework 5, Project Reactor 和 Spring Boot 2.0 进行构建；
- 动态路由：能够匹配任何请求属性；
- 可以对路由指定 Predicate（断言）和 Filter（过滤器）；
- 集成Hystrix的断路器功能；
- 集成 Spring Cloud 服务发现功能；
- 易于编写的 Predicate（断言）和 Filter（过滤器）；
- 请求限流功能；
- 支持路径重写。

##### 3、SpringCloud Gateway 与 Zuul的区别

- `Zuul 1.x`，是一个基于	`阻塞 I/ O `的 API Gateway
- `Zuul 1.x`   基于Servlet 2. 5使用阻塞架构它`不支持任何长连接`(如 WebSocket) Zuul 的设计模式和Nginx较像，每次 I/ O 操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Nginx 用C++ 实现，Zuul 用 Java 实现，而 JVM 本身会有第一次加载较慢的情况，使得Zuul 的性能相对较差。
- `Zuul 2.x`理念更先进，想基于`Netty非阻塞和支持长连接`，但SpringCloud目前还没有整合。 Zuul 2.x的性能较 Zuul 1.x 有较大提升。在性能方面，根据官方提供的基准测试， Spring Cloud Gateway 的 RPS（每秒请求数）是Zuul 的 1. 6 倍。
- Spring Cloud Gateway 建立 在 Spring Framework 5、 Project Reactor 和 Spring Boot 2 之上， 使用非阻塞 API。
- Spring Cloud Gateway 还 支持 WebSocket， 并且与Spring紧密集成拥有更好的开发体验

#### 2、Zuul1.x模型

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。



Servlet的生命周期?servlet由servlet container进行生命周期管理。

- container启动时构造servlet对象并调用servlet init()进行初始化；
- container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用service()。
- container关闭时调用servlet destory()销毁servlet；

#### 3、GateWay模型



WebFlux是什么

https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-new-framework





传统的Web框架，比如说：struts2，springmvc等都是基于Servlet API与Servlet容器基础之上运行的。
但是`在Servlet3.1之后有了异步非阻塞的支持`。而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程（Spring5必须让你使用java8）



## 二、三大核心概念

### 

### 1、Route(路由)

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

### 2、Predicate(断言)

参考的是Java8的java.util.function.Predicate
开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，`如果请求与断言相匹配则进行路由`

### 3、Filter(过滤)

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。



### 4、总体

web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。
predicate就是我们的匹配条件；而filter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202021638676.png)

## 三、Gateway工作流程

### 1、官网


客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。

Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，
在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。





### 2、核心逻辑

路由转发+执行过滤器链





## 四、入门案例



### 1、模块创建步骤

#### 1、建model

cloud-gateway-gateway9527

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

    <artifactId>cloud-gateway-gateway9527</artifactId>
    
    <dependencies>
        <!--gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--eureka-client-->
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
        <!--一般基础配置类-->
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



#### 3、yml

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka

```



#### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class GatewayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run (GatewayMain9527.class,args);
    }
}

```



### 2、网关映射配置

#### 1、yml配置

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
```



#### 2、配置说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202031106233.png)





#### 4、测试

1、启动 eureka集群，启动8001/8002，启动网关9527



添加网关前uri：http://localhost:8001/payment/get/1

添加网关后uri：http://localhost:9527/payment/get/1



### 3、YML配置说明

Gateway网关路由有两种配置方式：

#### 1、方式一：在配置文件yml中配置

既上面案例的方式



#### 1、方式二：代码中注入RouteLocator的Bean

```java
package com.kk.springcloud.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {
    /**
     * 配置了一个id为route-name的路由规则，
     * 当访问地址 http://localhost:9527/kk时会自动转发到地址：http://news.baidu.com/
     *
     * @param builder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes ( );
        routes.route ("path_route_mykk", r -> r.path ("/kk").uri ("http://www.baidu.com/")).build ( );
        return routes.build ( );
    }
    
    @Bean
    public RouteLocator customRouteLocator2(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes ( );
        routes.route ("path_route_mykk2", r -> r.path ("/kkz").uri ("http://www.weibo.com/kkz")).build ( );
        return routes.build ( );
    }
}
```





## 五、微服务名实现动态路由

### 1、概述

默认情况下Gateway会根据注册中心注册的服务列表，
以注册中心上微服务名为路径创建`动态路由进行转发，从而实现动态路由的功能`

### 2、启动

一个eureka7001 + 两个服务提供者8001/8002



### 4、yml

1、需要注意的是uri的协议为`lb`，表示启用Gateway的负载均衡功能。

2、`lb://serviceName`是spring cloud gateway在微服务中自动为我们创建的负载均衡uri

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



### 5、测试

url：http://localhost:9527/payment/lb

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202031206423.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202031208863.png)

## 六、Predicate(断言)的使用

### 1、启动看下日志

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202031215130.png)

### 2、Route Predicate Factories

Spring Cloud Gateway 创建 Route 对象时， 使用 RoutePredicateFactory 创建 Predicate 对象，Predicate 对象可以赋值给 Route。 Spring Cloud Gateway 包含许多内置的Route Predicate Factories。

所有这些谓词是都匹配HTTP请求的不同属性。多种谓词工厂可以组合，并通过逻辑 `and` 组合。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202031345887.png)

### 3、常用的Route Predicate

#### 1、After Route Predicate

必须要在配置断言的`时区的时间之后`对应的uri请求才能生效

```java
package com.kk.test;
import java.time.ZonedDateTime;

public class Test {
    public static void main(String[] args) {
        ZonedDateTime zbj = ZonedDateTime.now ( ); // 默认时区
        System.out.println (zbj);
//        ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York")); // 用指定时区获取当前时间
//        System.out.println(zny);
    }
}

```



```yml
            - After=2022-02-03T13:53:16.164+08:00[Asia/Shanghai]  # 断言，路径相匹配的进行路由
```

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由
            - After=2022-02-03T13:53:16.164+08:00[Asia/Shanghai]  # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

```



#### 2、Before Route Predicate

必须要在配置断言的`时区的时间之前`对应的uri请求才能生效

```yml
            - Before=2022-02-03T13:53:16.164+08:00[Asia/Shanghai]  # 断言，路径相匹配的进行路由
```



#### 3、Between Route Predicate

必须要在配置断言的`时区的时间范围内`对应的uri请求才能生效

```yml
            - Between=2022-02-03T13:53:16.164+08:00[Asia/Shanghai],2022-12-03T13:53:16.164+08:00[Asia/Shanghai]  # 断言，路径相匹配的进行路由
```



#### 4、Cookie Route Predicate

必须显示的指定携带的cookie信息，才能访问对应的uri



```yml
            - Cookie=username,mykk    # 断言，路径相匹配的进行路由
```



1、不带cookie的情况

`curl http://localhost:9527/payment/get/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032230261.png)

2、携带cookie的情况

`curl http://localhost:9527/payment/get/1   --cookie "username=mykk"`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032231053.png)

#### 5、Header Route Predicate

必须显示的指定携带的请求头Header 信息，才能访问对应的uri。

`两个参数：`一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

```yml
            - Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
```



1、不带Header的情况

`curl http://localhost:9527/payment/get/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032240793.png)



2、携带Header的情况

`curl http://localhost:9527/payment/get/1 -H "X-Request-Id:123"`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032242875.png)



#### 6、Host Route Predicate



必须显示的携带指定规则的主机地址 Host信息 ，才能访问对应的uri。

```yml
            - Host=**.mykk.com
```



访问测试

`curl http://localhost:9527/payment/get/1 -H "Host:www.mykk.com"`



`curl http://localhost:9527/payment/get/1 -H "Host:news.mykk.com"`



![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032311222.png)

#### 7、Method Route Predicate

必须显示的指定请求方式 ，才能访问对应的uri。

```yml
            - Method=GET   # 请求方式
```





#### 8、Path Route Predicate

路由到指定的路径



#### 9、Query Route Predicate

必须显示的携带指定规则的 请求参数 ，才能访问对应的uri。

说明：支持传入两个参数，一个是属性名，一个为属性值，属性值可以是正则表达式。



测试：

url：`curl http://localhost:9527/payment/get/1?username=111`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032317881.png)



## 七、Filter的使用

### 1、概述

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。

Spring Cloud Gateway 内置了多种路由过滤器，他们都由GatewayFilter的工厂类来产生



#### 1、生命周期

前置：pre

后置：post



#### 2、种类

单一：GatewayFilter  	

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#the-addrequestparameter-gatewayfilter-factory



31种之多

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032353348.png)



全局：GlobalFilter

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202032354731.png)

### 2、常用的GatewayFilter

AddRequestParameter



```yml
          filters:
            - AddRequestParameter=X-Request-Id,1024 #过滤器工厂会在匹配的请求头加上一对请求头，名称为X-Request-Id值为1024
```

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          filters:
            - AddRequestParameter=X-Request-Id,1024 #过滤器工厂会在匹配的请求头加上一对请求头，名称为X-Request-Id值为1024
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由
            - After=2022-02-03T13:53:16.164+08:00[Asia/Shanghai]  # 断言，路径相匹配的进行路由
#            - Cookie=username,mykk
#            - Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
#            - Host=**.mykk.com
#            - Method=GET   # 请求方式
            - Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由


eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



### 3、自定义过滤器

自定义全局GlobalFilter



#### 1、两个主要接口介绍

`implements GlobalFilter,Ordered`



#### 2、能干嘛

- 全局日志记录
- 统一网关鉴权
- ......



#### 3、代码

```java
package com.kk.springcloud.config.config;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;
import java.util.Date;

@Component
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    // 过滤逻辑
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("time:"+new Date ()+"\t 执行了自定义的全局过滤器: "+"MyLogGateWayFilter"+"hello");

        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            System.out.println("****用户名为null，无法登录");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    // 优先级，越少越大
    @Override
    public int getOrder() {
        return 0;
    }
}

```





测试：`curl http://localhost:9527/payment/get/1?uname=11`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202041235382.png)