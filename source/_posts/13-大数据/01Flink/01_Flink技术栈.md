---
title: 大数据-Flink流式计算框架(JAVA)
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 大数据-Flink(1.13)，基于java代码，电商场景案例
tags:
  - Flink
  - hadoop
  - kafka
categories:
  - 大数据
abbrlink: 1b79564d
reprintPolicy: cc_by
date: 2022-05-17 22:17:13
coverImg:
img:
password:
---



​	



## 0️⃣代码地址

https://github.com/mykkTo/Flink_java.git



## Ⅰ、Flink 初识

### 一、基本概述

#### 1、概述

Flink 是 Apache 基金会旗下的一个开源大数据处理框架。目前，Flink 已经成为各大公司 

大数据实时处理的发力重点，特别是国内以阿里为代表的一众互联网大厂都在全力投入，为 

Flink 社区贡献了大量源码。如今 Flink 已被很多人认为是大数据实时处理的方向和未来，许多 

公司也都在招聘和储备掌握 Flink 技术的人才。 





#### 2、源起和设计理念

Flink 起源于一个叫作 Stratosphere 的项目，它是由 3 所地处柏林的大学和欧洲其他一些大 学在 2010~2014 年共同进行的研究项目，由柏林理工大学的教授沃克尔·马尔科（Volker Markl） 领衔开发。2014 年 4 月，Stratosphere 的代码被复制并捐赠给了 Apache 软件基金会，Flink 就 是在此基础上被重新设计出来的。 在德语中，“flink”一词表示“快速、灵巧”。项目的 logo 是一只彩色的松鼠，当然了， 这不仅是因为 Apache 大数据项目对动物的喜好（是否联想到了 Hadoop、Hive？），更是因为 松鼠这种小动物完美地体现了“快速、灵巧”的特点。关于 logo 的颜色，还一个有趣的缘由： 柏林当地的松鼠非常漂亮，颜色是迷人的红棕色；而 Apache 软件基金会的 logo，刚好也是一 根以红棕色为主的渐变色羽毛。于是，Flink 的松鼠 Logo 就设计成了红棕色，而且拥有一个漂 亮的渐变色尾巴，尾巴的配色与 Apache 软件基金会的 logo 一致。这只松鼠色彩炫目，既呼应 了 Apache 的风格，似乎也预示着 Flink 未来将要大放异彩。

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182118661.png)







### 二、Flink 的应用

Flink 是一个大数据流处理引擎，它可以为不同的行业提供大数据实时处理的解决方案。 随着 Flink 的快速发展完善，如今在世界范围许多公司都可以见到 Flink 的身影。 目前在全球范围内，北美、欧洲和金砖国家均是 Flink 的应用热门区域。当然，这些地区 其实也就是 IT、互联网行业较发达的地区。 

Flink 在国内热度尤其高，一方面是因为阿里的贡献和带头效应，另一方面也跟中国的应 用场景密切相关。中国的人口规模与互联网使用普及程度，决定了对大数据处理的速度要求越 来越高，也迫使中国的互联网企业去追逐更高的数据处理效率。试想在中国，一个网站可能要 面对数亿的日活用户、每秒数亿次的计算峰值，这对很多国外的公司来说是无法想象的。而  Flink 恰好给我们高速准确的处理海量流式数据提供了可能。 



#### 1、企业中的应用

Flink 为全球许多公司和企业的关键业务应用提供了强大的支持。 对于数据处理而言，任何行业、任何公司的需求其实都是一样的：数据规模大、实时性要 求高、确保结果准确、方便扩展、故障后可恢复——而这些要求，作为新一代大数据流式处理 引擎的 Flink 统统可以满足！这也正是 Flink 在全世界范围得到广泛应用的原因。 



#### 2、主要的应用场景

1. **电商和市场营销**

举例：实时数据报表、广告投放、实时推荐 

在电商行业中，网站点击量是统计 PV、UV 的重要来源，也是如今“流量经济”的最主要 数据指标。很多公司的营销策略，比如广告的投放，也是基于点击量来决定的。另外，在网站上提供给用户的实时推荐，往往也是基于当前用户的点击行为做出的。 网站获得的点击数据可能是连续且不均匀的，还可能在同一时间大量产生，这是典型的数 据流。如果我们希望把它们全部收集起来，再去分析处理，就会面临很多问题：首先，我们需 要很大的空间来存储数据；其次，收集数据的过程耗去了大量时间，统计分析结果的实时性就 大大降低了；另外，分布式处理无法保证数据的顺序，如果我们只以数据进入系统的时间为准， 可能导致最终结果计算错误。 我们需要的是直接处理数据流，而 Flink 就可以做到这一点。 

2. **物联网（IOT）**

举例：传感器实时数据采集和显示、实时报警，交通运输业 

物联网是流数据被普遍应用的领域。各种传感器不停获得测量数据，并将它们以流的形式 传输至数据中心。而数据中心会将数据处理分析之后，得到运行状态或者报警信息，实时地显 示在监控屏幕上。所以在物联网中，低延迟的数据传输和处理，以及准确的数据分析通常很关 键。 交通运输业也体现了流处理的重要性。比如说，如今高铁运行主要就是依靠传感器检测数 据，测量数据包括列车的速度和位置，以及轨道周边的状况。这些数据会从轨道传给列车，再 从列车传到沿途的其他传感器；与此同时，数据报告也被发送回控制中心。因为列车处于高速 行驶状态，因此数据处理的实时性要求是极高的。如果流数据没有被及时正确处理，调整意见 和警告就不能相应产生，后果可能会非常严重。 







3. **物流配送和服务业**

举例：订单状态实时更新、通知信息推送 

在很多服务型应用中，都会涉及订单状态的更新和通知的推送。这些信息基于事件触发， 不均匀地连续不断生成，处理之后需要及时传递给用户。这也是非常典型的数据流的处理。 



4. **银行和金融业**

举例：实时结算和通知推送，实时检测异常行为 

银行和金融业是另一个典型的应用行业。用户的交易行为是连续大量发生的，银行面对的 是海量的流式数据。由于要处理的交易数据量太大，以前的银行是按天结算的，汇款一般都要 隔天才能到账。所以有一个说法叫作“银行家工作时间”，说的就是银行家不仅不需要 996，甚 至下午早早就下班了：因为银行需要早点关门进行结算，这样才能保证第二天营业之前算出准 确的账。这显然不能满足我们快速交易的需求。在全球化经济中，能够提供 24 小时服务变得 越来越重要。现在交易和报表都会快速准确地生成，我们跨行转账也可以做到瞬间到账，还可以接到实时的推送通知。这就需要我们能够实时处理数据流。 另外，信用卡欺诈的检测也需要及时的监控和报警。一些金融交易市场，对异常交易行为 的及时检测可以更好地进行风险控制；还可以对异常登录进行检测，从而发现钓鱼式攻击，从 而避免巨大的损失。







