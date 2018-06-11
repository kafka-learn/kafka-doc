# Introduction

# 简介

The easiest way to write mission-critical real-time applications and microservices

Kafka Steam是编写关键任务的实时应用程序和微服务的最简单方法

Kafka Streams is a client library for building applications and microservices, where the input and output data are stored in Kafka clusters. It combines the simplicity of writing and deploying standard Java and Scala applications on the client side with the benefits of Kafka's server-side cluster technology.

Kafka Stream是用于构建应用程序和微服务的客户端库，输入和输出数据存储在Kafka集群中。它将在客户端部署标准Java和Scala应用程序的简单性与Kafka的服务器端集群技术的优点相结合。

## Why you'll love using Kafka Streams!

* Elastic, highly scalable, fault-tolerant

    可伸缩性，高度可扩展性，容错性

* Deploy to containers, VMs, bare metal, cloud

    能部署到容器，虚拟机，裸机，云

* Equally viable for small, medium, & large use cases

    对于小型、中型和大型用例都可行

* Fully integrated with Kafka security

    与Kafka安全整合

* Write standard Java applications

    能编写成标准的Java 应用

* Exactly-once processing semantics

    确切的一次处理语义

* No separate processing cluster required

    不需要单独的处理集群

* Develop on Mac, Linux, Windows

    能在Mac、Linux、Windows上进行开发

## Hello Kafka Streams

下面的代码示例实现了一个可伸缩的，高度可扩展的，容错的，有状态的WordCount应用程序，并且可以在大规模生产中运行

* Java 8+

```java
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.common.utils.Bytes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.kstream.Produced;
import org.apache.kafka.streams.state.KeyValueStore;

import java.util.Arrays;
import java.util.Properties;

public class WordCountApplication {

    public static void main(final String[] args) throws Exception {
        Properties config = new Properties();
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application");
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker1:9092");
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> textLines = builder.stream("TextLinesTopic");
        KTable<String, Long> wordCounts = textLines
            .flatMapValues(textLine -> Arrays.asList(textLine.toLowerCase().split("\\W+")))
            .groupBy((key, word) -> word)
            .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"));
        wordCounts.toStream().to("WordsWithCountsTopic", Produced.with(Serdes.String(), Serdes.Long()));

        KafkaStreams streams = new KafkaStreams(builder.build(), config);
        streams.start();
    }

}
```

* Java 7

```java
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.common.utils.Bytes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.ValueMapper;
import org.apache.kafka.streams.kstream.KeyValueMapper;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.kstream.Produced;
import org.apache.kafka.streams.state.KeyValueStore;

import java.util.Arrays;
import java.util.Properties;

public class WordCountApplication {

    public static void main(final String[] args) throws Exception {
        Properties config = new Properties();
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application");
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker1:9092");
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> textLines = builder.stream("TextLinesTopic");
        KTable<String, Long> wordCounts = textLines
            .flatMapValues(new ValueMapper<String, Iterable<String>>() {
                @Override
                public Iterable<String> apply(String textLine) {
                    return Arrays.asList(textLine.toLowerCase().split("\\W+"));
                }
            })
            .groupBy(new KeyValueMapper<String, String, String>() {
                @Override
                public String apply(String key, String word) {
                    return word;
                }
            })
            .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"));


        wordCounts.toStream().to("WordsWithCountsTopic", Produced.with(Serdes.String(), Serdes.Long()));

        KafkaStreams streams = new KafkaStreams(builder.build(), config);
        streams.start();
    }

}
```

* Scala

```scala
import java.lang.Long
import java.util.Properties
import java.util.concurrent.TimeUnit

import org.apache.kafka.common.serialization._
import org.apache.kafka.common.utils.Bytes
import org.apache.kafka.streams._
import org.apache.kafka.streams.kstream.{KStream, KTable, Materialized, Produced}
import org.apache.kafka.streams.state.KeyValueStore

import scala.collection.JavaConverters.asJavaIterableConverter

object WordCountApplication {

    def main(args: Array[String]) {
        val config: Properties = {
            val p = new Properties()
            p.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application")
            p.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker1:9092")
            p.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass)
            p.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass)
            p
        }

        val builder: StreamsBuilder = new StreamsBuilder()
        val textLines: KStream[String, String] = builder.stream("TextLinesTopic")
        val wordCounts: KTable[String, Long] = textLines
            .flatMapValues(textLine => textLine.toLowerCase.split("\\W+").toIterable.asJava)
            .groupBy((_, word) => word)
            .count(Materialized.as("counts-store").asInstanceOf[Materialized[String, Long, KeyValueStore[Bytes, Array[Byte]]]])
        wordCounts.toStream().to("WordsWithCountsTopic", Produced.`with`(Serdes.String(), Serdes.Long()))

        val streams: KafkaStreams = new KafkaStreams(builder.build(), config)
        streams.start()

        Runtime.getRuntime.addShutdownHook(new Thread(() => {
            streams.close(10, TimeUnit.SECONDS)
        }))
    }

}
```