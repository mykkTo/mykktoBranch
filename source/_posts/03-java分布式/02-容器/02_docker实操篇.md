---
title: docker 操作篇
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 'docker进阶内容（dockerFile,docker网络，微服务以及容器编排使用...）'
tags:
  - dockerfile
  - docker网络
  - docker-compose
categories:
  - docker
abbrlink: abdfa13a
reprintPolicy: cc_by
date: 2022-07-05 23:17:13
coverImg:
img:
password:
---











## [---------------------------  Docker 部署篇  --------------------------](/posts/caff0950)



## 〇、本章源代码

https://gitee.com/TK_LIMR/springcloud2021To2021.git

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207172321968.png)



## Ⅰ、DockerFile

### 一、是什么

#### 1、官网

https://docs.docker.com/engine/reference/builder



#### 2、概述

Dockerfile是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207071619187.png)



#### 3、步骤

1. 编写Dockerfile文件
2. docker build命令构建镜像
3. docker run依镜像运行容器实例





### 二、DockerFile构建过程解析



#### 1、Dockerfile内容基础知识

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. \#表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交



#### 2、Docker执行Dockerfile的大致流程

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成



#### 3、小总结

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

- Dockerfile是软件的原材料
- Docker镜像是软件的交付品
- Docker容器则可以认为是软件镜像的运行态，也即依照镜像运行的容器实例



Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207072105501.png)



1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;
2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时会真正开始提供服务;
3. Docker容器，容器是直接提供服务的。







### 三、DockerFile常用保留字指令



- FROM：基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from

- MAINTAINER：镜像维护者的姓名和邮箱地址

- RUN：容器构建时需要运行的命令，RUN是在 docker build时运行

  - shell格式：`RUN yum -y install vim`
  - exec格式：

  ![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207072127945.png)

- EXPOSE：当前容器对外暴露出的端口

- WORKDIR：指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

- USER：指定该镜像以什么样的用户去执行，如果都不指定，默认是root

- ENV：用来在构建镜像过程中设置环境变量

  - ENV MY_PATH /usr/mytest
  - 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；
    也可以在其它指令中直接使用这些环境变量
  - 比如：WORKDIR $MY_PATH

- ADD：将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包

- COPY：类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

  - COPY src dest
  - COPY ["src", "dest"]
  - <src源路径>：源文件或者源目录
  - <dest目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

- VOLUME：容器数据卷，用于数据保存和持久化工作

- CMD：指定容器启动后的要干的事情

  - 注意：
    - Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换
    - ![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207072139655.png)
  - 它和前面RUN命令的区别
    - CMD是在docker run 时运行。
    - RUN是在 docker build时运行。

- ENTRYPOINT：也是用来指定一个容器启动时要运行的命令

  - 类似于 CMD 指令，但是ENTRYPOINT不会被docker run后面的命令覆盖，
    而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序





小总结

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207072142349.png)



### 四、案例

#### 1、自定义镜像myCentosJava8







##### 1、要求

Centos7镜像具备vim+ifconfig+jdk8



**准备：JDK8下载位置**

https://mirrors.yangxingzhen.com/jdk/



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112143176.png)



##### 2、编写

```dockerfile
FROM centos:7
MAINTAINER jack<mykkto.cn>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash

```



##### 3、构建



docker build -t 新镜像名字:TAG 

`docker build -t centosjava8:1.5 .`

<font color ='red'>注意：上面TAG后面有个空格，有个点</font>



命令：



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112154614.png)



成功：![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112154465.png)









##### 4、运行

`docker run -it 新镜像名字:TAG `

`docker run -it centosjava8:1.5 /bin/bash`



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112157315.png)



##### 5、总结

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。



特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录



#### 2、虚悬镜像

##### 1、是什么

仓库名、标签都是<none>的镜像，俗称dangling image



写一个

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112209211.png)



##### 2、查看

`docker image ls -f dangling=true`



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112211864.png)



##### 3、删除

`docker image prune`

虚悬镜像已经失去存在价值，可以删

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112220390.png)





### 五、总结

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112222733.png)





## Ⅱ、Docker 网络



### 一、是什么

#### 1、默认

**docker不启动，默认网络情况**

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112239583.png)



#### 2、启动

**docker启动后，网络情况**

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112238959.png)



### 二、命令

 



##### 1、All命令

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112247058.png)



