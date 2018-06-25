# 4.4 The Producer

# 4.4 生产者

## Load balancing

## 负载均衡

The producer sends data directly to the broker that is the leader for the partition without any intervening routing tier. To help the producer do this all Kafka nodes can answer a request for metadata about which servers are alive and where the leaders for the partitions of a topic are at any given time to allow the producer to appropriately direct its requests.

生产者将数据直接发送给作为分区leader的代理，而不需要任何中间路由层。为了让生产者做到这一点，所有的Kafka节点都可以响应关于那些服务器处于活动状态的元数据请求，以及主题分区的leader在任何给定时间的位置，以便生产者适当地指导其请求。

The client controls which partition it publishes messages to. This can be done at random, implementing a kind of random load balancing, or it can be done by some semantic partitioning function. We expose the interface for semantic partitioning by allowing the user to specify a key to partition by and using this to hash to a partition (there is also an option to override the partition function if need be). For example if the key chosen was a user id then all data for a given user would be sent to the same partition. This in turn will allow consumers to make locality assumptions about their consumption. This style of partitioning is explicitly designed to allow locality-sensitive processing in consumers.

客户端控制生产者将消息发布到哪个分区。这可以实现一种随机负载平衡随时完成，或者可以通过某种语义分区功能完成。我们公开了用于语义分区的接口，通过允许用户指定分区的key并使用它来散列分区（如果需要，还可以选择覆盖分区函数）。例如，如果选择的key是用户ID，那么给定用户的所有数据都将被发送到同一个分区。这也将允许消费者对他们的消费做出当地假设。这种风格明确的分区设计允许消费者进行区域敏感的处理。

## Asynchronous send

Batching is one of the big drivers of efficiency, and to enable batching the Kafka producer will attempt to accumulate data in memory and to send out larger batches in a single request. The batching can be configured to accumulate no more than a fixed number of messages and to wait no longer than some fixed latency bound (say 64k or 10 ms). This allows the accumulation of more bytes to send, and few larger I/O operations on the servers. This buffering is configurable and gives a mechanism to trade off a small amount of additional latency for better throughput.

批处理是效率的重要推动力之一，为了实现批量生产，Kafka生产者将尝试在内存中积累数据，并在单个请求中发送更大批量的数据。批处理可以配置为累积不超过固定数量的消息，并且不超过某个固定的延迟限制（比如64k或10ms）。这允许发送更多字节的累积，并且在服务器上进行一些较大的I/O操作。这种缓冲是可配置的，并提供了一种机制来折中少量额外的延迟以获得更好的吞吐量。

Details on [configuration](http://kafka.apache.org/documentation/#producerconfigs) and the [api](http://kafka.apache.org/082/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html) for the producer can be found elsewhere in the documentation.

详细的生产者[配置](../configs/producer.md)和生产者的[api](http://kafka.apache.org/082/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html)可以在文档的其它地方找到。