---
title: SpringCloud-Alibaba-Sentinel 实现熔断与限流
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 阿里开源的轻量级流量限制和熔断器，能解决雪崩，
tags:
  - springcloud-alibaba
  - 熔断器
  - sentinel
  - 限流
categories:
  - 分布式技术栈
abbrlink: 32724da3
reprintPolicy: cc_by
date: 2022-03-013 15:17:13
coverImg:
img:
password:
---



## 一、概述

### 1、官网

https://github.com/alibaba/Sentinel



https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

### 2、是什么



一句话解释，就是类似于 Hystrix



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312233839.png)

### 3、去哪下

https://github.com/alibaba/Sentinel/releases

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312233941.png)

### 4、能干嘛

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312234011.png)



**服务使用中的各种问题**

- 服务雪崩
- 服务降级
- 服务熔断
- 服务限流







## 二、安装控制台

### 1、sentinel组件由两部分构成

- 后台
- 前台8080

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220313211351.png)

### 2、安装步骤

#### 1、拉取镜像

```docker
docker pull bladex/sentinel-dashboard:1.7.0
```



#### 2、启动并创建容器

```docker
docker run --name sentinel -d  -p 8858:8858  bladex/sentinel-dashboard:1.7.0
```



### 3、访问

`ip:8858`



登录账号密码均为`sentinel`



## 三、初始化工程

### 1、启动准备工作

启动 sentinel （8858），nacos（8848）

### 2、model

#### 1、建model

cloudalibaba-sentinel-service8401

#### 2、pom

```xml
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
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

    <artifactId>cloudalibaba-sentinel-service8401</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件+actuator -->
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
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.6.3</version>
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
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        #Nacos服务注册中心地址
        server-addr: 101.34.180.133:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
#        dashboard: 101.34.180.133:8858
#        dashboard: 106.52.23.202:8080
        # 项目和 sentinel 不在同一台机器无法查看实时监控
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719
        # 本地机器ip(docker容器必须加上)
#        client-ip: 169.254.135.77

management:
  endpoints:
    web:
      exposure:
        include: '*'

```



#### 4、主启动

```java
package com.kk.springcloud;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class SpringCloudSentinelMain8401 {
    public static void main(String[] args) {
        SpringApplication.run (SpringCloudSentinelMain8401.class,args);
    }
}

```



#### 5、业务类

```java
package com.kk.springcloud.controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA()
    {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB()
    {
        return "------testB";
    }
}

```





### 3、操作 sentinel控制台

#### 1、刚进来

空空如也，啥都没有

### ![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220313224347.png)





#### 2、懒加载说明



- Sentinel采用的懒加载说明
- 需要执行一次
  - http://localhost:8401/testA
  - http://localhost:8401/testB
- ![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220314002216.png)
- 这边需要注意一点：如果你的`实时监控`没有数据，可能是因为 sentinel 和项目不再`同一个 机器`或者 `sentinel访问不到` 项目就监控不到了



## 四、流控制规则

### 1、基本介绍

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220314222312.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220314222329.png)

### 2、流控模式

#### 1、直接(默认)



表示：1秒钟内查询1次就是OK，若超过次数1，就直接-快速失败，报默认错误



##### 1、配置内容

![1647528650781](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1647528650781.png)



##### 2、测试效果

访问：http://localhost:8401/testA

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318194405.png)



##### 3、结论思考

直接调用默认报错信息，技术方面OK，

但是否应该有我们自己的后续处理，类似有个fallback的兜底方法



#### 2、关联

##### 1、是什么

- 当关联的资源达到阈值时，就限流自己
- 当与A关联的资源B达到阀值后，就限流A自己
- B惹事，A挂了



##### 2、配置A

当关联资源 /testB 的 qps 阀值超过1时，就限流 /testA 的Rest访问地址，当关联资源到阈值后限制配置好的资源名

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318194759.png)





##### 3、postman模拟并发密集访问testB



创建一个集合并发测试文件夹

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318195358.png)



将测试的url放进文件夹

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318200022.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318200053.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318200149.png)



##### 4、测试

