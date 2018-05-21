kafka的一些常用的使用场景

# Messaging （消息）

Kafka works well as a replacement for a more traditional message broker. Message brokers are used for a variety of reasons (to decouple processing from data producers, to buffer unprocessed messages, etc). In comparison to most messaging systems Kafka has better throughput, built-in partitioning, replication, and fault-tolerance which makes it a good solution for large scale message processing applications.

kafka可以很好的作为传统的消息代理的替代品。消息代理被使用的原因很多（例如从数据生成器中进行分离处理，缓冲未处理的数据），与大多数消息系统相比，Kafka具有更好的吞吐量、内置分区、备份和容错功能，这使得它成为大型消息处理应用程序的良好解决方案。

In our experience messaging uses are often comparatively low-throughput, but may require low end-to-end latency and often depend on the strong durability guarantees Kafka provides.

根据我们的经验，消息传递的使用通常是相对较低的吞吐量，但是可能需要较低的端到端延迟，并且常常依赖于Kafka提供的强大的持久性保证。

In this domain Kafka is comparable to traditional messaging systems such as ActiveMQ or RabbitMQ.

在这个领域，Kafka可与传统的消息传递系统(如ActiveMQ或RabbitMQ)相媲美。

# Website Activity Tracking （网站活动追踪）

The original use case for Kafka was to be able to rebuild a user activity tracking pipeline as a set of real-time publish-subscribe feeds. This means site activity (page views, searches, or other actions users may take) is published to central topics with one topic per activity type. These feeds are available for subscription for a range of use cases including real-time processing, real-time monitoring, and loading into Hadoop or offline data warehousing systems for offline processing and reporting.

Kafka最初的用例是能够将用户活动跟踪管道重新构建为一组实时发布订阅源。这意味着站点活动(页面视图、搜索或用户可能采取的其他操作)被发布到中心topics，每个活动类型都有一个topic。这些提要可用于订阅一系列的用例，包括实时处理、实时监控和加载到Hadoop或离线数据仓库系统进行离线处理和报告。

Activity tracking is often very high volume as many activity messages are generated for each user page view.

活动跟踪通的体量通常非常高，因为每个用户页面视图都会生成许多活动消息。

# Metrics（指标，度量）

Kafka is often used for operational monitoring data. This involves aggregating statistics from distributed applications to produce centralized feeds of operational data.

Kafka通常用于操作监控中的数据。这包括从分布式应用程序收集统计数据，以生成操作数据的集中提要。

# Log Aggregation （日志聚合）

Many people use Kafka as a replacement for a log aggregation solution. Log aggregation typically collects physical log files off servers and puts them in a central place (a file server or HDFS perhaps) for processing. Kafka abstracts away the details of files and gives a cleaner abstraction of log or event data as a stream of messages. This allows for lower-latency processing and easier support for multiple data sources and distributed data consumption. In comparison to log-centric systems like Scribe or Flume, Kafka offers equally good performance, stronger durability guarantees due to replication, and much lower end-to-end latency.

很多人使用Kafka作为日志聚合解决方案的替代品。日志聚合通常会从服务器上收集物理日志文件，并将它们放在一个中心位置(可能是文件服务器或HDFS)进行处理。Kafka将文件的细节抽象出来，并将日志或事件数据作为消息流进行更清晰的抽象。这允许低延迟处理和更容易地支持多个数据源和分布式数据消耗。与像Scribe或Flume这样的以日志为中心的系统相比，Kafka因为其的备份策略能提供同样良好的性能，更强的持久性保证，以及更低的端到端延迟。

# Stream Processing （流处理）

Many users of Kafka process data in processing pipelines consisting of multiple stages, where raw input data is consumed from Kafka topics and then aggregated, enriched, or otherwise transformed into new topics for further consumption or follow-up processing. For example, a processing pipeline for recommending news articles might crawl article content from RSS feeds and publish it to an "articles" topic; further processing might normalize or deduplicate this content and published the cleansed article content to a new topic; a final processing stage might attempt to recommend this content to users. Such processing pipelines create graphs of real-time data flows based on the individual topics. Starting in 0.10.0.0, a light-weight but powerful stream processing library called Kafka Streams is available in Apache Kafka to perform such data processing as described above. Apart from Kafka Streams, alternative open source stream processing tools include Apache Storm and Apache Samza.

很多Kafka的使用用户在处理管道中数据中包含多个阶段，其中原始输入数据从Kafka topics中消费而来，然后聚合、富集或转换为新的topic以供进一步消费或后续处理。例如，一条推荐新闻文章的处理管道可以从RSS提要抓取文章内容并将其发布到“articles”topic;进一步的处理可能使该内容规范化或删除，并将已处理的文章内容发布到一个新topic;最后一个处理阶段可能尝试向用户推荐该内容。这样的处理管道基于单个主题会创建出一个实时数据流图。从Kafka的0.10.0.0版本开始，一个轻量级但强大的流处理库(称为Kafka Streams)在Apache Kafka中可用来执行如上所述的数据处理。除了Kafka Streams之外，其他开源流处理工具包括Apache Storm和Apache Samza。

# Event Sourcing （事件采集）

Event sourcing is a style of application design where state changes are logged as a time-ordered sequence of records. Kafka's support for very large stored log data makes it an excellent backend for an application built in this style.

事件采集是一种应用程序设计风格，其状态的变化会根据时间顺序记录下来。Kafka支持大量的日志数据存储，使得它成为以这种风格构建的应用程序的一种优秀后端。


# Commit Log （提交日志）

Kafka can serve as a kind of external commit-log for a distributed system. The log helps replicate data between nodes and acts as a re-syncing mechanism for failed nodes to restore their data. The log compaction feature in Kafka helps support this usage. In this usage Kafka is similar to Apache BookKeeper project.

Kafka可以作为分布式系统的一种外部提交日志。该日志帮助在节点之间备份数据，并充当失败节点的重新同步机制，以恢复它们的数据。Kafka的日志压缩特性有助于支持这种用法。在这个用法中，Kafka类似于Apache BookKeeper项目。

---