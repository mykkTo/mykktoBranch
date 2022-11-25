---
title: RabbitMQ-入门+实战篇
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 本篇主要rabittmq入门案例，原生和集成spring以及boot，以及各种工作中的问题实战方案
tags:
  - amqp
  - rabbitmq
categories:
  - 消息中间件
abbrlink: faf1dfc8
reprintPolicy: cc_by
date: 2022-10-08 23:21:32
coverImg:
img:
password:
---





## 源代码位置

https://gitee.com/TK_LIMR/springcloud2021To2021

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210082337634.png)





## 目录：

- 入门
  - 概述
  - 快速入门案例
  - 工作模式案例
  - 整合Spring
  - 整合SpringBoot
  
- 实战
  - 高级特性
    - 消息可靠性投递（生产者 -> MQ）
    - Consumer ACK (MQ -> 消费者)
    - 消费端限流
    - TTL 过期时间
    - 死信队列
    - ★ 延迟队列 
    - X 日志与监控（了解）
    - X 消息可靠性分析与追踪（慎用）
    - X 管理
  - 应用问题
    - 消息可靠性保障
      - 消息补偿机制
    - 消息幂等性处理
      - 乐观锁解决方案
  - 高可用集群搭建











## Ⅰ、入门篇

## 一、概述

### 1、MQ概述

#### 是什么：

MQ全称 Message Queue（消息队列），是在消息的传输过程中保存消息的容器。多用于分布式系统之间进行通信



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092134272.png)



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092134946.png)



#### 小结：

- MQ，消息队列，存储消息的中间件 
- 分布式系统通信两种方式：直接远程调用 和 借助第三方 完成间接通信 
- 发送方称为生产者，接收方称为消费者



### 2、MQ优势和劣势



#### 1、优势

- 应用解耦
- 异步提速
- 削峰填谷



#### 1-1、应用解耦

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092141083.png)

传统方式，订单系统要发送指定系统，或者打断指定系统的发送需要修改订单系统代码



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092143705.png)

引入 mq，发送给mq即可，选择或者打断指定系统发送只需要控制 topic规则即可，使得应用解耦，提升容错性和可维护性



#### 1-2、异步提速

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092147654.png)



一个下单操作耗时：20 + 300 + 300 + 300 = 920ms ，用户点击完下单按钮后，需要等待920ms才能得到下单响应，太慢！ 



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092148893.png)



用户点击完下单按钮后，只需等待25ms就能得到下单响应 (20 + 5 = 25ms)。 

提升用户体验和系统吞吐量（单位时间内处理请求的数目）



#### 1-3、削峰填谷



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092150352.png)



传统模式当大量并发来临，可能将系统冲垮



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092150963.png)



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092200717.png)



使用了 MQ 之后，限制消费消息的速度为1000，这样一来，高峰期产生的数据势必会被积压在 MQ 中，高峰 

就被“削”掉了，但是因为消息积压，在高峰期过后的一段时间内，消费消息的速度还是会维持在1000，直到消费完积压的消息，这就叫做“填谷”。 



#### 1-4、小结

- 应用解耦：提高系统容错性和可维护性
- 异步提速：提升用户体验和系统吞吐量
- 削峰填谷：提高系统稳定性



#### 2、劣势

- 系统可用性降低
- 系统复杂度变高
- 一致性问题



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092203294.png)



#### 2-1、系统可用性降低

系统引入的外部依赖越多，系统稳定性越差，需要保证可用的越多；一旦MQ宕机，就会对业务影响（如何保证高可用？？  集群）

#### 2-2、系统复杂度提高

传统的系统是同步调用，现在通过MQ异步调用（如何保证消息没有重复消费，怎么处理消息丢失问题，以及消息的有序性）

#### 2-3、一致性问题

A系统处理完业务，通过MQ给 B,C,D 三个系统发送消息，如果B,C成功、D失败，如何保证消息数据的一致性？



#### 2-4、小结

既然 MQ 有优势也有劣势，那么使用 MQ 需要满足什么条件呢？

- 生产者不需要从消费者处获取反馈；上层不需要等待下层回调处理，才能让异步成为可能
- 容许短暂的不一致性
- 在解耦、提速、削峰方面的收益，大于管理MQ的成本



### 3、常见MQ对比

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092229549.png)



### 4、RabbitMQ简介

#### 1、架构图

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092235471.png)



#### 2、组件说明（↑）

