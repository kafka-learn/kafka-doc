## 3. CONFIGURATION

### 3.1 Broker Conifg 

### 3.1 代理配置

The essential configurations are the following:

基本配置是如下三个:

- `broker.id`


- `log.dirs`


- `zookeeper.connect`

Topic-level configurations and defaults are discussed in more detail [below](http://kafka.apache.org/documentation/#topicconfigs).

主题的配置和默认值将在[后面](./topic.md)进行更详细的讨论。




| Name                                     | Description                              | Type     | Default                                  | VALID VALUES                | Importance | DYNAMIC UPDATE MODE |
| ---------------------------------------- | ---------------------------------------- | -------- | ---------------------------------------- | --------------------------- | ---------- | ------------------- |
| zookeeper.connect                        | Zookeeper host string                    | string   |                                          |                             | high       | read-only           |
| advertised.host.name                     | DEPRECATED: only used when `advertised.listeners` or `listeners` are not set. Use `advertised.listeners` instead. Hostname to publish to ZooKeeper for clients to use. In IaaS environments, this may need to be different from the interface to which the broker binds. If this is not set, it will use the value for `host.name` if configured. Otherwise it will use the value returned from java.net.InetAddress.getCanonicalHostName(). | string   | null                                     |                             | high       | read-only           |
| advertised.listeners                     | Listeners to publish to ZooKeeper for clients to use, if different than the `listeners` config property. In IaaS environments, this may need to be different from the interface to which the broker binds. If this is not set, the value for `listeners` will be used. Unlike `listeners` it is not valid to advertise the 0.0.0.0 meta-address. | string   | null                                     |                             | high       | per-broker          |
| advertised.port                          | DEPRECATED: only used when `advertised.listeners` or `listeners` are not set. Use `advertised.listeners` instead. The port to publish to ZooKeeper for clients to use. In IaaS environments, this may need to be different from the port to which the broker binds. If this is not set, it will publish the same port that the broker binds to. | int      | null                                     |                             | high       | read-only           |
| auto.create.topics.enable                | Enable auto creation of topic on the server | boolean  | true                                     |                             | high       | read-only           |
| auto.leader.rebalance.enable             | Enables auto leader balancing. A background thread checks and triggers leader balance if required at regular intervals | boolean  | true                                     |                             | high       | read-only           |
| background.threads                       | The number of threads to use for various background processing tasks | int      | 10                                       | [1,...]                     | high       | cluster-wide        |
| broker.id                                | The broker id for this server. If unset, a unique broker id will be generated.To avoid conflicts between zookeeper generated broker id's and user configured broker id's, generated broker ids start from reserved.broker.max.id + 1. | int      | -1                                       |                             | high       | read-only           |
| compression.type                         | Specify the final compression type for a given topic. This configuration accepts the standard compression codecs ('gzip', 'snappy', 'lz4'). It additionally accepts 'uncompressed' which is equivalent to no compression; and 'producer' which means retain the original compression codec set by the producer. | string   | producer                                 |                             | high       | cluster-wide        |
| delete.topic.enable                      | Enables delete topic. Delete topic through the admin tool will have no effect if this config is turned off | boolean  | true                                     |                             | high       | read-only           |
| host.name                                | DEPRECATED: only used when `listeners` is not set. Use `listeners` instead. hostname of broker. If this is set, it will only bind to this address. If this is not set, it will bind to all interfaces | string   | ""                                       |                             | high       | read-only           |
| leader.imbalance.check.interval.seconds  | The frequency with which the partition rebalance check is triggered by the controller | long     | 300                                      |                             | high       | read-only           |
| leader.imbalance.per.broker.percentage   | The ratio of leader imbalance allowed per broker. The controller would trigger a leader balance if it goes above this value per broker. The value is specified in percentage. | int      | 10                                       |                             | high       | read-only           |
| listeners                                | Listener List - Comma-separated list of URIs we will listen on and the listener names. If the listener name is not a security protocol, listener.security.protocol.map must also be set.Specify hostname as 0.0.0.0 to bind to all interfaces.Leave hostname empty to bind to default interface.Examples of legal listener lists:PLAINTEXT://myhost:9092,SSL://:9091CLIENT://0.0.0.0:9092,REPLICATION://localhost:9093 | string   | null                                     |                             | high       | per-broker          |
| log.dir                                  | The directory in which the log data is kept (supplemental for log.dirs property) | string   | /tmp/kafka-logs                          |                             | high       | read-only           |
| log.dirs                                 | The directories in which the log data is kept. If not set, the value in log.dir is used | string   | null                                     |                             | high       | read-only           |
| log.flush.interval.messages              | The number of messages accumulated on a log partition before messages are flushed to disk | long     | 9223372036854775807                      | [1,...]                     | high       | cluster-wide        |
| log.flush.interval.ms                    | The maximum time in ms that a message in any topic is kept in memory before flushed to disk. If not set, the value in log.flush.scheduler.interval.ms is used | long     | null                                     |                             | high       | cluster-wide        |
| log.flush.offset.checkpoint.interval.ms  | The frequency with which we update the persistent record of the last flush which acts as the log recovery point | int      | 60000                                    | [0,...]                     | high       | read-only           |
| log.flush.scheduler.interval.ms          | The frequency in ms that the log flusher checks whether any log needs to be flushed to disk | long     | 9223372036854775807                      |                             | high       | read-only           |
| log.flush.start.offset.checkpoint.interval.ms | The frequency with which we update the persistent record of log start offset | int      | 60000                                    | [0,...]                     | high       | read-only           |
| log.retention.bytes                      | The maximum size of the log before deleting it | long     | -1                                       |                             | high       | cluster-wide        |
| log.retention.hours                      | The number of hours to keep a log file before deleting it (in hours), tertiary to log.retention.ms property | int      | 168                                      |                             | high       | read-only           |
| log.retention.minutes                    | The number of minutes to keep a log file before deleting it (in minutes), secondary to log.retention.ms property. If not set, the value in log.retention.hours is used | int      | null                                     |                             | high       | read-only           |
| log.retention.ms                         | The number of milliseconds to keep a log file before deleting it (in milliseconds), If not set, the value in log.retention.minutes is used | long     | null                                     |                             | high       | cluster-wide        |
| log.roll.hours                           | The maximum time before a new log segment is rolled out (in hours), secondary to log.roll.ms property | int      | 168                                      | [1,...]                     | high       | read-only           |
| log.roll.jitter.hours                    | The maximum jitter to subtract from logRollTimeMillis (in hours), secondary to log.roll.jitter.ms property | int      | 0                                        | [0,...]                     | high       | read-only           |
| log.roll.jitter.ms                       | The maximum jitter to subtract from logRollTimeMillis (in milliseconds). If not set, the value in log.roll.jitter.hours is used | long     | null                                     |                             | high       | cluster-wide        |
| log.roll.ms                              | The maximum time before a new log segment is rolled out (in milliseconds). If not set, the value in log.roll.hours is used | long     | null                                     |                             | high       | cluster-wide        |
| log.segment.bytes                        | The maximum size of a single log file    | int      | 1073741824                               | [14,...]                    | high       | cluster-wide        |
| log.segment.delete.delay.ms              | The amount of time to wait before deleting a file from the filesystem | long     | 60000                                    | [0,...]                     | high       | cluster-wide        |
| message.max.bytes                        | The largest record batch size allowed by Kafka. If this is increased and there are consumers older than 0.10.2, the consumers' fetch size must also be increased so that the they can fetch record batches this large. In the latest message format version, records are always grouped into batches for efficiency. In previous message format versions, uncompressed records are not grouped into batches and this limit only applies to a single record in that case. This can be set per topic with the topic level `max.message.bytes` config. | int      | 1000012                                  | [0,...]                     | high       | cluster-wide        |
| min.insync.replicas                      | When a producer sets acks to "all" (or "-1"), min.insync.replicas specifies the minimum number of replicas that must acknowledge a write for the write to be considered successful. If this minimum cannot be met, then the producer will raise an exception (either NotEnoughReplicas or NotEnoughReplicasAfterAppend). When used together, min.insync.replicas and acks allow you to enforce greater durability guarantees. A typical scenario would be to create a topic with a replication factor of 3, set min.insync.replicas to 2, and produce with acks of "all". This will ensure that the producer raises an exception if a majority of replicas do not receive a write. | int      | 1                                        | [1,...]                     | high       | cluster-wide        |
| num.io.threads                           | The number of threads that the server uses for processing requests, which may include disk I/O | int      | 8                                        | [1,...]                     | high       | cluster-wide        |
| num.network.threads                      | The number of threads that the server uses for receiving requests from the network and sending responses to the network | int      | 3                                        | [1,...]                     | high       | cluster-wide        |
| num.recovery.threads.per.data.dir        | The number of threads per data directory to be used for log recovery at startup and flushing at shutdown | int      | 1                                        | [1,...]                     | high       | cluster-wide        |
| num.replica.alter.log.dirs.threads       | The number of threads that can move replicas between log directories, which may include disk I/O | int      | null                                     |                             | high       | read-only           |
| num.replica.fetchers                     | Number of fetcher threads used to replicate messages from a source broker. Increasing this value can increase the degree of I/O parallelism in the follower broker. | int      | 1                                        |                             | high       | cluster-wide        |
| offset.metadata.max.bytes                | The maximum size for a metadata entry associated with an offset commit | int      | 4096                                     |                             | high       | read-only           |
| offsets.commit.required.acks             | The required acks before the commit can be accepted. In general, the default (-1) should not be overridden | short    | -1                                       |                             | high       | read-only           |
| offsets.commit.timeout.ms                | Offset commit will be delayed until all replicas for the offsets topic receive the commit or this timeout is reached. This is similar to the producer request timeout. | int      | 5000                                     | [1,...]                     | high       | read-only           |
| offsets.load.buffer.size                 | Batch size for reading from the offsets segments when loading offsets into the cache. | int      | 5242880                                  | [1,...]                     | high       | read-only           |
| offsets.retention.check.interval.ms      | Frequency at which to check for stale offsets | long     | 600000                                   | [1,...]                     | high       | read-only           |
| offsets.retention.minutes                | Offsets older than this retention period will be discarded | int      | 1440                                     | [1,...]                     | high       | read-only           |
| offsets.topic.compression.codec          | Compression codec for the offsets topic - compression may be used to achieve "atomic" commits | int      | 0                                        |                             | high       | read-only           |
| offsets.topic.num.partitions             | The number of partitions for the offset commit topic (should not change after deployment) | int      | 50                                       | [1,...]                     | high       | read-only           |
| offsets.topic.replication.factor         | The replication factor for the offsets topic (set higher to ensure availability). Internal topic creation will fail until the cluster size meets this replication factor requirement. | short    | 3                                        | [1,...]                     | high       | read-only           |
| offsets.topic.segment.bytes              | The offsets topic segment bytes should be kept relatively small in order to facilitate faster log compaction and cache loads | int      | 104857600                                | [1,...]                     | high       | read-only           |
| port                                     | DEPRECATED: only used when `listeners` is not set. Use `listeners` instead. the port to listen and accept connections on | int      | 9092                                     |                             | high       | read-only           |
| queued.max.requests                      | The number of queued requests allowed before blocking the network threads | int      | 500                                      | [1,...]                     | high       | read-only           |
| quota.consumer.default                   | DEPRECATED: Used only when dynamic default quotas are not configured for or in Zookeeper. Any consumer distinguished by clientId/consumer group will get throttled if it fetches more bytes than this value per-second | long     | 9223372036854775807                      | [1,...]                     | high       | read-only           |
| quota.producer.default                   | DEPRECATED: Used only when dynamic default quotas are not configured for，or  in Zookeeper. Any producer distinguished by clientId will get throttled if it produces more bytes than this value per-second | long     | 9223372036854775807                      | [1,...]                     | high       | read-only           |
| replica.fetch.min.bytes                  | Minimum bytes expected for each fetch response. If not enough bytes, wait up to replicaMaxWaitTimeMs | int      | 1                                        |                             | high       | read-only           |
| replica.fetch.wait.max.ms                | max wait time for each fetcher request issued by follower replicas. This value should always be less than the replica.lag.time.max.ms at all times to prevent frequent shrinking of ISR for low throughput topics | int      | 500                                      |                             | high       | read-only           |
| replica.high.watermark.checkpoint.interval.ms | The frequency with which the high watermark is saved out to disk | long     | 5000                                     |                             | high       | read-only           |
| replica.lag.time.max.ms                  | If a follower hasn't sent any fetch requests or hasn't consumed up to the leaders log end offset for at least this time, the leader will remove the follower from isr | long     | 10000                                    |                             | high       | read-only           |
| replica.socket.receive.buffer.bytes      | The socket receive buffer for network requests | int      | 65536                                    |                             | high       | read-only           |
| replica.socket.timeout.ms                | The socket timeout for network requests. Its value should be at least replica.fetch.wait.max.ms | int      | 30000                                    |                             | high       | read-only           |
| request.timeout.ms                       | The configuration controls the maximum amount of time the client will wait for the response of a request. If the response is not received before the timeout elapses the client will resend the request if necessary or fail the request if retries are exhausted. | int      | 30000                                    |                             | high       | read-only           |
| socket.receive.buffer.bytes              | The SO_RCVBUF buffer of the socket sever sockets. If the value is -1, the OS default will be used. | int      | 102400                                   |                             | high       | read-only           |
| socket.request.max.bytes                 | The maximum number of bytes in a socket request | int      | 104857600                                | [1,...]                     | high       | read-only           |
| socket.send.buffer.bytes                 | The SO_SNDBUF buffer of the socket sever sockets. If the value is -1, the OS default will be used. | int      | 102400                                   |                             | high       | read-only           |
| transaction.max.timeout.ms               | The maximum allowed timeout for transactions. If a client’s requested transaction time exceed this, then the broker will return an error in InitProducerIdRequest. This prevents a client from too large of a timeout, which can stall consumers reading from topics included in the transaction. | int      | 900000                                   | [1,...]                     | high       | read-only           |
| transaction.state.log.load.buffer.size   | Batch size for reading from the transaction log segments when loading producer ids and transactions into the cache. | int      | 5242880                                  | [1,...]                     | high       | read-only           |
| transaction.state.log.min.isr            | Overridden min.insync.replicas config for the transaction topic. | int      | 2                                        | [1,...]                     | high       | read-only           |
| transaction.state.log.num.partitions     | The number of partitions for the transaction topic (should not change after deployment). | int      | 50                                       | [1,...]                     | high       | read-only           |
| transaction.state.log.replication.factor | The replication factor for the transaction topic (set higher to ensure availability). Internal topic creation will fail until the cluster size meets this replication factor requirement. | short    | 3                                        | [1,...]                     | high       | read-only           |
| transaction.state.log.segment.bytes      | The transaction topic segment bytes should be kept relatively small in order to facilitate faster log compaction and cache loads | int      | 104857600                                | [1,...]                     | high       | read-only           |
| transactional.id.expiration.ms           | The maximum amount of time in ms that the transaction coordinator will wait before proactively expire a producer's transactional id without receiving any transaction status updates from it. | int      | 604800000                                | [1,...]                     | high       | read-only           |
| unclean.leader.election.enable           | Indicates whether to enable replicas not in the ISR set to be elected as leader as a last resort, even though doing so may result in data loss | boolean  | false                                    |                             | high       | cluster-wide        |
| zookeeper.connection.timeout.ms          | The max time that the client waits to establish a connection to zookeeper. If not set, the value in zookeeper.session.timeout.ms is used | int      | null                                     |                             | high       | read-only           |
| zookeeper.max.in.flight.requests         | The maximum number of unacknowledged requests the client will send to Zookeeper before blocking. | int      | 10                                       | [1,...]                     | high       | read-only           |
| zookeeper.session.timeout.ms             | Zookeeper session timeout                | int      | 6000                                     |                             | high       | read-only           |
| zookeeper.set.acl                        | Set client to use secure ACLs            | boolean  | false                                    |                             | high       | read-only           |
| broker.id.generation.enable              | Enable automatic broker id generation on the server. When enabled the value configured for reserved.broker.max.id should be reviewed. | boolean  | true                                     |                             | medium     | read-only           |
| broker.rack                              | Rack of the broker. This will be used in rack aware replication assignment for fault tolerance. Examples: `RACK1`, `us-east-1d` | string   | null                                     |                             | medium     | read-only           |
| connections.max.idle.ms                  | Idle connections timeout: the server socket processor threads close the connections that idle more than this | long     | 600000                                   |                             | medium     | read-only           |
| controlled.shutdown.enable               | Enable controlled shutdown of the server | boolean  | true                                     |                             | medium     | read-only           |
| controlled.shutdown.max.retries          | Controlled shutdown can fail for multiple reasons. This determines the number of retries when such failure happens | int      | 3                                        |                             | medium     | read-only           |
| controlled.shutdown.retry.backoff.ms     | Before each retry, the system needs time to recover from the state that caused the previous failure (Controller fail over, replica lag etc). This config determines the amount of time to wait before retrying. | long     | 5000                                     |                             | medium     | read-only           |
| controller.socket.timeout.ms             | The socket timeout for controller-to-broker channels | int      | 30000                                    |                             | medium     | read-only           |
| default.replication.factor               | default replication factors for automatically created topics | int      | 1                                        |                             | medium     | read-only           |
| delegation.token.expiry.time.ms          | The token validity time in seconds before the token needs to be renewed. Default value 1 day. | long     | 86400000                                 | [1,...]                     | medium     | read-only           |
| delegation.token.master.key              | Master/secret key to generate and verify delegation tokens. Same key must be configured across all the brokers.  If the key is not set or set to empty string, brokers will disable the delegation token support. | password | null                                     |                             | medium     | read-only           |
| delegation.token.max.lifetime.ms         | The token has a maximum lifetime beyond which it cannot be renewed anymore. Default value 7 days. | long     | 604800000                                | [1,...]                     | medium     | read-only           |
| delete.records.purgatory.purge.interval.requests | The purge interval (in number of requests) of the delete records request purgatory | int      | 1                                        |                             | medium     | read-only           |
| fetch.purgatory.purge.interval.requests  | The purge interval (in number of requests) of the fetch request purgatory | int      | 1000                                     |                             | medium     | read-only           |
| group.initial.rebalance.delay.ms         | The amount of time the group coordinator will wait for more consumers to join a new group before performing the first rebalance. A longer delay means potentially fewer rebalances, but increases the time until processing begins. | int      | 3000                                     |                             | medium     | read-only           |
| group.max.session.timeout.ms             | The maximum allowed session timeout for registered consumers. Longer timeouts give consumers more time to process messages in between heartbeats at the cost of a longer time to detect failures. | int      | 300000                                   |                             | medium     | read-only           |
| group.min.session.timeout.ms             | The minimum allowed session timeout for registered consumers. Shorter timeouts result in quicker failure detection at the cost of more frequent consumer heartbeating, which can overwhelm broker resources. | int      | 6000                                     |                             | medium     | read-only           |
| inter.broker.listener.name               | Name of listener used for communication between brokers. If this is unset, the listener name is defined by security.inter.broker.protocol. It is an error to set this and security.inter.broker.protocol properties at the same time. | string   | null                                     |                             | medium     | read-only           |
| inter.broker.protocol.version            | Specify which version of the inter-broker protocol will be used.This is typically bumped after all brokers were upgraded to a new version.Example of some valid values are: 0.8.0, 0.8.1, 0.8.1.1, 0.8.2, 0.8.2.0, 0.8.2.1, 0.9.0.0, 0.9.0.1 Check ApiVersion for the full list. | string   | 1.1-IV0                                  |                             | medium     | read-only           |
| log.cleaner.backoff.ms                   | The amount of time to sleep when there are no logs to clean | long     | 15000                                    | [0,...]                     | medium     | cluster-wide        |
| log.cleaner.dedupe.buffer.size           | The total memory used for log deduplication across all cleaner threads | long     | 134217728                                |                             | medium     | cluster-wide        |
| log.cleaner.delete.retention.ms          | How long are delete records retained?    | long     | 86400000                                 |                             | medium     | cluster-wide        |
| log.cleaner.enable                       | Enable the log cleaner process to run on the server. Should be enabled if using any topics with a cleanup.policy=compact including the internal offsets topic. If disabled those topics will not be compacted and continually grow in size. | boolean  | true                                     |                             | medium     | read-only           |
| log.cleaner.io.buffer.load.factor        | Log cleaner dedupe buffer load factor. The percentage full the dedupe buffer can become. A higher value will allow more log to be cleaned at once but will lead to more hash collisions | double   | 0.9                                      |                             | medium     | cluster-wide        |
| log.cleaner.io.buffer.size               | The total memory used for log cleaner I/O buffers across all cleaner threads | int      | 524288                                   | [0,...]                     | medium     | cluster-wide        |
| log.cleaner.io.max.bytes.per.second      | The log cleaner will be throttled so that the sum of its read and write i/o will be less than this value on average | double   | 1.7976931348623157E308                   |                             | medium     | cluster-wide        |
| log.cleaner.min.cleanable.ratio          | The minimum ratio of dirty log to total log for a log to eligible for cleaning | double   | 0.5                                      |                             | medium     | cluster-wide        |
| log.cleaner.min.compaction.lag.ms        | The minimum time a message will remain uncompacted in the log. Only applicable for logs that are being compacted. | long     | 0                                        |                             | medium     | cluster-wide        |
| log.cleaner.threads                      | The number of background threads to use for log cleaning | int      | 1                                        | [0,...]                     | medium     | cluster-wide        |
| log.cleanup.policy                       | The default cleanup policy for segments beyond the retention window. A comma separated list of valid policies. Valid policies are: "delete" and "compact" | list     | delete                                   | [compact, delete]           | medium     | cluster-wide        |
| log.index.interval.bytes                 | The interval with which we add an entry to the offset index | int      | 4096                                     | [0,...]                     | medium     | cluster-wide        |
| log.index.size.max.bytes                 | The maximum size in bytes of the offset index | int      | 10485760                                 | [4,...]                     | medium     | cluster-wide        |
| log.message.format.version               | Specify the message format version the broker will use to append messages to the logs. The value should be a valid ApiVersion. Some examples are: 0.8.2, 0.9.0.0, 0.10.0, check ApiVersion for more details. By setting a particular message format version, the user is certifying that all the existing messages on disk are smaller or equal than the specified version. Setting this value incorrectly will cause consumers with older versions to break as they will receive messages with a format that they don't understand. | string   | 1.1-IV0                                  |                             | medium     | read-only           |
| log.message.timestamp.difference.max.ms  | The maximum difference allowed between the timestamp when a broker receives a message and the timestamp specified in the message. If log.message.timestamp.type=CreateTime, a message will be rejected if the difference in timestamp exceeds this threshold. This configuration is ignored if log.message.timestamp.type=LogAppendTime.The maximum timestamp difference allowed should be no greater than log.retention.ms to avoid unnecessarily frequent log rolling. | long     | 9223372036854775807                      |                             | medium     | cluster-wide        |
| log.message.timestamp.type               | Define whether the timestamp in the message is message create time or log append time. The value should be either `CreateTime` or `LogAppendTime` | string   | CreateTime                               | [CreateTime, LogAppendTime] | medium     | cluster-wide        |
| log.preallocate                          | Should pre allocate file when create new segment? If you are using Kafka on Windows, you probably need to set it to true. | boolean  | false                                    |                             | medium     | cluster-wide        |
| log.retention.check.interval.ms          | The frequency in milliseconds that the log cleaner checks whether any log is eligible for deletion | long     | 300000                                   | [1,...]                     | medium     | read-only           |
| max.connections.per.ip                   | The maximum number of connections we allow from each ip address | int      | 2147483647                               | [1,...]                     | medium     | read-only           |
| max.connections.per.ip.overrides         | Per-ip or hostname overrides to the default maximum number of connections | string   | ""                                       |                             | medium     | read-only           |
| max.incremental.fetch.session.cache.slots | The maximum number of incremental fetch sessions that we will maintain. | int      | 1000                                     | [0,...]                     | medium     | read-only           |
| num.partitions                           | The default number of log partitions per topic | int      | 1                                        | [1,...]                     | medium     | read-only           |
| password.encoder.old.secret              | The old secret that was used for encoding dynamically configured passwords. This is required only when the secret is updated. If specified, all dynamically encoded passwords are decoded using this old secret and re-encoded using password.encoder.secret when broker starts up. | password | null                                     |                             | medium     | read-only           |
| password.encoder.secret                  | The secret used for encoding dynamically configured passwords for this broker. | password | null                                     |                             | medium     | read-only           |
| principal.builder.class                  | The fully qualified name of a class that implements the KafkaPrincipalBuilder interface, which is used to build the KafkaPrincipal object used during authorization. This config also supports the deprecated PrincipalBuilder interface which was previously used for client authentication over SSL. If no principal builder is defined, the default behavior depends on the security protocol in use. For SSL authentication, the principal name will be the distinguished name from the client certificate if one is provided; otherwise, if client authentication is not required, the principal name will be ANONYMOUS. For SASL authentication, the principal will be derived using the rules defined by `sasl.kerberos.principal.to.local.rules` if GSSAPI is in use, and the SASL authentication ID for other mechanisms. For PLAINTEXT, the principal will be ANONYMOUS. | class    | null                                     |                             | medium     | per-broker          |
| producer.purgatory.purge.interval.requests | The purge interval (in number of requests) of the producer request purgatory | int      | 1000                                     |                             | medium     | read-only           |
| queued.max.request.bytes                 | The number of queued bytes allowed before no more requests are read | long     | -1                                       |                             | medium     | read-only           |
| replica.fetch.backoff.ms                 | The amount of time to sleep when fetch partition error occurs. | int      | 1000                                     | [0,...]                     | medium     | read-only           |
| replica.fetch.max.bytes                  | The number of bytes of messages to attempt to fetch for each partition. This is not an absolute maximum, if the first record batch in the first non-empty partition of the fetch is larger than this value, the record batch will still be returned to ensure that progress can be made. The maximum record batch size accepted by the broker is defined viamessage.max.bytes` (broker config) `or `max.message.bytes` (topic config). | int      | 1048576                                  | [0,...]                     | medium     | read-only           |
| replica.fetch.response.max.bytes         | Maximum bytes expected for the entire fetch response. Records are fetched in batches, and if the first record batch in the first non-empty partition of the fetch is larger than this value, the record batch will still be returned to ensure that progress can be made. As such, this is not an absolute maximum. The maximum record batch size accepted by the broker is defined via `message.max.bytes` (broker config) or `max.message.bytes` (topic config). | int      | 10485760                                 | [0,...]                     | medium     | read-only           |
| reserved.broker.max.id                   | Max number that can be used for a broker.id | int      | 1000                                     | [0,...]                     | medium     | read-only           |
| sasl.enabled.mechanisms                  | The list of SASL mechanisms enabled in the Kafka server. The list may contain any mechanism for which a security provider is available. Only GSSAPI is enabled by default. | list     | GSSAPI                                   |                             | medium     | per-broker          |
| sasl.jaas.config                         | JAAS login context parameters for SASL connections in the format used by JAAS configuration files. JAAS configuration file format is described [here](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/LoginConfigFile.html). The format for the value is: ' (=)*;' | password | null                                     |                             | medium     | per-broker          |
| sasl.kerberos.kinit.cmd                  | Kerberos kinit command path.             | string   | /usr/bin/kinit                           |                             | medium     | per-broker          |
| sasl.kerberos.min.time.before.relogin    | Login thread sleep time between refresh attempts. | long     | 60000                                    |                             | medium     | per-broker          |
| sasl.kerberos.principal.to.local.rules   | A list of rules for mapping from principal names to short names (typically operating system usernames). The rules are evaluated in order and the first rule that matches a principal name is used to map it to a short name. Any later rules in the list are ignored. By default, principal names of the form {username}/{hostname}@{REALM} are mapped to {username}. For more details on the format please see [security authorization and acls](http://kafka.apache.org/documentation/#security_authz). Note that this configuration is ignored if an extension of KafkaPrincipalBuilder is provided by the `principal.builder.class `configuration. | list     | DEFAULT                                  |                             | medium     | per-broker          |
| sasl.kerberos.service.name               | The Kerberos principal name that Kafka runs as. This can be defined either in Kafka's JAAS config or in Kafka's config. | string   | null                                     |                             | medium     | per-broker          |
| sasl.kerberos.ticket.renew.jitter        | Percentage of random jitter added to the renewal time. | double   | 0.05                                     |                             | medium     | per-broker          |
| sasl.kerberos.ticket.renew.window.factor | Login thread will sleep until the specified window factor of time from last refresh to ticket's expiry has been reached, at which time it will try to renew the ticket. | double   | 0.8                                      |                             | medium     | per-broker          |
| sasl.mechanism.inter.broker.protocol     | SASL mechanism used for inter-broker communication. Default is GSSAPI. | string   | GSSAPI                                   |                             | medium     | per-broker          |
| security.inter.broker.protocol           | Security protocol used to communicate between brokers. Valid values are: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL. It is an error to set this and inter.broker.listener.name properties at the same time. | string   | PLAINTEXT                                |                             | medium     | read-only           |
| ssl.cipher.suites                        | A list of cipher suites. This is a named combination of authentication, encryption, MAC and key exchange algorithm used to negotiate the security settings for a network connection using TLS or SSL network protocol. By default all the available cipher suites are supported. | list     | ""                                       |                             | medium     | per-broker          |
| ssl.client.auth                          | Configures kafka broker to request client authentication. The following settings are common:  `ssl.client.auth=required` If set to required client authentication is required. `ssl.client.auth=requested` This means client authentication is optional. unlike requested , if this option is set client can choose not to provide authentication information about itself `ssl.client.auth=none` This means client authentication is not needed. | string   | none                                     | [required, requested, none] | medium     | per-broker          |
| ssl.enabled.protocols                    | The list of protocols enabled for SSL connections. | list     | TLSv1.2,TLSv1.1,TLSv1                    |                             | medium     | per-broker          |
| ssl.key.password                         | The password of the private key in the key store file. This is optional for client. | password | null                                     |                             | medium     | per-broker          |
| ssl.keymanager.algorithm                 | The algorithm used by key manager factory for SSL connections. Default value is the key manager factory algorithm configured for the Java Virtual Machine. | string   | SunX509                                  |                             | medium     | per-broker          |
| ssl.keystore.location                    | The location of the key store file. This is optional for client and can be used for two-way authentication for client. | string   | null                                     |                             | medium     | per-broker          |
| ssl.keystore.password                    | The store password for the key store file. This is optional for client and only needed if ssl.keystore.location is configured. | password | null                                     |                             | medium     | per-broker          |
| ssl.keystore.type                        | The file format of the key store file. This is optional for client. | string   | JKS                                      |                             | medium     | per-broker          |
| ssl.protocol                             | The SSL protocol used to generate the SSLContext. Default setting is TLS, which is fine for most cases. Allowed values in recent JVMs are TLS, TLSv1.1 and TLSv1.2. SSL, SSLv2 and SSLv3 may be supported in older JVMs, but their usage is discouraged due to known security vulnerabilities. | string   | TLS                                      |                             | medium     | per-broker          |
| ssl.provider                             | The name of the security provider used for SSL connections. Default value is the default security provider of the JVM. | string   | null                                     |                             | medium     | per-broker          |
| ssl.trustmanager.algorithm               | The algorithm used by trust manager factory for SSL connections. Default value is the trust manager factory algorithm configured for the Java Virtual Machine. | string   | PKIX                                     |                             | medium     | per-broker          |
| ssl.truststore.location                  | The location of the trust store file.    | string   | null                                     |                             | medium     | per-broker          |
| ssl.truststore.password                  | The password for the trust store file. If a password is not set access to the truststore is still available, but integrity checking is disabled. | password | null                                     |                             | medium     | per-broker          |
| ssl.truststore.type                      | The file format of the trust store file. | string   | JKS                                      |                             | medium     | per-broker          |
| alter.config.policy.class.name           | The alter configs policy class that should be used for validation. The class should implement the `org.apache.kafka.server.policy.AlterConfigPolicy` interface. | class    | null                                     |                             | low        | read-only           |
| alter.log.dirs.replication.quota.window.num | The number of samples to retain in memory for alter log dirs replication quotas | int      | 11                                       | [1,...]                     | low        | read-only           |
| alter.log.dirs.replication.quota.window.size.seconds | The time span of each sample for alter log dirs replication quotas | int      | 1                                        | [1,...]                     | low        | read-only           |
| authorizer.class.name                    | The authorizer class that should be used for authorization | string   | ""                                       |                             | low        | read-only           |
| create.topic.policy.class.name           | The create topic policy class that should be used for validation. The class should implement the `org.apache.kafka.server.policy.CreateTopicPolicy` interface. | class    | null                                     |                             | low        | read-only           |
| delegation.token.expiry.check.interval.ms | Scan interval to remove expired delegation tokens. | long     | 3600000                                  | [1,...]                     | low        | read-only           |
| listener.security.protocol.map           | Map between listener names and security protocols. This must be defined for the same security protocol to be usable in more than one port or IP. For example, internal and external traffic can be separated even if SSL is required for both. Concretely, the user could define listeners with names INTERNAL and EXTERNAL and this property as: `INTERNAL:SSL,EXTERNAL:SSL`. As shown, key and value are separated by a colon and map entries are separated by commas. Each listener name should only appear once in the map. Different security (SSL and SASL) settings can be configured for each listener by adding a normalised prefix (the listener name is lowercased) to the config name. For example, to set a different keystore for the INTERNAL listener, a config with name `listener.name.internal.ssl.keystore.location` would be set. If the config for the listener name is not set, the config will fallback to the generic config (i.e. `ssl.keystore.location`). | string   | PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL |                             | low        | per-broker          |
| metric.reporters                         | A list of classes to use as metrics reporters. Implementing the `org.apache.kafka.common.metrics.MetricsReporter `interface allows plugging in classes that will be notified of new metric creation. The JmxReporter is always included to register JMX statistics. | list     | ""                                       |                             | low        | cluster-wide        |
| metrics.num.samples                      | The number of samples maintained to compute metrics. | int      | 2                                        | [1,...]                     | low        | read-only           |
| metrics.recording.level                  | The highest recording level for metrics. | string   | INFO                                     |                             | low        | read-only           |
| metrics.sample.window.ms                 | The window of time a metrics sample is computed over. | long     | 30000                                    | [1,...]                     | low        | read-only           |
| password.encoder.cipher.algorithm        | The Cipher algorithm used for encoding dynamically configured passwords. | string   | AES/CBC/PKCS5Padding                     |                             | low        | read-only           |
| password.encoder.iterations              | The iteration count used for encoding dynamically configured passwords. | int      | 4096                                     | [1024,...]                  | low        | read-only           |
| password.encoder.key.length              | The key length used for encoding dynamically configured passwords. | int      | 128                                      | [8,...]                     | low        | read-only           |
| password.encoder.keyfactory.algorithm    | The SecretKeyFactory algorithm used for encoding dynamically configured passwords. Default is PBKDF2WithHmacSHA512 if available and PBKDF2WithHmacSHA1 otherwise. | string   | null                                     |                             | low        | read-only           |
| quota.window.num                         | The number of samples to retain in memory for client quotas | int      | 11                                       | [1,...]                     | low        | read-only           |
| quota.window.size.seconds                | The time span of each sample for client quotas | int      | 1                                        | [1,...]                     | low        | read-only           |
| replication.quota.window.num             | The number of samples to retain in memory for replication quotas | int      | 11                                       | [1,...]                     | low        | read-only           |
| replication.quota.window.size.seconds    | The time span of each sample for replication quotas | int      | 1                                        | [1,...]                     | low        | read-only           |
| ssl.endpoint.identification.algorithm    | The endpoint identification algorithm to validate server hostname using server certificate. | string   | null                                     |                             | low        | per-broker          |
| ssl.secure.random.implementation         | The SecureRandom PRNG implementation to use for SSL cryptography operations. | string   | null                                     |                             | low        | per-broker          |
| transaction.abort.timed.out.transaction.cleanup.interval.ms | The interval at which to rollback transactions that have timed out | int      | 60000                                    | [1,...]                     | low        | read-only           |
| transaction.remove.expired.transaction.cleanup.interval.ms | The interval at which to remove transactions that have expired due to transactional.id.expiration.ms passing | int      | 3600000                                  | [1,...]                     | low        | read-only           |
| zookeeper.sync.time.ms                   | How far a ZK follower can be behind a ZK leader | int      | 2000                                     |                             | low        | read-only           |




| 名称                                       | 描述                                       | 类型       | 默认值                                      | 有效值                         | 重要性    | 动态更新模式       |
| ---------------------------------------- | ---------------------------------------- | -------- | ---------------------------------------- | --------------------------- | ------ | ------------ |
| zookeeper.connect                        | 要连接的Zookeeper主机列表                        | string   |                                          |                             | high   | read-only    |
| advertised.host.name                     | 已不再使用：只有当没有设置`advertised.listeners` 或`listeners`的时候才使用。改用advertised.listeners。 发布主机名到ZooKeeper以供客户端使用。 在IaaS环境中，这可能需要设置和代理绑定的接口不同。 如果未设置，则将使用host.name的值（如果已配置）。 否则，它将使用从java.net.InetAddress.getCanonicalHostName（）返回的值。 | string   | null                                     |                             | high   | read-only    |
| advertised.listeners                     | 给客户端用的发布至zookeeper的监听，broker会上送此地址到zookeeper，zookeeper会将此地址提供给消费者，消费者根据此地址获取消息。如果和listeners项配置不同则以此为准，在IaaS环境，此配置项可能和 broker绑定的接口主机名称不同，如果此配置项没有配置则以上面的listeners为准。 | string   | null                                     |                             | high   | per-broker   |
| advertised.port                          | 已弃用：仅在advertised.listeners或listeners未设置时才使用。 改用advertised.listeners。 是用于发布到ZooKeeper供客户端使用的端口。 在IaaS环境中，这项配置可能代理绑定的端口不同。 如果未设置，它将和代理绑定到相同端口。 | int      | null                                     |                             | high   | read-only    |
| auto.create.topics.enable                | 是否允许自动创建主题，如果设为true，那么produce，consume或者fetch metadata一个不存在的topic时，就会自动创建一个默认replication factor和partition number的topic。 | boolean  | true                                     |                             | high   | read-only    |
| auto.leader.rebalance.enable             | 是否启动自动leader平衡，当某个broker（是leader地位)崩溃恢复后，是否通过后台线程使其恢复leader地位 | boolean  | true                                     |                             | high   | read-only    |
| background.threads                       | 用于各种后台处理任务的线程数                           | int      | 10                                       | [1,...]                     | high   | cluster-wide |
| broker.id                                | 此服务器的代理ID。 如果未设置，则会生成唯一的ID。为避免zookeeper生成的ID与用户配置的代理ID之间发生冲突，生成的ID将从reserved.broker.max.id + 1开始。 | int      | -1                                       |                             | high   | read-only    |
| compression.type                         | 指定给定主题的最终压缩类型。 该配置接受标准压缩编解码器（'gzip'，'snappy'，'lz4'）。 它另外接受不压缩和使用生产者设置的压缩类型。 | string   | producer                                 |                             | high   | cluster-wide |
| delete.topic.enable                      | 是否启动删除topic。如果设置为false,你在删除topic的时候无法删除，但是会打上一个你将删除该topic的标记，等到你修改这一属性的值为true后重新启动Kafka集群的时候，集群自动将那些标记删除的topic删除掉，对应的log.dirs目录下的topic目录和数据也会被删除。而将这一属性设置为true之后，你就能成功删除你想要删除的topic了 | boolean  | true                                     |                             | high   | read-only    |
| host.name                                | 如果设置了它，会仅绑定这个地址。如果没有设置，则会绑定所有的网络接口，并提交一个给ZK。**不推荐使用 只有当listeners没有设置时才有必要使用** | string   | ""                                       |                             | high   | read-only    |
| leader.imbalance.check.interval.seconds  | 控制器触发分区再平衡检测的频率                          | long     | 300                                      |                             | high   | read-only    |
| leader.imbalance.per.broker.percentage   | 每个broker允许的不平衡leader的比率，不平衡leader的比率大于此值，控制器就会触发leader平衡操作。以百分比形式指定 | int      | 10                                       |                             | high   | read-only    |
| listeners                                | 监听器列表 - 以逗号分隔的URI列表和监听器名称列表。 如果监听器名称没有使用安全协议，则还必须设置listener.security.protocol.map。将主机名称指定为0.0.0.0以绑定到所有接口。将主机名称留空以绑定到默认接口。合法侦听器列表的示例：PLAINTEXT：//为myhost：9092，SSL：//：9091CLIENT：//0.0.0.0：9092，复制：//本地主机：9093 | string   | null                                     |                             | high   | per-broker   |
| log.dir                                  | 保存日志数据的（单个）目录（对log.dirs属性的补充）            | string   | /tmp/kafka-logs                          |                             | high   | read-only    |
| log.dirs                                 | 保存日志数据的目录，可以设置多个日志文件夹，以逗号隔开。 如果未设置，则使用log.dir中的值 | string   | null                                     |                             | high   | read-only    |
| log.flush.interval.messages              | 数据刷新(sync)到硬盘前之前累积的消息条数，因为磁盘IO操作是一个慢操作,但又是一个”数据可靠性”的必要手段,所以此参数的设置,需要在”数据可靠性”与”性能”之间做必要的权衡.如果此值过大,将会导致每次”fsync”的时间较长(IO阻塞),如果此值过小,将会导致”fsync”的次数较多 | long     | 9223372036854775807                      | [1,...]                     | high   | cluster-wide |
| log.flush.interval.ms                    | 当达到下面的时间(ms)时，执行一次强制的flush操作。interval.ms和interval.messages无论哪个达到，都会flush。如果未设置，则使用log.flush.scheduler.interval.ms中的值 | long     | null                                     |                             | high   | cluster-wide |
| log.flush.offset.checkpoint.interval.ms  | 记录上次把log刷到磁盘的时间点的频率，用来日后的recovery。       | int      | 60000                                    | [0,...]                     | high   | read-only    |
| log.flush.scheduler.interval.ms          | 检查是否需要将日志刷新（持久化）到磁盘的频率（以毫秒为单位）           | long     | 9223372036854775807                      |                             | high   | read-only    |
| log.flush.start.offset.checkpoint.interval.ms | 改变起始offset的位置的频率                         | int      | 60000                                    | [0,...]                     | high   | read-only    |
| log.retention.bytes                      | 最多保留的文件字节大小                              | long     | -1                                       |                             | high   | cluster-wide |
| log.retention.hours                      | 删除之前保留日志文件的小时数（以小时为单位），同log.retention.minutes和log.retention.ms相比，处于第三个级别 | int      | 168                                      |                             | high   | read-only    |
| log.retention.minutes                    | 在删除日志文件之前保留日志文件的分钟数（以分钟为单位），级别次于log.retention.ms属性。 如果未设置，则使用log.retention.hours中的值 | int      | null                                     |                             | high   | read-only    |
| log.retention.ms                         | 在删除日志文件之前保留日志文件的毫秒数（以毫秒为单位），如果未设置，则使用log.retention.minutes中的值 | long     | null                                     |                             | high   | cluster-wide |
| log.roll.hours                           | 当达到配置的时间，会强制新建一个segment。这个参数会在日志segment没有达到log.segment.bytes设置的大小，也会强制新建一个segment。级别次于log.roll.ms属性 | int      | 168                                      | [1,...]                     | high   | read-only    |
| log.roll.jitter.hours                    | 指定日志切分段的小时数，避免日志切分时造成惊群，级别次于log.roll.jitter.ms | int      | 0                                        | [0,...]                     | high   | read-only    |
| log.roll.jitter.ms                       | 指定日志切分段的毫秒数，如果不设置，默认使用log.roll.jitter.hours | long     | null                                     |                             | high   | cluster-wide |
| log.roll.ms                              | 文件切分的时间间隔，与大小限制同时起作用的（以毫秒为单位）。 如果未设置，则使用log.roll.hours中的值 | long     | null                                     |                             | high   | cluster-wide |
| log.segment.bytes                        | 单个日志文件的最大大小                              | int      | 1073741824                               | [14,...]                    | high   | cluster-wide |
| log.segment.delete.delay.ms              | 保存日志的时间，到时间就删除                           | long     | 60000                                    | [0,...]                     | high   | cluster-wide |
| message.max.bytes                        | 服务器接受单个消息的最大大小。如果消费者是0.10.2之前的版本，必须要处理消息的大小要大于等于该项配置。在最新的消息格式版本中，记录总是分组为批次以提高效率。 在以前的消息格式版本中，未压缩的记录不会分组到批次中，并且此限制仅适用于该情况下的单个记录。也可以使用主题中`max.message.bytes`配置项目的值。 | int      | 1000012                                  | [0,...]                     | high   | cluster-wide |
| min.insync.replicas                      | 当生产者acks的值设置为all或者-1的时候，此配置项为消息副本的最小写入成功的数量，如果成功数量没有达到此值则生产者就会抛出异常（NotEnoughReplicas或NotEnoughReplicasAfterAppend）。当min.insync.replicas和acks一起使用，允许你执行更大的持久性保证。 一个典型的场景是创建一个复制因子为3的主题，将min.insync.replicas设置为2，生产者将acks设为“全部”。 这将确保生产者在大多数副本没有正常写入时引发异常。 | int      | 1                                        | [1,...]                     | high   | cluster-wide |
| num.io.threads                           | 服务器用来处理请求的I/O线程的数目；这个线程数目至少要等于硬盘的个数。     | int      | 8                                        | [1,...]                     | high   | cluster-wide |
| num.network.threads                      | 服务器用来处理网络请求的网络线程数目；一般你不需要更改这个属性          | int      | 3                                        | [1,...]                     | high   | cluster-wide |
| num.recovery.threads.per.data.dir        | 用于启动时的日志恢复和关闭时的刷新时每个数据目录的线程数             | int      | 1                                        | [1,...]                     | high   | cluster-wide |
| num.replica.alter.log.dirs.threads       | 可以在日志目录之间移动副本的线程数（可能包括磁盘I / O）           | int      | null                                     |                             | high   | read-only    |
| num.replica.fetchers                     | 从leader进行复制消息的线程数，增加此值可以提高follower代理中的I / O并行度。 | int      | 1                                        |                             | high   | cluster-wide |
| offset.metadata.max.bytes                | 允许client(消费者)保存它们元数据(offset)的最大的数据量      | int      | 4096                                     |                             | high   | read-only    |
| offsets.commit.required.acks             | 在offset commit可以接受之前，需要设置确认的数目，一般不需要更改   | short    | -1                                       |                             | high   | read-only    |
| offsets.commit.timeout.ms                | offset commit会延迟直至此超时或所需的副本都收到offset commit，这类似于producer请求的超时 | int      | 5000                                     | [1,...]                     | high   | read-only    |
| offsets.load.buffer.size                 | 此设置对应于offset manager在读取缓存offset segment的批量大小（以字节为单位). | int      | 5242880                                  | [1,...]                     | high   | read-only    |
| offsets.retention.check.interval.ms      | offset管理器检查失效offsets的频率                  | long     | 600000                                   | [1,...]                     | high   | read-only    |
| offsets.retention.minutes                | 话题偏移量日志保留窗口时间，单位分钟                       | int      | 1440                                     | [1,...]                     | high   | read-only    |
| offsets.topic.compression.codec          | 压缩编解码器 - 压缩可用于实现“原子”提交                   | int      | 0                                        |                             | high   | read-only    |
| offsets.topic.num.partitions             | 偏移的提交主题的分区数目。 由于目前不支持部署之后改变，我们建议您使用生产较高的设置 | int      | 50                                       | [1,...]                     | high   | read-only    |
| offsets.topic.replication.factor         | 偏移量主题的复制因子数目（设置得更高以确保可用性）。 如果offset topic创建时，broker比复制因子少，offset topic将以较少的副本创建。 | short    | 3                                        | [1,...]                     | high   | read-only    |
| offsets.topic.segment.bytes              | offset topic的Segment大小。因为它使用压缩的topic，所有Sgment的大小应该保持小一点，以促进更快的日志压实和负载 | int      | 104857600                                | [1,...]                     | high   | read-only    |
| port                                     | 已经弃用：仅在listeners配置项未设置时使用。 改用listeners。BrokerServer接受客户端连接的端口号 | int      | 9092                                     |                             | high   | read-only    |
| queued.max.requests                      | 在网络线程(network threads)停止读取新请求之前，可以排队等待I/O线程处理的最大请求个数。若是等待IO的请求超过这个数值，那么会停止接受外部消息 | int      | 500                                      | [1,...]                     | high   | read-only    |
| quota.consumer.default                   | 已经弃用: 仅在动态默认配额未在Zookeeper中配置或使用时使用。以clientid或consumer group区分的consumer端每秒可以抓取的最大byte | long     | 9223372036854775807                      | [1,...]                     | high   | read-only    |
| quota.producer.default                   | 已经弃用：仅在动态默认配额未配置或在Zookeeper中使用。 producer端每秒可以产生的最大byte，如果大于该值，任何由clientId标识的生产者都将受到限制 | long     | 9223372036854775807                      | [1,...]                     | high   | read-only    |
| replica.fetch.min.bytes                  | fetch的最小数据尺寸,如果leader中尚未同步的数据不足此值,将会阻塞,直到满足条件，或者是等待时间replicaMaxWaitTimeMs | int      | 1                                        |                             | high   | read-only    |
| replica.fetch.wait.max.msreplicas        | follower同leader之间同步数据通信的最大等待时间，失败了会重试。这个值须小于replica.lag.time.max.ms，以防止低吞吐量主题ISR频繁收缩 | int      | 500                                      |                             | high   | read-only    |
| replica.high.watermark.checkpoint.interval.ms | 每一个副本存储自己的high watermark到磁盘的频率，用来日后的recovery | long     | 5000                                     |                             | high   | read-only    |
| replica.lag.time.max.ms                  | 如果follower没有发送任何提取请求，或者没有使leader日志结束偏移量发生改变，则leader将从isr中移除follower。 replicas响应partition leader的最长等待时间，若是超过这个时间，就将replicas列入ISR(in-sync replicas)，并认为它是死的，不会再加入管理中 | long     | 10000                                    |                             | high   | read-only    |
| replica.socket.receive.buffer.bytes      | 复制过程leader接受请求的buffer大小                  | int      | 65536                                    |                             | high   | read-only    |
| replica.socket.timeout.ms                | 复制数据过程中，replica发送给leader的网络请求的socket超时时间,至少等于replica.fetch.wait.max.ms | int      | 30000                                    |                             | high   | read-only    |
| request.timeout.ms                       | 客户端等待响应的最长时间，如果超时将重发几次，最终报错              | int      | 30000                                    |                             | high   | read-only    |
| socket.receive.buffer.bytes              | socket的接受缓冲区，socket的调优参数SO_RCVBUFF，如果该值为-1，则将使用操作系统默认值。 | int      | 102400                                   |                             | high   | read-only    |
| socket.request.max.bytes                 | socket请求的最大数值，防止跑光内存，message.max.bytes必然要小于socket.request.max.bytes，会被topic创建时的指定参数覆盖 | int      | 104857600                                | [1,...]                     | high   | read-only    |
| socket.send.buffer.bytes                 | server端用来处理socket连接的SO_SNDBUFF（发送）缓冲大小。如果该值为-1，则将使用操作系统默认值。 | int      | 102400                                   |                             | high   | read-only    |
| transaction.max.timeout.ms               | 交易的最大允许超时时间。 如果客户请求的交易时间超过这个时间，那么代理将在InitProducerIdRequest中返回一个错误。 这可以防止客户过度超时，可以阻止消费者阅读交易中包含的主题。 | int      | 900000                                   | [1,...]                     | high   | read-only    |
| transaction.state.log.load.buffer.size   | 在把生产者ID和事务加载到缓存中时，从事务日志段中读取的批量大小。        | int      | 5242880                                  | [1,...]                     | high   | read-only    |
| transaction.state.log.min.isr            | 覆盖事务主题的min.insync.replicas配置。            | int      | 2                                        | [1,...]                     | high   | read-only    |
| transaction.state.log.num.partitions     | 事务主题的分区数量（部署后不应更改）。                      | int      | 50                                       | [1,...]                     | high   | read-only    |
| transaction.state.log.replication.factor | 事务主题的副本数（设置得更高以确保可用性）。 只有集群中副本数达到配置值，内部主题才创建成功 | short    | 3                                        | [1,...]                     | high   | read-only    |
| transaction.state.log.segment.bytes      | 事务主题的segment大小，为了促进更快的日志压缩和缓存加载，它应保持相对较小 | int      | 104857600                                | [1,...]                     | high   | read-only    |
| transactional.id.expiration.ms           | 在未收到任何事务状态更新之前，生产者的事务id的存活时间（在过期前的等待时间）（以毫秒为单位）。 | int      | 604800000                                | [1,...]                     | high   | read-only    |
| unclean.leader.election.enable           | 是否启用不在ISR集合中的副本作为最后选择的领导者，使用的风险是会导致数据丢失  | boolean  | false                                    |                             | high   | cluster-wide |
| zookeeper.connection.timeout.ms          | 客户端与zookeeper建立连接等待的最长时间。 如果未设置，则使用zookeeper.session.timeout.ms中的值 | int      | null                                     |                             | high   | read-only    |
| zookeeper.max.in.flight.requests         | 客户端在阻塞之前可以发送给Zookeeper的最大数量的未确认请求。       | int      | 10                                       | [1,...]                     | high   | read-only    |
| zookeeper.session.timeout.ms             | Zookeeper会话超时时间                          | int      | 6000                                     |                             | high   | read-only    |
| zookeeper.set.acl                        | 将客户端设置为使用安全ACL                           | boolean  | false                                    |                             | high   | read-only    |
| broker.id.generation.enable              | 在服务器上使用自动生成代理ID。 启用后，应该检查reserved.broker.max.id配置的值。 | boolean  | true                                     |                             | medium | read-only    |
| broker.rack                              | broker的机架信息，如果指定了机架信息则在副本分配时会尽可能地让分区的副本分不到不同的机架上，提高容错能力。示例：RACK1，us-east-1d | string   | null                                     |                             | medium | read-only    |
| connections.max.idle.ms                  | 空闲连接超时间：超过该时间，关闭服务器套接字处理器线程              | long     | 600000                                   |                             | medium | read-only    |
| controlled.shutdown.enable               | 是否在在关闭broker时主动迁移leader partition        | boolean  | true                                     |                             | medium | read-only    |
| controlled.shutdown.max.retries          | 由于多种原因，关闭broker时候主动迁移partition可能会失败。 这决定了发生此类故障时的重试次数 | int      | 3                                        |                             | medium | read-only    |
| controlled.shutdown.retry.backoff.ms     | 该配置确定重试之前等待的时间。在每次重试之前，系统需要时间从导致先前故障的状态（控制器故障切换，复制滞后等）中恢复。 | long     | 5000                                     |                             | medium | read-only    |
| controller.socket.timeout.ms             | 控制器套接字超时时间                               | int      | 30000                                    |                             | medium | read-only    |
| default.replication.factor               | 自动创建的主题的默认复制因子                           | int      | 1                                        |                             | medium | read-only    |
| delegation.token.expiry.time.ms          | 令牌有效时间（秒）。 默认值1天。                        | long     | 86400000                                 | [1,...]                     | medium | read-only    |
| delegation.token.master.key              | 主/密钥来生成和验证授权令牌。 必须在所有代理中配置相同的密钥。 如果密钥未设置或设置为空字符串，代理将禁用委托令牌。 | password | null                                     |                             | medium | read-only    |
| delegation.token.max.lifetime.ms         | 令牌具有的最大寿命，超过不能再续约。 默认值7天。                | long     | 604800000                                | [1,...]                     | medium | read-only    |
| delete.records.purgatory.purge.interval.requests | 删除记录的请求间隔（请求数）                           | int      | 1                                        |                             | medium | read-only    |
| fetch.purgatory.purge.interval.requests  | 取消请求清除的事件时间间隔（请求数量）                      | int      | 1000                                     |                             | medium | read-only    |
| group.initial.rebalance.delay.ms         | 在执行第一次重新平衡之前等待更多消费者加入的时间。 较长的延迟意味着重新平衡次数可能较少，但会增加处理开始前的时间。 | int      | 3000                                     |                             | medium | read-only    |
| group.max.session.timeout.ms             | 注册用户的被允许的最大会话超时时间。更长的超时时间让消费者有更多的时间来处理心跳之间的消息，但需要更长的时间来检测故障。 | int      | 300000                                   |                             | medium | read-only    |
| group.min.session.timeout.ms             | 注册用户的被允许的最小会话超时时间。 更短的超时时间会导致更快的故障检测，但会以更频繁的消费者心跳为代价，从而消耗更多的代理资源。 | int      | 6000                                     |                             | medium | read-only    |
| inter.broker.listener.name               | 用于代理之间通信的监听器名称。 如果未设置，则监听器名称由security.inter.broker.protocol定义。 同时设置此和security.inter.broker.protocol属性是错误的。 | string   | null                                     |                             | medium | read-only    |
| inter.broker.protocol.version            | 指定将使用哪个版本的代理间协议。在所有代理升级到新版本后，通常会遇到问题。一些有效值的示例为：0.8.0，0.8.1，0.8.1.1，0.8.2， 0.8.2.0，0.8.2.1，0.9.0.0，0.9.0.1查看ApiVersion以获取完整列表。 | string   | 1.1-IV0                                  |                             | medium | read-only    |
| log.cleaner.backoff.ms                   | 在没有日志清理的情况下的睡眠时间                         | long     | 15000                                    | [0,...]                     | medium | cluster-wide |
| log.cleaner.dedupe.buffer.size           | 保留日志重复数据的总内存                             | long     | 134217728                                |                             | medium | cluster-wide |
| log.cleaner.delete.retention.ms          | 要删除的记录能保留多久？                             | long     | 86400000                                 |                             | medium | cluster-wide |
| log.cleaner.enable                       | 在服务器上运行的日志清理器进程。 如果使用的主题（配置cleanup.policy = compact）包含内部偏移量主题的，则应该启用它。 如果被禁用，这些主题将不会被压缩并不再增长。 | boolean  | true                                     |                             | medium | read-only    |
| log.cleaner.io.buffer.load.factor        | 日志清理器重复数据缓存负载因子。 如果设置较大将允许一次清理更多日志，但会导致更多散列冲突 | double   | 0.9                                      |                             | medium | cluster-wide |
| log.cleaner.io.buffer.size               | 日志清理程序的I / O缓冲区的大小                       | int      | 524288                                   | [0,...]                     | medium | cluster-wide |
| log.cleaner.io.max.bytes.per.second      | 限制日志清理器的读取和写入i / o的总和，使其小于该值。            | double   | 1.7976931348623157E308                   |                             | medium | cluster-wide |
| log.cleaner.min.cleanable.ratio          | 日志清理条件：脏日志与总日志的最小比率                      | double   | 0.5                                      |                             | medium | cluster-wide |
| log.cleaner.min.compaction.lag.ms        | 消息在日志中保持未压缩的最短时间。 仅适用于正在压缩的日志。           | long     | 0                                        |                             | medium | cluster-wide |
| log.cleaner.threads                      | 用于日志清理的后台线程数量                            | int      | 1                                        | [0,...]                     | medium | cluster-wide |
| log.cleanup.policy                       | 超出保留窗口时间的segment的默认清理策略。 d多个策略用逗号分隔。 有效的策略是：“删除”和“紧凑” | list     | delete                                   | [compact, delete]           | medium | cluster-wide |
| log.index.interval.bytes                 | 添加一个条目到偏移索引的时间间隔                         | int      | 4096                                     | [0,...]                     | medium | cluster-wide |
| log.index.size.max.bytes                 | offset索引的最大字节数                           | int      | 10485760                                 | [4,...]                     | medium | cluster-wide |
| log.message.format.version               | 指定附加到日志的消息格式版本。 该值应该是有效的ApiVersion。 例如：0.8.2，0.9.0.0，0.10.0，查看ApiVersion获取更多细节。 设置了特定的消息格式版本后，磁盘上的所有现有消息都小于或等于指定的版本。 如果错误地设置此值，则会导致使用旧版本的用户将无法理解接收到消息。 | string   | 1.1-IV0                                  |                             | medium | read-only    |
| log.message.timestamp.difference.max.ms  | 代理收到消息时的时间戳与消息中指定的时间戳之间允许的最大差异。 如果log.message.timestamp.type = CreateTime，若时间戳的差值超过此阈值，则会拒绝该消息。 如果log.message.timestamp.type = LogAppendTime，则忽略此配置。允许的最大时间戳差异不应大于log.retention.ms，以避免不必要地频繁进行日志滚动。 | long     | 9223372036854775807                      |                             | medium | cluster-wide |
| log.message.timestamp.type               | 消息中的时间戳是消息创建时间还是日志追加时间。 该值可以是CreateTime或LogAppendTime | string   | CreateTime                               | [CreateTime, LogAppendTime] | medium | cluster-wide |
| log.preallocate                          | 创建新segment时是否应该预先分配文件。 在Windows上使用Kafka，则可能需要将其设置为true。 | boolean  | false                                    |                             | medium | cluster-wide |
| log.retention.check.interval.ms          | 日志清理程序检查是否有日志符合删除条件的频率（以毫秒为单位）           | long     | 300000                                   | [1,...]                     | medium | read-only    |
| max.connections.per.ip                   | 每个IP地址允许的最大连接数                           | int      | 2147483647                               | [1,...]                     | medium | read-only    |
| max.connections.per.ip.overrides         | 每个IP或主机名将覆盖默认的最大连接数                      | string   | ""                                       |                             | medium | read-only    |
| max.incremental.fetch.session.cache.slots | 维护的增量获取的会话的最大数量。                         | int      | 1000                                     | [0,...]                     | medium | read-only    |
| num.partitions                           | 每个主题的日志分区数量                              | int      | 1                                        | [1,...]                     | medium | read-only    |
| password.encoder.old.secret              | 用于密码加密的旧秘密。 这只有在更新密码时才需要。 如果配置了该项，所有动态编码的密码将使用此旧密码解码，并在代理启动时使用password.encoder.secret进行重新加密。 | password | null                                     |                             | medium | read-only    |
| password.encoder.secret                  | 用于加密此代理的动态配置密码的密码。                       | password | null                                     |                             | medium | read-only    |
| principal.builder.class                  | 实现KafkaPrincipalBuilder接口的类的完全限定名，该接口用于构建授权期间使用的KafkaPrincipal对象。 此配置还支持以前用于通过SSL进行客户端身份验证的弃用的PrincipalBuilder接口。 如果未定义主体构建器，则默认行为取决于所使用的安全协议。 对于SSL身份验证，如果提供了主体名称，则主体名称将作为客户端证书的可分辨名称; 否则，如果不需要客户端身份验证，则主体名称将是匿名。 对于SASL认证，如果GSSAPI可用以及对其他机制而言SASL认证ID也可用，则使用sasl.kerberos.principal.to.local.rules定义的规则导出主体。 对于PLAINTEXT，也是匿名的。 | class    | null                                     |                             | medium | per-broker   |
| producer.purgatory.purge.interval.requests | producer请求清除时间                           | int      | 1000                                     |                             | medium | read-only    |
| queued.max.request.bytes                 | 队列中排队的最大字节数                              | long     | -1                                       |                             | medium | read-only    |
| replica.fetch.backoff.ms                 | 发生分区错误时的睡眠（用于处理的)时间。                     | int      | 1000                                     | [0,...]                     | medium | read-only    |
| replica.fetch.max.bytesThe number of bytes of messages to attem | 尝试为每个分区获取的消息的字节数。 如果抓取的第一个非空分区中的第一个记录批次大于此值，则这不是绝对最大值，但仍会返回记录批次以确保可以取得进度。 代理接受的最大记录批处理大小通过message.max.bytes（broker config）或max.message.bytes（topic config）定义。 | int      | 1048576                                  | [0,...]                     | medium | read-only    |
| replica.fetch.response.max.bytes         | 预期的整个获取响应的最大字节数。 记录是批量获取的，并且如果第一批记录在第一个非空分区的分区中大于此值，则这批记录仍将被返回以确保可以进行处理。 因此，这不是绝对的最大值。 代理接受的最大记录批量大小通过message.max.bytes（broker config）或max.message.bytes（topic config）定义。 | int      | 10485760                                 | [0,...]                     | medium | read-only    |
| reserved.broker.max.id                   | broker.id的最大值                            | int      | 1000                                     | [0,...]                     | medium | read-only    |
| sasl.enabled.mechanisms                  | 服务器中使用的SASL机制列表。 该列表可能包含安全提供程序中可用的任何机制。 只有GSSAPI默认启用。 | list     | GSSAPI                                   |                             | medium | per-broker   |
| sasl.jaas.config                         | 要实现使用JAAS配置文件格式的SASL连接，需要配置JAAS登录上下文参数。 [这里](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/LoginConfigFile.html)描述JAAS配置文件格式。 该值的格式是：'（=）*;' | password | null                                     |                             | medium | per-broker   |
| sasl.kerberos.kinit.cmd                  | Kerberos kinit命令路径。                      | string   | /usr/bin/kinit                           |                             | medium | per-broker   |
| sasl.kerberos.min.time.before.relogin    | 登录线程在两次刷新之间的休眠时间                         | long     | 60000                                    |                             | medium | per-broker   |
| sasl.kerberos.principal.to.local.rules   | 从主体名称到短名称（通常是操作系统用户名）映射的规则列表。 规则按顺序进行评估，并使用与主体名称匹配的第一条规则将其映射到短名称。 列表中后面的任何规则都会被忽略。 默认情况下，{username} / {hostname} @ {REALM}形式的主体名称映射到{username}。 有关格式的更多详细信息，请参阅[security authorization and acls](http://kafka.apache.org/documentation/#security_authz)。 请注意，如果由principal.builder.class配置提供扩展的KafkaPrincipalBuilder，则会忽略此配置。 | list     | DEFAULT                                  |                             | medium | per-broker   |
| sasl.kerberos.service.name               | Kafka运行的Kerberos主体名称。 这可以在Kafka的JAAS配置中或在Kafka的配置中定义。 | string   | null                                     |                             | medium | per-broker   |
| sasl.kerberos.ticket.renew.jitter        | 延长随机jitter的更新时间的百分比。                     | double   | 0.05                                     |                             | medium | per-broker   |
| sasl.kerberos.ticket.renew.window.factor | 上次刷新到票证到期的指定窗口时间因子，在这期间登录线程将一直处于睡眠状态，之后它将尝试续订票证。 | double   | 0.8                                      |                             | medium | per-broker   |
| sasl.mechanism.inter.broker.protocol     | 用于broker之间通信的SASL机制。 默认是GSSAPI。          | string   | GSSAPI                                   |                             | medium | per-broker   |
| security.inter.broker.protocol           | 用于在代理之间进行通信的安全协议。 有效值包括：PLAINTEXT，SSL，SASL_PLAINTEXT，SASL_SSL。 不能同时设置此属性和inter.broker.listener.name属性。 | string   | PLAINTEXT                                |                             | medium | read-only    |
| ssl.cipher.suites                        | 密码套件列表。它是集认证，加密，MAC和密钥交换算法一起的命名组合，使用TLS或SSL网络协议进行网络连接的安全设置。 默认情况下，所有可用的密码套件都受支持。 | list     | ""                                       |                             | medium | per-broker   |
| ssl.client.auth                          | 配置kafka代理以请求客户端认证。 基本设置如下：ssl.client.auth =requited 如果设置为required,表示必要要进行客户端身份验证。 ssl.client.auth = requested 这意味着客户端身份验证是可选的。 与请求不同，如果设置了此选项，则客户端可以选择不提供有关其自身的身份验证信息。ssl.client.auth = none这意味着不需要客户端身份验证。 | string   | none                                     | [required, requested, none] | medium | per-broker   |
| ssl.enabled.protocols                    | 启用SSL连接的协议列表。                            | list     | TLSv1.2,TLSv1.1,TLSv1                    |                             | medium | per-broker   |
| ssl.key.password                         | 密钥存储文件中的私钥密码。 这对客户端是可选的。                 | password | null                                     |                             | medium | per-broker   |
| ssl.keymanager.algorithm                 | 密钥管理器工厂使用的用于SSL连接的算法。 默认值是为Java虚拟机配置的密钥管理器工厂算法。 | string   | SunX509                                  |                             | medium | per-broker   |
| ssl.keystore.location                    | 密钥存储文件的位置。 这对于客户端是可选的，并且可以用于客户端的双向认证。    | string   | null                                     |                             | medium | per-broker   |
| ssl.keystore.password                    | 密钥存储文件的存储密码。 这对客户端是可选的，只有在配置了ssl.keystore.location时才需要。 | password | null                                     |                             | medium | per-broker   |
| ssl.keystore.type                        | 密钥存储文件的文件格式。 这对客户端是可选的。                  | string   | JKS                                      |                             | medium | per-broker   |
| ssl.protocol                             | 用于生成SSL Context的SSL协议。 默认设置是TLS。 最近的JVM中允许的值是TLS，TLSv1.1和TLSv1.2。 较早的JVM可能支持SSL，SSLv2和SSLv3，但由于已知的安全漏洞，不鼓励使用。 | string   | TLS                                      |                             | medium | per-broker   |
| ssl.provider                             | 用于SSL连接的安全提供程序的名称。 默认值是JVM的默认安全提供程序。     | string   | null                                     |                             | medium | per-broker   |
| ssl.trustmanager.algorithm               | 信任管理器工厂使用的用于SSL连接的算法。 缺省值是为Java虚拟机配置的信任管理器工厂算法。 | string   | PKIX                                     |                             | medium | per-broker   |
| ssl.truststore.location                  | 信任存储文件的位置                                | string   | null                                     |                             | medium | per-broker   |
| ssl.truststore.password                  | 信任存储文件的密码。 如果未设置密码，对信任库的访问仍然可用，但完整性检查被禁用。 | password | null                                     |                             | medium | per-broker   |
| ssl.truststore.type                      | 信任存储文件的文件格式。                             | string   | JKS                                      |                             | medium | per-broker   |
| alter.config.policy.class.name           | 应该用于验证的修改配置策略类。 该类应该实现org.apache.kafka.server.policy.AlterConfigPolicy接口。 | class    | null                                     |                             | low    | read-only    |
| alter.log.dirs.replication.quota.window.num | 在内存中保留的用于修改日志dirs复制配额的样本数量               | int      | 11                                       | [1,...]                     | low    | read-only    |
| alter.log.dirs.replication.quota.window.size.seconds | 用于更改日志文件目录的复制配额的每个样本的时间跨度                | int      | 1                                        | [1,...]                     | low    | read-only    |
| authorizer.class.name                    | 应该用于授权的授权者类                              | string   | ""                                       |                             | low    | read-only    |
| create.topic.policy.class.name           | 应该用于验证的创建主题策略类。 该类应该实现org.apache.kafka.server.policy.CreateTopicPolicy接口。 | class    | null                                     |                             | low    | read-only    |
| delegation.token.expiry.check.interval.ms | 扫描间隔以删除过期的代理令牌。                          | long     | 3600000                                  | [1,...]                     | low    | read-only    |
| listener.security.protocol.map           | 监听器名称和安全协议之间的映射。 多个端口或IP使用相同的安全协议。例如，即使两个都需要SSL，内部和外部流量也可以分开定义。 具体来说，用户可以定义名称为INTERNAL和EXTERNAL的监听器，并且该属性为：`INTERNAL：SSL，EXTERNAL：SSL`  键和值由冒号分隔，映射条目以逗号分隔。 每个监听器名称只会在Map中出现一次。可以为每个监听器配置不同的安全性（SSL和SASL）设置，方法是将标准化前缀（侦听器名称小写）添加到配置名称。 例如，要为INTERNAL监听器设置不同的密钥库，将会设置名称为“listener.name.internal.ssl.keystore.location”的配置。 如果未设置监听器名称的配置，则配置将回退到通用配置（即`ssl.keystore.location`）。 | string   | PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL |                             | low    | per-broker   |
| metric.reporters                         | 类的列表，用于衡量指标。实现MetricReporter接口，将允许增加一些类，这些类在新的衡量指标产生时就会改变。JmxReporter总会包含用于注册JMX统计 | list     | ""                                       |                             | low    | cluster-wide |
| metrics.num.samples                      | 用于维护metrics的样本数                          | int      | 2                                        | [1,...]                     | low    | read-only    |
| metrics.recording.level                  | 度量标准的最高记录级别。                             | string   | INFO                                     |                             | low    | read-only    |
| metrics.sample.window.ms                 | metrics系统维护可配置的样本数量，在一个可修正的window  size。这项配置配置了窗口大小，例如。我们可能在30s的期间维护两个样本。当一个窗口推出后，我们会擦除并重写最老的窗口 | long     | 30000                                    | [1,...]                     | low    | read-only    |
| password.encoder.cipher.algorithm        | 用于加密密码的密码算法。                             | string   | AES/CBC/PKCS5Padding                     |                             | low    | read-only    |
| password.encoder.iterations              | 加密密码的迭代次数                                | int      | 4096                                     | [1024,...]                  | low    | read-only    |
| password.encoder.key.length              | 加密密码的密钥长度。                               | int      | 128                                      | [8,...]                     | low    | read-only    |
| password.encoder.keyfactory.algorithm    | SecretKeyFactory算法用于动态加密密码。 如果配置了该项，默认为PBKDF2WithHmacSHA512，否则为PBKDF2WithHmacSHA1。 | string   | null                                     |                             | low    | read-only    |
| quota.window.num                         | 保留在内存中的客户端配额的样本数                         | int      | 11                                       | [1,...]                     | low    | read-only    |
| quota.window.size.seconds                | 客户端配额的每个样本的时间跨度                          | int      | 1                                        | [1,...]                     | low    | read-only    |
| replication.quota.window.num             | 要保留在内存中用于复制配额的样本数                        | int      | 11                                       | [1,...]                     | low    | read-only    |
| replication.quota.window.size.seconds    | 每个样本复制配额的时间跨度                            | int      | 1                                        | [1,...]                     | low    | read-only    |
| ssl.endpoint.identification.algorithm    | 使用服务器证书验证服务器主机名的端点识别算法。                  | string   | null                                     |                             | low    | per-broker   |
| ssl.secure.random.implementation         | 用SSL加密操作实现SecureRandom PRNG              | string   | null                                     |                             | low    | per-broker   |
| transaction.abort.timed.out.transaction.cleanup.interval.ms | 回滚已超时的事务的时间间隔                            | int      | 60000                                    | [1,...]                     | low    | read-only    |
| transaction.remove.expired.transaction.cleanup.interval.ms | 删除过期事务的时间间隔                              | int      | 3600000                                  | [1,...]                     | low    | read-only    |
| zookeeper.sync.time.ms                   | ZK follower可以落后ZK leader的最大时间，实际同步时间     | int      | 2000                                     |                             | low    | read-only    |

More details about broker configuration can be found in the scala class `kafka.server.KafkaConfig`.

关于代理配置的更多细节可以去查阅scala类的kafka.server.KafkaConfig的配置文件。

#### 3.1.1 Updating Broker Configs

#### 3.1.1 更新代理配置信息

From Kafka version 1.1 onwards, some of the broker configs can be updated without restarting the broker. See the Dynamic Update Mode column in Broker Configs for the update mode of each broker config.

从Kafka 1.1版本开始，一些代理配置可以在不重启代理的情况下进行更新。您可以去看上面的代理配置表中动态更新模式那一栏（最后一栏），以获得每种代理配置的更新模式。

- read-only: Requires a broker restart for update
- per-broker: May be updated dynamically for each broker
- cluster-wide: May be updated dynamically as a cluster-wide default. May also be updated as a per-broker value for testing.



- `read-only`: 需要代理重新启动才能进行更新
- `per-broker`: 每个代理都可以动态更新
- `cluster-wide`: 可以作为集群默认值进行动态更新。也可以作为每个代理的值进行更新，以供测试使用。

To alter the current broker configs for broker id 0 (for example, the number of log cleaner threads):

修改id为0的broker的配置文件（例如，修改日志清理器的线程数量）:

```
> bin /kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --alter --add-config log.cleaner.threads=2
```

To describe the current dynamic broker configs for broker id 0:

 查看id为0的代理的当前动态代理配置信息:

```
> bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --describe
```

To delete a config override and revert to the statically configured or default value for broker id 0 (for example, the number of log cleaner threads):

删除配置覆盖，并恢复到id为0的代理的静态配置或默认值（例如，恢复日志清理器的线程数量配置）:

```
> bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --alter --delete-config log.cleaner.threads
```

Some configs may be configured as a cluster-wide default to maintain consistent values across the whole cluster. All brokers in the cluster will process the cluster default update. For example, to update log cleaner threads on all brokers:

某些配置可能被配置为集群默认值，以在整个集群中保持一致的值。集群中所有代理都可以进行集群默认更新。 例如，要更新所有代理上的日志清理器线程数量这个配置：

```
> bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-default --alter --add-config log.cleaner.threads=2
```

To describe the currently configured dynamic cluster-wide default configs:

查看当前集群的cluster-wide级别的默认配置：

```
> bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-default --describe
```

All configs that are configurable at cluster level may also be configured at per-broker level (e.g. for testing). If a config value is defined at different levels, the following order of precedence is used:

在集群级别上配置的所有配置都可以被配置为per-broker级别（方便测试）。如果一个配置被配置了不同的级别，则使用以下优先顺序

- Dynamic per-broker config stored in ZooKeeper
- Dynamic cluster-wide default config stored in ZooKeeper
- Static broker config from server.properties
- Kafka default, see [broker configs](http://kafka.apache.org/documentation/#brokerconfigs)


- 存储在Zookeeper中的动态per-broker配置

- 存储在Zookeeper中的动态cluster-wide级别的默认配置

- 在server.properities中的静态代理配置

- Kafka的默认配置，见[代理配置](#31-broker-conifg-)

  ​

**Updating Log Cleaner Configs**

**更新日志清理器配置**

Log cleaner configs may be updated dynamically at cluster-default level used by all brokers. The changes take effect on the next iteration of log cleaning. One or more of these configs may be updated:

日志清理器这个配置可以在所有代理使用的集群默认级别上进行动态更新。这些改变将会在下一次日志清理中生效。可以被更新的配置项如下：

- `log.cleaner.threads`

- `log.cleaner.io.max.bytes.per.second`

- `log.cleaner.dedupe.buffer.size`

- `log.cleaner.io.buffer.size`

- `log.cleaner.io.buffer.load.factor`

- `log.cleaner.backoff.ms`

  ​

**Updating Password Configs Dynamically**

**动态更新密码配置**

Password config values that are dynamically updated are encrypted before storing in ZooKeeper. The broker config `password.encoder.secret` must be configured in `server.properties` to enable dynamic update of password configs. The secret may be different on different brokers.

可以动态更新的密码配置值在存进Zookeeper之前会加密。需要在 `server.properties` 文件中配置 `password.encoder.secret` 才能启用密码配置的动态更新。不同代理的密钥可能不同。

The secret used for password encoding may be rotated with a rolling restart of brokers. The old secret used for encoding passwords currently in ZooKeeper must be provided in the static broker config `password.encoder.old.secret` and the new secret must be provided in `password.encoder.secret`. All dynamic password configs stored in ZooKeeper will be re-encoded with the new secret when the broker starts up.

用于密码加密的密钥会随着代理的重启顺序不同而轮换。当前在Zookeeper中存储的用于加密的旧密钥必须要配置`password.encoder.old.secret`，新密钥就必须要配置`password.encoder.secret`.存储在Zookeeper中的所有动态密码配置会在代理启动时候用新密钥进行重新加密。

In Kafka 1.1.x, all dynamically updated password configs must be provided in every alter request when updating configs using `kafka-configs.sh` even if the password config is not being altered. This constraint will be removed in a future release.

在Kfaka1.1版本之后，即使没有修改密码配置，当运行`kafka-configs.sh`进行更新配置的时候，必须要在每个更改请求中提供所有动态更新的密码配置。未来版本中将删除此限制。

##### **Updating SSL Keystore of an Existing Listener**

**更新现有监听器的SSL密钥库**

Brokers may be configured with SSL keystores with short validity periods to reduce the risk of compromised certificates. Keystores may be updated dynamically without restarting the broker. The config name must be prefixed with the listener prefix `listener.name.{listenerName}.` so that only the keystore config of a specific listener is updated. The following configs may be updated in a single alter request at per-broker level:

代理配置了有效期较短的SSL密钥库，以降低证书受损的风险。密钥库可以在不用重启代理的情况下动态更新。配置名必须以监听器前缀 `listener.name.{listenerName}`作为前缀。以便于只更新特定监听器的密钥库配置。一下配置可以在每个per-broker级别的单个更改请求中更新。

- `ssl.keystore.type`
- `ssl.keystore.location`
- `ssl.keystore.password`
- `ssl.key.password`

If the listener is the inter-broker listener, the update is allowed only if the new keystore is trusted by the truststore configured for that listener. For other listeners, no trust validation is performed on the keystore by the broker. Certificates must be signed by the same certificate authority that signed the old certificate to avoid any client authentication failures.

如果监听器是代理之间的监听器，那么只有为该监听器配置的信任库信任新密钥库时才允许更新。对于其他监听器，代理不会在密钥库上执行信任验证。证书必须由签署旧证书的相同证书颁发机构签署，以避免任何客户端身份验证失败。

**Updating Default Topic Configuration**

**更新默认主题配置**

Default topic configuration options used by brokers may be updated without broker restart. The configs are applied to topics without a topic config override for the equivalent per-topic config. One or more of these configs may be overridden at cluster-default level used by all brokers.

代理使用的默认主题配置选项可以在不重启的情况下更新。为了等效每个主题配置，这种配置被应用于没有主题配置覆盖的主题,。一个或多个这种配置可能会覆盖所有代理使用的cluster-default级别。在默认的集群级别下的代理的这些配置中的部分会被覆盖。

- `log.segment.bytes`
- `log.roll.ms`
- `log.roll.hours`
- `log.roll.jitter.ms`
- `log.roll.jitter.hours`
- `log.index.size.max.bytes`
- `log.flush.interval.messages`
- `log.flush.interval.ms`
- `log.retention.bytes`
- `log.retention.ms`
- `log.retention.minutes`
- `log.retention.hours`
- `log.index.interval.bytes`
- `log.cleaner.delete.retention.ms`
- `log.cleaner.min.compaction.lag.ms`
- `log.cleaner.min.cleanable.ratio`
- `log.cleanup.policy`
- `log.segment.delete.delay.ms`
- `unclean.leader.election.enable`
- `min.insync.replicas`
- `max.message.bytes`
- `compression.type`
- `log.preallocate`
- `log.message.timestamp.type`
- `log.message.timestamp.difference.max.ms`

In Kafka version 1.1.x, changes to `unclean.leader.election.enable`  take effect only when a new controller is elected. Controller re-election may be forced by running:

在Kafka1.1版本以后，只有当重新选择新控制器的视乎，对 `unclean.leader.election.enable` 的修改才会生效。控制器的重选可以被强制执行。

```
> bin/zookeeper-shell.sh localhost
rmr /controller
```

**Updating Thread Configs**

**更新线程配置**

The size of various thread pools used by the broker may be updated dynamically at cluster-default level used by all brokers. Updates are restricted to the range `currentSize / 2`to `currentSize * 2` to ensure that config updates are handled gracefully.

代理使用的各种线程池的大小可以在由所有代理使用的cluster-default级别上进行动态更新。线程池大小的更新被限制在`currentSize / 2` to `currentSize * 2` 这个范围内，以确保配置更新能够被安全处理。

- `num.network.threads`
- `num.io.threads`
- `num.replica.fetchers`
- `num.recovery.threads.per.data.dir`
- `log.cleaner.threads`
- `background.threads`

**Adding and Removing Listeners**

**添加和删除监听器**

Listeners may be added or removed dynamically. When a new listener is added, security configs of the listener must be provided as listener configs with the listener prefix `listener.name.{listenerName}.`. If the new listener uses SASL, the JAAS configuration of the listener must be provided using the JAAS configuration property `sasl.jaas.config` with the listener and mechanism prefix. See [JAAS configuration for Kafka brokers](http://kafka.apache.org/documentation/#security_jaas_broker) for details.

监听器可以被动态添加和删除。当添加一个新的监听器的时候，监听器的安全配置必须要作为监听器配置一部分提供给监听器前缀`listener.name.{listenerName}.`. 如果新侦听器使用SASL，则必须使用带侦听器和机制前缀的JAAS配置属性`sasl.jaas.config`来提供侦听器的JAAS配置。有关详细信息，请参阅Kafka代理的[JAAS配置](../security/authentication_sasl.md)。

In Kafka version 1.1.x, the listener used by the inter-broker listener may not be updated dynamically. To update the inter-broker listener to a new listener, the new listener may be added on all brokers without restarting the broker. A rolling restart is then required to update `inter.broker.listener.name`.

在Kafka的1.1及其后面的版本中，在代理之间使用的监听器不可以动态更新。要想将代理之间使用的监听器更新为一个新的监听器，可以在所有代理上添加新的监听器而无需重启代理。然后需要滚动重启来更新`inter.broker.listener.name`.

In addition to all the security configs of new listeners, the following configs may be updated dynamically at per-broker level:

处理新的监听器的所有安全配置之外，一下配置也可以在每个per-broker级别上动态更新:

- `listeners`
- `advertised.listeners`
- `listener.security.protocol.map`

Inter-broker listener must be configured using the static broker configuration `inter.broker.listener.name` or  `inter.broker.security.protocol`.

代理之间的监听器必须使用静态代理配置 `inter.broker.listener.name`或者是 `inter.broker.security.protocol`进行配置.


