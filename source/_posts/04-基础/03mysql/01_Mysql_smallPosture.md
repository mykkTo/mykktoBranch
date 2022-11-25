---
title: mysql开启远程连接
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 最近快过年了，过去肯定是要敲代码，写博文的，近期在写SpringCloud全家桶，数据库一直是在本地，想着还有几台云机在云上运行着，于是连接了下，出现了如下问题
tags:
  - 小姿势
  - linux
  - mysql
categories:
  - 数据库
abbrlink: bf62bc57
reprintPolicy: cc_by
date: 2022-01-27 22:27:39
coverImg:
img:
password:
---



## 1、前言

最近快过年了，回去肯定是要敲代码，写博文的，近期在写SpringCloud全家桶，数据库一直是在本地，想着还有几台云机在云上运行着，于是连接了下，出现了如下问题：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/sjfhgjhfg84.jpg)

之前还是好的，可能挺久没用的权限自己关闭了，安装是docker 可以参考之前博主的简书文章 ：

https://www.jianshu.com/p/f554c85b25c1



### 版本顺便说下

5.7.35 MySQL Community Server (GPL)

## 2、开启远程连接

```shell
#开启远程连接
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'a1b2c3' WITH GRANT OPTION;
#root 用户名
#a1b2c3 密码
```

```shell
#刷新权限，立即生效
flush privileges;
```



## 3、修改密码

```shell
#修改密码(5.7.35)
set password = password ('a1b2c3');

#修改密码（高版本 8.0+）
update mysql.user set authentication_string=password('a1b2c3') where user='a1b2c3';


```

```shell
#刷新权限，立即生效
flush privileges;
```