### 三、流式计算演变

我们已经了解，Flink 的主要应用场景，就是 <font color ='red'>处理大规模的数据流</font>。那为什么一定要用 Flink 呢？数据处理还有没有其他的方式？要解答这个疑惑，我们就需要先从流处理和批处理的概念 讲起。 

#### 1、流处理和批处理

数据处理有不同的方式。 

对于具体应用来说，有些场景数据是一个一个来的，是一组有序的数据序列，我们把它叫作“数据流”；而有些场景的数据，本身就是一批同时到来，是一个有限的数据集，这就是批量数据（有时也直接叫数据集）。 

容易想到，处理数据流，当然应该“来一个就处理一个”，这种数据处理模式就叫作流处理；因为这种处理是即时的，所以也叫实时处理。与之对应，处理批量数据自然就应该一批读入、一起计算，这种方式就叫作<font color ='red'>批处理，也叫作离线处理</font>。 



那真实的应用场景中，到底是数据流更常见、还是批量数据更常见呢？ 

生活中，这两种形式的数据都有，如图 1-4 所示。比如我们日常发信息，可以一句一句地 说，也可以写一大段一起发过去。一句一句的信息，就是一个一个的数据，它们构成的序列就是一个数据流；而一大段信息，是一组数据的集合，对应就是批量数据（数据集）。 

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182129352.png)



当然，有经验的人都会知道，一句一句地发，你一言我一语，有来有往这才叫聊天；一大 段信息直接砸过去，别人看着都眼晕，很容易就没下文了——如果是很重要的整篇内容（比如 表白信），写成文档或者邮件发过去可能效果会更好。 所以我们看到，“聊天”这个生活场景，数据的生成、传递和接收处理，都是流式的；而 “写信”的场景，数据的生成尽管应该也是流式的（字总得一个个写），但我们可以把它们收集起来，统一传输、统一处理（当然我们还可以进一步较真：处理也是流式的，字得一个一个读）。 不论传输处理的方式是怎样的，数据的生成，一般都是流式的。 



#### 2、传统事务处理

IT 互联网公司往往会用不同的应用程序来处理各种业务。比如内部使用的企业资源规划 （ERP）系统、客户关系管理（CRM）系统，还有面向客户的 Web 应用程序。这些系统一般都 会进行分层设计：“计算层”就是应用程序本身，用于数据计算和处理；而“存储层”往往是传统的关系型数据库，用于数据存储，如图

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182136879.png)

我们发现，这里的应用程序在处理数据的模式上有共同之处：接收的数据是持续生成的事 件，比如用户的点击行为，客户下的订单，或者操作人员发出的请求。处理事件时，应用程序 需要先读取远程数据库的状态，然后按照处理逻辑得到结果，将响应返回给用户，并更新数据库状态。一般来说，一个数据库系统可以服务于多个应用程序，它们有时会访问相同的数据库或表



对于各种事件请求，事务处理的方式能够保证实时响应，好处是一目了然的。但是我们知道，这样的架构对表和数据库的设计要求很高；当数据规模越来越庞大、系统越来越复杂时，可能需要对表进行重构，而且一次联表查询也会花费大量的时间，甚至不能及时得到返回结果。于是，作为程序员就只好将更多的精力放在表的设计和重构，以及 SQL 的调优上，而无法专注于业务逻辑的实现了——我们都知道，这种工作费力费时，却没法直接体现在产品上给老板看，简直就是噩梦。



那有没有更合理、更高效的处理架构呢？**↓**



#### 3、有状态的流处理



不难想到，如果我们对于事件流的处理非常简单，例如收到一条请求就返回一个“收到”，那就可以省去数据库的查询和更新了。但是这样的处理是没什么实际意义的。在现实的应用中，往往需要还其他一些额外数据。我们可以把需要的额外数据保存成一个“状态”，然后针对这条数据进行处理，并且更新状态。在传统架构中，这个状态就是保存在数据库里的。这就是所谓的“有状态的流处理”。 



为了加快访问速度，我们可以直接将状态保存在本地内存，如图所示。

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182141239.png)



当应用收到一个新事件时，它可以从状态中读取数据，也可以更新状态。而当状态是从内存中读写的时候，这就和访问本地变量没什么区别了，实时性可以得到极大的提升。 另外，数据规模增大时，我们也不需要做重构，只需要构建分布式集群，各自在本地计算就可以了，可扩展性也变得更好。



因为采用的是一个分布式系统，所以还需要保护本地状态，防止在故障时数据丢失。我们可以定期地将应用状态的一致性检查点（checkpoint）存盘，写入远程的持久化存储，遇到故障时再去读取进行恢复，这样就保证了更好的容错性。 

##### 



#### 4、有状态流架构

##### 1、事件驱动型（Event-Driven）应用

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182143210.png)



- 事件驱动型应用是一类具有状态的应用，它从一个或多个事件流提取数据，并根据到来的 

事件触发计算、状态更新或其他外部动作。比较典型的就是以 Kafka 为代表的消息队列几乎都 

是事件驱动型应用。 

- 这其实跟传统事务处理本质上是一样的，区别在于基于有状态流处理的事件驱动应用，不 

再需要查询远程数据库，而是在本地访问它们的数据，如图上所示，这样在吞吐量和延迟方 

面就可以有更好的性能。 

- 另外远程持久性存储的检查点保证了应用可以从故障中恢复。检查点可以异步和增量地完 

成，因此对正常计算的影响非常小

##### 

##### 2、数据分析（Data Analysis）型应用

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182152243.png)

- 所谓的数据分析，就是从原始数据中提取信息和发掘规律。传统上，数据分析一般是先将 

数据复制到数据仓库（Data Warehouse），然后进行批量查询。如果数据有了更新，必须将最 

新数据添加到要分析的数据集中，然后重新运行查询或应用程序。 

- 如今，Apache Hadoop 生态系统的组件，已经是许多企业大数据架构中不可或缺的组成部 

分。现在的做法一般是将大量数据（如日志文件）写入 Hadoop 的分布式文件系统（HDFS）、 

S3 或 HBase 等批量存储数据库，以较低的成本进行大容量存储。然后可以通过 SQL-on-Hadoop 

类的引擎查询和处理数据，比如大家熟悉的 Hive。这种处理方式，是典型的批处理，特点是 

