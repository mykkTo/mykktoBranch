---
title: SpringCloud-Config分布式配置
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务，所以一套集中式的、动态的配置管理设施是必不可少的。
tags:
  - springcloud
  - 配置中心
  - config
categories:
  - 分布式技术栈
abbrlink: fc38d8b7
reprintPolicy: cc_by
date: 2022-02-04 09:33:12
coverImg:
img:
password:
---



## 1、概述

### 1、官网

https://docs.spring.io/spring-cloud-config/docs/2.2.8.RELEASE/reference/html/

### 2、分布式系统面临的---配置问题

   微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

​    SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个配置文件的管理。

### 3、是什么

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202041444024.png)

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为`各个不同微服务`应用的所有环境提供了一个`中心化的外部配置`。

### 4、怎么玩

SpringCloud Config分为`服务端`和`客户端`两部分。

服务端也称为`分布式配置中心，它是一个独立的微服务应用`，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

### 5、能干嘛

- 集中管理配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
- 将配置信息以REST接口的形式暴露，post、curl访问刷新均可......

### 6、与GitHub整合配置

由于SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件)，
但最推荐的还是Git，而且使用的是http/https访问的形式

### 

## 2、Config服务端配置

### 1、创建github仓库



#### 1、新建仓库

用你自己的账号在GitHub/gitee 上新建一个名为springcloud-config的新Repository

#### 2、获得刚新建的git地址

`https://gitee.com/TK_LIMR/spring-cloud-config.git`

#### 3、本地硬盘目录上新建git仓库并clone

`git clone https://gitee.com/TK_LIMR/spring-cloud-config.git`



#### 4、编辑application.yml环境

```yml
# 不同的开发环境，不同的微服务名字
spring:
  profiles:
    active:
    - dev
---
spring:
  profiles: dev     #开发环境
  application: 
    name: microservicecloud-config-kk-dev
config:
  info: version1
---
spring:
  profiles: test   #测试环境
  application: 
    name: microservicecloud-config-kk-test
#  请保存为UTF-8格式
```



#### 5、上传到gitee上

（1）`git add .`

（2）`git commit -m "init yml"`

（3）`git push origin master`



![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202051328243.png)

### 2、项目搭建

#### 1、建model

新建Module模块cloud-config-center-3344

#### 2、pom

```xml
	 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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

    <artifactId>cloud-config-center-3344</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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



#### 3、yml

```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/TK_LIMR/spring-cloud-config.git #GitHub上面的git仓库名字
          ####权限登录(这里填写自己的)
          force-pull: true
          username: xxxxxxxx@163.com
          password: xxxxxxxx
          ####搜索目录
          search-paths:
            - springcloud-config
      ####读取分支
      label: master

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```



#### 4、主启动

`@EnableConfigServer`

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run (ConfigCenterMain3344.class,args);
    }
}

```



#### 5、本地hosts配置

`127.0.0.1  config-3344.com`



#### 6、启动测试



生产：`http://config-3344.com:3344/master/config-prod.yml`

开发：`http://config-3344.com:3344/master/config-dev.yml`

测试：`http://config-3344.com:3344/master/config-test.yml`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202052133730.png)

### 3、配置读取规则

#### 1、官网

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202052136337.png)



#### 2、/{label}/{application}-{profile}.yml

参数说明：`/分支/服务名/环境`

##### 1、master分支

`http://config-3344.com:3344/master/config-dev.yml`

`http://config-3344.com:3344/master/config-test.yml`

`http://config-3344.com:3344/master/config-prod.yml`

##### 2、dev分支

`http://config-3344.com:3344/dev/config-dev.yml`

`http://config-3344.com:3344/dev/config-test.yml`

`http://config-3344.com:3344/dev/config-prod.yml`

#### 3、/{application}-{profile}.yml

参数说明：`/服务名/环境`

`http://config-3344.com:3344/config-dev.yml`

`http://config-3344.com:3344/config-test.yml`

`http://config-3344.com:3344/config-prod.yml`

`http://config-3344.com:3344/config-xxxx.yml(不存在的配置)`

#### 4、/{application}/{profile}[/{label}]

参数说明：`/服务名/环境/分支`

`http://config-3344.com:3344/config/dev/master`

`http://config-3344.com:3344/config/test/master`

`http://config-3344.com:3344/config/test/dev`

#### 5、参数说明

/{label}-{name}-{profiles}.yml

label：分支(branch)
name ：服务名
profiles：环境(dev/test/prod)



## 3、Config客户端配置

### 1、建model和测试

#### 1、建model

新建cloud-config-client-3355

#### 2、pom

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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

    <artifactId>cloud-config-client-3355</artifactId>
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

applicaiton.yml 是用户级的资源配置项
bootstrap.yml   是系统级的，优先级更加高

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
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run (ConfigClientMain3355.class,args);
    }
}

```



#### 5、业务类

```java
package com.kk.springcloud.controller;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}

```



#### 6、测试

1、启动Config配置中心3344微服务并自测

`http://config-3344.com:3344/master/config-prod.yml`

`http://config-3344.com:3344/master/config-dev.yml`



2、启动3355作为Client准备访问

`http://localhost:3355/configInfo`



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220211204233.png)



### 2、动态刷新问题

- Linux运维修改GitHub上的配置文件内容做调整
- 刷新3344，发现ConfigServer配置中心立刻响应
- 刷新3355，发现ConfigClient客户端没有任何响应
- 3355没有变化除非自己重启或者重新加载
- 难到每次运维修改配置文件，客户端都需要重启？？噩梦



## 4、Config客户端之动态刷新

操作3355模块



#### 1、POM引入actuator监控

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



#### 2、修改YML，暴露监控端口

```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



#### 3、业务类Controller修改

@RefreshScope



```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class ConfigClientController {
    @Value("${spring.application.name}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```



#### 4、此时修改github---> 3344 ---->3355

1、添加一个版本号为1

![](https://v1.mykkto.cn/image/blog/2022/springcloud/202202052320639.png)

2、此时看下3344（config服务端），没有重启的状态下

没有更新！

3、此时看下3355（config客户端），没有重启的状态下

没有更新！



4、总结：是不会更新的



5、解决：需要运维人员运行解决

- 必须是POST请求
- `curl -X POST "http://localhost:3355/actuator/refresh"`



#### 5、还存在的问题

- 每个微服务都要执行一次post请求，手动刷新？

  写循环代码解决（shell脚本）

- 可否广播，一次通知，处处生效？

  鸡肋所在

- 可以使用阿里的nacos 替换解决

- 整合SpringCloud-Bus解决