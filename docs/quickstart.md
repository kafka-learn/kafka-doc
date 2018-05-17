
# quickstart

# 快速开始

This tutorial assumes you are starting fresh and have no existing Kafka or ZooKeeper data. Since Kafka console scripts are different for Unix-based and Windows platforms, on Windows platforms use```bin\windows\``` instead of ```bin/```, and change the script extension to ```.bat```.

本教程假定您刚开始使用Kafka，并且本地不存在Kafka和ZooKeeper环境。由于基于Unix的平台和Windows平台的Kafka控制台脚本是不同的，所以在Windows平台上使用```bin\windows\```而不是```bin/```，并且需要将脚本扩展名更改为```.bat```。

## Step 1: Download the code

## 第一步：下载Kafka代码

[Download](https://www.apache.org/dyn/closer.cgi?path=/kafka/1.1.0/kafka_2.11-1.1.0.tgz) the 1.1.0 release and un-tar it.

[下载](https://www.apache.org/dyn/closer.cgi?path=/kafka/1.1.0/kafka_2.11-1.1.0.tgz) 1.1.0 release版本，并解压它

```bash
> tar -xzf kafka_2.11-1.1.0.tgz
> cd kafka_2.11-1.1.0
```

## Step 2: Start the server

## 第二步：启动Kafka服务端

Kafka uses [ZooKeeper](https://zookeeper.apache.org/) so you need to first start a ZooKeeper server if you don't already have one. You can use the convenience script packaged with kafka to get a quick-and-dirty single-node ZooKeeper instance.

Kafka使用了[ZooKeeper](https://zookeeper.apache.org/)，因此如果您还没有ZooKeeper服务器，您需要首先启动一个ZooKeeper服务器。您可以使用与kafka打包在一起的便捷脚本来获得快速且简单的单节点（single-node）ZooKeeper实例。


```bash
> bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```

Now start the Kafka server:

然后开启Kafka服务：

```bash
> bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```

## Step 3: Create a topic

## 第三步：创建主题

Let's create a topic named "test" with a single partition and only one replica:

让我们创建一个名称为“test”，带有一个分区(partion)，且只有一个replica(备份)的主题：

```bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

We can now see that topic if we run the list topic command:

我们可以通过运行查看主题列表的命令来看到该主题：

```bash
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```

Alternatively, instead of manually creating topics you can also configure your brokers to auto-create topics when a non-existent topic is published to.

另外，在发布一个不存在的主题时，您也可以通过配置代理(broker)来实现自动创建主题，而不是手动创建主题。

## Step 4: Send some messages

## 第四步：发送一些消息

Kafka comes with a command line client that will take input from a file or from standard input and send it out as messages to the Kafka cluster. By default, each line will be sent as a separate message.

Kafka带有一个命令行客户端，它将从一个文件或标准输入中获取输入，并将其作为消息发送到Kafka集群。默认情况下，每行将作为一个单独的消息发送。

Run the producer and then type a few messages into the console to send to the server.

运行生产者，然后在控制台中输入一些消息以发送到服务器。

```bash
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

## Step 5: Start a consumer

## 第五步：开启一个消费者

Kafka also has a command line consumer that will dump out messages to standard output.

Kafka也有一个命令行消费者，能将消息转储到标准输出。

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```

If you have each of the above commands running in a different terminal then you should now be able to type messages into the producer terminal and see them appear in the consumer terminal.

如果您将上述每个命令都在不同的终端中运行，那么您现在应该能够将消息键入生产者终端，并将它们显示在消费者终端中。

All of the command line tools have additional options; running the command with no arguments will display usage information documenting them in more detail.

所有的命令行工具都有附加的选项; 在没有参数的情况下运行这些命令将更详细地显示它们的使用信息。

## Step 6: Setting up a multi-broker cluster

## 第六步：设置多代理集群

So far we have been running against a single broker, but that's no fun. For Kafka, a single broker is just a cluster of size one, so nothing much changes other than starting a few more broker instances. But just to get feel for it, let's expand our cluster to three nodes (still all on our local machine).

到目前为止，我们都在单个代理中运行，但这不太有趣。对于Kafka来说，单代理即一个节点的集群，所以没有太大的变化，除非启动多个代理实例。但为了体验下多代理，我们将集群扩展为三个节点（仍然都在本地机器上）。

First we make a config file for each of the brokers (on Windows use the ```copy``` command instead):