可以处理海量数据，但实时性较差，所以也叫离线分析。 

- 如果我们有了一个复杂的流处理引擎，数据分析其实也可以实时执行。流式查询或应用程 

序不是读取有限的数据集，而是接收实时事件流，不断生成和更新结果。结果要么写入外部数 

据库，要么作为内部状态进行维护。 

- Apache Flink 同时支持流式与批处理的数据分析应用

- 与批处理分析相比，流处理分析最大的优势就是低延迟，真正实现了实时。另外，流处理 

不需要去单独考虑新数据的导入和处理，实时更新本来就是流处理的基本模式。当前企业对流 

式数据处理的一个热点应用就是实时数仓，很多公司正是基于 Flink 来实现的。 



##### 3、数据管道（Data Pipeline）型应用

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182155443.png)



- 如图，ETL 与数据管道之间的区别

- ETL 也就是数据的提取、转换、加载，是在存储系统之间转换和移动数据的常用方法。 

在数据分析的应用中，通常会定期触发 ETL 任务，将数据从事务数据库系统复制到分析数据 

库或数据仓库。 

- 所谓数据管道的作用与 ETL 类似。它们可以转换和扩展数据，也可以在存储系统之间移 

动数据。不过如果我们用流处理架构来搭建数据管道，这些工作就可以连续运行，而不需要再 

去周期性触发了。比如，数据管道可以用来监控文件系统目录中的新文件，将数据写入事件日

志。连续数据管道的明显优势是减少了将数据移动到目的地的延迟，而且更加通用，可以用于 

更多的场景。



##### 4、Lambda 架构

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182156240.png)



- 对于有状态的流处理，当数据越来越多时，我们必须用分布式的集群架构来获取更大的吞 

吐量。但是分布式架构会带来另一个问题：怎样保证数据处理的顺序是正确的呢？ 

- 对于批处理来说，这并不是一个问题。因为所有数据都已收集完毕，我们可以根据需要选 

择、排列数据，得到想要的结果。可如果我们采用“来一个处理一个”的流处理，就可能出现 

“乱序”的现象：本来先发生的事件，因为分布处理的原因滞后了。怎么解决这个问题呢？ 

- 以 Storm 为代表的第一代分布式开源流处理器，主要专注于具有毫秒延迟的事件处理，特 

点就是一个字“快”；而对于准确性和结果的一致性，是不提供内置支持的，因为结果有可能 

取决于到达事件的时间和顺序。另外，第一代流处理器通过检查点来保证容错性，但是故障恢 

复的时候，即使事件不会丢失，也有可能被重复处理——所以无法保证 exactly-once。

- 与批处理器相比，可以说第一代流处理器牺牲了结果的准确性，用来换取更低的延迟。而 

批处理器恰好反过来，牺牲了实时性，换取了结果的准确

- 我们自然想到，如果可以让二者做个结合，不就可以同时提供快速和准确的结果了吗？正 

是基于这样的思想，Lambda 架构被设计出来，如上图。我们可以认为这是第二代流处 

理架构，但事实上，它只是第一代流处理器和批处理器的简单合并。 



#### 5、新一代流处理器

- 之前的分布式流处理架构，都有明显的缺陷，人们也一直没有放弃对流处理器的改进和完 

善。终于，在原有流处理器的基础上，新一代分布式开源流处理器诞生了。为了与之前的系统 

区分，我们一般称之为第三代流处理器，代表当然就是 Flink。 

- 第三代流处理器通过巧妙的设计，完美解决了乱序数据对结果正确性的影响。这一代系统 

还做到了精确一次（exactly-once）的一致性保障，是第一个具有一致性和准确结果的开源流 

处理器。另外，先前的流处理器仅能在高吞吐和低延迟中二选一，而新一代系统能够同时提供 

这两个特性。所以可以说，这一代流处理器仅凭一套系统就完成了 Lambda 架构两套系统的工 

作，它的出现使得 Lambda 架构黯然失色。 

- 除了低延迟、容错和结果准确性之外，新一代流处理器还在不断添加新的功能，例如高可 

用的设置，以及与资源管理器（如 YARN 或 Kubernetes）的紧密集成等等。 





### 四、Flink 的特性总结



**Flink 是第三代分布式流处理器，它的功能丰富而强大。**

##### 

#### 1、核心特性

##### 

- 高吞吐和低延迟。每秒处理数百万个事件，毫秒级延迟。

- 结果的准确性。Flink 提供了事件时间（event-time）和处理时间（processing-time） 

  语义。对于乱序事件流，事件时间语义仍然能提供一致且准确的结果。

- 精确一次（exactly-once）的状态一致性保证。

- 可以连接到最常用的存储系统，如 Apache Kafka、Apache Cassandra、Elasticsearch、 

  JDBC、Kinesis 和（分布式）文件系统，如 HDFS 和 S3。

- 高可用。本身高可用的设置，加上与 K8s，YARN 和 Mesos 的紧密集成，再加上从故 

  障中快速恢复和动态扩展任务的能力，Flink 能做到以极少的停机时间 7×24 全天候 

  运行

- 能够更新应用程序代码并将作业（jobs）迁移到不同的 Flink 集群，而不会丢失应用 

  程序的状态。 







#### 2、分层 API

除了上述这些特性之外，Flink 还是一个非常易于开发的框架，因为它拥有易于使用的分层 API，整体 API 分层如图：

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182203484.png)



最底层级的抽象仅仅提供了有状态流，它将处理函数（Process Function）嵌入到了 

DataStream API 中。底层处理函数（Process Function）与 DataStream API 相集成，可以对某 

些操作进行抽象，它允许用户可以使用自定义状态处理来自一个或多个数据流的事件，且状态 

具有一致性和容错保证。除此之外，用户可以注册事件时间并处理时间回调，从而使程序可以 

处理复杂的计算。 

实际上，大多数应用并不需要上述的底层抽象，而是直接针对核心 API（Core APIs） 进 

行编程，比如 DataStream API（用于处理有界或无界流数据）以及 DataSet API（用于处理有界 

数据集）。这些 API 为数据处理提供了通用的构建模块，比如由用户定义的多种形式的转换 

（transformations）、连接（joins）、聚合（aggregations）、窗口（windows）操作等。



### 五、Flink vs Spark

#### 1、数据处理架构



我们已经知道，数据处理的基本方式，可以分为批处理和流处理两种。



<font color ='red'>批处理</font>针对的是<font color ='red'>有界数据集</font>，非常适合需要访问海量的全部数据才能完成的计算工作，一 般用于离线统计。 

