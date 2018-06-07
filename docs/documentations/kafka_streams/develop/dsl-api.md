# Streams DSL

The Kafka Streams DSL (Domain Specific Language) is built on top of the Streams Processor API. It is the recommended for most users, especially beginners. Most data processing operations can be expressed in just a few lines of DSL code.

Kafka Streams DSL（Domain Specific Language）构建在Streams Processor API之上。这是推荐大多数用户使用的，特别是初学者。大部分数据处理操作都可以用几行DSL代码表示。

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

        * [连接](dsl-api.md)

            * [Join co-partitioning requirements](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#join-co-partitioning-requirements)

            * [KStream-KStream Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#kstream-kstream-join)

            * [KTable-KTable Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#ktable-ktable-join)

            * [KStream-KTable Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#kstream-ktable-join)

            * [KStream-GlobalKTable Join](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#kstream-globalktable-join)


            * [连接共分区(co-partitioning)要求](dsl-api.md)

            * [KStream-KStream连接](dsl-api.md)

            * [KTable-KTable连接](dsl-api.md)

            * [KStream-KTable连接](dsl-api.md)

            * [KStream-GlobalKTable连接](dsl-api.md)

        * [Windowing](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#windowing)

        * [窗口](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#windowing)
            * [Tumbling time windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#tumbling-time-windows)

            * [Hopping time windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#hopping-time-windows)

            * [Sliding time windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#sliding-time-windows)

            * [Session Windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#session-windows)


            * [翻转时间窗口](dsl-api.md)

            * [跳跃时间窗口](dsl-api.md)

            * [滑动时间窗口](dsl-api.md)

            * [会话窗口](dsl-api.md)

    * [Applying processors and transformers (Processor API integration)](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#applying-processors-and-transformers-processor-api-integration)

    * [使用处理器和转换器(处理器API集成)](dsl-api.md)

* [Writing streams back to Kafka](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#writing-streams-back-to-kafka)

* [将流(streams)写回到Kafka](dsl-api.md)

## OVERVIEW

## 概述

In comparison to the [Processor API](http://kafka.apache.org/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api), only the DSL supports:

与[处理器API](processor-api.md)相比，只有DSL才支持的情形是：

* Built-in abstractions for [streams and tables](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-duality) in the form of [KStream](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-kstream), [KTable](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-ktable), and [GlobalKTable](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-globalktable). Having first-class support for streams and tables is crucial because, in practice, most use cases require not just either streams or databases/tables, but a combination of both. For example, if your use case is to create a customer 360-degree view that is updated in real-time, what your application will be doing is transforming many input streams of customer-related events into an output table that contains a continuously updated 360-degree view of your customers.

* 内置抽象的[KStream](../concepts.md)，[KTable](../concepts.md)和[GlobalKTable](../concepts.md)形式的[流和表(streams and tables)](../concepts.md)。对流和表(streams and tables)提供一流的支持是非常重要的，因为在实践中，大多数用例不仅需要流或数据库/表，而且还需要两者的组合。例如，如果您的用例是创建实时更新的360度的客户视图，那么您的应用程序将要做的是将许多与客户相关的事件输入流转换为包含不断更新的360度的客户视图的输出表。

* Declarative, functional programming style with [stateless transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless) (e.g. ```map``` and ```filter```) as well as [stateful transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateful) such as [aggregations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) (e.g. ```count``` and ```reduce```), [joins](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins) (e.g. ```leftJoin```), and [windowing](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing) (e.g. [session windows](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#windowing-session)).

* 具有[无状态转换](dsl-api.md)（例如```map```和 ```filter```）或诸如[聚合](dsl-api.md)（例如```count```和 ```reduce```）、[连接](dsl-api.md)（例如```leftJoin```）和[窗口](dsl-api.md)（例如[会话窗口](dsl-api.md)（session windows））这些特征的[有状态转换](dsl-api.md)的声明性和函数式编程风格。

With the DSL, you can define [processor topologies](http://kafka.apache.org/documentation/streams/concepts.html#streams-concepts-processor-topology) (i.e., the logical processing plan) in your application. The steps to accomplish this are:

您可以在应用程序中通过DSL来定义[处理器拓扑](../concepts.md)（即逻辑处理计划）。完成此定义的步骤是：

1. Specify [one or more input streams that are read from Kafka topics](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-sources).

2. Compose [transformations](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations) on these streams.

3. Write the [resulting output streams back to Kafka topics](http://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-destinations), or expose the processing results of your application directly to other applications through [interactive queries](http://kafka.apache.org/documentation/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries) (e.g., via a REST API).


1. 指定[一个或多个从Kafka主题中读取的输入流](dsl-api.md)。

2. 组合这些流的[转换](dsl-api.md)。

3. 将[结果输出流写回Kafka主题](dsl-api.md)，或通过[交互式查询](interactive-queries.md)（例如，通过REST API）将应用程序的处理结果直接公开给其他应用程序。

After the application is run, the defined processor topologies are continuously executed (i.e., the processing plan is put into action). A step-by-step guide for writing a stream processing application using the DSL is provided below.

应用程序运行后，定义的处理器拓扑将不断执行（也就是将处理计划付诸实施）。下面提供了使用DSL编写流处理应用程序的分步指南。

For a complete list of available API functionality, see also the [Streams](http://kafka.apache.org/javadoc/org/apache/kafka/streams/package-summary.html) API docs.

有关可用API功能的完整列表，另请参阅[Streams](http://kafka.apache.org/javadoc/org/apache/kafka/streams/package-summary.html) API文档。

## CREATING SOURCE STREAMS FROM KAFKA

## 创建来自Kafka的source流

You can easily read data from Kafka topics into your application. The following operations are supported.

您可以轻松地将来自Kafka主题的数据读入您的应用程序。以下操作均能得到支持。

* **Reading from Kafka**

* **从Kafka中读取数据**

    * **Stream**

        * *input topics* → KStream

            * **Description**

            * **描述**

                Creates a [KStream](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-kstream) from the specified Kafka input topics and interprets the data as a [record stream](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-kstream). A `KStream` represents a partitioned record stream. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsBuilder.html#stream(java.lang.String))

                根据指定的Kafka输入主题创建`KStream`，并将数据解释为`消息流`。 `KStream`表示分区的消息流。[（细节）](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsBuilder.html#stream(java.lang.String))
                
                In the case of a KStream, the local KStream instance of every application instance will be populated with data from only **a subset** of the partitions of the input topic. Collectively, across all application instances, all input topic partitions are read and processed.

                在KStream的情况下，每个应用程序实例的本地KStream实例将仅填充来自输入主题分区的`子集`的数据。总而言之，在所有应用程序实例中，都会读取和处理所有的输入主题分区。

                ```java
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
                ```

                If you do not specify SerDes explicitly, the default SerDes from the [configuration](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-configuration) are used.

                如果您没有明确指定SerDes，则使用[配置](config-streams.md)中的默认SerDes。

                You **must specify SerDes explicitly** if the key or value types of the records in the Kafka input topics do not match the configured default SerDes. For information about configuring default SerDes, available SerDes, and implementing your own custom SerDes see [Data Types and Serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes).

                如果Kafka输入主题中消息的键或值的类型与配置中的默认SerDes不匹配，则**必须明确指定SerDes**。有关配置默认SerDes，可用SerDes和实现自己的自定义SerDes的信息，请参阅[数据类型和序列化](datatypes.md)。

                Several variants of `stream` exist, for example to specify a regex pattern for input topics to read from).

                存在多种`流`的变体，例如为输入主题指定一种正则表达式模式以供读取）。
    
    * **Table**

        * *input topic* → KTable

            * **Description**

            * **描述**

                Reads the specified Kafka input topic into a [KTable](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-ktable). The topic is interpreted as a changelog stream, where records with the same key are interpreted as UPSERT aka INSERT/UPDATE (when the record value is not `null`) or as DELETE (when the value is `null`) for that key. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsBuilder.html#table-java.lang.String(java.lang.String))

                将指定的Kafka输入主题读入`KTable`。该主题被解释为更新日志流，其中具有相同键的消息被解释为UPSERT即INSERT/UPDATE（当消息值不为`null`时）或DELETE（当消息值为`null`时）该键。[（细节）](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsBuilder.html#table-java.lang.String(java.lang.String))

                In the case of a KStream, the local KStream instance of every application instance will be populated with data from only **a subset** of the partitions of the input topic. Collectively, across all application instances, all input topic partitions are read and processed.

                在KStream的情况下，每个应用程序实例的本地KStream实例将仅填充来自输入主题分区的**子集**的数据。总而言之，会在所有应用程序实例中，读取和处理所有输入主题分区。

                You must provide a name for the table (more precisely, for the internal [state store](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-state) that backs the table). This is required for supporting [interactive queries](http://kafka.apache.org/11/documentation/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries) against the table. When a name is not provided the table will not queryable and an internal name will be provided for the state store.

                您必须为该表提供一个名称（更确切地说，是为支持该表的内部[状态存储器](../architecture.md)提供）。这对于支持表的[交互式查询](interactive-queries.md)是必需的。如果没有提供名称，表将不可查询，那么会为表提供一个内部名称以便进行状态存储。

                If you do not specify SerDes explicitly, the default SerDes from the [configuration](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-configuration) are used.

                如果您没有明确指定SerDes，则使用[配置](config-streams.md)中的默认SerDes。

                You **must specify SerDes explicitly** if the key or value types of the records in the Kafka input topics do not match the configured default SerDes. For information about configuring default SerDes, available SerDes, and implementing your own custom SerDes see [Data Types and Serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes).

                如果Kafka输入主题中消息的键或值的类型与配置的默认SerDes不匹配，则**必须明确指定SerDes**。有关配置默认的SerDes，可用的SerDes和实现自己的自定义SerDes的信息，请参阅[数据类型和序列化](datatypes.md)。

                Several variants of `table` exist, for example to specify the `auto.offset.reset` policy to be used when reading from the input topic. 

                存在多种`表`的变体，例如，指定从输入主题读取时使用的`auto.offset.reset`策略。
    
    * **Global Table**

        * *input topic* → GlobalKTable

            * **Description**

                Reads the specified Kafka input topic into a [GlobalKTable](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-globalktable). The topic is interpreted as a changelog stream, where records with the same key are interpreted as UPSERT aka INSERT/UPDATE (when the record value is not `null`) or as DELETE (when the value is `null`) for that key. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsBuilder.html#globalTable-java.lang.String(java.lang.String))

                将指定的Kafka输入主题读入`GlobalKTable`。该主题被解释为更新日志流，其中具有相同键的消息被解释为UPSERT又名INSERT/UPDATE（当消息值不为`null`时）或DELETE（当消息值为`null`时）该键。[（细节）](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/StreamsBuilder.html#globalTable-java.lang.String(java.lang.String))

                In the case of a GlobalKTable, the local GlobalKTable instance of every application instance will be populated with data from **all** the partitions of the input topic.

                对于GlobalKTable，每个应用程序实例的本地GlobalKTable实例将填充来自输入主题的**所有**分区的数据。

                You must provide a name for the table (more precisely, for the internal [state store](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-state) that backs the table). This is required for supporting [interactive queries](http://kafka.apache.org/11/documentation/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries) against the table. When a name is not provided the table will not queryable and an internal name will be provided for the state store.

                您必须为该表提供一个名称（更确切地说，是为支持该表的内部[状态存储器](../architecture.md)提供）。这对于支持表的[交互式查询](interactive-queries.md)是必需的。如果没有提供名称，表将不可查询，并且将为状态存储器提供内部名称。

                ```java
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
                ```

                You **must specify SerDes explicitly** if the key or value types of the records in the Kafka input topics do not match the configured default SerDes. For information about configuring default SerDes, available SerDes, and implementing your own custom SerDes see [Data Types and Serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes).

                如果Kafka输入主题中消息的键或值的类型与配置的默认SerDes不匹配，则**必须明确指定SerDes**。有关配置默认的SerDes，可用的SerDes和实现自己的自定义SerDes的信息，请参阅[数据类型和序列化](datatypes.md)。

                Several variants of `globalTable` exist to e.g. specify explicit SerDes.

                `globalTable`的几种变体存在于例如指定的显式SerDes中。


## TRANSFORM A STREAM

## 转换流

The KStream and KTable interfaces support a variety of transformation operations. Each of these operations can be translated into one or more connected processors into the underlying processor topology. Since KStream and KTable are strongly typed, all of these transformation operations are defined as generic functions where users could specify the input and output data types.

KStream和KTable接口支持各种转换操作。这些操作中的每一个都可以被转换为一个或多个连接的处理器，并转换为底层处理器拓扑。由于KStream和KTable是强类型的，所有这些转换操作都被定义为通用函数，用户可以指定输入和输出数据类型。

Some KStream transformations may generate one or more KStream objects, for example: - ```filter``` and ```map``` on a KStream will generate another KStream - ```branch``` on KStream can generate multiple KStreams

一些KStream转换可能会生成一个或多个KStream对象，例如：```filter```和```map```KStream会生成另一个KStream，```branch```KStream可以生成多个KStream

Some others may generate a KTable object, for example an aggregation of a KStream also yields a KTable. This allows Kafka Streams to continuously update the computed value upon arrivals of [late records](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-aggregations) after it has already been produced to the downstream transformation operators.

其他的可能会生成KTable对象，例如KStream的聚合也会生成KTable。当下游转换操作符生产好[迟到消息(late records)](../concepts.md)后，Kafka Streams能对这些到达的迟到消息不断地更新计算值。

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

* **Transformation**

    * **Branch**

        * KStream → KStream[]

            * **Description**   

                Branch (or split) a `KStream` based on the supplied predicates into one or more `KStream` instances. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#branch-org.apache.kafka.streams.kstream.Predicate...-))

                将基于提供的谓词的`KStream`分支（或拆分）为一个或多个`KStream`实例。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#branch-org.apache.kafka.streams.kstream.Predicate...-)）

                Predicates are evaluated in order. A record is placed to one and only one output stream on the first match: if the n-th predicate evaluates to true, the record is placed to n-th stream. If no predicate matches, the the record is dropped.

                这些谓词将按顺序进行评估。消息被放置到第一个匹配上的一个且仅此一个输出流：如果第n个谓词的计算结果为true，则消息将被放置到第n个流中。如果谓词没有匹配命中，则消息被删除。

                Branching is useful, for example, to route records to different downstream topics.

                分支操作可有效地将消息路由到不同的下游主题。

                ```java
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
                ```

    * **Filter**

        * KStream → KStream

        * KTable → KTable

            * **Description**

                Evaluates a boolean function for each element and retains those for which the function returns true. ([KStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#filter-org.apache.kafka.streams.kstream.Predicate-), [KTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#filter-org.apache.kafka.streams.kstream.Predicate-))

                为每个元素计算布尔函数并保留那些返回true的函数。（[KStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#filter-org.apache.kafka.streams.kstream.Predicate-)，[KTable细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#filter-org.apache.kafka.streams.kstream.Predicate-)）

                ```java
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
                ```

    * **Inverse Filter**

        * KStream → KStream

        * KTable → KTable

            * **Description**

                Evaluates a boolean function for each element and drops those for which the function returns true. ([KStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#filter-org.apache.kafka.streams.kstream.Predicate-), [KTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#filter-org.apache.kafka.streams.kstream.Predicate-))

                为每个元素计算布尔函数并删除那些返回true的函数。（[KStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#filter-org.apache.kafka.streams.kstream.Predicate-)，[KTable细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#filter-org.apache.kafka.streams.kstream.Predicate-)）

                ```java
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
                ```
                
    * **FlatMap**

        * KStream → KStream

            * **Description**

                Takes one record and produces zero, one, or more records. You can modify the record keys and values, including their types. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#flatMapValues-org.apache.kafka.streams.kstream.ValueMapper-))

                获取一条消息并生成零个，一个或多个消息。您可以修改消息的键和值，包括它们的类型。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#flatMapValues-org.apache.kafka.streams.kstream.ValueMapper-)）

                **Marks the stream for data re-partitioning:** Applying a grouping or a join after `flatMap` will result in re-partitioning of the records. If possible use `flatMapValues` instead, which will not cause data re-partitioning.

                **标记数据重分区的流：**在`flatMap`之后应用分组或连接操作将导致消息的重分区。如果可能，请使用`flatMapValues`，这不会导致数据重分区。

                ```java
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
                ```
    * **FlatMap (values only)**

        * KStream → KStream

            * **Description**

                Takes one record and produces zero, one, or more records, while retaining the key of the original record. You can modify the record values and the value type. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#flatMapValues-org.apache.kafka.streams.kstream.ValueMapper-))

                获取一条消息并生成零个，一个或多个消息，同时保留原始消息的关键字。您可以修改消息的值和值的类型。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#flatMapValues-org.apache.kafka.streams.kstream.ValueMapper-)）

                `flatMapValues` is preferable to `flatMap` because it will not cause data re-partitioning. However, you cannot modify the key or key type like `flatMap` does.

                `flatMapValues`优于`flatMap`，因为它不会导致数据重分区。但是，您不能像`flatMap`那样修改键或键的类型。

                ```java
                // Split a sentence into words.
                KStream<byte[], String> sentences = ...;
                KStream<byte[], String> words = sentences.flatMapValues(value -> Arrays.asList(value.split("\\s+")));

                // Java 7 example: cf. `mapValues` for how to create `ValueMapper` instances
                ``` 

    * **Foreach**

        * KStream → void

        * KStream → void

        * KTable → void

            * **Description**              

                **Terminal operation.** Performs a stateless action on each record. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#foreach-org.apache.kafka.streams.kstream.ForeachAction-))

                **终端操作。**对每条消息执行无状态操作。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#foreach-org.apache.kafka.streams.kstream.ForeachAction-)）

                You would use `foreach` to cause side effects based on the input data (similar to `peek`) and then stop further processing of the input data (unlike `peek`, which is not a terminal operation).

                您可以使用`foreach`根据输入数据产生副作用（类似于`peek`），然后停止对输入数据的进一步处理（与`peek`不同，它不是终端操作）。

                **Note on processing guarantees:** Any side effects of an action (such as writing to external systems) are not trackable by Kafka, which means they will typically not benefit from Kafka’s processing guarantees.

                **关于处理保障的注意事项：**Kafka无法追踪任何行为的副作用（如写入外部系统），这意味着他们通常不会从Kafka的处理中得到保障。

                ```java
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
                ```

    * **GroupByKey**

        * KStream → KGroupedStream

            * **Description**             

                Groups the records by the existing key. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#groupByKey--))

                按现有键对消息进行分组。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#groupByKey--)）

                Grouping is a prerequisite for [aggregating a stream or a table](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) and ensures that data is properly partitioned (“keyed”) for subsequent operations.

                分组是[聚合一个流或表](dsl-api.md)的先决条件，并确保数据正确分区（“keyed”）以供后续操作使用。

                **When to set explicit SerDes:** Variants of `groupByKey` exist to override the configured default SerDes of your application, which **you must do** if the key and/or value types of the resulting `KGroupedStream` do not match the configured default SerDes.

                **何时设置显式SerDes：** 存在用于覆盖应用程序中已配置的默认SerDes的`groupByKey`的变体，如果生成的`KGroupedStream`的键和、或值类型与配置的默认SerDes不匹配，则必须执行此操作。

                **Note**

                **注意**

                **Grouping vs. Windowing:** A related operation is [windowing](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing), which lets you control how to “sub-group” the grouped records of the same key into so-called windows for stateful operations such as windowed [aggregations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) or windowed [joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins).

                **分组与窗口化：** 相关的操作是[窗口化](dsl-api.md)，它允许您来控制如何将同一个键的分组消息“分组”到用于有状态操作（例如窗口化[聚合操作](dsl-api.md)或窗口化[连接操作](dsl-api.md)）的所谓窗口中。

                **Causes data re-partitioning if and only if the stream was marked for re-partitioning.** `groupByKey` is preferable to `groupBy` because it re-partitions data only if the stream was already marked for re-partitioning. However, `groupByKey` does not allow you to modify the key or key type like `groupBy` does.

                **当且仅当流被标记为重分区时，会导致数据重分区。**`groupByKey`优于`groupBy`，因为它仅在流已标记为重分区时重分区数据。然而，`groupByKey`不允许您像`groupBy`那样修改键或键的类型。

                ```java
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
                ```
    
    * **GroupBy**

        * KStream → KGroupedStream

        * KTable → KGroupedTable

            * **Description**  

                Groups the records by a new key, which may be of a different key type. When grouping a table, you may also specify a new value and value type. `groupBy` is a shorthand for `selectKey(...).groupByKey()`. ([KStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-), [KTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-))

                通过一个新的键对消息进行分组，可能是根据不同的键的类型。分组表时，您也可以指定一个新的值和值类型。`groupBy`是`selectKey(...).groupByKey()`的简写。 （[KStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-)，[KTable细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-)）

                Grouping is a prerequisite for [aggregating a stream or a table](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) and ensures that data is properly partitioned (“keyed”) for subsequent operations.

                分组是[聚合一个流或表](dsl-api.md)的先决条件，并确保数据正确分区（“keyed”）以供后续操作使用。

                **When to set explicit SerDes:** Variants of `groupBy` exist to override the configured default SerDes of your application, which **you must do** if the key and/or value types of the resulting `KGroupedStream` or `KGroupedTable` do not match the configured default SerDes.

                **何时设置显式SerDes：** 存在用于覆盖应用程序中已配置的默认SerDes的`groupBy`的变体，如果生成的`KGroupedStream`或`KGroupedTable`的键和、或值类型与配置的默认SerDes不匹配，则必须执行此操作。

                **Note**

                **Grouping vs. Windowing:** A related operation is [windowing](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing), which lets you control how to “sub-group” the grouped records of the same key into so-called windows for stateful operations such as windowed [aggregations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) or windowed [joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins).

                **分组与窗口化：** 相关的操作是[窗口化](dsl-api.md)，它允许您来控制如何将同一个键的分组消息“分组”到用于有状态操作（例如窗口化[聚合操作](dsl-api.md)或窗口化[连接操作](dsl-api.md)）的所谓窗口中。

                **Always causes data re-partitioning:** `groupBy` always causes data re-partitioning. If possible use `groupByKey` instead, which will re-partition data only if required.

                **总是导致数据重分区：** `groupBy`总是导致数据重分区。如果可能，请使用`groupByKey`，只有在需要时才会重分区数据。

                ```java
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
                ```

    * **Map**

        * KStream → KStream

            * **Description**  

                Takes one record and produces one record. You can modify the record key and value, including their types. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#map-org.apache.kafka.streams.kstream.KeyValueMapper-))

                获取一条消息并生成一条消息。您可以修改消息的键和值，包括它们的类型。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#map-org.apache.kafka.streams.kstream.KeyValueMapper-)）

                **Marks the stream for data re-partitioning:** Applying a grouping or a join after `map` will result in re-partitioning of the records. If possible use `mapValues` instead, which will not cause data re-partitioning.

                **将流标记为数据重分区：** 在`map`之后应用分组或连接操作将导致消息的重分区。如果可能，请使用`mapValues`，这不会导致数据重分区。

                ```java
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
                ```

    * **Map (values only)**

        * KStream → KStream

        * KTable → KTable

            * **Description**             

                Takes one record and produces one record, while retaining the key of the original record. You can modify the record value and the value type. ([KStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-), [KTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-))

                获取一条消息并生成一条消息，同时保留原始消息的键。您可以修改消息的值和值类型。（[KStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-)，[KTable细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#groupBy-org.apache.kafka.streams.kstream.KeyValueMapper-)）

                `mapValues` is preferable to `map` because it will not cause data re-partitioning. However, it does not allow you to modify the key or key type like `map` does.

                `mapValues`优于`map`，因为它不会导致数据重分区。但是，它不允许您像`map`一样修改键或键的类型。

                ```java
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
                ```
    * **Peek**

        * KStream → KStream

            * **Description**                 

                Performs a stateless action on each record, and returns an unchanged stream. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#peek-org.apache.kafka.streams.kstream.ForeachAction-))

                对每条消息执行无状态操作，并返回未更新的流。（[细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#peek-org.apache.kafka.streams.kstream.ForeachAction-))）

                You would use `peek` to cause side effects based on the input data (similar to `foreach`) and continue processing the input data (unlike `foreach`, which is a terminal operation). `peek` returns the input stream as-is; if you need to modify the input stream, use `map` or `mapValues` instead.

                你会使用`peek`而导致基于输入数据的副作用（类似于`foreach`），并继续处理输入数据（不像`foreach`，这是一个终端操作）。`peek`按原样返回输入流；如果您需要修改输入流，请改用`map`或`mapValues`。

                `peek` is helpful for use cases such as logging or tracking metrics or for debugging and troubleshooting.

                对于诸如日志记录、跟踪指标或用于调试和故障排除的用例，`peek`很有用。

                **Note on processing guarantees:** Any side effects of an action (such as writing to external systems) are not trackable by Kafka, which means they will typically not benefit from Kafka’s processing guarantees.

                **关于处理保障的注意事项：**Kafka无法追踪任何行为的副作用（如写入外部系统），这意味着他们通常不会从Kafka的处理中得到保障。

                ```java
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
                ```

    * **Print**

        * KStream → void

            * **Description**            

                **Terminal operation.** Prints the records to `System.out`. See Javadocs for serde and `toString()` caveats. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#print--))

                **终端操作。** 将消息打印到`System.out`。请参阅Javadocs的serde和`toString()`的注意事项。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#print--)）

                Calling `print()` is the same as calling `foreach((key, value) -> System.out.println(key + ", " + value))`

                调用`print()`与调用`foreach((key，value) - > System.out.println(key +","+ value))`相同

                ```java
                KStream<byte[], String> stream = ...;
                // print to sysout
                stream.print();

                // print to file with a custom label
                stream.print(Printed.toFile("streams.out").withLabel("streams"));
                ```

    * **SelectKey**

        * KStream → KStream

            * **Description**             

                Assigns a new key – possibly of a new key type – to each record. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#selectKey-org.apache.kafka.streams.kstream.KeyValueMapper-))

                为每个消息分配一个新的键——也可能是一个新的键的类型。（[细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#selectKey-org.apache.kafka.streams.kstream.KeyValueMapper-))）

                Calling `selectKey(mapper)` is the same as calling `map((key, value) -> mapper(key, value), value)`.

                调用`selectKey(mapper)`与调用`map((key，value) -> mapper(key，value), value)`相同。

                **Marks the stream for data re-partitioning:** Applying a grouping or a join after `selectKey` will result in re-partitioning of the records.

                **标记数据重分区的流：** 在`selectKey`之后应用分组或连接操作将导致消息的重分区。

                ```java
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
                ```

    * **Table to Stream**

        * KTable → KStream

            * **Description**  

                Get the changelog stream of this table. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#toStream--))

                获取此表的更新日志流。（[细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#toStream--))）

                ```java
                KTable<byte[], String> table = ...;

                // Also, a variant of `toStream` exists that allows you
                // to select a new key for the resulting stream.
                KStream<byte[], String> stream = table.toStream();
                ```

## Stateful transformations

## 有状态转换

Stateful transformations depend on state for processing inputs and producing outputs and require a [state store](http://kafka.apache.org/11/documentation/streams/architecture.html#streams-architecture-state) associated with the stream processor. For example, in aggregating operations, a windowing state store is used to collect the latest aggregation results per window. In join operations, a windowing state store is used to collect all of the records received so far within the defined window boundary.

有状态转换依赖于处理输入和产生输出的状态，并且需要与流处理器相关联的[状态存储器](../architecture.md)。例如，在聚合（aggregation）操作中，使用窗口状态存储器来收集每个窗口的最新聚合结果。在连接（join）操作中，窗口状态存储器用于收集到当前为止能接受到的在定义的窗口边界内收到的所有消息。

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

* **Transformation**

    * **Aggregate**

        * KGroupedStream → KTable

        * KGroupedTable → KTable

            * **Description**

                **Rolling aggregation.** Aggregates the values of (non-windowed) records by the grouped key. Aggregating is a generalization of `reduce` and allows, for example, the aggregate value to have a different type than the input values. ([KGroupedStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedStream.html), [KGroupedTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedTable.html))

                **滚动聚合。** 通过键的分组来聚合（非窗口）消息的值。聚合是`reduce`的泛化，它允许那些聚合的值具有与输入值不同的类型。（[KGroupedStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedStream.html)，[KGroupedTable细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedTable.html)）

                When aggregating a grouped stream, you must provide an initializer (e.g., `aggValue = 0`) and an “adder” aggregator (e.g., `aggValue + curValue`). When aggregating a grouped table, you must provide a “subtractor” aggregator (think: `aggValue - oldValue`).

                聚合已分组的流时，您必须提供初始值（例如，`aggValue = 0`）和“adder”聚合器（例如，`aggValue + curValue`）。聚合已分组的表时，您必须提供“subtractor”聚合器（想想：`aggValue - oldValue`）。

                Several variants of `aggregate` exist, see Javadocs for details.

                有几种不同的`聚合`形式，详情请参阅Javadocs。

                ```java
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
                ```

                Detailed behavior of `KGroupedStream`:

                `KGroupedStream`的详细行为：

                * Input records with `null` keys are ignored.

                * When a record key is received for the first time, the initializer is called (and called before the adder).

                * Whenever a record with a non-`null` value is received, the adder is called.


                * 带有`null`键的输入消息被忽略。

                * 当第一次接收到消息的键时，initializer被调用（并在adder之前被调用）。

                * 每当收到一个值为非`null`的消息时，就调用adder。

                Detailed behavior of `KGroupedTable`:

                `KGroupedTable`的详细行为：

                * Input records with `null` keys are ignored.

                * When a record key is received for the first time, the initializer is called (and called before the adder and subtractor). Note that, in contrast to `KGroupedStream`, over time the initializer may be called more than once for a key as a result of having received input tombstone records for that key (see below).

                * When the first non-`null` value is received for a key (e.g., INSERT), then only the adder is called.

                * When subsequent non-`null` values are received for a key (e.g., UPDATE), then (1) the subtractor is called with the old value as stored in the table and (2) the adder is called with the new value of the input record that was just received. The order of execution for the subtractor and adder is not defined.

                * When a tombstone record – i.e. a record with a `null` value – is received for a key (e.g., DELETE), then only the subtractor is called. Note that, whenever the subtractor returns a `null` value itself, then the corresponding key is removed from the resulting `KTable`. If that happens, any next input record for that key will trigger the initializer again.


                * 键为`null`的输入消息会被忽略。

                * 当第一次接收到消息的键时，initializer被调用（并在adder和subtractor之前调用）。请注意，与`KGroupedStream`相比，随着时间的推移，initializer可能因为接收到该键的输入逻辑删除消息（参见下文）而针对某个键被调用一次以上。

                * 当接收到一个键的第一个非`null`值（例如，INSERT）时，则只调用adder。

                * 当接收到一个键（例如UPDATE）的后续非`null`值时，则（1）对存储在表中的旧值调用subtractor，（2）对刚刚收到的输入消息的新值调用adder。subtractor和adder的执行顺序是不确定的。

                * 当接收到一个键的消息删除逻辑（即具有`null`值的消息）时（例如DELETE），则仅调用subtractor。请注意，每当subtractor本身返回一个`null`值时，相应的键就从生成的`KTable`中移除。如果发生这种情况，该键的下一个输入消息无论是什么都会再次触发initializer。

                See the example at the bottom of this section for a visualization of the aggregation semantics.

                请参阅本节底部的示例，了解聚合语义的可视化。

    * **Aggregate (windowed)**

        * KGroupedStream → KTable

            * **Description**

                **Windowed aggregation.** Aggregates the values of records, [per window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing), by the grouped key. Aggregating is a generalization of `reduce` and allows, for example, the aggregate value to have a different type than the input values. ([TimeWindowedKStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/TimeWindowedKStream.html), [SessionWindowedKStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/SessionWindowedKStream.html))

                **窗口聚合。** 按键的分组将[每个窗口](dsl-api.md)的消息值聚合。聚合是`reduce`的泛化，并且允许聚合值与输入值具有不同的类型。（[TimeWindowedKStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/TimeWindowedKStream.html)，[SessionWindowedKStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/SessionWindowedKStream.html)）

                You must provide an initializer (e.g., `aggValue = 0`), “adder” aggregator (e.g., `aggValue + curValue`), and a window. When windowing based on sessions, you must additionally provide a “session merger” aggregator (e.g., `mergedAggValue = leftAggValue + rightAggValue`).

                您必须提供一个initializer（例如，`aggValue = 0`），“adder”聚合器（例如，`aggValue + curValue`）和一个窗口。当根据会话进行窗口化时，还必须提供“会话合并”聚合器（例如，`mergedAggValue = leftAggValue + rightAggValue`）。 

                The windowed `aggregate` turns a `TimeWindowedKStream<K, V>` or `SessionWindowdKStream<K, V>` into a windowed `KTable<Windowed<K>, V>`.

                窗口化`聚合`将`TimeWindowedKStream<K, V>`或`SessionWindowdKStream<K, V>`变成窗口化的`KTable<Windowed<K>, V>`。

                Several variants of `aggregate` exist, see Javadocs for details.

                有几种不同的`聚合`形式，详情请参阅Javadocs。

                ```java
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
                ```

                Detailed behavior:

                详细情况：

                * The windowed aggregate behaves similar to the rolling aggregate described above. The additional twist is that the behavior applies *per window*.

                * Input records with null keys are ignored in general.

                * When a record key is received for the first time for a given window, the initializer is called (and called before the adder).

                * Whenever a record with a non-null value is received for a given window, the adder is called.

                * When using session windows: the session merger is called whenever two sessions are being merged.


                * 窗口化聚合的行为类似于上述的滚动聚合。该方案的不同之处在于对*每个窗口*的行为都适用。

                * 一般情况下，输入的键为null的消息将被忽略。

                * 当给定窗口首次接收到消息的键时，initializer会被调用（并在adder之前被调用）。

                * 每当给定窗口接收到具有非null值的消息时，就调用adder。

                * 使用会话窗口时：session merger在每次合并两个会话时被调用。

                See the example at the bottom of this section for a visualization of the aggregation semantics.

                请参阅本节底部的示例，了解聚合语义的可视化。

    * **Count**

        * KGroupedStream → KTable

        * KGroupedTable → KTable

            * **Description**

                **Rolling aggregation.** Counts the number of records by the grouped key. ([KGroupedStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedStream.html), [KGroupedTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedTable.html))

                **滚动聚合。** 通过分组的键计算消息数。（[KGroupedStream细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedStream.html))，[KGroupedTable细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedTable.html))）

                Several variants of `count` exist, see Javadocs for details.

                有几种`count`变体，详情请参阅Javadocs。

                ```java
                KGroupedStream<String, Long> groupedStream = ...;
                KGroupedTable<String, Long> groupedTable = ...;

                // Counting a KGroupedStream
                KTable<String, Long> aggregatedStream = groupedStream.count();

                // Counting a KGroupedTable
                KTable<String, Long> aggregatedTable = groupedTable.count();
                ```

                Detailed behavior for `KGroupedStream`:

                `KGroupedStream`的详细行为：

                * Input records with `null` keys or values are ignored.

                * 键或值为`null`的输入消息将被忽略。

                Detailed behavior for `KGroupedTable`:

                `KGroupedTable`的详细行为：

                * Input records with `null` keys are ignored. Records with `null` values are not ignored but interpreted as “tombstones” for the corresponding key, which indicate the deletion of the key from the table.

                * 具有`null`键的输入消息将被忽略。具有`null`值的消息不会被忽略，但会被解释为相应键的“删除逻辑（tombstones）”，这表明从表中删除了键。

    * **Count (windowed)**

        * KGroupedStream → KTable

            * **Description**           

                **Windowed aggregation.** Counts the number of records, [per window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing), by the grouped key. ([TimeWindowedKStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/TimeWindowedKStream.html), [SessionWindowedKStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/SessionWindowedKStream.html))

                **窗口聚合。** 通过分组键计算每个窗口的消息数。（[TimeWindowedKStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/TimeWindowedKStream.html)，[SessionWindowedKStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/SessionWindowedKStream.html)）

                The windowed `count` turns a `TimeWindowedKStream<K, V>` or `SessionWindowedKStream<K, V>` into a windowed `KTable<Windowed<K>, V>`.

                窗口化的`count`将`TimeWindowedKStream<K, V>`或`SessionWindowedKStream<K, V>`转换为窗口化的`KTable <Windowed<K>, V>`。

                Several variants of `count` exist, see Javadocs for details.

                有几种`count`变体，详情请参阅Javadocs。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * Input records with `null` keys or values are ignored.

                * 输入中带有`null`键或值的消息将被忽略。

    * **Reduce**

        * KGroupedStream → KTable

        * KGroupedTable → KTable

            * **Description**  

                **Rolling aggregation.** Combines the values of (non-windowed) records by the grouped key. The current record value is combined with the last reduced value, and a new reduced value is returned. The result value type cannot be changed, unlike `aggregate`. ([KGroupedStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedStream.html), [KGroupedTable details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedTable.html))

                **滚动聚合。** 按分组的键组合（非窗口）消息的值。当前消息值与最后一个减少的值结合在一起，并返回一个新的减少后的值。与`aggregate`不同，结果的值类型不能更改。（[KGroupedStream细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedStream.html))，[KGroupedTable细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KGroupedTable.html))）

                When reducing a grouped stream, you must provide an “adder” reducer (e.g., `aggValue + curValue`). When reducing a grouped table, you must additionally provide a “subtractor” reducer (e.g., `aggValue - oldValue`).

                在减少分组的流时，您必须提供“adder”的减少器（reducer）（例如，`aggValue + curValue`）。在减少分组的表时，还必须提供一个“subtractor”减少器（reducer）（例如，`aggValue - oldValue`）。

                Several variants of `reduce` exist, see Javadocs for details.

                有几种`reduce`存在，详见Javadocs。

                ```java
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
                ```

                Detailed behavior for `KGroupedStream`:

                `KGroupedStream`的详细行为：

                * Input records with `null` keys are ignored in general.

                * 一般情况下，输入的键为`null`的消息将被忽略。

                * When a record key is received for the first time, then the value of that record is used as the initial aggregate value.

                * 当第一次接收到一个消息的键时，那个消息的值被用作初始聚合值。

                * Whenever a record with a non-`null` value is received, the adder is called.

                * 每当收到一个非`null`值的消息时，就调用adder。

                Detailed behavior for `KGroupedTable`:

                `KGroupedTable`的详细行为：

                * Input records with `null` keys are ignored in general.

                * When a record key is received for the first time, then the value of that record is used as the initial aggregate value. Note that, in contrast to `KGroupedStream`, over time this initialization step may happen more than once for a key as a result of having received input tombstone records for that key (see below).

                * When the first non-`null` value is received for a key (e.g., INSERT), then only the adder is called.

                * When subsequent non-`null` values are received for a key (e.g., UPDATE), then (1) the subtractor is called with the old value as stored in the table and (2) the adder is called with the new value of the input record that was just received. The order of execution for the subtractor and adder is not defined.

                * When a tombstone record – i.e. a record with a `null` value – is received for a key (e.g., DELETE), then only the subtractor is called. Note that, whenever the subtractor returns a `null` value itself, then the corresponding key is removed from the resulting `KTable`. If that happens, any next input record for that key will re-initialize its aggregate value.


                * 一般情况下，输入的键为`null`的消息将被忽略。

                * 当第一次接收到一个消息键时，那个消息的值被用作初始聚合值。请注意，与`KGroupedStream`相反，随着时间的推移，由于接收到该键的输入逻辑删除的消息（见下文），此初始化步骤可能会多次发生。

                * 当接收到一个键的第一个非`null`值（例如，INSERT）时，则只调用adder。

                * 当接收到一个键（例如UPDATE）的后续非`null`值时，则（1）对存储在表中的旧值调用subtractor，（2）对刚刚收到的输入消息的新值调用adder。subtractor和adder的执行顺序是不确定的。

                * 当接收到一个键的消息删除逻辑（即具有`null`值的消息）时（例如DELETE），则仅调用subtractor。请注意，每当subtractor本身返回一个`null`值时，相应的键就从生成的`KTable`中移除。如果发生这种情况，该键的下一个输入消息无论是什么都会再次触发initializer。 

                See the example at the bottom of this section for a visualization of the aggregation semantics.

                请参阅本节底部的示例，了解聚合语义的可视化。

    * **Reduce (windowed)**

        * KGroupedStream → KTable

            * **Description**  

                **Windowed aggregation.** Combines the values of records, [per window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing), by the grouped key. The current record value is combined with the last reduced value, and a new reduced value is returned. Records with `null` key or value are ignored. The result value type cannot be changed, unlike `aggregate`. ([TimeWindowedKStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/TimeWindowedKStream.html), [SessionWindowedKStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/SessionWindowedKStream.html))

                **窗口聚合。** 通过分组的键组合[每个窗口](dsl-api.md)的消息值。当前消息值与最后一个减少的值结合在一起，并返回一个新的减少的值。键或值为`null`的输入消息被忽略。与`aggregate`不同，结果值的类型不能更改。（[TimeWindowedKStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/TimeWindowedKStream.html)，[SessionWindowedKStream细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/SessionWindowedKStream.html))）

                The windowed `reduce` turns a turns a `TimeWindowedKStream<K, V>` or a `SessionWindowedKStream<K, V>` into a windowed `KTable<Windowed<K>, V>`.

                窗口化的`Reduce`将`TimeWindowedKStream<K, V>`或`SessionWindowedKStream<K, V>`变成一个窗口化的`KTable<Windowed<K>, V>`。

                Several variants of `reduce` exist, see Javadocs for details.
                
                有几种`reduce`存在，详见Javadocs。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The windowed reduce behaves similar to the rolling reduce described above. The additional twist is that the behavior applies per window.

                * Input records with `null` keys are ignored in general.

                * When a record key is received for the first time for a given window, then the value of that record is used as the initial aggregate value.

                * Whenever a record with a non-`null` value is received for a given window, the adder is called.


                * 窗口减少表现与上述滚动减少类似。该方案的不同之处在于对每个窗口的行为都适用。

                * 一般情况下，输入的键为`null`的消息将被忽略。

                * 当给定窗口首次接收到消息的键时，该消息的值将用作初始聚合值。

                * 每当给定窗口接收到具有非`null`值的消息时，就调用adder。

                See the example at the bottom of this section for a visualization of the aggregation semantics.

                请参阅本节底部的示例，了解聚合语义的可视化。

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

<table border="1" class="docutils">
                        <colgroup>
                            <col width="11%" />
                            <col width="17%" />
                            <col width="15%" />
                            <col width="17%" />
                            <col width="18%" />
                            <col width="22%" />
                        </colgroup>
                        <thead valign="bottom">
                        <tr class="row-odd"><th class="head">&nbsp;</th>
                            <th class="head" colspan="2">KStream <code class="docutils literal"><span class="pre">wordCounts</span></code></th>
                            <th class="head" colspan="2">KGroupedStream <code class="docutils literal"><span class="pre">groupedStream</span></code></th>
                            <th class="head">KTable <code class="docutils literal"><span class="pre">aggregated</span></code></th>
                        </tr>
                        <tr class="row-even"><th class="head">Timestamp</th>
                            <th class="head">Input record</th>
                            <th class="head">Grouping</th>
                            <th class="head">Initializer</th>
                            <th class="head">Adder</th>
                            <th class="head">State</th>
                        </tr>
                        </thead>
                        <tbody valign="top">
                        <tr class="row-odd"><td>1</td>
                            <td>(hello, 1)</td>
                            <td>(hello, 1)</td>
                            <td>0 (for hello)</td>
                            <td>(hello, 0 + 1)</td>
                            <td><div class="first last line-block">
                                <div class="line"><strong>(hello, 1)</strong></div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-even"><td>2</td>
                            <td>(kafka, 1)</td>
                            <td>(kafka, 1)</td>
                            <td>0 (for kafka)</td>
                            <td>(kafka, 0 + 1)</td>
                            <td><div class="first last line-block">
                                <div class="line">(hello, 1)</div>
                                <div class="line"><strong>(kafka, 1)</strong></div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-odd"><td>3</td>
                            <td>(streams, 1)</td>
                            <td>(streams, 1)</td>
                            <td>0 (for streams)</td>
                            <td>(streams, 0 + 1)</td>
                            <td><div class="first last line-block">
                                <div class="line">(hello, 1)</div>
                                <div class="line">(kafka, 1)</div>
                                <div class="line"><strong>(streams, 1)</strong></div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-even"><td>4</td>
                            <td>(kafka, 1)</td>
                            <td>(kafka, 1)</td>
                            <td>&nbsp;</td>
                            <td>(kafka, 1 + 1)</td>
                            <td><div class="first last line-block">
                                <div class="line">(hello, 1)</div>
                                <div class="line">(kafka, <strong>2</strong>)</div>
                                <div class="line">(streams, 1)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-odd"><td>5</td>
                            <td>(kafka, 1)</td>
                            <td>(kafka, 1)</td>
                            <td>&nbsp;</td>
                            <td>(kafka, 2 + 1)</td>
                            <td><div class="first last line-block">
                                <div class="line">(hello, 1)</div>
                                <div class="line">(kafka, <strong>3</strong>)</div>
                                <div class="line">(streams, 1)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-even"><td>6</td>
                            <td>(streams, 1)</td>
                            <td>(streams, 1)</td>
                            <td>&nbsp;</td>
                            <td>(streams, 1 + 1)</td>
                            <td><div class="first last line-block">
                                <div class="line">(hello, 1)</div>
                                <div class="line">(kafka, 3)</div>
                                <div class="line">(streams, <strong>2</strong>)</div>
                            </div>
                            </td>
                        </tr>
                        </tbody>
                    </table>

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

<table border="1" class="docutils">
                        <colgroup>
                            <col width="9%" />
                            <col width="14%" />
                            <col width="15%" />
                            <col width="11%" />
                            <col width="11%" />
                            <col width="11%" />
                            <col width="11%" />
                            <col width="19%" />
                        </colgroup>
                        <thead valign="bottom">
                        <tr class="row-odd"><th class="head">&nbsp;</th>
                            <th class="head" colspan="3">KTable <code class="docutils literal"><span class="pre">userProfiles</span></code></th>
                            <th class="head" colspan="3">KGroupedTable <code class="docutils literal"><span class="pre">groupedTable</span></code></th>
                            <th class="head">KTable <code class="docutils literal"><span class="pre">aggregated</span></code></th>
                        </tr>
                        <tr class="row-even"><th class="head">Timestamp</th>
                            <th class="head">Input record</th>
                            <th class="head">Interpreted as</th>
                            <th class="head">Grouping</th>
                            <th class="head">Initializer</th>
                            <th class="head">Adder</th>
                            <th class="head">Subtractor</th>
                            <th class="head">State</th>
                        </tr>
                        </thead>
                        <tbody valign="top">
                        <tr class="row-odd"><td>1</td>
                            <td>(alice, E)</td>
                            <td>INSERT alice</td>
                            <td>(E, 5)</td>
                            <td>0 (for E)</td>
                            <td>(E, 0 + 5)</td>
                            <td>&nbsp;</td>
                            <td><div class="first last line-block">
                                <div class="line"><strong>(E, 5)</strong></div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-even"><td>2</td>
                            <td>(bob, A)</td>
                            <td>INSERT bob</td>
                            <td>(A, 3)</td>
                            <td>0 (for A)</td>
                            <td>(A, 0 + 3)</td>
                            <td>&nbsp;</td>
                            <td><div class="first last line-block">
                                <div class="line"><strong>(A, 3)</strong></div>
                                <div class="line">(E, 5)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-odd"><td>3</td>
                            <td>(charlie, A)</td>
                            <td>INSERT charlie</td>
                            <td>(A, 7)</td>
                            <td>&nbsp;</td>
                            <td>(A, 3 + 7)</td>
                            <td>&nbsp;</td>
                            <td><div class="first last line-block">
                                <div class="line">(A, <strong>10</strong>)</div>
                                <div class="line">(E, 5)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-even"><td>4</td>
                            <td>(alice, A)</td>
                            <td>UPDATE alice</td>
                            <td>(A, 5)</td>
                            <td>&nbsp;</td>
                            <td>(A, 10 + 5)</td>
                            <td>(E, 5 - 5)</td>
                            <td><div class="first last line-block">
                                <div class="line">(A, <strong>15</strong>)</div>
                                <div class="line">(E, <strong>0</strong>)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-odd"><td>5</td>
                            <td>(charlie, null)</td>
                            <td>DELETE charlie</td>
                            <td>(null, 7)</td>
                            <td>&nbsp;</td>
                            <td>&nbsp;</td>
                            <td>(A, 15 - 7)</td>
                            <td><div class="first last line-block">
                                <div class="line">(A, <strong>8</strong>)</div>
                                <div class="line">(E, 0)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-even"><td>6</td>
                            <td>(null, E)</td>
                            <td><em>ignored</em></td>
                            <td>&nbsp;</td>
                            <td>&nbsp;</td>
                            <td>&nbsp;</td>
                            <td>&nbsp;</td>
                            <td><div class="first last line-block">
                                <div class="line">(A, 8)</div>
                                <div class="line">(E, 0)</div>
                            </div>
                            </td>
                        </tr>
                        <tr class="row-odd"><td>7</td>
                            <td>(bob, E)</td>
                            <td>UPDATE bob</td>
                            <td>(E, 3)</td>
                            <td>&nbsp;</td>
                            <td>(E, 0 + 3)</td>
                            <td>(A, 8 - 3)</td>
                            <td><div class="first last line-block">
                                <div class="line">(A, <strong>5</strong>)</div>
                                <div class="line">(E, <strong>3</strong>)</div>
                            </div>
                            </td>
                        </tr>
                        </tbody>
                    </table>

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

连接时必须对输入数据进行共分区（co-partitioned）。这确保了在处理期间，要连接的两侧的具有相同键的输入消息被传送到相同的流任务。**用户需要确保连接的数据是共分区的。**

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

### 提示

**Kafka Streams partly verifies the co-partitioning requirement:** During the partition assignment step, i.e. at runtime, Kafka Streams verifies whether the number of partitions for both sides of a join are the same. If they are not, a ```TopologyBuilderException``` (runtime exception) is being thrown. Note that Kafka Streams cannot verify whether the partitioning strategy matches between the input streams/tables of a join – it is up to the user to ensure that this is the case.

**Kafka Streams部分验证共同分区要求：** 在分区分配步骤中，即在运行时，Kafka Streams验证连接两端的分区数是否相同。如果它们不同，则抛出`TopologyBuilderException`（运行时异常）。请注意，Kafka Streams无法验证分割策略是否在连接的输入流/表之间相匹配 - 需由用户确定是否属于这种情况。

**Ensuring data co-partitioning:** If the inputs of a join are not co-partitioned yet, you must ensure this manually. You may follow a procedure such as outlined below.

**确保数据共分区：** 如果连接的输入数据尚未做共同分区操作，则必须手动确保这一点。您可以按照以下程序进行操作。

1. Identify the input KStream/KTable in the join whose underlying Kafka topic has the smaller number of partitions. Let’s call this stream/table “SMALLER”, and the other side of the join “LARGER”. To learn about the number of partitions of a Kafka topic you can use, for example, the CLI tool ```bin/kafka-topics``` with the ```--describe``` option.

1. 识别连接中输入的KStream/KTable，其底层Kafka主题的分区数量较少。我们把这个流/表称为“SMALLER”，相反的，将分区数量较多的连接称为“LARGER”。要了解Kafka主题的分区数量，可以使用CLI工具`bin/kafka-topics`和`--describe`选项。

2. Pre-create a new Kafka topic for “SMALLER” that has the same number of partitions as “LARGER”. Let’s call this new topic “repartitioned-topic-for-smaller”. Typically, you’d use the CLI tool ```bin/kafka-topics``` with the ```--create``` option for this.

2. 为“SMALLER”预先创建一个与“LARGER”的分区数相同的新Kafka主题。让我们称之为“repartitioned-topic-for-smaller”新主题。通常情况下，你可以使用CLI工具`bin/kafka-topics`和`--create`选项。

3. Within your application, re-write the data of “SMALLER” into the new Kafka topic. You must ensure that, when writing the data with ```to``` or ```through```, the same partitioner is used as for “LARGER”.

3. 在您的应用程序中，将“SMALLER”的数据重新写入新的Kafka主题。您必须确保在使用```to```或```through```写入数据时使用与“LARGER”相同的分区程序。

    * If “SMALLER” is a KStream: ```KStream#to("repartitioned-topic-for-smaller")```.
    * If “SMALLER” is a KTable: ```KTable#to("repartitioned-topic-for-smaller")```.


    * 如果“SMALLER”是KStream：`KStream#to("repartitioned-topic-for-smaller")`。
    * 如果“SMALLER”是KTable：`KTable＃#o("repartitioned-topic-for-smaller")`。

4. Within your application, re-read the data in “repartitioned-topic-for-smaller” into a new KStream/KTable.

4. 在您的应用程序中，重新读取“repartitioned-topic-for-smaller”中的数据到新的KStream/KTable中。

    * If “SMALLER” is a KStream: ```StreamsBuilder#stream("repartitioned-topic-for-smaller")```.
    * If “SMALLER” is a KTable: ```StreamsBuilder#table("repartitioned-topic-for-smaller")```.


    * 如果“SMALLER”是KStream：`StreamsBuilder#stream("repartitioned-topic-for-smaller")`。
    * 如果“SMALLER”是KTable：`StreamsBuilder#table("repartitioned-topic-for-smaller")`。

5. Within your application, perform the join between “LARGER” and the new stream/table.

5. 在你的应用程序中，执行“LARGER”和新的流/表之间的连接。

### KStream-KStream Join

### 连接KStream-KStream

KStream-KStream joins are always [windowed](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-sliding) joins, because otherwise the size of the internal state store used to perform the join – e.g., a [sliding window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-sliding) or “buffer” – would grow indefinitely. For stream-stream joins it’s important to highlight that a new input record on one side will produce a join output for each matching record on the other side, and there can be multiple such matching records in a given join window (cf. the row with timestamp 15 in the join semantics table below, for example).

KStream-KStream连接始终是[窗口](dsl-api.md)连接，否则用于执行连接操作的内部状态存储器的大小（例如，[滑动窗口](dsl-api.md)或“缓冲区”）将无限增长。对于stream-stream连接，重要的是强调一边的新输入消息将为另一边的每个匹配消息产生连接后的输出，并且在给定的连接窗口中可以有多个这样的匹配消息（参见下面的连接语义表中的时间戳15）。

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

通过用户提供的`ValueJoiner`，可以有效地连接输出消息，如下所示：

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

* **Transformation**

    * **Inner Join (windowed)**

        * (KStream, KStream) → KStream

            * **Description**
	
                Performs an INNER JOIN of this stream with another stream. Even though this operation is windowed, the joined stream will be of type `KStream<K, ...>` rather than `KStream<Windowed<K>, ...>`. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#join-org.apache.kafka.streams.kstream.KStream-org.apache.kafka.streams.kstream.ValueJoiner-org.apache.kafka.streams.kstream.JoinWindows-))

                对另一个流执行此流的INNER JOIN操作。即使该操作是窗口化的，连接后的流将是`KStream<K, ...>`类型而不是`KStream<Windowed<K>, ...>`。（[细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#join-org.apache.kafka.streams.kstream.KStream-org.apache.kafka.streams.kstream.ValueJoiner-org.apache.kafka.streams.kstream.JoinWindows-))）

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须[共同分区](dsl-api.md)。

                **Causes data re-partitioning of a stream if and only if the stream was marked for re-partitioning (if both are marked, both are re-partitioned).**

                **当且仅当流被标记为重分区时（如果两者都被标记，两者都被重分区），才会导致数据重分区。**

                Several variants of `join` exists, see the Javadocs for details.

                存在多个`join`变体，请参阅Javadoc以获取详细信息。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`, and *window-based*, i.e. two input records are joined if and only if their timestamps are “close” to each other as defined by the user-supplied `JoinWindows`, i.e. the window defines an additional join predicate over the record timestamps.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`和*基于窗口*的连接条件，即两个输入消息被连接，当且仅当它们的时间戳彼此“接近”时，由用户定义`JoinWindows`，即窗口在消息时间戳上定义一个额外的连接条件。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Input records with a `null` key or a `null` value are ignored and do not trigger the join.

                    * 具有`null`键或`null`值的输入消息将被忽略，并且不会触发连接操作。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

    * **Left Join (windowed)**

        * (KStream, KStream) → KStream

            * **Description**

                Performs a LEFT JOIN of this stream with another stream. Even though this operation is windowed, the joined stream will be of type `KStream<K, ...>` rather than `KStream<Windowed<K>, ...>`. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#leftJoin-org.apache.kafka.streams.kstream.KStream-org.apache.kafka.streams.kstream.ValueJoiner-org.apache.kafka.streams.kstream.JoinWindows-))

                对另一个流执行此流的LEFT JOIN操作。即使该操作是窗口化的，连接后的流将是`KStream<K, ...>`类型而不是`KStream<Windowed<K>, ...>`。（[细节]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#leftJoin-org.apache.kafka.streams.kstream.KStream-org.apache.kafka.streams.kstream.ValueJoiner-org.apache.kafka.streams.kstream.JoinWindows-))）

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须[共同分区](dsl-api.md)。

                **Causes data re-partitioning of a stream if and only if the stream was marked for re-partitioning (if both are marked, both are re-partitioned).**

                **当且仅当流被标记为重分区时（如果两者都被标记，两者都被重分区），才会导致数据重分区。**

                Several variants of `leftJoin` exists, see the Javadocs for details.

                存在几个`leftJoin`的变体，请参阅Javadocs以获取详细信息。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`, and *window-based*, i.e. two input records are joined if and only if their timestamps are “close” to each other as defined by the user-supplied `JoinWindows`, i.e. the window defines an additional join predicate over the record timestamps.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`和*基于窗口*的连接条件，即两个输入消息被连接，当且仅当它们的时间戳彼此“接近”时，由用户定义`JoinWindows`，即窗口在消息时间戳上定义一个额外的连接条件。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Input records with a `null` key or a `null` value are ignored and do not trigger the join.

                    * 具有`null`键或`null`值的输入消息将被忽略，并且不会触发连接操作。

                * For each input record on the left side that does not have any match on the right side, the `ValueJoiner` will be called with `ValueJoiner#apply(leftRecord.value, null)`; this explains the row with timestamp=3 in the table below, which lists `[A, null]` in the LEFT JOIN column.

                * 对于左侧没有任何匹配的每个输入消息，`ValueJoiner`将调用`ValueJoiner#apply(leftRecord.value, null)`；这解释了下表中的timestamp = 3的行，其中列出了LEFT JOIN列中的`[A, null]`。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

    * **Outer Join (windowed)**

        * (KStream, KStream) → KStream

            * **Description**

                Performs an OUTER JOIN of this stream with another stream. Even though this operation is windowed, the joined stream will be of type `KStream<K, ...>` rather than `KStream<Windowed<K>, ...>`. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#outerJoin-org.apache.kafka.streams.kstream.KStream-org.apache.kafka.streams.kstream.ValueJoiner-org.apache.kafka.streams.kstream.JoinWindows-)

                用另一个流执行此流的OUTER JOIN操作。即使该操作是窗口化的，连接后的流将是`KStream<K, ...>`类型而不是`KStream<Windowed<K>, ...>`。[（细节）]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#outerJoin-org.apache.kafka.streams.kstream.KStream-org.apache.kafka.streams.kstream.ValueJoiner-org.apache.kafka.streams.kstream.JoinWindows-))

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须[共同分区](dsl-api.md)。

                **Causes data re-partitioning of a stream if and only if the stream was marked for re-partitioning (if both are marked, both are re-partitioned).**

                **当且仅当流被标记为重分区时（如果两者都被标记，两者都被重分区），才会导致数据重分区。**

                Several variants of `outerJoin` exists, see the Javadocs for details.

                存在几个`outerJoin`的变体，请参阅Javadocs以获取详细信息。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`, and *window-based*, i.e. two input records are joined if and only if their timestamps are “close” to each other as defined by the user-supplied `JoinWindows`, i.e. the window defines an additional join predicate over the record timestamps.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`和*基于窗口*的连接条件，即两个输入消息被连接，当且仅当它们的时间戳彼此“接近”时，由用户定义的`JoinWindows`，即窗口在消息时间戳上定义一个额外的连接条件。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Input records with a `null` key or a `null` value are ignored and do not trigger the join.

                    * 具有`null`键或`null`值的输入消息将被忽略，并且不会触发连接操作。

                * For each input record on one side that does not have any match on the other side, the `ValueJoiner` will be called with `ValueJoiner#apply(leftRecord.value, null)` or `ValueJoiner#apply(null, rightRecord.value)`, respectively; this explains the row with timestamp=3 in the table below, which lists `[A, null]` in the OUTER JOIN column (unlike LEFT JOIN, `[null, x]` is possible, too, but no such example is shown in the table).

                * 对于每一个没有任何匹配的输入消息，`ValueJoiner`将分别通过`ValueJoiner#apply(leftRecord.value, null)`或`ValueJoiner#apply(null, rightRecord.value)`来调用；这解释了下表中的timestamp = 3的行，它在OUTER JOIN列中列出了`[A, null]`（不同于LEFT JOIN，`[null, x]`也是可能的，但表中没有显示这样的示例）。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

**Semantics of stream-stream joins:** The semantics of the various stream-stream join variants are explained below. To improve the readability of the table, assume that (1) all records have the same key (and thus the key in the table is omitted), (2) all records belong to a single join window, and (3) all records are processed in timestamp order. The columns INNER JOIN, LEFT JOIN, and OUTER JOIN denote what is passed as arguments to the user-supplied [ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html) for the ```join```, ```leftJoin```, and ```outerJoin``` methods, respectively, whenever a new input record is received on either side of the join. An empty table cell denotes that the ```ValueJoiner``` is not called at all.

**stream-stream连接的语义：** 下面将解释各种stream-stream连接变体的语义。为了提高表的可读性，假设（1）所有消息具有相同的键（因此表中的键被省略），（2）所有消息属于单个连接窗口，以及（3）所有消息按时间戳顺序处理。INNER JOIN，LEFT JOIN和OUTER JOIN列分别表示每当在连接的任一侧收到新的输入消息时，分别作为参数传递给用户提供的[ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html)的`join`，`leftJoin`和`outerJoin`方法的内容。一个空表单元表示`ValueJoiner`不会被调用。

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

时间戳 | 左边 (KStream) | 右边 (KStream) | (内) 连接 |	左连接 | 外连接
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

### KTable-KTable 连接

KTable-KTable joins are always *non-windowed* joins. They are designed to be consistent with their counterparts in relational databases. The changelog streams of both KTables are materialized into local state stores to represent the latest snapshot of their [table duals](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-ktable). The join result is a new KTable that represents the changelog stream of the join operation.

KTable-KTable连接始终是*非窗口*连接。它们旨在与关系数据库中的对应方保持一致。两个KTables的更新日志流都持久化到本地状态存储器中，以表示其[双重表](../concepts.md)（table duals）的最新快照。连接结果是一个新的KTable，代表连接操作的更新日志流。

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

通过用户提供的`ValueJoiner`，可以有效地连接输出消息，如下所示：

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

* **Transformation**

    * **Inner Join**

        * (KTable, KTable) → KTable

            * **Description**

                Performs an INNER JOIN of this table with another table. The result is an ever-updating KTable that represents the “current” result of the join. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#join-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-)

                结合另一个表，对此表执行INNER JOIN操作。结果是一个不断更新的KTable，它代表了“当前”连接的结果。[（细节）]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#join-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-))

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须共同分区。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Input records with a `null` key are ignored and do not trigger the join.

                    * Input records with a `null` value are interpreted as *tombstones* for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join. When an input tombstone is received, then an output tombstone is forwarded directly to the join result KTable if required (i.e. only if the corresponding key actually exists already in the join result KTable).


                    * 带有`null`键的输入消息将被忽略，不会触发连接操作。

                    * 具有`null`值的输入消息被解释为相应键的*删除逻辑*（tombstones），其指示从表中删除键。删除逻辑不会触发连接操作。当接收到输入删除逻辑时，如果需要（即仅当相应的键实际上已经存在于连接结果KTable中），才将输出删除逻辑直接转发给连接结果KTable。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

    * **Left Join**

        * (KTable, KTable) → KTable

            * **Description**

                Performs a LEFT JOIN of this table with another table. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#leftJoin-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-)

                与另一个表执行此表的LEFT JOIN操作。[（细节）](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#leftJoin-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-)


                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须共同分区。

                ```java
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
                ```
                
                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Input records with a `null` key are ignored and do not trigger the join.

                    * Input records with a `null` value are interpreted as *tombstones* for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join. When an input tombstone is received, then an output tombstone is forwarded directly to the join result KTable if required (i.e. only if the corresponding key actually exists already in the join result KTable).


                    * 带有`null`键的输入消息被忽略，不会触发连接操作。

                    * 具有`null`值的输入消息被解释为相应键的*删除逻辑*（tombstones），其指示从表中删除键。删除逻辑不会触发连接操作。当接收到输入删除逻辑时，如果需要（即仅当相应的键实际上已经存在于连接结果KTable中），才将输出删除逻辑直接转发给连接结果KTable。

                * For each input record on the left side that does not have any match on the right side, the `ValueJoiner` will be called with `ValueJoiner#apply(leftRecord.value, null)`; this explains the row with timestamp=3 in the table below, which lists `[A, null]` in the LEFT JOIN column.

                * 对于左侧没有任何匹配的每个输入记录，`ValueJoiner`将调用`ValueJoiner#apply(leftRecord.value, null)`；这解释了下表中的timestamp = 3的行，其中列出了LEFT JOIN列中的`[A, null]`。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

    * **Outer Join**

        * (KTable, KTable) → KTable

            * **Description**

                Performs an OUTER JOIN of this table with another table. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#outerJoin-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-)

                与另一个表执行此表的OUTER JOIN。[（细节）]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KTable.html#outerJoin-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-))

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须[共同分区](dsl-api.md)。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Input records with a `null` key are ignored and do not trigger the join.
                
                    * Input records with a `null` value are interpreted as *tombstones* for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join. When an input tombstone is received, then an output tombstone is forwarded directly to the join result KTable if required (i.e. only if the corresponding key actually exists already in the join result KTable).


                    * 带有`null`键的输入消息被忽略，不会触发连接操作。

                    * 具有`null`值的输入消息被解释为相应键的*删除逻辑*（tombstones），其指示从表中删除键。删除逻辑不会触发连接操作。当接收到输入删除逻辑时，如果需要（即仅当相应的键实际上已经存在于连接结果KTable中），才将输出删除逻辑直接转发给连接结果KTable。
                
                * For each input record on one side that does not have any match on the other side, the `ValueJoiner` will be called with `ValueJoiner#apply(leftRecord.value, null)` or `ValueJoiner#apply(null, rightRecord.value)`, respectively; this explains the rows with timestamp=3 and timestamp=7 in the table below, which list `[A, null]` and `[null, b]`, respectively, in the OUTER JOIN column.

                * 对于每一个没有任何匹配的输入消息，`ValueJoiner`将分别通过 `ValueJoiner#apply(leftRecord.value, null)`或`ValueJoiner#apply(null, rightRecord.value)`来调用；这解释了下表中的timestamp = 3和timestamp = 7的行，它们分别在OUTER JOIN列中列出`[A, null]`和`[null, b]`。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

**Semantics of table-table joins:** The semantics of the various table-table join variants are explained below. To improve the readability of the table, you can assume that (1) all records have the same key (and thus the key in the table is omitted) and that (2) all records are processed in timestamp order. The columns INNER JOIN, LEFT JOIN, and OUTER JOIN denote what is passed as arguments to the user-supplied [ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html) for the ```join```, ```leftJoin```, and ```outerJoin``` methods, respectively, whenever a new input record is received on either side of the join. An empty table cell denotes that the ```ValueJoiner``` is not called at all.

**表连接的语义：** 下面将解释各种table-table连接变体的语义。为了提高表的可读性，可以假设（1）所有消息具有相同的键（因此表中的键被省略），并且（2）所有消息按时间戳顺序处理。INNER JOIN，LEFT JOIN和OUTER JOIN列分别表示每当在连接的任一侧收到新的输入消息时，分别作为参数传递给用户提供的[ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html)的`join`，`leftJoin`和`outerJoin`方法的内容。一个空表单元表示`ValueJoiner`不会被调用。

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

时间戳 |左边 (KTable) | 右边 (KTable) | (内) 连接 | 左连接 | 外连接
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

### KStream-KTable连接

KStream-KTable joins are always *non-windowed* joins. They allow you to perform *table lookups* against a KTable (changelog stream) upon receiving a new record from the KStream (record stream). An example use case would be to enrich a stream of user activities (KStream) with the latest user profile information (KTable).

KStream-KTable连接始终是*非窗口*连接。它们允许您在接收到来自KStream（消息流）的新消息时，对KTable（更新日志流）执行*表查找*。一个示例是用最新的用户配置文件信息（KTable）来增强用户活动流（KStream）。

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

通过用户提供的`ValueJoiner`，可以有效地连接输出消息，如下所示：

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

* **Transformation**

    * **Inner Join**

        * (KStream, KTable) → KStream

            * **Description**

                Performs an INNER JOIN of this stream with the table, effectively doing a table lookup. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#join-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-)

                对该表执行此流的INNER JOIN操作，从而有效地进行表查找。[（细节）]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#join-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-))

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须[共同分区](dsl-api.md)。

                **Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.**

                **当且仅当流被标记为重分区时，才会导致数据流重分区。**

                Several variants of `join` exists, see the Javadocs for details.

                存在多个`join`变体，请参阅Javadoc以获取详细信息。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.
                
                    * Input records for the stream with a `null` key or a `null` value are ignored and do not trigger the join.
                
                    * Input records for the table with a `null` value are interpreted as *tombstones* for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join.


                    * 只有左侧（流）的输入消息触发连接。右侧（表）的输入消息仅更新内部右侧连接状态。
                    
                    * 具有`null`键或`null`值的流的输入消息将被忽略，并且不会触发连接操作。
                    
                    * 具有`null`值的表的输入消息被解释为相应键的*删除逻辑*（tombstones），其指示从表中删除键。删除逻辑不会触发连接操作。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

    * **Left Join**

        * (KStream, KTable) → KStream

            * **Description**

                Performs a LEFT JOIN of this stream with the table, effectively doing a table lookup. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#leftJoin-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-)

                对该表执行此流的LEFT JOIN操作，从而有效地进行表查找。[（细节）]((http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#leftJoin-org.apache.kafka.streams.kstream.KTable-org.apache.kafka.streams.kstream.ValueJoiner-))

                **Data must be co-partitioned:** The input data for both sides must be [co-partitioned](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

                **数据必须共同分区：** 双方的输入数据必须共同分区。

                **Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.**

                **当且仅当流被标记为重分区时，才会导致数据流重分区。**

                Several variants of `leftJoin` exists, see the Javadocs for details.

                存在几个`leftJoin`的变体，请参阅Javadocs以获取详细信息。

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is *key-based*, i.e. with the join predicate `leftRecord.key == rightRecord.key`.

                * 连接是*基于键*的，即连接条件为`leftRecord.key == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以连接输出消息。

                    * Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.

                    * Input records for the stream with a `null` key or a `null` value are ignored and do not trigger the join.

                    * Input records for the table with a `null` value are interpreted as *tombstones* for the corresponding key, which indicate the deletion of the key from the table. Tombstones do not trigger the join.


                    * 只有左侧（流）的输入消息能触发连接操作。右侧（表格）的输入消息仅更新内部右侧连接状态。

                    * 具有`null`键或`null`值的流的输入消息将被忽略，并且不会触发连接操作。

                    * 具有`null`值的表的输入消息被解释为相应键的*删除逻辑*（tombstones），其指示从表中删除键。删除逻辑不会触发连接操作。

                For each input record on the left side that does not have any match on the right side, the `ValueJoiner` will be called with `ValueJoiner#apply(leftRecord.value, null)`; this explains the row with timestamp=3 in the table below, which lists `[A, null]` in the LEFT JOIN column.

                对于每个左侧没有任何匹配的输入消息，`ValueJoiner`将调用`ValueJoiner#apply(leftRecord.value, null)`;这解释了下表中的timestamp = 3的行，其中列出了LEFT JOIN列中的`[A, null]`。

                See the semantics overview at the bottom of this section for a detailed description.

                有关详细说明，请参阅本节底部的语义概述。

**Semantics of stream-table joins:** The semantics of the various stream-table join variants are explained below. To improve the readability of the table we assume that (1) all records have the same key (and thus we omit the key in the table) and that (2) all records are processed in timestamp order. The columns INNER JOIN and LEFT JOIN denote what is passed as arguments to the user-supplied [ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html) for the ```join``` and ```leftJoin``` methods, respectively, whenever a new input record is received on either side of the join. An empty table cell denotes that the ```ValueJoiner``` is not called at all.

**stream-table连接的语义：** 下面将解释各种stream-table连接变体的语义。为了提高表的可读性，我们假设（1）所有消息具有相同的键（因此我们省略了表中的键），并且（2）所有消息按时间戳顺序处理。INNER JOIN和LEFT JOIN列分别表示在连接的任一侧收到新的输入消息时，分别作为用户提供的[ValueJoiner](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/ValueJoiner.html)的参数传递给`join`和`leftJoin`方法。一个空表格单元表示`ValueJoiner`不会被调用。

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

时间戳 | 左边 (KStream) | 右边 (KTable) | （内）连接 | 左连接
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

## KStream-GlobalKTable连接

KStream-GlobalKTable joins are always non-windowed joins. They allow you to perform *table lookups* against a [GlobalKTable](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-globalktable) (entire changelog stream) upon receiving a new record from the KStream (record stream). An example use case would be “star queries” or “star joins”, where you would enrich a stream of user activities (KStream) with the latest user profile information (GlobalKTable) and further context information (further GlobalKTables).

KStream-GlobalKTable连接始终是非窗口连接。它们允许您在接收到来自KStream（消息流）的新消息时，针对[GlobalKTable](../concepts.md)（整个更新日志流）执行*表查找*。一个示例是“星形查询”或“星形连接”，您可以在其中使用最新的用户配置文件信息（GlobalKTable）和其他上下文信息（更多GlobalKTables）来增强用户活动流（KStream）。

At a high-level, KStream-GlobalKTable joins are very similar to [KStream-KTable joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-ktable). However, global tables provide you with much more flexibility at the [some expense](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-globalktable) when compared to partitioned tables:

在高层次上，KStream-GlobalKTable连接与[KStream-KTable连接](dsl-api.md)非常相似。但是，与分区表相比，全局表为您提供了更高的灵活性，但[开销却很高](../concepts.md)：

* They do not require [data co-partitioning](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-co-partitioning).

* They allow for efficient “star joins”; i.e., joining a large-scale “facts” stream against “dimension” tables

* They allow for joining against foreign keys; i.e., you can lookup data in the table not just by the keys of records in the stream, but also by data in the record values.

* They make many use cases feasible where you must work on heavily skewed data and thus suffer from hot partitions.

* They are often more efficient than their partitioned KTable counterpart when you need to perform multiple joins in succession.


* 它们不需要[数据共分区](dsl-api.md)。

* 他们允许高效的“星形连接”；即将大规模的“事实”（facts）流与“维度”（dimension）表结合起来

* 他们允许在连接的时候不管外键（against foreign keys）；即可以在表中不仅仅通过流中消息的键来查找数据，还可以通过消息的值中的数据来查找。

* 您有时必须处理严重倾斜的数据，因此会受到热分区的困扰，而它们使得这样的许多用例可行。

* 当您需要连续执行多个连接时，它们通常比分区的KTable对象更高效。

Join output records are effectively created as follows, leveraging the user-supplied ```ValueJoiner```:

通过用户提供的`ValueJoiner`，可以有效地创建连接输出消息，如下所示：

```java
KeyValue<K, LV> leftRecord = ...;
KeyValue<K, RV> rightRecord = ...;
ValueJoiner<LV, RV, JV> joiner = ...;

KeyValue<K, JV> joinOutputRecord = KeyValue.pair(
    leftRecord.key, /* by definition, leftRecord.key == rightRecord.key */
    joiner.apply(leftRecord.value, rightRecord.value)
  );
```

* **Transformation**

    * **Inner Join**

        * (KStream, GlobalKTable) → KStream
	        
            * **Description**

                Performs an INNER JOIN of this stream with the global table, effectively doing a table lookup. [(details)](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#join-org.apache.kafka.streams.kstream.GlobalKTable-org.apache.kafka.streams.kstream.KeyValueMapper-org.apache.kafka.streams.kstream.ValueJoiner-)

                执行此流与全局表的INNER JOIN操作，有效地进行表查找。[（细节）](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#join-org.apache.kafka.streams.kstream.GlobalKTable-org.apache.kafka.streams.kstream.KeyValueMapper-org.apache.kafka.streams.kstream.ValueJoiner-)

                The `GlobalKTable` is fully bootstrapped upon (re)start of a `KafkaStreams` instance, which means the table is fully populated with all the data in the underlying topic that is available at the time of the startup. The actual data processing begins only once the bootstrapping has completed.

                `GlobalKTable`在`KafkaStreams`实例（重新）启动时被完全引导，这意味着该表涵盖了启动时可用的底层主题中的所有数据。实际的数据处理仅在引导完成后才开始。

                **Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.**

                **当且仅当流被标记为重分区时，才会导致数据流重分区。**

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is indirectly *key-based*, i.e. with the join predicate `KeyValueMapper#apply(leftRecord.key, leftRecord.value) == rightRecord.key`.

                这个连接是间接*基于键*的，即连接条件为`KeyValueMapper#apply(leftRecord.key, leftRecord.value) == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以产生连接输出消息。

                    * Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.

                    * 只有左侧（流）的输入消息触发连接操作。右侧（表）的输入消息仅更新内部右侧连接状态。
                
                    * Input records for the stream with a `null` key or a `null` value are ignored and do not trigger the join.

                    * 具有`null`键或`null`值的流的输入消息将被忽略，并且不会触发连接。
                
                    * Input records for the table with a `null` value are interpreted as *tombstones*, which indicate the deletion of a record key from the table. Tombstones do not trigger the join.

                    * 具有`null`值的表的输入消息被解释为*删除逻辑*（tombstones），其指示从表中删除消息键。删除逻辑不会触发连接操作。

    * **Left Join**

        * (KStream, GlobalKTable) → KStream
	        
            * **Description**

                Performs a LEFT JOIN of this stream with the global table, effectively doing a table lookup. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#leftJoin-org.apache.kafka.streams.kstream.GlobalKTable-org.apache.kafka.streams.kstream.KeyValueMapper-org.apache.kafka.streams.kstream.ValueJoiner-))

                对全局表执行此流的LEFT JOIN操作，从而有效地进行表查找。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#leftJoin-org.apache.kafka.streams.kstream.GlobalKTable-org.apache.kafka.streams.kstream.KeyValueMapper-org.apache.kafka.streams.kstream.ValueJoiner-)）

                The `GlobalKTable` is fully bootstrapped upon (re)start of a `KafkaStreams` instance, which means the table is fully populated with all the data in the underlying topic that is available at the time of the startup. The actual data processing begins only once the bootstrapping has completed.

                `GlobalKTable`在`KafkaStreams`实例（重新）启动时被完全引导，这意味着该表涵盖了启动时可用的底层主题中的所有数据。实际的数据处理仅在引导完成后才开始。

                **Causes data re-partitioning of the stream if and only if the stream was marked for re-partitioning.**

                **当且仅当流被标记为重分区时，才会导致数据流重分区。**

                ```java
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
                ```

                Detailed behavior:

                详细行为：

                * The join is indirectly *key-based*, i.e. with the join predicate `KeyValueMapper#apply(leftRecord.key, leftRecord.value) == rightRecord.key`.

                * 这个连接是间接*基于键*的，即连接条件为`KeyValueMapper#apply(leftRecord.key, leftRecord.value) == rightRecord.key`。

                * The join will be triggered under the conditions listed below whenever new input is received. When it is triggered, the user-supplied `ValueJoiner` will be called to produce join output records.

                * 每当收到新的输入时，连接将在下面列出的条件下触发。当它被触发时，用户提供的`ValueJoiner`将被调用以产生连接输出消息。

                    * Only input records for the left side (stream) trigger the join. Input records for the right side (table) update only the internal right-side join state.

                    * 只有左侧（流）的输入消息触发连接操作。右侧（表）的输入消息仅更新内部右侧连接状态。
                
                    * Input records for the stream with a `null` key or a `null` value are ignored and do not trigger the join.

                    * 具有`null`键或`null`值的流的输入消息将被忽略，并且不会触发连接。
                
                    * Input records for the table with a `null` value are interpreted as *tombstones*, which indicate the deletion of a record key from the table. Tombstones do not trigger the join.

                    * 具有`null`值的表的输入消息被解释为*删除逻辑*（tombstones），其指示从表中删除消息键。删除逻辑不会触发连接操作。

                * For each input record on the left side that does not have any match on the right side, the `ValueJoiner` will be called with `ValueJoiner#apply(leftRecord.value, null)`.

                * 对于右侧没有任何匹配的在左侧输入的消息，`ValueJoiner`将调用`ValueJoiner#apply(leftRecord.value, null)`。

**Semantics of stream-table joins:** The join semantics are identical to [KStream-KTable joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins-kstream-ktable). The only difference is that, for KStream-GlobalKTable joins, the left input record is first “mapped” with a user-supplied ```KeyValueMapper``` into the table’s keyspace prior to the table lookup.

**stream-table连接的语义：** 连接语义与[KStream-KTable连接](dsl-api.md)相同。唯一的区别是，对于KStream-GlobalKTable连接，在表查找之前，左输入消息首先被用户提供的`KeyValueMapper`“map”到表的键空间中。

## Windowing

## 窗口化

Windowing lets you control how to group records that have the same key for stateful operations such as [aggregations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-aggregating) or [joins](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins) into so-called windows. Windows are tracked per record key.

通过窗口化操作，您可以控制如何将具有相同键的消息进行分组，这些消息用于有状态操作（如[聚合](dsl-api.md)或在所谓的窗口中[连接](dsl-api.md)）。窗口化操作按每个消息键进行追踪。

### Note

### 注意

A related operation is [grouping](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-transformations-stateless), which groups all records that have the same key to ensure that data is properly partitioned (“keyed”) for subsequent operations. Once grouped, windowing allows you to further sub-group the records of a key.

相关的操作是[分组](dsl-api.md)，将具有相同键的所有消息分组以确保数据被正确分区（“keyed”）来用于后续操作。一旦分组以后，窗口允许您进一步分组一个键的消息。

For example, in join operations, a windowing state store is used to store all the records received so far within the defined window boundary. In aggregating operations, a windowing state store is used to store the latest aggregation results per window. Old records in the state store are purged after the specified [window retention period](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-windowing). Kafka Streams guarantees to keep a window for at least this specified time; the default value is one day and can be changed via ```Windows#until()``` and ```SessionWindows#until()```.

例如，在连接操作中，窗口状态存储器用于存储到当前为止能接受到的所有在定义的窗口边界内的消息。在聚合操作中，窗口状态存储器用于存储每个窗口的最新聚合结果。在指定的[窗口保留期限](../concepts.md)后清除状态存储器中的旧消息。Kafka Streams保证至少在这个特定时间内保持一个窗口；默认值是一天，可以通过`Windows#until()`和`SessionWindows#until()`来更改。

The DSL supports the following types of windows:

DSL支持以下类型的窗口：

Window name | Behavior | Short description
--- | --- | ---
[Tumbling time window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-tumbling) | 	Time-based | Fixed-size, non-overlapping, gap-less windows
[Hopping time window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-hopping) | 	Time-based | Fixed-size, overlapping windows
[Sliding time window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-sliding) | Time-based | Fixed-size, overlapping windows that work on differences between record timestamps
[Session window](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#windowing-session) | 	Session-based	| Dynamically-sized, non-overlapping, data-driven windows

窗口名称 | 行为 | 简述
--- | --- | ---
[翻转时间窗口](dsl-api.md) | 基于时间的 | 固定大小，不重叠，无间隙的窗口
[跳跃时间窗口](dsl-api.md) | 基于时间的 | 固定大小，重叠的窗口
[滑动时间窗口](dsl-api.md) | 基于时间的 | 固定大小的重叠窗口，作用于消息时间戳之间的不同之处
[会话窗口](dsl-api.md) | 基于会话的 | 动态大小的，不重叠的数据驱动窗口

### Tumbling time windows

### 翻转时间窗口

Tumbling time windows are a special case of hopping time windows and, like the latter, are windows based on time intervals. They model fixed-size, non-overlapping, gap-less windows. A tumbling window is defined by a single property: the window’s size. A tumbling window is a hopping window whose window size is equal to its advance interval. Since tumbling windows never overlap, a data record will belong to one and only one window.

翻转时间窗口是跳跃时间窗口的特例，并且像后者一样，是基于时间间隔的窗口。他们基于固定大小，不重叠，无间隙的窗口。翻转窗口由单个属性定义：窗口的*大小*。翻转窗口是窗口大小等于其提前间隔的跳跃窗口。由于翻转窗口不会重叠，因此消息数据将属于一个且仅属于一个窗口。

![](../../../imgs/streams-time-windows-tumbling.png)

This diagram shows windowing a stream of data records with tumbling windows. Windows do not overlap because, by definition, the advance interval is identical to the window size. In this diagram the time numbers represent minutes; e.g. t=5 means “at the five-minute mark”. In reality, the unit of time in Kafka Streams is milliseconds, which means the time numbers would need to be multiplied with 60 * 1,000 to convert from minutes to milliseconds (e.g. t=5 would become t=300,000).

此图显示了使用翻转窗口获取消息数据流的过程。窗口不重叠，因为根据定义，提前间隔与窗口大小相同。在此图中，时间单位是分钟；例如 t = 5意味着“在五分钟时”。实际上，Kafka Streams中的时间单位是毫秒，这意味着时间数字需要乘以60 * 1,000以将分钟转换为毫秒（例如，t = 5将变为t = 300,000）。

Tumbling time windows are *aligned to the epoch*, with the lower interval bound being inclusive and the upper bound being exclusive. “Aligned to the epoch” means that the first window starts at timestamp zero. For example, tumbling windows with a size of 5000ms have predictable window boundaries ```[0;5000),[5000;10000),...``` — and **not** ```[1000;6000),[6000;11000),...``` or even something “random” like ```[1452;6452),[6452;11452),...```.

翻转时间窗口与现实时间对齐，下限是包括在内的，而上限是不包括的。“与现实时间对齐”意味着第一个窗口从时间戳零开始。例如，大小为5000毫米的翻转窗口具有可预测的窗口边界`[0;5000),[5000;10000)...`而不是`[1000;6000),[6000;11000),...`，更有甚者，也不是某种 “随机”的间隔，如`[1452;6452),[6452;11452),...`。

The following code defines a tumbling window with a size of 5 minutes:

以下代码定义了大小为5分钟的滚动窗口：

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

### 跳跃时间窗口

Hopping time windows are windows based on time intervals. They model fixed-sized, (possibly) overlapping windows. A hopping window is defined by two properties: the window’s *size* and its *advance interval* (aka “hop”). The advance interval specifies by how much a window moves forward relative to the previous one. For example, you can configure a hopping window with a size 5 minutes and an advance interval of 1 minute. Since hopping windows can overlap – and in general they do – a data record may belong to more than one such windows.

跳跃时间窗口是基于时间间隔的窗口操作。他们基于固定大小，（可能）重叠的窗口。跳跃窗口由两个属性定义：窗口的大小和提前间隔（又称“跳跃”）。提前间隔指定窗口相对于前一个窗口向前移动多少。例如，您可以配置大小为5分钟和提前间隔为1分钟的跳跃窗口。由于跳跃窗口可能重叠 - 并且通常来说它们都会这样 - 消息数据可能属于多个这样的窗口。

### Note

### 注意

**Hopping windows vs. sliding windows:** Hopping windows are sometimes called “sliding windows” in other stream processing tools. Kafka Streams follows the terminology in academic literature, where the semantics of sliding windows are different to those of hopping windows.

**跳跃窗口与滑动窗口：** 跳跃窗口在其他流处理工具中有时称为“滑动窗口”。Kafka Streams遵循学术文献中的术语，其中滑动窗口的语义与跳跃窗口的语义不同。

The following code defines a hopping window with a size of 5 minutes and an advance interval of 1 minute:

以下代码定义了一个大小为5分钟，提前间隔为1分钟的跳跃窗口：

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

此图显示使用跳跃窗口来获取消息数据流。在此图中，时间单位是分钟；例如t = 5意味着“在第五分钟时”。实际上，Kafka Streams中的时间单位是毫秒，这意味着这个时间的数字需要乘以60 * 1,000以将分钟转换为毫秒（例如，t = 5将变为t = 300,000）。

Hopping time windows are aligned to the epoch, with the lower interval bound being inclusive and the upper bound being exclusive. “Aligned to the epoch” means that the first window starts at timestamp zero. For example, hopping windows with a size of 5000ms and an advance interval (“hop”) of 3000ms have predictable window boundaries ```[0;5000),[3000;8000),...``` — and **not** ```[1000;6000),[4000;9000),...``` or even something “random” like ```[1452;6452),[4452;9452),...```.

跳跃时间窗口与现实时间对齐，下限是包括在内的，而上限是不包括的。“与现实时间对齐”意味着第一个窗口从时间戳零开始。例如，大小为5000ms的跳跃窗口和3000ms的提前间隔（“跳跃”）具有可预测的窗口边界`[0;5000),[3000;8000)...`而不是`[1000;6000),[4000;9000)...`，更有甚者，也**不是**像`[1452;6452),[4452;9452),...`的时间范围。

Unlike non-windowed aggregates that we have seen previously, windowed aggregates return a windowed KTable whose keys type is ```Windowed<K>```. This is to differentiate aggregate values with the same key from different windows. The corresponding window instance and the embedded key can be retrieved as ```Windowed#window()``` and ```Windowed#key()```, respectively.

与我们之前看到的非窗口形式的聚合不同，窗口形式的聚合返回一个窗口化的KTable，其键类型为`Windowed <K>`。这是为了区分不同窗口中的相同键的聚合值。相应的窗口实例和内嵌的键可分别作为`Windowed#window()`和`Windowed#key()`来获取。

### Sliding time windows

### 滑动时间窗口

Sliding windows are actually quite different from hopping and tumbling windows. In Kafka Streams, sliding windows are used only for [join operations](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-joins), and can be specified through the ```JoinWindows``` class.

滑动窗口、跳跃窗口和翻转窗口实际上完全不同。在Kafka Streams中，滑动窗口仅用于[连接操作](dsl-api.md)，并且可以通过`JoinWindows`类来指定。

A sliding window models a fixed-size window that slides continuously over the time axis; here, two data records are said to be included in the same window if (in the case of symmetric windows) the difference of their timestamps is within the window size. Thus, sliding windows are not aligned to the epoch, but to the data record timestamps. In contrast to hopping and tumbling windows, the lower and upper window time interval bounds of sliding windows are *both inclusive*.

滑动窗口模拟在时间轴上连续滑动的固定大小的窗口；在这里，如果（在对称窗口的情况下）它们的时间戳的差值在窗口大小内，则说这两条消息数据被包含在相同的窗口中。因此，滑动窗口不与现实时间对齐，而是与消息数据的时间戳对齐。与跳跃和翻转窗口相比，滑动窗口的下窗口和上窗口之间的时间间隔*都包括在内*。

### Session Windows

### 会话窗口

Session windows are used to aggregate key-based events into so-called sessions, the process of which is referred to as sessionization. Sessions represent a **period of activity** separated by a defined **gap of inactivity** (or “idleness”). Any events processed that fall within the inactivity gap of any existing sessions are merged into the existing sessions. If an event falls outside of the session gap, then a new session will be created.

会话窗口用于将基于键的事件聚合到所谓的会话中，该处理过程称为会话(sessionization)。会话表示被一个确定的**不活动（或“闲置”）的间隔**分开的**一段时间的活动**。任何处于任何现有会话的不活动间隔内的事件都会合并到现有会话中。如果一个事件超出了会话间隔，那么将创建一个新的会话。

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

* **Transformation**

    * **Process**

        * KStream -> void

	        * **Description**

                **Terminal operation.** Applies a `Processor` to each record. `process()` allows you to leverage the [Processor API](http://kafka.apache.org/11/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api) from the DSL. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#process-org.apache.kafka.streams.processor.ProcessorSupplier-java.lang.String...-))

                **终端操作。** 将`处理器`应用于每条消息。`process()`允许您利用DSL中的[处理器API](processor-api.md)。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#process-org.apache.kafka.streams.processor.ProcessorSupplier-java.lang.String...-)）

                This is essentially equivalent to adding the `Processor` via `Topology#addProcessor()` to your [processor topology](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-processor-topology).

                这基本等同于通过`Topology#addProcessor()`将`处理器`添加到[处理器拓扑](../concepts.md)。

                An example is available in the [javadocs](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#process-org.apache.kafka.streams.processor.ProcessorSupplier-java.lang.String...-).

                [javadoc](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#process-org.apache.kafka.streams.processor.ProcessorSupplier-java.lang.String...-)中提供了一个示例。

    * **Transform**

        * KStream -> KStream

	        * **Description**

                Applies a `Transformer` to each record. `transform()` allows you to leverage the [Processor API](http://kafka.apache.org/11/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api) from the DSL. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transform-org.apache.kafka.streams.kstream.TransformerSupplier-java.lang.String...-))

                将`Transformer`应用于每条消息。`transform()`允许您利用DSL中的[处理器API](processor-api.md)。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transform-org.apache.kafka.streams.kstream.TransformerSupplier-java.lang.String...-)）

                Each input record is transformed into zero, one, or more output records (similar to the stateless `flatMap`). The `Transformer` must return `null` for zero output. You can modify the record’s key and value, including their types.

                每个输入消息都被转换为零个，一个或多个输出消息（类似于无状态的`flatMap`）。`Transformer`必须为零输出返回`null`值。您可以修改消息的键和值，包括它们的类型。

                **Marks the stream for data re-partitioning:** Applying a grouping or a join after `transform` will result in re-partitioning of the records. If possible use `transformValues` instead, which will not cause data re-partitioning.

                **标记数据流重分区：** 在`transform`之后应用分组或连接操作将导致消息的重分区。如果可能，请使用`transformValues`，这不会导致数据重分区。

                `transform` is essentially equivalent to adding the `Transformer` via `Topology#addProcessor()` to your [processor topology](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-processor-topology).

                操作`transform`本质上等同于通过`Topology#addProcessor()`将`Transformer`添加到[处理器拓扑](../concepts.md)中。

                An example is available in the [javadocs](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transform-org.apache.kafka.streams.kstream.TransformerSupplier-java.lang.String...-).

                [javadoc](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transform-org.apache.kafka.streams.kstream.TransformerSupplier-java.lang.String...-)中提供了一个示例。

    * **Transform (values only)**

        * KStream -> KStream

	        * **Description**

                Applies a `ValueTransformer` to each record, while retaining the key of the original record. `transformValues()` allows you to leverage the [Processor API](http://kafka.apache.org/11/documentation/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api) from the DSL. ([details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transformValues-org.apache.kafka.streams.kstream.ValueTransformerSupplier-java.lang.String...-))

                将`ValueTransformer`应用于每条消息，同时保留原始消息的键。`transformValues()`允许您利用DSL中的[处理器API](processor-api.md)。（[细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transformValues-org.apache.kafka.streams.kstream.ValueTransformerSupplier-java.lang.String...-)）

                Each input record is transformed into exactly one output record (zero output records or multiple output records are not possible). The `ValueTransformer` may return `null` as the new value for a record.

                每个输入消息都被转换为一个输出消息（零个输出消息或多个输出消息是不可能的）。`ValueTransformer`可能会返回`null`作为消息的新值。

                `transformValues` is preferable to `transform` because it will not cause data re-partitioning.

                `transformValues`优于`transform`，因为它不会导致数据重分区。

                `transformValues` is essentially equivalent to adding the `ValueTransformer` via `Topology#addProcessor()` to your [processor topology](http://kafka.apache.org/11/documentation/streams/concepts.html#streams-concepts-processor-topology).

                `transformValues`实质上等价于通过`Topology#addProcessor()`将`ValueTransformer`添加到[处理器拓扑](../concepts.md)中。

                An example is available in the [javadocs](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transformValues-org.apache.kafka.streams.kstream.ValueTransformerSupplier-java.lang.String...-).

                [javadoc](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#transformValues-org.apache.kafka.streams.kstream.ValueTransformerSupplier-java.lang.String...-)中提供了一个示例。

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

```java
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
```

## WRITING STREAMS BACK TO KAFKA

## 把流写回到Kafka

Any streams and tables may be (continuously) written back to a Kafka topic. As we will describe in more detail below, the output data might be re-partitioned on its way to Kafka, depending on the situation.

任何流和表都可以（连续地）写回Kafka主题。正如我们下面将要详细描述的那样，根据具体情况，输出数据可能会在前往Kafka的途中重新分区。

* **Writing to Kafka**

    * **To**

        * KStream -> void

	        * **Description**

                **Terminal operation.** Write the records to a Kafka topic. ([KStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#to(java.lang.String)))

                **终端操作。** 将消息写入Kafka主题。（[KStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#to(java.lang.String))）

                When to provide serdes explicitly:

                何时明确提供serdes：

                * If you do not specify SerDes explicitly, the default SerDes from the [configuration](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-configuration) are used.

                * 如果您没有明确指定SerDes，则使用[配置](config-streams.md)中的默认SerDes。

                * You **must specify SerDes explicitly** via the `Produced` class if the key and/or value types of the `KStream` do not match the configured default SerDes.

                * 如果`KStream`的键和、或值的类型与配置的默认SerDes不匹配，则**必须通过`Produced`类明确指定SerDes**。

                * See [Data Types and Serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes) for information about configuring default SerDes, available SerDes, and implementing your own custom SerDes.

                有关配置默认SerDes，可用SerDes和实现自己的自定义SerDes的信息，请参阅[数据类型和序列化](datatypes.md)。

                A variant of `to` exists that enables you to specify how the data is produced by using a `Produced` instance to specify, for example, a `StreamPartitioner` that gives you control over how output records are distributed across the partitions of the output topic.

                `to`的变体，使您可以指定数据的生成方式，方法是使用`Produced`实例指定例如`StreamPartitioner`，以便控制输出消息在输出主题分区间的分布方式。

                ```java
                KStream<String, Long> stream = ...;
                KTable<String, Long> table = ...;


                // Write the stream to the output topic, using the configured default key
                // and value serdes of your `StreamsConfig`.

                // 使用配置的默认键将流写入输出主题
                // 和你的`StreamsConfig`的值serdes。
                stream.to("my-stream-output-topic");

                // Same for table
                table.to("my-table-output-topic");

                // Write the stream to the output topic, using explicit key and value serdes,
                // (thus overriding the defaults of your `StreamsConfig`).

                // 使用明确的键和值serdes将流写入输出主题（因此覆盖“StreamsConfig”的默认值）。
                stream.to("my-stream-output-topic", Produced.with(Serdes.String(), Serdes.Long());
                ```

                **Causes data re-partitioning if any of the following conditions is true:**

                **如果满足以下任一条件，则导致数据重分区：**

                1. If the output topic has a different number of partitions than the stream/table.

                1. 如果输出主题具有与流或表不同的分区数量。
                
                2. If the `KStream` was marked for re-partitioning.
                
                2. 如果`KStream`被标记为重分区。

                3. If you provide a custom `StreamPartitioner` to explicitly control how to distribute the output records across the partitions of the output topic.

                3.  如果您提供自定义`StreamPartitioner`以明确控制如何在输出主题的分区之间发布输出消息。
                
                4. If the key of an output record is `null`.

                4. 如果输出消息的键为`null`。

    * **Through**

        * KStream -> KStream

        * KTable -> KTable

	        * **Description**

                Write the records to a Kafka topic and create a new stream/table from that topic. Essentially a shorthand for `KStream#to()` followed by `StreamsBuilder#stream()`, same for tables. ([KStream details](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#through(java.lang.String)))

                将消息写入Kafka主题，并根据该主题创建新的流/表。对于流，基本上是`KStream#to()`后跟`StreamsBuilder#stream()`的简写，对于表也类似。（[KStream细节](http://kafka.apache.org/11/javadoc/org/apache/kafka/streams/kstream/KStream.html#through(java.lang.String))）

                When to provide SerDes explicitly:

                何时明确提供SerDes：

                * If you do not specify SerDes explicitly, the default SerDes from the [configuration](http://kafka.apache.org/11/documentation/streams/developer-guide/config-streams.html#streams-developer-guide-configuration) are used.

                * 如果您没有明确指定SerDes，则使用[配置](config-streams.md)中的默认SerDes。
                
                * You **must specify SerDes explicitly** if the key and/or value types of the `KStream` or `KTable` do not match the configured default SerDes.

                * 如果`KStream`或`KTable`的键和、或值的类型与配置中默认SerDes不匹配，则**必须明确指定SerDes**。
                
                * See [Data Types and Serialization](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#streams-developer-guide-serdes) for information about configuring default SerDes, available SerDes, and implementing your own custom SerDes.

                * 有关配置默认SerDes，可用SerDes和实现自己的自定义SerDes的信息，请参阅[数据类型和序列化](datatypes.md)。

                A variant of `through` exists that enables you to specify how the data is produced by using a `Produced` instance to specify, for example, a `StreamPartitioner` that gives you control over how output records are distributed across the partitions of the output topic.
        
                `through`的一个变体，它使您能够指定如何使用`Produced`实例来指定数据，例如指定一个`StreamPartitioner`，以便控制输出消息在输出主题的分区间的分布方式。

                ```java
                StreamsBuilder builder = ...;
                KStream<String, Long> stream = ...;
                KTable<String, Long> table = ...;

                // Variant 1: Imagine that your application needs to continue reading and processing
                // the records after they have been written to a topic via ``to()``.  Here, one option
                // is to write to an output topic, then read from the same topic by constructing a
                // new stream from it, and then begin processing it (here: via `map`, for example).

                // 变种1：假设您的应用程序在通过``to()``将消息写入主题后需要继续读取和处理消息。 这里有一个选项
                // 是写入输出主题，然后构造一个从同一主题读取
                // 的新流，来开始处理它（例如：通过`map`）。
                stream.to("my-stream-output-topic");
                KStream<String, Long> newStream = builder.stream("my-stream-output-topic").map(...);

                // Variant 2 (better): Since the above is a common pattern, the DSL provides the
                // convenience method ``through`` that is equivalent to the code above.
                // Note that you may need to specify key and value serdes explicitly, which is
                // not shown in this simple example.

                // 变体2（更优）：由于上述是一种常见模式，因此DSL提供了
                // 相当于上面代码的便捷方法``through``。
                // 请注意，您可能需要明确指定在这个简单的例子中没有显示的键和值的serdes
                KStream<String, Long> newStream = stream.through("user-clicks-topic").map(...);

                // ``through`` is also available for tables
                // ``through``对表也起作用
                KTable<String, Long> newTable = table.through("my-table-output-topic").map(...);
                ```

                **Causes data re-partitioning if any of the following conditions is true:**

                **如果满足以下任一条件，则导致数据重分区：**

                1. If the output topic has a different number of partitions than the stream/table.

                1. 如果输出主题具有与流/表不同的分区数量。

                2. If the `KStream` was marked for re-partitioning.

                2. 如果`KStream`被标记为重分区。
                
                3. If you provide a custom `StreamPartitioner` to explicitly control how to distribute the output records across the partitions of the output topic.
                
                3. 如果您提供自定义`StreamPartitioner`以明确控制如何在输出主题的分区之间发布输出消息。

                4. If the key of an output record is `null`.

                4. 如果输出消息的键为`null`。

### Note

### 注意

**When you want to write to systems other than Kafka:** Besides writing the data back to Kafka, you can also apply a [custom processor](http://kafka.apache.org/11/documentation/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-process) as a stream sink at the end of the processing to, for example, write to external databases. First, doing so is not a recommended pattern – we strongly suggest to use the [Kafka Connect API](http://kafka.apache.org/11/documentation/connect/index.html#kafka-connect) instead. However, if you do use such a sink processor, please be aware that it is now your responsibility to guarantee message delivery semantics when talking to such external systems (e.g., to retry on delivery failure or to prevent message duplication).

**当您要将消息写入其它系统而非Kafka时：**除了将数据写回Kafka之外，还可以在处理结束时将[定制处理器](dsl-api.md)作为流接收器应用于如外部数据库等。首先，这样做是不推荐的——我们强烈建议使用[Kafka Connect API](../../kafka_connect.md)。但是，如果您确实使用了这样的接收处理器，请注意，现在与此类外部系统交互时您将有责任去保证消息传递语义（例如，重试交付失败或防止消息重复）。