##### 2、主查看网络

`docker network ls`

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112247338.png)



##### 3、查看网络源数据

docker network inspect  XXX网络名字



##### 4、删除网络

docker network rm XXX网络名字



##### 5、案例

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207112250806.png)





### 三、能干嘛

- 容器间的互联和通信以及端口映射
- 容器IP变动时候可以通过服务名直接网络通信而不受到影响



### 四、网络模式





#### 1、列表



##### 1、是什么



- bridge模式：使用--network  bridge指定，默认使用docker0
- host模式：使用--network host指定
- none模式：使用--network none指定
- container模式：使用--network container:NAME或者容器ID指定







#### 2、规则



docker容器内部的ip是有可能会发生改变的



**说明**

1、启动两个容器说明

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207122237578.png)



2、docker inspect 容器ID or 容器名字

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207122240639.png)



#### 3、案例：



#### 3-1、bridge

##### 1、是什么

Docker 服务默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），该桥接网络的名称为docker0，它在<font color ='red'>内核层</font>连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到<font color ='red'>同一个物理网络</font>。Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，<font color ='red'>让主机和容器之间可以通过网桥相互通信</font>。



#查看 bridge 网络的详细信息，并通过 grep 获取名称项

`docker network inspect bridge | grep name`



`ifconfig | grep docker `



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207122244593.png)



##### 2、案例

(1）说明：

- Docker使用Linux桥接，在宿主机虚拟一个Docker容器网桥(docker0)，Docker启动一个容器时会根据Docker网桥的网段分配给容器一个IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信。
- docker run 的时候，没有指定network的话默认使用的网桥模式就是bridge，使用的就是docker0。在宿主机ifconfig,就可以看到docker0和自己create的network(后面讲)eth0，eth1，eth2……代表网卡一，网卡二，网卡三……，lo代表127.0.0.1，即localhost，inet addr用来表示网卡的IP地址
- 网桥docker0创建一对对等虚拟设备接口一个叫veth，另一个叫eth0，成对匹配。
     3.1 整个宿主机的网桥模式都是docker0，类似一个交换机有一堆接口，每个接口叫veth，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这样一对接口叫veth pair）；
     3.2 每个容器实例内部也有一块网卡，每个接口叫eth0；
     3.3 docker0上面的每个veth匹配某个容器实例内部的eth0，两两配对，一一匹配。
   通过上述，将宿主机上的所有容器都连接到这个内部网络上，两个容器在同一个网络下,会从这个网关下各自拿到分配的ip，此时两个容器的网络是互通的。

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207122248655.png)



(2）命令：

```docker
docker run -d -p 8081:8080   --name tomcat81 billygoo/tomcat8-jdk8

docker run -d -p 8082:8080   --name tomcat82 billygoo/tomcat8-jdk8
```





(3）验证：

```bash
[root@VM-0-13-centos ~]# ip addr| tail -n 8

```



 ![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207132212867.png)

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207132216184.png)





#### 3-2、

##### 1、是什么

**直接使用宿主机的 IP 地址与外界进行通信，不再需要额外进行NAT 转换。**



##### 2、案例



(1)说明

容器将<font color ='red'>不会获得</font>一个独立的Network Namespace， 而是和宿主机共用一个Network Namespace。<font color ='red'>容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口。</font>

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207132223740.png)



（2）命令

```docker
docker run -d     --network host --name tomcat83 billygoo/tomcat8-jdk8
```



(3)无之前的配对显示了，看容器实例内部

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207132241253.png)



(4)没有设置-p的端口映射了，如何访问启动的tomcat83 ?



就是默认端口，http://宿主机IP:8080/

比如：tomcat 是 8080，nginx 是 80 ，mysql 是3306



#### 3-3、none

##### 1、是什么

禁用网络功能，只有lo标识(就是127.0.0.1表示本地回环)



##### 2、案例

```docker 
docker run -d -p 8084:8080 --network none --name tomcat84 billygoo/tomcat8-jdk8
```

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207132256020.png)

#### 3-4、container

##### 1、是什么

新建的容器和已经存在的一个容器共享一个网络ip配置而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207132257038.png)



##### 2、案例