- Broker：接收和分发消息的应用，RabbitMQ Server就是 Message Broker
- Virtual host：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网 络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多 个vhost，每个用户在自己的 vhost 创建 exchange／queue 等 
- Connection：publisher／consumer 和 broker 之间的 TCP 连接
- Channel：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线 程，通常每个thread创建单独的 channel 进行通讯，AMQP method 包含了channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。Channel 作为轻量级的 Connection  极大减少了操作系统建立 TCP connection 的开销
- Exchange：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到  queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)
- Queue：消息最终被送到这里等待 consumer 取走
- Binding：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key。Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据



#### 3、RabbitMQ 6中工作模式

简单模式、work queues、Publish/Subscribe 发布与订阅模式、Routing 

路由模式、Topics 主题模式、RPC 远程调用模式（远程调用，不太算 MQ；不作介绍）

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092247367.png)





#### 4、**JMS**

- JMS 即   Java 平台中关于面向消息中间件的API 
- JMS 是 JavaEE 规范中的一种，类比JDBC
- 很多消息中间件都实现了JMS规范，例如：ActiveMQ。RabbitMQ 官方没有提供 JMS 的实现包，但是开源社区有

#### 5、RabbitMQ小结

- RabbitMQ 是基于 AMQP 协议使用 Erlang 语言开发的一款消息队列产品
- RabbitMQ提供了6种工作模式，我们学习5种
- AMQP 是协议，类比HTTP
- JMS 是 API 规范接口，类比 JDBC



## 二、快速入门



### 1、步骤

使用简单模式完成消息传递：

① 创建工程（生成者、消费者） 

② 分别添加依赖 

③ 编写生产者发送消息 

④ 编写消费者接收消息



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092257321.png)



### 2、小结

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210092258177.png)



在上图的模型中，有以下概念：

- P：生产者，也就是要发送消息的程序 
- C：消费者：消息的接收者，会一直等待消息到来（线程一直在监听）
- queue：消息队列，图中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息





## 三、工作模式



### 1、**Work queues 工作队列模式**

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210102229031.png)



#### 1、说明

- **Work Queues：**与入门程序的简单模式相比，多了一个或一些消费端，多个消费端共同消费同一个队列中的消息
- **应用场景**：对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度。



#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210102236513.png)

#### 3、小结

- 在一个队列中如果有多个消费者，那么消费者之间对于同一个消息的关系是**竞争**的关系。 
- Work Queues  对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度。例如：短信服务部署多个，只需要有一个节点成功发送即可。



### 2、**Pub/Sub 订阅模式**

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210102249529.png)



在订阅模型中，多了一个 Exchange 角色，而且过程略有变化：

- P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
- C：消费者，消息的接收者，会一直等待消息到来
- Queue：消息队列，接收消息、缓存消息
- X：Exchange：交换机，一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有常见以下3种类型：
  - Fanout：广播，将消息交给所有绑定到交换机的队列
  - Direct：定向，把消息交给符合指定routing key 的队列
  - Topic：通配符，把消息交给符合routing pattern（路由模式）的队列



**Exchange**（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与 Exchange 绑定，或者没有符合路由规则的队列，那么消息会丢失



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210102309609.png)





小结：

- 交换机需要与队列进行绑定，绑定之后；一个消息可以被多个消费者都收到
- 发布订阅模式与工作队列模式的区别：
  - 工作队列模式不用定义交换机，而发布/订阅模式需要定义交换机
  - 发布/订阅模式的生产方是面向交换机发送消息，工作队列模式的生产方是面向队列发送消息(底层使用默认交换机) 
  - 发布/订阅模式需要设置队列和交换机的绑定，工作队列模式不需要设置，实际上工作队列模式会将队列绑定到默认的交换机



### 3、**Routing 路由模式**

#### 1、模式说明

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个 RoutingKey（路由key） 
- 消息的发送方在向 Exchange 发送消息给 Consumer 时，也必须指定消息的 RoutingKey
- Exchange 不再把消息交给每一个绑定的队列，而是根据消息的 Routing Key 进行判断，只有队列的 Routingkey 与消息的 Routing key 完全一致，才会接收到消息

#### 2、图解

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210102333166.png)



- P：生产者，向 Exchange 发送消息，发送消息时，会指定一个routing key
- X：Exchange（交换机），接收生产者的消息，然后把消息递交给与 routing key 完全匹配的队列
- C1：消费者，其所在队列指定了需要 routing key 为 error 的消息
- C2：消费者，其所在队列指定了需要 routing key 为 info、error、warning 的消息



#### 3、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210111953871.png)