<font color ='red'>流处理</font>主要针对的是数据流，特点是<font color ='red'>无界</font>、实时, 对系统传输的每个数据依次执行操作， 一般用于实时统计。 



从根本上说，Spark 和 Flink 采用了完全不同的数据处理方式。可以说，两者的世界观是 截然相反的。 

Spark 以批处理为根本，并尝试在批处理之上支持流计算；在 Spark 的世界观中，万物皆批次，离线数据是一个大批次，而实时数据则是由一个一个无限的小批次组成的。所以对于流处理框架 Spark Streaming 而言，其实并不是真正意义上的“流”处理，而是“微批次” （micro-batching）处理

##### 

##### 1.无界数据流

所谓无界数据流，就是有头没尾，数据的生成和传递会开始但永远不会结束，我们无法等待所有数据都到达，因为输入是无界的，永无止境，数据没有“都到达”的 时候。所以对于无界数据流，必须连续处理，也就是说必须在获取数据后立即处理。在处理无界流时，为了保证结果的正确性，我们必须能够做到按照顺序处理数据。 



##### 2.无界数据流

有界数据流有明确定义的开始和结束，所以我们可以通过获取所有数据来处理有界流。处理有界流就不需要严格保证数据的顺序了，因为总可以对有界数据集进行排序。有界流的处理也就是批处理。



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182208414.png)



正因为这种架构上的不同，Spark 和 Flink 在不同的应用领域上表现会有差别。一般来说，Spark 基于微批处理的方式做同步总有一个“攒批”的过程，所以会有额外开销，因此无法在流处理的低延迟上做到极致。在低延迟流处理场景，Flink 已经有明显的优势。而在海量数据的批处理领域，Spark 能够处理的吞吐量更大，加上其完善的生态和成熟易用的 API，目前同样优势比较明显。 



#### 2、数据模型和运行架构



- 除了三观不合，Spark 和 Flink 在底层实现最主要的差别就在于数据模型不同。

- Spark 底层数据模型是弹性分布式数据集（RDD），Spark Streaming 进行微批处理的底层 

接口 DStream，实际上处理的也是一组组小批数据 RDD 的集合。可以看出，Spark 在设计上本 

身就是以批量的数据集作为基准的，更加适合批处理的场景。 

- 而 Flink 的基本数据模型是数据流（DataFlow），以及事件（Event）序列。Flink 基本上是 

完全按照 Google 的 DataFlow 模型实现的，所以从底层数据模型上看，Flink 是以处理流式数 

据作为设计目标的，更加适合流处理的场景。 

- 数据模型不同，对应在运行处理的流程上，自然也会有不同的架构。Spark 做批计算，需 

要将任务对应的 DAG 划分阶段（Stage），一个完成后经过 shuffle 再进行下一阶段的计算。而 

Flink 是标准的流式执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理。 



#### 3、Spark 还是 Flink

Spark 和 Flink 可以说目前是各擅胜场，批处理领域 Spark 称王，而在流处理方面 Flink 当仁不让。具体到项目应用中，不仅要看是流处理还是 

批处理，还需要在延迟、吞吐量、可靠性，以及开发容易度等多个方面进行权衡。



如果在工作中需要从 Spark 和 Flink 这两个主流框架中选择一个来进行实时流处理，我们更加推荐使用 Flink，主要的原因有： 

- Flink 的延迟是毫秒级别，而 Spark Streaming 的延迟是秒级延迟。
- Flink 提供了严格的精确一次性语义保证。
- Flink 的窗口 API 更加灵活、语义更丰富。
- Flink 提供事件时间语义，可以正确处理延迟数据。
- Flink 提供了更加灵活的对状态编程的 API。





## Ⅱ、Flink 部署



### 一、Flink 快速上手

#### 1、创建项目(1.8)





##### 1、maven

在项目的 pom 文件中，增加<properties>标签设置属性，然后增加<denpendencies>标签引入需要的依赖。我们需要添加的依赖最重要的就是 Flink 的相关组件，包括 flink-java、 flink-streaming-java，以及 flink-clients（客户端，也可以省略）。另外，为了方便查看运行日志， 我们引入 slf4j 和 log4j 进行日志理。 

```xml
    <properties>
        <flink.version>1.13.0</flink.version>
        <java.version>1.8</java.version>
        <scala.binary.version>2.12</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>
    <dependencies>
        <!-- 引入 Flink 相关依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- 引入日志管理相关依赖-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.14.0</version>
        </dependency>
    </dependencies>

```



在属性中，我们定义了<scala.binary.version>，这指代的是所依赖的 Scala 版本。这有一点奇怪：Flink 底层是 Java，而且我们也只用 Java API，为什么还会依赖 Scala 呢？这是因为 Flink的架构中使用了 Akka 来实现底层的分布式通信，而 Akka 是用 Scala 开发的



##### 2、配置日志管理

在目录 src/main/resources 下添加文件:log4j.properties，内容配置如下：

```properties
log4j.rootLogger=error, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
```





#### 2、编写代码

搭好项目框架，接下来就是我们的核心工作——往里面填充代码。我们会用一个最简单的示例来说明 Flink 代码怎样编写：`统计一段文字中，每个单词出现的频次`。这就是传说中的 WordCount 程序——它是大数据领域非常经典的入门案例，地位等同于初学编程语言时的Hello World。



##### 1、批处理

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205182231937.png)

1、根目录下创建一个 txt文件，内容如下

```txt
hello world
hello flink
hello java
```



2、思路

先逐行读入文件数据，然后将每一行文字拆分成单词；接着按照单词分组，统计每组数据的个数，就是对应单词的频次。



3、Java 类 BatchWordCount

```java
package com.kk.wc;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.AggregateOperator;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.FlatMapOperator;
import org.apache.flink.api.java.operators.UnsortedGrouping;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

public class BatchWordCount {
    public static void main(String[] args) throws Exception {
        // 1. 创建执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment ( );
        // 2. 从文件读取数据 按行读取(存储的元素就是每行的文本)
        DataSource<String> listDateSource = env.readTextFile ("input/words.txt");
        // 3. 将每行数据进行分词，转换成二元组类型
        // Collector: flink 定义的收集器
        // Tuple: 二元组类型 <String,Long>,K 就是具体的单词，V 就是个数
        FlatMapOperator<String, Tuple2<String, Long>> wordAndOneTuple = listDateSource.flatMap ((String line, Collector<Tuple2<String, Long>> out) -> {
            // 每行根据空格分隔出，单词
            String[] words = line.split (" ");
            // out.collect  就是输出的意思
            for (String word : words) {
                // Tuple2.of 构建二元组实例
                out.collect (Tuple2.of (word, 1L));
            }
        })
                // returns 解决 scala 泛型擦除问题
                .returns (Types.TUPLE (Types.STRING, Types.LONG));

        // 根据转换得到的运算子  wordAndOneTuple

        // 4. 按照 word 进行分组, 0 表示第 1 个字段索引，就是上面泛型<k,v> 中的 k 为 word 字段
        UnsortedGrouping<Tuple2<String, Long>> wordAndOneGroup = wordAndOneTuple.groupBy (0);

        // 5. 分组进行聚合(求和)统计，1 表示第 2 个字段索引，就是上面泛型<k,v> 中的 v 为 1L 数值的字段
        AggregateOperator<Tuple2<String, Long>> sum = wordAndOneGroup.sum (1);

        // 6. 打印
        sum.print ( );
    }
}

```







