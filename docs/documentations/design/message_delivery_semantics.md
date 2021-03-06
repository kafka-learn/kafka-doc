# 4.6 Message Delivery Semantics

# 4.6 消息传递语义

Now that we understand a little about how producers and consumers work, let's discuss the semantic guarantees Kafka provides between producer and consumer. Clearly there are multiple possible message delivery guarantees that could be provided:

现在我们对生产者和消费者的工作方式有了一定的了解，让我们来讨论Kafka在生产者和消费者之间提供的语义保证。显然，可以提供多种可能的消息传递保证：

* _At most once_—Messages may be lost but are never redelivered.
* _At least once_—Messages are never lost but may be redelivered.
* _Exactly once_—this is what people actually want, each message is delivered once and only once.


* _最多一次(at most once)_ - 消息可能会丢失，但不会重新发送。
* _至少一次(at least once)_ - 消息永远不会丢失，但可能要重新发送。
* _恰好一次(Exactly once)_ - 这是人们真正想要的，每条消息只传递一次。

It's worth noting that this breaks down into two problems: the durability guarantees for publishing a message and the guarantees when consuming a message.

值得注意的是，这可以分成两个问题：发布消息的持久性保证和消费消息时的保证。

Many systems claim to provide "exactly once" delivery semantics, but it is important to read the fine print, most of these claims are misleading (i.e. they don't translate to the case where consumers or producers can fail, cases where there are multiple consumer processes, or cases where data written to disk can be lost).

很多系统声称提供“恰好一次(exactly once)”的交付语义，但阅读这些细则是非常重要的，这些声明大多是误导性的（即它们不转化为消费者或生产者失败的情况，也不转化为多消费者进程情况和写入磁盘时数据丢失的情况）。

Kafka's semantics are straight-forward. When publishing a message we have a notion of the message being "committed" to the log. Once a published message is committed it will not be lost as long as one broker that replicates the partition to which this message was written remains "alive". The definition of committed message, alive partition as well as a description of which types of failures we attempt to handle will be described in more detail in the next section. For now let's assume a perfect, lossless broker and try to understand the guarantees to the producer and consumer. If a producer attempts to publish a message and experiences a network error it cannot be sure if this error happened before or after the message was committed. This is similar to the semantics of inserting into a database table with an autogenerated key.

Kafka的语义是清晰直接的。在发布消息时，有个消息的概念会被“提交(committed)”到日志中。一旦发布的消息被提交，只要有一个备份了此消息分区且仍然“活着”的代理，该消息就不会丢失。已提交消息的定义，活动分区以及我们会尝试处理哪些类型的故障的描述将在下一节中更详细地描述。现在让我们假设一个完好无损的代理，并试图去理解生产者和消费者的保证。如果生产者试图发布消息并遇到网络错误，则无法确定此错误是在提交消息之前还是在之后发生的。这与使用自动生成的键插入到数据库表中的语义相似。

Prior to 0.11.0.0, if a producer failed to receive a response indicating that a message was committed, it had little choice but to resend the message. This provides at-least-once delivery semantics since the message may be written to the log again during resending if the original request had in fact succeeded. Since 0.11.0.0, the Kafka producer also supports an idempotent delivery option which guarantees that resending will not result in duplicate entries in the log. To achieve this, the broker assigns each producer an ID and deduplicates messages using a sequence number that is sent by the producer along with every message. Also beginning with 0.11.0.0, the producer supports the ability to send messages to multiple topic partitions using transaction-like semantics: i.e. either all messages are successfully written or none of them are. The main use case for this is exactly-once processing between Kafka topics (described below).

在0.11.0.0版本之前，如果生产者未能收到指示消息已提交的响应，则其只能重新发送消息。这提供了至少一次（at-least-once）传递语义，因为如果原始请求实际上已经成功，那么在重新发送期间可能再次将消息写入日志。从0.11.0.0版本开始，Kafka生产者还支持一个幂等传递选项，它保证重新发送消息时不会在日志中产生重复的条目。为了达到这个目的，生产者发送消息时，代理为每个生产者分配一个ID，并通过生产者发送的每个消息的序列号来消除重复消息。从0.11.0.0版本开始，生产者支持使用类似事务的语义将消息发送到多个主题分区：即，所有消息要么都被成功写入，要么都不成功。这个主要用例恰好出现在Kafka主题之间进行恰好一次（exactly-once）的处理（如下所述）。

Not all use cases require such strong guarantees. For uses which are latency sensitive we allow the producer to specify the durability level it desires. If the producer specifies that it wants to wait on the message being committed this can take on the order of 10 ms. However the producer can also specify that it wants to perform the send completely asynchronously or that it wants to wait only until the leader (but not necessarily the followers) have the message.

并非所有用例都需要这种强有力的保证。对于延迟敏感的使用场景，我们允许生产者指定其期望的持久性级别（durability level）。如果生产者指定了它想要等待提交的消息，则这可能需要10毫秒的量级。当然，生产者也可以指定它想要完全异步执行发送的消息，或者它只想等到leader（不一定是follower）有消息为止。

Now let's describe the semantics from the point-of-view of the consumer. All replicas have the exact same log with the same offsets. The consumer controls its position in this log. If the consumer never crashed it could just store this position in memory, but if the consumer fails and we want this topic partition to be taken over by another process the new process will need to choose an appropriate position from which to start processing. Let's say the consumer reads some messages -- it has several options for processing the messages and updating its position.

现在让我们从消费者的角度来描述语义。所有的副本具有相同的日志和相同的偏移量。消费者控制其在此日志中的位置。如果消费者从未崩溃掉，它可以将这个位置存储在内存中，但是如果消费者失败了，并且我们希望这个主题分区被另一个进程接管，那么新进程将需要选择一个合适的开始处理的位置。假设消费者已经读取了一些消息 - 这里有几个选项用于处理这些消息并更新消息的位置。

1. It can read the messages, then save its position in the log, and finally process the messages. In this case there is a possibility that the consumer process crashes after saving its position but before saving the output of its message processing. In this case the process that took over processing would start at the saved position even though a few messages prior to that position had not been processed. This corresponds to "at-most-once" semantics as in the case of a consumer failure messages may not be processed.

2. It can read the messages, process the messages, and finally save its position. In this case there is a possibility that the consumer process crashes after processing messages but before saving its position. In this case when the new process takes over the first few messages it receives will already have been processed. This corresponds to the "at-least-once" semantics in the case of consumer failure. In many cases messages have a primary key and so the updates are idempotent (receiving the same message twice just overwrites a record with another copy of itself).


1. 它可以读取消息，然后保存它在日志中的位置，最后再处理消息。在这种情况下，消费者进程可能会在保存其位置之后但在保存其消息处理输出之前崩溃掉。此种情况下，即使该保存的位置之前有一些消息尚未处理，接管处理的进程仍将从该保存的位置开始。这对应于“最多一次（at-most-once）”语义，例如在消费者失败的情况下，消息可能不会被处理。

2. 它可以读取消息，处理消息，最后再保存其位置。在这种情况下，消费者进程可能会在处理消息之后但在保存其位置之前崩溃掉。此种情况下，当新进程接管它接收到的前几条消息时，这几条消息已经处理完毕了。这对应于消费者失败情况下的“至少一次（at-least-once）”语义。在很多情况下，消息具有主键，因此更新是幂等的（接收相同的消息两次，却只用消息的另一个副本去覆盖）。

So what about exactly once semantics (i.e. the thing you actually want)? When consuming from a Kafka topic and producing to another topic (as in a [Kafka Streams](https://kafka.apache.org/documentation/streams/) application), we can leverage the new transactional producer capabilities in 0.11.0.0 that were mentioned above. The consumer's position is stored as a message in a topic, so we can write the offset to Kafka in the same transaction as the output topics receiving the processed data. If the transaction is aborted, the consumer's position will revert to its old value and the produced data on the output topics will not be visible to other consumers, depending on their "isolation level." In the default "read_uncommitted" isolation level, all messages are visible to consumers even if they were part of an aborted transaction, but in "read_committed," the consumer will only return messages from transactions which were committed (and any messages which were not part of a transaction).

所以，恰好一次（exactly once）语义是否是真正想要的呢？。当从Kafka的一个主题中消费消息并产生到另一个主题（如在[Kafka Stream](../kafka_streams/introduction.md)应用程序中）时，我们可以利用上面提到的0.11.0.0版本中的新事务生成功能。消费者消费的位置作为消息存储在主题中，然后我们就可以在同一个事务中向Kafka写入偏移量，就像输出主题接收到处理后的数据。如果事务中止，则消费者的位置将恢复到其旧值，并且根据其“隔离级别”，输出主题上的生成数据对其他消费者而言将不可见。在默认的“read_uncommitted”隔离级别中，即使消费者是中止事务的一部分，消费者对所有消息仍都是可见的，但在“read_committed”中，消费者只会从已提交的事务中返回消息（以及任何不属于事务的消息）。

When writing to an external system, the limitation is in the need to coordinate the consumer's position with what is actually stored as output. The classic way of achieving this would be to introduce a two-phase commit between the storage of the consumer position and the storage of the consumers output. But this can be handled more simply and generally by letting the consumer store its offset in the same place as its output. This is better because many of the output systems a consumer might want to write to will not support a two-phase commit. As an example of this, consider a [Kafka Connect](https://kafka.apache.org/documentation/#connect) connector which populates data in HDFS along with the offsets of the data it reads so that it is guaranteed that either data and offsets are both updated or neither is. We follow similar patterns for many other data systems which require these stronger semantics and for which the messages do not have a primary key to allow for deduplication.

在写入外部系统时，限制在于需要协调消费者的位置与输出的实际存储位置。实现这一目标的经典方法是在消费者位置的存储与消费者输出的存储之间引入两阶段提交。但是，这可以通过让消费者将其偏移量存储在与其输出相同的位置来简单地处理。这样做更好一些，因为消费者希望写入的很多输出系统可能不支持两阶段提交。例如，考虑一个[Kafka Connect](../kafka_connect.md)连接器，该连接器将HDFS中的数据与其读取的数据的偏移一起填充，以确保数据和偏移既可以更新，也可以不更新。对于需要这些更强大语义的其他数据系统，我们遵循类似模式，并且对于没有主键的消息，其也能被备份。

So effectively Kafka supports exactly-once delivery in [Kafka Streams](https://kafka.apache.org/documentation/streams/), and the transactional producer/consumer can be used generally to provide exactly-once delivery when transfering and processing data between Kafka topics. Exactly-once delivery for other destination systems generally requires cooperation with such systems, but Kafka provides the offset which makes implementing this feasible (see also [Kafka Connect](https://kafka.apache.org/documentation/#connect)). Otherwise, Kafka guarantees at-least-once delivery by default, and allows the user to implement at-most-once delivery by disabling retries on the producer and committing offsets in the consumer prior to processing a batch of messages.

因此，Kafka能有效的支持在[Kafka Stream](../kafka_streams/introduction.md)中进行恰好一次（exactly-once）传递，并且在传送和处理Kafka主题之间的数据时，事务性生产者/消费者通常可以用于提供恰好一次传送。对于其它目标系统的恰好一次交付通常需要与此类系统合作，但Kafka提供了实现这种可行的偏移量（另请参阅[Kafka Connect](../kafak_connect.md)）。否则，Kafka默认保证至少一次交付，并允许用户在处理一批消息之前禁用生产者的重试和消费者的偏移，从而实现最多一次的交付。