首先，我们为每个代理创建一个配置文件(在Windows上使用```copy```命令)：

```bash
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
```

Now edit these new files and set the following properties:

修改其中的配置内容：

```
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dir=/tmp/kafka-logs-1
```

```
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dir=/tmp/kafka-logs-2
```

The broker.id property is the unique and permanent name of each node in the cluster. We have to override the port and log directory only because we are running these all on the same machine and we want to keep the brokers from all trying to register on the same port or overwrite each other's data.

broker.id 属性是集群中每个节点的唯一且永久的名称。我们还必须覆盖端口和日志目录，因为我们在同一台机器上运行这些端口和日志目录，并且我们希望让所有代理都试图在同一个端口注册或覆盖彼此的数据。

We already have Zookeeper and our single node started, so we just need to start the two new nodes:

我们已经配置了Zookeeper并且启动了我们的单个服务器节点，所以我们只需要启动两个新节点：

```bash
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```

Now create a new topic with a replication factor of three:

现在创建一个带有三个备份因子的新主题：

```bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

Okay but now that we have a cluster how can we know which broker is doing what? To see that run the "describe topics" command:

好吧，但现在我们有一个集群，我们怎么知道哪个代理在做什么？这需要运行“描述主题”命令去查看：

```bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```

Here is an explanation of output. The first line gives a summary of all the partitions, each additional line gives information about one partition. Since we have only one partition for this topic there is only one line.

这里对输出进行解释。第一行给出所有分区的摘要信息，每个附加行提供有关一个分区的信息。由于我们只有一个分区，所以附加行只有一行。

* "leader" is the node responsible for all reads and writes for the given partition. Each node will be the leader for a randomly selected portion of the partitions.

    “leader”是负责给定分区的所有读写操作的节点。每个节点将随机的被选择成为某些分区的leader。

* "replicas" is the list of nodes that replicate the log for this partition regardless of whether they are the leader or even if they are currently alive.

    “replicas”是备份了此分区日志的节点列表，无论其是leader，还是当前仍存活着的。

* "isr" is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader.
    
    “isr”是一组“同步”的replicas。这是replicas列表的子集，其是目前仍存活着的并被指引了leader。

Note that in my example node 1 is the leader for the only partition of the topic.

请注意，在我的示例中，节点1是该主题唯一分区的leader。

We can run the same command on the original topic we created to see where it is:

我们可以在最开始创建的那个主题上运行同样的命令，以查看它的信息：

```bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```

So there is no surprise there—the original topic has no replicas and is on server 0, the only server in our cluster when we created it.

所以在这里并不奇怪 - 最开始的那个主题没有replicas，其在服务器0上，该服务器是我们创建集群时唯一的服务器。

Let's publish a few messages to our new topic:

让我们向新主题发布一些消息：

```bash
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

Now let's consume these messages:

现在让我们来消费这些消息：

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

Now let's test out fault-tolerance. Broker 1 was acting as the leader so let's kill it:

现在我们来测试容错性。代理1充当了leader，所以让我们杀死它：

```bash
> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
```

Leadership has switched to one of the slaves and node 1 is no longer in the in-sync replica set:

leader已切换成其中一个从属节点了，并且节点1不再处于同步replicas集中：

```bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```

But the messages are still available for consumption even though the leader that took the writes originally is down:

即使原先的leader写入失败，这些消息仍然可用于消费：

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

## Step 7: Use Kafka Connect to import/export data

## 第七步: 使用Kafka Connect来导入/导出数据

Writing data from the console and writing it back to the console is a convenient place to start, but you'll probably want to use data from other sources or export data from Kafka to other systems. For many systems, instead of writing custom integration code you can use Kafka Connect to import or export data.

从控制台写入数据并将其写回控制台是一个简单的开始，但您可能需要使用其它来源的数据或将数据从Kafka导出到其他系统。对于许多系统，您可以使用Kafka Connect导入或导出数据，而不是编写自定义集成代码。

Kafka Connect is a tool included with Kafka that imports and exports data to Kafka. It is an extensible tool that runs connectors, which implement the custom logic for interacting with an external system. In this quickstart we'll see how to run Kafka Connect with simple connectors that import data from a file to a Kafka topic and export data from a Kafka topic to a file.

