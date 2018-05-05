
# broker Configs

配置最重要的三个配置如下

1. broker.id
2. log.dirs
3. zookeeper.connect


name | description | type | default | valid values | importance | dynamic update mode
--- | --- | --- | --- | --- | --- | --- 
zookeeper.connect | Zookeeper host string | string | --- | --- | high | read-only 
advertised.host.name | DEPRECATED: only used when `advertised.listeners` or `listeners` are not set. Use `advertised.listeners` instead. Hostname to publish to ZooKeeper for clients to use. In IaaS environments, this may need to be different from the interface to which the broker binds. If this is not set, it will use the value for `host.name` if configured. Otherwise it will use the value returned from java.net.InetAddress.getCanonicalHostName().| string | null | --- | high | read only 
advertised.port| --- | --- | --- | --- | --- | --- 
auto.create.topics.enable| --- | --- | --- | --- | --- | --- 
auto.leader.rebalance.enable| --- | --- | --- | --- | --- | --- 
background.threads| --- | --- | --- | --- | --- | --- 
broker.id| The broker id for this server. If unset, a unique broker id will be generated.To avoid conflicts between zookeeper generated broker id's and user configured broker id's, generated broker ids start from reserved.broker.max.id + 1. | int | -1 | --- | high | read-only 
compression.type | --- | --- | --- | --- | --- | --- 
delete.topic.enable | --- | --- | --- | --- | --- | --- 
host.name | --- | --- | --- | --- | --- | --- 
leader.imbalance.check.interval.seconds | --- | --- | --- | --- | --- | --- 
leader.imbalance.per.broker.percentage| --- | --- | --- | --- | --- | --- 
listeners | --- | --- | --- | --- | --- | --- 
log.dir	| The directory in which the log data is kept (supplemental for log.dirs property) | string | /tmp/kafka-logs | --- | high | read only 
log.dirs| The directories in which the log data is kept. If not set, the value in log.dir is used | string | null | --- | high| read only 
log.flush.interval.messages | --- | --- | --- | --- | --- | --- 
log.flush.interval.ms | --- | --- | --- | --- | --- | --- 
log.flush.offset.checkpoint.interval.ms | --- | --- | --- | --- | --- | --- 
log.flush.scheduler.interval.ms | --- | --- | --- | --- | --- | --- 
log.flush.start.offset.checkpoint.interval.ms | --- | --- | --- | --- | --- | --- 
log.retention.bytes | --- | --- | --- | --- | --- | ---
log.retention.hours | --- | --- | --- | --- | --- | ---
log.retention.minutes | --- | --- | --- | --- | --- | ---
log.retention.ms | --- | --- | --- | --- | --- | ---
log.roll.hours | --- | --- | --- | --- | --- | ---
log.roll.jitter.hours | --- | --- | --- | --- | --- | ---
log.roll.jitter.ms | --- | --- | --- | --- | --- | ---
log.roll.ms | --- | --- | --- | --- | --- | ---
log.segment.bytes | --- | --- | --- | --- | --- | ---
log.segment.delete.delay.ms | --- | --- | --- | --- | --- | ---
message.max.bytes | --- | --- | --- | --- | --- | ---
min.insync.replicas | --- | --- | --- | --- | --- | ---
num.io.threads | --- | --- | --- | --- | --- | ---
num.network.threads | --- | --- | --- | --- | --- | ---
num.recovery.threads.per.data.dir | --- | --- | --- | --- | --- | ---
num.replica.alter.log.dirs.threads | --- | --- | --- | --- | --- | ---
num.replica.fetchers | --- | --- | --- | --- | --- | ---
offset.metadata.max.bytes | --- | --- | --- | --- | --- | ---
offsets.commit.required.acks | --- | --- | --- | --- | --- | ---
offsets.commit.timeout.ms | --- | --- | --- | --- | --- | ---
offsets.load.buffer.size | --- | --- | --- | --- | --- | ---
offsets.retention.check.interval.ms | --- | --- | --- | --- | --- | ---
offsets.retention.minutes | --- | --- | --- | --- | --- | ---
offsets.topic.compression.codec | --- | --- | --- | --- | --- | ---
offsets.topic.num.partitions | --- | --- | --- | --- | --- | ---
offsets.topic.replication.factor | --- | --- | --- | --- | --- | ---
offsets.topic.segment.bytes | --- | --- | --- | --- | --- | ---
port | --- | --- | --- | --- | --- | ---
queued.max.requests | --- | --- | --- | --- | --- | ---
quota.consumer.default | --- | --- | --- | --- | --- | ---
quota.producer.default | --- | --- | --- | --- | --- | ---
replica.fetch.min.bytes | --- | --- | --- | --- | --- | ---
replica.fetch.wait.max.ms | --- | --- | --- | --- | --- | ---
replica.high.watermark.checkpoint.interval.ms | --- | --- | --- | --- | --- | ---
replica.lag.time.max.ms | --- | --- | --- | --- | --- | ---
replica.socket.receive.buffer.bytes | --- | --- | --- | --- | --- | ---
replica.socket.timeout.ms | --- | --- | --- | --- | --- | ---
request.timeout.ms | --- | --- | --- | --- | --- | ---
socket.receive.buffer.bytes | --- | --- | --- | --- | --- | ---
socket.request.max.bytes | --- | --- | --- | --- | --- | ---
socket.send.buffer.bytes | --- | --- | --- | --- | --- | ---
transaction.max.timeout.ms | --- | --- | --- | --- | --- | ---
transaction.state.log.load.buffer.size | --- | --- | --- | --- | --- | ---
transaction.state.log.min.isr | --- | --- | --- | --- | --- | ---
transaction.state.log.num.partitions | --- | --- | --- | --- | --- | ---
transaction.state.log.replication.factor | --- | --- | --- | --- | --- | ---
transaction.state.log.segment.bytes | --- | --- | --- | --- | --- | ---
transactional.id.expiration.ms | --- | --- | --- | --- | --- | ---
unclean.leader.election.enable | --- | --- | --- | --- | --- | ---
zookeeper.connection.timeout.ms | --- | --- | --- | --- | --- | ---
zookeeper.max.in.flight.requests | --- | --- | --- | --- | --- | ---
zookeeper.session.timeout.ms | --- | --- | --- | --- | --- | ---
zookeeper.set.acl | --- | --- | --- | --- | --- | ---
broker.id.generation.enable | --- | --- | --- | --- | --- | ---
broker.rack | --- | --- | --- | --- | --- | ---
connections.max.idle.ms | --- | --- | --- | --- | --- | ---
controlled.shutdown.enable | --- | --- | --- | --- | --- | ---
controlled.shutdown.max.retries | --- | --- | --- | --- | --- | ---
controlled.shutdown.retry.backoff.ms | --- | --- | --- | --- | --- | ---
controller.socket.timeout.ms | --- | --- | --- | --- | --- | ---
default.replication.factor | --- | --- | --- | --- | --- | ---
delegation.token.expiry.time.ms | --- | --- | --- | --- | --- | ---
delegation.token.master.key | --- | --- | --- | --- | --- | ---
delegation.token.max.lifetime.ms | --- | --- | --- | --- | --- | ---
delete.records.purgatory.purge.interval.requests | --- | --- | --- | --- | --- | ---
fetch.purgatory.purge.interval.requests | --- | --- | --- | --- | --- | ---
group.initial.rebalance.delay.ms | --- | --- | --- | --- | --- | ---
group.max.session.timeout.ms | --- | --- | --- | --- | --- | ---
group.min.session.timeout.ms | --- | --- | --- | --- | --- | ---
inter.broker.listener.name | --- | --- | --- | --- | --- | ---
inter.broker.protocol.version | --- | --- | --- | --- | --- | ---
log.cleaner.backoff.ms | --- | --- | --- | --- | --- | ---
log.cleaner.dedupe.buffer.size | --- | --- | --- | --- | --- | ---
log.cleaner.delete.retention.ms | --- | --- | --- | --- | --- | ---
log.cleaner.enable | --- | --- | --- | --- | --- | ---
log.cleaner.io.buffer.load.factor | --- | --- | --- | --- | --- | ---
log.cleaner.io.buffer.size | --- | --- | --- | --- | --- | ---
log.cleaner.io.max.bytes.per.second | --- | --- | --- | --- | --- | ---
log.cleaner.min.cleanable.ratio | --- | --- | --- | --- | --- | ---
log.cleaner.min.compaction.lag.ms| --- | --- | --- | --- | --- | ---
log.cleaner.threads | --- | --- | --- | --- | --- | ---
log.cleanup.policy | --- | --- | --- | --- | --- | ---
log.index.interval.bytes | --- | --- | --- | --- | --- | ---
log.index.size.max.bytes | --- | --- | --- | --- | --- | ---
log.message.format.version | --- | --- | --- | --- | --- | ---
log.message.timestamp.difference.max.ms | --- | --- | --- | --- | --- | ---
log.message.timestamp.type | --- | --- | --- | --- | --- | ---
log.preallocate | --- | --- | --- | --- | --- | ---
log.retention.check.interval.ms | --- | --- | --- | --- | --- | ---
max.connections.per.ip | --- | --- | --- | --- | --- | ---
max.connections.per.ip.overrides | --- | --- | --- | --- | --- | ---
max.incremental.fetch.session.cache.slots | --- | --- | --- | --- | --- | ---
num.partitions | --- | --- | --- | --- | --- | ---
password.encoder.old.secret | --- | --- | --- | --- | --- | ---
password.encoder.secret | --- | --- | --- | --- | --- | ---
principal.builder.class | --- | --- | --- | --- | --- | ---
producer.purgatory.purge.interval.requests | --- | --- | --- | --- | --- | ---
queued.max.request.bytes | --- | --- | --- | --- | --- | ---
replica.fetch.backoff.ms | --- | --- | --- | --- | --- | ---
replica.fetch.max.bytes | --- | --- | --- | --- | --- | ---
replica.fetch.response.max.bytes | --- | --- | --- | --- | --- | ---
reserved.broker.max.id | --- | --- | --- | --- | --- | ---
sasl.enabled.mechanisms | --- | --- | --- | --- | --- | ---
sasl.jaas.config | --- | --- | --- | --- | --- | ---
sasl.kerberos.kinit.cmd | --- | --- | --- | --- | --- | ---
sasl.kerberos.min.time.before.relogin | --- | --- | --- | --- | --- | ---
sasl.kerberos.principal.to.local.rules | --- | --- | --- | --- | --- | ---
sasl.kerberos.service.name | --- | --- | --- | --- | --- | ---
sasl.kerberos.ticket.renew.jitter | --- | --- | --- | --- | --- | ---
sasl.kerberos.ticket.renew.window.factor | --- | --- | --- | --- | --- | ---
sasl.mechanism.inter.broker.protocol | --- | --- | --- | --- | --- | ---
security.inter.broker.protocol | --- | --- | --- | --- | --- | ---
ssl.cipher.suites | --- | --- | --- | --- | --- | ---
ssl.client.auth | --- | --- | --- | --- | --- | ---
ssl.enabled.protocols | --- | --- | --- | --- | --- | ---
ssl.key.password | --- | --- | --- | --- | --- | ---
ssl.keymanager.algorithm | --- | --- | --- | --- | --- | ---
ssl.keystore.location | --- | --- | --- | --- | --- | ---
ssl.keystore.password | --- | --- | --- | --- | --- | ---
ssl.keystore.type | --- | --- | --- | --- | --- | ---
ssl.protocol | --- | --- | --- | --- | --- | ---
ssl.provider | --- | --- | --- | --- | --- | ---
ssl.trustmanager.algorithm | --- | --- | --- | --- | --- | ---
ssl.truststore.location | --- | --- | --- | --- | --- | ---
ssl.truststore.password | --- | --- | --- | --- | --- | ---
ssl.truststore.type | --- | --- | --- | --- | --- | ---
alter.config.policy.class.name | --- | --- | --- | --- | --- | ---
alter.log.dirs.replication.quota.window.num | --- | --- | --- | --- | --- | ---
alter.log.dirs.replication.quota.window.size.seconds | --- | --- | --- | --- | --- | ---
authorizer.class.name | --- | --- | --- | --- | --- | ---
create.topic.policy.class.name | --- | --- | --- | --- | --- | ---
delegation.token.expiry.check.interval.ms | --- | --- | --- | --- | --- | ---
listener.security.protocol.map | --- | --- | --- | --- | --- | ---
metric.reporters | --- | --- | --- | --- | --- | ---
metrics.num.samples | --- | --- | --- | --- | --- | ---
metrics.recording.level | --- | --- | --- | --- | --- | ---
metrics.sample.window.ms | --- | --- | --- | --- | --- | ---
password.encoder.cipher.algorithm | --- | --- | --- | --- | --- | ---
password.encoder.iterations | --- | --- | --- | --- | --- | ---
password.encoder.key.length | --- | --- | --- | --- | --- | ---
password.encoder.keyfactory.algorithm | --- | --- | --- | --- | --- | ---
quota.window.num | --- | --- | --- | --- | --- | ---
quota.window.size.seconds | --- | --- | --- | --- | --- | ---
replication.quota.window.num | --- | --- | --- | --- | --- | ---
replication.quota.window.size.seconds | --- | --- | --- | --- | --- | ---
ssl.endpoint.identification.algorithm | --- | --- | --- | --- | --- | ---
ssl.secure.random.implementation | --- | --- | --- | --- | --- | ---
transaction.abort.timed.out.transaction.cleanup.interval.ms | --- | --- | --- | --- | --- | ---
transaction.remove.expired.transaction.cleanup.interval.ms | --- | --- | --- | --- | --- | ---
zookeeper.sync.time.ms | --- | --- | --- | --- | --- | ---
| --- | --- | --- | --- | --- | ---

---
