# Architecture

# 架构

Kafka Streams simplifies application development by building on the Kafka producer and consumer libraries and leveraging the native capabilities of Kafka to offer data parallelism, distributed coordination, fault tolerance, and operational simplicity. In this section, we describe how Kafka Streams works underneath the covers.

Kafka Streams通过构建Kafka生产者和消费者库并利用Kafka的本地功能提供数据并行性，分布式协调，容错性和操作简单性，简化了应用程序开发。在本节中，我们将介绍Kafka Streams是如何工作的。

The picture below shows the anatomy of an application that uses the Kafka Streams library. Let's walk through some details.

下图显示了使用Kafka Streams库的应用程序的组织结构。我们来看看一些细节。

<div align="center">
<img src="../../imgs/streams-architecture-overview.jpg" height="459" width="375" >
</div>

## Stream Partitions and Tasks

## Stream分区和任务

The messaging layer of Kafka partitions data for storing and transporting it. Kafka Streams partitions data for processing it. In both cases, this partitioning is what enables data locality, elasticity, scalability, high performance, and fault tolerance. Kafka Streams uses the concepts of partitions and tasks as logical units of its parallelism model based on Kafka topic partitions. There are close links between Kafka Streams and Kafka in the context of parallelism:

Kafka 的消息传输层用于储存和运输它分区的数据。Kafka Streams 分区数据用于处理它。 在这两种情况下，这种分区都可以实现数据的局部性，弹性，可扩展性，高性能和容错性。Kafka Streams使用分区和任务的概念作为基于Kafka主题分区的并行模型的逻辑单元。在并行的背景下，Kafka Stream 和 Kafka 之间有着密切的联系：

* Each stream partition is a totally ordered sequence of data records and maps to a Kafka topic partition.

* A data record in the stream maps to a Kafka message from that topic.

* The keys of data records determine the partitioning of data in both Kafka and Kafka Streams, i.e., how data is routed to specific partitions within topics.


* 流中的数据记录映射到该主题的Kafka消息。

* 每个流分区都是完全有序的数据记录序列，并映射到Kafka主题分区。

* 数据记录的 `keys` 决定了Kafka和Kafka Stream中数据的分区，即数据如何路由到主题中的特定分区。

An application's processor topology is scaled by breaking it into multiple tasks. More specifically, Kafka Streams creates a fixed number of tasks based on the input stream partitions for the application, with each task assigned a list of partitions from the input streams (i.e., Kafka topics). The assignment of partitions to tasks never changes so that each task is a fixed unit of parallelism of the application. Tasks can then instantiate their own processor topology based on the assigned partitions; they also maintain a buffer for each of its assigned partitions and process messages one-at-a-time from these record buffers. As a result stream tasks can be processed independently and in parallel without manual intervention.

应用程序的处理器拓扑通过将其分解为多个任务来进行扩展。更具体地说，Kafka Streams根据应用程序的输入流分区创建固定数量的任务，每个任务都分配了来自输入流（即Kafka主题）的分区列表。分配给任务的分区不会改变，因此每个任务都是应用程序的固定并行单元。任务然后可以基于所分配的分区实例化它们自己的处理器拓扑;它们还为每个分配的分区保留一个缓冲区，并从这些记录缓冲区中一次一个地处理消息。因此，流任务可以独立并且平行进行处理，无需人工干预。

It is important to understand that Kafka Streams is not a resource manager, but a library that "runs" anywhere its stream processing application runs. Multiple instances of the application are executed either on the same machine, or spread across multiple machines and tasks can be distributed automatically by the library to those running application instances. The assignment of partitions to tasks never changes; if an application instance fails, all its assigned tasks will be automatically restarted on other instances and continue to consume from the same stream partitions.

弄清Kafka Streams不是一个资源管理器，而是一个库（library）是非常重要的，可以使用该库在任何地方进行流处理应用。应用程序的多个实例可以在同一台机器上执行，也可以分布在多台机器上，任务可以由库自动分配给正在运行的应用程序实例。分配给任务的分区不会改变; 如果应用程序实例失败，则其所有分配的任务将在其他实例上自动重新启动，并继续从相同的流分区中使用。

The following diagram shows two tasks each assigned with one partition of the input streams.

下图显示了两个任务，每个任务分配一个输入流分区。

![Kafka Streams Architecture Tasks](../../imgs/streams-architecture-tasks.jpg)

## Threading Model

## 线程模型

Kafka Streams allows the user to configure the number of threads that the library can use to parallelize processing within an application instance. Each thread can execute one or more tasks with their processor topologies independently. For example, the following diagram shows one stream thread running two stream tasks.

Kafka Streams允许用户配置库可用于在应用程序实例中并行处理的线程数。每个线程都可以独立执行一个或多个具有处理器拓扑的任务。例如，下图显示了一个运行两个流任务的流线程。

![Kafka Streams Architecture Threads](../../imgs/streams-architecture-threads.jpg)

Starting more stream threads or more instances of the application merely amounts to replicating the topology and having it process a different subset of Kafka partitions, effectively parallelizing processing. It is worth noting that there is no shared state amongst the threads, so no inter-thread coordination is necessary. This makes it very simple to run topologies in parallel across the application instances and threads. The assignment of Kafka topic partitions amongst the various stream threads is transparently handled by Kafka Streams leveraging Kafka's coordination functionality.