Alpine Linux 是一款独立的、非商业的通用 Linux 发行版，专为追求安全性、简单性和资源效率的用户而设计。 可能很多人没听说过这个 Linux 发行版本，但是经常用 Docker 的朋友可能都用过，因为他小，简单，安全而著称，所以作为基础镜像是非常好的一个选择，可谓是麻雀虽小但五脏俱全，镜像非常小巧，不到 6M的大小，所以特别适合容器打包。



```docker
docker run -it       -d                              --name alpine1  alpine /bin/sh

docker run -it  -d --network container:alpine1 --name alpine2  alpine /bin/sh
```



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207161032214.png)



**关闭alpine1，再看看alpine2**

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207161039429.png)





#### 3-5、★ 自定义网络

##### 1、是什么

字面意思，自定义的网络



##### 2、案例

**一类：before**

```docker
docker run -d -p 8081:8080   --name tomcat81 billygoo/tomcat8-jdk8
docker run -d -p 8082:8080   --name tomcat82 billygoo/tomcat8-jdk8
```

上述成功启动并用docker exec进入各自容器实例内部



**存在的问题：**可以按照IP ping 通，但是无法用服务名 ping 通



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207161129303.png)







**二类：after**

1、新建桥接网络

```docker
docker network create jack_network
```



2、新建容器加入上一步新建的自定义网络

```docker
docker run -d -p 8081:8080 --network jack_network  --name tomcat81 billygoo/tomcat8-jdk8

docker run -d -p 8082:8080 --network jack_network  --name tomcat82 billygoo/tomcat8-jdk8

```



3、测试互 ping

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207161722508.png)



**4、结论**

- <font color ='red'>自定义网络本身就维护好了主机名和ip的对应关系（ip和域名都能通）</font>





### 五、Docker架构图解

#### 

从其架构和运行流程来看，Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职。 

Docker 运行的基本流程为：

1 用户是使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者。
2 Docker Daemon 作为 Docker 架构中的主体部分，首先提供 Docker Server 的功能使其可以接受 Docker Client 的请求。
3 Docker Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式的存在。
4 Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 Graph driver将下载镜像以Graph的形式存储。
5 当需要为 Docker 创建网络环境时，通过网络管理驱动 Network driver 创建并配置 Docker 容器网络环境。
6 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 Execdriver 来完成。
7 Libcontainer是一项独立的容器管理包，Network driver以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作。



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207161729655.png)



## Ⅲ、Docker微服务实战

### 一、创建一个普通模块



#### 1、建model

docker_boot



#### 2、改pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>docker_boot</artifactId>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mapper.version>4.1.5</mapper.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>

    <dependencies>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>


    <!-- <build > 主要用于编译设置 -->
    <build>
        <!-- 定义打包成jar的名字 -->
        <!-- 这里如果不定义 , 打包成的jar名字格式为 : <artifactId> + <version> -->
        <finalName>docker_jar</finalName>
        <plugins>
            <!--SpringBoot maven插件-->
            <!-- 可以将应用打成一个可执行的jar包 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- 设置启动入口 -->
                <!-- manClass即使不配置 , SprinBoot也在打包的时候也清楚入口是哪个 , 其实不用配置 -->
                <configuration>
                    <mainClass>com.kk.DockerBootApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```



#### 3、写yml

```yml
server:
  port: 6001
```





#### 4、主启动

```java
package com.kk;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DockerBootApplication {
    public static void main(String[] args) {
        SpringApplication.run (DockerBootApplication.class, args);
    }
}
```





#### 5、业务类

```java
package com.kk.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
public class OrderController {
    @Value("${server.port}")
    private String port;

    @RequestMapping("/order/docker")
    public String helloDocker() {
        return "hello docker" + "\t" + port + "\t" + UUID.randomUUID ( ).toString ( );
    }

    @RequestMapping(value = "/order/index", method = RequestMethod.GET)
    public String index() {
        return "服务端口号: " + "\t" + port + "\t" + UUID.randomUUID ( ).toString ( );
    }
}

