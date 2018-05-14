# CONCEPTS

# 概念

Kafka Streams is a client library for processing and analyzing data stored in Kafka. It builds upon important stream processing concepts such as properly distinguishing between event time and processing time, windowing support, and simple yet efficient management and real-time querying of application state.

Kafka Streams 是一个用于处理和分析存储在Kafka中数据的客户端库。它基于重要的流处理概念，如正确区分事件时间和处理时间，窗口支持以及简单高效的管理和实时查询应用程序状态。

Kafka Streams has a low barrier to entry: You can quickly write and run a small-scale proof-of-concept on a single machine; and you only need to run additional instances of your application on multiple machines to scale up to high-volume production workloads. Kafka Streams transparently handles the load balancing of multiple instances of the same application by leveraging Kafka's parallelism model.

Kafka Streams 的使用门槛低：您可以在单台机器上快速编写和运行小规模的概念证明。而且您只需要在多台机器上运行应用程序的其他实例即可扩展到大规模生产工作负载。通过利用Kafka的并行模型，Kafka Streams 可以透明的处理同一应用程序的多个实例的负载均衡。

Some highlights of Kafka Streams:

下面是 Kafka Streams 的一些亮点：

* Designed as a simpintegratedle and lightweight client library, which can be easily embedded in any Java application and  with any existing packaging, deployment and operational tools that users have for their streaming applications.

* 设计为简单轻量级的客户端库，可轻松嵌入任何JAVA程序中并与用户的流式应用的任何现有包，部署和操作工具集成。

* Has no external dependencies on systems other than Apache Kafka itself as the internal messaging layer; notably, it uses Kafka's partitioning model to horizontally scale processing while maintaining strong ordering guarantees.

* 除了Apache Kafka本身作为内部消息传递层之外，没有外部依赖关系; 值得注意的是，它使用Kafka的分区模型来横向扩展处理，同时保持强有力的顺序保证。

* Supports fault-tolerant local state, which enables very fast and efficient stateful operations like windowed joins and aggregations.

* 支持本地状态容错，可以实现非常快速且高效的有状态操作，如窗口连接和聚合。

* Supports exactly-once processing semantics to guarantee that each record will be processed once and only once even when there is a failure on either Streams clients or Kafka brokers in the middle of processing.

* 支持一次处理语义，即使处理过程中出现任何Streams客户端或Kafka代理失败，也能确保每个记录都将被处理一次且仅处理一次。

* Employs one-record-at-a-time processing to achieve millisecond processing latency, and supports event-time based windowing operations with late arrival of records.

* 采用一次一个记录处理，以实现毫秒处理延迟。并且支持基于事件时间的窗口操作，用于记录延迟到达的记录。

* Offers necessary stream processing primitives, along with a high-level Streams DSL and a low-level Processor API.

* 提供必要的流处理原语，以及高级Streams DSL和低级Processor API。

We first summarize the key concepts of Kafka Streams.

我们首次总结 Kafka Streams 的关键概念。

## Stream Processing Topology

## 流处理拓扑

* A stream is the most important abstraction provided by Kafka Streams: it represents an unbounded, continuously updating data set. A stream is an ordered, replayable, and fault-tolerant sequence of immutable data records, where a data record is defined as a key-value pair.

* 流是Kafka Streams提供的最重要的抽象：它代表一个无限的，不断更新的数据集。 流是有序的，可重放的和容错的不变数据记录序列，其中数据记录被定义为键值对。

* A stream processing application is any program that makes use of the Kafka Streams library. It defines its computational logic through one or more processor topologies, where a processor topology is a graph of stream processors (nodes) that are connected by streams (edges).

* 流处理应用程序是使用Kafka Streams库的任何程序。 它通过一个或多个处理器拓扑来定义其计算逻辑，其中处理器拓扑是通过流（边）连接的流处理器（节点）的图形。

* A stream processor is a node in the processor topology; it represents a processing step to transform data in streams by receiving one input record at a time from its upstream processors in the topology, applying its operation to it, and may subsequently produce one or more output records to its downstream processors.

* 流处理器是处理器拓扑中的一个节点; 它表示一个处理步骤，通过从拓扑中的上游处理器一次接收一个输入记录，将其操作应用于其中，并随后可以向其下游处理器产生一个或多个输出记录，从而变换流中的数据。

There are two special processors in the topology:

在拓扑中有两个特殊的处理器：

* Source Processor: A source processor is a special type of stream processor that does not have any upstream processors. It produces an input stream to its topology from one or multiple Kafka topics by consuming records from these topics and forwarding them to its down-stream processors.