##### 2、流处理（有界）

```java
package com.kk.wc;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/*
 * @Description:    流处理有界流
 * @Author:         阿K
 * @CreateDate:     2022/5/19 21:14
 * @Param:
 * @Return:
**/
public class BoundedStreamWordCount {
    public static void main(String[] args) throws Exception{

        // 1. 创建流式执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );

        // 2. 从文件读取数据 按行读取(存储的元素就是每行的文本)
        DataStreamSource<String> listDateSource = env.readTextFile ("input/words.txt");

        // 3. 将每行数据进行分词，转换成二元组类型
        SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = listDateSource.flatMap ((String line, Collector<Tuple2<String,Long>> out) -> {
            String[] words = line.split (" ");
            for (String word : words) {
                out.collect (Tuple2.of (word,1L));
            }
        }).returns (Types.TUPLE (Types.STRING, Types.LONG));

        // 4. 分组(keyBy 按照 key 分组
        KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne.keyBy (data -> data.f0);

        // 5. 求和
        SingleOutputStreamOperator<Tuple2<String, Long>> result = wordAndOneKS.sum (1);

        // 6. 打印
        result.print ();

        // 7. 执行
        env.execute ();

    }
}

```





##### 3、流处理（无界）

利用 nc 模拟实时推送，可以参考底下 【5】

```java
package com.kk.wc;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/*
 * @Description:    流式计算（nc 测试无界流）
 * @Author:         阿K
 * @CreateDate:     2022/5/19 22:43
 * @Param:
 * @Return:
**/
public class StreamWordCount {
    public static void main(String[] args) throws Exception{
        // 1. 创建流式执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );

        // 2. 读取文本流(远程机)
        DataStreamSource<String> lineDataStream = env.socketTextStream ("101.34.180.133", 7777);

        // 3. 将每行数据进行分词，转换成二元组类型
        SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = lineDataStream.flatMap ((String line, Collector<Tuple2<String,Long>> out) -> {
            String[] words = line.split (" ");
            for (String word : words) {
                out.collect (Tuple2.of (word,1L));
            }
        }).returns (Types.TUPLE (Types.STRING, Types.LONG));

        // 4. 分组(keyBy 按照 key 分组
        KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne.keyBy (data -> data.f0);

        // 5. 求和
        SingleOutputStreamOperator<Tuple2<String, Long>> result = wordAndOneKS.sum (1);

        // 6. 打印
        result.print ();

        // 7. 执行
        env.execute ();
    }
}

```

测试上面可以用 nc 测试（实时模拟数据发送）

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205212248685.png)





### 二、作业部署

#### 1、搭建Flink

Flink 是一个分布式的流处理框架，所以实际应用一般都需要搭建集群环境。作者初学就搭建单机好了



**有条件就用 docker 快速占用小，可以参考下面第六**



##### 1.官网下载

flink-1.13.0-bin-scala_2.12.tgz



##### 2.解压

```bash
mkdir /opt/module/

tar -zxvf flink-1.13.0-bin-scala_2.12.tgz -C /opt/module/

```

 



##### 3.启动

```bash
 cd flink-1.13.0/
 
 # 启动
 bin/start-cluster.sh
 
 # 停止
 bin/stop-cluster.sh
```











#### 2、打包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.kk</groupId>
    <artifactId>FlinkTutorial</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <flink.version>1.13.0</flink.version>
        <java.version>1.8</java.version>
        <scala.binary.version>2.12</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>

    <dependencies>
        <!-- 引入Flink相关依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_2.11</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-elasticsearch6_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-statebackend-rocksdb_${scala.binary.version}</artifactId>
            <version>1.13.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner-blink_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-csv</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-cep_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!-- 引入日志管理相关依赖-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.14.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.5</version>
            <scope>provided</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```



会生成两个 jar ，一个有 flink依赖的比较大，一个没有依赖的比较小



#### 3、提交作业

1、选择一个 jar  小的那个没有依赖的

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205212240022.png)

2、上传，登录UI面板：http://101.34.180.133:8081/

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205212242404.png)



3、配置启动类

一个节点选择 1，两个节点选择 2

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205220012706.png)



4、启动 nc 模拟

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205212248685.png)



5、任务提交成功之后，可点击左侧导航栏的“Running Jobs”查看程序运行列表情况

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205220014860.png)



### 



## Ⅲ、DataStream API



### 一、概念

DataStream（数据流）本身是 Flink 中一个用来表示数据集合的类，这套核心 API 可以做`流处理`以及`批处理`,这套API 主要做的是数据的转换



一个 Flink 程序，其实就是对 DataStream 的各种转换。具体来说，代码基本上都由以下几部分构成

- 获取执行环境
- 读取数据源
- 定义基于数据的转换操作
- 定义计算结果的输出位置
- 触发程序执行



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205232214873.png)



### 二、执行环境



运行环境：本地 JVM 中执行程序，也可以提交到远程集群上运行。



#### 1、创建执行环境

我 们 要 获 取 的 执 行 环 境 ， 是 StreamExecutionEnvironment 类的对象，这是所有 Flink 程序的基础。

创建执行环境的方式，就是调用这个类的静态方法，具体有以下三种：

- **getExecutionEnvironment**

  - ```java
            StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
    ```

  - 智能判断：如果当前程序是独立运行，则返回一个本地环境；如果是集群环境，则返回集群环境

- **createLocalEnvironment**

  - ```java
            StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment ( );
    ```

  - 这个方法返回一个本地执行环境。可以在调用时传入一个参数，指定默认的并行度；如果不传入，则默认并行度就是本地的 CPU 核心数。

- **createRemoteEnvironment**

  - ```java
          StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment
                    .createRemoteEnvironment (
                            "host", // JobManager 主机名
                            1234, // JobManager 进程端口号
                            "path/to/jarFile.jar" // 提交给 JobManager 的 JAR 包
                    );
    ```

  - 在获取到程序执行环境后，还可以对执行环境进行灵活的设置。比如可以全局设置程序的并行度、禁用算子链，还可以定义程序的时间语义、配置容错机制。关于时间语义和容错机制



#### 2、执行模式	

```java
        // 批处理环境
        ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment ( );
        // 流处理环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