```







### 二、打 jar + dockerfile

#### 1、打包

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207170017227.png)



#### 2、编写Dockerfile

```dockerfile
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER jack
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为 jack_docker.jar
ADD docker_jar.jar jack_docker.jar
# 运行jar包
RUN bash -c 'touch /jack_docker.jar'
ENTRYPOINT ["java","-jar","/jack_docker.jar"]
#暴露6001端口作为微服务
EXPOSE 6001
```



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207170932494.png)



#### 3、构建镜像

```docker
docker build -t jack_docker:1.6 .
```



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207170932570.png)





#### 4、运行容器

```docker
docker run -d -p 6001:6001 jack_docker:1.6
```





#### 5、访问测试

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207170936037.png)





## Ⅳ、Docker-compose容器编排

### 一、概述



#### 1、是什么

Docker-Compose是Docker官方的开源项目，快速构建多个容器，负责实现对Docker容器集群的快速编排。



#### 2、安装

1、下载：快速国内镜像

```bash
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```



2、安装

```bash
sudo chmod +x /usr/local/bin/docker-compose
```



3、测试

```docker
docker-compose --version
```



#### 3、核心



1、文件

docker-compose.yml



2、服务（service）

一个个应用容器实例，比如订单微服务、库存微服务、mysql容器、nginx容器或者redis容器



3、工程（project）

由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。



### 二、步骤



1. 编写Dockerfile定义各个微服务应用并构建出对应的镜像文件
2. 使用 docker-compose.yml 定义一个完整业务单元，安排好整体应用中的各个容器服务。
3. 最后，执行docker-compose up命令 来启动并运行整个应用程序，完成一键部署上线





### 三、命令

>Compose常用命令
>docker-compose -h                           # 查看帮助
>docker-compose up                           # 启动所有docker-compose服务
><font color ='red'>docker-compose up -d                        # 启动所有docker-compose服务并后台运行</font>
><font color ='red'>docker-compose down                         # 停止并删除容器、网络、卷、镜像。</font>
>docker-compose exec  yml里面的服务id                 # 进入容器实例内部  docker-compose exec docker-compose.yml文件中写的服务id /bin/bash
>docker-compose ps                      # 展示当前docker-compose编排过的运行的所有容器
>docker-compose top                     # 展示当前docker-compose编排过的容器进程docker-compose logs  yml里面的服务id     # 查看容器输出日志
><font color ='red'>docker-compose config     # 检查配置</font>
><font color ='red'>docker-compose config -q  # 检查配置，有问题才有输出</font>
>docker-compose restart   # 重启服务
>docker-compose start     # 启动服务
>docker-compose stop      # 停止服务





### 四、案例

#### 1、改造微服务



1、建表

```sql
 
CREATE TABLE `t_user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL DEFAULT '' COMMENT '用户名',
  `password` varchar(50) NOT NULL DEFAULT '' COMMENT '密码',
  `sex` tinyint(4) NOT NULL DEFAULT '0' COMMENT '性别 0=女 1=男 ',
  `deleted` tinyint(4) unsigned NOT NULL DEFAULT '0' COMMENT '删除标志，默认0不删除，1删除',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='用户表'

```





2、改pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>docker_boot</artifactId>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mapper.version>4.1.5</mapper.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>

    <dependencies>
        <!--guava Google 开源的 Guava 中自带的布隆过滤器-->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.0</version>
        </dependency>
        <!-- redisson -->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.13.4</version>
        </dependency>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--swagger2-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!--SpringBoot与Redis整合依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--springCache-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <!--springCache连接池依赖包-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <!-- jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.1.0</version>
        </dependency>
        <!--Mysql数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!--SpringBoot集成druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <!--mybatis和springboot整合-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.spring.boot.version}</version>
        </dependency>
        <!-- 添加springboot对amqp的支持 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.10</version>
        </dependency>
        <!--通用基础配置junit/devtools/test/log4j/lombok/hutool-->
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.2.3</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
        <!--persistence-->
        <dependency>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
            <version>1.0.2</version>
        </dependency>
        <!--通用Mapper-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
            <version>${mapper.version}</version>
        </dependency>
    </dependencies>


    <!-- <build > 主要用于编译设置 -->
    <build>
        <!-- 定义打包成jar的名字 -->
        <!-- 这里如果不定义 , 打包成的jar名字格式为 : <artifactId> + <version> -->
        <finalName>docker_jar</finalName>
        <plugins>
            <!--SpringBoot maven插件-->
            <!-- 可以将应用打成一个可执行的jar包 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- 设置启动入口 -->
                <!-- manClass即使不配置 , SprinBoot也在打包的时候也清楚入口是哪个 , 其实不用配置 -->
                <configuration>
                    <mainClass>com.kk.DockerBootApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```



3、application.properties.properties

```boostrap.properties
server.port=6001
# ========================alibaba.druid相关配置=====================
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://106.52.23.202:3306/db2022?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.druid.test-while-idle=false
# ========================redis相关配置=====================
spring.redis.database=0
spring.redis.host=106.12.159.22
spring.redis.port=6379
spring.redis.password=
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=-1ms
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.min-idle=0
# ========================mybatis相关配置===================
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.kk.docker.entities
# ========================swagger=====================
spring.swagger2.enabled=true

