---
title: Nacos 高可用集群（docker-compose）
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 这篇文章本来是要写进 nacos学习手册，但是后面搭建遇到很多坑找了很多资料，就单独整理后写出这篇。
tags:
  - springcloud-alibaba
  - 集群
  - nacos
  - docker-compose
categories:
  - 分布式技术栈
abbrlink: d9c3bea0
reprintPolicy: cc_by
date: 2022-03-12 11:17:13
coverImg:
img:
password:
---



## 一、架构

### 1、前言

本来这部分是要在  [Spring-nacos 手册](/posts/af0a257d)中 ，但是后面搭建遇到很多坑找了很多资料，就单独整理后写出这篇。

### 2、架构图

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312143557.png)



一个ngxin 负载 三个 nacos节点，mysql 主从两个节点



## 二、搭建 mysql 主从

### 1、拉取和创建

```docker
# 1、拉取
[root@VM_0_17_centos ~]docker pull mysql:5.7.13

# 2、启动
[root@VM_0_17_centos ~]docker run --name master -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7.13

# 参数说明

--name 为容器指定名称，这里是master
-p 将容器的指定端口映射到主机的指定端口，这里是将容器的3306端口映射到主机的3306端口
-e 设置环境变量，这里是指定root账号的密码为root
-d 后台运行容器，并返回容器ID
mysql:5.7.13 指定运行的mysql版

# 3、查看是否启动,以有容器
[root@VM_0_17_centos ~]# docker ps -a
```



### 2、开放端口

```linux
firewall-cmd --zone=public --add-port=3306/tcp --permanent

firewall-cmd --reload


# 说明
--permanent 永久开启，避免下次开机需要再次手动开启端口
```





### 3、创建[主容器]的复制账号

使用Navicat友好的图像化界面执行SQL



```sql
GRANT REPLICATION SLAVE ON *.* to 'backup'@'%' identified by 'backup';
show grants for 'backup'@'%';
```



### 4、修改MySQL[主容器]配置环境



#### 1、创建配置文件目录，目录结构如下：

/usr/local/mysql/master
/usr/local/mysql/slave1
/usr/local/mysql/slave2

```linux
[root@VM_0_17_centos ~]# mkdir -p /usr/local/mysql/master /usr/local/mysql/slave1 /usr/local/mysql/slave2
```



#### 2、拷贝一份MySQL配置文件

```linux
[root@VM_0_17_centos local]# docker cp master:/etc/mysql/my.cnf /usr/local/mysql/master/my.cnf
```



#### 3、进到master目录下，已存在拷贝的my.cnf

```linux
[root@VM_0_17_centos master]# ll
total 4
-rw-r--r-- 1 root root 1801 May 10 10:27 my.cnf
```



#### 4、修改my.cnf，在 [mysqld] 节点最后加上后保存

```my.cnf
log-bin=mysql-bin
server-id=1
```

`log-bin=mysql-bin` 使用binary logging，mysql-bin是log文件名的前缀

`server-id=1` 唯一服务器ID，非0整数，不能和其他服务器的server-id重复



#### 5、将修改后的文件覆盖Docker中MySQL中的配置文件

```linux
docker cp /usr/local/mysql/master/my.cnf master:/etc/mysql/my.cnf
```



#### 6、重启 mysql 的docker , 让配置生效

```linux
[root@VM_0_17_centos master]# docker restart master
```



### 5、运行MySQL [从]容器



#### 1、首先运行从容器

```linux
[root@VM_0_17_centos ~]# docker run --name slave1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7.13
```



#### 2、与主容器相似，拷贝配置文件至slave1目录修改后覆盖回Docker中

```my.cof
log-bin=mysql-bin
server-id=2
```



#### 3、别忘记，重启slave1容器，使配置生效

#### 

```linux
[root@VM_0_17_centos master]# docker slave1 master
```



### 6、配置主从复制

#### 1、使用Navicat连接 [slave1]后新建查询，执行以下SQL

```sql
CHANGE MASTER TO 
MASTER_HOST='连接Navicat的ip',
MASTER_PORT=3306,
MASTER_USER='backup',
MASTER_PASSWORD='backup';

START SLAVE;
```



MASTER_HOST 填Navicat连接配置中的ip应该就可以

MASTER_PORT 主容器的端口

MASTER_USER 同步账号的用户名

MASTER_PASSWORD 同步账号的密码



#### 2、检查是否配置成功

