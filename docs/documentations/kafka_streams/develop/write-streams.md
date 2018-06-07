# Writing a Streams Application

# 编写一个Streams应用程序

### Table of Contents

### 目录表

* [Libraries and Maven artifacts](http://kafka.apache.org/11/documentation/streams/developer-guide/write-streams.html#libraries-and-maven-artifacts)

* [Using Kafka Streams within your application code](http://kafka.apache.org/11/documentation/streams/developer-guide/write-streams.html#using-kafka-streams-within-your-application-code)


* [库和Maven artifacts](write-streams.md)

* [在应用程序代码中使用Kafka Streams](write-streams.md)

Any Java application that makes use of the Kafka Streams library is considered a Kafka Streams application. The computational logic of a Kafka Streams application is defined as a [processor topology](http://kafka.apache.org/11/documentation/streams/core-concepts.html#streams_topology), which is a graph of stream processors (nodes) and streams (edges).

任何使用Kafka Streams库编写的Java应用程序都被视为Kafka Streams应用程序。Kafka Streams应用程序的计算逻辑被定义为[处理器拓扑](../concepts.md)，它是流处理器（节点）和流（边）的图形化表示。

You can define the processor topology with the Kafka Streams APIs:

您可以使用Kafka Streams API定义处理器拓扑：

[Kafka Streams DSL](dsl-api.md)

A high-level API that provides provides the most common data transformation operations such as ```map```, ```filter```, ```join```, and ```aggregations``` out of the box. The DSL is the recommended starting point for developers new to Kafka Streams, and should cover many use cases and stream processing needs.

这种由Kafka提供的高级API提供了最常用的数据转换操作，例如```map```，```filter```，```join```和```aggregations```。对于开发Kafka Streams的入门开发人员来说，DSL是推荐的起点，而且它应该涵盖许多用例和流处理需求。

[Processor API](http://kafka.apache.org/11/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api)

[处理器API](processor-api.md)

A low-level API that lets you add and connect processors as well as interact directly with state stores. The Processor API provides you with even more flexibility than the DSL but at the expense of requiring more manual work on the side of the application developer (e.g., more lines of code).

这是一种可以让您添加和连接处理器以及直接与状态存储器进行交互的低级API。处理器API为您提供比DSL更大的灵活性，但代价是应用程序开发人员需要更多的手动工作（例如，更多的代码行数）。

## LIBRARIES AND MAVEN ARTIFACTS

## 库和Maven artifacts

This section lists the Kafka Streams related libraries that are available for writing your Kafka Streams applications.

本节列出了可用于编写Kafka Streams应用程序的Kafka Streams相关库。

You can define dependencies on the following libraries for your Kafka Streams applications.

您可以为Kafka Streams应用程序定义以下库的依赖关系。

 Group ID | Artifact ID | Version | Description 
--- | --- | --- | --- 
 ```org.apache.kafka``` | ```kafka-streams``` | ```1.1.0``` | (Required) Base library for Kafka Streams. 
 ```org.apache.kafka``` | ```kafka-clients``` | ```1.1.0``` | (Required) Kafka client library. Contains built-in serializers/deserializers.

  组ID | 工件ID | 版本 | 描述 
 --- | --- | --- | --- 
 ```org.apache.kafka``` | ```kafka-streams``` | ```1.1.0``` | （必填）Kafka Streams的基础库。 |
 ```org.apache.kafka``` | ```kafka-clients``` | ```1.1.0``` | （必填）Kafka客户端库。包含内置的序列化/反序列化器。

### Tip

### 提示

See the section [Data Types and Serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes) for more information about Serializers/Deserializers.

有关序列化/反序列化器的更多信息，请参见[数据类型和序列化](datatypes.md)部分。

Example ```pom.xml``` snippet when using Maven:

使用Maven时```pom.xml```片段的例子：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>1.1.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

## USING KAFKA STREAMS WITHIN YOUR APPLICATION CODE

## 在您的应用程序代码中使用KAFKA STREAMS

You can call Kafka Streams from anywhere in your application code, but usually these calls are made within the ```main()``` method of your application, or some variant thereof. The basic elements of defining a processing topology within your application are described below.

您可以从应用程序代码中的任何位置调用Kafka Streams，但通常这些调用是在您的应用程序的```main()```方法或其某些变体中进行的。下面介绍在应用程序中定义处理拓扑的基本元素。

First, you must create an instance of ```KafkaStreams```.

首先，您必须创建一个```KafkaStreams```的实例。

* The first argument of the ```KafkaStreams``` constructor takes a topology (either ```StreamsBuilder#build()``` for the [DSL](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl) or ```Topology``` for the [Processor API](http://kafka.apache.org/11/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api)) that is used to define a topology.

* The second argument is an instance of ```StreamsConfig```, which defines the configuration for this specific topology.


* ```KafkaStreams```构造方法的第一个参数需要一个拓扑（用于[DSL](dsl-api.md)的```StreamsBuilder＃build()```或用于[处理器API](processor-api.md)的```拓扑```）用于定义拓扑。

* 第二个参数是```StreamsConfig```的一个实例，它定义了这个特定拓扑的配置。

Code example:

代码示例：

```java
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.StreamsBuilder;
import org.apache.kafka.streams.processor.Topology;

// Use the builders to define the actual processing topology, e.g. to specify
// from which input topics to read, which stream operations (filter, map, etc.)
// should be called, and so on.  We will cover this in detail in the subsequent
// sections of this Developer Guide.

// 使用builders来定义实际的处理拓扑，例如,指定
// 从哪个输入主题读取，哪些流操作（filter，map等）
// 应该被调用，等等。我们将在本开发者指南的后续章节中
// 详细介绍。

StreamsBuilder builder = ...;  // when using the DSL // 使用DSL
Topology topology = builder.build();
//
// OR
//
Topology topology = ...; // when using the Processor API // 或者使用处理器API

// Use the configuration to tell your application where the Kafka cluster is,
// which Serializers/Deserializers to use by default, to specify security settings,
// and so on.

//使用配置来告诉您的应用程序Kafka集群的位置，默认使用哪些序列化/反序列化器来指定安全设置等等。
StreamsConfig config = ...;

KafkaStreams streams = new KafkaStreams(topology, config);
```

At this point, internal structures are initialized, but the processing is not started yet. You have to explicitly start the Kafka Streams thread by calling the ```KafkaStreams#start()``` method:

此时，内部结构已初始化，但处理尚未开始。您必须通过调用```KafkaStreams＃start()```方法显式启动Kafka Streams线程：

```java
// Start the Kafka Streams threads
// 启动Kafka Streams线程
streams.start();
```

If there are other instances of this stream processing application running elsewhere (e.g., on another machine), Kafka Streams transparently re-assigns tasks from the existing instances to the new instance that you just started. For more information, see [Stream Partitions and Tasks](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-tasks) and [Threading Model](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-threads).

如果此stream应用程序的其他实例在别处运行（例如，在另一台计算机上），则Kafka Streams会透明地将任务从现有实例重新分配给刚刚启动的新实例。更多信息，请参阅[流分区和任务](../architecture.md)和[线程模型](../architecture.md)。

To catch any unexpected exceptions, you can set an ```java.lang.Thread.UncaughtExceptionHandler``` before you start the application. This handler is called whenever a stream thread is terminated by an unexpected exception:

要捕获任何意外的异常，可以在启动应用程序之前设置```java.lang.Thread.UncaughtExceptionHandler```。只要stream线程被意外异常终止，就会调用此处理方法：

```java
// Java 8+, using lambda expressions
// Java 8+, 使用lambda表达式
streams.setUncaughtExceptionHandler((Thread thread, Throwable throwable) -> {
  // here you should examine the throwable/exception and perform an appropriate action!
  // 在这里您应该检查throwable / exception并执行适当的操作！
});


// Java 7
streams.setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
  public void uncaughtException(Thread thread, Throwable throwable) {
    // here you should examine the throwable/exception and perform an appropriate action!
    // 在这里您应该检查throwable / exception并执行适当的操作！
  }
});
```

To stop the application instance, call the ```KafkaStreams#close() ```method:

要停止应用程序实例，请调用```KafkaStreams＃close()```方法：

```java
// Stop the Kafka Streams threads
// 停止Kafka Streams线程
streams.close();
```

To allow your application to gracefully shutdown in response to SIGTERM, it is recommended that you add a shutdown hook and call ```KafkaStreams#close```.

为了让您的应用程序正常关闭以响应SIGTERM，建议您添加一个关闭钩子并调用```KafkaStreams＃close```。

* Here is a shutdown hook example in Java 8+:

    这是Java 8+中的关闭钩子的示例：

```java
// Add shutdown hook to stop the Kafka Streams threads.
// You can optionally provide a timeout to `close`.

// 添加关闭钩子以停止Kafka Streams线程。
// 您可以选择提供一个时间限额来关闭。
Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
```

* Here is a shutdown hook example in Java 7:

    这是Java 7中的关闭钩子示例：

```java
// Add shutdown hook to stop the Kafka Streams threads.
// You can optionally provide a timeout to `close`.

// 添加关闭钩子以停止Kafka Streams线程。
// 您可以选择提供一个时间限额来关闭。
Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
  @Override
  public void run() {
      streams.close();
  }
}));
```

After an application is stopped, Kafka Streams will migrate any tasks that had been running in this instance to available remaining instances.

在应用程序停止后，Kafka Streams会将在此实例中运行的所有任务迁移到剩余可用的实例中。