```





#### 3、触发程序执行

1、当 main()方法被调用时，其实只是定义了作业的每个执行操作，然后添加到数据流图中；这时并没有真正处理数据【因为数据可能还没来】

2、只有等到数据到来，才会触发真正的计算，这也被称为“延迟执行”或“懒执行”

3、所以我们需要显式地调用执行环境的 execute()方法，来触发程序执行。execute()方法将一直等待作业完成，然后返回一个执行结果

```java
env.execute();
```



### 三、源算子

#### 1、准备工作

为了更好地理解，我们先构建一个实际应用场景。比如网站的访问操作，可以抽象成一个三元组（用户名，用户访问的 urrl，用户访问 url 的时间戳），所以在这里，我们可以创建一个类 Event，将用户行为包装成它的一个对象。Event 包含了以下一些字段

|  字段名   | 数据类型 |        说明         |
| :-------: | :------: | :-----------------: |
|   user    |  String  |       用户名        |
|    url    |  String  |    用户访问的url    |
| timeStamp |   Long   | 用户访问url的时间戳 |



```java
package com.kk.model2.pojo;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Event {
    public String user;
    public String url;
    public Long timestamp;
}

```



<font color ='red'>这里需要注意，我们定义的 Event，有这样几个特点：</font>

- 类是公有（public）的
- 有一个无参的构造方法
- 所有属性都是公有（public）的
- 所有属性的类型都是可以序列化的

Flink 会把这样的类作为一种特殊的 POJO 数据类型来对待，方便数据的解析和序列化。



#### 2、常规读数据

集合读数据-文件读数据-Socket 读数据

```java
package com.kk.model2;


import com.kk.model2.pojo.Event;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.ArrayList;

/*
 * @Description:    集合中读取数据
 * @Author:         阿K
 * @CreateDate:     2022/5/24 22:04
 * @Param:
 * @Return:
 **/
public class SourceTest1 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        // 设置并行度为 1 ，保证有序运行
        env.setParallelism (1);

        // 1.从文件中读取数据
        DataStreamSource<String> stream1 = env.readTextFile ("input/clicks.csv");

        // 2.从集合中读取数据
        ArrayList<Integer> nums = new ArrayList<> ( );
        nums.add (2);
        nums.add (5);
        DataStreamSource<Integer> numStream = env.fromCollection (nums);

        ArrayList<Event> events = new ArrayList<> ( );
        events.add (new Event ("Mary", "./home", 1000L));
        events.add (new Event ("Bob", "./cart", 2000L));
        DataStreamSource<Event> stream2 = env.fromCollection (events);


        // 3.从元素读取数据
        DataStreamSource<Event> stream3 = env.fromElements (
                new Event ("Mary", "./home", 1000L),
                new Event ("Bob", "./cart", 2000L)
        );

        // 从Socket文本流读取
        DataStreamSource<String> stream4 = env.socketTextStream ("localhost", 7777);

//        stream1.print("1");
//        numStream.print("nums");
//        stream2.print("2");
//        stream3.print("3");

        stream4.print ("4");

        env.execute ( );

    }

}

```



#### 3、Kafka 读数据 ★

Flink 官方提供的是一个通用的 Kafka 连接器，它会自动跟踪最新版本的 Kafka 客户端

```xml
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
```

然后调用 env.addSource()，传入 FlinkKafkaConsumer 的对象实例就可以了

```java
package com.kk.model2;

import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;

public class SourceKafkaTest {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        Properties properties = new Properties ( );
        properties.setProperty ("bootstrap.servers", "106.12.159.22:9092");
//        properties.setProperty("group.id", "consumer-group");
//        properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
//        properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
//        properties.setProperty("auto.offset.reset", "latest");

        DataStreamSource<String> stream = env.addSource (new FlinkKafkaConsumer<String> (
                "sun",
                new SimpleStringSchema ( ),
                properties
        ));

        stream.print ( );
        env.execute ( );
    }
}

```



**创建 FlinkKafkaConsumer 时需要传入三个参数：** 

⚫ 第一个参数 topic，定义了从哪些主题中读取数据。可以是一个 topic，也可以是 topic列表，还可以是匹配所有想要读取的 topic 的正则表达式。当从多个 topic 中读取数据时，Kafka 连接器将会处理所有 topic 的分区，将这些分区的数据放到一条流中去。 

⚫ 第二个参数是一个 DeserializationSchema 或者 KeyedDeserializationSchema。Kafka 消息被存储为原始的字节数据，所以需要反序列化成 Java 或者 Scala 对象。上面代码中使用的 SimpleStringSchema，是一个内置的 DeserializationSchema，它只是将字节数组简单地反序列化成字符串。DeserializationSchema 和 KeyedDeserializationSchema 是公共接口，所以我们也可以自定义反序列化逻辑。 

⚫ 第三个参数是一个 Properties 对象，设置了 Kafka 客户端的一些属性。 



**结果：**

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202205302348183.png)



#### 4、自定义 Source

想要读取的数据源来自某个外部系统，而 flink 既没有预实现的方法、也没有提供连接器，

我们创建一个自定义的数据源，实现 SourceFunction 接口

⚫ run()方法：使用运行时上下文对象（SourceContext）向下游发送数据； 

⚫ cancel()方法：通过标识位控制退出循环，来达到中断数据源的效果。 	

```java
package com.kk.model2;

import com.kk.model2.pojo.Event;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Calendar;
import java.util.Random;

public class ClickSource implements SourceFunction<Event> {

    // 声明一个布尔变量，作为控制数据生成的标识位
    private Boolean running = true;

    @Override
    public void run(SourceContext<Event> ctx) throws Exception {
        // 在指定的数据集中随机选取数据
        Random random = new Random ( );
        String[] users = {"Mary", "Alice", "Bob", "Cary"};
        String[] urls = {"./home", "./cart", "./fav", "./prod?id=1", "./prod?id=2"};
        while (running) {
            ctx.collect (new Event (
                    users[random.nextInt (users.length)],
                    urls[random.nextInt (urls.length)],
                    Calendar.getInstance ( ).getTimeInMillis ( )
            ));
            // 隔 1 秒生成一个点击事件，方便观测
            Thread.sleep (1000);
        }
    }

    @Override
    public void cancel() {
        running = false;
    }
}

