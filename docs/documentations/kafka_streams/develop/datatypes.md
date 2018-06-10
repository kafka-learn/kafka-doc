# Data Types and Serialization

# 数据类型和序列化

Every Kafka Streams application must provide SerDes (Serializer/Deserializer) for the data types of record keys and record values (e.g. `java.lang.String`) to materialize the data when necessary. Operations that require such SerDes information include: `stream()`, `table()`, `to()`, `through()`, `groupByKey()`, `groupBy()`.

每个Kafka Streams应用程序都必须为消息的键和值（例如`java.lang.String`）的数据类型提供SerDes（序列化器（Serializer）/反序列化器（Deserializer）），以便在必要时物化数据。需要此类SerDes信息的操作包括：`stream()`，`table()`，`to()`，`through()`，`groupByKey()`，`groupBy()`。

You can provide SerDes by using either of these methods:

您可以使用以下任一方法提供SerDes：

* By setting default SerDes via a `StreamsConfig` instance.

* By specifying explicit SerDes when calling the appropriate API methods, thus overriding the defaults.


* 通过`StreamsConfig`实例设置默认的SerDes。

* 通过在调用适当的API时指定显式的SerDes，以覆盖默认值。

### Table of Contents

### 目录

* [Configuring SerDes](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#configuring-serdes)

* [配置SerDes](datatypes.md)

* [Overriding default SerDes](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#overriding-default-serdes)

* [覆盖默认的SerDes](datatypes.md)

* [Available SerDes](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#available-serdes)

* [可用的SerDes](datatypes.md)

    * [Primitive and basic types](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#primitive-and-basic-types)

    * [原始类型和基本类型](datatypes.md)

    * [Avro](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#avro)

    * [Avro类型](datatypes.md)

    * [JSON](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#json)

    * [JSON类型](datatypes.md)
    
    * [Further serdes](http://kafka.apache.org/11/documentation/streams/developer-guide/datatypes.html#further-serdes)

    * [更多serdes](datatypes.md)

## CONFIGURING SERDES

## 配置SerDes

SerDes specified in the Streams configuration via `StreamsConfig` are used as the default in your Kafka Streams application.

在Streams配置中通过`StreamsConfig`指定的SerDes被用作Kafka Streams应用程序的默认设置。

```java
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.StreamsConfig;

Properties settings = new Properties();
// Default serde for keys of data records (here: built-in serde for String type)
// 消息数据的键的默认serde（这里是：为String类型内置的serde）
settings.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
// Default serde for values of data records (here: built-in serde for Long type)
// 消息数据的值的默认serde（这里是：为Long类型内置的serde）
settings.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.Long().getClass().getName());

StreamsConfig config = new StreamsConfig(settings);
```

## OVERRIDING DEFAULT SERDES

## 覆盖默认的SerDes

You can also specify SerDes explicitly by passing them to the appropriate API methods, which overrides the default serde settings:

您也可以通过将SerDes传递给适当的API方法的途径来明确指定SerDes，这会覆盖默认的serde设置：

```java
import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serdes;

final Serde<String> stringSerde = Serdes.String();
final Serde<Long> longSerde = Serdes.Long();

// The stream userCountByRegion has type `String` for record keys (for region)
// and type `Long` for record values (for user counts).
// 流userCountByRegion对于消息键（对于区域）具有“String”类型，对于消息值（对于用户计// 数）具有类型“Long”。
KStream<String, Long> userCountByRegion = ...;
userCountByRegion.to("RegionCountsTopic", Produced.with(stringSerde, longSerde));
```

If you want to override serdes selectively, i.e., keep the defaults for some fields, then don’t specify the serde whenever you want to leverage the default settings:

如果您想要选择性地覆盖serdes，即保留某些字段的默认值，那么只要您想要使用默认设置就不要自行指定serde：

```java
import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serdes;

// Use the default serializer for record keys (here: region as String) by not specifying the key serde,
// but override the default serializer for record values (here: userCount as Long).
/** 不去指定键的serde，而使用消息键的默认序列化程序（在这里：region是String类型），但是覆盖消息值的默认序列化程序（这里：userCount是Long类型）。*/
final Serde<Long> longSerde = Serdes.Long();
KStream<String, Long> userCountByRegion = ...;
userCountByRegion.to("RegionCountsTopic", Produced.valueSerde(Serdes.Long()));
```

## AVAILABLE SERDES

## 可用的SerDes

## Primitive and basic types

## 原始类型和基本类型

Apache Kafka includes several built-in serde implementations for Java primitives and basic types such as `byte[]` in its `kafka-clients`Maven artifact:

Apache Kafka包含几个用于Java原始类型和基本类型的内置serde实现，例如`kafka-clients`的Maven artifact中的`byte[]`：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

This artifact provides the following serde implementations under the package [org.apache.kafka.common.serialization](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization), which you can leverage when e.g., defining default serializers in your Streams configuration.

此artifact在[org.apache.kafka.common.serialization](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization)包下提供了以下serde实现，您可以在您的Streams配置中定义默认序列化器时使用它们。

Data type | Serde
--- | ---
byte[] | `Serdes.ByteArray()`, `Serdes.Bytes()` (see tip below)
ByteBuffer | `Serdes.ByteBuffer()`
Double | `Serdes.Double()`
Integer | `Serdes.Integer()`
Long | `Serdes.Long()`
String | `Serdes.String()`

数据类型 | Serde
--- | ---
byte[] | `Serdes.ByteArray()`, `Serdes.Bytes()` (参见下方提示)
ByteBuffer | `Serdes.ByteBuffer()`
Double | `Serdes.Double()`
Integer | `Serdes.Integer()`
Long | `Serdes.Long()`
String | `Serdes.String()`

### Tip

### 提示

[Bytes](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/utils/Bytes.java) is a wrapper for Java’s `byte[]` (byte array) that supports proper equality and ordering semantics. You may want to consider using `Bytes` instead of `byte[]` in your applications.

[Bytes](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/utils/Bytes.java)是Java的`byte[]`（字节数组）的封装，它支持正确的判断相等和排序语义。您可能需要考虑在您的应用程序中使用`Bytes`而不是`byte[]`。

## JSON

The code examples of Kafka Streams also include a basic serde implementation for JSON:

Kafka Streams的代码示例还包含JSON的基本serde实现：

* [JsonPOJOSerializer](https://github.com/apache/kafka/blob/1.0/streams/examples/src/main/java/org/apache/kafka/streams/examples/pageview/JsonPOJOSerializer.java)

* [JsonPOJODeserializer](https://github.com/apache/kafka/blob/1.0/streams/examples/src/main/java/org/apache/kafka/streams/examples/pageview/JsonPOJODeserializer.java)

You can construct a unified JSON serde from the `JsonPOJOSerializer` and `JsonPOJODeserializer` via `Serdes.serdeFrom(<serializerInstance>, <deserializerInstance>)`. The [PageViewTypedDemo](https://github.com/apache/kafka/blob/1.0/streams/examples/src/main/java/org/apache/kafka/streams/examples/pageview/PageViewTypedDemo.java) example demonstrates how to use this JSON serde.

您可以通过`Serdes.serdeFrom(<serializerInstance>, <deserializerInstance>)`从`JsonPOJOSerializer`和`JsonPOJODeserializer`构建一个统一的JSON serde。[PageViewTypedDemo](https://github.com/apache/kafka/blob/1.0/streams/examples/src/main/java/org/apache/kafka/streams/examples/pageview/PageViewTypedDemo.java)示例将演示如何使用此JSON serde。

## IMPLEMENTING CUSTOM SERDES

## 实现自定义的SERDES

If you need to implement custom SerDes, your best starting point is to take a look at the source code references of existing SerDes (see previous section). Typically, your workflow will be similar to:

如果您需要实现自定义的SerDes，最好先去查看现有的SerDes源代码（参见上一节）。通常，您的工作流程将类似于：

1. Write a serializer for your data type `T` by implementing [org.apache.kafka.common.serialization.Serializer](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Serializer.java).

1. 通过实现[org.apache.kafka.common.serialization.Serializer](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Serializer.java)为您的数据类型`T`编写序列化程序。

2. Write a deserializer for `T` by implementing [org.apache.kafka.common.serialization.Deserializer](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Deserializer.java).

2. 通过实现[org.apache.kafka.common.serialization.Deserializer](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Deserializer.java)为`T`写一个反序列化器。

3. Write a serde for `T` by implementing [org.apache.kafka.common.serialization.Serde](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Serde.java), which you either do manually (see existing SerDes in the previous section) or by leveraging helper functions in [Serdes](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Serdes.java) such as `Serdes.serdeFrom(Serializer<T>, Deserializer<T>)`.

3. 通过实现[org.apache.kafka.common.serialization.Serde](https://github.com/apache/kafka/blob/1.0/clients/src/main/java/org/apache/kafka/common/serialization/Serde.java)来为`T`写一个serde，您可以手动（参见上一节中的现有SerDes）或者利用Serdes中的辅助方法，例如`Serdes.serdeFrom(Serializer<T>, Deserializer<T>)`。
