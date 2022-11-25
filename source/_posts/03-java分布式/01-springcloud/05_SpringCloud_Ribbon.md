---
title: SpringCloud- Ribbon负载均衡服务
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: Ribbon负载均衡服务框架
tags:
  - netflix
  - springcloud
  - 负载均衡
  - ribbon
categories:
  - 分布式技术栈
abbrlink: 9091e07b
reprintPolicy: cc_by
date: 2022-01-17 22:42:11
coverImg:
img:
password:
---



## 1、概述

### 1、是什么

- Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。
- 简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

### 2、官网资料

#### 1、文档

https://github.com/Netflix/ribbon/wiki/Getting-Started

#### 2、Ribbon目前也进入维护模式

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205050.png)



未来替换方案：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205311.png)

### 3、能干吗

#### 1、LB（负载均衡）

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。
常见的负载均衡有软件Nginx，LVS，硬件 F5等。



总结：Ribbon = 负载均衡+RestTemplate调用

#### 2、区别（ribbon VS nginx）

 Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

#### 3、划分

1、集中式LB

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；

2、进程内LB

- 将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

- Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

## 2、Ribbon案例

### 1、架构说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205346.png)



Ribbon在工作时分成两步

- 第一步先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server.
- 第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。
  其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。



总结：Ribbon其实就是一个软负载均衡的客户端组件，
他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。

### 2、POM

#### 1、坐标

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```



#### 2、eureka-client 自带 ribbon

证明如下： 可以看到spring-cloud-starter-netflix-eureka-client 确实引入了Ribbon

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205633.png)

### 3、二说RestTemplate的使用

#### 1、官网

https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205714.png)



#### 2、getForObject方法/getForEntity（读）

- 返回对象为响应体中数据转化成的对象，基本上可以理解为Json

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205923.png)



- 返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等

  ![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220118205956.png)

  

#### 3、postForObject/postForEntity(写)

写一个单元测试用例，测试用例的内容是向指定的URL提交一个Post(帖子).

```java
   @Test
   void testSimple()  {
      // 请求地址
      String url = "http://jsonplaceholder.typicode.com/posts";

      // 要发送的数据对象
      PostDTO postDTO = new PostDTO();
      postDTO.setUserId(110);
      postDTO.setTitle("zimug 发布文章");
      postDTO.setBody("zimug 发布文章 测试内容");

      // 发送post请求，并输出结果
      PostDTO result = restTemplate.postForObject(url, postDTO, PostDTO.class);
      System.out.println(result);
   }
```



上面的所有的postForObject请求传参方法，postForEntity都可以使用，**使用方法上也几乎是一致的**，只是在返回结果接收的时候略有差别。使用`ResponseEntity<T> responseEntity`来接收响应结果。用responseEntity.getBody()获取响应体。响应体内容同postForObject方法返回结果一致。剩下的这些响应信息就是postForEntity比postForObject多出来的内容

- `HttpStatus statusCode = responseEntity.getStatusCode();  `获取整体的响应状态信息
- ` int statusCodeValue = responseEntity.getStatusCodeValue(); ` 获取响应码值
- `HttpHeaders headers = responseEntity.getHeaders(); `获取响应头等

```java
@Test
public void testEntityPoJo() {
   // 请求地址
   String url = "http://jsonplaceholder.typicode.com/posts";

   // 要发送的数据对象
   PostDTO postDTO = new PostDTO();
   postDTO.setUserId(110);
   postDTO.setTitle("zimug 发布文章");
   postDTO.setBody("zimug 发布文章 测试内容");

   // 发送post请求，并输出结果
   ResponseEntity<String> responseEntity
               = restTemplate.postForEntity(url, postDTO, String.class);
   String body = responseEntity.getBody(); // 获取响应体
   System.out.println("HTTP 响应body：" + postDTO.toString());

   //以下是postForEntity比postForObject多出来的内容
   HttpStatus statusCode = responseEntity.getStatusCode(); // 获取响应码
   int statusCodeValue = responseEntity.getStatusCodeValue(); // 获取响应码值
   HttpHeaders headers = responseEntity.getHeaders(); // 获取响应头

   System.out.println("HTTP 响应状态：" + statusCode);
   System.out.println("HTTP 响应状态码：" + statusCodeValue);
   System.out.println("HTTP Headers信息：" + headers);
}
```





#### 4、GET请求方法

```java
<T> T getForObject(String url, Class<T> responseType, Object... uriVariables);
 
<T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables);
 
<T> T getForObject(URI url, Class<T> responseType);
 
<T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables);
 
<T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables);
 
<T> ResponseEntity<T> getForEntity(URI var1, Class<T> responseType);
```



#### 5、POST请求方法

```java
 
<T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
 
<T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
 
<T> T postForObject(URI url, @Nullable Object request, Class<T> responseType);
 
<T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
 
<T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
 
<T> ResponseEntity<T> postForEntity(URI url, @Nullable Object request, Class<T> responseType);