```

这个数据源，我们后面会频繁使用，所以在后面的代码中涉及到 ClickSource()数据源，使用上面的代码就可以了。

下面的代码我们来读取一下自定义的数据源。有了自定义的 source function，接下来只要调用 addSource()就可以了

```java
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        // 有了自定义的 source function，调用 addSource 方法
        DataStreamSource<Event> stream = env.addSource(new ClickSource());
        stream.print("SourceCustom");
        env.execute();
    }
```



所以如果我们想要自定义并行的数据源的话，需要使用 ParallelSourceFunction

```java
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.addSource (new CustomSource ( )).setParallelism (2).print ( );
        env.execute ( );
    }

    public static class CustomSource implements ParallelSourceFunction<Integer> {
        private boolean running = true;
        private Random random = new Random ( );

        @Override
        public void run(SourceContext<Integer> sourceContext) throws Exception {
            while (running) {
                sourceContext.collect (random.nextInt ( ));
            }
        }

        @Override
        public void cancel() {
            running = false;
        }
    }
```



#### 5、Flink 支持的数据类型

##### 1. **Flink** **的类型系统**

Flink 有自己一整套类型系统。Flink 使用“类型信息” （TypeInformation）来统一表示数据类型。TypeInformation 类是 Flink 中所有类型描述符的基类。 

它涵盖了类型的一些基本属性，并为每个数据类型生成特定的序列化器、反序列化器和比较器。



##### 2. **Flink** **支持的数据类型**

（1）基本类型 

所有 Java 基本类型及其包装类，再加上 Void、String、Date、BigDecimal 和 BigInteger。

（2）数组类型 

包括基本类型数组（PRIMITIVE_ARRAY）和对象数组(OBJECT_ARRAY)

（3）复合数据类型 

⚫ Java 元组类型（TUPLE）：这是 Flink 内置的元组类型，是 Java API 的一部分。最多 25 个字段，也就是从 Tuple0~Tuple25，不支持空字段 

⚫ Scala 样例类及 Scala 元组：不支持空字段 

⚫ 行类型（ROW）：可以认为是具有任意个字段的元组,并支持空字段 

⚫ POJO：Flink 自定义的类似于 Java bean 模式的类 

（4）辅助类型 

Option、Either、List、Map 等 

（5）泛型类型（GENERIC） 

Flink 支持所有的 Java 类和 Scala 类。不过如果没有按照上面 POJO 类型的要求来定义，就会被 Flink 当作泛型类来处理。Flink 会把泛型类型当作黑盒，无法获取它们内部的属性；它们也不是由 Flink 本身序列化的，而是由 Kryo 序列化的。在这些类型中，元组类型和 POJO 类型最为灵活，因为它们支持创建复杂类型。而相比之下，POJO 还支持在键（key）的定义中直接使用字段名，这会让我们的代码可读性大大增加。所以，在项目实践中，往往会将流处理程序中的元素类型定为 Flink 的 POJO 类型。 

Flink 对 POJO 类型的要求如下： 

⚫ 类是公共的（public）和独立的（standalone，也就是说没有非静态的内部类）； 

⚫ 类有一个公共的无参构造方法； 

⚫ 类中的所有字段是 public 且非 final 的；或者有一个公共的 getter 和 setter 方法，这些方法需要符合 Java bean 的命名规范。 

所以我们看到，之前的 UserBehavior，就是我们创建的符合 Flink POJO 定义的数据类型。



##### 3. 类型提示（Type Hints）

由于 Java 中泛型擦除的存在，在某些特殊情况下（比如 Lambda 表达式中），为了解决这类问题，Java API 提供了专门的“类型提示”（type hints）

```java
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG));

---------------------------------------------------------------------------
returns(new TypeHint<Tuple2<Integer, SomeType>>(){})
```



### 四、转换算子

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206012345924.png)



#### 1、基本转换算子



##### **1. 映射**（map）

主要用于将数据流中的数据进行转换，形成新的数据流。简单来说，消费一个元素就产出一个元素

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206012359042.png)



我们只需要基于 DataStrema 调用 map()方法就可以进行转换处理。方法需要传入的参数是接口 MapFunction 的实现；返回值类型还是 DataStream

eg：提取 Event 中的 user 字段的功能

```java
	package com.kk.model2;

import com.kk.model2.pojo.Event;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransMapTest {
    public static void main(String[] args) throws Exception {
        // 创造执行环境
        // 并行为 1
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        // 构建数据
        DataStreamSource<Event> stream = env.fromElements (
                new Event ("Mary", "./home", 1000L),
                new Event ("Bob", "./cart", 2000L)
        );

        // 写法一：传入匿名类，实现 MapFunction
        stream.map (new MapFunction<Event, String> ( ) {
            @Override
            public String map(Event e) throws Exception {
                return e.user;
            }
        }).print ( );

        // 写法二： 传入 MapFunction 的实现类
        //stream.map (new UserExtractor ()).print ();

        env.execute ( );

    }

    public static class UserExtractor implements MapFunction<Event, String> {
        @Override
        public String map(Event e) throws Exception {
            return e.user;
        }
    }
}

```

上面代码中，MapFunction 实现类的泛型类型，与输入数据类型和输出数据的类型有关。在实现 MapFunction 接口的时候，需要指定两个泛型，分别是输入事件和输出事件的类型，还需要重写一个 map()方法，定义从一个输入事件转换为另一个输出事件的具体逻辑。 



##### **2. 过滤（filter）**

filter 转换操作，顾名思义是对数据流执行一个过滤，通过一个布尔条件表达式设置过滤条件，对于每一个流内元素进行判断，若为 true 则元素正常输出，若为 false 则元素被过滤掉

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206072208877.png)

```java
package com.kk.model2;

import com.kk.model2.pojo.Event;
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransFilterTest {
    public static void main(String[] args) throws Exception {

        // 创造执行环境
        // 并行为 1
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        // 构建数据
        DataStreamSource<Event> stream = env.fromElements (
                new Event ("Mary", "./home", 1000L),
                new Event ("Bob", "./cart", 2000L)
        );

        // 写法一：传入匿名类实现 FilterFunction
        stream.filter (new FilterFunction<Event> ( ) {
            @Override
            public boolean filter(Event e) throws Exception {
                return e.user.equals ("Bob");
            }
        }).print ( );

        // 写法二：传入 FilterFunction 实现类
        stream.filter (new UserFilter ( )).print ( );

        env.execute ( );
    }

    public static class UserFilter implements FilterFunction<Event> {
        @Override
        public boolean filter(Event e) throws Exception {
            return e.user.equals ("Mary");
        }
    }
}