启动并发对 /testB run，再次过程中访问  /testB  发现出错

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220318200544.png)



#### 3、链路

- 多个请求调用了同一个微服务



### 2X（流控模式总结）



- 直接：对当前资源限流
- 关联：高优先级资源触发阈值，对低优先级资源限流。
- 链路：阈值统计时，只统计从指定资源进入当前资源的请求，是对请求来源的限流

### 3、流控效果

#### 1、快速失败

达到峰值，直接失败，抛出异常（Blocked by Sentinel (flow limiting)）



源码（修改扩展位置）：com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController



#### 2、WarmUp（预热）



##### 1、公式

阈值除以coldFactor(默认值为3),经过预热时长后才会达到阈值



##### 2、官网

- 默认coldFactor为3，即请求 QPS 从 threshold / 3 开始，经预热时长逐渐升至设定的 QPS 阈值。
- 限流，冷启动
  - https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8



##### 3、源码

com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319093823.png)



##### 4、WarmUp配置

- 默认 coldFactor 为 3，即请求QPS从(threshold / 3) 开始，经多少预热时长才逐渐升至设定的 QPS 阈值。
- 案例，阀值为10+预热时长设置5秒。
  系统初始化的阀值为10 / 3 约等于3,即阀值刚开始为3；然后过了5秒后阀值才慢慢升高恢复到10

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319094044.png)



##### 5、点击测试

多次点击http://localhost:8401/testB

刚开始不行，后续慢慢OK



##### 6、应用场景

如：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来，慢慢的把阀值增长到设置的阀值。



#### 3、排队等待

##### 1、说明

匀速排队，阈值必须设置为QPS



##### 2、官网

https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319094259.png)

##### 3、源码

com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController



##### 4、配置



- 匀速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效。
- 设置含义： /testB  每秒1次请求，超过的话就排队等待，等待的超时时间为20000毫秒。

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319094533.png)



##### 5、测试

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319095241.png)



## 五、降级规则

### 1、官网



https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7



### 2、基本介绍

#### 1、说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319095912.png)



Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，
让请求快速失败，避免影响到其它的资源而导致级联错误。

当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）

#### 2、hystrix比较



Sentinel的断路器是`没有半开`状态的



半开的状态系统自动去检测是否请求有异常，
没有异常就`关闭断路器`恢复使用，
有异常则`继续打开`断路器不可用。具体可以参考Hystrix



以下是hystrix断路器结构图

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319100840.png)





### 3、降级策略实战

#### 1、RT

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319225244.png)



##### 1、代码

```java
    @GetMapping("/testD")
    public String testD() {
        //暂停几秒钟线程
        try {
            TimeUnit.SECONDS.sleep (1);
        } catch (InterruptedException e) {
            e.printStackTrace ( );
        }
        log.info ("testD 测试RT");
        return "------testD";
    }
```



##### 2、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220319225631.png)



##### 3、jmeter压测

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321223309.png)



##### 4、结果

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321223653.png)

#### 2、异常比例



1、是什么

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321223744.png)



##### 2、代码

```java
    @GetMapping("/testD")
    public String testD() {
        log.info ("testD 测试异常比例");
        int age = 10 / 0;
        return "------testD";
    }
```



##### 3、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321223956.png)



##### 4、jmeter

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321224035.png)





##### 5、结论

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321224157.png)





#### 3、异常数

##### 1、是什么

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321224235.png)



##### 2、需知

异常数是按照分钟统计的



##### 3、代码

```java
    @GetMapping("/testE")
    public String testE() {
        log.info ("testE 测试异常比例");
        int age = 10 / 0;
        return "------testE 测试异常比例";
    }
```



##### 4、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321224436.png)

##### 5、jmeter

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321224523.png)



### 



### 



## 六、热点key限流

### 1、基本介绍

#### 1、是什么

**何为热点**

热点即经常访问的数据，很多时候我们希望统计或者限制某个热点数据中访问频次最高的TopN数据，并对其访问进行限流或者其它操作

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321225130.png)

#### 2、官网

https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81



#### 3、兜底方法

