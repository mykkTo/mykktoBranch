---
title: SpringCloud-Alibaba-Nacos 服务注册+配置中心
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 阿里开源利器：注册中心（cp/ap切换）+ 配置中心，支持大量组件集成(k8s,dubbo)，雪崩保护，多访问协议等强大功能。
tags:
  - springcloud-alibaba
  - 配置中心
  - nacos
  - 服务注册与发现
categories:
  - 分布式技术栈
abbrlink: af0a257d
reprintPolicy: cc_by
date: 2022-02-23 21:12:31
coverImg:
img:
password:
---



## [友情链接-早期Nacos文章](https://www.jianshu.com/p/85738b23a550)

## 一、Nacos简介

### 1、是什么

项目文档：https://github.com/alibaba/Nacos

使用文档：https://nacos.io/zh-cn/index.html

- 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- Nacos = Eureka+Config +Bus

### 2、能干嘛

- 替代Eureka做服务注册中心
- 替代Config做服务配置中心



### 3、注册中心比较

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220222205601.png)





## 二、安装（docker）

### 1、pull镜像



推荐稳定版版本(官方推荐1.3.1),如果不指定版本的话则就是latest版本(对应nacos的1.4版本)

`docker  pull nacos/nacos-server:1.3.1  `



### 2、运行并创建容器

`docker run  --name nacosName  -e MODE=standalone -d -v  /mnt/logs/nacos:/home/logs/nacos  -p 8848:8848  nacos/nacos-server:1.1.4`

```objectivec
-d 后台运行

-p 外部访问端口:内部被映射端口
Docker相对于虚拟机
    外部访问端口就是    外网访问的端口
    内部被映射端口就是   该镜像在docker里面的端口

–name 容器的名称
镜像名称：版本号  nacos/nacos-server:1.3.1  （运行该镜像）

-e 环境变量设置
-e MODE=standalone

-v  映射到centos上的某个目录:配置某个容器的目录
-v   /mnt/logs/nacos:/home/logs/nacos
```



### 3、查看启动日志

```objectivec
 #查看已经启动的容器  获取容器ID
docker  ps      
#查看指定容器的输出日志
docker logs --since  10m   nacos的容器id      #查看指定容器的输出日志
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220222231829.png)

### 4、访问

```undefined
http://lhttp://localhost:8848/nacos

登录账号 登录密码
nacos     nacos
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220222232020.png)



### 



## 三、Nacos（注册中心）

### 1、基于Nacos生产者

#### 1、建model

cloudalibaba-provider-payment9001



#### 2、pom

1、父pom

```xml
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223213749.png)

2、本模块pom

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

    <artifactId>cloudalibaba-provider-payment9001</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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





#### 3、yml

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 101.34.180.133:8848 #配置Nacos地址

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

@EnableDiscoveryClient
@SpringBootApplication
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run (PaymentMain9001.class, args);
    }
}

```



#### 5、业务类

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: " + serverPort + "\t id" + id;
    }
}

```



#### 6、测试



1、访问：`http://localhost:9001/payment/nacos/1`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223222628.png)

2、nacos客户端

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223222646.png)



#### 7、映射出一模一样的9002，测试负载

1、复制配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223222838.png)



2、编写配置

-DServer.port=9002

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223223054.png)



3、查看是否成功启动

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223223239.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223223313.png)

### 2、基于Nacos消费者

#### 1、建model

cloudalibaba-consumer-nacos-order83

#### 2、pom

为什么nacos支持负载均衡

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223224157.png)

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

    <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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



#### 3、yml

```yml
server:
  port: 83


spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 101.34.180.133:8848 #配置Nacos地址


#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider


```



#### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain83
{
    public static void main(String[] args)
    {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
} 

```



#### 5、业务类



ApplicationContextBean：

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
    public RestTemplate getRestTempe() {
        return new RestTemplate ( );
    }
}

```



OrderNacosController：

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import javax.annotation.Resource;

@RestController
public class OrderNacosController {
    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping("/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject (serverURL + "/payment/nacos/" + id, String.class);
    }
}

```





#### 6、测试

1、客户端注册

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223225123.png)



2、访问：`http://localhost:83/consumer/payment/nacos/13`

1,2,1,2,1,2 切换效果存在

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223225213.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223225236.png)