#### 4、小结

**Routing** 模式要求队列在绑定交换机时要指定 **routing key**，消息会转发到符合 routing key 的队列。



### 4、**Topics 通配符模式**

#### 1、说明

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210111955491.png)



Q1 Queue：绑定的是 usa.# ，因此凡是以 usa. 开头的 routing key 都会被匹配到

Q2 Queue：绑定的是 #.news ，因此凡是以 .news 结尾的 routing key 都会被匹配



#### 2、代码



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210112101689.png)



#### 3、小结

Topic 主题模式可以实现 Pub/Sub 发布与订阅模式和 Routing 路由模式的功能，只是 Topic 在配置routing key 的时候可以使用通配符，显得更加灵活。



### 5、工作模式总结 

1. 简单模式 HelloWorld ：一个生产者、一个消费者，不需要设置交换机（使用默认的交换机）。 

2. 工作队列模式 Work Queue ：一个生产者、多个消费者（竞争关系），不需要设置交换机（使用默认的交换机）。 

3. 发布订阅模式 Publish/subscribe ：需要设置类型为 fanout 的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，交换机会将消息发送到绑定的队列。 

4. 路由模式 Routing ：需要设置类型为 direct 的交换机，交换机和队列进行绑定，并且指定 routing key，当发送消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。 

5. 通配符模式 Topic ：需要设置类型为 topic 的交换机，交换机和队列进行绑定，并且指定通配符方式的 routing key，当发送消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。



## 四、Spring 整合 RabbitMQ



### 1、代码



#### 1-1、生产者

① 创建生产者工程 

② 添加依赖 

③ 配置整合 

④ 编写代码发送消息 



#### 1-2、消费者

① 创建生产者工程 

② 添加依赖 

③ 配置整合 

④ 编写消息监听器



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210112244387.png)





### 2、小结



- 使用 Spring 整合 RabbitMQ 将组件全部使用配置方式实现，简化编码 
- Spring 提供 RabbitTemplate 简化发送消息 API 
- 使用监听机制简化消费者编码



## 五、SpringBoot 整合 RabbitMQ

### 1、代码



![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210112252668.png)



## Ⅱ、实战篇



## 一、高级特性



### 1、消息可靠性投递（生产者  ->  MQ）

****



#### 1、概述

在使用 RabbitMQ 的时候，作为消息发送方希望杜绝任何消息丢失或者投递失败场景。RabbitMQ 为我们提供了两种方式用来控制消息的投递可靠性模式。 

- 确认模式
- 回退模式



**rabbitmq 整个消息投递的路径为：**

producer  ---> rabbitmq broker ---> exchange ---> queue ---> consumer


- 消息从 producer 到 exchange 则会返回一个 confirmCallback 的回调
- 消息从 exchange 到 queue 投递失败则返回一个 returnCallback 回调

我们可以利用这两个 callback 回调 控制消息的可靠性投递

#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210162224718.png)



#### 3、小结

- 设置ConnectionFactory的publisher-confirms="true" 开启 确认模式

- 使用rabbitTemplate.setConfirmCallback设置回调函数。当消息发送到exchange后回 

  调confirm方法。在方法中判断ack，如果为true，则发送成功，如果为false，则发 

  送失败，需要处理

- 设置ConnectionFactory的publisher-returns="true" 开启 退回模式

- 使用rabbitTemplate.setReturnCallback设置退回函数，当消息从exchange路由到 

  queue失败后，如果设置了rabbitTemplate.setMandatory(true)参数，则会将消息退 

  回给producer。并执行回调函数returnedMessage

- 在RabbitMQ中也提供了事务机制，但是性能较差，此处不做讲解。 

  使用channel下列方法，完成事务控制： 

  txSelect(), 用于将当前channel设置成transaction模式 

  txCommit()，用于提交事务 

  txRollback(),用于回滚事务



### 2、**Consumer Ack**

**MQ -> 消费者**



#### 1、概述

ack指Acknowledge，确认。 表示消费端收到消息后的确认方式。 

有三种确认方式： 

• 自动确认：acknowledge="none" 

• 手动确认：acknowledge="manual" 

• 根据异常情况确认：acknowledge="auto"，（这种方式使用麻烦，不作讲解）



其中自动确认是指，当消息一旦被Consumer接收到，则自动确认收到，并将相应 message RabbitMQ 的消息缓存中移除。但是在实际业务处理中，很可能消息接收到，业务处理出现异常，那么该消息就会丢失。如果设置了手动确认方式，则需要在业务处理成功后，调用channel.basicAck()，手动签收，如果出现异常，则调用channel.basicNack()方法，让其自动重新发送消息。