```





4、主启动

```java
package com.kk;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication
@MapperScan("com.kk.docker.mapper")
public class DockerBootApplication {
    public static void main(String[] args) {
        SpringApplication.run (DockerBootApplication.class, args);
    }
}
```



5、业务类

5-1、config配置类



**RedisConfig**

```java
package com.kk.docker.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.io.Serializable;

@Configuration
@Slf4j
public class RedisConfig {
    /**
     * @param lettuceConnectionFactory
     * @return redis序列化的工具配置类，下面这个请一定开启配置
     * 127.0.0.1:6379> keys *
     * 1) "ord:102"  序列化过
     * 2) "\xac\xed\x00\x05t\x00\aord:102"   野生，没有序列化过
     */
    @Bean
    public RedisTemplate<String, Serializable> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<> ( );

        redisTemplate.setConnectionFactory (lettuceConnectionFactory);
        //设置key序列化方式string
        redisTemplate.setKeySerializer (new StringRedisSerializer ( ));
        //设置value的序列化方式json
        redisTemplate.setValueSerializer (new GenericJackson2JsonRedisSerializer ( ));

        redisTemplate.setHashKeySerializer (new StringRedisSerializer ( ));
        redisTemplate.setHashValueSerializer (new GenericJackson2JsonRedisSerializer ( ));

        redisTemplate.afterPropertiesSet ( );

        return redisTemplate;
    }

}
```

**SwaggerConfig**

```java
package com.kk.docker.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.text.SimpleDateFormat;
import java.util.Date;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Value("${spring.swagger2.enabled}")
    private Boolean enabled;

    @Bean
    public Docket createRestApi() {
        return new Docket (DocumentationType.SWAGGER_2)
                .apiInfo (apiInfo ( ))
                .enable (enabled)
                .select ( )
                .apis (RequestHandlerSelectors.basePackage ("com.kk.docker")) //你自己的package
                .paths (PathSelectors.any ( ))
                .build ( );
    }

    public ApiInfo apiInfo() {
        return new ApiInfoBuilder ( )
                .title ("java 学习 docker" + "\t" + new SimpleDateFormat ("yyyy-MM-dd").format (new Date ( )))
                .description ("docker-compose")
                .version ("1.0")
                .termsOfServiceUrl ("https://www.mykkto.cn")
                .build ( );
    }
}

```





5-2、新建entity



**User**

```java
package com.kk.docker.entirys;

import javax.persistence.Column;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import java.util.Date;

@Table(name = "t_user")
public class User {
    @Id
    @GeneratedValue(generator = "JDBC")
    private Integer id;

    /**
     * 用户名
     */
    private String username;

    /**
     * 密码
     */
    private String password;

    /**
     * 性别 0=女 1=男
     */
    private Byte sex;

    /**
     * 删除标志，默认0不删除，1删除
     */
    private Byte deleted;

    /**
     * 更新时间
     */
    @Column(name = "update_time")
    private Date updateTime;

    /**
     * 创建时间
     */
    @Column(name = "create_time")
    private Date createTime;

    /**
     * @return id
     */
    public Integer getId() {
        return id;
    }

    /**
     * @param id
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取用户名
     *
     * @return username - 用户名
     */
    public String getUsername() {
        return username;
    }

    /**
     * 设置用户名
     *
     * @param username 用户名
     */
    public void setUsername(String username) {
        this.username = username;
    }

    /**
     * 获取密码
     *
     * @return password - 密码
     */
    public String getPassword() {
        return password;
    }

    /**
     * 设置密码
     *
     * @param password 密码
     */
    public void setPassword(String password) {
        this.password = password;
    }

    /**
     * 获取性别 0=女 1=男
     *
     * @return sex - 性别 0=女 1=男
     */
    public Byte getSex() {
        return sex;
    }

    /**
     * 设置性别 0=女 1=男
     *
     * @param sex 性别 0=女 1=男
     */
    public void setSex(Byte sex) {
        this.sex = sex;
    }