* 源处理器：源处理器是一种特殊类型的流处理器，没有任何上游处理器。 它通过从这些主题中消耗记录并将它们转发到其下游处理器，从一个或多个Kafka主题中为其拓扑生成输入流。

* Sink Processor: A sink processor is a special type of stream processor that does not have down-stream processors. It sends any received records from its up-stream processors to a specified Kafka topic.

* 水槽处理器：宿处理器是一种特殊类型的流处理器，没有下游处理器。 它将来自其上游处理器的所有记录发送到指定的Kafka主题。

Note that in normal processor nodes other remote systems can also be accessed while processing the current record. Therefore the processed results can either be streamed back into Kafka or written to an external system.

请注意，在正常的处理器节点中，其他远程系统也可以在处理当前记录时访问。 因此处理后的结果可以回传到 Kafka 或写入外部系统。

![Kafka Streams Architecture Topology](../../imgs/streams-architecture-topology.jpg)

Kafka Streams offers two ways to define the stream processing topology: the Kafka Streams DSL provides the most common data transformation operations such as map, filter, join and aggregations out of the box; the lower-level Processor API allows developers define and connect custom processors as well as to interact with state stores.

Kafka Streams提供了两种定义流处理拓扑的方法：Kafka Streams DSL提供最常用的数据转换操作，例如地图，过滤器，连接和聚合; 较低级别的处理器API允许开发人员定义和连接定制处理器以及与状态存储进行交互。

A processor topology is merely a logical abstraction for your stream processing code. At runtime, the logical topology is instantiated and replicated inside the application for parallel processing (see Stream Partitions and Tasks for details).

处理器拓扑仅仅是您的流处理代码的逻辑抽象。 在运行时，逻辑拓扑将被实例化并在应用程序内部复制以进行并行处理（有关详细信息，请参阅流分区和任务）。

## Time

## 时间
A critical aspect in stream processing is the notion of time, and how it is modeled and integrated. For example, some operations such as windowing are defined based on time boundaries.

流处理中的一个关键方面是时间概念，以及它如何建模和整合。 例如，某些操作（如窗口）是基于时间边界来定义的。

Common notions of time in streams are:

流中的时间常见概念是：

* Event time - The point in time when an event or data record occurred, i.e. was originally created "at the source". Example: If the event is a geo-location change reported by a GPS sensor in a car, then the associated event-time would be the time when the GPS sensor captured the location change.

* 事件时间 - 发生事件或数据记录的时间点，即最初创建时的“来源”。 示例：如果事件是由汽车中的GPS传感器报告的地理位置变化，则相关的事件时间将是GPS传感器捕获位置变化的时间。

* Processing time - The point in time when the event or data record happens to be processed by the stream processing application, i.e. when the record is being consumed. The processing time may be milliseconds, hours, or days etc. later than the original event time. Example: Imagine an analytics application that reads and processes the geo-location data reported from car sensors to present it to a fleet management dashboard. Here, processing-time in the analytics application might be milliseconds or seconds (e.g. for real-time pipelines based on Apache Kafka and Kafka Streams) or hours (e.g. for batch pipelines based on Apache Hadoop or Apache Spark) after event-time.

* 处理时间 - 恰好由流处理应用程序处理事件或数据记录的时间点，即记录正在被消耗的时间点。 处理时间可能比原始事件时间晚数毫秒，数小时或数天等。 示例：设想一个分析应用程序，该应用程序可读取并处理从汽车传感器报告的地理位置数据，将其呈现给车队管理仪表板。 在这里，分析应用程序中的处理时间可能是毫秒或几秒（例如，基于Apache Kafka和Kafka Streams的实时管道）或几小时（例如，基于Apache Hadoop或Apache Spark的批处理管道）。

* Ingestion time - The point in time when an event or data record is stored in a topic partition by a Kafka broker. The difference to event time is that this ingestion timestamp is generated when the record is appended to the target topic by the Kafka broker, not when the record is created "at the source". The difference to processing time is that processing time is when the stream processing application processes the record. For example, if a record is never processed, there is no notion of processing time for it, but it still has an ingestion time.

* 摄取时间 - 由 kafka 代理将事件或数据记录存储在主题分区中的时间点。 与事件时间的区别在于，当记录被卡夫卡经纪人追加到目标主题时，而不是记录在“源”创建时生成此摄取时间戳。 处理时间的差异在于处理时间是流处理应用程序处理记录的时间。 例如，如果记录从未处理过，没有处理时间的概念，但它仍然有摄取时间。