```



## 3、Ribbon核心组件IRule

### 1、IRule：

根据特定算法中从服务列表中选取一个要访问的服务

- 轮询:  `com.netflix.loadbalancer.RoundRobinRule` 
- 随机:  `com.netflix.loadbalancer.RandomRule`
-  `com.netflix.loadbalancer.RetryRule` 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
- `WeightedResponseTimeRule` 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- `BestAvailableRule` 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- `AvailabilityFilteringRule` 先过滤掉故障实例，再选择并发较小的实例
- `ZoneAvoidanceRule` 默认规则,复合判断server所在区域的性能和server的可用性选择服务器



### 2、案例

#### 修改cloud-consumer-order80



#### 1、细节说明

官方文档明确给出了警告：
这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，
否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119201448.png)





#### 2、新建package 

com.kk.myrule

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119201717.png)

#### 3、新建MySelfRule规则类

```java
package com.kk.myrule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule() {
        return new RandomRule ( );//定义为随机
    }
}
```





#### 4、主启动类添加@RibbonClient

```java
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119202018.png)



#### 5、测试

启动：（1）eureka（单个或者集群都行），（2）生产者8001（单个或者集群都行），（3）消费者80

`http://localhost/consumer/payment/get/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119202717.png)

## 4、Ribbon负载均衡算法



### 1、原理



负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。

> List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");

> 如：   List [0] instances = 127.0.0.1:8002
> 　　　List [1] instances = 127.0.0.1:8001

> 8001+ 8002 组合成为集群，它们共计2台机器，集群总数为2， 按照轮询算法原理：

> 当总请求数为1时： 1 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001
> 当总请求数位2时： 2 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002
> 当总请求数位3时： 3 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001
> 当总请求数位4时： 4 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002
> 如此类推......



### 2、RoundRobinRule源码

- 这里不是死循环，而是自旋锁，compareAndSet（CAS操作，JUC中乐观锁的底层实现）

  ![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119203035.png)



### 3、手写



#### 1、8001和8002分别加入

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119233626.png)



```java
    @Value("${server.port}")
    private String serverPort;


    @GetMapping(value = "/payment/lb")
    public String getPaymentLB()
    {
        return serverPort;
    }
```



#### 2、去掉注解@LoadBalanced

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119203829.png)





#### 3、新建LoadBalancer接口

```java
package com.kk.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;
import java.util.List;

public interface LoadBalancer {
    ServiceInstance instances(List<ServiceInstance> serviceInstances);
}

```



#### 4、新建MyLB

```java
package com.kk.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

@Component
public class MyLB implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger (0);

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
        int index = getAndIncrement ( ) % serviceInstances.size ( );
        return serviceInstances.get (index);
    }

    public final int getAndIncrement() {
        int current;
        int next;
        do {
            current = this.atomicInteger.get ( );
            next = current >= 2147483647 ? 0 : current + 1;
        } while (!this.atomicInteger.compareAndSet (current, next));
        System.out.println ("*****next: " + next);
        return next;
    }
}
```



#### 5、OrderController

```java
    @Resource
    private LoadBalancer loadBalancer;

    @GetMapping("/consumer/payment/lb")
    public String getPaymentLB() {
        List<ServiceInstance> instances = discoveryClient.getInstances ("CLOUD-PAYMENT-SERVICE");

        if (instances == null || instances.size ( ) <= 0) {
            return null;
        }
        ServiceInstance serviceInstance = loadBalancer.instances (instances);
        URI uri = serviceInstance.getUri ( );

        return restTemplate.getForObject (uri + "/payment/lb", String.class);
    }
```



```java
package com.kk.springcloud.controller;

import com.kk.springcloud.lb.LoadBalancer;
import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import javax.annotation.Resource;
import java.net.URI;
import java.util.List;

@RestController
@Slf4j
public class OrderController {

    //public static final String PAYMENT_URL = "http://localhost:8001";
    public static final String PAYMENT_URL = "http://cloud-payment-service";

    //可以获取注册中心上的服务列表
    @Resource
    private DiscoveryClient discoveryClient;

    @Resource
    private LoadBalancer loadBalancer;

    @GetMapping("/consumer/payment/lb")
    public String getPaymentLB() {
        List<ServiceInstance> instances = discoveryClient.getInstances ("CLOUD-PAYMENT-SERVICE");

        if (instances == null || instances.size ( ) <= 0) {
            return null;
        }
        ServiceInstance serviceInstance = loadBalancer.instances (instances);
        URI uri = serviceInstance.getUri ( );

        return restTemplate.getForObject (uri + "/payment/lb", String.class);
    }

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment> create(Payment payment) {
        return restTemplate.postForObject (PAYMENT_URL + "/payment/create", payment, CommonResult.class);  //写操作
    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
        return restTemplate.getForObject (PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }
}

```



#### 6、测试

`http://localhost/consumer/payment/lb`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119234324.png)

#### 注意：8001和8002别名配置都要配好

否则自定义算法 获取的url会报错

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220119234130.png)