#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210201914384.png)



#### 3、小结

- 在rabbit:listener-container标签中设置acknowledge属性，设置ack方式 
  - none：自动确认，
  - manual：手动确认 
- 如果在消费端没有出现异常，则调用channel.basicAck(deliveryTag,false);方法确认签收消息 
- 如果出现异常，则在catch中调用 basicNack或 basicReject，拒绝消息，让MQ重新发送消息。





### 3、消息可靠性总结

- 持久化 
  - exchange要持久化 
  - queue要持久化 
  - message要持久化 

- 生产方确认Confirm 
- 消费方确认Ack
- Broker高可用



### 4、消费端限流



#### 1、概念

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202118692.png)

#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210162342353.png)



#### 3、小结

- 在<rabbit:listener-container> 中配置 prefetch属性设置消费端一次拉取多少消息 
- 消费端的确认模式一定为手动确认。acknowledge="manual"



### 5、TTL 过期时间



#### 1、概述

- TTL 全称 Time To Live（存活时间/过期时间）
- 当消息到达存活时间后，还没有被消费，会被自动清除
- RabbitMQ可以对消息设置过期时间，也可以对整个队列（Queue）设置过期时间

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202120659.png)

#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202131038.png)

#### 3、小结

- 设置队列过期时间使用参数：x-message-ttl，单位：ms(毫秒)，会对整个队列消息统一过期
- 设置消息过期时间使用参数：expiration。单位：ms(毫秒)，当该消息在队列头部时（消费时），会单独判断这一消息是否过期
- 如果两者都进行了设置，以时间短的为准



### 6、死信队**列**



#### 1、概述

**死信队列其实说的就是私信交换机**



死信队列，英文缩写：DLX 。Dead Letter Exchange（死信交换机），当消息成为Dead message后，可以被重新发送到另一个交换机，这个交换机就是DLX

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202139176.png)



**消息成为死信的三种情况：** 

1. 队列消息长度到达限制； （能放10条，放了11条，1条超过被成为死信），可以设置队列存放的数量

2. 消费者拒接消费消息，basicNack/basicReject；并且不把消息重新放入原目标队列,requeue=false； 

3. 原队列存在消息过期设置，消息到达超时时间未被消费；





**队列绑定死信交换机**

给队列设置参数： x-dead-letter-exchange 和 x-dead-letter-routing-key

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202143643.png)





#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202224114.png)

#### 3、小结

- 死信交换机和死信队列和普通的没有区别
- 当消息成为死信后，如果该队列绑定了死信交换机，则消息会被死信交换机重新路由到死信队列
- 消息成为死信的三种情况
  - 队列消息长度到达限制
  - 消费者拒接消费消息，并且不重回队列
  - 原队列存在消息过期设置，消息到达超时时间未被消费



### 7、延迟队列

#### 1、概念

延迟队列，即消息进入队列后不会立即被消费，只有到达指定时间后，才会被消费



**需求：**

- 下单后，30分钟未支付，取消订单，回滚库存
- 新用户注册成功7天后，发送短信问候

**实现方式：**

- 定时器
- 延迟队列

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202243511.png)



很可惜，在RabbitMQ中并未提供延迟队列功能。 

但是可以使用：TTL+死信队列 组合实现延迟队列的效果

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202245709.png)

#### 2、代码

![](https://v1.mykkto.cn/image/blog/2022/mq/rabbitmq/202210202245024.png)



#### 3、小结

- 延迟队列 指消息进入队列后，可以被延迟一定时间，再进行消费
- RabbitMQ没有提供延迟队列功能，但是可以使用 ： TTL + DLX 来实现延迟队列效果

### 



## 二、应用问题



### 1、







## 参考文献 ↓



windown安装 rabbitmq ：https://blog.csdn.net/qq_25919879/article/details/113055350

docker 安装 rabbitmq：https://www.jb51.net/article/233585.htm 

docker 安装后 界面端功能缺失问题：https://blog.csdn.net/qq_45369827/article/details/115921401

springboot-rabbitmq 确认模式弃用配置：https://blog.csdn.net/z69183787/article/details/109371628



单元测试，发布确认回调 ack 总是 false：https://blog.csdn.net/dh554112075/article/details/90137869



spring-boot 死信队列：https://blog.csdn.net/weixin_52062043/article/details/127077941



