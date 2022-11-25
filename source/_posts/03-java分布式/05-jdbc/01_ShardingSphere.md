---
title: ShardingSphere分库分表【作废】
author: mykk
top: false 
cover: false
toc: true
mathjax: false
summary: 本编是主要对ShardingSphere-JDBC and Proxy进行学习在分布式下分库分表
tags:
  - ShardingSphere
  - 分库分表
  - jdbc
categories:
  - 中间件
abbrlink: 97bfaea3
reprintPolicy: cc_by
date: 2022-04-09 09:01:02
coverImg:
img:
password:
---



## 一、概述

### 1、学英语

博主是个二流子，英语不会但又格外喜欢

ShardingSphere(塞腚-菲儿)【分片-球】、Proxy(普拉克谁)【代理】

### 2、分库分表

#### 1、什么是分库分表



**分库分表就是为了解决由于数据量过大而导致数据库性能降低的问题，将原来独立的数据库拆分成若干数据库组成，将数据大表拆分成若干数据表组成，使得单一数据库、单一数据表的数据量变小，从而达到提升数据库性能的目的。**





#### 2、分库分表方式



**分库分表**



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220411233301.png)



分库分表包括`分库`和`分表`两个部分：

在生产中通常包括：**垂直分库、水平分库、垂直分表、水平分表**四种方式。



**水平拆分：**根据表中数据的逻辑关系，将表中的数据按照某种条件拆分到多台数据库上。



**垂直拆分：**把单一的表拆分成多个表，并分散到不同的数据库（主机）上

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220411233652.png)



#### 2.1 垂直分库

![](https://v1.mykkto.cn/image/blog/2022/springcloud/图3png.png)



#### 2.2 垂直分表

![](https://v1.mykkto.cn/image/blog/2022/springcloud/图2.png)



#### 2.3 水平分表

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220411232915.png)



#### 2.4 水平分库



![](https://v1.mykkto.cn/image/blog/2022/springcloud/图4.png)

#### 3、为什么要分库分表

一般的机器（4核16G），单库的MySQL并发（QPS+TPS）超过了2k，系统基本就宕机了。最好是并发量控制在1k左右。



- QPS：每秒并发量
- TPS：每秒吞吐量



分库分表目的：解决高并发，和数据量大的问题。





#### 4、分库分表的场景

（1）在数据库设计时候考虑垂直分库和垂直分表 

（2）随着数据库数据量增加，不要马上考虑做水平切分，首`先考虑缓存处理，读写分离，使用索引`等等方式，如果这些方式不能根本解决问题了，

再考虑做水平分库和水平分表



#### 5、分库分表带来的问题



• 事务一致性问题

• 跨节点关联查询的问题 ( Join )。 

• 跨节点分页、分组、排序问题。 

• 存在多数据源管理的问题





### 



## 二、Sharding-JDBC快速入门



### 官网

http://shardingsphere.apache.org/index_zh.html



### 1、基本介绍

Sharding-JDBC是当当网研发的开源分布式数据库中间件，从3.0 开始Sharding-JDBC 被包含在Sharding-Sphere中，

之后该项目进入进入Apache孵化器，4.版本之后的版本为Apache版本。



定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。



#### **Sharding-JDBC架构图：**

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220412141303.png)





#### **Sharding-Proxy架构图：**

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220412141632.png)



#### Sharding-Jdbc混合架构：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220412142051.png)



ShardingSphere-JDBC 采用无中心化架构，适用于 Java 开发的高性能的轻量级 OLTP（连接事务处理） 应用；ShardingSphere-Proxy 提供静态入口以及异构语言的支持，适用于 OLAP（连接数据分析） 应用以及对分片数据库进行管理和运维的场景。

Apache ShardingSphere 是多接入端共同组成的生态圈。 通过混合使用 ShardingSphere-JDBC 和 ShardingSphere-Proxy，并采用同一注册中心统一配置分片策略，能够灵活的搭建适用于各种场景的应用系统，使得架构师更加自由地调整适合与当前业务的最佳系统架构。



#### 与jdbc性能对比



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413201213.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413201228.png)



### 2、需求说明

#### 1、案例一：水平分表

手动创建两张表，t_order_1和t_order_2，这两张表是订单表，拆分后的表，通过Sharding-Jdbc向课程表插入数据，按照一定的分片规则，**主键为偶数的进入t_order_1，另一部分数据进入t_order_2**，通过Sharding-Jdbc 查询数据，根据 SQL语句的内容从t_order_1或t_order_2查询数据。



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413232842.png)



#### 2、案例二、水平分库

在案例一的基础上，扩展出两个数据库，根据 user_id 

偶数入库，sharding_jdbc1

奇数入库，sharding_jdbc2



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413233510.png)