```sql
show slave status;
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312161305.png)



### 7、检查主从

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312161712.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312161734.png)



## 三、搭建 nacos 集群

### 1、前言

docker-compose配置的源码，已经上传到 github,可直接clone ：

https://github.com/mykkTo/nacos-cluster-docker.git



### 2、配置文件说明



1、Nacos共用的`init.d/custom.properties`，与官方保持一致，按需使用



2、docker-compose-nacos1.yml

以 1为例， 带个都大同小异

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312163817.png)





### 3、启动 nacos 集群

#### 1、将源代码配置修改后，分别上传到三台主机

- **101.34.180.133 对应 nacos-1**
- **106.52.23.202 对应 nacos-2**
- **119.45.122.161 对应 nacos-3**



#### 2、启动容器

分别在各主机上进入各自对应的nacos目录中，启动容器，命令如下：



133服务器：

```docker
$ cd nacos-cluster-docker/nacos-1
$ docker-compose -f docker-compose-nacos1.yml up -d
```



202服务器：

```bash
$ cd nacos-cluster-docker/nacos-2
$ docker-compose -f docker-compose-nacos2.yml up -d
```

161服务器：

```bash
$ cd nacos-cluster-docker/nacos-3
$ docker-compose -f docker-compose-nacos3.yml up -d
```

### 

#### 3、查看日志

查看日志分别在对应的nacos-*目录下，执行

```linux
tail -f cluster-logs/nacos*/nacos.log
```



#### 4、停止容器

```docker
$ docker-compose -f docker-compose-nacos1.yml stop
```



#### 5、访问Nacos UI界面

![](https://v1.mykkto.cn/image/blog/2022/springcloud/P9RM@BX1BNB}%8_G{7UT9SQ.png)



这里我们看到Nacos集群各节点已经正常了，LEADER与FOLLOWER已经选出，一切正常了



## 四、nginx  负载



### 1、创建并启动

#### 1、拉取

```docker
docker pull nginx
```



#### 2、启动

```docker
docker run --name nginx-test -p 80:80 -d nginx
```

-- name 容器命名

-v 映射目录

-d 设置容器后台运行

-p 本机端口映射 将容器的80端口映射到本机的80端口







### 2、映射到本地



#### 1、创建

首先在本机创建nginx的一些文件存储目录

```linux
mkdir -p /root/nginx/www /root/nginx/logs /root/nginx/conf
```



**www**: nginx存储网站网页的目录

**logs**: nginx日志目录

**conf**: nginx配置文件目录





#### 2、映射

（1）先查看容器

```docker
docker ps -a
```



（2）映射

```docker
docker cp 481e121fb29f:/etc/nginx/nginx.conf /root/nginx/conf
```



### 3、启动容器

需要说明下，ngxin-test 容器是为了获得容器的配置文件，最终使用的是 nginx-web

目前已经启动 nginx-test 80端口，若是 nginx-web指定的也是 80，就需要关闭 nginx-test了

```docker
docker stop nginx-test
```



#### 1、新容器映射

创建新nginx容器nginx-web,并将**www,logs,conf**目录映射到本地

```docker
docker run -d -p 80:80 --name nginx-web -v /root/nginx/www:/usr/share/nginx/html -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/logs:/var/log/nginx nginx
```



#### 2、启动

```docker 
docker start nginx-web
```



### 4、配置负载均衡

#### 1、进入 配置

在 root 底下

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312210733.png)



#### 2、配置源码

```nginx.conf
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    #gzip  on;
	upstream nacos-cluster { 
		server 101.34.180.133:8848;
		server 106.52.23.202:8848;
		server 119.45.122.161:8848;
	}
	server {
		listen 80;
		location /{
			proxy_pass http://nacos-cluster;
		}
	}
}
```



主要的是这块

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312210843.png)





说明下：

新手可能不太会nginx，listen 为 80 是因为你容器启动时候是 80，当访问 80的时候转到 以上三个 ip 负载轮训，还可以设置权重可以去看文档



#### 3、重启 

```docker
docker restart nginx-web
```





## 五、Spring-boot连接

#### 1、yml配置

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312211044.png)



#### 2、查看客户端

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220312211146.png)



## 六、问题汇总

### 1、关于 503

异常信息：java.lang.IllegalStateException: failed to req API:/nacos/v1/ns/instance after all servers([101.34.180.133:8848]) tried: failed to req API:101.34.180.133:8848/nacos/v1/ns/instance. code:503 msg: server is DOWN now, please try again later!



个人见解：这个问题博主认为是集群节点少于3个出现的，因为服务器过期了一台剩下了 两台，所以报这个错误，三台没有这个问题。



## 七、参考文档 ↓



https://www.cnblogs.com/hellxz/p/nacos-cluster-docker.html

https://docs.docker.com/compose/install/

https://www.cnblogs.com/bigband/p/13515219.html

https://my.oschina.net/u/3773384/blog/1810111

https://www.jianshu.com/p/658911a8cff3

https://www.yht7.com/news/92162

https://blog.csdn.net/weixin_40461281/article/details/92586378

https://www.cnblogs.com/ilinuxer/p/6916969.html

https://blog.csdn.net/weixin_40461281/article/details/92586378