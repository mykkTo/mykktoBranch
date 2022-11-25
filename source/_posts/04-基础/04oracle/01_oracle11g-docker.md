---
title: Oracle11g(docker版)
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 众所周知，云原生时代，安装用docker特别快
tags:
  - oracle
  - docker
  - 安装
categories:
  - 数据库
abbrlink: 42dbeac3
reprintPolicy: cc_by
date: 2022-02-28 12:49:31
coverImg:
img:
password:
---



## 一、Docker安装和配置

### 1、镜像拉取（第三方）

```bash
  docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```



### 2、下载完后，查看

```bash
 docker images
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228105216.png)

### 3、创建容器

```dockerfile
  docker run -d -p 1521:1521 --name oracle11g registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228105956.png)



### 4、启动容器，操作

#### 1、启动

```dockerfile
  docker start oracle11g
```

#### 2、进入容器

```dockerfile
 docker exec -it oracle11g bash
```

#### 3、切换root用户

```
su root
密码：helowin
```

注意：现在还不能退出，继续操作

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228110834.png)







### 5、编辑profile文件配置ORACLE环境变量

在docker中查找并编辑profile文件 vi /etc/profile

```bash
  export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
  export ORACLE_SID=helowin
  export PATH=$ORACLE_HOME/bin:$PATH
```

在最后上加：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228111122.png)

保存并退出 ：wq



### 6、oracle的配置

#### 1. 创建软连接

```bash
  ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
```

#### 2.切换到oracle 用户

这里还要说一下，一定要写中间的 - 必须要，否则软连接无效

```bash
su - oracle
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228111344.png)



### 7、oracle数据库的操作

#### 1. 登录sqlplus并修改sys、system用户密码

```bash
  sqlplus /nolog
  conn /as sysdba
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228103729.png)



#### 2. 修改和创建用户

```sql
alter user system identified by system;

alter user sys identified by sys;

也可以创建用户  create user test identified by test;

并给用户赋予权限  grant connect,resource,dba to test;
```



#### 3. scott用户的开启

```sql
  --解锁scott用户（安装时若使用默认情况没有解锁和设置密码进行下列操作，要超级管理员操作）
  alter user scott account unlock;

  --解锁scott用户的密码【此句也可以用来重置密码】
  alter user scott identified by tiger;

```



## 二、客户端连接

### 1、navicat连接

打开navicat后（navicat12不用配置oci.dll文件了）

![1646018355584](C:\Users\my_kk\AppData\Roaming\Typora\typora-user-images\1646018355584.png)

### 2、pl/sql 连接

101.xx.xxx.133:1521/helowinXDB
密码：tiger

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228112446.png)



## 三、其他功能

#### 1、scott赋予最高权限

分两条运行

```bash
CONN / AS SYSDBA;
GRANT DBA TO SCOTT;
```



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228114707.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220228114949.png)

## 参考

https://www.cnblogs.com/laoluoits/p/13942119.html

https://www.cnblogs.com/flyingsand/p/9463460.html