sentinel系统默认的提示：Blocked by Sentinel (flow limiting)

可以指定，自定义兜底方法



 从HystrixCommand 到@SentinelResource





### 2、案例

#### 1、代码



```java
    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey", blockHandler = "dealHandler_testHotKey")
    public String testHotKey(@RequestParam(value = "p1", required = false) String p1,
                             @RequestParam(value = "p2", required = false) String p2) {
        return "------testHotKey";
    }

    public String dealHandler_testHotKey(String p1, String p2, BlockException exception) {
        return "-----dealHandler_testHotKey";
    }
```





#### 2、配置说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322212525.png)



**限流模式只支持QPS模式，固定写死了。（这才叫热点）
`@SentinelResource`注解的方法参数索引，0代表第一个参数，1代表第二个参数，以此类推
单机阀值以及统计窗口时长表示在此窗口时间超过阀值就限流。**
`上面的抓图就是第一个参数有值的话，1秒的QPS为1，超过就限流，限流后调用` dealHandler_testHotKey支持方法。



#### 3、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322213124.png)



`@SentinelResource(value = "testHotKey",blockHandler = "dealHandler_testHotKey")`



方法testHotKey里面第一个参数只要QPS超过每秒1次，马上降级处理



#### 4、测试

http://localhost:8401/testHotKey?p1=abc

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322213413.png)

http://localhost:8401/testHotKey?p1=abc&p2=33

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322213432.png)

http://localhost:8401/testHotKey?p2=33

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322213506.png)



### 3、参数高级选项

#### 1、作用

上述案例，第一个参数p1，当QPS超过1秒1次点击后马上被限流

若是我们有一个需求，p1特例为5 ，QPS 阈值为 200就可以通过 这个实现



#### 2、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322213926.png)

#### 3、测试



http://localhost:8401/testHotKey?p1=5

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322214044.png)



http://localhost:8401/testHotKey?p1=3

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220322214058.png)

#### 4、结论

当p1等于5的时候，阈值变为 200



当p1不等于5的时候，阈值为平常的 1



热点参数的注意点，参数必须是基本类型或者String







## 七、@SentinelResource

### 1、按资源名称限流+后续处理



#### 1、代码

1、代码

```java
package com.kk.springcloud.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RateLimitController
{
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource()
    {
        return new CommonResult (200,"按资源名称限流测试OK",new Payment (2020L,"serial001"));
    }
    public CommonResult handleException(BlockException exception)
    {
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
    }
}

```

2、pom

```xml
        <dependency>
            <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
```





#### 2、配置流控规则

##### 1、步骤

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323102223.png)

##### 2、图形配置和代码关系

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323102301.png)

##### 3、配置说明

表示1秒钟内查询次数大于1，就跑到我们自定义的处流，限流



#### 3、测试

1秒钟点击1下，OK

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323103703.png)



超过上述，疯狂点击，返回了自己定义的限流处理信息，限流发生

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323103648.png)







### 2、按照Url地址限流+后续处理



#### 1、作用

通过访问的URL来限流，会返回Sentinel自带默认的限流处理信息



#### 2、代码

```java
    @GetMapping("/rateLimit/byUrl")
    @SentinelResource(value = "byUrl")
    public CommonResult byUrl() {
        return new CommonResult (200, "按url限流测试OK", new Payment (2020L, "serial002"));
    }
```



#### 3、访问（1次）

为了刷新实时配置，线上就没必要这个操作了

http://localhost:8401//rateLimit/byUrl

#### 4、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323105553.png)

#### 5、访问（狂点）

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323105616.png)

### 3、兜底方案面临的问题



1  系统默认的，没有体现我们自己的业务要求。

2  依照现有条件，我们自定义的处理方法又和业务代码耦合在一块，不直观。

3  每个业务方法都添加一个兜底的，那代码膨胀加剧。

4  全局统一的处理方法没有体现



### 4、用户自定义限流处理逻辑

#### 1、自定义限流处理类



创建CustomerBlockHandler类用于自定义限流处理逻辑

