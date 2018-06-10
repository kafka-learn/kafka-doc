# Topic-Level Configs

Configurations pertinent to topics have both a server default as well an optional per-topic override. If no per-topic configuration is given the server default is used. The override can be set at topic creation time by giving one or more ```--config``` options.This example creates a topic named my-topic with a custom max message size and flush rate:

与topic相关的配置包含服务器给出的默认值，也包含可选的每个topic的覆盖值。如果没有给出每个topic的配置，则使用服务器默认值。可以在创建topic的时候通过```--config```给出一些覆盖值。下面的例子给出了一个名称为"my-topic"的 topic，并给出了最大消息容量和缓冲率：

```bash
> bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic my-topic --partitions 1
    --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
```

Overrides can also be changed or set later using the alter configs command. This example updates the max message size for my-topic:

也可以在配置创建之后，对其进行后续的修改，如下是更新“my-topic”的最大消息容量的操作：

```bash
> bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name my-topic
    --alter --add-config max.message.bytes=128000
```

To check overrides set on the topic you can do

运行如下的命令可以检查是否正确修改了

```bash
> bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name my-topic --describe
```

To remove an override you can do

可以执行如下操作来删除配置

```bash
> bin/kafka-configs.sh --zookeeper localhost:2181  --entity-type topics --entity-name my-topic --alter --delete-config max.message.bytes
```

The following are the topic-level configurations. The server's default configuration for this property is given under the Server Default Property heading. A given server default config value only applies to a topic if it does not have an explicit topic config override.

以下是主题级别的配置。服务器的此属性的默认配置在服务器默认属性标题下给出。给定的服务器默认配置值只适用于主题，如果它没有明确的主题配置覆盖。