### 3、服务注册中心对比



#### 1、nacos全景图

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223225259.png)



#### 2、Nacos和CAP

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223225739.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220223225758.png)



#### 3、Nacos 支持AP和CP模式的切换

`C是所有节点在同一时间看到的数据是一致的；而A的定义是所有的请求都会收到响应。`

何时选择使用何种模式？
一般来说，
如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如 Spring cloud 和 Dubbo 服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须，K8S服务和DNS服务则适用于CP模式。
CP模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。


curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'



## 四、Nacos（配置中心）



### 1、建model

#### 1、建model

cloudalibaba-config-nacos-client3377

#### 2、pom

```xml
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
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

    <artifactId>cloudalibaba-config-nacos-client3377</artifactId>

    <dependencies>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础配置-->
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

##### 1、why配置两个

springboot中配置文件的加载是存在优先级顺序的，`bootstrap优先级高于application`



##### 2、bootstrap.yml

```yml
        file-extension: yaml #指定yaml格式的配置
```

```yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置

```



##### 3、application.yml

```yml
spring:
  profiles:
    active: dev # 表示开发环境
```





#### 4、主启动

```java
package com.kk.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
        SpringApplication.run (NacosConfigClientMain3377.class, args);
    }
}
```



#### 5、业务类

`@RefreshScope`

```java
package com.kk.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope //在控制器类加入@RefreshScope注解使当前类下的配置支持Nacos的动态刷新功能。
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```



### 2、配置中心基础

#### 1、规则

${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220225235713.png)



#### 2、新增配置文件

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227230440.png)









`nacos-config-client-dev.yaml`



```yaml
config: 
    info:  nacos to info
```





#### 3、对应配置说明

- prefix 默认为 spring.application.name 的值
- spring.profile.active 即为当前环境对应的 profile，可以通过配置项 spring.profile.active 来配置。
- file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227194612.png)



#### 4、历史配置（回滚）

Nacos会记录配置文件的历史版本默认保留30天，此外还有一键回滚功能，回滚操作将会触发配置更新



nacos-config-client-dev.yaml

DEFAULT_GROUP

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227200717.png)





### 3、测试

- 启动 3377 
- 访问：http://localhost:3377/config/info
- 测试自带刷新功能
  - 修改下Nacos中的yaml配置文件，再次调用查看配置的接口，就会发现配置已经刷新
  - ![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227230902.png)
  - ![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227230927.png)

### 4、配置中心分类



#### 1、Nacos的图形化管理界面

1、配置管理

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227231427.png)

2、名称空间

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220227234647.png)







#### 3、三种方案加载配置

##### 1、DataID方案

（1）说明

指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置



（2）新建两个配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228211808.png)





（3）配置是什么就加载什么

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228211941.png)





##### 2、Group方案

（1）说明

通过Group实现环境区分



（2）新建组

![1646054747679](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1646054747679.png)





（3）控制台上的效果

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228212608.png)





（4）Springboot上的配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228212916.png)

在config下增加一条group的配置即可。可配置为DEV_GROUP或TEST_GROUP



##### 3、Namespace方案

（1）新建dev/test的Namespace

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228214104.png)



（2）回到服务管理-服务列表查看

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228214344.png)



（3）按照域名配置填写

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228214437.png)



（4）yml配置文件

application.yml

![1646056075067](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1646056075067.png)



bootstrap.yml

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228214832.png)



## 五、★ Nacos（集群和持久化）

### 1、基本说明

#### 1、架构图

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228215456.png)

#### 2、官方信息

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228220344.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228220404.png)

### 2、Nacos持久化配置解释



#### 1、Nacos默认自带的是嵌入式数据库derby



#### 2、derby切换到mysql 步骤



（1）进入容器，找到对应的配置文件

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220308230058.png)



（2）在数据库执行以上 sql 

schema.sql

```sql
 