```java
package com.kk.springcloud.handler;

import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.kk.springcloud.entities.CommonResult;

public class CustomerBlockHandler {
    public static CommonResult handleException(BlockException exception) {
        return new CommonResult (2020, "自定义的限流处理信息......CustomerBlockHandler");
    }
}

```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323115746.png)





#### 2、使用

```java
    /**
     * 自定义通用的限流处理逻辑，
     blockHandlerClass = CustomerBlockHandler.class
     blockHandler = handleException
     上述配置：找CustomerBlockHandler类里的handleException2方法进行兜底处理
     */
    /**
     * 自定义通用的限流处理逻辑
     */
    @GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
            blockHandlerClass = CustomerBlockHandler.class, blockHandler = "handleException")
    public CommonResult customerBlockHandler() {
        return new CommonResult (200, "按客户自定义限流处理逻辑");
    }
```





#### 3、配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323120859.png)





#### 4、配置说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323121020.png)



#### 5、测试

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323121049.png)



### 5、更多注解属性说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323121421.png)

## 八、服务熔断功能

### 1、整合

sentinel整合ribbon+openFeign+fallback

### 2、Ribbon

#### 1、生产者

##### 1、建model



cloudalibaba-provider-payment9003

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

    <artifactId>cloudalibaba-provider-payment9003</artifactId>
    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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



##### 3、yml

```yml
server:
  port: 9003

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```



##### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run (PaymentMain9003.class,args);
    }
}
```



##### 5、业务类

```java
package com.kk.springcloud.controller;

import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    public static HashMap<Long, Payment> hashMap = new HashMap<> ( );

    static {
        hashMap.put (1L, new Payment (1L, "28a8c1e3bc2742d8848569891fb42181"));
        hashMap.put (2L, new Payment (2L, "bba8c1e3bc2742d8848569891ac32182"));
        hashMap.put (3L, new Payment (3L, "6ua8c1e3bc2742d8848569891xt92183"));
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
        Payment payment = hashMap.get (id);
        CommonResult<Payment> result = new CommonResult (200, "from mysql,serverPort:  " + serverPort, payment);
        return result;
    }
}
```



##### 6、启动

同一个服务，启动9003，9004



`-Dserver.port=9004`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324000149.png)



##### 7、测试

http://localhost:9003/paymentSQL/1

http://localhost:9004/paymentSQL/1



#### 2、消费者

##### 1、model

新建cloudalibaba-consumer-nacos-order84

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

    <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>
    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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



##### 3、yml

```yml
server:
  port: 84

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
 
```



##### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run (OrderNacosMain84.class,args);
    }
}
```

##### 5、配置类

1、rabbion 负载配置

```java
package com.kk.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate ( );
    }
}
```







##### 6、业务类

1、只配置fallback（本例sentinel无配置）

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324205223.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324210258.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324210320.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324210332.png)

2、只配置blockHandler

```java
package com.kk.springcloud.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {
    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback", blockHandler = "blockHandler") //blockHandler负责在sentinel里面配置的降级限流
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject (SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
        if (id == 4) {
            throw new IllegalArgumentException ("非法参数异常....");
        } else if (result.getData ( ) == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录");
        }
        return result;
    }

    // 兜底
    public CommonResult handlerFallback(@PathVariable Long id, Throwable e) {
        Payment payment = new Payment (id, "null");
        return new CommonResult<> (444, "兜底异常handlerFallback,exception内容  " + e.getMessage ( ), payment);
    }

    // 降级
    public CommonResult blockHandler(@PathVariable Long id, BlockException blockException) {
        Payment payment = new Payment (id, "null");
        return new CommonResult<> (445, "blockHandler-sentinel限流,无此流水: blockException  " + blockException.getMessage ( ), payment);
    }

}
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211100.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211322.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211510.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211427.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211443.png)

3、fallback和blockHandler都配置

```java
package com.kk.springcloud.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class CircleBreakerController {
    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback", fallback = "handlerFallback", blockHandler = "blockHandler")
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject (SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
        if (id == 4) {
            throw new IllegalArgumentException ("非法参数异常....");
        } else if (result.getData ( ) == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录");
        }
        return result;
    }

    // 兜底
    public CommonResult handlerFallback(@PathVariable Long id, Throwable e) {
        Payment payment = new Payment (id, "null");
        return new CommonResult<> (444, "fallback,无此流水,exception  " + e.getMessage ( ), payment);
    }

    // 降级
    public CommonResult blockHandler(@PathVariable Long id, BlockException blockException) {
        Payment payment = new Payment (id, "null");
        return new CommonResult<> (445, "blockHandler-sentinel限流,无此流水: blockException  " + blockException.getMessage ( ), payment);
    }

}
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211706.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324211811.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324212202.png)



