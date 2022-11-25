---
title: SpringCloud 项目构建 and 技术选型
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: SpringCloud项目搭建
tags:
  - springcloud
  - 搭建
categories:
  - 分布式技术栈
abbrlink: 9df20094
reprintPolicy: cc_by
date: 2022-01-12 23:42:15
coverImg:
img:
password:
---



## 1、版本

![](https://v1.mykkto.cn/image/blog/202120220112221111.png)

## 2、官网地址【Spring Cloud】

### 1、英文

[https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/](https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/)

### 2、中文

[https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md](https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md)

### 3、springboot 

[https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/)

## 3、项目搭建-父工程构建
- 父工程坐标

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.kk</groupId>
    <artifactId>springcloud2021to2022</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>


    <!-- 统一管理jar包版本 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>

    <!-- 子模块继承之后，提供作用：锁定版本+子modlue不用写groupId和version  -->
    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud Hoxton.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```






## 4、项目搭建-Rest微服务工程构建

### 1、坐标

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
    <artifactId>cloud-provider-payment8001</artifactId>
    
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-jdbc -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

```



### 2、yml

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

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.kk.springclond.entities



```





### 3、主启动

```java
@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run (PaymentMain8001.class, args);
    }
}
```



### 4、业务类

#### 1、建表

```sql
CREATE TABLE `payment` (
`id`  bigint NOT NULL AUTO_INCREMENT ,
`serial`  varchar(255) NULL ,
PRIMARY KEY (`id`)
)

```

#### 2、实体（entity）

##### （1）通用返回结果实体

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {
    private Integer code;
    private String message;
    private T data;

    public CommonResult(Integer code, String message) {
        this(code,message,null);
    }
}
```



##### （2）Payment

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment {
    private Long id;
    private String serial;
}
```



#### 3、dao层

##### 1、接口PaymentDao编写

```java
package com.kk.springcloud.dao;
import com.kk.springcloud.entities.Payment;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

@Mapper
public interface PaymentDao {
    public int create(Payment payment); //写

    public Payment getPaymentById(@Param("id") Long id);  //读取
}

```



##### 2、mybatis的映射文件PaymentMapper.xml



###### （1）路径

`src\main\resources\mapper\PaymentMapper.xml`

###### （2）头文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kk.springcloud.dao.PaymentDao">

    <insert id="create" parameterType="com.kk.springcloud.entities.Payment" useGeneratedKeys="true" keyProperty="id">
            insert into payment(serial) values(${serial});

    </insert>

    <resultMap id="BaseResultMap" type="com.kk.springcloud.entities.Payment">
        <id column="id" property="id" jdbcType="BIGINT"></id>
        <id column="serial" property="serial" jdbcType="VARCHAR"></id>
    </resultMap>
    <select id="getPaymentById" parameterType="Long" resultMap="BaseResultMap">
            select * from payment where id=#{id}
    </select>


</mapper>
```



#### 4、service

##### 1、接口

```java
package com.kk.springcloud.service;
import com.kk.springcloud.entities.Payment;
import org.apache.ibatis.annotations.Param;

public interface PaymentService {
    public int create(Payment payment); //写

    public Payment getPaymentById(@Param("id") Long id);  //读取
}

```

##### 2、实现类

```java
package com.kk.springcloud.service.impl;
import com.kk.springcloud.dao.PaymentDao;
import com.kk.springcloud.entities.Payment;
import com.kk.springcloud.service.PaymentService;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

@Service
public class PaymentServiceImpl implements PaymentService {
    @Resource
    private PaymentDao paymentDao;

    public int create(Payment payment) {
        return paymentDao.create (payment);
    }

    public Payment getPaymentById(Long id) {
        return paymentDao.getPaymentById (id);
    }
}


```



#### 5、controller

```java
package com.kk.springcloud.controller;
import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import com.kk.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;

@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @PostMapping(value = "/payment/create")
    public CommonResult create(Payment payment) {
        int result = paymentService.create (payment);
        log.info ("*****插入结果：" + result);
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
            return new CommonResult (200, "查询成功", payment);
        } else {
            return new CommonResult (444, "没有对应记录，查询ID：" + id, null);
        }
    }
}
```



#### 5、测试

##### 1、插入：使用post请求才能被插入，所以在url上无效，可以使用postman

`http://localhost:8001/payment/create?serial=mykk02`

![](https://v1.mykkto.cn/image/blog/202120220113221758.png)

##### 2、查询：get请求，url，postman皆可

`http://localhost:8001/payment/get/1`

![](https://v1.mykkto.cn/image/blog/202120220113222000.png)



## 5、热部署

### 1、子工程 pom

上面的依赖已经有加了，是这个

![](https://v1.mykkto.cn/image/blog/202120220113222850.png)





### 2、父工程 pom插件

上面的依赖已经有加了，是这个

![](https://v1.mykkto.cn/image/blog/2022/springcloud1642084380903.png)



### 3、设置自动编译

![](https://v1.mykkto.cn/image/blog/2022/springcloud1642084468730.png)

### 4、开启自动更新

#### 1、打开设置面板

`ctrl + shift + alt + /`同时按住，点击第一个`Registry...`

![](https://v1.mykkto.cn/image/blog/2022/springcloud/1642084655326.png)





#### 2、两个打勾

![](https://v1.mykkto.cn/image/blog/2022/springcloud/1642084779624.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/1642084830736.png)

### 5、重启 IDEA



## 6、项目搭建-Order订单微服务构建

### 1、坐标

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

    <artifactId>cloud-consumer-order80</artifactId>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```



### 2、yml

```yml
server:
  port: 80
```



### 3、主启动

```java
package com.kk.springcloud;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run (OrderMain80.class,args);
    }
}
```



### 4、业务类

#### 1、创建entities

- 将cloud-provider-payment8001工程下的entities包下的两个实体类复制过来



#### 2、RestTemplate

##### 1、官网：

- https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

##### 2、是什么

![](https://v1.mykkto.cn/image/blog/202120220113225719.png)



##### 3、怎么用

![](https://v1.mykkto.cn/image/blog/202120220113225906.png)





#### 3、配置类

ApplicationContextConfig

```java
package com.kk.springcloud.config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```



#### 4、创建 controller

```java
package com.kk.springcloud.controller;
import com.kk.springcloud.entities.CommonResult;
import com.kk.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderController {

    public static final String PAYMENT_URL = "http://localhost:8001";

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



#### 5、测试

##### 1、先启动cloud-provider-payment8001

##### 2、再启动cloud-consumer-order80

##### 3、测试消费者接口

`http://localhost/consumer/payment/get/1`



##### 注意点：被调用的生产者接口传参记得加注解

![](https://v1.mykkto.cn/image/blog/202120220113230733.png)





## 7、项目重构

### 1、观察问题

#### 1、系统中有重复部分，重构

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114204913.png)

### 2、新建公共模块【cloud-api-commons】

#### 1、pom

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
    <artifactId>cloud-api-commons</artifactId>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
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

        <!-- https://mvnrepository.com/artifact/cn.hutool/hutool-all -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>
    </dependencies>

</project>

```



#### 2、实体

将订单模块和支付模块公共实体放到这里

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114205744.png)

### 3、改造订单和支付

#### 1、删除各自的原先有过的entities文件夹



#### 2、各自分别引入公共模块

```xml
        <!--引入公共部分-->
        <dependency>
            <groupId>com.kk</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
```



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114210139.png)

## 7、项目模块结构图

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220114210304.png)