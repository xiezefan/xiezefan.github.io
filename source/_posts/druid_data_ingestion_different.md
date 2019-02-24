---
layout: post
title: "Druid 几种数据摄入方式的区别"
date: 2018-04-06
categories:
- druid
tags:
- druid
---


我们在从Kafka，RabbitMQ，Storm 中摄入实时数据流时到Druid的时候，可以使用Realtime Node，Index Server，Tranquility进行数据摄入。

本文主要探索这几种数据摄入方式的区别。

<!-- more -->


## Realtime Node

Realtime Node 可以直接配置Firehose从Kafka，RabbitMQ等消息队列中获取数据，数据一旦被摄入，很快就可以被查询到， 同时Realtime Node还会周期性的将摄入的数据合并成Segment，提交给DeepStorage并交由Historical Node加载，以达到提供实时数据的查询的功能。

### Realtime Node的局限性

但这种模式有一些缺陷

#### Kafka 摄入缺陷

一旦Realtime Node宕机，那么该节点上未提交的数据将全部丢失。 

正常这种时候我们需要使用创建 replica 副本以保证高可用，但在使用Kafka摄入数据的场景会有一些缺陷， 这里涉及到Segment的[Shard机制](http://druid.io/docs/0.12.0/ingestion/stream-pull.html#sharding)。

简单讲，在大量数据的摄入的场景，通常一个Segment会分为多个Shard分片，每一个分片会有不同的`partitionNum `分区号。当设置了副本的时候，某个分片的副本将会拥有该分片相同的分区号。在数据查询的时候，Druid在相同分区号的分片中知会随机选择一个进行数据查询数据。

而举一个具体的场景，假如在你有一个Kafka的Topic有编号为1，2，3的partitions，并且你有2个实时节点去消费这个Topic，节点1消费了partitions 1与2， 节点2消费了partitions 3。他们拥有相同的Group。
这时候你需要进行Replication以保证高可用，所以你启动了另外2个实时节点并以新的Kafka Topic去消费数据。 这时候看节点3可能消费了partitons 1，节点4可能消费了partitions 2与3。
前文提到Segment partitionNum查询的特性，这时候Druid会认为节点3与4是同一个分区，所以数据查询只会从中挑选一个节点进行查询，但很明显他们数据是不一样的。

![Druid Realtime Node缺陷](http://pics.xiezefan.me/Druid Realtime Node defects.png)


#### 数据丢失风险

另外，假设你当前正在执行一个任务计算今日每个小时的数据汇总，同时你有一个批处理任务从离线集群获取数据重做Segment，并覆盖当前的Segment。
如果你在执行计算任务期间发生了Segment覆盖，那么在这个场景下很可能出现数据丢失。

#### Schema更新需重启节点

在Schema更新的时候，你需要重启Realtime Node以生效新的配置，这在大规模集群管理上效率非常低。

## Indexing Service

以下是Druid官方的Indexing Service架构图

![Druid Indexing Service Architecture](http://pics.xiezefan.me/indexing_service.png)

Indexing Service分为以下几个部分:

1. Overload : 负责接收请求，分配任务给Middle Manager执行
2. Middle Manager :  执行提交的任务的工作者节点，会将任务分发给运行在单独的JVM的Peon
3. Peon : 指定Task的容器
4. Task : 在Middle Manager管理的Peon下执行的任务，支持各种各样的任务如Hadoop摄入，Kafka摄入。同时开发者可以自己定制Task以满足业务需求
5. ZooKeeper: 维护任务的信息

简单讲Indexing Service是一套分布式任务调度系统，按计划执行数据摄入，Segment合并，压缩，删除等任务。同时支持自定义任务扩展。

甚至你可以通过扩展将Indexing Service接入Autoscaling自动伸缩服务，在任务排队过长的时候，自动创建Middle Manager节点，再节点空闲的时候，自动进行缩容。

### 锁机制

[Task的锁机制](http://druid.io/docs/0.12.0/ingestion/tasks.html#locking)可以在我们进行publish segment等时候获取指定时间段的segment的排他锁或共享锁，以避免上文描述的Realtime Node Segment覆盖的时候可能产生的数据丢失风险

### Kafka Indexing Service

[Kafka Indexing Service](http://druid.io/docs/0.12.0/development/extensions-core/kafka-ingestion.html) 是在Indexing Service上封装的一个扩展，使用Kafka自己的分区和偏移机制来读取数据，因此能够提供准确一次的服务保证。 解决Realtime Node摄入Kafka数据的缺陷。

与 Tranquility 对比的好处是，他能够读取来自Kafka的非近期的数据，并且不受窗口期(window period )对其他摄取机制的影响。支持管理索引任务的状态，以协调切换，管理故障，并确保维护可扩展性和复制要求。

截止至当前Druid 0.11.0 Kafka Indexing server还未完全Release，需要谨慎在生产上使用。

## Tranquility
Tranquility 是在 Index Service 封装的一个类库，帮助我们从Kafka，Hadoop，HTTP，Storm，Samza，Spark Streaming或者自己的JVM程序发送数据给Indexing Service。

### Tranquility 实现原理

Tranquility 帮助我们解决Indexing Service在Partitioning分区， Replication副本，Service Discover服务发现，Schema rollover Schema平滑变更上的一些难题。

Tranquility基于Indexing Service的EventReceiverFirehose来实现，该Firehose会暴露一个HTTP API供Tranquility实时Push数据给Indexing Service。

在解决分区与副本问题上，Tranquility会给每一个Segment+ partitionNum指派一个Task来进行数据摄入，在超过Window Period后，Task将会停止接收数据并合并Segment，然后提交到Deep Storage。

而副本的实现上，Tranquility将每个副本创建不同Task，相同的Segment与partitionNum，相同Segment与partitionNum的数据将会同事发给相同的副本的Task，副本的Task之间乎不通讯。

当Schema发生改变的时候，在Indexing Service必须重启任务才生效。 而Tranquility的做法是只会应用到新的时间段生成的Segment，旧的Segment讲保持不变。由此时间不停机修改Schema。

最后，我们通常需要启动实例进行数据处理并发送给Druid，Tranquility使用Zookeeper管理不同实例的任务协调，保证数据被正确想到相应的Task中。

### Tranquility的缺陷

1. 超过Window Period的数据将会被丢弃，必须定时从离线集群中重做Segment来实现数据互补。
2. Tranquility与Indexing Service的通讯异常会导致重试，无论是重试成功或失败，都可能导致数据丢失或者重复数据

## 总结

1. Kafka Indexing Service 与 Tranquility 是官方推荐的安全的Kafka摄入数据的方式。
2. 更多使用Indexing Service起替代Realtime Node