The choice between event-time and ingestion-time is actually done through the configuration of Kafka (not Kafka Streams): From Kafka 0.10.x onwards, timestamps are automatically embedded into Kafka messages. Depending on Kafka's configuration these timestamps represent event-time or ingestion-time. The respective Kafka configuration setting can be specified on the broker level or per topic. The default timestamp extractor in Kafka Streams will retrieve these embedded timestamps as-is. Hence, the effective time semantics of your application depend on the effective Kafka configuration for these embedded timestamps.

事件时间和摄取时间之间的选择实际上是通过配置Kafka（而非Kafka Streams）完成的：从Kafka 0.10.x开始，时间戳会自动嵌入到Kafka消息中。 根据Kafka的配置，这些时间戳代表事件时间或摄取时间。 可以在代理级别或每个主题上指定相应的Kafka配置设置。 Kafka Streams中的默认时间戳提取器将按原样检索这些嵌入的时间戳。 因此，应用程序的有效时间语义取决于这些嵌入时间戳的有效Kafka配置。

Kafka Streams assigns a timestamp to every data record via the TimestampExtractor interface. These per-record timestamps describe the progress of a stream with regards to time and are leveraged by	time-dependent operations such as window operations. As a result, this time will only advance when a new record arrives at the processor. We call this data-driven time the stream time of the application to differentiate with the wall-clock time when this application is actually executing. Concrete implementations of the TimestampExtractor interface will then provide different semantics to the stream time definition. For example retrieving or computing timestamps based on the actual contents of data records such as an embedded timestamp field to provide event time semantics, and returning the current wall-clock time thereby yield processing time semantics to stream time. Developers can thus enforce different notions of time depending on their business needs.

Kafka Streams通过 `TimestampExtractor`接口为每个数据记录分配一个时间戳。 这些每条记录的时间戳记描述了流的时间方面的进展，并由时间相关的操作（例如窗口操作）利用。 结果，这一次只会在新记录到达处理器时才会推进。 我们将这个数据驱动时间称为应用程序的流时间，以便在应用程序实际执行时与挂钟时间区分开来。 TimestampExtractor接口的具体实现将为流时间定义提供不同的语义。 例如基于数据记录的实际内容（诸如嵌入时间戳字段）来检索或计算时间戳以提供事件时间语义，并且返回当前挂钟时间，从而产生处理时间语义以产生时间流。 因此，开发人员可以根据业务需求实施不同的时间概念。

Finally, whenever a Kafka Streams application writes records to Kafka, then it will also assign timestamps to these new records. The way the timestamps are assigned depends on the context:

最后，无论何时Kafka Streams应用程序向Kafka写入记录，它都会为这些新记录分配时间戳。 时间戳的分配方式取决于上下文：

* When new output records are generated via processing some input record, for example, context.forward() triggered in the process() function call, output record timestamps are inherited from input record timestamps directly.

* 当通过处理某些输入记录生成新输出记录时，例如，在`process()`函数调用中触发`context.forward()`时，输出记录时间戳直接从输入记录时间戳继承。

* When new output records are generated via periodic functions such as Punctuator#punctuate(), the output record timestamp is defined as the current internal time (obtained through context.timestamp()) of the stream task.

* 当通过周期性函数（如`Punctuator＃punctuate()`）生成新的输出记录时，输出记录时间戳被定义为流任务的当前内部时间（通过`context.timestamp()`获取）。

* For aggregations, the timestamp of a resulting aggregate update record will be that of the latest arrived input record that triggered the update.

* 对于聚合，生成的聚合更新记录的时间戳将是触发更新的最新到达输入记录的时间戳记。

## States

## 状态

Some stream processing applications don't require state, which means the processing of a message is independent from the processing of all other messages. However, being able to maintain state opens up many possibilities for sophisticated stream processing applications: you can join input streams, or group and aggregate data records. Many such stateful operators are provided by the Kafka Streams DSL.

某些流处理应用程序不需要状态，这意味着消息的处理与所有其他消息的处理无关。 但是，能够维护状态为复杂的流处理应用程序打开了许多可能性：您可以加入输入流或分组和汇总数据记录。 Kafka Streams DSL提供了许多这种有状态的运营商。

Kafka Streams provides so-called state stores, which can be used by stream processing applications to store and query data. This is an important capability when implementing stateful operations. Every task in Kafka Streams embeds one or more state stores that can be accessed via APIs to store and query data required for processing. These state stores can either be a persistent key-value store, an in-memory hashmap, or another convenient data structure. Kafka Streams offers fault-tolerance and automatic recovery for local state stores.

