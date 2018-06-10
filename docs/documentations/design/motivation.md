# 4.1 Motivation

# 4.1 动机

We designed Kafka to be able to act as a unified platform for handling all the real-time data feeds [a large company might have](http://kafka.apache.org/documentation/#introduction). To do this we had to think through a fairly broad set of use cases.

我们设计的Kafka可以作为一个统一的平台来处理[一家大型公司可能拥有](../../introduction.md)的所有实时数据。为了做到这一点，我们必须考虑一系列相当广泛的用途。

It would have to have high-throughput to support high volume event streams such as real-time log aggregation.

它必须具有高吞吐量，以支持大容量的事件流，如实时日志聚合。

It would need to deal gracefully with large data backlogs to be able to support periodic data loads from offline systems.

它需要优雅地处理大量的数据积压(backlogs)，以支持定期地装载来自离线系统的数据。

It also meant the system would have to handle low-latency delivery to handle more traditional messaging use-cases.

这也意味着系统必须处理低延迟交付，以处理更传统的消息用例。

We wanted to support partitioned, distributed, real-time processing of these feeds to create new, derived feeds. This motivated our partitioning and consumer model.

我们希望能对这些数据种子(feeds)支持分区的、分布式的、实时的处理，以创造出新的、衍生的数据种子。这促使我们构造了分区和消费者模式。

Finally in cases where the stream is fed into other data systems for serving, we knew the system would have to be able to guarantee fault-tolerance in the presence of machine failures.

最后，对于数据流被送入到其他数据系统中去提供服务的这些场景，我们的系统必须能够在机器故障时保证容错能力。

Supporting these uses led us to a design with a number of unique elements, more akin to a database log than a traditional messaging system. We will outline some elements of the design in the following sections.

为了支持这些用途，我们设计了一些独特的元素，比起传统的消息系统，Kafka更像是一个数据库日志系统。我们将在后面的章节中介绍这些设计的元素。
