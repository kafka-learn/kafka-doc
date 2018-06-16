# 4.8 Log Compaction

# 4.8 日志压缩

Log compaction ensures that Kafka will always retain at least the last known value for each message key within the log of data for a single topic partition. It addresses use cases and scenarios such as restoring state after application crashes or system failure, or reloading caches after application restarts during operational maintenance. Let's dive into these use cases in more detail and then describe how compaction works.

日志压缩可确保Kafka始终为单个主题分区的数据日志中的每个消息的key保留至少是最后已知的值。它指明了用例和场景，例如在应用程序崩溃或系统故障后恢复状态，或者在运行维护期间重新启动应用程序后重新加载缓存。让我们更详细地介绍这些用例，然后描述压缩日志是如何工作的。

So far we have described only the simpler approach to data retention where old log data is discarded after a fixed period of time or when the log reaches some predetermined size. This works well for temporal event data such as logging where each record stands alone. However an important class of data streams are the log of changes to keyed, mutable data (for example, the changes to a database table).

到目前为止，我们只描述了简单的数据保留方法，其中旧日志数据在固定时间段后或日志达到某个预定大小时会被丢弃。这适用于时间事件数据，例如记录每个消息独立的地方。然而，重要的一类数据流是对键控，可变数据（例如对数据库表的更改）进行更改的日志。

Let's discuss a concrete example of such a stream. Say we have a topic containing user email addresses; every time a user updates their email address we send a message to this topic using their user id as the primary key. Now say we send the following messages over some time period for a user with id 123, each message corresponding to a change in email address (messages for other ids are omitted):

我们来讨论一个这样的流的具体例子。即假设我们有一个包含用户电子邮件地址的主题; 每次用户更新他们的电子邮件地址时，我们都会使用他们的用户标识作为主键向此主题发送消息。现在我们在id为123的用户的某个时间段内发送以下消息，每个消息对应于电子邮件地址的更改（其它id的消息都被省略了）：

```
123 => bill@microsoft.com
        .
        .
        .
123 => bill@gatesfoundation.org
        .
        .
        .
123 => bill@gmail.com
```

Log compaction gives us a more granular retention mechanism so that we are guaranteed to retain at least the last update for each primary key (e.g. bill@gmail.com). By doing this we guarantee that the log contains a full snapshot of the final value for every key not just keys that changed recently. This means downstream consumers can restore their own state off this topic without us having to retain a complete log of all changes.

日志压缩为我们提供了更细化的保留机制，因此我们保证至少保留每个主键的最新的更新（例如bill@gmail.com）。通过这样做，我们保证日志包含每个key的最终值的完整快照，而不仅仅是最近更改的key。这意味着下游消费者可以从这个主题中恢复自己的状态，而无需保留包含所有更改的完整日志。

Let's start by looking at a few use cases where this is useful, then we'll see how it can be used.

我们先看几个有用的用例，然后看看如何使用它。

