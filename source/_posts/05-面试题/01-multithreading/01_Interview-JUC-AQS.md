---
title: 高阶面试题：JUC-AQS
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: AQS是JUC多线程中重要的基石，也是java高频面试题，作为程序员你必须会！
tags:
  - AQS
  - 多线程
  - JUC
categories:
  - 面试题
abbrlink: 3fb37166
reprintPolicy: cc_by
date: 2022-03-09 21:32:10
coverImg:
img:
password:
---







## 一、是什么

### 1、字面意思

抽象的队列同步器



结构关系图：

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221163441.png)

### 2、技术解释



- 用来构建锁或者其它同步器组件的`重量级基础框架及整个JUC体系的基石`。
- 通过内置的FIFO1`队列`来完成资源获取线程的排队工作，并通过一个`int类变量`
  表示持有锁的状态

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221163706.png)



CLH：Craig、Landin and Hagersten 队列，是一个单向链表，AQS中的队列是CLH变体的虚拟双向队列FIFO



## 二、AQS=JUC（基石）

AQS为什么是JUC内容中最重要的基石



### 1、AQS有关的锁

- ReentrantLock

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220221144606.png)

- CountDownLatch
- ReentrantReadWriteLock
- Semaphore
- ...............等等

### 2、进一步理解锁和同步器的关系

- 锁，面向锁的使用者：
  - 定义了程序员和锁交互的使用层API，隐藏了实现细节，你调用即可。
- 同步器，面向锁的实现者
  - 比如Java并发大神DougLee，提出统一规范并简化了锁的实现，
    屏蔽了同步状态管理、阻塞线程排队和通知、唤醒机制等。



## 三、能干嘛



### 1、加锁会导致阻塞

有阻塞就需要排队，实现排队必然需要队列



### 2、说明解释

抢到资源的线程直接使用处理业务，抢不到资源的必然涉及一种`排队等候机制`。抢占资源失败的线程继续去等待(类似银行业务办理窗口都满了，暂时没有受理窗口的顾客只能去`候客区排队等候`)，但等候线程仍然保留获取锁的可能且获取锁流程仍在继续(候客区的顾客也在等着叫号，轮到了再去受理窗口办理业务)。

既然说到了`排队等候机制`，那么就一定会有某种队列形成，这样的队列是什么数据结构呢？

如果共享资源被占用，`就需要一定的阻塞等待唤醒机制来保证锁分配`。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS的抽象表现。它将请求共享资源的线程封装成队列的结点（`Node`），通过`CAS、自旋以及LockSupport.park()`的方式，维护state变量的状态，使并发达到同步的效果。



LockSupport.park()：阻塞当前线程的执行，且**都不会释放当前线程占有的锁资源**；

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220222105221.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220222125213.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220222131031.png)

## 四、AQS 解读

### 1、AQS概述

#### 1、官网解释

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224111756.png)

#### 2、阻塞->队列

有阻塞就需要排队，实现排队必然需要队列



AQS使用一个volatile的int类型的成员变量来表示同步状态，通过内置的
FIFO队列来完成资源获取的排队工作将每条要去抢占资源的线程封装成
一个Node节点来实现锁的分配，通过CAS完成对State值的修改。



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224112845.png)



### 2、AQS内部体系架构

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224114013.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224113957.png)



#### 1、AQS自身

##### 1、AQS的int变量 ★

1、AQS的同步状态State成员变量

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224155656.png)

2、银行办理业务的受理窗口状态（通俗理解）

- 零就是没人（自由状态），可以办理
- 大于等于1，有人占用窗口，等着去



##### 2、AQS的CLH队列

1、CLH队列(三个大牛的名字组成)，为一个双向队列

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224161127.png)



2、银行候客区的等待顾客（通俗理解）



##### 3、小总结

- 有阻塞就需要排队，实现排队必然需要队列
- state变量+CLH双端队列

#### 2、内部类Node

内部类Node(Node类在AQS类内部)

##### 1、Node的int变量 ★

1、Node的等待状态waitState成员变量

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224161754.png)



2、说明

- 等候区其它顾客(其它线程)的等待状态
- 队列中每个排队的个体就是一个  Node



##### 2、Node此类详解

1、内部结构

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224162501.png)

2、属性说明

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220224162531.png)



### 3、AQS同步队列的基本结构

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220301135124.png)

CLH：Craig、Landin and Hagersten 队列，是个单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO）

FIFO：队列中用到了哨兵节点（傀儡节点）既，头节点（好处就是不用去判空，因为有头节点）

## 五、从ReentrantLock开始解读AQS

### 1、说明

**Lock接口的实现类，基本都是通过【聚合】了一个【队列同步器】的子类完成线程访问控制的**



1、可以看出内部子类 Sync，继承了 AQS 

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329210645.png)



2、看下类 UML图，Sync 底下又有两个子类（分别为公平和非公平）

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329211351.png)

### 2、公平锁和非公平锁

#### 1、先从创建入手

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329211608.png)

#### 2、追溯

1、默认构造方法，默认不传参构造

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329213723.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329213641.png)





2、带参传入，此时判断是非公平还是公平

![1648561077377](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1648561077377.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329213816.png)



3、很明显这是两个类，分别继承与Sync，上面有提到



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329213915.png)





#### 3、看源码

##### 1、看差异

1、可以明显看出公平锁与非公平锁的lock()方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件 `hasQueuedPredecessors()`

**hasQueuedPredecessors是公平锁加锁时判断等待队列中是否存在有效节点的方法**

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329214203.png)



##### 2、对比

公平锁：公平锁讲究先来先到，线程在获取锁时，如果这个锁的等待队列中已经有线程在等待，那么当前线程就会进入等待队列中；

非公平锁：不管是否有等待队列，如果可以获取锁，则立刻占有锁对象。也就是说队列的第一个排队线程在unpark()，之后还是需要竞争锁（存在线程竞争的情况下）



##### 3、必调 lock()

在创建完公平/非公平锁，调用` lock `方法进行加锁，最终都会调用 `acquire` 方法

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329221223.png)



### 3、源码Api解读



#### 1、lock()

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329221703.png)



#### 2、acquire()

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329223153.png)

**三个走向**

1、tryAcquire() -> tryAcquire () 由于子类 `FairSync` 实现

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329223909.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329223835.png)



2、调用 addWaiter() -> enq() 入队操作

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329224108.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329224256.png)



3、acquireQueued() -> cancelAcquire()

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329225026.png)

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220329225124.png)







## 参考文档

https://www.cnblogs.com/tong-yuan/p/11768904.html

https://baijiahao.baidu.com/s?id=1718317852417206951&wfr=spider&for=pc

https://blog.csdn.net/hengyunabc/article/details/28126139

哨兵节点解读：https://www.cnblogs.com/litexy/p/9749544.html