```

进行 filter 转换之后的新数据流的数据类型与原数据流是相同的。filter 转换需要传入的参数需要实现 FilterFunction 接口，而 FilterFunction 内要实现 filter()方法，就相当于一个返回布尔类型的条件表达式。 



##### **3、扁平映射（flatMap）**

flatMap 操作又称为扁平映射，主要是将数据流中的整体（一般是集合类型）拆分成一个一个的个体使用。消费一个元素，可以产生 0 到多个元素,也就是先按照某种规则对数据进行打散拆分，再对拆分后的元素做转换处理

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206072215902.png)

```java
package com.kk.model2;

import com.kk.model2.pojo.Event;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class TransFlatmapTest {
    public static void main(String[] args) throws Exception {
        // 创造执行环境
        // 并行为 1
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        // 构建数据
        DataStreamSource<Event> stream = env.fromElements (
                new Event ("Mary", "./home", 1000L),
                new Event ("Bob", "./cart", 2000L)
        );

        stream.flatMap (new FlatMapFunction<Event, String> ( ) {
            @Override
            public void flatMap(Event e, Collector<String> collector) throws Exception {
                if (e.user.equals ("Mary")) {
                    collector.collect (e.user);
                } else if (e.user.equals ("Bob")) {
                    collector.collect (e.user);
                    collector.collect (e.url);
                }
            }
        }).print ( );// print () 打印

        // 执行
        env.execute ( );
    }
}
```

flatMap 操作会应用在每一个输入事件上面，FlatMapFunction 接口中定义了 flatMap 方法， 用户可以重写这个方法，在这个方法中对输入数据进行处理，并决定是返回 0 个、1 个或多个结果数据。因此 flatMap 并没有直接定义返回值类型，而是通过一个“收集器”（Collector）来指定输出。希望输出结果时，只要调用收集器的.collect()方法就可以了；这个方法可以多次调用，也可以不调用。所以 flatMap 方法也可以实现 map 方法和 filter 方法的功能，当返回结果是 0 个的时候，就相当于对数据进行了过滤，当返回结果是 1 个的时候，相当于对数据进行了简单的转换操作。 

#### 2、聚合算子

##### 1. 按键分区



keyBy 是聚合前必须要用到的一个算子。keyBy 通过指定键（key），可以将一条流从逻辑上划分成不同的分区（partitions）。这里所说的分区，其实就是并行处理的子任务，也就对应着任务槽

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206072326233.png)

```java
package com.kk.model2;

import com.kk.model2.pojo.Event;
import org.apache.flink.streaming.api.datastream.DataStreamSink;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransKeyByTest {
    public static void main(String[] args) throws Exception {

        // 创造执行环境
        // 并行为 1
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        // 构建数据
        DataStreamSource<Event> stream = env.fromElements (
                new Event ("Mary", "./home", 1000L),
                new Event ("Bob", "./cart", 2000L)
        );

        // lambda
        DataStreamSink<Event> byKeyStream = stream.keyBy (e -> e.user).print ( );

        env.execute ( );

    }
}

```





##### 2. **简单聚合**

⚫ sum()：在输入流上，对指定的字段做叠加求和的操作。 

⚫ min()：在输入流上，对指定的字段求最小值。 

⚫ max()：在输入流上，对指定的字段求最大值。 

⚫ minBy()：与 min()类似，在输入流上针对指定字段求最小值。不同的是，min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值；而 minBy()则会返回包 含字段最小值的整条数据。 

⚫ maxBy()：与 max()类似，在输入流上针对指定字段求最大值。两者区别与 min()/minBy()完全一致。 

```java
package com.kk.model2;

import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransTupleAggreationTest {
    public static void main(String[] args) throws Exception {
        // 创造执行环境
        // 并行为 1
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment ( );
        env.setParallelism (1);

        // 构建数据
        DataStreamSource<Tuple2<String, Integer>> stream = env.fromElements (
                Tuple2.of ("a", 1),
                Tuple2.of ("a", 5),
                Tuple2.of ("a", 3),
                Tuple2.of ("b", 4),
                Tuple2.of ("b", 5),
                Tuple2.of ("b", 2)
        );

   //     stream.keyBy(r -> r.f0).sum(1).print("SUM:");
//        stream.keyBy(r -> r.f0).sum("f1").print();
        stream.keyBy(r -> r.f0).max(1).print("MAX");
//        stream.keyBy(r -> r.f0).max("f1").print();
//        stream.keyBy(r -> r.f0).min(1).print();
//        stream.keyBy(r -> r.f0).min("f1").print();
//        stream.keyBy(r -> r.f0).maxBy(1).print();
//        stream.keyBy(r -> r.f0).maxBy("f1").print();
//        stream.keyBy(r -> r.f0).minBy(1).print();
//        stream.keyBy(r -> r.f0).minBy("f1").print();

        env.execute ();
    }
}

```



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202206082139500.png)

而如果数据流的类型是 POJO 类，那么就只能通过字段名称来指定，不能通过位置来指定了

```java

```



##### 3. **归约聚合（**reduce**）**





#### 3、自定义函数



#### 4、物理分区





### 五、输出算子



#### 1、

#### 2、

#### 3、

#### 4、

#### 5、



## 参考地址 ↓



### 1、docker 加速（博主简书）

url：https://www.jianshu.com/p/f554c85b25c1



### 2、Hadoop 单机安装

url：https://blog.51cto.com/u_15187242/4760802

```docker
docker run -i -t --network host -p 50070:50070 -p 9000:9000 -p 8088:8088 -p 8040:8040 -p 8042:8042 -p 49707:49707 -p 50010:50010 -p 50075:50075 -p 50090:50090 sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash
```



### 3、Hadoop集群

url：https://dhcp.cn/k8s/docker/deploy_hadoop.html#reference





### 4、Hadoop常用端口

url：https://blog.csdn.net/qq_36816848/article/details/113106441



### 5、nc -lk 模拟实时数据

url：https://www.csdn.net/tags/MtzakgwsODA4NjktYmxvZwO0O0OO0O0O.html



### 6、Flink（docker）快速搭建

url：https://blog.csdn.net/weixin_42357472/article/details/118223101



### 7、Zookeeper(docker)快速搭建

拉取url：https://blog.csdn.net/u010416101/article/details/122803105

启动url：https://www.cnblogs.com/shanfeng1000/p/14488665.html



### 8、kafka(docker)快速搭建

搭建url：https://www.cnblogs.com/shanfeng1000/p/14638455.html

使用url：https://blog.csdn.net/qq_22041375/article/details/106180415