CREATE DATABASE nacos_config;
USE nacos_config;
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(255) DEFAULT NULL,
  `content` LONGTEXT NOT NULL COMMENT 'content',
  `md5` VARCHAR(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` TEXT COMMENT 'source user',
  `src_ip` VARCHAR(20) DEFAULT NULL COMMENT 'source ip',
  `app_name` VARCHAR(128) DEFAULT NULL,
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` VARCHAR(256) DEFAULT NULL,
  `c_use` VARCHAR(64) DEFAULT NULL,
  `effect` VARCHAR(64) DEFAULT NULL,
  `type` VARCHAR(64) DEFAULT NULL,
  `c_schema` TEXT,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(255) NOT NULL COMMENT 'group_id',
  `datum_id` VARCHAR(255) NOT NULL COMMENT 'datum_id',
  `content` LONGTEXT NOT NULL COMMENT '内容',
  `gmt_modified` DATETIME NOT NULL COMMENT '修改时间',
  `app_name` VARCHAR(128) DEFAULT NULL,
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';
 
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(128) NOT NULL COMMENT 'group_id',
  `app_name` VARCHAR(128) DEFAULT NULL COMMENT 'app_name',
  `content` LONGTEXT NOT NULL COMMENT 'content',
  `beta_ips` VARCHAR(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` VARCHAR(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` TEXT COMMENT 'source user',
  `src_ip` VARCHAR(20) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(128) NOT NULL COMMENT 'group_id',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` VARCHAR(128) NOT NULL COMMENT 'tag_id',
  `app_name` VARCHAR(128) DEFAULT NULL COMMENT 'app_name',
  `content` LONGTEXT NOT NULL COMMENT 'content',
  `md5` VARCHAR(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` TEXT COMMENT 'source user',
  `src_ip` VARCHAR(20) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` BIGINT(20) NOT NULL COMMENT 'id',
  `tag_name` VARCHAR(128) NOT NULL COMMENT 'tag_name',
  `tag_type` VARCHAR(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(128) NOT NULL COMMENT 'group_id',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` BIGINT(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` VARCHAR(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` BIGINT(64) UNSIGNED NOT NULL,
  `nid` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
  `data_id` VARCHAR(255) NOT NULL,
  `group_id` VARCHAR(128) NOT NULL,
  `app_name` VARCHAR(128) DEFAULT NULL COMMENT 'app_name',
  `content` LONGTEXT NOT NULL,
  `md5` VARCHAR(32) DEFAULT NULL,
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00',
  `src_user` TEXT,
  `src_ip` VARCHAR(20) DEFAULT NULL,
  `op_type` CHAR(10) DEFAULT NULL,
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';
 
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` VARCHAR(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';
 
 
CREATE TABLE `tenant_info` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` VARCHAR(128) NOT NULL COMMENT 'kp',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT 'tenant_id',
  `tenant_name` VARCHAR(128) DEFAULT '' COMMENT 'tenant_name',
  `tenant_desc` VARCHAR(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` VARCHAR(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` BIGINT(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` BIGINT(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';
 
CREATE TABLE users (
    username VARCHAR(50) NOT NULL PRIMARY KEY,
    PASSWORD VARCHAR(500) NOT NULL,
    enabled BOOLEAN NOT NULL
);
 
CREATE TABLE roles (
    username VARCHAR(50) NOT NULL,
    role VARCHAR(50) NOT NULL
);
 
INSERT INTO users (username, PASSWORD, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);
 
INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
 
 

```







（3）在 application.properties修改配置



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220308235804.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220308232218.png)

以上 五个配置均要修改



（4）改完之后重启

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220308234826.png)



（5）重新登录客户端，发现以前的数据没有了，说明切换成功

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220308234903.png)



### 3、高可用集群搭建

#### 1、架构图

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220309231606.png)





## 最终落地：[Nacos 高可用集群（docker-compose）](/posts/d9c3bea0)

#### 



## 参考文章

https://www.jianshu.com/p/d2c81d647323

https://blog.csdn.net/wwwwwww31311/article/details/113066637

https://www.cnblogs.com/hellxz/p/nacos-cluster-docker.html