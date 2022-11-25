---
title: 历来面基题【长期】
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 这篇文章长期更新，主要内容是自己和朋友面试遇到的以及常见的题目汇总
tags:
  - 原理
  - java
  - 分布式
  - 数据库
categories:
  - 面试题
abbrlink: c1193e28
reprintPolicy: cc_by
date: 2022-04-09 11:06:12
coverImg:
img:
password:
---



## 一、前言

### 1、说明

初稿没想好怎么定义，把最近遇到的面试题大概梳理下（可能比较潦草，暂时没花时间排版）



## 二、题目

一些题目会贴出问题方案，部分暂时没有归纳，不过在此站都能搜索得到，提下

本站采用**全文搜索**，可以方便的检索到问题以及方案。



- 数据库三范式（收集需求和怎么设计数据库的）
- Spring、SpringBoot、SpringCloud区别，展开
  - Spring特性用法，事务，jdbc模板之类
  - SpringBoot零配置......可以的话自定义配置聊下
  - SpringCloud分两部分说（netfix,alibaba），每个组件用法说下，可以的话原理展开下
- Seata ID(1)+组件(3)扩展说明，用法及区别
  - 这边ID用到的雪花算法，懂的话可以展开，如果会写的话，说出实现算法的原理最佳，以及雪花的缺陷（时钟问题），解决方案
  - 三个组件原理本站也有写
- 数据库优化，索引失效哪些情况
- 数据库，物化视图
  - 物化视图与普通的视图相比的区别是物化视图是建立的副本，它类似于一张表，需要占用存储空间。而对一个物化视图查询的执行效率与查询一个表是一样的。
  - 其实就是存在物理内存的副本表，但是性能和视图一样
- 最近项目描述及其展开说明（这个不用多说了吧）
- 同步集合用过哪些，说明，以及分布式锁说明
  - 这边之前想展开AQS说明但是讲了很多了口干舌燥的，水也不够喝了
  - 一方面对底层也是大概了解，FIFO内的Node深入API的话未必答的全面
  - 分布锁可以分为三个实现展开（lua、redission、zookeeper），lua又分为单体和集群都可以展开说明
  - lua 集群实现分布式锁，可以聊下redlock，redission底层就是.....
  - redission 底层自旋的看门口狗(watchdog)机制可以说下
- es用过说下，倒排索引机制原理说下
- 多线程扩展聊了CAS
  - 基本介绍下CAS，优缺点，缺点方案解决
  - 几种原子类说明，跟volatile可见性比较
  - 可以的话原子类原理展开，LongAdder快的原因
- Synchronized用过吗
  - 可以的话说明下用法，然后展开下锁升级（无锁-偏锁-轻锁-重锁）
  - 可以提下JUC中的读写锁（ReentrantReadWriteLock），大概说下读写四种情况以及锁降级
  - 自述完读写锁，可以提下更高效的邮戳锁（StampedLock），优缺点说下，特点说下（名字就可以看出是不重入锁）
- redis击穿、穿透、雪崩
  - 说下是什么，解决方案
  - 布隆过滤器实现原理









## 参考文档 ↓



物化视图：https://baijiahao.baidu.com/s?id=1709212821591222829&wfr=spider&for=pc

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206271903615.jpg)