#### 3、案例三、垂直分库

按照业余去区分库

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220416204322.png)



### 3、项目搭建

#### 1、技术选型

springboot2.2.1+MybatisPlus+Sharding-JDBC+Druid连接池



阿里镜像：比较快

https://start.aliyun.com/



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413200217.png)





**建表**

两张结构一样

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413214847.png)

```sql
DROP TABLE IF EXISTS `course_1`;
CREATE TABLE `course_1` (
`cid` bigint(20) NOT NULL COMMENT '课程id',
`cname` VARCHAR(50) NOT NULL COMMENT '课程名',
`user_id` bigint(20) NOT NULL COMMENT '课程老师',
`cstatus` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '课程状态',
PRIMARY KEY (`cid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
DROP TABLE IF EXISTS `course_2`;
CREATE TABLE `course_2` (
`cid` bigint(20) NOT NULL COMMENT '课程id',
`cname` VARCHAR(50) NOT NULL COMMENT '课程名',
`user_id` bigint(20) NOT NULL COMMENT '课程老师',
`cstatus` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '课程状态',
PRIMARY KEY (`cid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```







#### 2、建model



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413200115.png)

#### 3、导pom

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.20</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>4.0.0-RC1</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.0.5</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```





#### 4、基础业务代码

##### 1、entiry

```java
package com.kk.shardingjdbc.entiry;

import lombok.Data;

@Data
public class Course {
    private Long cid;
    private String cname;
    private Long userId;
    private String cstatus;
}


```



##### 2、mapper

```java
package com.kk.shardingjdbc.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.springframework.stereotype.Repository;

@Repository
public interface CourseMapper extends BaseMapper<CourseMapper> {
}
```



##### 3、扫描配置

```java
package com.kk.shardingjdbc;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.kk.shardingjdbc.entiry.mapper")
public class ShardingjdbcApplication {

    public static void main(String[] args) {
        SpringApplication.run (ShardingjdbcApplication.class, args);
    }
}
```







#### 5、写 application.properties

##### mysql 数据源 8.0的话

驱动包路径要加 **cj**，`com.mysql.cj.jdbc.Driver`

url要加**时区[?serverTimezone=GMT%2B8]**，`jdbc:mysql://localhost:3306/sharding_jdbc1?serverTimezone=GMT%2B8`





```yml
# sharding分片策略

# 配置数据源，给数据源起别名
spring.shardingsphere.datasource.names=s1

# 配置数据源具体内容，包含连接池，驱动，地址，用户名和密码
spring.shardingsphere.datasource.s1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.s1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.s1.url=jdbc:mysql://localhost:3306/sharding_jdbc1
spring.shardingsphere.datasource.s1.username=root
spring.shardingsphere.datasource.s1.password=a1b2c3

# 指定 course 表分布情况，配置表在哪个数据库里面，表名称都是什么 s1.course_1 , s1.course_2
#course 是表的前缀，  {1,2}是表的后缀，表示有course1 和 course2
spring.shardingsphere.sharding.tables.course.actual-data-nodes=s1.course_$->{1..2}

# 指定 以course为前缀的表里面主键 cid 。生成策略 SNOWFLAKE（雪花算法）
spring.shardingsphere.sharding.tables.course.key-generator.column=cid
spring.shardingsphere.sharding.tables.course.key-generator.type=SNOWFLAKE

# 指定分片策略，约定 cid值为偶数添加到表 course_1, 奇数到 course_2
#course 是表的前缀
#第二行的取模后，+1 操作防止取模后为 0
spring.shardingsphere.sharding.tables.course.table-strategy.inline.sharding-column=cid
spring.shardingsphere.sharding.tables.course.table-strategy.inline.algorithm-expression=course_$->{cid % 2 + 1}

# 打开sql 输出日志
spring.shardingsphere.props.sql.show=true

# 解决一个实体对应两个表问题
spring.main.allow-bean-definition-overriding=true



```



 



#### 6、测试类

```java
package com.kk.shardingjdbc;

import com.kk.shardingjdbc.entiry.Course;
import com.kk.shardingjdbc.mapper.CourseMapper;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
class ShardingjdbcApplicationTests {

    @Autowired
    private CourseMapper courseMapper;

    @Test
    public void addCourse() {
        for (int i = 0; i < 10; i++) {
            Course course = new Course ( );
            course.setCname ("java"+i);
            course.setUserId (100L);
            course.setCstatus ("Normal"+i);
            courseMapper.insert (course);
        }
    }
    
    @Test
    public void searchCourse() {
        QueryWrapper<Course> wrapper = new QueryWrapper<> ( );
        wrapper.eq ("cid",721132131354411008L);
        Course course = courseMapper.selectOne (wrapper);
        System.out.println (course );
    }
}

```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220413224245.png)





