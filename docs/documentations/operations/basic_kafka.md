# 6.1 Basic Kafka Operations

# 6.1 Kafka基础操作

This section will review the most common operations you will perform on your Kafka cluster. All of the tools reviewed in this section are available under the ```bin/``` directory of the Kafka distribution and each tool will print details on all possible commandline options if it is run with no arguments.

本节将回顾您将在Kafka集群上执行的最常见操作。 所有在本节中回顾的工具都可以在Kafka发行版的```bin/```目录下找到，如果没有参数运行，每个工具都会打印所有可能的命令行选项的详细信息。

## Adding and removing topics

## 添加和删除主题

You have the option of either adding topics manually or having them be created automatically when data is first published to a non-existent topic. If topics are auto-created then you may want to tune the default [topic configurations](http://kafka.apache.org/documentation/#topicconfigs) used for auto-created topics.

您可以选择手动添加主题，或者在数据首次发布到不存在的主题时自动创建主题。 如果主题是自动创建的，那么您可能需要调整用于自动创建主题的默认[主题配置](http://kafka.apache.org/documentation/#topicconfigs)。

Topics are added and modified using the topic tool:

使用主题工具添加和修改主题：

```bash
> bin/kafka-topics.sh --zookeeper zk_host:port/chroot --create --topic my_topic_name
      --partitions 20 --replication-factor 3 --config x=y
```

The replication factor controls how many servers will replicate each message that is written. If you have a replication factor of 3 then up to 2 servers can fail before you will lose access to your data. We recommend you use a replication factor of 2 or 3 so that you can transparently bounce machines without interrupting data consumption.

复制因子控制有多少服务器将复制每个写入的消息。 如果复制因子为3，则最多有2台服务器可能会失败，然后您将无法访问数据。 我们建议您使用2或3的复制因子，以便在不中断数据消耗的情况下透明地反弹机器。

The partition count controls how many logs the topic will be sharded into. There are several impacts of the partition count. First each partition must fit entirely on a single server. So if you have 20 partitions the full data set (and read and write load) will be handled by no more than 20 servers (not counting replicas). Finally the partition count impacts the maximum parallelism of your consumers. This is discussed in greater detail in the [concepts section](http://kafka.apache.org/documentation/#intro_consumers).

分区计数控制主题将被分成多少个日志。 分区计数有几个影响。 首先，每个分区必须完全适合一台服务器。 所以如果你有20个分区，完整的数据集（和读写负载）将由不超过20个服务器处理（不包括副本）。 最后，分区数会影响消费者的最大并行性。 这在[概念部分](http://kafka.apache.org/documentation/#intro_consumers)中有更详细的讨论。

Each sharded partition log is placed into its own folder under the Kafka log directory. The name of such folders consists of the topic name, appended by a dash (-) and the partition id. Since a typical folder name can not be over 255 characters long, there will be a limitation on the length of topic names. We assume the number of partitions will not ever be above 100,000. Therefore, topic names cannot be longer than 249 characters. This leaves just enough room in the folder name for a dash and a potentially 5 digit long partition id.

每个分片分区日志都放在Kafka日志目录下的自己的文件夹中。 这些文件夹的名称由主题名称组成，附有短划线（ - ）和分区ID。 由于典型的文件夹名称长度不能超过255个字符，因此主题名称的长度会受到限制。 我们假设分区的数量不会超过100,000。 因此，主题名称不能超过249个字符。 这在文件夹名称中留下了足够的空间以显示短划线和可能的5位长分区ID。

The configurations added on the command line override the default settings the server has for things like the length of time data should be retained. The complete set of per-topic configurations is documented [here](http://kafka.apache.org/documentation/#topicconfigs).

在命令行中添加的配置会覆盖服务器的默认设置，例如应该保留数据的时间长度。有关每个主题配置的完整集合记录在[这里](http://kafka.apache.org/documentation/#topicconfigs)。

## Modifying topics

## 修改主题

You can change the configuration or partitioning of a topic using the same topic tool.

您可以使用相同的主题工具更改主题的配置或分区。

To add partitions you can do

要添加分区，你可以做

```bash
> bin/kafka-topics.sh --zookeeper zk_host:port/chroot --alter --topic my_topic_name
      --partitions 40
```

Be aware that one use case for partitions is to semantically partition data, and adding partitions doesn't change the partitioning of existing data so this may disturb consumers if they rely on that partition. That is if data is partitioned by ```hash(key) % number_of_partitions``` then this partitioning will potentially be shuffled by adding partitions but Kafka will not attempt to automatically redistribute data in any way.

请注意，分区的一种用例是对数据进行语义分区，并且添加分区不会更改现有数据的分区，因此如果它们依赖于该分区，则可能会影响消费者。 也就是说，如果数据是用```hash（key）％number_of_partitions``分区的，那么这个分区可能会通过添加分区进行混洗，但Kafka不会尝试以任何方式自动重新分配数据。

To add configs:

添加配置：

```bash
> bin/kafka-configs.sh --zookeeper zk_host:port/chroot --entity-type topics --entity-name my_topic_name --alter --add-config x=y
```

To remove a config:

删除配置：

```bash
> bin/kafka-configs.sh --zookeeper zk_host:port/chroot --entity-type topics --entity-name my_topic_name --alter --delete-config x
```

And finally deleting a topic:

最后删除一个主题：

```bash
> bin/kafka-topics.sh --zookeeper zk_host:port/chroot --delete --topic my_topic_name
```

Kafka does not currently support reducing the number of partitions for a topic.

Kafka目前不支持减少某个主题的分区数量。

Instructions for changing the replication factor of a topic can be found [here](http://kafka.apache.org/documentation/#basic_ops_increase_replication_factor).

可以在[这里](http://kafka.apache.org/documentation/#basic_ops_increase_replication_factor)找到更改主题复制因子的指令。

## Graceful shutdown

The Kafka cluster will automatically detect any broker shutdown or failure and elect new leaders for the partitions on that machine. This will occur whether a server fails or it is brought down intentionally for maintenance or configuration changes. For the latter cases Kafka supports a more graceful mechanism for stopping a server than just killing it. When a server is stopped gracefully it has two optimizations it will take advantage of:

Kafka集群将自动检测任何代理关闭或故障，并为该计算机上的分区选择新的领导者。 无论服务器发生故障还是因为维护或配置更改而故意将其关闭，都会发生这种情况。 对于后者，Kafka支持更优雅的停止服务器的机制，而不仅仅是杀死它。 当服务器正常停止时，它有两个优化，它将利用：

1. It will sync all its logs to disk to avoid needing to do any log recovery when it restarts (i.e. validating the checksum for all messages in the tail of the log). Log recovery takes time so this speeds up intentional restarts.
2. It will migrate any partitions the server is the leader for to other replicas prior to shutting down. This will make the leadership transfer faster and minimize the time each partition is unavailable to a few milliseconds.


1. 它会将其所有日志同步到磁盘，以避免重新启动时需要执行任何日志恢复（即验证日志尾部中所有消息的校验和）。 日志恢复需要时间，所以这会加速有意重启。
2. 它将在关闭之前将服务器领导者的任何分区迁移到其他副本。 这将使领导传输更快，并将每个分区不可用的时间缩短到几毫秒。

Syncing the logs will happen automatically whenever the server is stopped other than by a hard kill, but the controlled leadership migration requires using a special setting:

只要服务器停止而不是通过硬性杀死，同步日志就会自动发生，但受控领导层迁移需要使用特殊设置：

```
controlled.shutdown.enable=true
```

Note that controlled shutdown will only succeed if all the partitions hosted on the broker have replicas (i.e. the replication factor is greater than 1 and at least one of these replicas is alive). This is generally what you want since shutting down the last replica would make that topic partition unavailable.

请注意，如果代理上承载的所有分区都具有副本（即复制因子大于1并且这些副本中至少有一个存活），则受控关闭只能成功。 这通常是您想要的，因为关闭最后一个副本会使该主题分区不可用。

## Balancing leadership

Whenever a broker stops or crashes leadership for that broker's partitions transfers to other replicas. This means that by default when the broker is restarted it will only be a follower for all its partitions, meaning it will not be used for client reads and writes.

每当代理停止或崩溃对该代理的分区转移到其他副本的领导。 这意味着，在代理重新启动时，默认情况下它将只是所有分区的跟随者，这意味着它不会用于客户端读取和写入。

To avoid this imbalance, Kafka has a notion of preferred replicas. If the list of replicas for a partition is 1,5,9 then node 1 is preferred as the leader to either node 5 or 9 because it is earlier in the replica list. You can have the Kafka cluster try to restore leadership to the restored replicas by running the command:

为了避免这种不平衡，Kafka有一个首选复制品的概念。 如果分区的副本列表为1,5,9，则节点1首选为节点5或9的组长，因为它在副本列表中较早。 您可以通过运行以下命令让Kafka集群尝试恢复已恢复副本的领导地位：

```bash
> bin/kafka-preferred-replica-election.sh --zookeeper zk_host:port/chroot
```

Since running this command can be tedious you can also configure Kafka to do this automatically by setting the following configuration:

由于运行此命令可能很乏味，因此您可以通过设置以下配置来自动配置Kafka自动执行此操作：

```bash
auto.leader.rebalance.enable=true
```

## Balancing Replicas Across Racks

The rack awareness feature spreads replicas of the same partition across different racks. This extends the guarantees Kafka provides for broker-failure to cover rack-failure, limiting the risk of data loss should all the brokers on a rack fail at once. The feature can also be applied to other broker groupings such as availability zones in EC2.

机架感知功能可将同一分区的复制副本分散到不同的机架上。这扩展了Kafka为代理故障提供的担保，以弥补机架故障，从而限制机架上所有代理一次失败时数据丢失的风险。 该功能也可以应用于其他代理分组，如EC2中的可用区域。

You can specify that a broker belongs to a particular rack by adding a property to the broker config:

您可以通过向代理配置添加属性来指定代理属于特定机架：

```bash
broker.rack=my-rack-id
```

When a topic is [created](http://kafka.apache.org/documentation/#basic_ops_add_topic), [modified](http://kafka.apache.org/documentation/#basic_ops_modify_topic) or replicas are [redistributed](http://kafka.apache.org/documentation/#basic_ops_cluster_expansion), the rack constraint will be honoured, ensuring replicas span as many racks as they can (a partition will span min(#racks, replication-factor) different racks).

当一个主题是[created](http://kafka.apache.org/documentation/#basic_ops_add_topic)，[modified](http://kafka.apache.org/documentation/#basic_ops_modify_topic)或副本是[redistributed](http://kafka.apache.org/documentation/#basic_ops_cluster_expansion)，机架约束将得到遵守，确保副本跨越尽可能多的机架（一个分区将跨越min（#racks，replication-factor）不同的机架）。

The algorithm used to assign replicas to brokers ensures that the number of leaders per broker will be constant, regardless of how brokers are distributed across racks. This ensures balanced throughput.

用于向代理分配副本的算法确保每个代理的领导者数量将保持不变，而不管代理如何在机架中分布。这确保了平衡的吞吐量

However if racks are assigned different numbers of brokers, the assignment of replicas will not be even. Racks with fewer brokers will get more replicas, meaning they will use more storage and put more resources into replication. Hence it is sensible to configure an equal number of brokers per rack.

但是，如果机架分配有不同数量的代理，则副本的分配将不均匀。 代理数量较少的机架将获得更多副本，这意味着他们将使用更多存储并将更多资源投入复制。 因此，每个机架配置相同数量的代理是明智的。

## Mirroring data between clusters

We refer to the process of replicating data between Kafka clusters "mirroring" to avoid confusion with the replication that happens amongst the nodes in a single cluster. Kafka comes with a tool for mirroring data between Kafka clusters. The tool consumes from a source cluster and produces to a destination cluster. A common use case for this kind of mirroring is to provide a replica in another datacenter. This scenario will be discussed in more detail in the next section.

我们指的是在“镜像”之间复制Kafka集群之间的数据的过程，以避免与单个集群中的节点之间发生的复制混淆。 Kafka附带了一个在Kafka集群之间镜像数据的工具。 该工具从源集群消耗并产生到目标集群。 这种镜像的常见用例是在另一个数据中心提供副本。 这种情况将在下一节中详细讨论。

You can run many such mirroring processes to increase throughput and for fault-tolerance (if one process dies, the others will take overs the additional load).

您可以运行许多这样的镜像过程来提高吞吐量和容错能力（如果一个进程死了，其他进程将承担额外的负载）。

Data will be read from topics in the source cluster and written to a topic with the same name in the destination cluster. In fact the mirror maker is little more than a Kafka consumer and producer hooked together.

将从源群集中的主题读取数据并将其写入目标群集中具有相同名称的主题。 事实上，镜子制造商只不过是一个卡夫卡消费者和制造商联合在一起。

The source and destination clusters are completely independent entities: they can have different numbers of partitions and the offsets will not be the same. For this reason the mirror cluster is not really intended as a fault-tolerance mechanism (as the consumer position will be different); for that we recommend using normal in-cluster replication. The mirror maker process will, however, retain and use the message key for partitioning so order is preserved on a per-key basis.

源和目标集群是完全独立的实体：它们可以具有不同数量的分区，并且偏移量不会相同。 由于这个原因，镜像集群并不是真正意义上的容错机制（因为消费者的位置会有所不同）; 为此，我们建议使用正常的群集内复制。 但镜像制作者进程将保留并使用消息密钥进行分区，以便按每个密钥保留顺序。

Here is an example showing how to mirror a single topic (named my-topic) from an input cluster:

以下示例显示如何从输入群集中镜像单个主题（名为my-topic）：

```bash
> bin/kafka-mirror-maker.sh
      --consumer.config consumer.properties
      --producer.config producer.properties --whitelist my-topic
```

Note that we specify the list of topics with the ```--whitelist``` option. This option allows any regular expression using [Java-style regular expressions](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html). So you could mirror two topics named A and B using ```--whitelist 'A|B'```. Or you could mirror all topics using ```--whitelist '*'```. Make sure to quote any regular expression to ensure the shell doesn't try to expand it as a file path. For convenience we allow the use of ',' instead of '|' to specify a list of topics.

请注意，我们使用```--whitelist```选项指定主题列表。 该选项允许使用[Java风格的正则表达式](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html)的任何正则表达式。 所以你可以使用```--whitelist'A | B```来反映名为A和B的两个主题。 或者你可以使用```--whitelist'*'``镜像所有主题。 确保引用任何正则表达式以确保shell不会尝试将其展开为文件路径。 为了方便起见，我们允许使用'，'而不是'|' 指定主题列表。

Sometimes it is easier to say what it is that you don't want. Instead of using ```--whitelist``` to say what you want to mirror you can use ```--blacklist``` to say what to exclude. This also takes a regular expression argument. However, ```--blacklist``` is not supported when the new consumer has been enabled (i.e. when ```bootstrap.servers``` has been defined in the consumer configuration).

有时候更容易说出你不想要的东西。 你可以使用```--blacklist```来表示要排除的内容，而不是使用```--whitelist```来表示你想要镜像的内容。 这也需要一个正则表达式参数。 但是，当新的使用者被启用时（即在使用者配置中已经定义了```bootstrap.servers```），不支持```--blacklist```。

Combining mirroring with the configuration ```auto.create.topics.enable=true``` makes it possible to have a replica cluster that will automatically create and replicate all data in a source cluster even as new topics are added.

将镜像与配置```auto.create.topics.enable = true```结合使用，可以创建一个副本群集，即使在添加新主题时也会自动创建并复制源群集中的所有数据。

## Checking consumer position

Sometimes it's useful to see the position of your consumers. We have a tool that will show the position of all consumers in a consumer group as well as how far behind the end of the log they are. To run this tool on a consumer group named my-group consuming a topic named my-topic would look like this:

有时看到消费者的位置很有用。我们有一个工具可以显示消费者群体中所有消费者的位置以及他们所在日志的结尾。 要在名为my-group的使用者组上使用名为my-topic的主题运行此工具，如下所示：

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group

Note: This will only show information about consumers that use the Java consumer API (non-ZooKeeper-based consumers).

TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG        CONSUMER-ID                                       HOST                           CLIENT-ID
my-topic                       0          2               4               2          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
my-topic                       1          2               3               1          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
my-topic                       2          2               3               1          consumer-2-42c1abd4-e3b2-425d-a8bb-e1ea49b29bb2   /127.0.0.1                     consumer-2
```

This tool also works with ZooKeeper-based consumers:

该工具也适用于基于ZooKeeper的消费者：

```bash
> bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --describe --group my-group

Note: This will only show information about consumers that use ZooKeeper (not those using the Java consumer API).

TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG        CONSUMER-ID
my-topic                       0          2               4               2          my-group_consumer-1
my-topic                       1          2               3               1          my-group_consumer-1
my-topic                       2          2               3               1          my-group_consumer-2
```

## Managing Consumer Groups

With the ConsumerGroupCommand tool, we can list, describe, or delete consumer groups. When using the new consumer API (where the broker handles coordination of partition handling and rebalance), the group can be deleted manually, or automatically when the last committed offset for that group expires. Manual deletion works only if the group does not have any active members. For example, to list all consumer groups across all topics:

通过ConsumerGroupCommand工具，我们可以列出，描述或删除使用者群组。 当使用新的客户API（代理处理分区处理和重新平衡的协调）时，可以手动删除该组，也可以在该组最后一次提交的偏移过期时自动删除。 手动删除仅在该组没有任何活动成员时才起作用。 例如，要列出所有主题中的所有消费者组：

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

test-consumer-group
```

To view offsets, as mentioned earlier, we "describe" the consumer group like this:

为了查看偏移量，如前所述，我们“描述”这样的消费者组：

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group

TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                    HOST            CLIENT-ID
topic3          0          241019          395308          154289          consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2
topic2          1          520678          803288          282610          consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2
topic3          1          241018          398817          157799          consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2
topic1          0          854144          855809          1665            consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1
topic2          0          460537          803290          342753          consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1
topic3          2          243655          398812          155157          consumer4-117fe4d3-c6c1-4178-8ee9-eb4a3954bee0 /127.0.0.1      consumer4
```

There are a number of additional "describe" options that can be used to provide more detailed information about a consumer group that uses the new consumer API:

还有一些额外的“描述”选项可用于提供有关使用新消费者API的消费者组的更多详细信息：

* --members: This option provides the list of all active members in the consumer group.


* --members: 该选项提供了用户组中所有活动成员的列表。

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group --members

CONSUMER-ID                                    HOST            CLIENT-ID       #PARTITIONS
consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1       2
consumer4-117fe4d3-c6c1-4178-8ee9-eb4a3954bee0 /127.0.0.1      consumer4       1
consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2       3
consumer3-ecea43e4-1f01-479f-8349-f9130b75d8ee /127.0.0.1      consumer3       0
```

* --members --verbose: On top of the information reported by the "--members" options above, this option also provides the partitions assigned to each member.


* --members --verbose：除上述“--members”选项报告的信息之外，此选项还提供分配给每个成员的分区。

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group --members --verbose

CONSUMER-ID                                    HOST            CLIENT-ID       #PARTITIONS     ASSIGNMENT
consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1       2               topic1(0), topic2(0)
consumer4-117fe4d3-c6c1-4178-8ee9-eb4a3954bee0 /127.0.0.1      consumer4       1               topic3(2)
consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2       3               topic2(1), topic3(0,1)
consumer3-ecea43e4-1f01-479f-8349-f9130b75d8ee /127.0.0.1      consumer3       0               -
```

* --offsets: This is the default describe option and provides the same output as the "--describe" option.

  --offsets: 这是默认描述选项，并提供与“--describe”选项相同的输出。

* --state: This option provides useful group-level information.

  --state: 该选项提供有用的组级信息。

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group --state

COORDINATOR (ID)          ASSIGNMENT-STRATEGY       STATE                #MEMBERS
localhost:9092 (0)        range                     Stable               4
```

To manually delete one or multiple consumer groups, the "--delete" option can be used:

要手动删除一个或多个用户组，可以使用“--delete”选项：

```bash
> bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --delete --group my-group --group my-other-group

Note: This will not show information about old Zookeeper-based consumers.
Deletion of requested consumer groups ('my-group', 'my-other-group') was successful.
```

If you are using the old high-level consumer and storing the group metadata in ZooKeeper (i.e. ```offsets.storage=zookeeper```), pass ```--zookeeper``` instead of ```bootstrap-server```:

如果您正在使用旧的高级使用者并在ZooKeeper中存储组元数据（即，```offsets.storage=zookeeper```），请传递```--zookeeper```而不是```bootstrap-server```：

```bash
> bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list
```

## Expanding your cluster

Adding servers to a Kafka cluster is easy, just assign them a unique broker id and start up Kafka on your new servers. However these new servers will not automatically be assigned any data partitions, so unless partitions are moved to them they won't be doing any work until new topics are created. So usually when you add machines to your cluster you will want to migrate some existing data to these machines.

将服务器添加到Kafka集群非常简单，只需为其分配唯一的代理ID并在新服务器上启动Kafka即可。 但是，这些新服务器不会自动分配任何数据分区，因此除非将分区移动到其中，否则在创建新主题之前它们将不会执行任何工作。 所以通常当您将机器添加到群集中时，您会希望将一些现有数据迁移到这些机器上。

The process of migrating data is manually initiated but fully automated. Under the covers what happens is that Kafka will add the new server as a follower of the partition it is migrating and allow it to fully replicate the existing data in that partition. When the new server has fully replicated the contents of this partition and joined the in-sync replica one of the existing replicas will delete their partition's data.

迁移数据的过程是手动启动的，但是完全自动化。 在封面之下发生的是，Kafka会将新服务器添加为正在迁移的分区的跟随者，并允许它完全复制该分区中的现有数据。 当新服务器完全复制了此分区的内容并加入了同步副本时，其中一个现有副本将删除其分区的数据。

The partition reassignment tool can be used to move partitions across brokers. An ideal partition distribution would ensure even data load and partition sizes across all brokers. The partition reassignment tool does not have the capability to automatically study the data distribution in a Kafka cluster and move partitions around to attain an even load distribution. As such, the admin has to figure out which topics or partitions should be moved around.

分区重新分配工具可用于跨代理移动分区。 理想的分区分布将确保所有代理的数据加载和分区大小。 分区重新分配工具没有能力自动研究Kafka集群中的数据分布，并且可以移动分区以获得均匀的负载分布。 因此，管理员必须找出哪些主题或分区应该移动。

The partition reassignment tool can run in 3 mutually exclusive modes:

分区重新分配工具可以以3种互斥方式运行：

* --generate: In this mode, given a list of topics and a list of brokers, the tool generates a candidate reassignment to move all partitions of the specified topics to the new brokers. This option merely provides a convenient way to generate a partition reassignment plan given a list of topics and target brokers.
* --execute: In this mode, the tool kicks off the reassignment of partitions based on the user provided reassignment plan. (using the --reassignment-json-file option). This can either be a custom reassignment plan hand crafted by the admin or provided by using the --generate option
* --verify: In this mode, the tool verifies the status of the reassignment for all partitions listed during the last --execute. The status can be either of successfully completed, failed or in progress


* --generate: 在此模式下，给定主题列表和经纪人列表，该工具会生成候选重新分配以将指定主题的所有分区移至新经纪人。 此选项仅提供了一种便捷方式，可以根据主题和目标代理列表生成分区重新分配计划。
* --execute: 在此模式下，该工具根据用户提供的重新分配计划启动分区重新分配。 （使用--reassignment-json-file选项）。 这可以是由管理员制作的自定义重新分配计划，也可以是使用--generate选项提供的自定义重新分配计划
* --verify: 在此模式下，该工具验证上次执行期间列出的所有分区的重新分配状态。 状态可以是成功完成，失败或正在进行

## Automatically migrating data to new machines

The partition reassignment tool can be used to move some topics off of the current set of brokers to the newly added brokers. This is typically useful while expanding an existing cluster since it is easier to move entire topics to the new set of brokers, than moving one partition at a time. When used to do this, the user should provide a list of topics that should be moved to the new set of brokers and a target list of new brokers. The tool then evenly distributes all partitions for the given list of topics across the new set of brokers. During this move, the replication factor of the topic is kept constant. Effectively the replicas for all partitions for the input list of topics are moved from the old set of brokers to the newly added brokers.

分区重新分配工具可用于将当前一组经纪人的一些主题移至新增的经纪人。 这在扩展现有群集时通常很有用，因为将整个主题移动到新代理集比移动一个分区更容易。 用于这样做时，用户应该提供应该移动到新的经纪人集合和新经纪人目标列表的主题列表。 该工具然后均匀分配给新代理集中的给定主题列表的所有分区。 在此过程中，该主题的复制因子保持不变。 实际上，输入主题列表的所有分区的副本都从旧的代理集合移动到新添加的代理。

For instance, the following example will move all partitions for topics foo1,foo2 to the new set of brokers 5,6. At the end of this move, all partitions for topics foo1 and foo2 will only exist on brokers 5,6.

例如，以下示例将把主题foo1，foo2的所有分区移至新的代理集5,6。 在本次移动结束时，主题foo1和foo2的所有分区将仅存在于代理5,6上。

Since the tool accepts the input list of topics as a json file, you first need to identify the topics you want to move and create the json file as follows:

由于该工具接受主题的输入列表作为json文件，因此首先需要确定要移动的主题并创建json文件，如下所示：

```bash
> cat topics-to-move.json
{"topics": [{"topic": "foo1"},
            {"topic": "foo2"}],
"version":1
}
```

Once the json file is ready, use the partition reassignment tool to generate a candidate assignment:

一旦json文件准备就绪，请使用分区重新分配工具来生成候选分配：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --topics-to-move-json-file topics-to-move.json --broker-list "5,6" --generate
Current partition replica assignment

{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
              {"topic":"foo1","partition":0,"replicas":[3,4]},
              {"topic":"foo2","partition":2,"replicas":[1,2]},
              {"topic":"foo2","partition":0,"replicas":[3,4]},
              {"topic":"foo1","partition":1,"replicas":[2,3]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}

Proposed partition reassignment configuration

{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
              {"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":2,"replicas":[5,6]},
              {"topic":"foo2","partition":0,"replicas":[5,6]},
              {"topic":"foo1","partition":1,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[5,6]}]
}
```

The tool generates a candidate assignment that will move all partitions from topics foo1,foo2 to brokers 5,6. Note, however, that at this point, the partition movement has not started, it merely tells you the current assignment and the proposed new assignment. The current assignment should be saved in case you want to rollback to it. The new assignment should be saved in a json file (e.g. expand-cluster-reassignment.json) to be input to the tool with the --execute option as follows:

该工具会生成一个候选分配，将所有分区从主题foo1，foo2移动到经纪人5,6。 但是，请注意，此时分区移动尚未开始，它只会告诉您当前分配和建议的新分配。 当您想要回滚到当前分配时，应保存该分配。 新的任务应该保存在一个json文件中（例如expand-cluster-reassignment.json），并使用--execute选项输入到工具中，如下所示：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file expand-cluster-reassignment.json --execute
Current partition replica assignment

{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
              {"topic":"foo1","partition":0,"replicas":[3,4]},
              {"topic":"foo2","partition":2,"replicas":[1,2]},
              {"topic":"foo2","partition":0,"replicas":[3,4]},
              {"topic":"foo1","partition":1,"replicas":[2,3]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
              {"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":2,"replicas":[5,6]},
              {"topic":"foo2","partition":0,"replicas":[5,6]},
              {"topic":"foo1","partition":1,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[5,6]}]
}
```

Finally, the --verify option can be used with the tool to check the status of the partition reassignment. Note that the same expand-cluster-reassignment.json (used with the --execute option) should be used with the --verify option:

最后，可以使用--verify选项和工具来检查分区重新分配的状态。 请注意，相同的expand-cluster-reassignment.json（与--execute选项一起使用）应与--verify选项一起使用：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file expand-cluster-reassignment.json --verify
Status of partition reassignment:
Reassignment of partition [foo1,0] completed successfully
Reassignment of partition [foo1,1] is in progress
Reassignment of partition [foo1,2] is in progress
Reassignment of partition [foo2,0] completed successfully
Reassignment of partition [foo2,1] completed successfully
Reassignment of partition [foo2,2] completed successfully
```

## Custom partition assignment and migration

The partition reassignment tool can also be used to selectively move replicas of a partition to a specific set of brokers. When used in this manner, it is assumed that the user knows the reassignment plan and does not require the tool to generate a candidate reassignment, effectively skipping the --generate step and moving straight to the --execute step

分区重新分配工具也可用于选择性地将分区的副本移动到特定的一组代理。 当以这种方式使用时，假定用户知道重新分配计划并且不需要工具产生候选重新分配，有效地跳过 - 生成步骤并直接移动到 - 执行步骤

For instance, the following example moves partition 0 of topic foo1 to brokers 5,6 and partition 1 of topic foo2 to brokers 2,3:

例如，以下示例将主题foo1的分区0移动到代理5,6并将主题foo2的分区1移动到代理2,3：

The first step is to hand craft the custom reassignment plan in a json file:

第一步是在json文件中手工制作自定义重新分配计划：

```bash
> cat custom-reassignment.json
{"version":1,"partitions":[{"topic":"foo1","partition":0,"replicas":[5,6]},{"topic":"foo2","partition":1,"replicas":[2,3]}]}
```

Then, use the json file with the --execute option to start the reassignment process:

然后，使用带有--execute选项的json文件来启动重新分配过程：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file custom-reassignment.json --execute
Current partition replica assignment

{"version":1,
"partitions":[{"topic":"foo1","partition":0,"replicas":[1,2]},
              {"topic":"foo2","partition":1,"replicas":[3,4]}]
}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}
```

The --verify option can be used with the tool to check the status of the partition reassignment. Note that the same expand-cluster-reassignment.json (used with the --execute option) should be used with the --verify option:

他可以使用--verify选项来检查分区重新分配的状态。 请注意，相同的expand-cluster-reassignment.json（与--execute选项一起使用）应与--verify选项一起使用：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file custom-reassignment.json --verify
Status of partition reassignment:
Reassignment of partition [foo1,0] completed successfully
Reassignment of partition [foo2,1] completed successfully
```

## Decommissioning brokers

The partition reassignment tool does not have the ability to automatically generate a reassignment plan for decommissioning brokers yet. As such, the admin has to come up with a reassignment plan to move the replica for all partitions hosted on the broker to be decommissioned, to the rest of the brokers. This can be relatively tedious as the reassignment needs to ensure that all the replicas are not moved from the decommissioned broker to only one other broker. To make this process effortless, we plan to add tooling support for decommissioning brokers in the future.

分区重新分配工具不具备为退役代理自动生成重新分配计划的功能。 因此，管理员必须制定重新分配计划，将代理上托管的所有分区的副本移至其他代理。 这可能比较单调，因为重新分配需要确保所有副本不会从退役经纪人转移到另一个经纪人。 为了使这一过程毫不费力，我们计划在未来为退役经纪人添加工具支持。

## Increasing replication factor

Increasing the replication factor of an existing partition is easy. Just specify the extra replicas in the custom reassignment json file and use it with the --execute option to increase the replication factor of the specified partitions.

增加现有分区的复制因子很容易。 只需在自定义重新分配json文件中指定额外副本，并将其与--execute选项一起使用即可增加指定分区的复制因子。

For instance, the following example increases the replication factor of partition 0 of topic foo from 1 to 3. Before increasing the replication factor, the partition's only replica existed on broker 5. As part of increasing the replication factor, we will add more replicas on brokers 6 and 7.

例如，以下示例将主题foo的分区0的复制因子从1增加到3.在增加复制因子之前，该分区的唯一副本存在于代理5上。作为增加复制因子的一部分，我们将添加更多副本 经纪人6和7。

The first step is to hand craft the custom reassignment plan in a json file:

第一步是在json文件中手工制作自定义重新分配计划：

```bash
> cat increase-replication-factor.json
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5,6,7]}]}
```

Then, use the json file with the --execute option to start the reassignment process:

然后，使用带有--execute选项的json文件来启动重新分配过程：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --execute
Current partition replica assignment

{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5,6,7]}]}
```

The --verify option can be used with the tool to check the status of the partition reassignment. Note that the same increase-replication-factor.json (used with the --execute option) should be used with the --verify option:

--verify选项可与该工具一起使用，以检查分区重新分配的状态。 请注意，与--verify选项一起使用相同的increase-replication-factor.json（与--execute选项一起使用）：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --verify
Status of partition reassignment:
Reassignment of partition [foo,0] completed successfully
```

You can also verify the increase in replication factor with the kafka-topics tool:

您还可以使用kafka-topics工具验证复制因子的增加情况：

```bash
> bin/kafka-topics.sh --zookeeper localhost:2181 --topic foo --describe
Topic:foo   PartitionCount:1    ReplicationFactor:3 Configs:
  Topic: foo    Partition: 0    Leader: 5   Replicas: 5,6,7 Isr: 5,6,7
```

## Limiting Bandwidth Usage during Data Migration

Kafka lets you apply a throttle to replication traffic, setting an upper bound on the bandwidth used to move replicas from machine to machine. This is useful when rebalancing a cluster, bootstrapping a new broker or adding or removing brokers, as it limits the impact these data-intensive operations will have on users.

Kafka允许您将复制流量应用于复制流量，设置用于将复制副本从机器移动到机器的带宽的上限。 在重新平衡群集，引导新代理或添加或删除代理时，这非常有用，因为它限制了这些数据密集型操作对用户的影响。

There are two interfaces that can be used to engage a throttle. The simplest, and safest, is to apply a throttle when invoking the kafka-reassign-partitions.sh, but kafka-configs.sh can also be used to view and alter the throttle values directly.

有两个接口可以用来连接油门。 最简单也是最安全的是在调用kafka-reassign-partitions.sh时应用节流阀，但也可以使用kafka-configs.sh直接查看和更改节流阀值。

So for example, if you were to execute a rebalance, with the below command, it would move partitions at no more than 50MB/s.

例如，如果要使用下面的命令执行重新平衡，它将以不超过50MB / s的速度移动分区。

```bash
$ bin/kafka-reassign-partitions.sh --zookeeper myhost:2181--execute --reassignment-json-file bigger-cluster.json —throttle 50000000
```

When you execute this script you will see the throttle engage:

当你执行这个脚本时，你会看到油门启动：

```bash
The throttle limit was set to 50000000 B/s
Successfully started reassignment of partitions.
```

Should you wish to alter the throttle, during a rebalance, say to increase the throughput so it completes quicker, you can do this by re-running the execute command passing the same reassignment-json-file:

如果你希望改变节流阀，在重新平衡期间，比如增加吞吐量以便它更快完成，你可以通过重新运行execute命令来传递同样的reassignment-json-file来实现：

```bash
$ bin/kafka-reassign-partitions.sh --zookeeper localhost:2181  --execute --reassignment-json-file bigger-cluster.json --throttle 700000000
  There is an existing assignment running.
  The throttle limit was set to 700000000 B/s
```

Once the rebalance completes the administrator can check the status of the rebalance using the --verify option. If the rebalance has completed, the throttle will be removed via the --verify command. It is important that administrators remove the throttle in a timely manner once rebalancing completes by running the command with the --verify option. Failure to do so could cause regular replication traffic to be throttled.

重新平衡完成后，管理员可以使用--verify选项检查重新平衡的状态。 如果重新平衡完成，油门将通过--verify命令删除。 通过使用--verify选项运行该命令后，重新平衡完成后，管理员必须及时删除节流阀。 如果不这样做可能会导致定期复制流量受到限制。

When the --verify option is executed, and the reassignment has completed, the script will confirm that the throttle was removed:

当执行--verify选项并且重新分配完成时，脚本将确认已删除油门：

```bash
> bin/kafka-reassign-partitions.sh --zookeeper localhost:2181  --verify --reassignment-json-file bigger-cluster.json
Status of partition reassignment:
Reassignment of partition [my-topic,1] completed successfully
Reassignment of partition [mytopic,0] completed successfully
Throttle was removed.
```

The administrator can also validate the assigned configs using the kafka-configs.sh. There are two pairs of throttle configuration used to manage the throttling process. The throttle value itself. This is configured, at a broker level, using the dynamic properties:

管理员还可以使用kafka-configs.sh验证分配的配置。 有两对节流阀配置用于管理节流过程。 油门值本身。 这是在代理级别使用动态属性进行配置的：

```bash
leader.replication.throttled.rate
  follower.replication.throttled.rate
```

There is also an enumerated set of throttled replicas:

还有一组枚举的复制副本：

```bash
leader.replication.throttled.replicas
  follower.replication.throttled.replicas
```

Which are configured per topic. All four config values are automatically assigned by kafka-reassign-partitions.sh (discussed below).

每个主题配置了哪些。 所有四个配置值都由kafka-reassign-partitions.sh自动分配（如下所述）。

To view the throttle limit configuration:

要查看油门限制配置：

```bash
> bin/kafka-configs.sh --describe --zookeeper localhost:2181 --entity-type brokers
Configs for brokers '2' are leader.replication.throttled.rate=700000000,follower.replication.throttled.rate=700000000
Configs for brokers '1' are leader.replication.throttled.rate=700000000,follower.replication.throttled.rate=700000000
```

This shows the throttle applied to both leader and follower side of the replication protocol. By default both sides are assigned the same throttled throughput value.

这显示应用于复制协议的领导者和追随者一方的节流阀。 默认情况下，双方都被分配相同的限制吞吐量值。

To view the list of throttled replicas:

要查看节流副本的列表，请执行以下操作：

```bash
> bin/kafka-configs.sh --describe --zookeeper localhost:2181 --entity-type topics
Configs for topic 'my-topic' are leader.replication.throttled.replicas=1:102,0:101,
    follower.replication.throttled.replicas=1:101,0:102
```

Here we see the leader throttle is applied to partition 1 on broker 102 and partition 0 on broker 101. Likewise the follower throttle is applied to partition 1 on broker 101 and partition 0 on broker 102.

这里我们看到领导者节流器被应用于代理102上的分区1和代理101上的分区0.同样，跟随者节流器被应用于代理101上的分区1和代理102上的分区0。

By default kafka-reassign-partitions.sh will apply the leader throttle to all replicas that exist before the rebalance, any one of which might be leader. It will apply the follower throttle to all move destinations. So if there is a partition with replicas on brokers 101,102, being reassigned to 102,103, a leader throttle, for that partition, would be applied to 101,102 and a follower throttle would be applied to 103 only.

默认情况下，kafka-reassign-partitions.sh将把leader leader应用于重新平衡之前存在的所有副本，其中任何一个都可能是领导者。 它会将跟随器油门应用到所有移动目的地。 因此，如果在经纪101,102上存在具有副本的分区，被重新分配给102,103，那么该分区的领导节流将应用于101,102，并且随从节流将仅应用于103。

If required, you can also use the --alter switch on kafka-configs.sh to alter the throttle configurations manually.

如果需要，还可以使用kafka-configs.sh上的--alter开关手动更改节流阀配置。

### Safe usage of throttled replication

Some care should be taken when using throttled replication. In particular:

在使用节制复制时应该小心。 尤其是：

(1) Throttle Removal:

The throttle should be removed in a timely manner once reassignment completes (by running kafka-reassign-partitions —verify).
(2) Ensuring Progress:

If the throttle is set too low, in comparison to the incoming write rate, it is possible for replication to not make progress. This occurs when:

```
max(BytesInPerSec) > throttle
```

Where BytesInPerSec is the metric that monitors the write throughput of producers into each broker.

The administrator can monitor whether replication is making progress, during the rebalance, using the metric:

```
kafka.server:type=FetcherLagMetrics,name=ConsumerLag,clientId=([-.\w]+),topic=([-.\w]+),partition=([0-9]+)
```

The lag should constantly decrease during replication. If the metric does not decrease the administrator should increase the throttle throughput as described above.

## Setting quotas

Quotas overrides and defaults may be configured at (user, client-id), user or client-id levels as described here. By default, clients receive an unlimited quota. It is possible to set custom quotas for each (user, client-id), user or client-id group.

配额覆盖和默认值可以在（用户，客户端ID），用户或客户端级别进行配置，如此处所述。 默认情况下，客户端会收到无限制的配额。 可以为每个（用户，客户端ID），用户或客户端ID组设置自定义配额。

Configure custom quota for (user=user1, client-id=clientA):

为（user = user1，client-id = clientA）配置自定义配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-name user1 --entity-type clients --entity-name clientA
Updated config for entity: user-principal 'user1', client-id 'clientA'.
```

Configure custom quota for user=user1:

为user=user1配置自定义配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-name user1
Updated config for entity: user-principal 'user1'.
```

Configure custom quota for client-id=clientA:

为client-id=clientA配置自定义配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type clients --entity-name clientA
Updated config for entity: client-id 'clientA'.
```

It is possible to set default quotas for each (user, client-id), user or client-id group by specifying --entity-default option instead of --entity-name.

可以通过指定--entity-default选项而不是--entity-name来为每个（用户，客户端ID），用户或客户端ID组设置默认配额。

Configure default client-id quota for user=userA:

为user = userA配置默认客户端配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-name user1 --entity-type clients --entity-default
Updated config for entity: user-principal 'user1', default client-id.
```

Configure default quota for user:

为用户配置默认配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-default
Updated config for entity: default user-principal.
```

Configure default quota for client-id:

配置client-id的默认配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type clients --entity-default
Updated config for entity: default client-id.
```

Here's how to describe the quota for a given (user, client-id):

以下是如何描述给定（用户，客户端ID）的配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users --entity-name user1 --entity-type clients --entity-name clientA
Configs for user-principal 'user1', client-id 'clientA' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200\
```

Describe quota for a given user:

描述给定用户的配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users --entity-name user1
Configs for user-principal 'user1' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```

Describe quota for a given client-id:

描述给定客户端ID的配额：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type clients --entity-name clientA
Configs for client-id 'clientA' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```

If entity name is not specified, all entities of the specified type are described. For example, describe all users:

如果未指定实体名称，则说明指定类型的所有实体。 例如，描述所有用户：

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users
Configs for user-principal 'user1' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
Configs for default user-principal are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```

Similarly for (user, client):

同样的（user，client）

```bash
> bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users --entity-type clients
Configs for user-principal 'user1', default client-id are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
Configs for user-principal 'user1', client-id 'clientA' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```

It is possible to set default quotas that apply to all client-ids by setting these configs on the brokers. These properties are applied only if quota overrides or defaults are not configured in Zookeeper. By default, each client-id receives an unlimited quota. The following sets the default quota per producer and consumer client-id to 10MB/sec.

通过在代理上设置这些配置，可以设置适用于所有客户端ID的默认配额。 只有在Zookeeper中未配置配额覆盖或默认配置时才应用这些属性。 默认情况下，每个客户端ID都会收到一个无限制的配额。 以下设置每个生产者和消费者客户端的默认配额为10MB /秒。

```bash
quota.producer.default=10485760
quota.consumer.default=10485760
```

Note that these properties are being deprecated and may be removed in a future release. Defaults configured using kafka-configs.sh take precedence over these properties.

请注意，这些属性已被弃用，并可能在未来版本中删除。 使用kafka-configs.sh配置的默认值优先于这些属性。