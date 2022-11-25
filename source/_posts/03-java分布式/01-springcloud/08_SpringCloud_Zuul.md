---
title: SpringCloud-Zuul路由网关
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: SpringCloud-Zuul网关
tags:
  - springcloud
  - netflix
  - 路由网关
  - zuul
categories:
  - 分布式技术栈
abbrlink: d633875f
reprintPolicy: cc_by
date: 2022-01-25 21:38:21
coverImg:
img:
password:
---



### 1、概述



### 1、官网

https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/#router-and-filter-zuul

### 2、是什么

Zuul是一种提供动态路由、监视、弹性、安全性等功能的边缘服务。

Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器。

API网关为微服务架构中的服务提供了统一的访问入口，客户端通过API网关访问相关服务。API网关的定义类似于设计模式中的门面模式，它相当于整个微服务架构中的门面，所有客户端的访问都通过它来进行路由及过滤。它实现了请求路由、负载均衡、校验过滤、服务容错、服务聚合等功能。
         
Zuul包含了如下最主要的功能：
代理+路由+过滤三大功能

### 3、能干嘛



#### 1、路由



#### 2、过滤



#### 3、负载均衡



#### 4、灰度发布（金丝雀发布）


起源是，矿井工人发现，金丝雀对瓦斯气体很敏感，矿工会在下井之前，先放一只金丝雀到井中，如果金丝雀不叫了，就代表瓦斯浓度高。

在灰度发布开始后，先启动一个新版本应用，但是并不直接将流量切过来，而是测试人员对新版本进行线上测试，启动的这个新版本应用，就是我们的金丝雀。如果没有问题，那么可以将少量的用户流量导入到新版本上，然后再对新版本做运行状态观察，收集各种运行时数据，如果此时对新旧版本做各种数据对比，就是所谓的A/B测试。新版本没什么问题，那么逐步扩大范围、流量，把所有用户都迁移到新版本上面来。

## 2、路由基本配置

路由功能负责将外部请求转发到具体的服务实例上去，是实现统一访问入口的基础

### 1、建Model

cloud-zuul-gateway9527



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

    <artifactId>cloud-zuul-gateway9527</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
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



### 3、yml

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-zuul-gateway

eureka:
  client:
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true

```



### 4、hosts修改(本地环境)



因为是本地环境，服务器，域名等资源有限



添加配置项：`C:\Windows\System32\drivers\etc`

```
127.0.0.1 myzuul.com
```



### 5、主启动

注意：`@EnableZuulProxy`

```java
package com.kk.springcloud;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class ZuulMain9527 {
    public static void main(String[] args) {
        SpringApplication.run (ZuulMain9527.class,args);
    }
}

```



### 6、启动顺序

1、eureka集群

2、8006生产者

3、9527网关



### 7、测试

#### 1、不用路由

`http://localhost:8001/payment/consul`



controller

```java
    @GetMapping("/payment/consul")
    public String paymentInfo() {
        return "springcloud with consul: " + serverPort + "\t\t" + UUID.randomUUID ( ).toString ( );
    }
```



#### 2、路由

（1）`zuul映射配置`+`注册中心注册后对外暴露的服务名称`+rest调用地址

（2）url：

http://myzuul.com:9527/cloud-payment-service/payment/consul

![](C:\Users\my_kk\Documents\Tencent Files\763856958\FileRecv\_posts\03javafenbushi\01springcloud\20220201141019.png)

## 3、路由访问映射规则

### 1、名称代理

#### 1、yml详解

```yml
zuul:
  routes: # 路由映射配置
    mypayment.path: /mypayment/**                 #IE地址栏输入的路径
    mypayment.serviceId: cloud-payment-service   # 指定服务端的名称
```



```yml
server:
  port: 9527

spring:
  application:
    name: cloud-zuul-gateway
    
eureka:
  client:
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true

zuul:
  routes: # 路由映射配置
    mypayment.path: /mypayment/**                 #IE地址栏输入的路径
    mypayment.serviceId: cloud-payment-service   # 指定服务端的名称
```



#### 2、测试

##### 1、路由访问：OK

http://myzuul.com:9527/mypayment/payment/consul

##### 2、原路径访问：OK

http://myzuul.com:9527/cloud-payment-service/payment/consul



### 2、忽略原有真实`服务名`

#### 1、yml配置

```yml
zuul:
  ignored-services: cloud-payment-service   #忽略服务名
```





#### 2、测试

1、使用服务名访问（失败）：http://myzuul.com:9527/cloud-payment-service/payment/consul

![](C:\Users\my_kk\Documents\Tencent Files\763856958\FileRecv\_posts\03javafenbushi\01springcloud\20220201165024.png)



2、映射访问：依旧可以！



##### 五角星：批量忽略

```yml
zuul: 
  ignored-services: "*"
```



### 3、路由转发和负载均衡功能

#### 1、生产者：SMS短信模块(8008)

##### 1、建model

cloud-provider-sms8008

##### 2、pom

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

    <artifactId>cloud-provider-sms8008</artifactId>

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



##### 3、yml

```yml
server:
  port: 8008

###服务名称(服务注册到eureka名称)
spring:
  application:
    name: cloud-provider-sms

eureka:
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      #defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
      #defaultZone: http://eureka7001.com:7001/eureka   # eureka集群加@老本版
 
 

```



##### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class SMSMain8008 {
    public static void main(String[] args) {
        SpringApplication.run (SMSMain8008.class,args);
    }
}

```





##### 5、业务类

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SMSController
{
    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/sms")
    public String sms()
    {
        return "sms provider service: "+"\t"+serverPort;
    }
}
```



