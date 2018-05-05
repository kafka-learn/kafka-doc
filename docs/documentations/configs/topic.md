
Configurations pertinent to topics have both a server default as well an optional per-topic override. If no per-topic configuration is given the server default is used. The override can be set at topic creation time by giving one or more --config options.

与 topic 相关的配置包含服务器默认值，也包含可选的每个topic的覆盖值。如果没有给出每个主题的配置，则服务器默认值将被使用。可以在创建topic的时候通过--config给出一些覆盖值。


下面的例子给出了一个名称为 my-topic 的 topic，并给出了 mx massage size 和 flush rate
```
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic my-topic --partitions 1
    --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
```

创建时的配置也可以在创建之后进行后续的修改,如下
```
bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name my-topic
    --alter --add-config max.message.bytes=128000
```

运行如下的命令去检查是否修改
```
bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name my-topic --describe
```

删除配置
```
bin/kafka-configs.sh --zookeeper localhost:2181  --entity-type topics --entity-name my-topic --alter --delete-config max.message.bytes
```

The following are the topic-level configurations. The server's default configuration for this property is given under the Server Default Property heading. A given server default config value only applies to a topic if it does not have an explicit topic config override.


name | description | type | default | valid values | server default property |importance |
--- | --- | --- | --- | --- | --- | --- 
cleanup.policy | A string that is either "delete" or "compact". This string designates the retention policy to use on old log segments. The default policy ("delete") will discard old segments when their retention time or size limit has been reached. The "compact" setting will enable log compaction on the topic. |	list | delete | [compact, delete] | log.cleanup.policy	| medium
compression.type |	Specify the final compression type for a given topic. This configuration accepts the standard compression codecs ('gzip', 'snappy', lz4). It additionally accepts 'uncompressed' which is equivalent to no compression; and 'producer' which means retain the original compression codec set by the producer. |	string	| producer	| [uncompressed, snappy, lz4, gzip, producer]	| compression.type |	medium
delete.retention.ms | --- | --- | --- | --- | --- | --- 
file.delete.delay.ms | --- | --- | --- | --- | --- | --- 
flush.messages| --- | --- | --- | --- | --- | --- 
flush.ms | --- | --- | --- | --- | --- | --- 
follower.replication.throttled.replicas | --- | --- | --- | --- | --- | --- 
index.interval.bytes | --- | --- | --- | --- | --- | --- 
leader.replication.throttled.replicas | --- | --- | --- | --- | --- | --- 
max.message.bytes | --- | --- | --- | --- | --- | --- 
message.format.version | --- | --- | --- | --- | --- | --- 
message.timestamp.difference.max.ms | --- | --- | --- message.timestamp.type | --- | --- | --- 
min.cleanable.dirty.ratio | --- | --- | --- | --- | --- | --- 
min.compaction.lag.ms | --- | --- | --- | --- | --- | --- 
min.insync.replicas | --- | --- | --- | --- | --- | --- 
preallocate | --- | --- | --- | --- | --- | --- 
retention.bytes | --- | --- | --- | --- | --- | --- 
retention.ms | --- | --- | --- | --- | --- | --- 
segment.bytes | --- | --- | --- | --- | --- | --- 
segment.index.bytes | --- | --- | --- | --- | --- | --- 
segment.jitter.ms | --- | --- | --- | --- | --- | --- 
unclean.leader.election.enable | --- | --- | --- | --- | --- | --- 
| --- | --- | --- | --- | --- | --- 

---