Kafka Connect是Kafka附带的一个工具，可以将数据导入和导出到Kafka。它是一个可扩展的工具，其运行connector，实现了与外部系统交互的自定义逻辑。在此快速入门中，我们将看到如何使用简单的connector运行Kafka Connect，这些connector将数据从文件导入到Kafka主题，并将数据从Kafka主题导出到文件。

First, we'll start by creating some seed data to test with:

首先，我们将通过创建一些种子数据来开始测试：

```bash
echo -e "foo\nbar" > test.txt
```

Or on Windows:

或者在Window系统上：

```bash
> echo foo> test.txt
> echo bar>> test.txt
```

Next, we'll start two connectors running in standalone mode, which means they run in a single, local, dedicated process. We provide three configuration files as parameters. The first is always the configuration for the Kafka Connect process, containing common configuration such as the Kafka brokers to connect to and the serialization format for data. The remaining configuration files each specify a connector to create. These files include a unique connector name, the connector class to instantiate, and any other configuration required by the connector.

接下来，我们将启动两个以独立模式运行的Connector，这意味着它们将在单个、本地的专用进程中运行。我们提供三个配置文件作为参数。首先是Kafka Connect过程的配置，包含常见的配置，例如要连接的Kafka代理和数据的序列化格式。其余的配置文件都指定了要创建的connector。这些文件包括唯一的connector名称，要实例化的connector类以及connector所需的任何其他配置。

```bash
> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

These sample configuration files, included with Kafka, use the default local cluster configuration you started earlier and create two connectors: the first is a source connector that reads lines from an input file and produces each to a Kafka topic and the second is a sink connector that reads messages from a Kafka topic and produces each as a line in an output file.

Kafka附带的这些示例配置文件使用您之前启动的默认本地群集配置，并创建两个Connector：第一个是source connector，从输入文件中读取行，并将数据生产到Kafka主题，第二个Connector为sink connector，它读取来自Kafka主题的消息，并在输出文件中将每个消息生成为一行。

During startup you'll see a number of log messages, including some indicating that the connectors are being instantiated. Once the Kafka Connect process has started, the source connector should start reading lines from ```test.txt``` and producing them to the topic ```connect-test```, and the sink connector should start reading messages from the topic ```connect-test``` and write them to the file ```test.sink.txt```. We can verify the data has been delivered through the entire pipeline by examining the contents of the output file:

在启动过程中，您会看到许多日志消息，包括一些指示Connector正在实例化的消息。一旦Kafka Connect进程启动，source connector会从```test.txt```开始读取行并将其生成到主题```connect-test```，并且sink connector会开始读取主题```connect-test```中的消息并将它们写入文件```test.sink.txt```。我们可以通过检查输出文件的内容来验证通过整个数据管道的传输：

```bash
> more test.sink.txt
foo
bar
```

Note that the data is being stored in the Kafka topic ```connect-test```, so we can also run a console consumer to see the data in the topic (or use custom consumer code to process it):

注意到数据存储在Kafka主题```connect-test```中，因此我们还可以运行消费者控制台以查看主题中的数据（或使用自定义的消费者代码来处理它）：

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
```

The connectors continue to process data, so we can add data to the file and see it move through the pipeline:

connector继续处理数据，所以我们可以将数据添加到文件中，并能看到它在整个管道中的移动：

```bash
> echo Another line>> test.txt
```

You should see the line appear in the console consumer output and in the sink file.

您可以看到该行出现在消费者控制台和接收文件中。

## Step 8: Use Kafka Streams to process data

## 第八步：利用Kafka Streams处理数据

Kafka Streams is a client library for building mission-critical real-time applications and microservices, where the input and/or output data is stored in Kafka clusters. Kafka Streams combines the simplicity of writing and deploying standard Java and Scala applications on the client side with the benefits of Kafka's server-side cluster technology to make these applications highly scalable, elastic, fault-tolerant, distributed, and much more. This [quickstart example](http://kafka.apache.org/11/documentation/streams/quickstart) will demonstrate how to run a streaming application coded in this library.

Kafka Streams是一个用于构建关键任务实时应用程序和微服务的客户端库，输入、输出数据存储在Kafka集群中。Kafka Streams结合了在客户端编写和部署标准Java和Scala应用程序的简单性以及Kafka服务器端集群技术的优势，使这些应用程序具有高度可伸缩性、弹性、容错性、分布式等特性。本[快速入门示例](./documentations/kafka_streams/run_demo_app.md)将演示如何运行在此库中编码的流式应用程序。