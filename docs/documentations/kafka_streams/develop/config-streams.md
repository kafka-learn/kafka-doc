# Configuring a Streams Application

# 配置Streams应用程序

Kafka and Kafka Streams configuration options must be configured before using Streams. You can configure Kafka Streams by specifying parameters in a ```StreamsConfig``` instance.

在使用Streams之前，必须配置Kafka和Kafka Streams配置选项。您可以通过在```StreamsConfig```实例中指定参数来配置Kafka Streams。

1.  Create a ```java.util.Properties``` instance.

1.  创建一个```java.util.Properties```实例。

2.  Set the [parameters](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-required-configs).

2.  设置[参数](config-streams.md)。

3.  Construct a ```StreamsConfig``` instance from the ```Properties``` instance. For example:

3.  从```Properties```实例构造一个```StreamsConfig```实例。 例如：


 ```java
import java.util.Properties;
import org.apache.kafka.streams.StreamsConfig;

Properties settings = new Properties();
// Set a few key parameters
// 设置一些关键参数
settings.put(StreamsConfig.APPLICATION_ID_CONFIG, "my-first-streams-application");
settings.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker1:9092");
// Any further settings
// 进一步设置
settings.put(... , ...);

// Create an instance of StreamsConfig from the Properties instance
// 从Properties实例创建一个StreamsConfig的实例
StreamsConfig config = new StreamsConfig(settings);
```

## CONFIGURATION PARAMETER REFERENCE

## 配置参数参考

This section contains the most common Streams configuration parameters. For a full reference, see the [Streams](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsConfig.html) Javadocs.

本节包含最常见的Streams配置参数。有关完整参考，请参阅[Streams](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsConfig.html) Javadocs。

* Required configuration parameters

* 必选的配置参数

    * application.id

    * bootstrap.servers

* Optional configuration parameters

* 可选的配置参数

    * default.deserialization.exception.handler

    * default.production.exception.handler

    * default.key.serde

    * default.value.serde

    * num.standby.replicas

    * num.stream.threads

    * partition.grouper

    * replication.factor

    * state.dir

    * timestamp.extractor

* Kafka consumers and producer configuration parameters

* Kafka消费者和生产者配置参数

    * Naming

    * Default Values

    * enable.auto.commit

    * rocksdb.config.setter

* Recommended configuration parameters for resiliency

* 推荐的弹性配置参数

    * acks

    * replication.factor

## Required configuration parameters

## 必选的配置参数

Here are the required Streams configuration parameters.

以下是必选的Streams配置参数。

Parameter Name | Importance	| Description | Default Value
--- | --- | --- | ---
application.id | Required |	An identifier for the stream processing application. Must be unique within the Kafka cluster. | None
bootstrap.servers | Required | A list of host/port pairs to use for establishing the initial connection to the Kafka cluster. | None

参数名称 | 重要性 | 描述 | 默认值
--- | --- | --- | ---
application.id | 必选 |	流处理应用程序的标识符。在Kafka集群中必须是唯一的。 | None
bootstrap.servers | 必选 | 用于建立到Kafka集群的初始连接的主机/端口对列表。 | None

### application.id

(Required) The application ID. Each stream processing application must have a unique ID. The same ID must be given to all instances of the application. It is recommended to use only alphanumeric characters, ```.``` (dot), ```-``` (hyphen), and ```_``` (underscore). Examples: ```"hello_world"```, ```"hello_world-v1.0.0"```

（必填）应用程序ID。每个流处理应用程序必须具有唯一的ID。必须为应用程序的所有实例提供相同的ID。建议仅使用字母数字字符```.```（点），```-```（连字符）和```_```（下划线）。例如：```"hello_world"```，```"hello_world-v1.0.0"```

This ID is used in the following places to isolate resources used by the application from others:

此ID用于以下情况中将应用程序使用的资源与其他资源隔离开来：

* As the default Kafka consumer and producer ```client.id``` prefix

* 作为默认的Kafka消费者和生产者```client.id```前缀

* As the Kafka consumer ```group.id``` for coordination

* 作为Kafka消费者```group.id```进行协调

* As the name of the subdirectory in the state directory (cf. ```state.dir```)

* 作为状态目录中的子目录的名称（参看```state.dir```）

* As the prefix of internal Kafka topic names

* 作为内部Kafka主题名称的前缀

Tip:

提示：

When an application is updated, the ```application.id``` should be changed unless you want to reuse the existing data in internal topics and state stores. For example, you could embed the version information within ```application.id```, as ```my-app-v1.0.0``` and ```my-app-v1.0.2```.

更新应用程序时，除非要在内部主题和状态存储器中重新使用现有数据，否则应更改```application.id```。例如，您可以在```application.id```中嵌入版本信息，例如```my-app-v1.0.0```和```my-app-v1.0.2```。

### bootstrap.servers