**降级优先于限流：若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。**



4、属性忽略

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324212355.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324212430.png)



**直接抛到前台对用户体验不好，细节注意**



### 3、Feign



#### 1、model

修改84模块

#### 2、pom

```xml
    <!--SpringCloud openfeign -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
```
![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324214128.png)

#### 3、yml

```yml
# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324214015.png)

#### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run (OrderNacosMain84.class,args);
    }
}
```

#### 5、业务类

##### 1、远程调服务接口

```java
package com.kk.springcloud.serivice;

import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * @auther mykk
 * @create 2022年3月24日 21:45:49
 * 使用 fallback 方式是无法获取异常信息的，
 * 如果想要获取异常信息，可以使用 fallbackFactory参数
 */
@FeignClient(value = "nacos-payment-provider", fallback = PaymentFallbackService.class)//调用中关闭9003服务提供者
public interface PaymentService {
    @GetMapping(value = "/paymentSQL/{id}")
    CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}
```



##### 2、兜底实现

```java
package com.kk.springcloud.serivice;

import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import org.springframework.stereotype.Component;

@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id) {
        return new CommonResult<> (444, "服务降级返回,没有该流水信息", new Payment (id, "errorSerial......"));
    }
}

```





##### 3、Controller

```java
//==================OpenFeign
@Resource
private PaymentService paymentService;

@GetMapping(value = "/consumer/openfeign/{id}")
public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
    if (id == 4) {
        throw new RuntimeException ("没有该id");
    }
    return paymentService.paymentSQL (id);
}
```





#### 6、测试

测试84调用9003，此时`故意关闭9003/9004微服务提供者，看84消费侧自动降级`，不会被耗死



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324221753.png)

### 4、熔断框架比较

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220323170940.png)



## 九、持久化规则

### 1、是什么

一旦我们重启应用，sentinel规则将消失，生产环境需要将配置规则进行持久化

### 2、怎么玩

将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台
的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上sentinel上的流控规则持续有效



### 3、实战

#### 1、model

修改cloudalibaba-sentinel-service8401

#### 2、pom

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324223255.png)

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

    <artifactId>cloudalibaba-sentinel-service8401</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba sentinel-datasource-nacos -->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <dependency>
            <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件+actuator -->
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
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.6.3</version>
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

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324223733.png)

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        #Nacos服务注册中心地址
        server-addr: 101.34.180.133:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        #        dashboard: 101.34.180.133:8858
        #        dashboard: 106.52.23.202:8080
        # 项目和 sentinel 不在同一台机器无法查看实时监控
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719
      datasource:
        ds1:
          nacos:
            server-addr: 101.34.180.133:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
        # 本地机器ip(docker容器必须加上)
#        client-ip: 169.254.135.77

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持

management:
  endpoints:
    web:
      exposure:
        include: '*'

```

#### 4、添加Nacos业务规则配置

```json
[
    {
        "resource": "/rateLimit/byUrl",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324224003.png)

#### 5、启动

**启动8401后刷新sentinel发现业务规则有了**

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324224338.png)





#### 6、测试（访问）

快速访问测试接口

http://localhost:8401/rateLimit/byUrl

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220324224436.png)





## 十二、参考文档 ↓



https://www.cnblogs.com/linjiqin/p/15369091.html

https://www.jianshu.com/p/373eb512ec48

https://m.imooc.com/article/details?article_id=289384

https://www.cnblogs.com/yunqing/p/11406225.html