| Name | Description | Type | Default | Valid Values | Server Default Property | Importance |
| :- | :- | :- | :- | :- | :- | :- |
| cleanup.policy | A string that is either "delete" or "compact". This string designates the retention policy to use on old log segments. The default policy ("delete") will discard old segments when their retention time or size limit has been reached. The "compact" setting will enable [log compaction]() on the topic. | list | delete | [compact, delete] | log.cleanup.policy | medium |
| compression.type | Specify the final compression type for a given topic. This configuration accepts the standard compression codecs ('gzip', 'snappy', lz4). It additionally accepts 'uncompressed' which is equivalent to no compression; and 'producer' which means retain the original compression codec set by the producer. | string | producer | [uncompressed, snappy, lz4, gzip, producer] | compression.type | medium |
| delete.retention.ms | The amount of time to retain delete tombstone markers for [compaction]() topics. This setting also gives a bound on the time in which a consumer must complete a read if they begin from offset 0 to ensure that they get a valid snapshot of the final stage (otherwise delete tombstones may be collected before they complete their scan). | long | 86400000 | [0,...] | log.cleaner.delete.retention.ms | medium |
| file.delete.delay.ms | The time to wait before deleting a file from the filesystem | long | 60000 | [0,...] | log.segment.delete.delay.ms | medium |
| flush.messages | This setting allows specifying an interval at which we will force an fsync of data written to the log. For example if this was set to 1 we would fsync after every message; if it were 5 we would fsync after every five messages. In general we recommend you not set this and use replication for durability and allow the operating system's background flush capabilities as it is more efficient. This setting can be overridden on a per-topic basis (see [topicconfigs](). | long | 9223372036854775807 | [0,...] | log.flush.interval.messages | medium |
| flush.ms | This setting allows specifying a time interval at which we will force an fsync of data written to the log. For example if this was set to 1000 we would fsync after 1000 ms had passed. In general we recommend you not set this and use replication for durability and allow the operating system's background flush capabilities as it is more efficient. | long | 9223372036854775807 | [0,...] | log.flush.interval.ms | medium |
| follower.replication.throttled.replicas | A list of replicas for which log replication should be throttled on the follower side. The list should describe a set of replicas in the form [PartitionId]:[BrokerId],[PartitionId]:[BrokerId]:... or alternatively the wildcard '*' can be used to throttle all replicas for this topic. | list | "" | [partitionId],[brokerId]:[partitionId],[brokerId]:... | follower.replication.throttled.replicas | medium |
| index.interval.bytes | This setting controls how frequently Kafka adds an index entry to it's offset index. The default setting ensures that we index a message roughly every 4096 bytes. More indexing allows reads to jump closer to the exact position in the log but makes the index larger. You probably don't need to change this. | int | 4096 | [0,...] | log.index.interval.bytes | medium |
| leader.replication.throttled.replicas | A list of replicas for which log replication should be throttled on the leader side. The list should describe a set of replicas in the form [PartitionId]:[BrokerId],[PartitionId]:[BrokerId]:... or alternatively the wildcard '*' can be used to throttle all replicas for this topic. | list | "" | [partitionId],[brokerId]:[partitionId],[brokerId]:... | leader.replication.throttled.replicas | medium |
| max.message.bytes | <p>The largest record batch size allowed by Kafka. If this is increased and there are consumers older than 0.10.2, the consumers' fetch size must also be increased so that the they can fetch record batches this large.</p><p>In the latest message format version, records are always grouped into batches for efficiency. In previous message format versions, uncompressed records are not grouped into batches and this limit only applies to a single record in that case.</p> | int | 1000012 | [0,...] | message.max.bytes | medium |
| message.format.version | Specify the message format version the broker will use to append messages to the logs. The value should be a valid ApiVersion. Some examples are: 0.8.2, 0.9.0.0, 0.10.0, check ApiVersion for more details. By setting a particular message format version, the user is certifying that all the existing messages on disk are smaller or equal than the specified version. Setting this value incorrectly will cause consumers with older versions to break as they will receive messages with a format that they don't understand. | string | 1.1-IV0 |  | log.message.format.version | medium |
| message.timestamp.difference.max.ms | The maximum difference allowed between the timestamp when a broker receives a message and the timestamp specified in the message. If message.timestamp.type=CreateTime, a message will be rejected if the difference in timestamp exceeds this threshold. This configuration is ignored if message.timestamp.type=LogAppendTime. | long | 9223372036854775807 | [0,...] | log.message.timestamp.difference.max.ms | medium |
| message.timestamp.type | Define whether the timestamp in the message is message create time or log append time. The value should be either `CreateTime` or `LogAppendTime` | string | CreateTime |  | log.message.timestamp.type | medium |
| min.cleanable.dirty.ratio | This configuration controls how frequently the log compactor will attempt to clean the log (assuming [log compaction]() is enabled). By default we will avoid cleaning a log where more than 50% of the log has been compacted. This ratio bounds the maximum space wasted in the log by duplicates (at 50% at most 50% of the log could be duplicates). A higher ratio will mean fewer, more efficient cleanings but will mean more wasted space in the log. | double | 0.5 | [0,...,1] | log.cleaner.min.cleanable.ratio | medium |
| min.compaction.lag.ms | The minimum time a message will remain uncompacted in the log. Only applicable for logs that are being compacted. | long | 0 | [0,...] | log.cleaner.min.compaction.lag.ms | medium |
| min.insync.replicas | When a producer sets acks to "all" (or "-1"), this configuration specifies the minimum number of replicas that must acknowledge a write for the write to be considered successful. If this minimum cannot be met, then the producer will raise an exception (either NotEnoughReplicas or NotEnoughReplicasAfterAppend).<br/>When used together, min.insync.replicas and acks allow you to enforce greater durability guarantees. A typical scenario would be to create a topic with a replication factor of 3, set min.insync.replicas to 2, and produce with acks of "all". This will ensure that the producer raises an exception if a majority of replicas do not receive a write. | int | 1 | [1,...] | min.insync.replicas | medium |
| preallocate | True if we should preallocate the file on disk when creating a new log segment. | boolean | false |  | log.preallocate | medium |
| retention.bytes | This configuration controls the maximum size a partition (which consists of log segments) can grow to before we will discard old log segments to free up space if we are using the "delete" retention policy. By default there is no size limit only a time limit. Since this limit is enforced at the partition level, multiply it by the number of partitions to compute the topic retention in bytes. | long | -1 |  | log.retention.bytes | medium |
| retention.ms | This configuration controls the maximum time we will retain a log before we will discard old log segments to free up space if we are using the "delete" retention policy. This represents an SLA on how soon consumers must read their data. | long | 604800000 |  | log.retention.ms | medium |
| segment.bytes | This configuration controls the segment file size for the log. Retention and cleaning is always done a file at a time so a larger segment size means fewer files but less granular control over retention. | int | 1073741824 | [14,...] | log.segment.bytes | medium |
| segment.index.bytes | This configuration controls the size of the index that maps offsets to file positions. We preallocate this index file and shrink it only after log rolls. You generally should not need to change this setting. | int | 10485760 | [0,...] | log.index.size.max.bytes | medium |
| segment.jitter.ms | The maximum random jitter subtracted from the scheduled segment roll time to avoid thundering herds of segment rolling | long | 0 | [0,...] | log.roll.jitter.ms | medium |
| segment.ms | This configuration controls the period of time after which Kafka will force the log to roll even if the segment file isn't full to ensure that retention can delete or compact old data. | long | 604800000 | [0,...] | log.roll.ms | medium |
| unclean.leader.election.enable | Indicates whether to enable replicas not in the ISR set to be elected as leader as a last resort, even though doing so may result in data loss. | boolean | false |  | unclean.leader.election.enable | medium |

