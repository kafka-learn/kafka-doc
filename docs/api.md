
# Kafka includes five core apis:

* The Producer API allows applications to send streams of data to topics in the Kafka cluster.

* The Consumer API allows applications to read streams of data from topics in the Kafka cluster.

* The Streams API allows transforming streams of data from input topics to output topics.

* The Connect API allows implementing connectors that continually pull from some source system or application into Kafka or push from Kafka into some sink system or application.

* The AdminClient API allows managing and inspecting topics, brokers, and other Kafka objects.

Kafka exposes all its functionality over a language independent protocol which has clients available in many programming languages. However only the Java clients are maintained as part of the main Kafka project, the others are available as independent open source projects. A list of non-Java clients is available [here](https://cwiki.apache.org/confluence/display/KAFKA/Clients).

## Producer API
The Producer API allows applications to send streams of data to topics in the Kafka cluster.
Examples showing how to use the producer are given in the [javadocs](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html).

To use the producer, you can use the following maven dependency:
```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

## Consumer API
The Consumer API allows applications to read streams of data from topics in the Kafka cluster.
Examples showing how to use the consumer are given in the [javadocs](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html).

To use the consumer, you can use the following maven dependency:

```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

## Streams API
The Streams API allows transforming streams of data from input topics to output topics.
Examples showing how to use this library are given in the [javadocs](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/streams/KafkaStreams.html)

Additional documentation on using the Streams API is available [here](TODO).

To use Kafka Streams you can use the following maven dependency:

```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>1.1.0</version>
</dependency>
```

## Connect API
The Connect API allows implementing connectors that continually pull from some source data system into Kafka or push from Kafka into some sink data system.
Many users of Connect won't need to use this API directly, though, they can use pre-built connectors without needing to write any code. Additional information on using Connect is available [here](TODO).

Those who want to implement custom connectors can see the [javadoc](http://kafka.apache.org/11/javadoc/overview-summary.html).

## AdminClient API
The AdminClient API supports managing and inspecting topics, brokers, acls, and other Kafka objects.
To use the AdminClient API, add the following Maven dependency:

```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>1.1.0</version>
</dependency>
```

For more information about the AdminClient APIs, see the [javadoc](http://kafka.apache.org/11/javadoc/index.html?org/apache/kafka/clients/admin/AdminClient.html).