    /**
     * 获取删除标志，默认0不删除，1删除
     *
     * @return deleted - 删除标志，默认0不删除，1删除
     */
    public Byte getDeleted() {
        return deleted;
    }

    /**
     * 设置删除标志，默认0不删除，1删除
     *
     * @param deleted 删除标志，默认0不删除，1删除
     */
    public void setDeleted(Byte deleted) {
        this.deleted = deleted;
    }

    /**
     * 获取更新时间
     *
     * @return update_time - 更新时间
     */
    public Date getUpdateTime() {
        return updateTime;
    }

    /**
     * 设置更新时间
     *
     * @param updateTime 更新时间
     */
    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }

    /**
     * 获取创建时间
     *
     * @return create_time - 创建时间
     */
    public Date getCreateTime() {
        return createTime;
    }

    /**
     * 设置创建时间
     *
     * @param createTime 创建时间
     */
    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }
}

```



**UserDTO**

```java
package com.kk.docker.entirys;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

import java.io.Serializable;
import java.util.Date;

@NoArgsConstructor
@AllArgsConstructor
@Data
@ApiModel(value = "用户信息")
@ToString
public class UserDTO implements Serializable {
    @ApiModelProperty(value = "用户ID")
    private Integer id;

    @ApiModelProperty(value = "用户名")
    private String username;

    @ApiModelProperty(value = "密码")
    private String password;

    @ApiModelProperty(value = "性别 0=女 1=男 ")
    private Byte sex;

    @ApiModelProperty(value = "删除标志，默认0不删除，1删除")
    private Byte deleted;

    @ApiModelProperty(value = "更新时间")
    private Date updateTime;

    @ApiModelProperty(value = "创建时间")
    private Date createTime;

    /**
     * @return id
     */
    public Integer getId() {
        return id;
    }

    /**
     * @param id
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取用户名
     *
     * @return username - 用户名
     */
    public String getUsername() {
        return username;
    }

    /**
     * 设置用户名
     *
     * @param username 用户名
     */
    public void setUsername(String username) {
        this.username = username;
    }

    /**
     * 获取密码
     *
     * @return password - 密码
     */
    public String getPassword() {
        return password;
    }

    /**
     * 设置密码
     *
     * @param password 密码
     */
    public void setPassword(String password) {
        this.password = password;
    }

    /**
     * 获取性别 0=女 1=男
     *
     * @return sex - 性别 0=女 1=男
     */
    public Byte getSex() {
        return sex;
    }

    /**
     * 设置性别 0=女 1=男
     *
     * @param sex 性别 0=女 1=男
     */
    public void setSex(Byte sex) {
        this.sex = sex;
    }

    /**
     * 获取删除标志，默认0不删除，1删除
     *
     * @return deleted - 删除标志，默认0不删除，1删除
     */
    public Byte getDeleted() {
        return deleted;
    }

    /**
     * 设置删除标志，默认0不删除，1删除
     *
     * @param deleted 删除标志，默认0不删除，1删除
     */
    public void setDeleted(Byte deleted) {
        this.deleted = deleted;
    }

    /**
     * 获取更新时间
     *
     * @return update_time - 更新时间
     */
    public Date getUpdateTime() {
        return updateTime;
    }

    /**
     * 设置更新时间
     *
     * @param updateTime 更新时间
     */
    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }

    /**
     * 获取创建时间
     *
     * @return create_time - 创建时间
     */
    public Date getCreateTime() {
        return createTime;
    }

    /**
     * 设置创建时间
     *
     * @param createTime 创建时间
     */
    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

}


```



5-3 新建mapper

**新建接口UserMapper** 

```java
package com.kk.docker.mapper;

import com.kk.docker.entirys.User;
import tk.mybatis.mapper.common.Mapper;

public interface UserMapper extends Mapper<User> {
}

```





**src\main\resources路径下新建mapper文件夹并新增UserMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.docker.mapper.UserMapper">
    <resultMap id="BaseResultMap" type="com.kk.docker.entirys.User">
        <!--
          WARNING - @mbg.generated
        -->
        <id column="id" jdbcType="INTEGER" property="id"/>
        <result column="username" jdbcType="VARCHAR" property="username"/>
        <result column="password" jdbcType="VARCHAR" property="password"/>
        <result column="sex" jdbcType="TINYINT" property="sex"/>
        <result column="deleted" jdbcType="TINYINT" property="deleted"/>
        <result column="update_time" jdbcType="TIMESTAMP" property="updateTime"/>
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime"/>
    </resultMap>
</mapper>

```