### 4、案例二

基于案例一扩展



#### 1、当前库结构

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220414205043.png)

#### 2、application.properties

```application.properties
# sharding分片策略

# 配置数据源，给数据源起别名
spring.shardingsphere.datasource.names=s1,s2

# 配置数据源具体内容，包含连接池，驱动，地址，用户名和密码
spring.shardingsphere.datasource.s1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.s1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.s1.url=jdbc:mysql://localhost:3306/sharding_jdbc1?serverTimezone=GMT%2B8
spring.shardingsphere.datasource.s1.username=root
spring.shardingsphere.datasource.s1.password=a1b2c3

# 配置数据源具体内容，包含连接池，驱动，地址，用户名和密码
spring.shardingsphere.datasource.s2.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.s2.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.s2.url=jdbc:mysql://localhost:3306/sharding_jdbc2?serverTimezone=GMT%2B8
spring.shardingsphere.datasource.s2.username=root
spring.shardingsphere.datasource.s2.password=a1b2c3

#---------------------------------------------------------------------------------------------------

# 指定 course 表分布情况，配置表在哪个数据库里面，表名称都是什么 s1.course_1 , s1.course_2
#course 是表的前缀，  {1,2}是表的后缀，表示有course1 和 course2
#spring.shardingsphere.sharding.tables.course.actual-data-nodes=s1.course_$->{1..2}
# 配置两个数据库
spring.shardingsphere.sharding.tables.course.actual-data-nodes=s$->{1..2}.course_$->{1..2}

# 指定 以course为前缀的表里面主键 cid 。生成策略 SNOWFLAKE（雪花算法）
spring.shardingsphere.sharding.tables.course.key-generator.column=cid
spring.shardingsphere.sharding.tables.course.key-generator.type=SNOWFLAKE

# 指定分片策略，约定 cid值为偶数添加到表 course_1, 奇数到 course_2
#course 是表的前缀
#第二行的取模后，+1 操作防止取模后为 0
spring.shardingsphere.sharding.tables.course.table-strategy.inline.sharding-column=cid
spring.shardingsphere.sharding.tables.course.table-strategy.inline.algorithm-expression=course_$->{cid % 2 + 1}


# 指定数据库分片策略
#约定user_id是偶数添加m1，是奇数添加m2
spring.shardingsphere.sharding.tables.course.database-strategy.inline.sharding-column=user_id
spring.shardingsphere.sharding.tables.course.database-strategy.inline.algorithm-expression=s$->{user_id % 2 + 1}

# 打开sql 输出日志
spring.shardingsphere.props.sql.show=true

# 解决一个实体对应两个表问题
spring.main.allow-bean-definition-overriding=true

```





#### 3、测试

```java
package com.kk.shardingjdbc;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.kk.shardingjdbc.entiry.Course;
import com.kk.shardingjdbc.mapper.CourseMapper;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
class ShardingjdbcApplicationTests {

    @Autowired
    private CourseMapper courseMapper;


    //----------------以下是水平分库测试

    @Test
    public void add() {
        for (int i = 0; i < 20; i++) {
            Course course = new Course ( );
            course.setUserId (0L + i);
            course.setCname ("java" + i);
            course.setCstatus ("Normal" + i);
            courseMapper.insert (course);
        }
    }

    @Test
    public void delete() {
        QueryWrapper<Course> wrapper = new QueryWrapper<> ( );
        wrapper.isNotNull ("cid");

        courseMapper.delete (wrapper);
    }


    //----------------以下是水平分表测试
    @Test
    public void addCourse() {
        for (int i = 0; i < 10; i++) {
            Course course = new Course ( );
            course.setCname ("java" + i);
            course.setUserId (100L);
            course.setCstatus ("Normal" + i);
            courseMapper.insert (course);
        }
    }

    @Test
    public void searchCourse() {
        QueryWrapper<Course> wrapper = new QueryWrapper<> ( );
        wrapper.eq ("cid", 721132131354411008L);
        Course course = courseMapper.selectOne (wrapper);
        System.out.println (course);
    }
}
```





### 5、案例三



#### 1、建表





#### 2、





## 三、Sharding-JDBC执行原理

### 1、

### 2、

### 3、

### 4、

### 5、



## 四、分库分表分类

### 1、

### 2、

### 3、

### 4、

### 5、



## 五、Mysql主从搭建

### 

**[路由：nacos高可用搭建](/posts/d9c3bea0#toc-heading-4)**





## 六、读写分离

### 1、

### 2、

### 3、

### 4、

### 5、





## 参考文档 ↓



https://blog.csdn.net/unique_perfect/article/details/116134490

https://www.kuangstudy.com/zl/sharding#1369532356595126274