##### 6、启动服务

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220201174945.png)



#### 2、网关：zuul（9527）

##### 1、yml

```yml
zuul:
  #  ignored-services: cloud-payment-service   #忽略服务名
  routes: # 路由映射配置
    mysms.path: /mysms/**                        # IE地址栏输入的路径
    mysms.serviceId: cloud-provider-sms       # 指定服务端的名称
```



```yml
server:
  port: 9527

spring:
  application:
    name: cloud-zuul-gateway

eureka:
  client:
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true

zuul:
  #  ignored-services: cloud-payment-service   #忽略服务名
  routes: # 路由映射配置
    mysms.path: /mysms/**                        # IE地址栏输入的路径
    mysms.serviceId: cloud-provider-sms       # 指定服务端的名称
    mypayment.path: /mypayment/**                # IE地址栏输入的路径
    mypayment.serviceId: cloud-payment-service   # 指定服务端的名称

```



##### 2、说明

由于Zuul自动集成了Ribbon和Hystrix，所以Zuul天生就有负载均衡和服务容错能力



##### 3、测试负载效果

url：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220201221800.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220201221821.png)



### 4、设置统一公共前缀

#### yml配置



`````yml
zuul: 
  prefix: /mykk
`````

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-zuul-gateway

eureka:
  client:
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true

zuul:
  prefix: /mykk
  #  ignored-services: cloud-payment-service   #忽略服务名
  routes: # 路由映射配置
    mysms.path: /mysms/**                        # IE地址栏输入的路径
    mysms.serviceId: cloud-provider-sms       # 指定服务端的名称
    mypayment.path: /mypayment/**                # IE地址栏输入的路径
    mypayment.serviceId: cloud-payment-service   # 指定服务端的名称

```



#### 测试url

（1）http://myzuul.com:9527/mykk/mypayment/payment/consul

（2）http://myzuul.com:9527/mykk/mysms/sms



## 4、查看路由信息

### 1、POM

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```



### 2、yml

```yml
# 开启查看路由的端点
management:
  endpoints:
    web:
      exposure:
        include: 'routes' 
```



### 3、查看路由详细信息

url：http://localhost:9527/actuator/routes

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202012239269.png)

## 5、过滤器

### 1、功能

过滤功能负责对请求过程进行额外的处理，是请求校验过滤及服务聚合的基础。



### 2、过滤器的生命周期

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202012253101.png)

### 3、ZuulFilter

#### 1、过滤类型

- pre：在请求被路由到目标服务前执行，比如权限校验、打印日志等功能；
- routing：在请求被路由到目标服务时执行
- post：在请求被路由到目标服务后执行，比如给目标服务的响应添加头信息，收集统计数据等功能；
- error：请求在其他阶段发生错误时执行。



#### 2、过滤顺序

数字小的先执行



#### 3、过滤是否开启

shouldFilter方法为true走



#### 4、执行逻辑

自己的业务逻辑

### 4、案例Case

#### 1、说明

前置过滤器，用于在请求路由到目标服务前打印请求日志

#### 2、自定义过滤器

过滤器代码：

```java
package com.kk.springcloud.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import javax.servlet.http.HttpServletRequest;
import java.util.Date;

@Component
@Slf4j
public class PreLogFilter extends ZuulFilter {

    /**
     * 定义过滤器的类型
     * pre:在请求被路由之前执行
     * route:在路由请求的时候执行
     * post:请求路由以后执行
     * error:处理请求时发生错误的时候执行
     *
     * @return 过滤器的类型
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 过滤器执行的顺序，配置多个有顺序的过滤
     * 执行顺序从小到大
     *
     * @return 执行顺序
     */
    @Override
    public int filterOrder() {
        // 优先级为0，数字越大，优先级越低 
        return 1;
    }

    /**
     * 是否开启过滤器
     * true:开启
     * false:禁用
     *
     * @return 是否开启过滤器
     */
    @Override
    public boolean shouldFilter() {
        // 是否开启
        return true;
    }

    /**
     * 过滤器的业务实现
     *
     * @return null 没有意义
     * @throws ZuulException 异常信息
     */
    @Override
    public Object run() throws ZuulException {
        // 业务逻辑代码
        RequestContext requestContext = RequestContext.getCurrentContext ( );
        HttpServletRequest request = requestContext.getRequest ( );
        String host = request.getRemoteHost ( );
        String method = request.getMethod ( );
        String uri = request.getRequestURI ( );
        log.info("=====> Remote host:{},method:{},uri:{}", host, method, uri);
        System.out.println ("********" + new Date ( ).getTime ( ));
        return null;
    }
}

```

#### 3、测试

(1)url：http://myzuul.com:9527/mykk/mysms/sms

(2)在调用8008之前会打印日志



#### 4、yml 配置开关

##### ★这里需要特别注意：开启这里之后，per配置失效，不清楚为什么，博主搞了很久尝试才发现是这个问题，建议使用硬编码，在java上配置开关

```yml
zuul:
  prefix: /mykk
  #  ignored-services: cloud-payment-service   #忽略服务名
  routes: # 路由映射配置
    mysms.path: /mysms/**                        # IE地址栏输入的路径
    mysms.serviceId: cloud-provider-sms       # 指定服务端的名称
    mypayment.path: /mypayment/**                # IE地址栏输入的路径
    mypayment.serviceId: cloud-payment-service   # 指定服务端的名称
    
    #yml配置开关
#  PreLogFilter:
#    pre:
#      disable: true
```





## 参考文章链接

### 1、限制IP过滤博文

https://www.jianshu.com/p/20d77ca5cfbc