1. Database change subscription. It is often necessary to have a data set in multiple data systems, and often one of these systems is a database of some kind (either a RDBMS or perhaps a new-fangled key-value store). For example you might have a database, a cache, a search cluster, and a Hadoop cluster. Each change to the database will need to be reflected in the cache, the search cluster, and eventually in Hadoop. In the case that one is only handling the real-time updates you only need recent log. But if you want to be able to reload the cache or restore a failed search node you may need a complete data set.
2. Event sourcing. This is a style of application design which co-locates query processing with application design and uses a log of changes as the primary store for the application.
3. Journaling for high-availability. A process that does local computation can be made fault-tolerant by logging out changes that it makes to its local state so another process can reload these changes and carry on if it should fail. A concrete example of this is handling counts, aggregations, and other "group by"-like processing in a stream query system. Samza, a real-time stream-processing framework, [uses this feature](http://samza.apache.org/learn/documentation/0.7.0/container/state-management.html) for exactly this purpose.


1. 数据库更改订阅。通常需要在多个数据系统中拥有一个数据集，而且这些系统中的一个是某种数据库（RDBMS或可能是一个新开发的键值存储）。例如，您可能有一个数据库，一个缓存，一个搜索集群和一个Hadoop集群。对数据库的每次更改都需要反应在缓存，搜索群集中，最终将反映在Hadoop中。在只处理实时更新的情况下，您只需要最近的日志。但是，如果您希望能够重新加载缓存或恢复失败的搜索节点，则可能需要完整的数据集。
2. 事件溯源。这是一种应用程序设计风格，它将查询处理与应用程序设计协同定位，并使用更改日志作为应用程序的主存储。
3. 日志记录的高可用性。执行本地计算的进程可以通过注销它对本地状态所做的更改来实现容错，以便另一个进程可以重新加载这些更改并在出现故障时继续进行。一个具体的例子是在流查询系统中处理计数，聚合和其它“按类的”处理。Samza是一个实时流处理框架，利用[该特点](http://samza.apache.org/learn/documentation/0.7.0/container/state-management.html)来完成此目的。

In each of these cases one needs primarily to handle the real-time feed of changes, but occasionally, when a machine crashes or data needs to be re-loaded or re-processed, one needs to do a full load. Log compaction allows feeding both of these use cases off the same backing topic. This style of usage of a log is described in more detail in [this blog post](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying).

在这些情况中，主要需要处理变更的实时馈送，但当机器崩溃或需要重新加载或重新处理数据时，则需要执行全部加载。日志压缩允许将这两个用例从同一个支持主题中提取出来。本[博客文章](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)更详细地介绍了这种日志的使用方式。

The general idea is quite simple. If we had infinite log retention, and we logged each change in the above cases, then we would have captured the state of the system at each time from when it first began. Using this complete log, we could restore to any point in time by replaying the first N records in the log. This hypothetical complete log is not very practical for systems that update a single record many times as the log will grow without bound even for a stable dataset. The simple log retention mechanism which throws away old updates will bound space but the log is no longer a way to restore the current state—now restoring from the beginning of the log no longer recreates the current state as old updates may not be captured at all.

总的想法很简单。如果我们拥有无限的日志保留时间，并且记录了上述情况下的每次更改，那么我们将在每次从第一次开始时就捕获系统的状态。使用这个完整的日志，我们可以通过对日志中的前N个消息恢复来回到任何时间点。对于多次更新某一条消息的系统而言，这种假设的完整日志并不是非常实用，因为即使对于稳定的数据集，日志也会无限制地增长。这种抛弃旧的日志的保留机制能限制空间，但这种日志不再是恢复当前状态的一种方式 - 现在将日志从头开始恢复不会重新创建当前状态，因为旧更新可能根本捕获不到。

Log compaction is a mechanism to give finer-grained per-record retention, rather than the coarser-grained time-based retention. The idea is to selectively remove records where we have a more recent update with the same primary key. This way the log is guaranteed to have at least the last state for each key.

日志压缩是提供更细粒度的对每个消息保留的机制，而不是更粗粒度的基于时间的保留。其是有选择地删除消息，我们有一个更新的更新与相同的主键。这样，日志能保证至少对每个key的能保留其最后一个状态。

This retention policy can be set per-topic, so a single cluster can have some topics where retention is enforced by size or time and other topics where retention is enforced by compaction.

此保留策略可以对每个主题单独进行设置，因此单个集群可以有一些主题，其中保留策略是按大小或时间强制执行的，其它主题是通过压缩执行保留的。

This functionality is inspired by one of LinkedIn's oldest and most successful pieces of infrastructure—a database changelog caching service called [Databus](https://github.com/linkedin/databus). Unlike most log-structured storage systems Kafka is built for subscription and organizes data for fast linear reads and writes. Unlike Databus, Kafka acts as a source-of-truth store so it is useful even in situations where the upstream data source would not otherwise be replayable.

此功能受到LinkedIn的最古老且最成功的基础架构之一 - 一个数据库更改日志高速缓存服务[Databus](https://github.com/linkedin/databus)的启发。与大多数结构化日志存储系统不同，Kafka是为订阅而构建的，它用于快速线性读取和写入的数据。与Databus不同的是，Kafka充当真实数据来源存储器，因此即使在上游数据源不可重播的情况下也是能够处理的。

## Log Compaction Basics

## 日志压缩基础

Here is a high-level picture that shows the logical structure of a Kafka log with the offset for each message.

这是一个高级图片，显示每个消息的偏移量和Kafka日志的逻辑结构。

<div align="center">
<img src="../../imgs/log_cleaner_anatomy.png" height="209" width="665" >
</div>

The head of the log is identical to a traditional Kafka log. It has dense, sequential offsets and retains all messages. Log compaction adds an option for handling the tail of the log. The picture above shows a log with a compacted tail. Note that the messages in the tail of the log retain the original offset assigned when they were first written—that never changes. Note also that all offsets remain valid positions in the log, even if the message with that offset has been compacted away; in this case this position is indistinguishable from the next highest offset that does appear in the log. For example, in the picture above the offsets 36, 37, and 38 are all equivalent positions and a read beginning at any of these offsets would return a message set beginning with 38.

日志的头部与传统的Kafka日志完全相同。它具有密集的、连续偏移量并保留了所有消息。日志压缩添加了处理日志尾部的选项。上图显示出了压缩尾部的日志。请注意，日志尾部的消息会保留第一次写入时指定的原始偏移量 - 这些消息永远不会更改。还需注意，即使具有该偏移量的消息已被压缩，所有偏移仍会有效的保留在日志中；在这种情况下，该位置与日志中出现的下一个最高偏移量无法区分。例如，在上图中，偏移量36,37和38都是等同位置，并且从这些偏移量中的任何偏移量开始的读取都将返回从38开始的消息集合。

Compaction also allows for deletes. A message with a key and a null payload will be treated as a delete from the log. This delete marker will cause any prior message with that key to be removed (as would any new message with that key), but delete markers are special in that they will themselves be cleaned out of the log after a period of time to free up space. The point in time at which deletes are no longer retained is marked as the "delete retention point" in the above diagram.

压缩也允许删除。包含有键且有效负载为空的消息将被视为已从日志中删除。这个删除标记会导致任何先前带有该key的消息被删除（与任何带有该key的新消息一样），但是删除标记是特殊的，因为它们自身会在一段时间后从日志中清除以释放空间。删除不再保留的时间点在上图中标记为“删除保留点（delete retention point）”。

The compaction is done in the background by periodically recopying log segments. Cleaning does not block reads and can be throttled to use no more than a configurable amount of I/O throughput to avoid impacting producers and consumers. The actual process of compacting a log segment looks something like this:

压缩是通过在后台定期的复制日志段完成的。日志清理不会阻止消息的读取，并且可以通过限制使用不超过可配置数量的I/O吞吐量来避免影响生产者和消费者。压缩日志段的实际过程如下图所示：

<div align="center">
<img src="../../imgs/log_compaction.png" height="399" width="592" >
</div>

## What guarantees does log compaction provide?

## 日志压缩提供了什么保证？

Log compaction guarantees the following:

日志压缩保证以下内容：

1. Any consumer that stays caught-up to within the head of the log will see every message that is written; these messages will have sequential offsets. The topic's ```min.compaction.lag.ms``` can be used to guarantee the minimum length of time must pass after a message is written before it could be compacted. I.e. it provides a lower bound on how long each message will remain in the (uncompacted) head.
2. Ordering of messages is always maintained. Compaction will never re-order messages, just remove some.
3. he offset for a message never changes. It is the permanent identifier for a position in the log.
4. Any consumer progressing from the start of the log will see at least the final state of all records in the order they were written. Additionally, all delete markers for deleted records will be seen, provided the consumer reaches the head of the log in a time period less than the topic's ```delete.retention.ms``` setting (the default is 24 hours). In other words: since the removal of delete markers happens concurrently with reads, it is possible for a consumer to miss delete markers if it lags by more than ```delete.retention.ms```.


1. 任何一直处于日志头部的消费者，都会看到每条写入的消息；这些消息将具有顺序偏移量。这个主题的```min.compaction.lag.ms```可以用来保证消息在写入之后必须经过一个最短时间后才能被压缩。即它给出了每个消息保留在（未压缩的）头部的时间下限。
2. 排序后的消息始终维持着。压缩操作不会重新排序消息，而是删除一些消息。
3. 消息的偏移不会改变。它是日志中位某个位置的永久标识符。
4. 从日志开始进行的任何消费者将按照他们写入的顺序至少看到所有记录的最终状态。另外，如果用户在小于主题```delete.retention.ms```设置（默认为24小时）的时间内到达日志头部，则会看到所有已删除记录的删除标记。换句话说：因为对删除标记的删除与消息的读取可以同时发生，所以如果消费者滞后时间超过了```delete.retention.ms```，则消费者可能看不到删除标记。

## Log Compaction Details

## 日志压缩细节

Log compaction is handled by the log cleaner, a pool of background threads that recopy log segment files, removing records whose key appears in the head of the log. Each compactor thread works as follows:

日志压缩是由日志清理器处理的，日志清理器是一个后台线程池，用于重新复制日志段文件，删除日志头部出现的消息。每个压缩线程的工作原理如下：

1. It chooses the log that has the highest ratio of log head to log tail
2. It creates a succinct summary of the last offset for each key in the head of the log
3. It recopies the log from beginning to end removing keys which have a later occurrence in the log. New, clean segments are swapped into the log immediately so the additional disk space required is just one additional log segment (not a fully copy of the log).
4. The summary of the log head is essentially just a space-compact hash table. It uses exactly 24 bytes per entry. As a result with 8GB of cleaner buffer one cleaner iteration can clean around 366GB of log head (assuming 1k messages).


1. 选择日志头部与日志尾部出现比率最高的日志
2. 为日志头部中的每个键创建一个其最后偏移量的简明摘要
3. 从头到尾复制日志，删除日志中发生次数较少的键。新的、干净的分段会立即交换到日志中，因此所需的额外磁盘空间只是新增的日志分段（而不是对日志的完全复制）。
4. 日志头部的摘要只是一个空间紧凑的散列表。每个条目使用24个字节。因此，使用8GB清理缓冲区时，一次清理迭代可以清理大约366GB的日志头（假设消息大小为1k）。

## Configuring The Log Cleaner

## 配置日志清理器

The log cleaner is enabled by default. This will start the pool of cleaner threads. To enable log cleaning on a particular topic you can add the log-specific property

日志清理器默认是启用的。这将会打开更干净的线程池。要在特定主题上启用日志清理，您可以添加日志的特定属性

```
log.cleanup.policy=compact
```

This can be done either at topic creation time or using the alter topic command.

这可以在创建主题时或使用改变主题配置的命令完成。

The log cleaner can be configured to retain a minimum amount of the uncompacted "head" of the log. This is enabled by setting the compaction time lag.

日志清理器可以保留日志未压缩“头部”的最小量。这是通过设置压缩滞后时间来启用的。

```
log.cleaner.min.compaction.lag.ms
```

This can be used to prevent messages newer than a minimum message age from being subject to compaction. If not set, all log segments are eligible for compaction except for the last segment, i.e. the one currently being written to. The active segment will not be compacted even if all of its messages are older than the minimum compaction time lag.

这可以用来防止比最小消息时间还新的消息受到压缩。如果未设置，则除了最后一个分段（即当前正在写入的分段）之外，所有日志段都有资格进行压缩。即使活动段的所有消息比最小压缩时间滞后更早，该活动段也不会被压缩。

Further cleaner configurations are described [here](http://kafka.apache.org/documentation.html#brokerconfigs).

更简明的的配置描述在[这里](../configs/broker.md)。