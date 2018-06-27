# 6.2 Datacenters

# 6.2 数据中心

Some deployments will need to manage a data pipeline that spans multiple datacenters. Our recommended approach to this is to deploy a local Kafka cluster in each datacenter with application instances in each datacenter interacting only with their local cluster and mirroring between clusters (see the documentation on the [mirror maker tool](http://kafka.apache.org/documentation/#basic_ops_mirror_maker) for how to do this).

某些部署需要管理跨越多个数据中心的数据管道。 我们推荐的方法是在每个数据中心内的每个数据中心部署一个本地Kafka集群，并在每个数据中心内与应用程序实例进行交互，只与本地集群交互并在集群之间进行镜像（请参阅[镜像制作工具](http://kafka.apache.org/documentation/#basic_ops_mirror_maker)上的文档是如何做到这一点的）。

This deployment pattern allows datacenters to act as independent entities and allows us to manage and tune inter-datacenter replication centrally. This allows each facility to stand alone and operate even if the inter-datacenter links are unavailable: when this occurs the mirroring falls behind until the link is restored at which time it catches up.

这种部署模式允许数据中心充当独立实体，并允许我们集中管理和调整数据中心之间的复制。 这样，即使数据中心间链路不可用，每个设施也可以独立运行并运行：当发生这种情况时，镜像会落后，直到链路恢复正常时为止。

For applications that need a global view of all data you can use mirroring to provide clusters which have aggregate data mirrored from the local clusters in all datacenters. These aggregate clusters are used for reads by applications that require the full data set.

对于需要所有数据的全局视图的应用程序，可以使用镜像来提供从所有数据中心的本地群集镜像聚合数据的群集。 这些聚合群集用于需要完整数据集的应用程序的读取。

This is not the only possible deployment pattern. It is possible to read from or write to a remote Kafka cluster over the WAN, though obviously this will add whatever latency is required to get the cluster.

这不是唯一可能的部署模式。 通过广域网读取或写入远程Kafka集群是可能的，但显然这将增加获取集群所需的任何延迟。

Kafka naturally batches data in both the producer and consumer so it can achieve high-throughput even over a high-latency connection. To allow this though it may be necessary to increase the TCP socket buffer sizes for the producer, consumer, and broker using the ```socket.send.buffer.bytes``` and ```socket.receive.buffer.bytes``` configurations. The appropriate way to set this is documented [here](https://en.wikipedia.org/wiki/Bandwidth-delay_product).

Kafka自然地在生产者和消费者中批量处理数据，因此即使通过高延迟连接也可以实现高吞吐量。 虽然可能需要使用```socket.send.buffer.bytes```和```socket.receive.buffer.bytes```来增加生产者，消费者和代理的TCP套接字缓冲区大小配置。 设置此的适当方式记录在[此处](https://en.wikipedia.org/wiki/Bandwidth-delay_product)。

It is generally not advisable to run a single Kafka cluster that spans multiple datacenters over a high-latency link. This will incur very high replication latency both for Kafka writes and ZooKeeper writes, and neither Kafka nor ZooKeeper will remain available in all locations if the network between locations is unavailable.

通常不建议通过高延迟链接运行跨越多个数据中心的单个Kafka集群。 这将对Kafka写入和ZooKeeper写入产生非常高的复制延迟，并且如果位置之间的网络不可用，Kafka和ZooKeeper都不会在所有位置都可用。