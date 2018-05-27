# Streams DSL

The Kafka Streams DSL (Domain Specific Language) is built on top of the Streams Processor API. It is the recommended for most users, especially beginners. Most data processing operations can be expressed in just a few lines of DSL code.

Kafka Streams DSL（域特定语言）构建在Streams Processor API之上。这是推荐大多数用户使用的，特别是初学者。大部分数据处理操作都可以用几行DSL代码表示。

### Table of Contents

### 目录

* [Overview](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#overview)

* [Creating source streams from Kafka](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#creating-source-streams-from-kafka)

* [Transform a stream](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#transform-a-stream)


* [概述](dsl-api.md)

* [从Kafka创建source流](dsl-api.md)

* [转换流](dsl-api.md)

    * [Stateless transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#stateless-transformations)

    * [Stateful transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#stateful-transformations)


    * [无状态的转换](dsl-api.md)

    * [有状态的转换](dsl-api.md)

        * [Aggregating](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#aggregating)

        * [Joining](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#joining)


        * [聚合](dsl-api.md)

        * [联结](dsl-api.md)

            * [Join co-partitioning requirements](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#join-co-partitioning-requirements)

            * [KStream-KStream Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#kstream-kstream-join)

            * [KTable-KTable Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#ktable-ktable-join)

            * [KStream-KTable Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#kstream-ktable-join)

            * [KStream-GlobalKTable Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#kstream-globalktable-join)


            * [连接共分区(co-partitioning)要求](dsl-api.md)

            * [KStream-KStream联结](dsl-api.md)

            * [KTable-KTable联结](dsl-api.md)

            * [KStream-KTable联结](dsl-api.md)

            * [KStream-GlobalKTable联结](dsl-api.md)

        * [Windowing](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#windowing)

        * [Windowing](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#windowing)
            * [Tumbling time windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#tumbling-time-windows)

            * [Hopping time windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#hopping-time-windows)

            * [Sliding time windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#sliding-time-windows)

            * [Session Windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#session-windows)


            * [翻转时间窗口](dsl-api.md)

            * [跳跃时间窗口](dsl-api.md)

            * [滑动时间窗口](dsl-api.md)

            * [会话窗口](dsl-api.md)

    * [Applying processors and transformers (Processor API integration)](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#applying-processors-and-transformers-processor-api-integration)

    * [使用处理器和变换器(处理器API集成)](dsl-api.md)

* [Writing streams back to Kafka](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#writing-streams-back-to-kafka)

* [将流(streams)写回到卡夫卡](dsl-api.md)

## OVERVIEW

## 概述

In comparison to the [Processor API](http://kafka.apache.org/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api), only the DSL supports:

与[处理器API](processor-api.md)相比，只有DSL支持的情况是：

* Built-in abstractions for [streams and tables](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-duality) in the form of [KStream](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-kstream), [KTable](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-ktable), and [GlobalKTable](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-globalktable). Having first-class support for streams and tables is crucial because, in practice, most use cases require not just either streams or databases/tables, but a combination of both. For example, if your use case is to create a customer 360-degree view that is updated in real-time, what your application will be doing is transforming many input streams of customer-related events into an output table that contains a continuously updated 360-degree view of your customers.

* 内置抽象的[KStream](../concepts.md)，[KTable](../concepts.md)和[GlobalKTable](../concepts.md)形式的[流和表(streams and tables)](../concepts.md)。对流和表(streams and tables)提供一流的支持是非常重要的，因为在实践中，大多数用例不仅需要流或数据库/表，而且还需要两者的组合。例如，如果您的用例是创建实时更新的360度客户视图，那么您的应用程序将要做的是将许多与客户相关的事件输入流转换为包含不断更新的360的输出表 客户的视角。

* Declarative, functional programming style with [stateless transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless) (e.g. ```map``` and ```filter```) as well as [stateful transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateful) such as [aggregations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) (e.g. ```count``` and ```reduce```), [joins](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins) (e.g. ```leftJoin```), and [windowing](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing) (e.g. [session windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#windowing-session)).

* 具有无状态转换（例如```map```和 ```filter```）以及诸如聚合（例如```count```和 ```reduce```），联接（例如```leftJoin```）和窗口（例如会话窗口（session windows））这些特征的有状态转换的声明性和函数式编程风格。

With the DSL, you can define [processor topologies](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-processor-topology) (i.e., the logical processing plan) in your application. The steps to accomplish this are:

使用DSL，您可以在应用程序中定义[处理器拓扑]
(../concepts.md)（即逻辑处理计划）。完成此步骤的步骤是：

1. Specify [one or more input streams that are read from Kafka topics](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-sources).

2. Compose [transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations) on these streams.

3. Write the [resulting output streams back to Kafka topics](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-destinations), or expose the processing results of your application directly to other applications through [interactive queries](http://kafka.apache.org/documentation/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries) (e.g., via a REST API).


1. 指定[一个或多个从Kafka主题读取的输入流](dsl-api.md)。

2. 组合这些流的[转换](jsl-api.md)。

3. 将[结果输出流写回Kafka主题](dsl-api.md)，或通过[交互式查询](interactive-queries.md)（例如，通过REST API）将应用程序的处理结果直接公开给其他应用程序。

After the application is run, the defined processor topologies are continuously executed (i.e., the processing plan is put into action). A step-by-step guide for writing a stream processing application using the DSL is provided below.

应用程序运行后，定义的处理器拓扑将不断执行（即处理计划付诸实施）。以下提供了使用DSL编写流处理应用程序的分步指南。

For a complete list of available API functionality, see also the [Streams](http://kafka.apache.org/javadoc/org/apache/kafka/streams/package-summary.html) API docs.

有关可用API功能的完整列表，另请参阅[Streams](http://kafka.apache.org/javadoc/org/apache/kafka/streams/package-summary.html) API文档。

## CREATING SOURCE STREAMS FROM KAFKA

## 创建来自Kafka的source流

You can easily read data from Kafka topics into your application. The following operations are supported.

您可以轻松地将来自Kafka主题的数据读入您的应用程序。以下操作均能得到支持。

Reading from Kafka | Description
------------ | --------
Stream

* *input topics* → KStream 
 
| Creates a KStream from the specified Kafka input topics and interprets the data as a record stream. A KStream represents a partitioned record stream. (details)

In the case of a KStream, the local KStream instance of every application instance will be populated with data from only a subset of the partitions of the input topic. Collectively, across all application instances, all input topic partitions are read and processed.

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.KStream;

StreamsBuilder builder = new StreamsBuilder();

KStream<String, Long> wordCounts = builder.stream(
    "word-counts-input-topic", /* input topic */
    Consumed.with(
      Serdes.String(), /* key serde */
      Serdes.Long()   /* value serde */
    );
If you do not specify SerDes explicitly, the default SerDes from the configuration are used.

You must specify SerDes explicitly if the key or value types of the records in the Kafka input topics do not match the configured default SerDes. For information about configuring default SerDes, available SerDes, and implementing your own custom SerDes see Data Types and Serialization.

Several variants of stream exist, for example to specify a regex pattern for input topics to read from).

Table | 

input topic → KTable
Reads the specified Kafka input topic into a KTable. The topic is interpreted as a changelog stream, where records with the same key are interpreted as UPSERT aka INSERT/UPDATE (when the record value is not null) or as DELETE (when the value is null) for that key. (details)

In the case of a KStream, the local KStream instance of every application instance will be populated with data from only a subset of the partitions of the input topic. Collectively, across all application instances, all input topic partitions are read and processed.

You must provide a name for the table (more precisely, for the internal state store that backs the table). This is required for supporting interactive queries against the table. When a name is not provided the table will not queryable and an internal name will be provided for the state store.

If you do not specify SerDes explicitly, the default SerDes from the configuration are used.

You must specify SerDes explicitly if the key or value types of the records in the Kafka input topics do not match the configured default SerDes. For information about configuring default SerDes, available SerDes, and implementing your own custom SerDes see Data Types and Serialization.

Several variants of table exist, for example to specify the auto.offset.reset policy to be used when reading from the input topic.

Global Table

input topic → GlobalKTable
Reads the specified Kafka input topic into a GlobalKTable. The topic is interpreted as a changelog stream, where records with the same key are interpreted as UPSERT aka INSERT/UPDATE (when the record value is not null) or as DELETE (when the value is null) for that key. (details)

In the case of a GlobalKTable, the local GlobalKTable instance of every application instance will be populated with data from all the partitions of the input topic.

You must provide a name for the table (more precisely, for the internal state store that backs the table). This is required for supporting interactive queries against the table. When a name is not provided the table will not queryable and an internal name will be provided for the state store.

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.GlobalKTable;

StreamsBuilder builder = new StreamsBuilder();

GlobalKTable<String, Long> wordCounts = builder.globalTable(
    "word-counts-input-topic",
    Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as(
      "word-counts-global-store" /* table/store name */)
      .withKeySerde(Serdes.String()) /* key serde */
      .withValueSerde(Serdes.Long()) /* value serde */
    );
You must specify SerDes explicitly if the key or value types of the records in the Kafka input topics do not match the configured default SerDes. For information about configuring default SerDes, available SerDes, and implementing your own custom SerDes see Data Types and Serialization.

Several variants of globalTable exist to e.g. specify explicit SerDes.

## TRANSFORM A STREAM

## 转换流

The KStream and KTable interfaces support a variety of transformation operations. Each of these operations can be translated into one or more connected processors into the underlying processor topology. Since KStream and KTable are strongly typed, all of these transformation operations are defined as generic functions where users could specify the input and output data types.

KStream和KTable接口支持各种转换操作。这些操作中的每一个都可以被转换为一个或多个连接的处理器，并转换为底层处理器拓扑。由于KStream和KTable是强类型的，所有这些转换操作都被定义为通用函数，用户可以指定输入和输出数据类型。

Some KStream transformations may generate one or more KStream objects, for example: - ```filter``` and ```map``` on a KStream will generate another KStream - ```branch``` on KStream can generate multiple KStreams

一些KStream转换可能会生成一个或多个KStream对象，例如：```filter```和```map```KStream会生成另一个KStream，```branch```KStream可以生成多个KStream

Some others may generate a KTable object, for example an aggregation of a KStream also yields a KTable. This allows Kafka Streams to continuously update the computed value upon arrivals of [late records](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-aggregations) after it has already been produced to the downstream transformation operators.

其他的可能会生成KTable对象，例如KStream的聚合也会生成KTable。这允许Kafka Streams在给下游转化操作符生产好的[迟到消息(late records)](../concepts.md)到达时连续不断地更新计算值。

All KTable transformation operations can only generate another KTable. However, the Kafka Streams DSL does provide a special function that converts a KTable representation into a KStream. All of these transformation methods can be chained together to compose a complex processor topology.

所有KTable转换操作只能生成另一个KTable。但是，Kafka Streams DSL确实提供了将KTable转换为KStream的特殊方法。所有这些转换方法可以链接在一起组成一个复杂的处理器拓扑。

These transformation operations are described in the following subsections:

这些转换操作在以下小节中进行介绍：

* [Stateless transformations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless)

* [Stateful transformations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateful)


* [无状态转换](dsl-api.md)

* [有状态转换](dsl-api.md)

## Stateless transformations

## 无状态转换

Stateless transformations do not require state for processing and they do not require a state store associated with the stream processor. Kafka 0.11.0 and later allows you to materialize the result from a stateless ```KTable``` transformation. This allows the result to be queried through [interactive queries](http://kafka.apache.org/11/documentation/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries). To materialize a ```KTable```, each of the below stateless operations [can be augmented](http://kafka.apache.org/11/documentation/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries-local-key-value-stores) with an optional ```queryableStoreName``` argument.

无状态转换不需要进行状态处理，并且不需要与流处理器关联的状态存储器。Kafka 0.11.0和更高版本允许您实现无状态KTable的转换。这将允许您通过[交互式查询](interactive-queries.md)来查询结果。为了实现```KTable```，下面的每个无状态操作都可以使用可选的```queryableStoreName```参数[进行扩充](interactive-queries.md)。

Transformation	Description
Branch

KStream → KStream[]
Branch (or split) a KStream based on the supplied predicates into one or more KStream instances. (details)

Predicates are evaluated in order. A record is placed to one and only one output stream on the first match: if the n-th predicate evaluates to true, the record is placed to n-th stream. If no predicate matches, the the record is dropped.

Branching is useful, for example, to route records to different downstream topics.

KStream<String, Long> stream = ...;
KStream<String, Long>[] branches = stream.branch(
    (key, value) -> key.startsWith("A"), /* first predicate  */
    (key, value) -> key.startsWith("B"), /* second predicate */
    (key, value) -> true                 /* third predicate  */
  );

// KStream branches[0] contains all records whose keys start with "A"
// KStream branches[1] contains all records whose keys start with "B"
// KStream branches[2] contains all other records

// Java 7 example: cf. `filter` for how to create `Predicate` instances
Filter

KStream → KStream
KTable → KTable
Evaluates a boolean function for each element and retains those for which the function returns true. (KStream details, KTable details)

KStream<String, Long> stream = ...;

// A filter that selects (keeps) only positive numbers
// Java 8+ example, using lambda expressions
KStream<String, Long> onlyPositives = stream.filter((key, value) -> value > 0);

// Java 7 example
KStream<String, Long> onlyPositives = stream.filter(
    new Predicate<String, Long>() {
      @Override
      public boolean test(String key, Long value) {
        return value > 0;
      }
    });
Inverse Filter

KStream → KStream
KTable → KTable
Evaluates a boolean function for each element and drops those for which the function returns true. (KStream details, KTable details)

KStream<String, Long> stream = ...;

// An inverse filter that discards any negative numbers or zero
// Java 8+ example, using lambda expressions
KStream<String, Long> onlyPositives = stream.filterNot((key, value) -> value <= 0);

// Java 7 example
KStream<String, Long> onlyPositives = stream.filterNot(
    new Predicate<String, Long>() {
      @Override
      public boolean test(String key, Long value) {
        return value <= 0;
      }
    });
FlatMap

KStream → KStream
Takes one record and produces zero, one, or more records. You can modify the record keys and values, including their types. (details)

Marks the stream for data re-partitioning: Applying a grouping or a join after flatMap will result in re-partitioning of the records. If possible use flatMapValues instead, which will not cause data re-partitioning.

KStream<Long, String> stream = ...;
KStream<String, Integer> transformed = stream.flatMap(
     // Here, we generate two output records for each input record.
     // We also change the key and value types.
     // Example: (345L, "Hello") -> ("HELLO", 1000), ("hello", 9000)
    (key, value) -> {
      List<KeyValue<String, Integer>> result = new LinkedList<>();
      result.add(KeyValue.pair(value.toUpperCase(), 1000));
      result.add(KeyValue.pair(value.toLowerCase(), 9000));
      return result;
    }
  );

// Java 7 example: cf. `map` for how to create `KeyValueMapper` instances
FlatMap (values only)

KStream → KStream
Takes one record and produces zero, one, or more records, while retaining the key of the original record. You can modify the record values and the value type. (details)

flatMapValues is preferable to flatMap because it will not cause data re-partitioning. However, you cannot modify the key or key type like flatMap does.

// Split a sentence into words.
KStream<byte[], String> sentences = ...;
KStream<byte[], String> words = sentences.flatMapValues(value -> Arrays.asList(value.split("\\s+")));

// Java 7 example: cf. `mapValues` for how to create `ValueMapper` instances
Foreach

KStream → void
KStream → void
KTable → void
Terminal operation. Performs a stateless action on each record. (details)

You would use foreach to cause side effects based on the input data (similar to peek) and then stop further processing of the input data (unlike peek, which is not a terminal operation).

Note on processing guarantees: Any side effects of an action (such as writing to external systems) are not trackable by Kafka, which means they will typically not benefit from Kafka’s processing guarantees.

KStream<String, Long> stream = ...;

// Print the contents of the KStream to the local console.
// Java 8+ example, using lambda expressions
stream.foreach((key, value) -> System.out.println(key + " => " + value));

// Java 7 example
stream.foreach(
    new ForeachAction<String, Long>() {
      @Override
      public void apply(String key, Long value) {
        System.out.println(key + " => " + value);
      }
    });
GroupByKey

KStream → KGroupedStream
Groups the records by the existing key. (details)

Grouping is a prerequisite for aggregating a stream or a table and ensures that data is properly partitioned (“keyed”) for subsequent operations.

When to set explicit SerDes: Variants of groupByKey exist to override the configured default SerDes of your application, which you must do if the key and/or value types of the resulting KGroupedStream do not match the configured default SerDes.

Note

Grouping vs. Windowing: A related operation is windowing, which lets you control how to “sub-group” the grouped records of the same key into so-called windows for stateful operations such as windowed aggregations or windowed joins.

Causes data re-partitioning if and only if the stream was marked for re-partitioning. groupByKey is preferable to groupBy because it re-partitions data only if the stream was already marked for re-partitioning. However, groupByKey does not allow you to modify the key or key type like groupBy does.

KStream<byte[], String> stream = ...;

// Group by the existing key, using the application's configured
// default serdes for keys and values.
KGroupedStream<byte[], String> groupedStream = stream.groupByKey();

// When the key and/or value types do not match the configured
// default serdes, we must explicitly specify serdes.
KGroupedStream<byte[], String> groupedStream = stream.groupByKey(
    Serialized.with(
      Serdes.ByteArray(), /* key */
      Serdes.String())     /* value */
  );
GroupBy

KStream → KGroupedStream
KTable → KGroupedTable
Groups the records by a new key, which may be of a different key type. When grouping a table, you may also specify a new value and value type. groupBy is a shorthand for selectKey(...).groupByKey(). (KStream details, KTable details)

Grouping is a prerequisite for aggregating a stream or a table and ensures that data is properly partitioned (“keyed”) for subsequent operations.

When to set explicit SerDes: Variants of groupBy exist to override the configured default SerDes of your application, which you must do if the key and/or value types of the resulting KGroupedStream or KGroupedTable do not match the configured default SerDes.

Note

Grouping vs. Windowing: A related operation is windowing, which lets you control how to “sub-group” the grouped records of the same key into so-called windows for stateful operations such as windowed aggregations or windowed joins.

Always causes data re-partitioning: groupBy always causes data re-partitioning. If possible use groupByKey instead, which will re-partition data only if required.

KStream<byte[], String> stream = ...;
KTable<byte[], String> table = ...;

// Java 8+ examples, using lambda expressions

// Group the stream by a new key and key type
KGroupedStream<String, String> groupedStream = stream.groupBy(
    (key, value) -> value,
    Serialized.with(
      Serdes.String(), /* key (note: type was modified) */
      Serdes.String())  /* value */
  );

// Group the table by a new key and key type, and also modify the value and value type.
KGroupedTable<String, Integer> groupedTable = table.groupBy(
    (key, value) -> KeyValue.pair(value, value.length()),
    Serialized.with(
      Serdes.String(), /* key (note: type was modified) */
      Serdes.Integer()) /* value (note: type was modified) */
  );


// Java 7 examples

// Group the stream by a new key and key type
KGroupedStream<String, String> groupedStream = stream.groupBy(
    new KeyValueMapper<byte[], String, String>>() {
      @Override
      public String apply(byte[] key, String value) {
        return value;
      }
    },
    Serialized.with(
      Serdes.String(), /* key (note: type was modified) */
      Serdes.String())  /* value */
  );

// Group the table by a new key and key type, and also modify the value and value type.
KGroupedTable<String, Integer> groupedTable = table.groupBy(
    new KeyValueMapper<byte[], String, KeyValue<String, Integer>>() {
      @Override
      public KeyValue<String, Integer> apply(byte[] key, String value) {
        return KeyValue.pair(value, value.length());
      }
    },
    Serialized.with(
      Serdes.String(), /* key (note: type was modified) */
      Serdes.Integer()) /* value (note: type was modified) */
  );
Map

KStream → KStream
Takes one record and produces one record. You can modify the record key and value, including their types. (details)

Marks the stream for data re-partitioning: Applying a grouping or a join after map will result in re-partitioning of the records. If possible use mapValues instead, which will not cause data re-partitioning.

KStream<byte[], String> stream = ...;

// Java 8+ example, using lambda expressions
// Note how we change the key and the key type (similar to `selectKey`)
// as well as the value and the value type.
KStream<String, Integer> transformed = stream.map(
    (key, value) -> KeyValue.pair(value.toLowerCase(), value.length()));

// Java 7 example
KStream<String, Integer> transformed = stream.map(
    new KeyValueMapper<byte[], String, KeyValue<String, Integer>>() {
      @Override
      public KeyValue<String, Integer> apply(byte[] key, String value) {
        return new KeyValue<>(value.toLowerCase(), value.length());
      }
    });
Map (values only)

KStream → KStream
KTable → KTable
Takes one record and produces one record, while retaining the key of the original record. You can modify the record value and the value type. (KStream details, KTable details)

mapValues is preferable to map because it will not cause data re-partitioning. However, it does not allow you to modify the key or key type like map does.

KStream<byte[], String> stream = ...;

// Java 8+ example, using lambda expressions
KStream<byte[], String> uppercased = stream.mapValues(value -> value.toUpperCase());

// Java 7 example
KStream<byte[], String> uppercased = stream.mapValues(
    new ValueMapper<String>() {
      @Override
      public String apply(String s) {
        return s.toUpperCase();
      }
    });
Peek

KStream → KStream
Performs a stateless action on each record, and returns an unchanged stream. (details)

You would use peek to cause side effects based on the input data (similar to foreach) and continue processing the input data (unlike foreach, which is a terminal operation). peek returns the input stream as-is; if you need to modify the input stream, use map or mapValues instead.

peek is helpful for use cases such as logging or tracking metrics or for debugging and troubleshooting.

Note on processing guarantees: Any side effects of an action (such as writing to external systems) are not trackable by Kafka, which means they will typically not benefit from Kafka’s processing guarantees.

KStream<byte[], String> stream = ...;

// Java 8+ example, using lambda expressions
KStream<byte[], String> unmodifiedStream = stream.peek(
    (key, value) -> System.out.println("key=" + key + ", value=" + value));

// Java 7 example
KStream<byte[], String> unmodifiedStream = stream.peek(
    new ForeachAction<byte[], String>() {
      @Override
      public void apply(byte[] key, String value) {
        System.out.println("key=" + key + ", value=" + value);
      }
    });
Print

KStream → void
Terminal operation. Prints the records to System.out. See Javadocs for serde and toString() caveats. (details)

Calling print() is the same as calling foreach((key, value) -> System.out.println(key + ", " + value))

KStream<byte[], String> stream = ...;
// print to sysout
stream.print();

// print to file with a custom label
stream.print(Printed.toFile("streams.out").withLabel("streams"));
SelectKey

KStream → KStream
Assigns a new key – possibly of a new key type – to each record. (details)

Calling selectKey(mapper) is the same as calling map((key, value) -> mapper(key, value), value).

Marks the stream for data re-partitioning: Applying a grouping or a join after selectKey will result in re-partitioning of the records.

KStream<byte[], String> stream = ...;

// Derive a new record key from the record's value.  Note how the key type changes, too.
// Java 8+ example, using lambda expressions
KStream<String, String> rekeyed = stream.selectKey((key, value) -> value.split(" ")[0])

// Java 7 example
KStream<String, String> rekeyed = stream.selectKey(
    new KeyValueMapper<byte[], String, String>() {
      @Override
      public String apply(byte[] key, String value) {
        return value.split(" ")[0];
      }
    });
Table to Stream

KTable → KStream
Get the changelog stream of this table. (details)

KTable<byte[], String> table = ...;

// Also, a variant of `toStream` exists that allows you
// to select a new key for the resulting stream.
KStream<byte[], String> stream = table.toStream();

## Stateful transformations

## 有状态转换

Stateful transformations depend on state for processing inputs and producing outputs and require a [state store](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-state) associated with the stream processor. For example, in aggregating operations, a windowing state store is used to collect the latest aggregation results per window. In join operations, a windowing state store is used to collect all of the records received so far within the defined window boundary.

有状态转换依赖于处理输入和产生输出的状态，并且需要与流处理器相关联的[状态存储器](../architecture.md)。例如，在聚合（aggregation）操作中，使用窗口状态存储器来收集每个窗口的最新聚合结果。在连接（join）操作中，窗口状态存储器用于收集迄今为止在定义的窗口边界内收到的所有消息。

Note, that state stores are fault-tolerant. In case of failure, Kafka Streams guarantees to fully restore all state stores prior to resuming the processing. See [Fault Tolerance](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-fault-tolerance) for further information.

请注意，该状态存储器是具有容错性的。如果发生故障，Kafka Streams保证完全恢复所有状态存储器来恢复之前的处理。有关更多信息，请参阅[容错性](architecture.md)。

Available stateful transformations in the DSL include:

DSL中可用的有状态转换包括：

* [Aggregating](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating)

* [Joining](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins)

* [Windowing](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing) (as part of aggregations and joins)

* [Applying custom processors and transformers](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-process), which may be stateful, for Processor API integration


* [聚合（Aggregating）](dsl-api.md)

* [连接（Joining）](dsl-api.md)

* [窗口化（Windowing）（作为聚合和连接的一部分）](dsl-api.md)

* [应用自定义的处理器和转换器](dsl-api.md)对集成处理器API来说可能是有状态的

The following diagram shows their relationships:

下图显示了它们之间的关系：

![](../../../imgs/streams-stateful_operations.png)

Stateful transformations in the DSL.

DSL中的状态转换。

Here is an example of a stateful application: the WordCount algorithm.

以下是有状态应用程序的示例：WordCount算法。

WordCount example in Java 8+, using lambda expressions:

Java 8+中的WordCount示例，使用lambda表达式：

```java
// Assume the record values represent lines of text.  For the sake of this example, you can ignore
// whatever may be stored in the record keys.

// 假设消息值代表文本行。在这个例子中，您可以忽略
// 消息键中可能存储的任何内容。
KStream<String, String> textLines = ...;

KStream<String, Long> wordCounts = textLines
    // Split each text line, by whitespace, into words.  The text lines are the record
    // values, i.e. you can ignore whatever data is in the record keys and thus invoke
    // `flatMapValues` instead of the more generic `flatMap`.

    // 将每个文本行按空格拆分为单词。文本行是消息值，即您可以忽略// 消息键中的任何数据，从而调用“flatMapValues”而不是更通用的// “flatMap”。
    .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
    // Group the stream by word to ensure the key of the record is the word.

    // 逐字分组以确保消息的关键字是单词。
    .groupBy((key, word) -> word)
    // Count the occurrences of each word (record key).
    // 计算每个单词（消息键）的出现次数。
    //
    // This will change the stream type from `KGroupedStream<String, String>` to
    // `KTable<String, Long>` (word -> count).

    // 这将流的类型从`KGroupedStream<String, String>`转换为//`KTable<String, Long>` (word -> count)。
    .count()
    // Convert the `KTable<String, Long>` into a `KStream<String, Long>`.

    // 将`KTable<String, Long>`转化为`KStream<String, Long>`。
    .toStream();
```

WordCount example in Java 7:

Java 7中的WordCount示例：

```java
// Code below is equivalent to the previous Java 8+ example above.

// 下面的代码等同于前面的Java 8+示例。
KStream<String, String> textLines = ...;

KStream<String, Long> wordCounts = textLines
    .flatMapValues(new ValueMapper<String, Iterable<String>>() {
        @Override
        public Iterable<String> apply(String value) {
            return Arrays.asList(value.toLowerCase().split("\\W+"));
        }
    })
    .groupBy(new KeyValueMapper<String, String, String>>() {
        @Override
        public String apply(String key, String word) {
            return word;
        }
    })
    .count()
    .toStream();
```

## Aggregating

## 聚合

After records are [grouped](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless) by key via ```groupByKey``` or ```groupBy``` – and thus represented as either a ```KGroupedStream``` or a ```KGroupedTable```, they can be aggregated via an operation such as ```reduce```. Aggregations are key-based operations, which means that they always operate over records (notably record values) of the same key. You can perform aggregations on [windowed](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing) or non-windowed data.

在通过```groupByKey```或```groupBy```根据键对消息进行[分组](dsl-api.md)——从而将其表示为```KGroupedStream```或```KGroupedTable```，它们可以通过诸如```reduce```之类的操作进行聚合。聚合是基于键的操作，这意味着它们总是对相同键的消息（特别是消息值）进行操作。您可以对[窗口](dsl-api.md)或非窗口数据执行聚合操作。

Transformation	Description
Aggregate

KGroupedStream → KTable
KGroupedTable → KTable
Rolling aggregation. Aggregates the values of (non-windowed) records by the grouped key. Aggregating is a generalization of reduce and allows, for example, the aggregate value to have a different type than the input values. (KGroupedStream details, KGroupedTable details)

When aggregating a grouped stream, you must provide an initializer (e.g., aggValue = 0) and an “adder” aggregator (e.g., aggValue + curValue). When aggregating a grouped table, you must provide a “subtractor” aggregator (think: aggValue - oldValue).

Several variants of aggregate exist, see Javadocs for details.

KGroupedStream<byte[], String> groupedStream = ...;
KGroupedTable<byte[], String> groupedTable = ...;

// Java 8+ examples, using lambda expressions

// Aggregating a KGroupedStream (note how the value type changes from String to Long)
KTable<byte[], Long> aggregatedStream = groupedStream.aggregate(
    () -> 0L, /* initializer */
    (aggKey, newValue, aggValue) -> aggValue + newValue.length(), /* adder */
    Materialized.as("aggregated-stream-store") /* state store name */
        .withValueSerde(Serdes.Long()); /* serde for aggregate value */

// Aggregating a KGroupedTable (note how the value type changes from String to Long)
KTable<byte[], Long> aggregatedTable = groupedTable.aggregate(
    () -> 0L, /* initializer */
    (aggKey, newValue, aggValue) -> aggValue + newValue.length(), /* adder */
    (aggKey, oldValue, aggValue) -> aggValue - oldValue.length(), /* subtractor */
    Materialized.as("aggregated-table-store") /* state store name */
	.withValueSerde(Serdes.Long()) /* serde for aggregate value */


// Java 7 examples

// Aggregating a KGroupedStream (note how the value type changes from String to Long)
KTable<byte[], Long> aggregatedStream = groupedStream.aggregate(
    new Initializer<Long>() { /* initializer */
      @Override
      public Long apply() {
        return 0L;
      }
    },
    new Aggregator<byte[], String, Long>() { /* adder */
      @Override
      public Long apply(byte[] aggKey, String newValue, Long aggValue) {
        return aggValue + newValue.length();
      }
    },
    Materialized.as("aggregated-stream-store")
        .withValueSerde(Serdes.Long());

// Aggregating a KGroupedTable (note how the value type changes from String to Long)
KTable<byte[], Long> aggregatedTable = groupedTable.aggregate(
    new Initializer<Long>() { /* initializer */
      @Override
      public Long apply() {
        return 0L;
      }
    },
    new Aggregator<byte[], String, Long>() { /* adder */
      @Override
      public Long apply(byte[] aggKey, String newValue, Long aggValue) {
        return aggValue + newValue.length();
      }
    },
    new Aggregator<byte[], String, Long>() { /* subtractor */
      @Override
      public Long apply(byte[] aggKey, String oldValue, Long aggValue) {
        return aggValue - oldValue.length();
      }
    },
    Materialized.as("aggregated-stream-store")
        .withValueSerde(Serdes.Long());
Detailed behavior of KGroupedStream:

Input records with null keys are ignored.
When a record key is received for the first time, the initializer is called (and called before the adder).
Whenever a record with a non-null value is received, the adder is called.
Detailed behavior of KGroupedTable:

Input records with null keys are ignored.
When a record key is received for the first time, the initializer is called (and called before the adder and subtractor). Note that, in contrast to KGroupedStream, over time the initializer may be called more than once for a key as a result of having received input tombstone records for that key (see below).
When the first non-null value is received for a key (e.g., INSERT), then only the adder is called.
When subsequent non-null values are received for a key (e.g., UPDATE), then (1) the subtractor is called with the old value as stored in the table and (2) the adder is called with the new value of the input record that was just received. The order of execution for the subtractor and adder is not defined.
When a tombstone record – i.e. a record with a null value – is received for a key (e.g., DELETE), then only the subtractor is called. Note that, whenever the subtractor returns a null value itself, then the corresponding key is removed from the resulting KTable. If that happens, any next input record for that key will trigger the initializer again.
See the example at the bottom of this section for a visualization of the aggregation semantics.

Aggregate (windowed)

KGroupedStream → KTable
Windowed aggregation. Aggregates the values of records, per window, by the grouped key. Aggregating is a generalization of reduce and allows, for example, the aggregate value to have a different type than the input values. (TimeWindowedKStream details, SessionWindowedKStream details)

You must provide an initializer (e.g., aggValue = 0), “adder” aggregator (e.g., aggValue + curValue), and a window. When windowing based on sessions, you must additionally provide a “session merger” aggregator (e.g., mergedAggValue = leftAggValue + rightAggValue).

The windowed aggregate turns a TimeWindowedKStream<K, V> or SessionWindowdKStream<K, V> into a windowed KTable<Windowed<K>, V>.

Several variants of aggregate exist, see Javadocs for details.

import java.util.concurrent.TimeUnit;
KGroupedStream<String, Long> groupedStream = ...;

// Java 8+ examples, using lambda expressions

// Aggregating with time-based windowing (here: with 5-minute tumbling windows)
KTable<Windowed<String>, Long> timeWindowedAggregatedStream = groupedStream.windowedBy(TimeUnit.MINUTES.toMillis(5))
    .aggregate(
      () -> 0L, /* initializer */
    	(aggKey, newValue, aggValue) -> aggValue + newValue, /* adder */
      Materialized.<String, Long, WindowStore<Bytes, byte[]>>as("time-windowed-aggregated-stream-store") /* state store name */
        .withValueSerde(Serdes.Long())); /* serde for aggregate value */

// Aggregating with session-based windowing (here: with an inactivity gap of 5 minutes)
KTable<Windowed<String>, Long> sessionizedAggregatedStream = groupedStream.windowedBy(SessionWindows.with(TimeUnit.MINUTES.toMillis(5)).
    aggregate(
    	() -> 0L, /* initializer */
    	(aggKey, newValue, aggValue) -> aggValue + newValue, /* adder */
    	(aggKey, leftAggValue, rightAggValue) -> leftAggValue + rightAggValue, /* session merger */
	    Materialized.<String, Long, SessionStore<Bytes, byte[]>>as("sessionized-aggregated-stream-store") /* state store name */
        .withValueSerde(Serdes.Long())); /* serde for aggregate value */

// Java 7 examples

// Aggregating with time-based windowing (here: with 5-minute tumbling windows)
KTable<Windowed<String>, Long> timeWindowedAggregatedStream = groupedStream.windowedBy(TimeUnit.MINUTES.toMillis(5))
    .aggregate(
        new Initializer<Long>() { /* initializer */
            @Override
            public Long apply() {
                return 0L;
            }
        },
        new Aggregator<String, Long, Long>() { /* adder */
            @Override
            public Long apply(String aggKey, Long newValue, Long aggValue) {
                return aggValue + newValue;
            }
        },
        Materialized.<String, Long, WindowStore<Bytes, byte[]>>as("time-windowed-aggregated-stream-store")
          .withValueSerde(Serdes.Long()));

// Aggregating with session-based windowing (here: with an inactivity gap of 5 minutes)
KTable<Windowed<String>, Long> sessionizedAggregatedStream = groupedStream.windowedBy(SessionWindows.with(TimeUnit.MINUTES.toMillis(5)).
    aggregate(
        new Initializer<Long>() { /* initializer */
            @Override
            public Long apply() {
                return 0L;
            }
        },
        new Aggregator<String, Long, Long>() { /* adder */
            @Override
            public Long apply(String aggKey, Long newValue, Long aggValue) {
                return aggValue + newValue;
            }
        },
        new Merger<String, Long>() { /* session merger */
            @Override
            public Long apply(String aggKey, Long leftAggValue, Long rightAggValue) {
                return rightAggValue + leftAggValue;
            }
        },
        Materialized.<String, Long, SessionStore<Bytes, byte[]>>as("sessionized-aggregated-stream-store")
          .withValueSerde(Serdes.Long()));
Detailed behavior:

The windowed aggregate behaves similar to the rolling aggregate described above. The additional twist is that the behavior applies per window.
Input records with null keys are ignored in general.
When a record key is received for the first time for a given window, the initializer is called (and called before the adder).
Whenever a record with a non-null value is received for a given window, the adder is called.
When using session windows: the session merger is called whenever two sessions are being merged.
See the example at the bottom of this section for a visualization of the aggregation semantics.

Count

KGroupedStream → KTable
KGroupedTable → KTable
Rolling aggregation. Counts the number of records by the grouped key. (KGroupedStream details, KGroupedTable details)

Several variants of count exist, see Javadocs for details.

KGroupedStream<String, Long> groupedStream = ...;
KGroupedTable<String, Long> groupedTable = ...;

// Counting a KGroupedStream
KTable<String, Long> aggregatedStream = groupedStream.count();

// Counting a KGroupedTable
KTable<String, Long> aggregatedTable = groupedTable.count();
Detailed behavior for KGroupedStream:

Input records with null keys or values are ignored.
Detailed behavior for KGroupedTable:

Input records with null keys are ignored. Records with null values are not ignored but interpreted as “tombstones” for the corresponding key, which indicate the deletion of the key from the table.
Count (windowed)

KGroupedStream → KTable
Windowed aggregation. Counts the number of records, per window, by the grouped key. (TimeWindowedKStream details, SessionWindowedKStream details)

The windowed count turns a TimeWindowedKStream<K, V> or SessionWindowedKStream<K, V> into a windowed KTable<Windowed<K>, V>.

Several variants of count exist, see Javadocs for details.

import java.util.concurrent.TimeUnit;
KGroupedStream<String, Long> groupedStream = ...;

// Counting a KGroupedStream with time-based windowing (here: with 5-minute tumbling windows)
KTable<Windowed<String>, Long> aggregatedStream = groupedStream.windowedBy(
    TimeWindows.of(TimeUnit.MINUTES.toMillis(5))) /* time-based window */
    .count();

// Counting a KGroupedStream with session-based windowing (here: with 5-minute inactivity gaps)
KTable<Windowed<String>, Long> aggregatedStream = groupedStream.windowedBy(
    SessionWindows.with(TimeUnit.MINUTES.toMillis(5))) /* session window */
    .count();
Detailed behavior:

Input records with null keys or values are ignored.
Reduce

KGroupedStream → KTable
KGroupedTable → KTable
Rolling aggregation. Combines the values of (non-windowed) records by the grouped key. The current record value is combined with the last reduced value, and a new reduced value is returned. The result value type cannot be changed, unlike aggregate. (KGroupedStream details, KGroupedTable details)

When reducing a grouped stream, you must provide an “adder” reducer (e.g., aggValue + curValue). When reducing a grouped table, you must additionally provide a “subtractor” reducer (e.g., aggValue - oldValue).

Several variants of reduce exist, see Javadocs for details.

KGroupedStream<String, Long> groupedStream = ...;
KGroupedTable<String, Long> groupedTable = ...;

// Java 8+ examples, using lambda expressions

// Reducing a KGroupedStream
KTable<String, Long> aggregatedStream = groupedStream.reduce(
    (aggValue, newValue) -> aggValue + newValue /* adder */);

// Reducing a KGroupedTable
KTable<String, Long> aggregatedTable = groupedTable.reduce(
    (aggValue, newValue) -> aggValue + newValue, /* adder */
    (aggValue, oldValue) -> aggValue - oldValue /* subtractor */);


// Java 7 examples

// Reducing a KGroupedStream
KTable<String, Long> aggregatedStream = groupedStream.reduce(
    new Reducer<Long>() { /* adder */
      @Override
      public Long apply(Long aggValue, Long newValue) {
        return aggValue + newValue;
      }
    });

// Reducing a KGroupedTable
KTable<String, Long> aggregatedTable = groupedTable.reduce(
    new Reducer<Long>() { /* adder */
      @Override
      public Long apply(Long aggValue, Long newValue) {
        return aggValue + newValue;
      }
    },
    new Reducer<Long>() { /* subtractor */
      @Override
      public Long apply(Long aggValue, Long oldValue) {
        return aggValue - oldValue;
      }
    });
Detailed behavior for KGroupedStream:

Input records with null keys are ignored in general.
When a record key is received for the first time, then the value of that record is used as the initial aggregate value.
Whenever a record with a non-null value is received, the adder is called.
Detailed behavior for KGroupedTable:

Input records with null keys are ignored in general.
When a record key is received for the first time, then the value of that record is used as the initial aggregate value. Note that, in contrast to KGroupedStream, over time this initialization step may happen more than once for a key as a result of having received input tombstone records for that key (see below).
When the first non-null value is received for a key (e.g., INSERT), then only the adder is called.
When subsequent non-null values are received for a key (e.g., UPDATE), then (1) the subtractor is called with the old value as stored in the table and (2) the adder is called with the new value of the input record that was just received. The order of execution for the subtractor and adder is not defined.
When a tombstone record – i.e. a record with a null value – is received for a key (e.g., DELETE), then only the subtractor is called. Note that, whenever the subtractor returns a null value itself, then the corresponding key is removed from the resulting KTable. If that happens, any next input record for that key will re-initialize its aggregate value.
See the example at the bottom of this section for a visualization of the aggregation semantics.

Reduce (windowed)

KGroupedStream → KTable
Windowed aggregation. Combines the values of records, per window, by the grouped key. The current record value is combined with the last reduced value, and a new reduced value is returned. Records with null key or value are ignored. The result value type cannot be changed, unlike aggregate. (TimeWindowedKStream details, SessionWindowedKStream details)

The windowed reduce turns a turns a TimeWindowedKStream<K, V> or a SessionWindowedKStream<K, V> into a windowed KTable<Windowed<K>, V>.

Several variants of reduce exist, see Javadocs for details.

import java.util.concurrent.TimeUnit;
KGroupedStream<String, Long> groupedStream = ...;

// Java 8+ examples, using lambda expressions

// Aggregating with time-based windowing (here: with 5-minute tumbling windows)
KTable<Windowed<String>, Long> timeWindowedAggregatedStream = groupedStream.windowedBy(
  TimeWindows.of(TimeUnit.MINUTES.toMillis(5)) /* time-based window */)
  .reduce(
    (aggValue, newValue) -> aggValue + newValue /* adder */
  );

// Aggregating with session-based windowing (here: with an inactivity gap of 5 minutes)
KTable<Windowed<String>, Long> sessionzedAggregatedStream = groupedStream.windowedBy(
  SessionWindows.with(TimeUnit.MINUTES.toMillis(5))) /* session window */
  .reduce(
    (aggValue, newValue) -> aggValue + newValue /* adder */
  );


// Java 7 examples

// Aggregating with time-based windowing (here: with 5-minute tumbling windows)
KTable<Windowed<String>, Long> timeWindowedAggregatedStream = groupedStream..windowedBy(
  TimeWindows.of(TimeUnit.MINUTES.toMillis(5)) /* time-based window */)
  .reduce(
    new Reducer<Long>() { /* adder */
      @Override
      public Long apply(Long aggValue, Long newValue) {
        return aggValue + newValue;
      }
    });

// Aggregating with session-based windowing (here: with an inactivity gap of 5 minutes)
KTable<Windowed<String>, Long> timeWindowedAggregatedStream = groupedStream.windowedBy(
  SessionWindows.with(TimeUnit.MINUTES.toMillis(5))) /* session window */
  .reduce(
    new Reducer<Long>() { /* adder */
      @Override
      public Long apply(Long aggValue, Long newValue) {
        return aggValue + newValue;
      }
    });
Detailed behavior:

The windowed reduce behaves similar to the rolling reduce described above. The additional twist is that the behavior applies per window.
Input records with null keys are ignored in general.
When a record key is received for the first time for a given window, then the value of that record is used as the initial aggregate value.
Whenever a record with a non-null value is received for a given window, the adder is called.
See the example at the bottom of this section for a visualization of the aggregation semantics.

**Example of semantics for stream aggregations:** A ```KGroupedStream``` → ```KTable``` example is shown below. The streams and the table are initially empty. Bold font is used in the column for “KTable ```aggregated”``` to highlight changed state. An entry such as ```(hello, 1)``` denotes a record with key ```hello``` and value ```1```. To improve the readability of the semantics table you can assume that all records are processed in timestamp order.

**流聚合的语义示例：**```KGroupedStream```→```KTable```示例如下所示。流和表最初是空的。粗体字用于“KTable```聚合```”列以突出显示更改的状态。诸如```(hello，1)```之类的条目表示具有键```hello```和值```1```的消息。为了提高语义表的可读性，您可以假设所有消息都按时间戳顺序处理。

```java
// Key: word, value: count

// 键：word，值：count
KStream<String, Integer> wordCounts = ...;

KGroupedStream<String, Integer> groupedStream = wordCounts
    .groupByKey(Serialized.with(Serdes.String(), Serdes.Integer()));

KTable<String, Integer> aggregated = groupedStream.aggregate(
    () -> 0, /* initializer */
    (aggKey, newValue, aggValue) -> aggValue + newValue, /* adder */
    Materialized.<String, Long, KeyValueStore<Bytes, byte[]>as("aggregated-stream-store" /* state store name */)
      .withKeySerde(Serdes.String()) /* key serde */
      .withValueSerde(Serdes.Integer()); /* serde for aggregate value */
```

### Note

### 提示

**Impact of record caches:** For illustration purposes, the column “KTable ```aggregated```” below shows the table’s state changes over time in a very granular way. In practice, you would observe state changes in such a granular way only when [record caches](http://kafka.apache.org/11/documentation/streams/developer-guide/memory-mgmt.html#streams-developer-guide-memory-management-record-cache) are disabled (default: enabled). When record caches are enabled, what might happen for example is that the output results of the rows with timestamps 4 and 5 would be [compacted](http://kafka.apache.org/11/documentation/streams/developer-guide/memory-mgmt.html#streams-developer-guide-memory-management-record-cache), and there would only be a single state update for the key ```kafka``` in the KTable (here: from ```(kafka 1)``` directly to ```(kafka, 3)```. Typically, you should only disable record caches for testing or debugging purposes – under normal circumstances it is better to leave record caches enabled.

**消息缓存的影响：**出于解释说明的目的，下面的“KTable```聚合```”列以非常细化的方式显示了表的状态随时间的变化。在实践中，只有当[消息缓存](memory-mgmt.md)被禁用时（默认：启用），才能以这种细粒度的方式观察状态的更新。当启用记录缓存时，可能会发生的情况是，具有时间戳4和5的行的输出结果将被[压缩](memory-mgmt.md)，并且KTable中的键```kafka```只会进行单个状态更新（这里是：从```(kafka 1)```直接更新成```(kafka，3)```。通常情况下，您只能通过禁用消息缓存才能进行测试或调试——在正常情况下，最好将记录缓存启用。

KStream `wordCounts` KGroupedStream `groupedStream` 	KTable `aggregated`
Timestamp Input record Grouping Initializer Adder State
1	(hello, 1)	(hello, 1)	0 (for hello)	(hello, 0 + 1)	
(hello, 1)
2	(kafka, 1)	(kafka, 1)	0 (for kafka)	(kafka, 0 + 1)	
(hello, 1)
(kafka, 1)
3	(streams, 1)	(streams, 1)	0 (for streams)	(streams, 0 + 1)	
(hello, 1)
(kafka, 1)
(streams, 1)
4	(kafka, 1)	(kafka, 1)	 	(kafka, 1 + 1)	
(hello, 1)
(kafka, 2)
(streams, 1)
5	(kafka, 1)	(kafka, 1)	 	(kafka, 2 + 1)	
(hello, 1)
(kafka, 3)
(streams, 1)
6	(streams, 1)	(streams, 1)	 	(streams, 1 + 1)	
(hello, 1)
(kafka, 3)
(streams, 2)

**Example of semantics for table aggregations:** A ```KGroupedTable``` → ```KTable``` example is shown below. The tables are initially empty. Bold font is used in the column for “KTable ```aggregated```” to highlight changed state. An entry such as ```(hello, 1)``` denotes a record with key ```hello``` and value ```1```. To improve the readability of the semantics table you can assume that all records are processed in timestamp order.

**表聚合的语义示例：**```KGroupedTable```→```KTable```示例如下所示。表最初是空的。粗体字用于“KTable```聚合```”列以突出显示更改的状态。诸如```(hello，1)```之类的条目表示具有键```hello```和值```1```的消息。为了提高语义表的可读性，您可以假设所有消息都按时间戳顺序处理。

```java
// Key: username, value: user region (abbreviated to "E" for "Europe", "A" for "Asia")

// 键：username，值：user region("E"开头代表"Europe", "A"开头代表"Asia")
KTable<String, String> userProfiles = ...;

// Re-group `userProfiles`.  Don't read too much into what the grouping does:
// its prime purpose in this example is to show the *effects* of the grouping
// in the subsequent aggregation.

// 重新组合userProfiles。不要过多读入分组的内容：本例
// 的主要目的是显示分组在后续聚合中的*效果*。
KGroupedTable<String, Integer> groupedTable = userProfiles
    .groupBy((user, region) -> KeyValue.pair(region, user.length()), Serdes.String(), Serdes.Integer());

KTable<String, Integer> aggregated = groupedTable.aggregate(
    () -> 0, /* initializer */
    (aggKey, newValue, aggValue) -> aggValue + newValue, /* adder */
    (aggKey, oldValue, aggValue) -> aggValue - oldValue, /* subtractor */
    Materialized.<String, Long, KeyValueStore<Bytes, byte[]>as("aggregated-table-store" /* state store name */)
      .withKeySerde(Serdes.String()) /* key serde */
      .withValueSerde(Serdes.Integer()); /* serde for aggregate value */
```

### Note

### 提示

**Impact of record caches:** For illustration purposes, the column “KTable ```aggregated```” below shows the table’s state changes over time in a very granular way. In practice, you would observe state changes in such a granular way only when [record caches](http://kafka.apache.org/11/documentation/streams/developer-guide/memory-mgmt.html#streams-developer-guide-memory-management-record-cache) are disabled (default: enabled). When record caches are enabled, what might happen for example is that the output results of the rows with timestamps 4 and 5 would be [compacted](http://kafka.apache.org/11/documentation/streams/developer-guide/memory-mgmt.html#streams-developer-guide-memory-management-record-cache), and there would only be a single state update for the key ```kafka``` in the KTable (here: from ```(kafka 1)``` directly to ```(kafka, 3)```. Typically, you should only disable record caches for testing or debugging purposes – under normal circumstances it is better to leave record caches enabled.

**消息缓存的影响：**出于解释说明的目的，下面的“KTable```聚合```”列以非常细化的方式显示了表的状态随时间的变化。在实践中，只有当[消息缓存](memory-mgmt.md)被禁用时（默认：启用），才能以这种细粒度的方式观察状态的更新。当启用消息缓存时，可能会发生的情况是，具有时间戳4和5的行的输出结果将被[压缩](memory-mgmt.md)，并且KTable中的键```kafka```只会进行单个状态更新（这里是：从```(kafka 1)```直接更新成```(kafka，3)```。通常情况下，您只能通过禁用消息缓存才能进行测试或调试——在正常情况下，最好将消息缓存启用。

 	KTable userProfiles	KGroupedTable groupedTable	KTable aggregated
Timestamp	Input record	Interpreted as	Grouping	Initializer	Adder	Subtractor	State
1	(alice, E)	INSERT alice	(E, 5)	0 (for E)	(E, 0 + 5)	 	
(E, 5)
2	(bob, A)	INSERT bob	(A, 3)	0 (for A)	(A, 0 + 3)	 	
(A, 3)
(E, 5)
3	(charlie, A)	INSERT charlie	(A, 7)	 	(A, 3 + 7)	 	
(A, 10)
(E, 5)
4	(alice, A)	UPDATE alice	(A, 5)	 	(A, 10 + 5)	(E, 5 - 5)	
(A, 15)
(E, 0)
5	(charlie, null)	DELETE charlie	(null, 7)	 	 	(A, 15 - 7)	
(A, 8)
(E, 0)
6	(null, E)	ignored	 	 	 	 	
(A, 8)
(E, 0)
7	(bob, E)	UPDATE bob	(E, 3)	 	(E, 0 + 3)	(A, 8 - 3)	
(A, 5)
(E, 3)

## Joining

## 连接

Streams and tables can also be joined. Many stream processing applications in practice are coded as streaming joins. For example, applications backing an online shop might need to access multiple, updating database tables (e.g. sales prices, inventory, customer information) in order to enrich a new data record (e.g. customer transaction) with context information. That is, scenarios where you need to perform table lookups at very large scale and with a low processing latency. Here, a popular pattern is to make the information in the databases available in Kafka through so-called change data capture in combination with [Kafka’s Connect API](http://kafka.apache.org/11/documentation/connect/index.html#kafka-connect), and then implementing applications that leverage the Streams API to perform very fast and efficient local joins of such tables and streams, rather than requiring the application to make a query to a remote database over the network for each record. In this example, the KTable concept in Kafka Streams would enable you to track the latest state (e.g., snapshot) of each table in a local state store, thus greatly reducing the processing latency as well as reducing the load of the remote databases when doing such streaming joins.

流和表也可以连接。在实际中许多流处理应用程序都以代码实现为流连接。例如，支持在线商店的应用程序可能需要访问多个数据库，更新数据库表格（例如销售价格，库存，客户信息）以便利用上下文信息丰富新的数据记录（例如，客户交易）。也就是说，您需要以超大规模和低处理延迟来执行表的查找操作。在这里，一种流行的方式是通过所谓的更改数据抓取与[Kafka Connect API](../../kafka_connect.md)相结合，使Kafka中数据库的信息可用，然后实现利用Streams API执行非常快速且高效的本地表连接的应用程序和流，而不是要求应用程序通过网络为每条记录查询远程数据库。在本例中，Kafka Streams中的KTable概念将使您能够跟踪本地状态存储器中每个表的最新状态（例如快照），从而在执行这样的流式连接时，大大减少处理延迟，并在执行操作时降低远程数据库的负载。

The following join operations are supported, see also the diagram in the [overview section](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateful-overview) of [Stateful Transformations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateful). Depending on the operands, joins are either [windowed](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing) joins or non-windowed joins.

以下连接操作能得到支持，另请参阅[状态转换](dsl-api.md)[概述部分](dsl-api.md)的图表。根据操作数，连接可以是[窗口](dsl-api.md)连接或非窗口连接。

Join operands | Type | (INNER) JOIN | LEFT JOIN | OUTER JOIN
--- | --- | --- | --- | ---
KStream-to-KStream | Windowed | Supported | Supported |	Supported
KTable-to-KTable | Non-windowed |	Supported |	Supported |	Supported
KStream-to-KTable	| Non-windowed | Supported | Supported | Not Supported
KStream-to-GlobalKTable |	Non-windowed | Supported | Supported | Not Supported
KTable-to-GlobalKTable | N/A | Not Supported | Not Supported | Not Supported

连接操作数 | 类型 | （内）连接 | 左连接 | 外连接
--- | --- | --- | --- | ---
KStream-to-KStream | 窗口 | 支持 | 支持 | 支持的
KTable-to-KTable | 非窗口 | 支持 | 支持 | 支持的
KStream-to-KTable | 非窗口 | 支持 | 支持 | 不支持
KStream-to-GlobalKTable | 非窗口 | 支持 | 支持 | 不支持
KTable-to-GlobalKTable | N/A | 不支持 | 不支持 | 不支持

Each case is explained in more detail in the subsequent sections.

每个案例在后面的章节中都会有更详细的解释。

### Join co-partitioning requirements

### 连接共分区（co-partitioning）的要求

Input data must be co-partitioned when joining. This ensures that input records with the same key, from both sides of the join, are delivered to the same stream task during processing. **It is the responsibility of the user to ensure data co-partitioning when joining.**

连接时必须对输入数据进行共分区（co-partitioned）。这确保了在处理期间，要连接的两侧的具有相同键的输入消息被传送到相同的流任务。**加入时确保数据共分区是用户的责任。**

### Tip

### 提示

If possible, consider using [global tables](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-globalktable) (```GlobalKTable```) for joining because they do not require data co-partitioning.

如果可能，请考虑使用[全局表](../concepts.md)（```GlobalKTable```）进行连接，因为它们不需要数据共分区。

The requirements for data co-partitioning are:

数据共分区的要求是：

* The input topics of the join (left side and right side) must have the **same number of partitions.**

* 连接的输入主题（左侧和右侧）必须具有**相同数量的分区**。

* All applications that write to the input topics must have the **same partitioning strategy** so that records with the same key are delivered to same partition number. In other words, the keyspace of the input data must be distributed across partitions in the same manner. This means that, for example, applications that use Kafka’s [Java Producer API](http://kafka.apache.org/11/documentation/clients/index.html#kafka-clients) must use the same partitioner (cf. the producer setting ```"partitioner.class"``` aka ```ProducerConfig.PARTITIONER_CLASS_CONFIG```), and applications that use the Kafka’s Streams API must use the same ```StreamPartitioner``` for operations such as ```KStream#to()```. The good news is that, if you happen to use the default partitioner-related settings across all applications, you do not need to worry about the partitioning strategy.

* 所有写入输入主题的应用程序必须具有**相同的分区策略**，以便将具有相同键的消息传送到相同的分区号。换句话说，输入数据的键空间必须以相同的方式分布在分区中。这意味着，例如，使用Kafka的[Java Producer API](../../../clients.md)的应用程序必须使用相同的分区程序（参见生产者设置中的```"partitioner.class"```，也就是```ProducerConfig.PARTITIONER_CLASS_CONFIG```），而使用Kafka的Streams API的应用程序必须使用相同的```StreamPartitioner```诸如```KStream＃to()```之类的操作。好消息是，如果您碰巧在所有应用程序中使用默认的与分区相关的设置，则无需担心分区策略。

Why is data co-partitioning required? Because [KStream-KStream](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-kstream), [KTable-KTable](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-ktable-ktable), and [KStream-KTable](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-ktable) joins are performed based on the keys of records (e.g., ```leftRecord.key == rightRecord.key```), it is required that the input streams/tables of a join are co-partitioned by key.

为什么需要数据共分区？由于[KStream-KStream](dsl-api.md)，[KTable-KTable](dsl-api.md)和[KStream-KTable](dsl-api.md)连接是基于消息的键（例如，```leftRecord.key == rightRecord.key```）来执行的，因此要求连接的输入流/表根据键来共分区。

The only exception are [KStream-GlobalKTable joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-globalktable). Here, co-partitioning is it not required because all partitions of the ```GlobalKTable```‘s underlying changelog stream are made available to each ```KafkaStreams``` instance, i.e. each instance has a full copy of the changelog stream. Further, a ```KeyValueMapper``` allows for non-key based joins from the ```KStream``` to the ```GlobalKTable```.

唯一的例外是[KStream-GlobalKTable连接](dsl-api.md)。在这里，不需要共分区，因为```GlobalKTable```底层的更新日志流的所有分区都可用于每个```KafkaStreams```实例，即每个实例都具有更新日志流的完整副本。此外，```KeyValueMapper```允许从```KStream```到```GlobalKTable```的非基于键的连接。

### Note

**Kafka Streams partly verifies the co-partitioning requirement:** During the partition assignment step, i.e. at runtime, Kafka Streams verifies whether the number of partitions for both sides of a join are the same. If they are not, a ```TopologyBuilderException``` (runtime exception) is being thrown. Note that Kafka Streams cannot verify whether the partitioning strategy matches between the input streams/tables of a join – it is up to the user to ensure that this is the case.

**Ensuring data co-partitioning:** If the inputs of a join are not co-partitioned yet, you must ensure this manually. You may follow a procedure such as outlined below.

1. Identify the input KStream/KTable in the join whose underlying Kafka topic has the smaller number of partitions. Let’s call this stream/table “SMALLER”, and the other side of the join “LARGER”. To learn about the number of partitions of a Kafka topic you can use, for example, the CLI tool ```bin/kafka-topics``` with the ```--describe``` option.

2. Pre-create a new Kafka topic for “SMALLER” that has the same number of partitions as “LARGER”. Let’s call this new topic “repartitioned-topic-for-smaller”. Typically, you’d use the CLI tool ```bin/kafka-topics``` with the ```--create``` option for this.

3. Within your application, re-write the data of “SMALLER” into the new Kafka topic. You must ensure that, when writing the data with ```to``` or ```through```, the same partitioner is used as for “LARGER”.

    * If “SMALLER” is a KStream: ```KStream#to("repartitioned-topic-for-smaller")```.
    * If “SMALLER” is a KTable: ```KTable#to("repartitioned-topic-for-smaller")```.

4. Within your application, re-read the data in “repartitioned-topic-for-smaller” into a new KStream/KTable.

    * If “SMALLER” is a KStream: ```StreamsBuilder#stream("repartitioned-topic-for-smaller")```.
    * If “SMALLER” is a KTable: ```StreamsBuilder#table("repartitioned-topic-for-smaller")```.

5. Within your application, perform the join between “LARGER” and the new stream/table.

### KStream-KStream Join

KStream-KStream joins are always [windowed](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-sliding) joins, because otherwise the size of the internal state store used to perform the join – e.g., a [sliding window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-sliding) or “buffer” – would grow indefinitely. For stream-stream joins it’s important to highlight that a new input record on one side will produce a join output for each matching record on the other side, and there can be multiple such matching records in a given join window (cf. the row with timestamp 15 in the join semantics table below, for example).

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

Transformation	Description
Inner Join (windowed)

(KStream, KStream) → KStream
Performs an INNER JOIN of this stream with another stream. Even though this operation is windowed, the joined stream will be of type KStream<K, ...> rather than KStream<Windowed<K>, ...>. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

Causes data re-partitioning of a stream if and only if the stream was marked for re-partitioning (if both are marked, both are re-partitioned).

Several variants of join exists, see the Javadocs for details.

import java.util.concurrent.TimeUnit;
KStream<String, Long> left = ...;
KStream<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.join(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue, /* ValueJoiner */
    JoinWindows.of(TimeUnit.MINUTES.toMillis(5)),
    Joined.with(
      Serdes.String(), /* key */
      Serdes.Long(),   /* left value */
      Serdes.Double())  /* right value */
  );

// Java 7 example
KStream<String, String> joined = left.join(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    },
    JoinWindows.of(TimeUnit.MINUTES.toMillis(5)),
    Joined.with(
      Serdes.String(), /* key */
      Serdes.Long(),   /* left value */
      Serdes.Double())  /* right value */
  );
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key, and window-based, i.e. two input records are joined if and only if their timestamps are “close” to each other as defined by the user-supplied JoinWindows, i.e. the window defines an additional join predicate over the record timestamps.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Input records with a null key or a null value are ignored and do not trigger the join.
See the semantics overview at the bottom of this section for a detailed description.

Left Join (windowed)

(KStream, KStream) → KStream
Performs a LEFT JOIN of this stream with another stream. Even though this operation is windowed, the joined stream will be of type KStream<K, ...> rather than KStream<Windowed<K>, ...>. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

Causes data re-partitioning of a stream if and only if the stream was marked for re-partitioning (if both are marked, both are re-partitioned).

Several variants of leftJoin exists, see the Javadocs for details.

import java.util.concurrent.TimeUnit;
KStream<String, Long> left = ...;
KStream<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.leftJoin(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue, /* ValueJoiner */
    JoinWindows.of(TimeUnit.MINUTES.toMillis(5)),
    Joined.with(
      Serdes.String(), /* key */
      Serdes.Long(),   /* left value */
      Serdes.Double())  /* right value */
  );

// Java 7 example
KStream<String, String> joined = left.leftJoin(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    },
    JoinWindows.of(TimeUnit.MINUTES.toMillis(5)),
    Joined.with(
      Serdes.String(), /* key */
      Serdes.Long(),   /* left value */
      Serdes.Double())  /* right value */
  );
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key, and window-based, i.e. two input records are joined if and only if their timestamps are “close” to each other as defined by the user-supplied JoinWindows, i.e. the window defines an additional join predicate over the record timestamps.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Input records with a null key or a null value are ignored and do not trigger the join.
For each input record on the left side that does not have any match on the right side, the ValueJoiner will be called with ValueJoiner#apply(leftRecord.value, null); this explains the row with timestamp=3 in the table below, which lists [A, null] in the LEFT JOIN column.

See the semantics overview at the bottom of this section for a detailed description.

Outer Join (windowed)

(KStream, KStream) → KStream
Performs an OUTER JOIN of this stream with another stream. Even though this operation is windowed, the joined stream will be of type KStream<K, ...> rather than KStream<Windowed<K>, ...>. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

Causes data re-partitioning of a stream if and only if the stream was marked for re-partitioning (if both are marked, both are re-partitioned).

Several variants of outerJoin exists, see the Javadocs for details.

import java.util.concurrent.TimeUnit;
KStream<String, Long> left = ...;
KStream<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.outerJoin(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue, /* ValueJoiner */
    JoinWindows.of(TimeUnit.MINUTES.toMillis(5)),
    Joined.with(
      Serdes.String(), /* key */
      Serdes.Long(),   /* left value */
      Serdes.Double())  /* right value */
  );

// Java 7 example
KStream<String, String> joined = left.outerJoin(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    },
    JoinWindows.of(TimeUnit.MINUTES.toMillis(5)),
    Joined.with(
      Serdes.String(), /* key */
      Serdes.Long(),   /* left value */
      Serdes.Double())  /* right value */
  );
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key, and window-based, i.e. two input records are joined if and only if their timestamps are “close” to each other as defined by the user-supplied JoinWindows, i.e. the window defines an additional join predicate over the record timestamps.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Input records with a null key or a null value are ignored and do not trigger the join.
For each input record on one side that does not have any match on the other side, the ValueJoiner will be called with ValueJoiner#apply(leftRecord.value, null) or ValueJoiner#apply(null, rightRecord.value), respectively; this explains the row with timestamp=3 in the table below, which lists [A, null] in the OUTER JOIN column (unlike LEFT JOIN, [null, x] is possible, too, but no such example is shown in the table).

See the semantics overview at the bottom of this section for a detailed description.

**Semantics of stream-stream joins:** The semantics of the various stream-stream join variants are explained below. To improve the readability of the table, assume that (1) all records have the same key (and thus the key in the table is omitted), (2) all records belong to a single join window, and (3) all records are processed in timestamp order. The columns INNER JOIN, LEFT JOIN, and OUTER JOIN denote what is passed as arguments to the user-supplied [ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html) for the ```join```, ```leftJoin```, and ```outerJoin``` methods, respectively, whenever a new input record is received on either side of the join. An empty table cell denotes that the ```ValueJoiner``` is not called at all.

Timestamp | Left (KStream) | Right (KStream) | (INNER) JOIN |	LEFT JOIN | OUTER JOIN
--- | --- | --- | --- | --- | ---
1 | null
2	| | null	 	 	 
3	| A	| | | [A, null] | [A, null]
4	| | a | [A, a] | [A, a] | [A, a]
5 | B |	| [B, a] | [B, a] | [B, a]
6	| | b	| [A, b], [B, b] | [A, b], [B, b] | [A, b], [B, b]
7 | null	 	 	 	 
8	| | null	 	 	 
9	| C	| |	[C, a], [C, b] | [C, a], [C, b] |	[C, a], [C, b]
10 | | c | [A, c], [B, c], [C, c] | [A, c], [B, c], [C, c] | [A, c], [B, c], [C, c]
11 | | null	 	 	 
12 | null	 	 	 	 
13 | | null	 	 	 
14 | | d |[A, d], [B, d], [C, d] | [A, d], [B, d], [C, d] |	[A, d], [B, d], [C, d]
15 | D | | [D, a], [D, b], [D, c], [D, d] | [D, a], [D, b], [D, c], [D, d] | [D, a], [D, b], [D, c], [D, d]

### KTable-KTable Join

KTable-KTable joins are always non-windowed joins. They are designed to be consistent with their counterparts in relational databases. The changelog streams of both KTables are materialized into local state stores to represent the latest snapshot of their [table duals](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-ktable). The join result is a new KTable that represents the changelog stream of the join operation.

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

Transformation	Description
Inner Join

(KTable, KTable) → KTable
Performs an INNER JOIN of this table with another table. The result is an ever-updating KTable that represents the “current” result of the join. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

KTable<String, Long> left = ...;
KTable<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KTable<String, String> joined = left.join(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue /* ValueJoiner */
  );

// Java 7 example
KTable<String, String> joined = left.join(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    });
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Input records with a null key are ignored and do not trigger the join.
Input records with a null value are interpreted as tombstones for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join. When an input tombstone is received, then an output tombstone is forwarded directly to the join result KTable if required (i.e. only if the corresponding key actually exists already in the join result KTable).
See the semantics overview at the bottom of this section for a detailed description.

Left Join

(KTable, KTable) → KTable
Performs a LEFT JOIN of this table with another table. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

KTable<String, Long> left = ...;
KTable<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KTable<String, String> joined = left.leftJoin(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue /* ValueJoiner */
  );

// Java 7 example
KTable<String, String> joined = left.leftJoin(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    });
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Input records with a null key are ignored and do not trigger the join.
Input records with a null value are interpreted as tombstones for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join. When an input tombstone is received, then an output tombstone is forwarded directly to the join result KTable if required (i.e. only if the corresponding key actually exists already in the join result KTable).
For each input record on the left side that does not have any match on the right side, the ValueJoiner will be called with ValueJoiner#apply(leftRecord.value, null); this explains the row with timestamp=3 in the table below, which lists [A, null] in the LEFT JOIN column.

See the semantics overview at the bottom of this section for a detailed description.

Outer Join

(KTable, KTable) → KTable
Performs an OUTER JOIN of this table with another table. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

KTable<String, Long> left = ...;
KTable<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KTable<String, String> joined = left.outerJoin(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue /* ValueJoiner */
  );

// Java 7 example
KTable<String, String> joined = left.outerJoin(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    });
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Input records with a null key are ignored and do not trigger the join.
Input records with a null value are interpreted as tombstones for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join. When an input tombstone is received, then an output tombstone is forwarded directly to the join result KTable if required (i.e. only if the corresponding key actually exists already in the join result KTable).
For each input record on one side that does not have any match on the other side, the ValueJoiner will be called with ValueJoiner#apply(leftRecord.value, null) or ValueJoiner#apply(null, rightRecord.value), respectively; this explains the rows with timestamp=3 and timestamp=7 in the table below, which list [A, null] and [null, b], respectively, in the OUTER JOIN column.

See the semantics overview at the bottom of this section for a detailed description.

**Semantics of table-table joins:** The semantics of the various table-table join variants are explained below. To improve the readability of the table, you can assume that (1) all records have the same key (and thus the key in the table is omitted) and that (2) all records are processed in timestamp order. The columns INNER JOIN, LEFT JOIN, and OUTER JOIN denote what is passed as arguments to the user-supplied [ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html) for the ```join```, ```leftJoin```, and ```outerJoin``` methods, respectively, whenever a new input record is received on either side of the join. An empty table cell denotes that the ```ValueJoiner``` is not called at all.

Timestamp |Left (KTable) | Right (KTable) | (INNER) JOIN | LEFT JOIN | OUTER JOIN
--- | --- | --- | --- | --- | ---
1	| null	 	 	 	 
2	| | null	 	 	 
3	| A | | | [A, null]	| [A, null]
4	| | a	| [A, a] | [A, a] | [A, a]
5 | B | |	[B, a] | [B, a] | [B, a]
6	| | b	| [B, b] | [B, b] | [B, b]
7 | null | | null	| null | [null, b]
8	| | null | | | null
9	| C	| | | [C, null] |	[C, null]
10 | | c | [C, c] |	[C, c] | [C, c]
11 | | null	| null | [C, null] | [C, null]
12 | null | | |	null | null
13 | | null	 	 	
14 | | d | | | [null, d]
15 | D | | [D, d] |	[D, d] | [D, d]

### KStream-KTable Join

KStream-KTable joins are always non-windowed joins. They allow you to perform table lookups against a KTable (changelog stream) upon receiving a new record from the KStream (record stream). An example use case would be to enrich a stream of user activities (KStream) with the latest user profile information (KTable).

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

Transformation	Description
Inner Join

(KStream, KTable) → KStream
Performs an INNER JOIN of this stream with the table, effectively doing a table lookup. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.

Several variants of join exists, see the Javadocs for details.

KStream<String, Long> left = ...;
KTable<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.join(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue, /* ValueJoiner */
    Joined.keySerde(Serdes.String()) /* key */
      .withValueSerde(Serdes.Long()) /* left value */
  );

// Java 7 example
KStream<String, String> joined = left.join(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    },
    Joined.keySerde(Serdes.String()) /* key */
      .withValueSerde(Serdes.Long()) /* left value */
  );
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.
Input records for the stream with a null key or a null value are ignored and do not trigger the join.
Input records for the table with a null value are interpreted as tombstones for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join.
See the semantics overview at the bottom of this section for a detailed description.

Left Join

(KStream, KTable) → KStream
Performs a LEFT JOIN of this stream with the table, effectively doing a table lookup. (details)

Data must be co-partitioned: The input data for both sides must be co-partitioned.

Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.

Several variants of leftJoin exists, see the Javadocs for details.

KStream<String, Long> left = ...;
KTable<String, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.leftJoin(right,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue, /* ValueJoiner */
    Joined.keySerde(Serdes.String()) /* key */
      .withValueSerde(Serdes.Long()) /* left value */
  );

// Java 7 example
KStream<String, String> joined = left.leftJoin(right,
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    },
    Joined.keySerde(Serdes.String()) /* key */
      .withValueSerde(Serdes.Long()) /* left value */
  );
Detailed behavior:

The join is key-based, i.e. with the join predicate leftRecord.key == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.
Input records for the stream with a null key or a null value are ignored and do not trigger the join.
Input records for the table with a null value are interpreted as tombstones for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join.
For each input record on the left side that does not have any match on the right side, the ValueJoiner will be called with ValueJoiner#apply(leftRecord.value, null); this explains the row with timestamp=3 in the table below, which lists [A, null] in the LEFT JOIN column.

See the semantics overview at the bottom of this section for a detailed description.

**Semantics of stream-table joins:** The semantics of the various stream-table join variants are explained below. To improve the readability of the table we assume that (1) all records have the same key (and thus we omit the key in the table) and that (2) all records are processed in timestamp order. The columns INNER JOIN and LEFT JOIN denote what is passed as arguments to the user-supplied [ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html) for the ```join``` and ```leftJoin``` methods, respectively, whenever a new input record is received on either side of the join. An empty table cell denotes that the ```ValueJoiner``` is not called at all.

Timestamp | Left (KStream) | Right (KTable) | (INNER) JOIN | LEFT JOIN
--- | --- | --- | --- |---
1	| null	 	 	 
2	| | null	 	 
3	| A | | |	[A, null]
4	| | a	 	 
5 | B	| | [B, a] | [B, a]
6 | |	b	 	 
7	| null	 	 	 
8	| | null	 	 
9	| C	| | |	[C, null]
10 | | c	 	 
11 | | null	 	 
12 | null	 	 	 
13 | | null	 	 
14 | | d	 	 
15 | D | | [D, d] | [D, d]

## KStream-GlobalKTable Join

KStream-GlobalKTable joins are always non-windowed joins. They allow you to perform table lookups against a [GlobalKTable](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-globalktable) (entire changelog stream) upon receiving a new record from the KStream (record stream). An example use case would be “star queries” or “star joins”, where you would enrich a stream of user activities (KStream) with the latest user profile information (GlobalKTable) and further context information (further GlobalKTables).

At a high-level, KStream-GlobalKTable joins are very similar to [KStream-KTable joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-ktable). However, global tables provide you with much more flexibility at the [some expense](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-globalktable) when compared to partitioned tables:

* They do not require [data co-partitioning](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).
* They allow for efficient “star joins”; i.e., joining a large-scale “facts” stream against “dimension” tables
* They allow for joining against foreign keys; i.e., you can lookup data in the table not just by the keys of records in the stream, but also by data in the record values.
* They make many use cases feasible where you must work on heavily skewed data and thus suffer from hot partitions.
* They are often more efficient than their partitioned KTable counterpart when you need to perform multiple joins in succession.

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

Transformation	Description
Inner Join

(KStream, GlobalKTable) → KStream
Performs an INNER JOIN of this stream with the global table, effectively doing a table lookup. (details)

The GlobalKTable is fully bootstrapped upon (re)start of a KafkaStreams instance, which means the table is fully populated with all the data in the underlying topic that is available at the time of the startup. The actual data processing begins only once the bootstrapping has completed.

Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.

KStream<String, Long> left = ...;
GlobalKTable<Integer, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.join(right,
    (leftKey, leftValue) -> leftKey.length(), /* derive a (potentially) new key by which to lookup against the table */
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue /* ValueJoiner */
  );

// Java 7 example
KStream<String, String> joined = left.join(right,
    new KeyValueMapper<String, Long, Integer>() { /* derive a (potentially) new key by which to lookup against the table */
      @Override
      public Integer apply(String key, Long value) {
        return key.length();
      }
    },
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    });
Detailed behavior:

The join is indirectly key-based, i.e. with the join predicate KeyValueMapper#apply(leftRecord.key, leftRecord.value) == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.
Input records for the stream with a null key or a null value are ignored and do not trigger the join.
Input records for the table with a null value are interpreted as tombstones, which indicate the deletion of a record key from the table. Tombstones do not trigger the join.
Left Join

(KStream, GlobalKTable) → KStream
Performs a LEFT JOIN of this stream with the global table, effectively doing a table lookup. (details)

The GlobalKTable is fully bootstrapped upon (re)start of a KafkaStreams instance, which means the table is fully populated with all the data in the underlying topic that is available at the time of the startup. The actual data processing begins only once the bootstrapping has completed.

Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.

KStream<String, Long> left = ...;
GlobalKTable<Integer, Double> right = ...;

// Java 8+ example, using lambda expressions
KStream<String, String> joined = left.leftJoin(right,
    (leftKey, leftValue) -> leftKey.length(), /* derive a (potentially) new key by which to lookup against the table */
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue /* ValueJoiner */
  );

// Java 7 example
KStream<String, String> joined = left.leftJoin(right,
    new KeyValueMapper<String, Long, Integer>() { /* derive a (potentially) new key by which to lookup against the table */
      @Override
      public Integer apply(String key, Long value) {
        return key.length();
      }
    },
    new ValueJoiner<Long, Double, String>() {
      @Override
      public String apply(Long leftValue, Double rightValue) {
        return "left=" + leftValue + ", right=" + rightValue;
      }
    });
Detailed behavior:

The join is indirectly key-based, i.e. with the join predicate KeyValueMapper#apply(leftRecord.key, leftRecord.value) == rightRecord.key.

The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied ValueJoiner will be called to produce join output records.

Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.
Input records for the stream with a null key or a null value are ignored and do not trigger the join.
Input records for the table with a null value are interpreted as tombstones, which indicate the deletion of a record key from the table. Tombstones do not trigger the join.
For each input record on the left side that does not have any match on the right side, the ValueJoiner will be called with ValueJoiner#apply(leftRecord.value, null).

**Semantics of stream-table joins:** The join semantics are identical to [KStream-KTable joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-ktable). The only difference is that, for KStream-GlobalKTable joins, the left input record is first “mapped” with a user-supplied ```KeyValueMapper``` into the table’s keyspace prior to the table lookup.

## Windowing

Windowing lets you control how to group records that have the same key for stateful operations such as [aggregations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) or [joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins) into so-called windows. Windows are tracked per record key.

### Note

A related operation is [grouping](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless), which groups all records that have the same key to ensure that data is properly partitioned (“keyed”) for subsequent operations. Once grouped, windowing allows you to further sub-group the records of a key.

For example, in join operations, a windowing state store is used to store all the records received so far within the defined window boundary. In aggregating operations, a windowing state store is used to store the latest aggregation results per window. Old records in the state store are purged after the specified [window retention period](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-windowing). Kafka Streams guarantees to keep a window for at least this specified time; the default value is one day and can be changed via ```Windows#until()``` and ```SessionWindows#until()```.

The DSL supports the following types of windows:

Window name | Behavior | Short description
--- | --- | ---
[Tumbling time window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-tumbling) | 	Time-based | Fixed-size, non-overlapping, gap-less windows
[Hopping time window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-hopping) | 	Time-based | Fixed-size, overlapping windows
[Sliding time window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-sliding) | Time-based | Fixed-size, overlapping windows that work on differences between record timestamps
[Session window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-session) | 	Session-based	| Dynamically-sized, non-overlapping, data-driven windows

### Tumbling time windows

Tumbling time windows are a special case of hopping time windows and, like the latter, are windows based on time intervals. They model fixed-size, non-overlapping, gap-less windows. A tumbling window is defined by a single property: the window’s size. A tumbling window is a hopping window whose window size is equal to its advance interval. Since tumbling windows never overlap, a data record will belong to one and only one window.

![](../../../imgs/streams-time-windows-tumbling.png)

This diagram shows windowing a stream of data records with tumbling windows. Windows do not overlap because, by definition, the advance interval is identical to the window size. In this diagram the time numbers represent minutes; e.g. t=5 means “at the five-minute mark”. In reality, the unit of time in Kafka Streams is milliseconds, which means the time numbers would need to be multiplied with 60 * 1,000 to convert from minutes to milliseconds (e.g. t=5 would become t=300,000).

Tumbling time windows are aligned to the epoch, with the lower interval bound being inclusive and the upper bound being exclusive. “Aligned to the epoch” means that the first window starts at timestamp zero. For example, tumbling windows with a size of 5000ms have predictable window boundaries ```[0;5000),[5000;10000),...``` — and **not** ```[1000;6000),[6000;11000),...``` or even something “random” like ```[1452;6452),[6452;11452),...```.

The following code defines a tumbling window with a size of 5 minutes:

```java
import java.util.concurrent.TimeUnit;
import org.apache.kafka.streams.kstream.TimeWindows;

// A tumbling time window with a size of 5 minutes (and, by definition, an implicit
// advance interval of 5 minutes).
long windowSizeMs = TimeUnit.MINUTES.toMillis(5); // 5 * 60 * 1000L
TimeWindows.of(windowSizeMs);

// The above is equivalent to the following code:
TimeWindows.of(windowSizeMs).advanceBy(windowSizeMs);
```

### Hopping time windows

Hopping time windows are windows based on time intervals. They model fixed-sized, (possibly) overlapping windows. A hopping window is defined by two properties: the window’s size and its advance interval (aka “hop”). The advance interval specifies by how much a window moves forward relative to the previous one. For example, you can configure a hopping window with a size 5 minutes and an advance interval of 1 minute. Since hopping windows can overlap – and in general they do – a data record may belong to more than one such windows.

### Note

**Hopping windows vs. sliding windows:** Hopping windows are sometimes called “sliding windows” in other stream processing tools. Kafka Streams follows the terminology in academic literature, where the semantics of sliding windows are different to those of hopping windows.

The following code defines a hopping window with a size of 5 minutes and an advance interval of 1 minute:

```java
import java.util.concurrent.TimeUnit;
import org.apache.kafka.streams.kstream.TimeWindows;

// A hopping time window with a size of 5 minutes and an advance interval of 1 minute.
// The window's name -- the string parameter -- is used to e.g. name the backing state store.
long windowSizeMs = TimeUnit.MINUTES.toMillis(5); // 5 * 60 * 1000L
long advanceMs =    TimeUnit.MINUTES.toMillis(1); // 1 * 60 * 1000L
TimeWindows.of(windowSizeMs).advanceBy(advanceMs);
```

![](../../../imgs/streams-time-windows-hopping.png)

This diagram shows windowing a stream of data records with hopping windows. In this diagram the time numbers represent minutes; e.g. t=5 means “at the five-minute mark”. In reality, the unit of time in Kafka Streams is milliseconds, which means the time numbers would need to be multiplied with 60 * 1,000 to convert from minutes to milliseconds (e.g. t=5 would become t=300,000).

Hopping time windows are aligned to the epoch, with the lower interval bound being inclusive and the upper bound being exclusive. “Aligned to the epoch” means that the first window starts at timestamp zero. For example, hopping windows with a size of 5000ms and an advance interval (“hop”) of 3000ms have predictable window boundaries ```[0;5000),[3000;8000),...``` — and **not** ```[1000;6000),[4000;9000),...``` or even something “random” like ```[1452;6452),[4452;9452),...```.

Unlike non-windowed aggregates that we have seen previously, windowed aggregates return a windowed KTable whose keys type is ```Windowed<K>```. This is to differentiate aggregate values with the same key from different windows. The corresponding window instance and the embedded key can be retrieved as ```Windowed#window()``` and ```Windowed#key()```, respectively.

### Sliding time windows

Sliding windows are actually quite different from hopping and tumbling windows. In Kafka Streams, sliding windows are used only for [join operations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins), and can be specified through the ```JoinWindows``` class.

A sliding window models a fixed-size window that slides continuously over the time axis; here, two data records are said to be included in the same window if (in the case of symmetric windows) the difference of their timestamps is within the window size. Thus, sliding windows are not aligned to the epoch, but to the data record timestamps. In contrast to hopping and tumbling windows, the lower and upper window time interval bounds of sliding windows are both inclusive.

### Session Windows

### 会话窗口

Session windows are used to aggregate key-based events into so-called sessions, the process of which is referred to as sessionization. Sessions represent a **period of activity** separated by a defined **gap of inactivity** (or “idleness”). Any events processed that fall within the inactivity gap of any existing sessions are merged into the existing sessions. If an event falls outside of the session gap, then a new session will be created.

会话窗口用于将基于键的事件聚合到所谓的会话中，其过程称为会话。会话表示被一个确定的**不活动（或“闲置”）的间隔**分开的**一段时间的活动**。任何处于任何现有会话的不活动间隔内的事件都会合并到现有会话中。如果一个事件超出了会话间隔，那么将创建一个新的会话。

Session windows are different from the other window types in that:

会话窗口与其他类型的窗口的不同之处在于：

* all windows are tracked independently across keys – e.g. windows of different keys typically have different start and end times

* 所有窗口都是根据键来独立跟踪的——例如，不同键的窗口通常具有不同的开始和结束时间

* their window sizes sizes vary – even windows for the same key typically have different sizes

* 他们的窗口大小不同——即使是键相同的窗口一般都具有不同的大小

The prime area of application for session windows is **user behavior analysis**. Session-based analyses can range from simple metrics (e.g. count of user visits on a news website or social platform) to more complex metrics (e.g. customer conversion funnel and event flows).

会话窗口的主要应用领域是**用户行为分析**。基于会话的分析可以从简单的度量（例如新闻网站或社交平台上的用户访问次数）到更复杂的度量（例如，客户转换渠道和事件流程）。

The following code defines a session window with an inactivity gap of 5 minutes:

以下代码定义了一个不活动间隔为5分钟的会话窗口：

```java
import java.util.concurrent.TimeUnit;
import org.apache.kafka.streams.kstream.SessionWindows;

// A session window with an inactivity gap of 5 minutes.
SessionWindows.with(TimeUnit.MINUTES.toMillis(5));
```

Given the previous session window example, here’s what would happen on an input stream of six records. When the first three records arrive (upper part of in the diagram below), we’d have three sessions (see lower part) after having processed those records: two for the green record key, with one session starting and ending at the 0-minute mark (only due to the illustration it looks as if the session goes from 0 to 1), and another starting and ending at the 6-minute mark; and one session for the blue record key, starting and ending at the 2-minute mark.

参见之前的会话窗口示例，以下是在包含六条消息的输入流上会发生的情况。当前三条消息到达时（下图中的上半部分），我们在处理完这些消息后会有三个会话（见下半部分）：两个用于消息键为绿色的消息，一个会话开始和结束于标记为0分钟处（仅仅是因为插图看起来好像会话是从0到1分钟一样），另一个标记在6分钟开始和结束；和一个消息键为蓝色的消息的会话，开始和结束时间为2分钟。

Detected sessions after having received three input records: two records for the green record key at t=0 and t=6, and one record for the blue record key at t=2. In this diagram the time numbers represent minutes; e.g. t=5 means “at the five-minute mark”. In reality, the unit of time in Kafka Streams is milliseconds, which means the time numbers would need to be multiplied with 60 * 1,000 to convert from minutes to milliseconds (e.g. t=5 would become t=300,000).

已收到三条输入消息后检测到的会话：t = 0和t = 6时消息键为绿色的两条消息，以及t = 2处消息键为蓝色的一条消息。在此图中，表示时间的数字代表分钟；例如t = 5意味着“在五分钟时的标记”。实际上，Kafka Streams中的时间单位是毫秒，这意味着表示时间的数字需要乘以60 * 1,000以将分钟转换为毫秒（例如，t = 5将变为t = 300,000）。

If we then receive three additional records (including two late-arriving records), what would happen is that the two existing sessions for the green record key will be merged into a single session starting at time 0 and ending at time 6, consisting of a total of three records. The existing session for the blue record key will be extended to end at time 5, consisting of a total of two records. And, finally, there will be a new session for the blue key starting and ending at time 11.

如果我们接收到三条附加消息（包括两条迟到消息），会发生的情况是消息键为绿色的两个现有会话将被合并到从时间0开始到时间6结束的单个共包括三条消息的会话中。消息键为蓝色的共由两条消息组成的现有会话将被延长到时间5结束。最后，消息键为蓝色的新会话在11时开始和结束。

Detected sessions after having received six input records. Note the two late-arriving data records at t=4 (green) and t=5 (blue), which lead to a merge of sessions and an extension of a session, respectively.

在收到六条输入消息后检测到的会话。注意t = 4（绿色）和t = 5（蓝色）两个迟到的消息数据，它们分别导致会话合并和会话延长。

## Applying processors and transformers (Processor API integration)

## 应用处理器和变换器（Processor API集成）

Beyond the aforementioned [stateless](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless) and [stateful](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless) transformations, you may also leverage the [Processor API](http://kafka.apache.org/11/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api) from the DSL. There are a number of scenarios where this may be helpful:

除了上述[无状态的](dsl-api.md)和[有状态](dsl-api.md)的转换之外，您还可以利用DSL中的[Processor API](processor-api.md)。下面几种情况可能会有所帮助：

* **Customization:** You need to implement special, customized logic that is not or not yet available in the DSL.

* **自定义：**您需要实现DSL中没有的或尚未提供的特殊自定义逻辑。

* **Combining ease-of-use with full flexibility where it’s needed:** Even though you generally prefer to use the expressiveness of the DSL, there are certain steps in your processing that require more flexibility and tinkering than the DSL provides. For example, only the Processor API provides access to a record’s metadata such as its topic, partition, and offset information. However, you don’t want to switch completely to the Processor API just because of that.

* **在需要时兼顾易用性(ease-of-use)和完全灵活性(full flexibility)**：尽管您通常更喜欢使用DSL的表达能力，但您的处理过程中有一些步骤需要提供比DSL更多的灵活性和纠错性。例如，只有Processor API可以访问消息的元数据，如主题，分区和偏移量信息。然而也正因如此，您不会希望完全切换到Processor API。

* **Migrating from other tools:** You are migrating from other stream processing technologies that provide an imperative API, and migrating some of your legacy code to the Processor API was faster and/or easier than to migrate completely to the DSL right away.

* **从其他工具迁移过来：**您正在从那些提供强制性API的其他流处理技术中迁移过来，并且将一些旧代码迁移到Processor API比立即完全迁移到DSL更快或更容易。

Transformation	Description
Process

KStream -> void
Terminal operation. Applies a Processor to each record. process() allows you to leverage the Processor API from the DSL. (details)

This is essentially equivalent to adding the Processor via Topology#addProcessor() to your processor topology.

An example is available in the javadocs.

Transform

KStream -> KStream
Applies a Transformer to each record. transform() allows you to leverage the Processor API from the DSL. (details)

Each input record is transformed into zero, one, or more output records (similar to the stateless flatMap). The Transformer must return null for zero output. You can modify the record’s key and value, including their types.

Marks the stream for data re-partitioning: Applying a grouping or a join after transform will result in re-partitioning of the records. If possible use transformValues instead, which will not cause data re-partitioning.

transform is essentially equivalent to adding the Transformer via Topology#addProcessor() to your processor topology.

An example is available in the javadocs.

Transform (values only)

KStream -> KStream
Applies a ValueTransformer to each record, while retaining the key of the original record. transformValues() allows you to leverage the Processor API from the DSL. (details)

Each input record is transformed into exactly one output record (zero output records or multiple output records are not possible). The ValueTransformer may return null as the new value for a record.

transformValues is preferable to transform because it will not cause data re-partitioning.

transformValues is essentially equivalent to adding the ValueTransformer via Topology#addProcessor() to your processor topology.

An example is available in the javadocs.

The following example shows how to leverage, via the ```KStream#process()``` method, a custom ```Processor``` that sends an email notification whenever a page view count reaches a predefined threshold.

以下示例说明如何通过```KStream#process()```方法来使用自定义处理器，该```Processor```在页面查看次数达到预定义阈值时发送电子邮件通知。

First, we need to implement a custom stream processor, ```PopularPageEmailAlert```, that implements the ```Processor``` interface:

首先，我们需要实现一个实现了```Processor```接口的自定义流处理器```PopularPageEmailAlert```：

```java
// A processor that sends an alert message about a popular page to a configurable email address
public class PopularPageEmailAlert implements Processor<PageId, Long> {

  private final String emailAddress;
  private ProcessorContext context;

  public PopularPageEmailAlert(String emailAddress) {
    this.emailAddress = emailAddress;
  }

  @Override
  public void init(ProcessorContext context) {
    this.context = context;

    // Here you would perform any additional initializations such as setting up an email client.
  }

  @Override
  void process(PageId pageId, Long count) {
    // Here you would format and send the alert email.
    //
    // In this specific example, you would be able to include information about the page's ID and its view count
    // (because the class implements `Processor<PageId, Long>`).
  }

  @Override
  void close() {
    // Any code for clean up would go here.  This processor instance will not be used again after this call.
  }

}
```

### Tip

### 提示

Even though we do not demonstrate it in this example, a stream processor can access any available state stores by calling ```ProcessorContext#getStateStore()```. Only such state stores are available that (1) have been named in the corresponding ```KStream#process()``` method call (note that this is a different method than ```Processor#process()```), plus (2) all global stores. Note that global stores do not need to be attached explicitly; however, they only allow for read-only access.

尽管我们在本例中没有演示流处理器，但它可以通过调用```ProcessorContext＃getStateStore()```来访问任何可用的状态存储器。 只有这样的状态存储器可用：（1）已在相应的```KStream#process()```方法调用中被指定（请注意，这是与```Processor#process()```不同的方法），（2）所有全局存储器。请注意，全局存储器不需要明确被附加; 但是，它们只允许只读访问。

Then we can leverage the ```PopularPageEmailAlert``` processor in the DSL via ```KStream#process```.

然后，我们可以通过```KStream#```进程来使用DSL中的```PopularPageEmailAlert```处理器。

In Java 8+, using lambda expressions:

在Java 8+中，使用lambda表达式：

KStream<String, GenericRecord> pageViews = ...;

// Send an email notification when the view count of a page reaches one thousand.
pageViews.groupByKey()
         .count()
         .filter((PageId pageId, Long viewCount) -> viewCount == 1000)
         // PopularPageEmailAlert is your custom processor that implements the
         // `Processor` interface, see further down below.
         .process(() -> new PopularPageEmailAlert("alerts@yourcompany.com"));

In Java 7:

在Java 7中：

// Send an email notification when the view count of a page reaches one thousand.
pageViews.groupByKey().
         .count()
         .filter(
            new Predicate<PageId, Long>() {
              public boolean test(PageId pageId, Long viewCount) {
                return viewCount == 1000;
              }
            })
         .process(
           new ProcessorSupplier<PageId, Long>() {
             public Processor<PageId, Long> get() {
               // PopularPageEmailAlert is your custom processor that implements
               // the `Processor` interface, see further down below.
               return new PopularPageEmailAlert("alerts@yourcompany.com");
             }
           });

## WRITING STREAMS BACK TO KAFKA

## 把流写回到Kafka

Any streams and tables may be (continuously) written back to a Kafka topic. As we will describe in more detail below, the output data might be re-partitioned on its way to Kafka, depending on the situation.

任何流和表都可以（连续地）写回Kafka主题。正如我们下面将要详细描述的那样，根据具体情况，输出数据可能会在前往Kafka的途中重新分区。

Writing to Kafka	Description
To

KStream -> void
Terminal operation. Write the records to a Kafka topic. (KStream details)

When to provide serdes explicitly:

If you do not specify SerDes explicitly, the default SerDes from the configuration are used.
You must specify SerDes explicitly via the Produced class if the key and/or value types of the KStream do not match the configured default SerDes.
See Data Types and Serialization for information about configuring default SerDes, available SerDes, and implementing your own custom SerDes.
A variant of to exists that enables you to specify how the data is produced by using a Produced instance to specify, for example, a StreamPartitioner that gives you control over how output records are distributed across the partitions of the output topic.

KStream<String, Long> stream = ...;
KTable<String, Long> table = ...;


// Write the stream to the output topic, using the configured default key
// and value serdes of your `StreamsConfig`.
stream.to("my-stream-output-topic");

// Same for table
table.to("my-table-output-topic");

// Write the stream to the output topic, using explicit key and value serdes,
// (thus overriding the defaults of your `StreamsConfig`).
stream.to("my-stream-output-topic", Produced.with(Serdes.String(), Serdes.Long());
Causes data re-partitioning if any of the following conditions is true:

If the output topic has a different number of partitions than the stream/table.
If the KStream was marked for re-partitioning.
If you provide a custom StreamPartitioner to explicitly control how to distribute the output records across the partitions of the output topic.
If the key of an output record is null.
Through

KStream -> KStream
KTable -> KTable
Write the records to a Kafka topic and create a new stream/table from that topic. Essentially a shorthand for KStream#to() followed by StreamsBuilder#stream(), same for tables. (KStream details)

When to provide SerDes explicitly:

If you do not specify SerDes explicitly, the default SerDes from the configuration are used.
You must specify SerDes explicitly if the key and/or value types of the KStream or KTable do not match the configured default SerDes.
See Data Types and Serialization for information about configuring default SerDes, available SerDes, and implementing your own custom SerDes.
A variant of through exists that enables you to specify how the data is produced by using a Produced instance to specify, for example, a StreamPartitioner that gives you control over how output records are distributed across the partitions of the output topic.

StreamsBuilder builder = ...;
KStream<String, Long> stream = ...;
KTable<String, Long> table = ...;

// Variant 1: Imagine that your application needs to continue reading and processing
// the records after they have been written to a topic via ``to()``.  Here, one option
// is to write to an output topic, then read from the same topic by constructing a
// new stream from it, and then begin processing it (here: via `map`, for example).
stream.to("my-stream-output-topic");
KStream<String, Long> newStream = builder.stream("my-stream-output-topic").map(...);

// Variant 2 (better): Since the above is a common pattern, the DSL provides the
// convenience method ``through`` that is equivalent to the code above.
// Note that you may need to specify key and value serdes explicitly, which is
// not shown in this simple example.
KStream<String, Long> newStream = stream.through("user-clicks-topic").map(...);

// ``through`` is also available for tables
KTable<String, Long> newTable = table.through("my-table-output-topic").map(...);
Causes data re-partitioning if any of the following conditions is true:

If the output topic has a different number of partitions than the stream/table.
If the KStream was marked for re-partitioning.
If you provide a custom StreamPartitioner to explicitly control how to distribute the output records across the partitions of the output topic.
If the key of an output record is null.

### Note

### 注意

**When you want to write to systems other than Kafka:** Besides writing the data back to Kafka, you can also apply a [custom processor](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-process) as a stream sink at the end of the processing to, for example, write to external databases. First, doing so is not a recommended pattern – we strongly suggest to use the [Kafka Connect API](http://kafka.apache.org/11/documentation/connect/index.html#kafka-connect) instead. However, if you do use such a sink processor, please be aware that it is now your responsibility to guarantee message delivery semantics when talking to such external systems (e.g., to retry on delivery failure or to prevent message duplication).

**当您要写入除Kafka以外的系统时：**除了将数据写回Kafka之外，还可以在处理结束时将[定制处理器](dsl-api.md)作为流接收器应用于如外部数据库等。首先，这样做不是推荐模式——我们强烈建议使用[Kafka Connect API](../../kafka_connect.md)。但是，如果您确实使用了这样的接收处理器，请注意，现在与此类外部系统交互时您将有责任去保证消息传递语义（例如，重试交付失败或防止消息重复）。