| 名称 | 描述 | 类型 | 默认值 | 有效值 | 服务器默认属性 | 重要性 |
| :- | :- | :- | :- | :- | :- | :- |
| cleanup.policy | 一个字符串，是"delete"或"compact"。此字符串指定在旧日志段上使用的保留策略。默认策略("delete")将在达到保留时间或大小限制时丢弃旧段。"compact"设置将启用该topic的[日志压缩]()。 | list | delete | [compact, delete] | log.cleanup.policy | medium |
| compression.type | 指定给定topic的最终压缩类型。该配置接受标准压缩编解码器('gzip'，'snappy'，lz4)。另外，它也接受'uncompressed'，这相当于没有压缩；'producer'意味着保留producer设置的原始压缩编解码器。 | string | producer | [uncompressed, snappy, lz4, gzip, producer] | compression.type | medium |
| delete.retention.ms | 为[日志压缩]()topic保留删除墓碑标记(delete tombstone markers)的时间量。此设置还会限制消费者在从偏移量0开始读取时的时间限制，以确保它们获得最终阶段的有效快照（否则在完成扫描之前可能会收集这些删除的tombstone）。 | long | 86400000 | [0,...] | log.cleaner.delete.retention.ms | medium |
| file.delete.delay.ms | 在从文件系统中删除文件之前需要等待的时间 | long | 60000 | [0,...] | log.segment.delete.delay.ms | medium |
| flush.messages | 此设置允许指定一个时间间隔，在这个时间段，我们将强制将数据写入日志中。例如，如果它设置为1，我们将在每条消息之后fsync；如果它是5，我们会在每5条消息之后进行fsync。一般来说，我们建议您不要设置它，并使用备份来提高耐久性，并允许操作系统的后台刷新功能，因为它更高效。此设置可以基于每个主题进行覆盖（请参阅[每个主题的配置部分]()）。 | long | 9223372036854775807 | [0,...] | log.flush.interval.messages | medium |
| flush.ms | 这个设置允许指定一个时间间隔，在这个时间段，我们将强制fsync的数据写入日志。例如，如果它设置为1000，我们将在1000ms后通过fsync。一般来说，我们建议您不要设置它，并使用备份来提高耐久性，并允许操作系统的后台刷新功能，因为它更高效。 | long | 9223372036854775807 | [0,...] | log.flush.interval.ms | medium |
| follower.replication.throttled.replicas | 应在follower侧限制日志复制的备份列表。该列表应该以[PartitionId]：[BrokerId]，[PartitionId]：[BrokerId]：...的形式描述一组备份(replicas)，或者可以使用通配符'*'来限制该topic的所有备份。 | list | "" | [partitionId],[brokerId]:[partitionId],[brokerId]:... | follower.replication.throttled.replicas | medium |
| index.interval.bytes | 此设置控制Kafka向其偏移索引添加索引条目的频率。默认设置能确保我们大致每4096个字节索引一条消息。允许使用更多的索引去读取日志中的确切位置，但这使得索引更大。你可能不需要去改变它。 | int | 4096 | [0,...] | log.index.interval.bytes | medium |
| leader.replication.throttled.replicas | 应该在leader侧限制日志复制的备份列表。该列表应该以[PartitionId]：[BrokerId]，[PartitionId]：[BrokerId]：...的形式描述一组副本，或者可以使用通配符'*'来限制该主题的所有备份。 | list | "" | [partitionId],[brokerId]:[partitionId],[brokerId]:... | leader.replication.throttled.replicas | medium |
| max.message.bytes | 1. Kafka允许的record batch的最大大小。如果增加并且有超过0.10.2版本的消费者，则消费者的获取大小也必须增加，以便他们可以获取这么大的record batch。2. 在最新的消息格式版本中，record是分组为批次以提高效率。在以前的消息格式版本中，未压缩的record不会分组到批次中，并且此限制仅适用于该情况下的单个record。| int | 1000012 | [0,...] | message.max.bytes | medium |
| message.format.version | 指定代理用于将消息附加到日志的消息格式的版本。该值应该是有效的ApiVersion。一些例子是：0.8.2，0.9.0.0，0.10.0，查看ApiVersion以获取更多细节。通过设置特定的消息格式版本，用户能证明磁盘上的所有现有消息都小于或等于指定的版本。如果错误地设置此值，则会导致使用旧版本的消费者中断，因为它们将接收到无法理解的格式的消息。| string | 1.1-IV0 |  | log.message.format.version | medium |
| message.timestamp.difference.max.ms | 代理收到消息时的时间戳与消息中指定的时间戳之间允许的最大差异。如果message.timestamp.type = CreateTime，则如果时间戳的差值超过此阈值，则消息会被拒绝。如果message.timestamp.type = LogAppendTime，则忽略此配置。| long | 9223372036854775807 | [0,...] | log.message.timestamp.difference.max.ms | medium |
| message.timestamp.type | 定义消息中的时间戳是消息创建时间还是日志追加时间。该值应该是`CreateTime`或者`LogAppendTime` | string | CreateTime |  | log.message.timestamp.type | medium |
| min.cleanable.dirty.ratio | 此配置控制日志压缩器清理日志的频率（假设启用了[日志压缩]()）。默认情况下，当超过50％日志已被压缩时，我们将避免清理日志。这个比率限制了重复日志浪费在日志中的最大空间（至多有50％是可重复的）。较高的比例意味着更少的更有效的清洁，但意味着日志中更多的浪费空间。| double | 0.5 | [0,...,1] | log.cleaner.min.cleanable.ratio | medium |
| min.compaction.lag.ms | 消息在日志中保持未压缩的最短时间。仅适用于正在压缩的日志。| long | 0 | [0,...] | log.cleaner.min.compaction.lag.ms | medium |
| min.insync.replicas | 当生产者将ack设置为"all"（或"-1"）时，此配置指定必须确认写入被视为成功的最小备份数量。如果这个最小值不能满足，那么生产者将引发一个异常（NotEnoughReplicas或NotEnoughReplicasAfterAppend）。当同时使用时，min.insync.replicas和acks允许您执行更大的耐久性保证。一个典型的场景是创建一个备份因子为3的主题，将min.insync.replicas设置为2，并生成一个"all"的备份。这将确保生产者在大多数备份未收到写入时就引发异常。| int | 1 | [1,...] | min.insync.replicas | medium |
| preallocate | 如果我们应该在创建新日志段时在磁盘上预先分配文件，则为True。| boolean | false |  | log.preallocate | medium |
| retention.bytes | 如果我们使用"delete"保留策略，则此配置将控制分区（由日志段组成）的最大大小，在我们放弃旧日志段释放空间之前，该分区可以增长到最大。默认情况下，其没有大小限制而只有时间限制。由于此限制是在分区级别实施的，请将其乘以分区数来计算主题保留（以字节为单位）。| long | -1 |  | log.retention.bytes | medium |
| retention.ms | 如果我们使用"delete"保留策略，则此配置将控制我们将保留日志的最长时间，然后我们将丢弃旧日志段以释放空间。这代表了消费者必须快速阅读数据的SLA。| long | 604800000 |  | log.retention.ms | medium |
| segment.bytes | 此配置控制日志的段文件大小。保留和清理总是一次完成一个文件，因此较大的段大小意味着较少的文件，但对保留的控制更少。 | int | 1073741824 | [14,...] | log.segment.bytes | medium |
| segment.index.bytes | 此配置控制将偏移量映射到文件位置的索引大小。我们预先分配这个索引文件并且只在日志滚动后收缩它。您通常不需要更改此设置。 | int | 10485760 | [0,...] | log.index.size.max.bytes | medium |
| segment.jitter.ms | 从预定的分段滚动时间中减去最大的随机抖动，以避免分段滚动群的剧烈抖动 | long | 0 | [0,...] | log.roll.jitter.ms | medium |
| segment.ms | 此配置控制Kafka即使段文件未满时也会强制日志滚动的时间段，以确保保留可删除或压缩旧数据。 | long | 604800000 | [0,...] | log.roll.ms | medium |
| unclean.leader.election.enable | 指示是否启用不在ISR集中的备份(replicas)作为最后选择的leader，即使这样做可能会导致数据丢失。| boolean | false |  | unclean.leader.election.enable | medium |