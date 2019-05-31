title: 浅谈Lambda架构和Kappa架构
date: 2019-05-31 21:17:38
tags:
- 存储
- 架构
- 系统设计
- 大数据

---

#数据系统的本质
query = function(all data)，对于数据系统的本质Marz提出了一个简单的公式。在大部分情况，在数据集上执行查询函数立即就能获得结果，但是当数据量到达一定量级时，function则需要变的更加复杂。针对OLTP场景的解决方案通常是shard（pre-sharding和auto-sharding ），针对OLAP场景的解决方案通常则是lambda架构和kappa架构。

#Lambda架构
Lambda架构是一个融合了批处理和近实时计算的大数据处理架构，由Storm的作者Nathan Marz提出，其设计目的在于提供一个能满足大数据系统关键特性的架构，包括高容错、低延迟、可扩展等。其整合离线计算与实时计算，融合不可变性、读写分离和复杂性隔离等原则，可集成Hadoop, Kafka, Spark，Storm等各类大数据组件。目前成为了业界大部分公司的数据处理架构的标配解决方案。
一个典型的lambda架构如下：
![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/lambda.png)
lambda架构分为三层: Batch, speed and serving。

### Batch Layer
Batch Layer的功能是存储数据集，并且在数据集上运行数据处理函数，例如hadoop和MapReduce。

### Speed Layer
Batch Layer可以很好的处理离线数据，但有很多场景数据不断实时生成，并且需要实时查询处理。Speed Layer的功能是处理批处理周期之间的增量实时数据。提供近实时的数据处理和业务快速响应。

### Serving Layer
Serving Layer用于响应用户的查询请求，合并Batch View和Realtime View中的结果数据集到最终的数据集。

这种架构主要面向的场景是逻辑比较复杂同时又希望延迟比较低的异步处理程序，比如搜索引擎、推荐引擎等。
系统从一个流中读取被我们定义为不可变的数据，分别灌入实时系统如Storm和批处理系统如Hadoop，然后各自输出自己的结果，这些结果会在查询端进行合并。当然，这种系统也可有很多变种，比如上图中的Kafka也可替换成其他的分布式队列，Storm也可以替换成其他的流式计算引擎。

###Lambda架构的不足
虽然Lambda架构具有诸多优点，以其成熟稳定的架构成为多数公司的大数据处理架构标配，但是也有一些不足：
+ 开发周期长：针对需求的变化既需要做批处理开发也需要做流式计算开发，重复的开发工作使开发周期更长。
+ 结果不一致：由于批处理和实时计算走的是两个计算框架和计算程序，不同开发人员对业务的理解也可能会不一致，算出的结果往往不同。
+ 存储成本大：由于批处理程序往往会产生很多中间表，造成数据膨胀，加大了存储成本。

#kappa架构
Kappa 架构是LinkedIn的Jay Kreps结合实际经验和个人体会，针对Lambda架构进行深度剖析，提出使用统一的流式计算架构来应对全量历史数据和增量实时数据。Kappa架构的核心思想，包括以下三点：
+ 用Kafka或者类似MQ队列系统收集各种各样的数据，你需要几天的数据量就保存几天。
+ 当需要全量重新计算时，重新起一个流计算实例，从头开始读取数据进行处理，并输出到一个新的结果存储中。
+ 当新的实例做完后，停止老的流计算实例，并把老的一些结果删除。
一个典型的kappa架构如下：
![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/kappa.png)

### Kappa架构的不足
Kappa架构的优点在于将实时和离线代码统一起来，方便维护而且统一了计算框架和数据口径。但是也有相应的不足：
+ 吞吐能力成为海量数据处理的瓶颈。面对海量的历史数据，流式计算的吞吐能力和并发能力往往力不从心，无法满足即时性要求。
+ 更高的服务器成本。相比于批处理，流式计算有着更高的计算成本和存储成本。

#如何选择
从长远看人力成本一定会远大于机器成本，系统的吞吐量可以随着硬件能力的提升和架构设计优化而改善。在大部分领域kappa架构可能是更适合的选择。

参考:
[Big Data Lambda Architecture](http://www.databasetube.com/database/big-data-lambda-architecture/)
[questioning the lambda architecture](https://www.oreilly.com/ideas/questioning-the-lambda-architecture)