Kafka Streams提供了所谓的状态存储，流处理应用程序可以使用它来存储和查询数据。 这是实施有状态操作时的一项重要功能。 Kafka Streams中的每个任务都嵌入了一个或多个可通过API访问的状态存储，以存储和查询处理所需的数据。 这些状态存储可以是持久性键值存储，内存中的散列表或其他方便的数据结构。 Kafka Streams为本地州商店提供容错和自动恢复功能。

Kafka Streams allows direct read-only queries of the state stores by methods, threads, processes or applications external to the stream processing application that created the state stores. This is provided through a feature called Interactive Queries. All stores are named and Interactive Queries exposes only the read operations of the underlying implementation.

Kafka Streams允许通过创建状态存储的流处理应用程序外部的方法，线程，进程或应用程序对状态存储进行直接只读查询。 这是通过称为交互式查询的功能提供的。 所有商店都是命名的，交互式查询只公开底层实现的读取操作。


## PROCESSING GUARANTEES

## 处理保证

In stream processing, one of the most frequently asked question is "does my stream processing system guarantee that each record is processed once and only once, even if some failures are encountered in the middle of processing?" Failing to guarantee exactly-once stream processing is a deal-breaker for many applications that cannot tolerate any data-loss or data duplicates, and in that case a batch-oriented framework is usually used in addition to the stream processing pipeline, known as the Lambda Architecture. Prior to 0.11.0.0, Kafka only provides at-least-once delivery guarantees and hence any stream processing systems that leverage it as the backend storage could not guarantee end-to-end exactly-once semantics. In fact, even for those stream processing systems that claim to support exactly-once processing, as long as they are reading from / writing to Kafka as the source / sink, their applications cannot actually guarantee that no duplicates will be generated throughout the pipeline. Since the 0.11.0.0 release, Kafka has added support to allow its producers to send messages to different topic partitions in a transactional and idempotent manner, and Kafka Streams has hence added the end-to-end exactly-once processing semantics by leveraging these features. More specifically, it guarantees that for any record read from the source Kafka topics, its processing results will be reflected exactly once in the output Kafka topic as well as in the state stores for stateful operations. Note the key difference between Kafka Streams end-to-end exactly-once guarantee with other stream processing frameworks' claimed guarantees is that Kafka Streams tightly integrates with the underlying Kafka storage system and ensure that commits on the input topic offsets, updates on the state stores, and writes to the output topics will be completed atomically instead of treating Kafka as an external system that may have side-effects. To read more details on how this is done inside Kafka Streams, readers are recommended to read KIP-129. In order to achieve exactly-once semantics when running Kafka Streams applications, users can simply set the processing.guarantee config value to exactly_once (default value is at_least_once). More details can be found in the Kafka Streams Configs section.

在流处理中，最常见的问题之一是“我的流处理系统是否保证每个记录只处理一次，即使在处理过程中遇到一些故障？”对于许多不能容忍任何数据丢失或数据重复的应用程序来说，无法确切地保证流处理是一种破坏行为，在这种情况下，除了流处理管道之外，通常还会使用面向批处理的框架，称为Lambda架构。在0.11.0.0之前，Kafka仅提供至少一次的传递保证，因此任何利用它作为后端存储的流处理系统都不能保证端到端完全一次的语义。事实上，即使对于那些声称只支持一次处理的流处理系统，只要他们正在读/写Kafka作为源/汇，它们的应用程序实际上不能保证在整个流水线中不会产生重复。自0.11.0.0发布以来，Kafka增加了支持，允许其制作者以事务和幂等方式向不同的主题分区发送消息，因此Kafka Streams通过利用这些功能添加了端到端的精确一次处理语义。更具体地说，它保证对于从源卡夫卡主题读取的任何记录，其处理结果将在卡夫卡输出主题以及状态存储区中反映一次。请注意，Kafka Streams端到端之间的主要区别在于，一次保证与其他流处理框架保证声称的保证是Kafka Streams与底层Kafka存储系统紧密集成，并确保提交输入主题偏移量，更新状态存储和写入输出主题将以原子方式完成，而不是将Kafka视为可能有副作用的外部系统。要详细了解如何在Kafka Streams内完成此操作，建议读者阅读KIP-129。为了在运行Kafka Streams应用程序时实现恰好一次的语义，用户可以简单地将processing.guarantee配置值设置为exactly_once（默认值是at_least_once）。更多细节可以在Kafka Streams Configs部分找到。






