# 5.5 Distribution

# 5.5 分布式

## Consumer Offset Tracking

The high-level consumer tracks the maximum offset it has consumed in each partition and periodically commits its offset vector so that it can resume from those offsets in the event of a restart. Kafka provides the option to store all the offsets for a given consumer group in a designated broker (for that group) called the offset manager. i.e., any consumer instance in that consumer group should send its offset commits and fetches to that offset manager (broker). The high-level consumer handles this automatically. If you use the simple consumer you will need to manage offsets manually. This is currently unsupported in the Java simple consumer which can only commit or fetch offsets in ZooKeeper. If you use the Scala simple consumer you can discover the offset manager and explicitly commit or fetch offsets to the offset manager. A consumer can look up its offset manager by issuing a GroupCoordinatorRequest to any Kafka broker and reading the GroupCoordinatorResponse which will contain the offset manager. The consumer can then proceed to commit or fetch offsets from the offsets manager broker. In case the offset manager moves, the consumer will need to rediscover the offset manager. If you wish to manage your offsets manually, you can take a look at these [code samples that explain how to issue OffsetCommitRequest and OffsetFetchRequest](https://cwiki.apache.org/confluence/display/KAFKA/Committing+and+fetching+consumer+offsets+in+Kafka).

高级消费者跟踪它在每个分区中消耗的最大偏移量，并定期提交其偏移量向量，以便在重新启动时从这些偏移量恢复。Kafka提供了一个选项，可以在指定代理（针对该组）的给定消费者组中存储所有偏移。即该消费者组中的任何消费者实例应该将其偏移提交，并且发送给该偏移管理器（代理）。高级消费者能自动处理。如果您使用的是简单消费者，则需要手动管理偏移量。目前在Java简单消费者中不支持该功能，它只能在ZooKeeper中提交或提取偏移量。如果您使用Scala简单消费者，您可以发现偏移量管理器，并明确提交或提取偏移量管理器的偏移量。消费者可以通过向任何Kafka代理发送GroupCoordinatorRequest并阅读其包含的偏移管理。接着，消费者可以继续从偏移管理器代理处提交或提取偏移量。在偏移量管理器移动的情况下，消费者将需要重新发现偏移量管理器。如果你想手动管理你的偏移，你可以看看这些代码示例[解释如何发布OffsetCommitRequest和OffsetFetchRequest](https://cwiki.apache.org/confluence/display/KAFKA/Committing+and+fetching+consumer+offsets+in+Kafka)。

When the offset manager receives an OffsetCommitRequest, it appends the request to a special [compacted](http://kafka.apache.org/documentation/#compaction) Kafka topic named __consumer_offsets. The offset manager sends a successful offset commit response to the consumer only after all the replicas of the offsets topic receive the offsets. In case the offsets fail to replicate within a configurable timeout, the offset commit will fail and the consumer may retry the commit after backing off. (This is done automatically by the high-level consumer.) The brokers periodically compact the offsets topic since it only needs to maintain the most recent offset commit per partition. The offset manager also caches the offsets in an in-memory table in order to serve offset fetches quickly.

当偏移量管理器接收到OffsetCommitRequest时，它会将请求附加到名为__consumer_offsets的特殊的[压缩的](http://kafka.apache.org/documentation/#compaction)Kafka主题。只有在偏移量主题的所有副本都接收到偏移量后，偏移量管理器才会向使用方发送成功的偏移量提交响应。如果偏移无法在可配置的超时内复制，则偏移提交将失败，并且客户可能会在退出后重试提交。（这由高级消费者自动完成。）代理定期压缩偏移量主题，因为它只需维护每个分区的最近偏移量提交。偏移量管理器还将偏移量缓存在内存表中以便快速提供偏移量提取。

When the offset manager receives an offset fetch request, it simply returns the last committed offset vector from the offsets cache. In case the offset manager was just started or if it just became the offset manager for a new set of consumer groups (by becoming a leader for a partition of the offsets topic), it may need to load the offsets topic partition into the cache. In this case, the offset fetch will fail with an OffsetsLoadInProgress exception and the consumer may retry the OffsetFetchRequest after backing off. (This is done automatically by the high-level consumer.)

当偏移量管理器接收到偏移量提取请求时，它只是从偏移量缓存中返回最后提交的偏移量向量。如果偏移量管理器刚刚启动，或者它刚成为新的一组消费者组的偏移量管理器（通过成为偏移量主题分区的leader），它可能需要将偏移量主题分区加载到缓存中。在这种情况下，偏移获取将失败并出现OffsetsLoadInProgress异常，并且客户可能会在退避后重试OffsetFetchRequest。（这由高级消费者自动完成。）

### Migrating offsets from ZooKeeper to Kafka

Kafka consumers in earlier releases store their offsets by default in ZooKeeper. It is possible to migrate these consumers to commit offsets into Kafka by following these steps:

早期版本中的Kafka使用者默认在ZooKeeper中存储它们的偏移量。通过执行以下步骤，可以将这些消费者迁移到Kafka中：

1. Set ```offsets.storage=kafka``` and ```dual.commit.enabled=true``` in your consumer config.
2. Do a rolling bounce of your consumers and then verify that your consumers are healthy.
3. Set ```dual.commit.enabled=false``` in your consumer config.
4. Do a rolling bounce of your consumers and then verify that your consumers are healthy.


1. 在消费者配置中设置```offsets.storage=kafka```和```dual.commit.enabled=true```
2. 将消费者做一个回滚，然后确认你的消费者是健康的。
3. 在消费者配置中设置```dual.commit.enabled=false```
4. 将消费者做一个回滚，然后确认你的消费者是健康的。

A roll-back (i.e., migrating from Kafka back to ZooKeeper) can also be performed using the above steps if you set ```offsets.storage=zookeeper```.

如果您设置了```offsetsets.storage = zookeeper```，则还可以使用上述步骤执行回滚（即从Kafka迁移回ZooKeeper）。

## ZooKeeper Directories

The following gives the ZooKeeper structures and algorithms used for co-ordination between consumers and brokers.

以下给出了用于消费者和代理之间协调的ZooKeeper结构和算法。

### Notation

When an element in a path is denoted [xyz], that means that the value of xyz is not fixed and there is in fact a ZooKeeper znode for each possible value of xyz. For example /topics/[topic] would be a directory named /topics containing a sub-directory for each topic name. Numerical ranges are also given such as [0...5] to indicate the subdirectories 0, 1, 2, 3, 4. An arrow -> is used to indicate the contents of a znode. For example /hello -> world would indicate a znode /hello containing the value "world".

当路径中的元素表示为[xyz]时，这意味着xyz的值不固定，并且实际上每个可能的xyz值都有一个ZooKeeper znode。例如/ topics/[topic ]将是一个名为 /topics 的目录，其中包含每个主题名称的子目录。还给出了数值范围，例如[0 ... 5]以指示子目录0,1,2,3,4。箭头 -> 用于指示znode的内容。例如 /hello -> world 会指示包含值“world”的znode /hello。

### Broker Node Registry

```
/brokers/ids/[0...N] --> {"jmx_port":...,"timestamp":...,"endpoints":[...],"host":...,"version":...,"port":...} (ephemeral node)
```

This is a list of all present broker nodes, each of which provides a unique logical broker id which identifies it to consumers (which must be given as part of its configuration). On startup, a broker node registers itself by creating a znode with the logical broker id under /brokers/ids. The purpose of the logical broker id is to allow a broker to be moved to a different physical machine without affecting consumers. An attempt to register a broker id that is already in use (say because two servers are configured with the same broker id) results in an error.

这是所有当前代理节点的列表，每个代理节点都提供一个唯一的逻辑代理标识符，用于向消费者标识它（必须将其作为其配置的一部分给出）。在启动时，代理节点通过在 /brokers/id 下创建逻辑代理标识创建znode来注册自己。逻辑代理标识的用途是允许代理移动到不同的物理机器而不影响用户。尝试注册已使用的代理标识（例如，因为两台服务器配置了相同的代理标识）会导致错误。

Since the broker registers itself in ZooKeeper using ephemeral znodes, this registration is dynamic and will disappear if the broker is shutdown or dies (thus notifying consumers it is no longer available).

由于代理使用临时znode在ZooKeeper中注册自己，此注册是动态的，并且在代理关闭或死亡（通知消费者此代理不再可用）时将消失。

### Broker Topic Registry

```
/brokers/topics/[topic]/partitions/[0...N]/state --> {"controller_epoch":...,"leader":...,"version":...,"leader_epoch":...,"isr":[...]} (ephemeral node)
```

Each broker registers itself under the topics it maintains and stores the number of partitions for that topic.

每个代理在其维护的主题下注册自己，并存储该主题的分区数量。

### Consumers and Consumer Groups

Consumers of topics also register themselves in ZooKeeper, in order to coordinate with each other and balance the consumption of data. Consumers can also store their offsets in ZooKeeper by setting ```offsets.storage=zookeeper```. However, this offset storage mechanism will be deprecated in a future release. Therefore, it is recommended to [migrate offsets storage to Kafka](http://kafka.apache.org/documentation/#offsetmigration).

主题的消费者也在ZooKeeper中注册自己，以便相互协调并平衡数据的消耗。消费者也可以通过设置```offsetsets.storage=zookeeper```将他们的偏移量存储在ZooKeeper中。但是，此偏移量存储机制在以后的版本中将不再使用。因此，建议[将存储迁移到Kafka](http://kafka.apache.org/documentation/#offsetmigration)。

Multiple consumers can form a group and jointly consume a single topic. Each consumer in the same group is given a shared group_id. For example if one consumer is your foobar process, which is run across three machines, then you might assign this group of consumers the id "foobar". This group id is provided in the configuration of the consumer, and is your way to tell the consumer which group it belongs to.

多个消费者可以组成一个组并共同消费一个主题。同一组中的每个消费者都有一个共享的group_id。例如，如果一个消费者是您的foobar进程，它通过三台机器运行，那么您可以为这组消费者分配id为“foobar”。该组ID是在消费者的配置中提供的，并且是告诉消费者它属于哪个组的消息的方式。

The consumers in a group divide up the partitions as fairly as possible, each partition is consumed by exactly one consumer in a consumer group.

组中的消费者尽可能公平地划分分区，每个分区恰好被消费者组中的一个消费者消费。

### Consumer Id Registry

In addition to the group_id which is shared by all consumers in a group, each consumer is given a transient, unique consumer_id (of the form hostname:uuid) for identification purposes. Consumer ids are registered in the following directory.

除了群组中的所有消费者共享的group_id之外，每个消费者都会获得一个临时的、唯一的consumer_id（形式为hostname：uuid）用于识别目的。消费者ID在以下目录中注册。

```
/consumers/[group_id]/ids/[consumer_id] --> {"version":...,"subscription":{...:...},"pattern":...,"timestamp":...} (ephemeral node)
```

Each of the consumers in the group registers under its group and creates a znode with its consumer_id. The value of the znode contains a map of &lt;topic, #streams&gt;. This id is simply used to identify each of the consumers which is currently active within a group. This is an ephemeral node so it will disappear if the consumer process dies.

组中的每个使用者都在其组中注册，并使用其consumer_id创建一个znode。znode的值包含 &lt;topic,＃streams&gt;的映射。此ID仅用于标识组中当前处于活动状态的每个消费者。这是一个短暂节点，所以如果消费者进程死亡，它将消失。

### Consumer Offsets

Consumers track the maximum offset they have consumed in each partition. This value is stored in a ZooKeeper directory if ```offsets.storage=zookeeper```.

消费者追踪他们在每个分区中消耗的最大偏移量。如果```offsetset.storage=zookeeper```，则这个值存储在一个ZooKeeper目录中。

```
/consumers/[group_id]/offsets/[topic]/[partition_id] --> offset_counter_value (persistent node)
```

### Partition Owner registry

Each broker partition is consumed by a single consumer within a given consumer group. The consumer must establish its ownership of a given partition before any consumption can begin. To establish its ownership, a consumer writes its own id in an ephemeral node under the particular broker partition it is claiming.

每个代理分区由给定消费者组内的单个消费者使用。消费者在开始消费之前必须确定其给定分区的所有权。为了建立它的所有权，消费者将自己的ID写入它声称的特定代理分区下的临时节点中。

```
/consumers/[group_id]/owners/[topic]/[partition_id] --> consumer_node_id (ephemeral node)
```

### Cluster Id

The cluster id is a unique and immutable identifier assigned to a Kafka cluster. The cluster id can have a maximum of 22 characters and the allowed characters are defined by the regular expression [a-zA-Z0-9_\\-]+, which corresponds to the characters used by the URL-safe Base64 variant with no padding. Conceptually, it is auto-generated when a cluster is started for the first time.

集群ID是分配给Kafka集群的唯一且不可变的标识符。集群ID最多可以有22个字符，并且允许的字符由正则表达式[a-zA-Z0-9_\\-]+ 定义，该正则表达式对应于不带填充的URL安全的Base64变体使用的字符。从概念上讲，它是在集群第一次启动时自动生成的。

Implementation-wise, it is generated when a broker with version 0.10.1 or later is successfully started for the first time. The broker tries to get the cluster id from the ```/cluster/id``` znode during startup. If the znode does not exist, the broker generates a new cluster id and creates the znode with this cluster id.

在实现方面，它是在版本为0.10.1或更高版本的代理首次成功启动时生成的。代理尝试在启动期间从```/cluster/id```znode获取集群标识。如果znode不存在，代理将生成新的群群标识并使用此集群标识创建znode。

### Broker node registration

The broker nodes are basically independent, so they only publish information about what they have. When a broker joins, it registers itself under the broker node registry directory and writes information about its host name and port. The broker also register the list of existing topics and their logical partitions in the broker topic registry. New topics are registered dynamically when they are created on the broker.

代理节点基本上是独立的，所以它们只发布关于它们的信息。代理加入时，它会将自身注册到代理节点注册目录下，并写入有关其主机名和端口的信息。代理还会在代理主题注册表中注册现有主题及其逻辑分区的列表。在代理上创建新主题时会动态注册新主题。

### Consumer registration algorithm

When a consumer starts, it does the following:

当消费者启动时，它执行以下操作：

1. Register itself in the consumer id registry under its group.
2. Register a watch on changes (new consumers joining or any existing consumers leaving) under the consumer id registry. (Each change triggers rebalancing among all consumers within the group to which the changed consumer belongs.)
3. Register a watch on changes (new brokers joining or any existing brokers leaving) under the broker id registry. (Each change triggers rebalancing among all consumers in all consumer groups.)
4. If the consumer creates a message stream using a topic filter, it also registers a watch on changes (new topics being added) under the broker topic registry. (Each change will trigger re-evaluation of the available topics to determine which topics are allowed by the topic filter. A new allowed topic will trigger rebalancing among all consumers within the consumer group.)
5. Force itself to rebalance within in its consumer group.


1. 将其自己注册在其组中的消费者ID注册表中。
2. 在消费者ID注册表下注册关于更改（新消费者加入或任何现有消费者离开）的观察者。（每次更改都会触发更改的消费者所属的群组内的所有消费者之间的再平衡。）
3. 在代理ID注册表下注册关于更改的观察者（新代理加入或任何现有代理离开）。（每个变化都会触发所有消费群体中所有消费者之间的再平衡。）
4. 如果消费者使用主题过滤器创建消息流，则还会在代理主题注册表下注册关于更改的监视（新增主题）。（每次更改都会触发对可用主题的重新评估，以确定主题过滤器允许哪些主题。新的允许主题将触发消费者组中所有消费者之间的重新平衡。）
5. 强迫自己在消费群组内重新平衡。

### Consumer rebalancing algorithm

The consumer rebalancing algorithms allows all the consumers in a group to come into consensus on which consumer is consuming which partitions. Consumer rebalancing is triggered on each addition or removal of both broker nodes and other consumers within the same group. For a given topic and a given consumer group, broker partitions are divided evenly among consumers within the group. A partition is always consumed by a single consumer. This design simplifies the implementation. Had we allowed a partition to be concurrently consumed by multiple consumers, there would be contention on the partition and some kind of locking would be required. If there are more consumers than partitions, some consumers won't get any data at all. During rebalancing, we try to assign partitions to consumers in such a way that reduces the number of broker nodes each consumer has to connect to.

消费者重新平衡算法允许组中的所有消费者就哪个消费者正在消费哪些分区达成共识。每次添加或删除同一组内的代理节点和其他消费者时都会触发消费者重新平衡。对于给定的主题和给定的消费者组，代理分区在组内的消费者之间均匀分配。分区总是由单个用户使用。这种设计简化了实现。如果我们允许一个分区被多个消费者同时使用，那么分区上就会出现争用，并且需要某种锁定。如果消费者比分区多，一些消费者根本就得不到任何数据。在重新平衡期间，我们尝试以减少每个消费者必须连接的代理节点数量的方式将消费分配给消费者。

Each consumer does the following during rebalancing:

每个消费者在重新平衡期间执行以下操作：

```
1. For each topic T that C<sub>i</sub> subscribes to
2.   let P<sub>T</sub> be all partitions producing topic T
3.   let C<sub>G</sub> be all consumers in the same group as C<sub>i</sub> that consume topic T
4.   sort P<sub>T</sub> (so partitions on the same broker are clustered together)
5.   sort C<sub>G</sub>
6.   let i be the index position of C<sub>i</sub> in C<sub>G</sub> and let N = size(P<sub>T</sub>)/size(C<sub>G</sub>)
7.   assign partitions from i*N to (i+1)*N - 1 to consumer C<sub>i</sub>
8.   remove current entries owned by C<sub>i</sub> from the partition owner registry
9.   add newly assigned partitions to the partition owner registry
        (we may need to re-try this until the original partition owner releases its ownership)
```

When rebalancing is triggered at one consumer, rebalancing should be triggered in other consumers within the same group about the same time.

当某个消费者触发再平衡时，同一时间内同一组内的其它消费者应该重新平衡。