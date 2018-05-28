# Apache Kafka® is a distributed streaming platform. What exactly does that mean?

# Apache Kafka® 作为一个分布式的流式平台，这到底意味着什么？

## A streaming platform has three key capabilities:

## 一个流平台有三个关键要素：

* Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system.

    发布和订阅记录流，类似于消息队列或企业消息系统

* Store streams of records in a fault-tolerant durable way.

    存储记录流并能容错

* Process streams of records as they occur.

    即时的处理记录流

## Kafka is generally used for two broad classes of applications:

## Kafka主要应用在两类场景:

* Building real-time streaming data pipelines that reliably get data between systems or applications

    建立实时流数据管道，可靠地在系统和应用之间获取传输数据

* Building real-time streaming applications that transform or react to the streams of data

    建立实时流应用，能传输和响应数据流
    
To understand how Kafka does these things, let's dive in and explore Kafka's capabilities from the bottom up.

要了解Kafka如何做到这些，让我们从底层开始深入分析Kafka。

## First a few concepts:

## 首先是一些相关概念:

* Kafka is run as a cluster on one or more servers that can span multiple datacenters.

    Kafka 运行在单个或者多个服务器集群中，并且可以跨多个数据中心。

* The Kafka cluster stores streams of records in categories called topics.

    Kafka 集群按照主题存储数据流记录（record）

* Each record consists of a key, a value, and a timestamp.

    每一个数据流记录（record）包括了一个key(键),value(值)和一个timestamp(时间戳)

## Kafka has four core APIs:

## Kafka 有4个核心APIs:

* The [Producer API](https://kafka.apache.org/documentation.html#producerapi) allows an application to publish a stream of records to one or more Kafka topics.

    [Producer API](./documentations/apis/producer.md)(生产者API)允许应用发布一个流记录到一个或多个Kafka主题中。

* The [Consumer API](https://kafka.apache.org/documentation.html#consumerapi) allows an application to subscribe to one or more topics and process the stream of records produced to them.

    [Consumer API](./documentations/apis/consumer.md)(消费者API)允许应用订阅一个或多个主题并处理主题中的流记录(records)。

* The [Streams API](https://kafka.apache.org/documentation/streams) allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.

    [Streams API](./documentations/apis/streams.md)允许一个应用表现为一个流处理器，消费从一个或多个主题得到的输入流，并产生一个输出流到一个或多个输出主题，即能有效的把输入流转变为输出流。

* The [Connector API](https://kafka.apache.org/documentation.html#connect) allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table.

    [Connector API](./documentations/apis/connect.md)允许构建并运行可重复使用的生产者或消费者,它们可以把Kafka的主题链接到已存在的应用或数据系统中。例如，一个链接到关系数据库的Kafka主题可能会捕获数据库表的任意变化。

<div align="center">
<img src="imgs/Kafka-apis.png" height="400" width="530" >
</div>


In Kafka the communication between the clients and the servers is done with a simple, high-performance, language agnostic [TCP protocol](https://kafka.apache.org/protocol.html). This protocol is versioned and maintains backwards compatibility with older version. We provide a Java client for Kafka, but clients are available in [many languages](https://cwiki.apache.org/confluence/display/KAFKA/Clients).

Kafka客户端和服务端之间的通信是建立在简单的、高效的、语言无关的[TCP协议](https://kafka.apache.org/protocol.html)上的。此协议带有版本且向后兼容。我们为Kafka提供了Java客户端，但是客户端可以使用[多种语言](https://cwiki.apache.org/confluence/display/KAFKA/Clients)。


## Topics and Logs

## 主题和日志

Let's first dive into the core abstraction Kafka provides for a stream of records—the topic.

让我们首先深入Kafka对于消息流的核心抽象概念-主题。

A topic is a category or feed name to which records are published. Topics in Kafka are always multi-subscriber; that is, a topic can have zero, one, or many consumers that subscribe to the data written to it.

一个主题是一个数据流记录（records）的提供者。Kafka中的主题一般是多订阅者的，即一个主题可以有0个，1个，多个消费者订阅。

For each topic, the Kafka cluster maintains a partitioned log that looks like this:

对每一个主题, Kafka 集群维护如下的一个分区日志：

<div align="center">
<img src="./imgs/log_anatomy.png" height="330" width="530" >
</div>

Each partition is an ordered, immutable sequence of records that is continually appended to—a structured commit log. The records in the partitions are each assigned a sequential id number called the offset that uniquely identifies each record within the partition.

每个分区是一个有序，不可改变的序列消息，它被不断追加一种结构化的操作日志。分区的消息都分配了一个连续的id号叫做偏移量，在每个分区中该偏移量都是唯一的。

The Kafka cluster durably persists all published records—whether or not they have been consumed—using a configurable retention period. For example, if the retention policy is set to two days, then for the two days after a record is published, it is available for consumption, after which it will be discarded to free up space. Kafka's performance is effectively constant with respect to data size so storing data for a long time is not a problem.

Kafka 集群保存所有已经发布出去的消息，直到它们过期，不论消息是否已经被消费掉。例如，如果保存的策略设置为两天，那么消息发布出去的两天内可以消费，两天之后，这些消息将被丢弃以腾出空间。Kafka的性能是常数级别的，不论数据大小，所以能对数据存储很长一段时间。

<div align="center">
<img src="./imgs/log_consumer.png" height="330" width="530" >
</div>


In fact, the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log. This offset is controlled by the consumer: normally a consumer will advance its offset linearly as it reads records, but, in fact, since the position is controlled by the consumer it can consume records in any order it likes. For example a consumer can reset to an older offset to reprocess data from the past or skip ahead to the most recent record and start consuming from "now".

实际上，每个消费者所持有的唯一元数据就是每个消费的日志偏移或具体位置。这个偏移量由消费者控制：通常当消费者读取消息后会线性的增加他的偏移量。但是，事实上，由于位置是由消费者控制的，消费者是可以在任何次序消费消息的。例如，一个消费者可以重置偏移量以便重新处理历史数据，或者是直接跳到最新的消息然后从最新的消息开始消费。

This combination of features means that Kafka consumers are very cheap—they can come and go without much impact on the cluster or on other consumers. For example, you can use our command line tools to "tail" the contents of any topic without changing what is consumed by any existing consumers.

Kafka的这些特性意味着Kafka消费者可以操作自如，方便的加入或者离开而不会对集群或者其它的消费者造成很大影响。例如，可以使用命令行工具去追踪任何主题的内容而不会改变这些内容，即使这些内容被其它消费者消费。

The partitions in the log serve several purposes. First, they allow the log to scale beyond a size that will fit on a single server. Each individual partition must fit on the servers that host it, but a topic may have many partitions so it can handle an arbitrary amount of data. Second they act as the unit of parallelism—more on that in a bit.

日志划分分区有多个目的。第一，可以处理更多的消息，不受单台服务器的限制。每一个分区会适应服务器的大小，一个topic可能会有多个分区，所以Kafka可以处理任意大小的数据。第二，分区可以作为并行处理的单元。

## Distribution

## 分布式

The partitions of the log are distributed over the servers in the Kafka cluster with each server handling data and requests for a share of the partitions. Each partition is replicated across a configurable number of servers for fault tolerance.

每个日志分区被分布在Kafka集群服务器上，每个服务器都能处理分区的数据和请求。根据配置每个分区还可以复制到其他服务器作为容错。

Each partition has one server which acts as the "leader" and zero or more servers which act as "followers". The leader handles all read and write requests for the partition while the followers passively replicate the leader. If the leader fails, one of the followers will automatically become the new leader. Each server acts as a leader for some of its partitions and a follower for others so load is well balanced within the cluster.

每个分区都有一个服务器充当“leader”，零个或多个服务器充当“follower”。leader处理对分区所有的读写请求，follower就会被动复制这个leader。如果leader宕机，其中一个follower会被推举为新的leader。一台服务器可以同时是一个分区的leader，另一个分区的follower，这样可以平衡负载，避免所有请求都只让一台或者某几台服务器处理。

## Geo-Replication

## 异地数据同步技术

Kafka MirrorMaker provides geo-replication support for your clusters. With MirrorMaker, messages are replicated across multiple datacenters or cloud regions. You can use this in active/passive scenarios for backup and recovery; or in active/active scenarios to place data closer to your users, or support data locality requirements.

Kafka MirrorMaker为集群提供异地数据同步支持。通过MirrorMaker，消息可以跨多个数据中心或者云区域进行复制。您可以在active/passive场景中进行备份或恢复；或者在active/passive方案中将数据置于更接近用户的位置，或者支持数据本地化。

## Producer

## 生产者

Producers publish data to the topics of their choice. The producer is responsible for choosing which record to assign to which partition within the topic. This can be done in a round-robin fashion simply to balance load or it can be done according to some semantic partition function (say based on some key in the record). More on the use of partitioning in a second!

生产者向所选的主题上发布消息。生产者负责选择哪个消息分配到指定主题的哪个分区中。最简单的方式是从分区列表中轮流选择，或可以根据一些语义分区函数来确定记录到哪个分区上（例如根据消息中的key进行划分）。

## Consumers

## 消费者 

Consumers label themselves with a consumer group name, and each record published to a topic is delivered to one consumer instance within each subscribing consumer group. Consumer instances can be in separate processes or on separate machines.

消费者通过消费者组名称标识它们自己，一个被发布到主题上的消息被分发给此消费者组中的一个消费者。消费者实例可以在单独的进程或者是服务器上。

If all the consumer instances have the same consumer group, then the records will effectively be load balanced over the consumer instances.

如果所有的消费者实例都在同一个消费者组中，那么消息将会有效地负载平衡给这些消费者实例，此时消息模型就成为了队列模型。

If all the consumer instances have different consumer groups, then each record will be broadcast to all the consumer processes.

如果所有的消费者实例在不同的消费者组中，那么每一条消息将会被广播给所有的消费者处理，此时消息模型变成了发布-订阅模型。


<div align="center">
<img src="./imgs/consumer-groups.png" height="300" width="520" >
</div>

A two server Kafka cluster hosting four partitions (P0-P3) with two consumer groups. Consumer group A has two consumer instances and group B has four.

如上图，两台服务器构成一个Kafka集群，并且托管4个分区（P0-P3），并且有两个消费者组，group A 有两个消费者实例， group B 有 四个消费者实例。

More commonly, however, we have found that topics have a small number of consumer groups, one for each "logical subscriber". Each group is composed of many consumer instances for scalability and fault tolerance. This is nothing more than publish-subscribe semantics where the subscriber is a cluster of consumers instead of a single process.

更常见的是，主题会有一个小规模的消费者群，每一个从逻辑上称之为订阅者。每个消费者群有很多消费者实例来保证可扩展性和容错性。这可以称为是发布-订阅模式，其中订阅方是一个消费者集群而不仅仅是一个单一的进程。

The way consumption is implemented in Kafka is by dividing up the partitions in the log over the consumer instances so that each instance is the exclusive consumer of a "fair share" of partitions at any point in time. This process of maintaining membership in the group is handled by the Kafka protocol dynamically. If new instances join the group they will take over some partitions from other members of the group; if an instance dies, its partitions will be distributed to the remaining instances.

Kafka中消费的实现方式是“公平”的将分区分配给消费者，每一个时刻分区都拥有它唯一的消费者。消费者组内的成员关系由Kafka协议动态维护。如果新的消费者加入了消费者组，那么它会从这个组其他的消费者中抽走一部分分区；如果部分消费者实例宕机，它的分区会被其他消费者实例接管。

Kafka only provides a total order over records within a partition, not between different partitions in a topic. Per-partition ordering combined with the ability to partition data by key is sufficient for most applications. However, if you require a total order over records this can be achieved with a topic that has only one partition, though this will mean only one consumer process per consumer group.

Kafka只保证同一个分区内消息的顺序，而不能确保同一个主题的不同分区间数据的顺序。因为主题分区中的消息只能由消费者组中的唯一一个消费者进行处理，所以消息肯定是按照先后顺序进行处理的。但是它也仅仅是保证主题的一个分区顺序处理，不能保证跨分区的消息先后进行处理。如果需要按顺序处理所有的消息，可以使用只有一个分区的主题，这意味着每个消费者组只能有一个消费者实例。

## Multi-tenancy

## 多租户架构

You can deploy Kafka as a multi-tenant solution. Multi-tenancy is enabled by configuring which topics can produce or consume data. There is also operations support for quotas. Administrators can define and enforce quotas on requests to control the broker resources that are used by clients. For more information, see the [security documentation](https://kafka.apache.org/documentation/#security).

您可以将Kafka部署为一个多租户方案。其实现是将主题配置为既可以生产又可以消费数据。此外，还对指标进行业务支持。管理员可以定义或者请求指标来控制客户端使用的代理资源。进一步的了解，可以访问[安全手册](https://kafka.apache.org/documentation/#security)。

## Guarantees

## 保证

At a high-level Kafka gives the following guarantees:

在一个高效的Kafka中必须有如下保证.
　　
* Messages sent by a producer to a particular topic partition will be appended in the order they are sent. That is, if a record M1 is sent by the same producer as a record M2, and M1 is sent first, then M1 will have a lower offset than M2 and appear earlier in the log.

    消息被生产者发送到一个特定的主题分区，消息将以发送的顺序追加到这个分区上面。比如，如果M1和M2消息都被同一个消费者发送，M1先发送，M1的偏移量将比M2的小，并且优先出现在日志中。

* A consumer instance sees records in the order they are stored in the log.

    一个消费者实例按照存储在日志上的顺序获取消息。

* For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any records committed to the log. 

    对一个备份因子是N的主题，我们可以容忍 N-1 个服务器发生宕机，而不会丢失任何已经提交到日志中的消息。

More details on these guarantees are given in the design section of the documentation.

有关这些保证的更多详细信息在文档的设计部分给出了。

## Kafka as a Messaging System

## Kafka作为一个消息系统

How does Kafka's notion of streams compare to a traditional enterprise messaging system?

Kafka的流(streams)与传统的企业消息系统相比如何？

Messaging traditionally has two models: [queuing](http://en.wikipedia.org/wiki/Message_queue) and [publish-subscribe](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). In a queue, a pool of consumers may read from a server and each record goes to one of them; in publish-subscribe the record is broadcast to all consumers. Each of these two models has a strength and a weakness. The strength of queuing is that it allows you to divide up the processing of data over multiple consumer instances, which lets you scale your processing. Unfortunately, queues aren't multi-subscriber—once one process reads the data it's gone. Publish-subscribe allows you broadcast data to multiple processes, but has no way of scaling processing since every message goes to every subscriber.

传统的消息有两种模型：[队列](http://en.wikipedia.org/wiki/Message_queue) 和 [发布/订阅](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)。 对于队列模型，可能有大批量的消费者从一台服务器上读取数据，并且服务器上的每一个消息都会被这些消费者读取。在发布/订阅模型中，消息会广播给所有用户。 这两个模型都有其优缺点。队列的优点是允许把数据处理分发给很多消费者实例，这将能扩大数据处理规模。但是队列不是多订阅者处理的，一旦一个进程读完，数据就消失了。发布/订阅模型允许数据广播到多个进程，但是由于所有的数据都是广播到所有的订阅者的，所以无法扩大数据处理规模。

The consumer group concept in Kafka generalizes these two concepts. As with a queue the consumer group allows you to divide up processing over a collection of processes (the members of the consumer group). As with publish-subscribe, Kafka allows you to broadcast messages to multiple consumer groups.

Kafka中的消费者组有两个新概念：数据处理可以通过消费者组分发给多个进程（即消费者组中的多个消费者进程，如同队列模型一样）。同时如同发布/订阅模型一样，Kafka允许广播消息到多个消费者组中。

The advantage of Kafka's model is that every topic has both these properties—it can scale processing and is also multi-subscriber—there is no need to choose one or the other.

Kafka 兼容了 队列 和 发布/订阅 两个模型中的优势：数据规模扩展 和 数据广播。

Kafka has stronger ordering guarantees than a traditional messaging system, too.

Kafka与传统的消息模型相比，其能更好的确保消息的顺序。

A traditional queue retains records in-order on the server, and if multiple consumers consume from the queue then the server hands out records in the order they are stored. However, although the server hands out records in order, the records are delivered asynchronously to consumers, so they may arrive out of order on different consumers. This effectively means the ordering of the records is lost in the presence of parallel consumption. Messaging systems often work around this by having a notion of "exclusive consumer" that allows only one process to consume from a queue, but of course this means that there is no parallelism in processing.

传统的队列在服务器上按次序的保存消息，如果很多消费者一起从队列中消费数据，服务器将按其存储的数据次序进行处理。然而，尽管服务器能按次序处理，消息是异步到达消费者的，所以对于某个消费者将不会确保数据的有序。这意味着对于并发消费的场景，消息会出现丢失现象。 所以消息系统通常设定一个排他的消费者进程进行消费，而不允许其它消费者进行消费，当然这意味着这种场景将不再是并发的。

Kafka does it better. By having a notion of parallelism—the partition—within the topics, Kafka is able to provide both ordering guarantees and load balancing over a pool of consumer processes. This is achieved by assigning the partitions in the topic to the consumers in the consumer group so that each partition is consumed by exactly one consumer in the group. By doing this we ensure that the consumer is the only reader of that partition and consumes the data in order. Since there are many partitions this still balances the load over many consumer instances. Note however that there cannot be more consumer instances in a consumer group than partitions.

Kafka在这方面做的更好。通过在主题内部使用一个并行机制-即分区, Kafka提供了顺序保证和负载均衡。每个分区仅由同一个消费者组中的一个消费者消费到。并确保消费者是该分区的唯一消费者，并按顺序消费数据。每个主题有多个分区，则需要对多个消费者做负载均衡，但请注意，相同的消费者组中不能有比分区更多的消费者，否则多出的消费者一直处于空等待，不会收到消息。

## Kafka as a Storage System

## Kafka作为一个存储系统

Any message queue that allows publishing messages decoupled from consuming them is effectively acting as a storage system for the in-flight messages. What is different about Kafka is that it is a very good storage system.

只要能从其消费的消息中分离出能发布的消息，任何一个消息队列都可以有效的充当一个动态消息的存储系统。与之不同的是，Kafka是一个非常好的存储系统。

Data written to Kafka is written to disk and replicated for fault-tolerance. Kafka allows producers to wait on acknowledgement so that a write isn't considered complete until it is fully replicated and guaranteed to persist even if the server written to fails.

写入到Kafka的数据会被写入磁盘并备份到集群中以保证容错性。Kafka允许生产者等待消息确认，所以对于一次写入操作，只有当所有数据都写完，才是完整的，并且即使服务器写入失败，Kafka也能确保数据的持久性，

The disk structures Kafka uses scale well—Kafka will perform the same whether you have 50 KB or 50 TB of persistent data on the server.

Kafka的磁盘结构适合处理大规模数据，无论你服务器上的持久化数据是50KB还是50TB，执行性能是相同的。

As a result of taking storage seriously and allowing the clients to control their read position, you can think of Kafka as a kind of special purpose distributed filesystem dedicated to high-performance, low-latency commit log storage, replication, and propagation.

由于Kafka对存储的严格要求，并且允许客户端控制数据的读取位置，所以您可以将Kafka视为一个拥有高性能、低延迟的提交日志存储，备份和传播等特点的特殊的分布式文件系统。

For details about the Kafka's commit log storage and replication design, please read [this](https://kafka.apache.org/documentation/#design) page.

如果想了解更多关于Kafka日志提交存储和副本设计方面的细节，请阅读[此页](./design.md)。

## Kafka for Stream Processing

## Kafka的流处理
It isn't enough to just read, write, and store streams of data, the purpose is to enable real-time processing of streams.

仅仅读、写和存储是不够的，Kafka的目标是实时的流处理。

In Kafka a stream processor is anything that takes continual streams of data from input topics, performs some processing on this input, and produces continual streams of data to output topics.

在Kafka中，流处理持续获取输入主题的数据，进行处理加工，然后写入到输出主题。

For example, a retail application might take in input streams of sales and shipments, and output a stream of reorders and price adjustments computed off this data.

例如，一个零售APP，接收销售和出货的输入流，统计数量或调整价格后输出。

It is possible to do simple processing directly using the producer and consumer APIs. However for more complex transformations Kafka provides a fully integrated Streams API. This allows building applications that do non-trivial processing that compute aggregations off of streams or join streams together.

可以直接使用producer和consumer API进行简单的处理。对于复杂的转换，Kafka提供了更强大的Streams API。可以构建对流进行聚合计算或连接等复杂处理操作的应用。

This facility helps solve the hard problems this type of application faces: handling out-of-order data, reprocessing input as code changes, performing stateful computations, etc.

该流处理工具有助于解决此类应用面临的硬性问题：无序的数据的处理，代码更改后的输入数据的再处理，执行状态的计算等。

The streams API builds on the core primitives Kafka provides: it uses the producer and consumer APIs for input, uses Kafka for stateful storage, and uses the same group mechanism for fault tolerance among the stream processor instances.

Streams API构建在Kafka的核心基元上：使用producer和consumer API作为输入，利用Kafka做状态存储，使用相同的组机制在流处理器实例之间进行容错保障。

## Putting the Pieces Together

## 将碎片化数据整合

This combination of messaging, storage, and stream processing may seem unusual but it is essential to Kafka's role as a streaming platform.

消息传递，存储和流处理的组合看似反常，但对于Kafka作为流式处理平台的作用至关重要。

A distributed file system like HDFS allows storing static files for batch processing. Effectively a system like this allows storing and processing historical data from the past.

像HDFS这样的分布式文件系统允许存储静态文件进行批处理，这样的话系统可以有效地存储和处理历史数据。

A traditional enterprise messaging system allows processing future messages that will arrive after you subscribe. Applications built in this way process future data as it arrives.

传统企业的消息系统允许在你订阅之后处理才到达的消息：以这种方式建立的应用会在这些消息到来时去处理它。

Kafka combines both of these capabilities, and the combination is critical both for Kafka usage as a platform for streaming applications as well as for streaming data pipelines.

Kafka结合了这些优势，这种组合对于Kafka作为流处理应用和流数据管道平台是至关重要的。

By combining storage and low-latency subscriptions, streaming applications can treat both past and future data the same way. That is a single application can process historical, stored data but rather than ending when it reaches the last record it can keep processing as future data arrives. This is a generalized notion of stream processing that subsumes batch processing as well as message-driven applications.

批处理以及消息驱动应用程序的流处理的概念：即通过组合存储和低延迟订阅，流处理应用可以用相同的方式对待过去和还未到达的数据。它是一个单一的应用程序，即它可以处理历史的存储数据，当它处理到最后一个消息时，它进入还未到达消息的等待，而不是结束。

Likewise for streaming data pipelines the combination of subscription to real-time events make it possible to use Kafka for very low-latency pipelines; but the ability to store data reliably make it possible to use it for critical data where the delivery of data must be guaranteed or for integration with offline systems that load data only periodically or may go down for extended periods of time for maintenance. The stream processing facilities make it possible to transform data as it arrives.

同样，对于流数据管道，结合实时事件的订阅，可将Kafka用做低延迟的管道；但是，可靠地存储数据的能力使得它可以用于必须传输的关键数据，或与仅定期加载数据或长时间维护的离线系统集成在一起。流处理工具可以在数据到达时转换它。

For more information on the guarantees, APIs, and capabilities Kafka provides see the rest of the [documentation](https://kafka.apache.org/documentation.html).

关于Kafka提供的保证，APIs，容量等更多信息，请看这个[文档](./documentation.md)。