5-4 新建service

```java
package com.kk.docker.service;

import com.kk.docker.entirys.User;
import com.kk.docker.mapper.UserMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
@Slf4j
public class UserService {

    public static final String CACHE_KEY_USER = "user:";

    @Resource
    private UserMapper userMapper;
    @Resource
    private RedisTemplate redisTemplate;

    /**
     * addUser
     *
     * @param user
     */
    public void addUser(User user) {
        //1 先插入mysql并成功
        int i = userMapper.insertSelective (user);

        if (i > 0) {
            //2 需要再次查询一下mysql将数据捞回来并ok
            user = userMapper.selectByPrimaryKey (user.getId ( ));
            //3 将捞出来的user存进redis，完成新增功能的数据一致性。
            String key = CACHE_KEY_USER + user.getId ( );
            redisTemplate.opsForValue ( ).set (key, user);
        }
    }

    /**F
     * findUserById
     *
     * @param id
     * @return
     */
    public User findUserById(Integer id) {
        User user = null;
        String key = CACHE_KEY_USER + id;

        //1 先从redis里面查询，如果有直接返回结果，如果没有再去查询mysql
        user = (User) redisTemplate.opsForValue ( ).get (key);

        if (user == null) {
            //2 redis里面无，继续查询mysql
            user = userMapper.selectByPrimaryKey (id);
            if (user == null) {
                //3.1 redis+mysql 都无数据
                //你具体细化，防止多次穿透，我们规定，记录下导致穿透的这个key回写redis
                return user;
            } else {
                //3.2 mysql有，需要将数据写回redis，保证下一次的缓存命中率
                redisTemplate.opsForValue ( ).set (key, user);
            }
        }
        return user;
    }
    
    public void deleteUser(Integer id) {
        
    }

    public void updateUser(User user) {
        
    }

    public User findUserById2(Integer id) {
        return null;
    }
}



```





5-5 新建controller

```java
package com.kk.docker.controller;

import cn.hutool.core.util.IdUtil;
import com.kk.docker.entirys.User;
import com.kk.docker.entirys.UserDTO;
import com.kk.docker.service.UserService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.util.Random;

@Api(description = "用户User接口")
@RestController
@Slf4j
public class UserController {
    @Resource
    private UserService userService;

    @ApiOperation("数据库新增3条记录")
    @RequestMapping(value = "/user/add", method = RequestMethod.POST)
    public void addUser() {
        for (int i = 1; i <= 3; i++) {
            User user = new User ( );

            user.setUsername ("jack" + i);
            user.setPassword (IdUtil.simpleUUID ( ).substring (0, 6));
            user.setSex ((byte) new Random ( ).nextInt (2));

            userService.addUser (user);
        }
    }

    @ApiOperation("删除1条记录")
    @RequestMapping(value = "/user/delete/{id}", method = RequestMethod.POST)
    public void deleteUser(@PathVariable Integer id) {
        userService.deleteUser (id);
    }

    @ApiOperation("修改1条记录")
    @RequestMapping(value = "/user/update", method = RequestMethod.POST)
    public void updateUser(@RequestBody UserDTO userDTO) {
        User user = new User ( );
        BeanUtils.copyProperties (userDTO, user);
        userService.updateUser (user);
    }

    @ApiOperation("查询1条记录")
    @RequestMapping(value = "/user/find/{id}", method = RequestMethod.GET)
    public User findUserById(@PathVariable Integer id) {
        return userService.findUserById2 (id);
    }
}


```





6、打包

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207182254361.png)



7、编写 Dockerfile

```dockerfile
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER jack
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为 jack_docker.jar
ADD docker_jar.jar jack_docker.jar
# 运行jar包
RUN bash -c 'touch /jack_docker.jar'
ENTRYPOINT ["java","-jar","/jack_docker.jar"]
#暴露6001端口作为微服务
EXPOSE 6001
```





8、构建镜像

```docker
docker build  -t  jack_docker:1.7 . 
```

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207182303132.png)



9、运行

```docker
docker run -d -p 6001:6001 --name  jackto17   jack_docker:1.7 
```





10、访问 swageer

http://106.XX.XX.XXX:6001/swagger-ui.html