启动更多流线程或应用程序的更多实例仅仅意味着复制拓扑并使其处理不同的Kafka分区子集，从而有效地并行处理。值得注意的是，线程之间没有共享状态，所以不需要线程间协调。这使得跨应用程序实例和线程并行运行拓扑非常简单。Kafka Streams利用Kafka的协调功能透明地处理各种流线程之间的Kafka主题分区分配。

As we described above, scaling your stream processing application with Kafka Streams is easy: you merely need to start additional instances of your application, and Kafka Streams takes care of distributing partitions amongst tasks that run in the application instances. You can start as many threads of the application as there are input Kafka topic partitions so that, across all running instances of an application, every thread (or rather, the tasks it runs) has at least one input partition to process.

如上所述，使用Kafka Streams扩展流处理应用程序非常简单：您只需启动应用程序的其他实例，而Kafka Streams负责在应用程序实例中运行的任务之间分配分区。您可以启动与输入Kafka主题分区一样多的应用程序线程，以便在应用程序的所有正在运行的实例中，每个线程（或者说它运行的任务）至少有一个要处理的输入分区。

## Local State Stores

## 本地状态存储

Kafka Streams provides so-called state stores, which can be used by stream processing applications to store and query data, which is an important capability when implementing stateful operations. The Kafka Streams DSL, for example, automatically creates and manages such state stores when you are calling stateful operators such as join() or aggregate(), or when you are windowing a stream.

Kafka Streams提供了所谓的状态存储，流处理应用程序可以使用它来存储和查询数据，这是实现有状态操作时的重要功能。例如，Kafka Streams DSL在您调用诸如`join()`或`aggregate()`之类的有状态运算符时，或者当您正在对流进行窗口化时，会自动创建和管理这些状态存储。

Every stream task in a Kafka Streams application may embed one or more local state stores that can be accessed via APIs to store and query data required for processing. Kafka Streams offers fault-tolerance and automatic recovery for such local state stores.

Kafka Streams应用程序中的每个流任务都可以嵌入一个或多个可通过API访问的本地状态存储，以存储和查询处理所需的数据。Kafka Streams为这类本地状态存储提供容错和自动恢复功能。

The following diagram shows two stream tasks with their dedicated local state stores.

下图显示了具有专用本地状态存储的两个流任务。

![Kafka Streams Architecture States](../../imgs/streams-architecture-states.jpg)

## Fault Tolerance

## 容错性

Kafka Streams builds on fault-tolerance capabilities integrated natively within Kafka. Kafka partitions are highly available and replicated; so when stream data is persisted to Kafka it is available even if the application fails and needs to re-process it. Tasks in Kafka Streams leverage the fault-tolerance capability offered by the Kafka consumer client to handle failures. If a task runs on a machine that fails, Kafka Streams automatically restarts the task in one of the remaining running instances of the application.

Kafka Streams基于Kafka本身集成的容错功能。Kafka 分区是高度可用和可复制的; 所以当流数据持久化到Kafka时，即使应用程序失败并需要重新处理它，它也是可用的。Kafka Streams中的任务利用Kafka客户端提供的容错功能来处理故障。如果某个任务在发生故障的计算机上运行，则Kafka Streams会自动在应用程序的其余运行实例之一中重新启动该任务。

In addition, Kafka Streams makes sure that the local state stores are robust to failures, too. For each state store, it maintains a replicated changelog Kafka topic in which it tracks any state updates. These changelog topics are partitioned as well so that each local state store instance, and hence the task accessing the store, has its own dedicated changelog topic partition. Log compaction is enabled on the changelog topics so that old data can be purged safely to prevent the topics from growing indefinitely. If tasks run on a machine that fails and are restarted on another machine, Kafka Streams guarantees to restore their associated state stores to the content before the failure by replaying the corresponding changelog topics prior to resuming the processing on the newly started tasks. As a result, failure handling is completely transparent to the end user.

此外，Kafka Streams也确保本地状态存储对失败具有鲁棒性。对于每个状态存储，它都会维护一个复制的更改日志Kafka主题，用于跟踪任何状态更新。这些更改日志主题也进行了分区，以便每个本地状态存储实例以及因此访问存储的任务都有其自己的专用更改日志主题分区。在更改日志主题上启用日志压缩，以便可以安全地清除旧数据，以防止主题无限增长。如果任务在失败的计算机上运行并在另一台计算机上重新启动，则Kafka Streams保证在恢复对新启动的任务的处理之前，通过重播相应的更改日志主题，将其关联的状态存储恢复到故障发生前的内容。因此，故障处理对最终用户来说是完全透明的。

Note that the cost of task (re)initialization typically depends primarily on the time for restoring the state by replaying the state stores' associated changelog topics. To minimize this restoration time, users can configure their applications to have standby replicas of local states (i.e. fully replicated copies of the state). When a task migration happens, Kafka Streams then attempts to assign a task to an application instance where such a standby replica already exists in order to minimize the task (re)initialization cost. See num.standby.replicas in the Kafka Streams Configs section.

请注意，任务（重新）初始化的成本通常主要取决于通过重播状态存储库的相关更改日志主题来恢复状态的时间。为了最小化这种恢复时间，用户可以将其应用程序配置为具有本地状态的备用副本（即完全复制的状态副本）。当发生任务迁移时，Kafka Streams会尝试将任务分配给已存在备用副本的应用程序实例，以最大程度地减少任务（重新）初始化成本。请参阅Kafka Streams Configs部分中的num.standby.replicas。




