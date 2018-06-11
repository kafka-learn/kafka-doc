# 2. APIS

Kafka includes five core apis:

Kafka 包含以下五个核心apis:

1. The [Producer](http://kafka.apache.org/documentation/#producerapi) API allows applications to send streams of data to topics in the Kafka cluster.

2. The [Consumer](http://kafka.apache.org/documentation/#consumerapi) API allows applications to read streams of data from topics in the Kafka cluster.

3. The [Streams](http://kafka.apache.org/documentation/#streamsapi) API allows transforming streams of data from input topics to output topics.

4. The [Connect](http://kafka.apache.org/documentation/#connectapi) API allows implementing connectors that continually pull from some source system or application into Kafka or push from Kafka into some sink system or application.

5. The [AdminClient](http://kafka.apache.org/documentation/#adminapi) API allows managing and inspecting topics, brokers, and other Kafka objects.


1. [Producer](./apis/producer.md) API允许应用发送数据流到Kafka集群中的主题(topics)。

2. [Consumer](./apis/consumer.md) API允许应用从Kafka集群的主题(topics)中获取数据流。

3. [Streams](./apis/streams.md) API允许从数据流从输入主题(topics)传送到输出主题(topics)。

4. [Connect](./apis/connect.md) API允许实现连接器，不断的从一些源系统或应用拉取数据到Kafka，或者从Kafka推送数据到一些汇聚系统（sink system）或应用。

5. [AdminClient](./apis/admin_client.md) API允许管理、检查主题(topics)，代理和其他Kafka对象。


Kafka exposes all its functionality over a language independent protocol which has clients available in many programming languages. However only the Java clients are maintained as part of the main Kafka project, the others are available as independent open source projects. A list of non-Java clients is available [here](https://cwiki.apache.org/confluence/display/KAFKA/Clients).

Kafka公开了其所有功能性协议，这些协议与语言无关，且Kafka提供了多种编程语言的客户端。然而，只有Java客户端作为Kafka项目的一部分被维护，其他客户端作为独立的开源项目。[这里](https://cwiki.apache.org/confluence/display/KAFKA/Clients)是非Java客户端的列表。

## 2.1 Producer API

The Producer API allows applications to send streams of data to topics in the Kafka cluster.

生产者API允许应用发送数据流到Kafka集群中的主题（topics）

Examples showing how to use the producer are given in the [javadocs](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html).

在[Java文档](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html)中给出了如何使用生产者的样例。

To use the producer, you can use the following maven dependency:

欲使用生产者，您可以添加以下maven依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 2.2 Consumer API

The Consumer API allows applications to read streams of data from topics in the Kafka cluster.

消费者API允许应用从Kafka集群的主题(topics)中获取数据流。

Examples showing how to use the consumer are given in the [javadocs](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html).

在[Java文档](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)中给出了如何使用消费者的样例。

To use the consumer, you can use the following maven dependency:

欲使用消费者，您可以添加以下maven依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 2.3 Streams API

The [Streams](http://kafka.apache.org/documentation/#streamsapi) API allows transforming streams of data from input topics to output topics.

 [Streams](./apis/streams.md) API允许数据流从输入主题(input topics)传送到输出主题(output topics)。

Examples showing how to use this library are given in the [javadocs](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/streams/KafkaStreams.html).

在[Java文档](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/streams/KafkaStreams.html)中给出了如何使用这个类库的样例。

Additional documentation on using the Streams API is available [here](http://kafka.apache.org/11/documentation/streams).

更多关于Streams API的文档在[这里](./kafka_streams/introduction.md)。

To use Kafka Streams you can use the following maven dependency:

欲使用Kafka Streams，您可以添加如下maven依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 2.4 Connect API

The Connect API allows implementing connectors that continually pull from some source data system into Kafka or push from Kafka into some sink data system.

连接器API允许实现连接器，它不断的从一些源数据（source data system）系统或应用拉取数据到Kafka，或者从Kafka推送数据到一些汇聚系统（sink system）或应用。

Many users of Connect won't need to use this API directly, though, they can use pre-built connectors without needing to write any code. Additional information on using Connect is available [here](http://kafka.apache.org/documentation.html#connect).

很多使用连接器的用户不需要直接使用此API，可以直接使用预先构建的连接器而不需要编写任何代码。更多关于连接器的文档在[这里](./kafka_connect.md)。

Those who want to implement custom connectors can see the [javadoc](http://kafka.apache.org/11/javadoc/overview-summary.html).

欲实现自定义的连接器，可以参考这个[Java文档](http://kafka.apache.org/11/javadoc/overview-summary.html)。

## 2.5 AdminClient API

The AdminClient API supports managing and inspecting topics, brokers, acls, and other Kafka objects.

AdminClient API允许管理、检查主题(topic)，代理和其他Kafka对象。

To use the AdminClient API, add the following Maven dependency:

欲使用AdminClient API，您可以添加如下Maven依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

For more information about the AdminClient APIs, see the [javadoc](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/admin/AdminClient.html).

更多关于AdminClient APIs的信息，参见[Java文档](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/admin/AdminClient.html)。

## 2.6 Legacy APIs

A more limited legacy producer and consumer api is also included in Kafka. These old Scala APIs are deprecated and only still available for compatibility purposes. Information on them can be found [here](http://kafka.apache.org/081/documentation.html#producerapi).

在Kafka中也包含了有更多限制的生产者和消费者API。这些老旧的Scala APIs已经被废弃，它们之所以仍然可用仅仅是为了兼容性。关于他们的一些信息可以在[这里](http://kafka.apache.org/081/documentation.html#producerapi)找到。