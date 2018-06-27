# 6.3 Kafka Configuration

# 6.3 Kafka 配置

## Important Client Configurations

The most important old Scala producer configurations control

最重要的旧的Scale生产者配置控制

* acks
* compression
* sync vs async production
* batch size (for async producers)

The most important new Java producer configurations control

最重要的新的Java生产者配置控制

* acks
* compression
* batch size

The most important consumer configuration is the fetch size.

最重要的消费者配置是抓取大小。

All configurations are documented in the [configuration](http://kafka.apache.org/documentation/#configuration) section.

所有配置都记录在[配置](http://kafka.apache.org/documentation/#configuration)部分。

## A Production Server Config

Here is an example production server configuration:

以下是生产服务器配置示例：

```bash
# ZooKeeper
zookeeper.connect=[list of ZooKeeper servers]

# Log configuration
num.partitions=8
default.replication.factor=3
log.dir=[List of directories. Kafka should have its own dedicated disk(s) or SSD(s).]

# Other configurations
broker.id=[An integer. Start with 0 and increment by 1 for each new broker.]
listeners=[list of listeners]
auto.create.topics.enable=false
min.insync.replicas=2
queued.max.requests=[number of concurrent requests]
```

Our client configuration varies a fair amount between different use cases.

我们的客户配置在不同用例之间变化很大。