(Required) The Kafka bootstrap servers. This is the same [setting](http://kafka.apache.org/documentation.html#producerconfigs) that is used by the underlying producer and consumer clients to connect to the Kafka cluster. Example: ```"kafka-broker1:9092,kafka-broker2:9092"```.

（必需）Kafka引导程序服务器。这与基础生产者和消费者客户端用于连接到Kafka集群的[设置](http://kafka.apache.org/documentation.html#producerconfigs)相同。 例如：```"kafka-broker1：9092，kafka-broker2：9092"```。

Tip:

提示：

Kafka Streams applications can only communicate with a single Kafka cluster specified by this config value. Future versions of Kafka Streams will support connecting to different Kafka clusters for reading input streams and writing output streams.

Kafka Streams应用程序只能与由此配置值指定的单个Kafka集群进行通信。未来版本的Kafka Streams将支持连接到不同的Kafka集群，以读取输入流并输出到输出流。

## Optional configuration parameters

## 可选的配置参数

Here are the optional [Streams](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsConfig.html) javadocs, sorted by level of importance:

以下是可选的[Streams](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsConfig.html) javadocs，按重要性排序：

* High: These parameters can have a significant impact on performance. Take care when deciding the values of these parameters.

* 重要：这些参数可能会对性能产生重大影响。在决定这些参数的值时要小心。

* Medium: These parameters can have some impact on performance. Your specific environment will determine how much tuning effort should be focused on these parameters.

* 中等：这些参数可能会对性能产生一些影响。您的具体环境将决定您应该集中在这些参数上做多大的调整。

* Low: These parameters have a less general or less significant impact on performance.

* 低：这些参数对性能影响较小或较不显着。

Parameter Name | Importance | Description |	Default Value
--- | --- | --- | ---
application.server | Low | A host:port pair pointing to an embedded user defined endpoint that can be used for discovering the locations of state stores within a single Kafka Streams application. The value of this must be different for each instance of the application. | the empty string
buffered.records.per.partition | Low | The maximum number of records to buffer per partition. | 1000
cache.max.bytes.buffering | Medium | Maximum number of memory bytes to be used for record caches across all threads. | 10485760 bytes
client.id | Medium | An ID string to pass to the server when making requests. (This setting is passed to the consumer/producer clients used internally by Kafka Streams.) | the empty string
commit.interval.ms | Low | The frequency with which to save the position (offsets in source topics) of tasks. | 30000 milliseconds
default.deserialization.exception.handler | Medium | Exception handling class that implements the ```DeserializationExceptionHandler``` interface. | ```LogAndContinueExceptionHandler```
default.production.exception.handler | Medium | Exception handling class that implements the ```ProductionExceptionHandler``` interface. | ```DefaultProductionExceptionHandler```
key.serde | Medium | Default serializer/deserializer class for record keys, implements the ```Serde``` interface (see also value.serde). | ```Serdes.ByteArray().getClass().getName()```
metric.reporters | Low | A list of classes to use as metrics reporters. | the empty list
metrics.num.samples | Low | The number of samples maintained to compute metrics. | 2
metrics.recording.level | Low | The highest recording level for metrics. | ```INFO```
metrics.sample.window.ms | Low | The window of time a metrics sample is computed over. | 30000 milliseconds
num.standby.replicas | Medium | The number of standby replicas for each task. | 0
num.stream.threads | Medium | The number of threads to execute stream processing. | 1
partition.grouper | Low | Partition grouper class that implements the ```PartitionGrouper``` interface. | See [Partition Grouper](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-partition-grouper)
poll.ms | Low | The amount of time in milliseconds to block waiting for input. | 100 milliseconds
replication.factor | High | The replication factor for changelog topics and repartition topics created by the application. | 1
state.cleanup.delay.ms | Low | The amount of time in milliseconds to wait before deleting state when a partition has migrated. | 6000000 milliseconds
state.dir | High | Directory location for state stores.	 | ```/var/lib/kafka-streams```
timestamp.extractor | Medium | Timestamp extractor class that implements the ```TimestampExtractor``` interface. | See [Timestamp Extractor](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-timestamp-extractor)
value.serde | Medium | Default serializer/deserializer class for record values, implements the ```Serde``` interface (see also key.serde). | ```Serdes.ByteArray().getClass().getName()``` 
windowstore.changelog.additional.retention.ms |	Low | Added to a windows maintainMs to ensure data is not deleted from the log prematurely. Allows for clock drift. | 86400000 milliseconds = 1 day

### default.deserialization.exception.handler

The default deserialization exception handler allows you to manage record exceptions that fail to deserialize. This can be caused by corrupt data, incorrect serialization logic, or unhandled record types. These exception handlers are available:

默认的反序列化异常处理程序允许您管理无法反序列化的消息异常。这可能是由数据损坏，序列化逻辑错误或未处理的消息类型造成的。这些异常处理程序可供选择：

* [LogAndContinueExceptionHandler](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/errors/LogAndContinueExceptionHandler.html): This handler logs the deserialization exception and then signals the processing pipeline to continue processing more records. This log-and-skip strategy allows Kafka Streams to make progress instead of failing if there are records that fail to deserialize.

* [LogAndContinueExceptionHandler](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/errors/LogAndContinueExceptionHandler.html)：该处理程序记录反序列化异常，然后通知处理管道继续处理更多消息。如果存在无法反序列化的消息，此记录和跳过策略允许Kafka Streams继续处理，而不是失效。

* [LogAndFailExceptionHandler](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/errors/LogAndFailExceptionHandler.html). This handler logs the deserialization exception and then signals the processing pipeline to stop processing more records.

* [LogAndFailExceptionHandler](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/errors/LogAndFailExceptionHandler.html).此处理程序记录反序列化异常，然后通知处理管道停止处理更多消息。

### default.production.exception.handler

The default production exception handler allows you to manage exceptions triggered when trying to interact with a broker such as attempting to produce a record that is too large. By default, Kafka provides and uses the [DefaultProductionExceptionHandler](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/errors/DefaultProductionExceptionHandler.html) that always fails when these exceptions occur.

默认生产异常处理程序允许您管理尝试与代理进行交互时触发的异常，例如尝试生产过大的消息。默认情况下，Kafka提供并使用的是在发生这些异常时总是失效的[DefaultProductionExceptionHandler](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/errors/DefaultProductionExceptionHandler.html)。

Each exception handler can return a ```FAIL``` or ```CONTINUE``` depending on the record and the exception thrown. Returning ```FAIL``` will signal that Streams should shut down and ```CONTINUE``` will signal that Streams should ignore the issue and continue processing. If you want to provide an exception handler that always ignores records that are too large, you could implement something like the following:

```java
import java.util.Properties;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.common.errors.RecordTooLargeException;
import org.apache.kafka.streams.errors.ProductionExceptionHandler;
import org.apache.kafka.streams.errors.ProductionExceptionHandler.ProductionExceptionHandlerResponse;
 
class IgnoreRecordTooLargeHandler implements ProductionExceptionHandler {
    public void configure(Map<String, Object> config) {}
 
    public ProductionExceptionHandlerResponse handle(final ProducerRecord<byte[], byte[]> record,
                                                     final Exception exception) {
        if (exception instanceof RecordTooLargeException) {
            return ProductionExceptionHandlerResponse.CONTINUE;
        } else {
            return ProductionExceptionHandlerResponse.FAIL;
        }
    }
}
 
Properties settings = new Properties();
 
// other various kafka streams settings, e.g. bootstrap servers, application id, etc
// 其他各种kafka streams设置，例如：引导程序服务器，应用程序ID等 
 
settings.put(StreamsConfig.DEFAULT_PRODUCTION_EXCEPTION_HANDLER_CLASS_CONFIG,
             IgnoreRecordTooLargeHandler.class);
```

### default.key.serde

The default Serializer/Deserializer class for record keys. Serialization and deserialization in Kafka Streams happens whenever data needs to be materialized, for example:

记录消息键的默认序列化/反序列化类。Kafka Streams的序列化和反序列化发生在数据需要物化时，例如：

* Whenever data is read from or written to a *Kafka topic* (e.g., via the ```StreamsBuilder#stream()``` and ```KStream#to()``` methods).

* 无论何时读取或写入*Kafka主题*的数据（例如，通过```StreamsBuilder＃stream()```和```KStream＃to()```方法）。

* Whenever data is read from or written to a *state store*.

* 无论何时从*状态存储器*中读取或写入数据。

This is discussed in more detail in [Data types and serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes).

这在[数据类型和序列化](datatypes.md)中有更详细的讨论。

### default.value.serde

The default Serializer/Deserializer class for record values. Serialization and deserialization in Kafka Streams happens whenever data needs to be materialized, for example:

记录消息值的默认序列化/反序列化类。Kafka Streams中的序列化和反序列化发生在数据需要物化时，例如：

* Whenever data is read from or written to a *Kafka topic* (e.g., via the ```StreamsBuilder#stream()``` and ```KStream#to()``` methods).

* 无论何时读取或写入*Kafka主题*的数据（例如，通过```StreamsBuilder＃stream()```和```KStream＃to()```方法）。

* Whenever data is read from or written to a *state store*.

* 无论何时从*状态存储器*中读取或写入数据。

This is discussed in more detail in [Data types and serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes).

这在[数据类型和序列化](datatypes.md)中有更详细的讨论。

### num.standby.replicas

The number of standby replicas. Standby replicas are shadow copies of local state stores. Kafka Streams attempts to create the specified number of replicas and keep them up to date as long as there are enough instances running. Standby replicas are used to minimize the latency of task failover. A task that was previously running on a failed instance is preferred to restart on an instance that has standby replicas so that the local state store restoration process from its changelog can be minimized. Details about how Kafka Streams makes use of the standby replicas to minimize the cost of resuming tasks on failover can be found in the [State](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-state) section.

备用副本的数量。备用副本是本地状态存储器的影子副本。只要有足够的实例在运行，Kafka Streams就会尝试创建指定数量的副本并保持最新状态。备用副本用于最小化任务故障转移的延迟。先前在失效实例上运行的任务优先于具有备用副本的实例重新启动，以便将本地状态存储器从更新日志中恢复的过程最小化。有关Kafka Streams如何利用备用副本来最大限度地减少故障恢复时恢复任务的成本的详细信息，请参阅[State](../architecture.md)部分。

### num.stream.threads

This specifies the number of stream threads in an instance of the Kafka Streams application. The stream processing code runs in these thread. For more information about Kafka Streams threading model, see [Threading Model](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-threads).

这指定了Kafka Streams应用程序实例中的流线程数。流处理代码在这些线程中运行。有关Kafka Streams线程模型的更多信息，请参阅[线程模型](../architecture.md)。

### partition.grouper

A partition grouper creates a list of stream tasks from the partitions of source topics, where each created task is assigned with a group of source topic partitions. The default implementation provided by Kafka Streams is [DefaultPartitionGrouper](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/DefaultPartitionGrouper.html). It assigns each task with one partition for each of the source topic partitions. The generated number of tasks equals the largest number of partitions among the input topics. Usually an application does not need to customize the partition grouper.

分区分组程序根据source主题分区创建流任务列表，其中每个创建的任务都分配有一组source主题分区。Kafka Streams提供的默认实现是[DefaultPartitionGrouper](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/DefaultPartitionGrouper.html)。它为每个任务分配每个source主题分区中的一个分区。生成的任务数量等于输入主题中分区的最大数量。通常，应用程序不需要定制分区分组程序。

### replication.factor

This specifies the replication factor of internal topics that Kafka Streams creates when local states are used or a stream is repartitioned for aggregation. Replication is important for fault tolerance. Without replication even a single broker failure may prevent progress of the stream processing application. It is recommended to use a similar replication factor as source topics.

这指定了在使用本地状态或重新分区流进行聚合时Kafka Streams创建的内部主题的复制因子。复制对容错非常重要。即使单个代理失效，也不会复制流处理应用程序的进度。建议使用与source主题类似的复制因子。

Recommendation:

建议：

Increase the replication factor to 3 to ensure that the internal Kafka Streams topic can tolerate up to 2 broker failures. Note that you will require more storage space as well (3 times more with the replication factor of 3).

将复制因子提高到3以确保内部Kafka Streams主题最多可以容忍2个代理失效。请注意，您还需要更多的存储空间（复制因子为3时需要3倍的空间）。

### state.dir

The state directory. Kafka Streams persists local states under the state directory. Each application has a subdirectory on its hosting machine that is located under the state directory. The name of the subdirectory is the application ID. The state stores associated with the application are created under this subdirectory.

状态目录。Kafka Streams在状态目录下保持本地状态。每个应用程序在其宿主机器上都有一个位于状态目录下的子目录。子目录的名称是应用程序ID。与该应用程序关联的状态存储器在该子目录下创建。

### timestamp.extractor

A timestamp extractor pulls a timestamp from an instance of [ConsumerRecord](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/consumer/ConsumerRecord.html). Timestamps are used to control the progress of streams.

时间戳提取器从[ConsumerRecord](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/consumer/ConsumerRecord.html)的实例中提取时间戳。时间戳用于控制流的进度。

The default extractor is [FailOnInvalidTimestamp](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/FailOnInvalidTimestamp.html). This extractor retrieves built-in timestamps that are automatically embedded into Kafka messages by the Kafka producer client since [Kafka version 0.10](https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message). Depending on the setting of Kafka’s server-side ```log.message.timestamp.type``` broker and ```message.timestamp.type``` topic parameters, this extractor provides you with:

默认提取器是[FailOnInvalidTimestamp](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/FailOnInvalidTimestamp.html)。此提取器检索那些被Kafka生产者客户端自[Kafka version 0.10](https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message)以来自动嵌入到Kafka消息中的内置时间戳。根据Kafka的服务器端```log.message.timestamp.type``` 代理和```message.timestamp.type```主题参数的设置，此提取器为您提供：

* **event-time** processing semantics if ```log.message.timestamp.type``` is set to ```CreateTime``` aka “producer time” (which is the default). This represents the time when a Kafka producer sent the original message. If you use Kafka’s official producer client, the timestamp represents milliseconds since the epoch.

* 如果```log.message.timestamp.type```被设置为```CreateTime```又名“生产者时间”（这是默认值），则可以使用**event-time**处理语义。这代表了Kafka生产者发送原始消息的时间。如果您使用Kafka的官方生产者客户端，则时间戳表示自该时期以来的毫秒数。

* **ingestion-time** processing semantics if ```log.message.timestamp.type``` is set to ```LogAppendTime``` aka “broker time”. This represents the time when the Kafka broker received the original message, in milliseconds since the epoch.

* 如果将```log.message.timestamp.type```设置为```LogAppendTime```又名“代理时间”，则可以使用**ingestion-time**处理语义。这表示Kafka代理收到原始消息的时间，单位是自该时期以来的毫秒数。

The ```FailOnInvalidTimestamp``` extractor throws an exception if a record contains an invalid (i.e. negative) built-in timestamp, because Kafka Streams would not process this record but silently drop it. Invalid built-in timestamps can occur for various reasons: if for example, you consume a topic that is written to by pre-0.10 Kafka producer clients or by third-party producer clients that don’t support the new Kafka 0.10 message format yet; another situation where this may happen is after upgrading your Kafka cluster from ```0.9``` to ```0.10```, where all the data that was generated with ```0.9``` does not include the ```0.10``` message timestamps.

如果消息包含无效（即负值）的内置时间戳，则```FailOnInvalidTimestamp```提取器会抛出异常，因为Kafka Streams不会处理该消息，而是会自动将其删除。无效的内置时间戳可能会因为各种原因出现：例如，如果您使用的是0.10版本之前的Kafka生产者客户端写入的主题，或者第三方生产者客户端不支持新的Kafka 0.10版本的消息格式;另一种可能发生的情况是在将Kafka集群从```0.9```升级到```0.10```之后，其中所有使用```0.9```生成的数据都不包含```0.10```的消息时间戳。

If you have data with invalid timestamps and want to process it, then there are two alternative extractors available. Both work on built-in timestamps, but handle invalid timestamps differently.

如果您的数据包含无效的时间戳并且想要处理它，那么有两种可选的提取器。两者都使用内置时间戳，但处理无效时间戳的方式不同。

* [LogAndSkipOnInvalidTimestamp](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/LogAndSkipOnInvalidTimestamp.html): This extractor logs a warn message and returns the invalid timestamp to Kafka Streams, which will not process but silently drop the record. This log-and-skip strategy allows Kafka Streams to make progress instead of failing if there are records with an invalid built-in timestamp in your input data.

* [LogAndSkipOnInvalidTimestamp](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/LogAndSkipOnInvalidTimestamp.html)：此提取器记录警告消息并将无效时间戳返回给Kafka Streams，Kafka Streams将不会处理，但会自动丢弃消息。此记录和跳过策略允许Kafka Streams在输入数据中存在无效内置时间戳的消息的情况下继续处理，而不是失效。

* [UsePreviousTimeOnInvalidTimestamp](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/UsePreviousTimeOnInvalidTimestamp.html). This extractor returns the record’s built-in timestamp if it is valid (i.e. not negative). If the record does not have a valid built-in timestamps, the extractor returns the previously extracted valid timestamp from a record of the same topic partition as the current record as a timestamp estimation. In case that no timestamp can be estimated, it throws an exception.

* [UsePreviousTimeOnInvalidTimestamp](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/UsePreviousTimeOnInvalidTimestamp.html)。如果消息有效的话（即不是负数），这个提取器返回它的内置时间戳。如果消息没有有效的内置时间戳，那么提取器将先前从与当前消息相同的主题分区中提取的一条消息的有效时间戳返回为预测时间戳。如果没有时间戳可以作为预测时间戳，它会抛出一个异常。

Another built-in extractor is [WallclockTimestampExtractor](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/WallclockTimestampExtractor.html). This extractor does not actually “extract” a timestamp from the consumed record but rather returns the current time in milliseconds from the system clock (think: ```System.currentTimeMillis()```), which effectively means Streams will operate on the basis of the so-called **processing-time** of events.

另一个内置提取器是[WallclockTimestampExtractor](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/WallclockTimestampExtractor.html)。这个提取器实际上并没有从所消费的消息中“提取”时间戳，而是从系统时钟返回当前时间（以毫秒为单位）（想想：```System.currentTimeMillis()```），这意味着Streams将根据所谓的事件的**处理时间**来完成相应的操作。

You can also provide your own timestamp extractors, for instance to retrieve timestamps embedded in the payload of messages. If you cannot extract a valid timestamp, you can either throw an exception, return a negative timestamp, or estimate a timestamp. Returning a negative timestamp will result in data loss – the corresponding record will not be processed but silently dropped. If you want to estimate a new timestamp, you can use the value provided via ```previousTimestamp``` (i.e., a Kafka Streams timestamp estimation). Here is an example of a custom ```TimestampExtractor``` implementation:

您也可以规定自己的时间戳提取器，例如检索嵌入到消息负载中的时间戳。如果无法提取有效的时间戳，则可以抛出异常，返回负值时间戳或预测时间戳。返回负值时间戳会导致数据丢失——相应的消息将不会被处理，而会自动丢弃。如果您想预测新的时间戳，则可以使用由```previousTimestamp```提供的值（即，Kafka Streams时间戳预测）。以下是自定义```TimestampExtractor```实现的示例：

```java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.streams.processor.TimestampExtractor;

// Extracts the embedded timestamp of a record (giving you "event-time" semantics).
// 提取消息的内嵌式时间戳（给出"event-time"语义）。
public class MyEventTimeExtractor implements TimestampExtractor {

  @Override
  public long extract(final ConsumerRecord<Object, Object> record, final long previousTimestamp) {
    // `Foo` is your own custom class, which we assume has a method that returns
    // the embedded timestamp (milliseconds since midnight, January 1, 1970 UTC).
    // `Foo`是您自己定制的类，我们假设它有一个返回内嵌时间戳的方法（从UTC时间1970年1月1日午夜
    // 开始，单位是毫秒）。
    long timestamp = -1;
    final Foo myPojo = (Foo) record.value();
    if (myPojo != null) {
      timestamp = myPojo.getTimestampInMillis();
    }
    if (timestamp < 0) {
      // Invalid timestamp!  Attempt to estimate a new timestamp,
      // otherwise fall back to wall-clock time (processing-time).
      // 无效的时间戳！ 尝试预测新的时间戳，否则回退到挂钟时间（处理时间）。
      if (previousTimestamp >= 0) {
        return previousTimestamp;
      } else {
        return System.currentTimeMillis();
      }
    }
  }

}
```

You would then define the custom timestamp extractor in your Streams configuration as follows:

然后，您将在Streams配置中定义自定义时间戳提取器，如下所示：

```java
import java.util.Properties;
import org.apache.kafka.streams.StreamsConfig;

Properties streamsConfiguration = new Properties();
streamsConfiguration.put(StreamsConfig.TIMESTAMP_EXTRACTOR_CLASS_CONFIG, MyEventTimeExtractor.class);
```

## Kafka consumers and producer configuration parameters

## Kafka消费者和生产者配置参数

You can specify parameters for the Kafka [consumers](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/consumer/package-summary.html) and [producers](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/producer/package-summary.html) that are used internally. The consumer and producer settings are defined by specifying parameters in a ```StreamsConfig``` instance.

您可以为内部使用的Kafka[消费者](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/consumer/package-summary.html)和[生产者](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/producer/package-summary.html)指定参数。消费者和生产者设置是通过在```StreamsConfig```实例中指定参数来定义的。

In this example, the Kafka [consumer session timeout](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/consumer/ConsumerConfig.html#SESSION_TIMEOUT_MS_CONFIG) is configured to be 60000 milliseconds in the Streams settings:

在此示例中，Kafka[消费者会话超时](http://kafka.apache.org/11/javadoc/org/apache/kafka/clients/consumer/ConsumerConfig.html#SESSION_TIMEOUT_MS_CONFIG)在Streams设置中配置为60000毫秒：

```java
Properties streamsSettings = new Properties();
// Example of a "normal" setting for Kafka Streams
streamsSettings.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker-01:9092");
// Customize the Kafka consumer settings of your Streams application
streamsSettings.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 60000);
StreamsConfig config = new StreamsConfig(streamsSettings);
```

## Naming

## 命名

Some consumer and producer configuration parameters use the same parameter name. For example, ```send.buffer.bytes``` and ```receive.buffer.bytes``` are used to configure TCP buffers; ```request.timeout.ms``` and ```retry.backoff.ms``` control retries for client request. You can avoid duplicate names by prefix parameter names with ```consumer```. or ```producer``` (e.g., ```consumer.send.buffer.bytes``` and ```producer.send.buffer.bytes```).

一些消费者和生产者配置参数使用相同的参数名称。例如，```send.buffer.bytes```和```receive.buffer.bytes```用于配置TCP缓冲区;```request.timeout.ms```和```retry.backoff.ms```控制客户端请求的重试次数。您可以通过使用前缀参数名称```consumer```或 ```producer```来避免重名。（例如，```consumer.send.buffer.bytes```和```producer.send.buffer.bytes```）。

```java
Properties streamsSettings = new Properties();
// same value for consumer and producer
// 消费者和生产者相同的值
streamsSettings.put("PARAMETER_NAME", "value");
// different values for consumer and producer
// 消费者和生产者不同的值
streamsSettings.put("consumer.PARAMETER_NAME", "consumer-value");
streamsSettings.put("producer.PARAMETER_NAME", "producer-value");
// alternatively, you can use
// 或者，您可以使用
streamsSettings.put(StreamsConfig.consumerPrefix("PARAMETER_NAME"), "consumer-value");
streamsSettings.put(StreamsConfig.producerPrefix("PARAMETER_NAME"), "producer-value");
```

## Default Values

## 默认值

Kafka Streams uses different default values for some of the underlying client configs, which are summarized below. For detailed descriptions of these configs, see [Producer Configs](http://kafka.apache.org/11/documentation.html#producerconfigs) and [Consumer Configs](http://kafka.apache.org/11/documentation.html#newconsumerconfigs).

对于一些底层客户端配置，Kafka Streams使用不同的默认值，下面总结了这些配置。有关这些配置的详细说明，请参阅[Producer Configs](http://kafka.apache.org/11/documentation.html#producerconfigs)和[Consumer Configs](http://kafka.apache.org/11/documentation.html#newconsumerconfigs)。

Parameter Name | Corresponding Client | Streams Default
--- | --- | ---
auto.offset.reset | Consumer | earliest
enable.auto.commit | Consumer | false
linger.ms | Producer | 100
max.poll.interval.ms | Consumer | Integer.MAX_VALUE
max.poll.records | Consumer | 1000
retries	| Producer | 10
rocksdb.config.setter | Consumer | 	

参数名称 | 相应的客户端 | 默认的Streams
--- | --- | ---
auto.offset.reset | 消费者 | earliest
enable.auto.commit | 消费者 | false
linger.ms | 生产者 | 100
max.poll.interval.ms | 消费者 | Integer.MAX_VALUE
max.poll.records | 消费者 | 1000
retries	| 生产者 | 10
rocksdb.config.setter | 消费者 | 	 

### enable.auto.commit

The consumer auto commit. To guarantee at-least-once processing semantics and turn off auto commits, Kafka Streams overrides this consumer config value to ```false```. Consumers will only commit explicitly via *commitSync* calls when the Kafka Streams library or a user decides to commit the current processing state.

消费者自动提交。为了保证至少处理一次处理语义并关闭自动提交，Kafka Streams将此消费者配置值重写为```false```。当Kafka Streams库或用户决定提交当前处理状态时，消费者只能通过*commitSync*调用显式提交。

### rocksdb.config.setter

The RocksDB configuration. Kafka Streams uses RocksDB as the default storage engine for persistent stores. To change the default configuration for RocksDB, implement ```RocksDBConfigSetter``` and provide your custom class via [rocksdb.config.setter](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/state/RocksDBConfigSetter.html).

RocksDB配置。Kafka Streams使用RocksDB作为持久性存储的默认存储引擎。要更改RocksDB的默认配置，请实现```RocksDBConfigSetter```并通过[rocksdb.config.setter](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/state/RocksDBConfigSetter.html)提供您的自定义类。

Here is an example that adjusts the memory size consumed by RocksDB.

这是一个调整RocksDB消耗的内存大小的例子。

```java
    public static class CustomRocksDBConfig implements RocksDBConfigSetter {

       @Override
       public void setConfig(final String storeName, final Options options, final Map<String, Object> configs) {
         // See #1 below.
         BlockBasedTableConfig tableConfig = new org.rocksdb.BlockBasedTableConfig();
         tableConfig.setBlockCacheSize(16 * 1024 * 1024L);
         // See #2 below.
         tableConfig.setBlockSize(16 * 1024L);
         // See #3 below.
         tableConfig.setCacheIndexAndFilterBlocks(true);
         options.setTableFormatConfig(tableConfig);
         // See #4 below.
         options.setMaxWriteBufferNumber(2);
       }
    }

Properties streamsSettings = new Properties();
streamsConfig.put(StreamsConfig.ROCKSDB_CONFIG_SETTER_CLASS_CONFIG, CustomRocksDBConfig.class);
```

Notes for example:

请注意：

1. ```BlockBasedTableConfig tableConfig = new org.rocksdb.BlockBasedTableConfig();``` Reduce block cache size from the default, shown [here](https://github.com/apache/kafka/blob/1.0/streams/src/main/java/org/apache/kafka/streams/state/internals/RocksDBStore.java#L81), as the total number of store RocksDB databases is partitions (40) * segments (3) = 120.

1. ```BlockBasedTableConfig tableConfig = new org.rocksdb.BlockBasedTableConfig();```由于存储RocksDB数据库的总数是分区（40）*段（3）= 120，因此请将块缓存大小从默认值降低，如[此处](https://github.com/apache/kafka/blob/1.0/streams/src/main/java/org/apache/kafka/streams/state/internals/RocksDBStore.java#L81)所示。

2. ```tableConfig.setBlockSize(16 * 1024L);``` Modify the default [block size](https://github.com/apache/kafka/blob/1.0/streams/src/main/java/org/apache/kafka/streams/state/internals/RocksDBStore.java#L82) per these instructions from the [RocksDB GitHub](https://github.com/facebook/rocksdb/wiki/Memory-usage-in-RocksDB#indexes-and-filter-blocks).

2. ```tableConfig.setBlockSize(16 * 1024L);```根据[RocksDB GitHub](https://github.com/facebook/rocksdb/wiki/Memory-usage-in-RocksDB#indexes-and-filter-blocks)的这些指令修改默认[块大小](https://github.com/apache/kafka/blob/1.0/streams/src/main/java/org/apache/kafka/streams/state/internals/RocksDBStore.java#L82)。

3. ```tableConfig.setCacheIndexAndFilterBlocks(true);``` Do not let the index and filter blocks grow unbounded. For more information, see the [RocksDB GitHub](https://github.com/facebook/rocksdb/wiki/Block-Cache#caching-index-and-filter-blocks).

3. ```tableConfig.setCacheIndexAndFilterBlocks(true);```不要让索引和过滤器块无限制地增长。有关更多信息，请参阅[RocksDB GitHub](https://github.com/facebook/rocksdb/wiki/Block-Cache#caching-index-and-filter-blocks)。

4. ```options.setMaxWriteBufferNumber(2);``` See the advanced options in the [RocksDB GitHub](https://github.com/facebook/rocksdb/blob/8dee8cad9ee6b70fd6e1a5989a8156650a70c04f/include/rocksdb/advanced_options.h#L103).

4. ```options.setMaxWriteBufferNumber(2);```请参阅[RocksDB GitHub](https://github.com/facebook/rocksdb/blob/8dee8cad9ee6b70fd6e1a5989a8156650a70c04f/include/rocksdb/advanced_options.h#L103)中的高级选项。

## Recommended configuration parameters for resiliency

## 建议的弹性配置参数

There are several Kafka and Kafka Streams configuration options that need to be configured explicitly for resiliency in face of broker failures:

这是几个Kafka和Kafka Streams需要明确配置以便弹性应对代理发生故障的配置选项：

Parameter Name | Corresponding Client | Default value | Consider setting to
--- | --- | --- | ---
acks | Producer | acks=1 | acks=all
replication.factor | Streams | 1 | 3
min.insync.replicas | Broker | 1 | 2

参数名称 | 相应客户端 | 默认值 | 酌情设置
--- | --- | --- | ---
acks | 生产者 | ```acks=1``` | ```acks=all```
replication.factor | Streams | ```1``` | ```3```
min.insync.replicas | 代理 | ```1``` | ```2```

Increasing the replication factor to 3 ensures that the internal Kafka Streams topic can tolerate up to 2 broker failures. Changing the acks setting to “all” guarantees that a record will not be lost as long as one replica is alive. The tradeoff from moving to the default values to the recommended ones is that some performance and more storage space (3x with the replication factor of 3) are sacrificed for more resiliency.

将复制因子增加到3可确保内部Kafka Streams主题最多可容忍2个代理失效。将acks设置更改为“all”可以保证只要一个副本处于活动状态，消息就不会丢失。从默认值转换为推荐值的权衡是牺牲一些性能和更多的存储空间（复制因子为3时是3倍）以提高弹性。

### acks

The number of acknowledgments that the leader must have received before considering a request complete. This controls the durability of records that are sent. The possible values are:

在衡量一个请求是否完成前，领导者已经收到的确认的数量。这将控制发送的消息的持久性。可能的值是：

* ```acks=0``` The producer does not wait for acknowledgment from the server and the record is immediately added to the socket buffer and considered sent. No guarantee can be made that the server has received the record in this case, and the ```retries``` configuration will not take effect (as the client won’t generally know of any failures). The offset returned for each record will always be set to ```-1```.

*  ```acks = 0```生产者不等待来自服务器的确认，且该消息立即被添加到套接字缓冲区并被认为已经被发送。 在这种情况下，不能保证服务器已经收到消息，并且```retries```配置不会生效（因为客户端通常不会知道任何故障）。为每条消息返回的偏移量将始终设置为```-1```。

* ```acks=1``` The leader writes the record to its local log and responds without waiting for full acknowledgement from all followers. If the leader immediately fails after acknowledging the record, but before the followers have replicated it, then the record will be lost.

* ```acks = 1```领导者将消息写入其本地日志并作出响应，无需等待所有追随者的完整确认。如果领导者在确认消息后，但在追随者复制之前，立即失效，则消息将丢失。

* ```acks=all``` The leader waits for the full set of in-sync replicas to acknowledge the record. This guarantees that the record will not be lost if there is at least one in-sync replica alive. This is the strongest available guarantee.

* ```acks = all```领导者等待所有同步副本确认消息。这保证了如果至少有一个同步副本处于活动状态，则该消息不会丢失。这是最可靠的保证。

For more information, see the [Kafka Producer documentation](https://kafka.apache.org/documentation/#producerconfigs).

有关更多信息，请参阅[Kafka Producer文档](https://kafka.apache.org/documentation/#producerconfigs)。

### replication.factor

See the [description here](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#replication-factor-parm).

请参阅[此处的说明](config-streams.md)。

You define these settings via ```StreamsConfig```:

您可以通过```StreamsConfig```定义这些设置：

```java
Properties streamsSettings = new Properties();
streamsSettings.put(StreamsConfig.REPLICATION_FACTOR_CONFIG, 3);
streamsSettings.put(StreamsConfig.producerPrefix(ProducerConfig.ACKS_CONFIG), "all");
```

Note

提示

A future version of Kafka Streams will allow developers to set their own app-specific configuration settings through ```StreamsConfig``` as well, which can then be accessed through [ProcessorContext](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/ProcessorContext.html).

未来版本的Kafka Streams将允许开发人员通过```StreamsConfig```设置自己的应用程序特定配置，然后可以通过[ProcessorContext](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/processor/ProcessorContext.html)访问它们。