#### 2、不用Compose



1. 单独启动容器 redis
2. 单独启动容器 mysql
3. 单独启动容器 微服务



#### 3、存在的问题

- 先后顺序要求固定，先mysql+redis才能微服务访问成功
- 多个run命令......
- 容器间的启停或宕机，有可能导致IP地址对应的容器实例变化，映射出错，
  要么生产IP写死(可以但是不推荐)，要么通过服务调用



#### 4、使用Compose

**1、编写 docker-compose.yml文件**

```Dockerfile
version: "3"

services:
  microService:
    image: jack_docker:1.8
    container_name: ms01
    ports:
      - "6001:6001"
    volumes:
      - /app/microService:/data
    networks:
      - mykk_net
    depends_on:
      - redis
      - mysql

  redis:
    image: redis:6.0.8
    ports:
      - "6379:6379"
    volumes:
      - /app/redis/redis.conf:/etc/redis/redis.conf
      - /app/redis/data:/data
    networks:
      - mykk_net
    command: redis-server /etc/redis/redis.conf

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: 'a1b2c3'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_DATABASE: 'db2022'
      MYSQL_USER: 'jack'
      MYSQL_PASSWORD: 'a1b2c3'
    ports:
       - "3306:3306"
    volumes:
       - /app/mysql/db:/var/lib/mysql
       - /app/mysql/conf/my.cnf:/etc/my.cnf
       - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - mykk_net
    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问

networks:
   mykk_net:

```





**2、第二次修改微服务工程 docker_boot**

2-1 改 YML：通过服务名访问，IP无关

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207192321216.png)



2-2 打包，编写 Dockerfile

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207192336342.png)

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207192335353.png)



2-3 构建镜像

```docker
docker build -t  jack_docker:1.8 . 
```

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207192348732.png)



**3、执行 docker-compose**

执行 `docker-compose up`
或者
执行 `docker-compose up -d`



#### 4、创建表

> docker exec -it 容器实例id /bin/bash
>
> <br>
>
> mysql -uroot -p
>
> <br>
>
> create database db2022;
>
> <br>
>
> use db2022;
>
> <br>
>
> CREATE TABLE `t_user` (
>   `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
>   `username` VARCHAR(50) NOT NULL DEFAULT '' COMMENT '用户名',
>   `password` VARCHAR(50) NOT NULL DEFAULT '' COMMENT '密码',
>   `sex` TINYINT(4) NOT NULL DEFAULT '0' COMMENT '性别 0=女 1=男 ',
>   `deleted` TINYINT(4) UNSIGNED NOT NULL DEFAULT '0' COMMENT '删除标志，默认0不删除，1删除',
>   `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
>   `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
>   PRIMARY KEY (`id`)
> ) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';



#### 5、关停

```docker
docker-compose stop
```

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202207202134367.png)





### 五、命令(docker-compose)

>Compose常用命令
>docker-compose -h                           # 查看帮助
><font color ='red'>docker-compose up     </font>                      # 启动所有docker-compose服务
><font color ='red'>docker-compose up -d      </font>                  # 启动所有docker-compose服务并后台运行
><font color ='red'>docker-compose down </font>                        # 停止并删除容器、网络、卷、镜像。
>docker-compose exec  yml里面的服务id                 # 进入容器实例内部  docker-compose exec docker-compose.yml文件中写的服务id /bin/bash
>docker-compose ps                      # 展示当前docker-compose编排过的运行的所有容器
>docker-compose top                     # 展示当前docker-compose编排过的容器进程docker-compose logs  yml里面的服务id     # 查看容器输出日志
><font color ='red'>dokcer-compose config </font>    # 检查配置
><font color ='red'>dokcer-compose config -q </font> # 检查配置，有问题才有输出
>docker-compose restart   # 重启服务
>docker-compose start     # 启动服务
><font color ='red'>docker-compose stop  </font>    # 停止服务





## 



## 参考地址 ↓



### 1、docker 加速（博主简书）

url：https://www.jianshu.com/p/f554c85b25c1



### 2、DockerBuild报错：The command ‘/bin/sh -c yum install -y vim‘ returned a non-zero code: 1

url：https://blog.csdn.net/weixin_53402685/article/details/125296621



### 3、docker-compose 日志输出

url：https://www.cnblogs.com/sunstudy/articles/16340509.html

