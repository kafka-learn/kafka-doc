# Kafka Connect Configs

# Kafka Connect 配置

以下是Kafka Connect框架的配置。

NAME |	DESCRIPTION	| TYPE	| DEFAULT	| VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
config.storage.topic | The name of the Kafka topic where connector configurations are stored	| string	| 	|	| high
group.id | A unique string that identifies the Connect cluster group this worker belongs to. | string	| 	|	| high
key.converter | Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the keys in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro. | class	| 	|	| high
offset.storage.topic | The name of the Kafka topic where connector offsets are stored | string	| 	|	| high
status.storage.topic | The name of the Kafka topic where connector and task status are stored | string	| 	|	| high
value.converter | Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the values in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro. | class | | | high
internal.key.converter | Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the keys in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro. This setting controls the format used for internal bookkeeping data used by the framework, such as configs and offsets, so users can typically use any functioning Converter implementation. |  class | | | low
internal.value.converter | Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the values in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro. This setting controls the format used for internal bookkeeping data used by the framework, such as configs and offsets, so users can typically use any functioning Converter implementation. |  class | | | low
bootstrap.servers | A list of host/port pairs to use for establishing the initial connection to the Kafka cluster. The client will make use of all servers irrespective of which servers are specified here for bootstrapping—this list only impacts the initial hosts used to discover the full set of servers. This list should be in the form host1:port1,host2:port2,.... Since these servers are just used for the initial connection to discover the full cluster membership (which may change dynamically), this list need not contain the full set of servers (you may want more than one, though, in case a server is down). | list |	localhost:9092	| |	high
heartbeat.interval.ms | The expected time between heartbeats to the group coordinator when using Kafka's group management facilities. Heartbeats are used to ensure that the worker's session stays active and to facilitate rebalancing when new members join or leave the group. The value must be set lower than session.timeout.ms, but typically should be set no higher than 1/3 of that value. It can be adjusted even lower to control the expected time for normal rebalances. | int	| 3000| | high
rebalance.timeout.ms | The maximum allowed time for each worker to join the group once a rebalance has begun. This is basically a limit on the amount of time needed for all tasks to flush any pending data and commit offsets. If the timeout is exceeded, then the worker will be removed from the group, which will cause offset commit failures. | int	| 60000| | high
session.timeout.ms | The timeout used to detect worker failures. The worker sends periodic heartbeats to indicate its liveness to the broker. If no heartbeats are received by the broker before the expiration of this session timeout, then the broker will remove the worker from the group and initiate a rebalance. Note that the value must be in the allowable range as configured in the broker configuration by group.min.session.timeout.ms and group.max.session.timeout.ms. | int | 10000 | |high
ssl.key.password|	The password of the private key in the key store file. This is optional for client.|	password|	null |	|	high
ssl.keystore.location	| The location of the key store file. This is optional for client and can be used for two-way authentication for client.	| string |	null | | high
ssl.keystore.password	| The store password for the key store file. This is optional for client and only needed if ssl.keystore.location is configured.	| password	|null	| |	high
ssl.truststore.location	| The location of the trust store file.	| string	| null| |	high
ssl.truststore.password	| The password for the trust store file. If a password is not set access to the truststore is still available, but integrity checking is disabled.	| password |	null | | high
connections.max.idle.ms	| Close idle connections after the number of milliseconds specified by this config. |	long|	540000	| |	medium
receive.buffer.bytes	| The size of the TCP receive buffer (SO_RCVBUF) to use when reading data. If the value is -1, the OS default will be used. |	int| 	32768 |	[0,...]	| medium
request.timeout.ms	| The configuration controls the maximum amount of time the client will wait for the response of a request. If the response is not received before the timeout elapses the client will resend the request if necessary or fail the request if retries are exhausted.	| int|	40000|	[0,...]	| medium
sasl.jaas.config|	JAAS login context parameters for SASL connections in the format used by JAAS configuration files. JAAS configuration file format is described here. The format for the value is: ' (=)*;'	| password|	null	| |	medium
sasl.kerberos.service.name	| The Kerberos principal name that Kafka runs as. This can be defined either in Kafka's JAAS config or in Kafka's config.	| string|	null| |		medium
sasl.mechanism	| SASL mechanism used for client connections. This may be any mechanism for which a security provider is available. GSSAPI is the default mechanism.	| string	| GSSAPI |	|	medium
security.protocol	| Protocol used to communicate with brokers. Valid values are: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL.	| string	| PLAINTEXT	| |	medium
send.buffer.bytes	| The size of the TCP send buffer (SO_SNDBUF) to use when sending data. If the value is -1, the OS default will be used.	| int| 	131072|	[0,...]|	medium
ssl.enabled.protocols	|The list of protocols enabled for SSL connections.|	list	| TLSv1.2,TLSv1.1,TLSv1	| |	medium
ssl.keystore.type	| The file format of the key store file. This is optional for client.	|string	| JKS| |		medium
ssl.protocol |	The SSL protocol used to generate the SSLContext. Default setting is TLS, which is fine for most cases. Allowed values in recent JVMs are TLS, TLSv1.1 and TLSv1.2. SSL, SSLv2 and SSLv3 may be supported in older JVMs, but their usage is discouraged due to known security vulnerabilities.	| string	| TLS | |	medium
ssl.provider	| The name of the security provider used for SSL connections. Default value is the default security provider of the JVM.|	string	| null| |	medium
ssl.truststore.type	| The file format of the trust store file.	| string	| JKS| |	medium
worker.sync.timeout.ms	| When the worker is out of sync with other workers and needs to resynchronize configurations, wait up to this amount of time before giving up, leaving the group, and waiting a backoff period before rejoining.	| int |	3000 | |		medium
worker.unsync.backoff.ms |	When the worker is out of sync with other workers and fails to catch up within worker.sync.timeout.ms, leave the Connect cluster for this long before rejoining.	| int	| 300000 |	| 	medium
access.control.allow.methods	| Sets the methods supported for cross origin requests by setting the Access-Control-Allow-Methods header. The default value of the Access-Control-Allow-Methods header allows cross origin requests for GET, POST and HEAD.	| string	| ""	|	 | low
access.control.allow.origin	| Value to set the Access-Control-Allow-Origin header to for REST API requests.To enable cross origin access, set this to the domain of the application that should be permitted to access the API, or '*' to allow access from any domain. The default value only allows access from the domain of the REST API.	|string	| ""	| |	low
client.id	| An id string to pass to the server when making requests. The purpose of this is to be able to track the source of requests beyond just ip/port by allowing a logical application name to be included in server-side request logging. |	string	|""		| | low
config.storage.replication.factor	| Replication factor used when creating the configuration storage topic	| short	| 3 | 	[1,...] |	low
header.converter |	HeaderConverter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the header values in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro. By default, the SimpleHeaderConverter is used to serialize header values to strings and deserialize them by inferring the schemas.| 	class |org.apache.kafka.connect.storage.SimpleHeaderConverter	| |	low
listeners	| List of comma-separated URIs the REST API will listen on. The supported protocols are HTTP and HTTPS. Specify hostname as 0.0.0.0 to bind to all interfaces. Leave hostname empty to bind to default interface. Examples of legal listener lists: HTTP://myhost:8083,HTTPS://myhost:8084	| list |	null | |		low
metadata.max.age.ms	| The period of time in milliseconds after which we force a refresh of metadata even if we haven't seen any partition leadership changes to proactively discover any new brokers or partitions.	| long	| 300000	| [0,...]	|low
metric.reporters	| A list of classes to use as metrics reporters. Implementing the org.apache.kafka.common.metrics.MetricsReporter interface allows plugging in classes that will be notified of new metric creation. The JmxReporter is always included to register JMX statistics.	| list |	""	| |	low
metrics.num.samples |	The number of samples maintained to compute metrics.	| int	2	| [1,...]	| low
metrics.recording.level	| The highest recording level for metrics.	| string |	INFO |	[INFO, DEBUG]	| low
metrics.sample.window.ms |	The window of time a metrics sample is computed over.	|long	| 30000	| [0,...] | 	low
offset.flush.interval.ms |	Interval at which to try committing offsets for tasks.	|long	| 60000	|	| low
offset.flush.timeout.ms	| Maximum number of milliseconds to wait for records to flush and partition offset data to be committed to offset storage before cancelling the process and restoring the offset data to be committed in a future attempt.	| long |   5000	| |	low
offset.storage.partitions	| The number of partitions used when creating the offset storage topic	| int	| 25	| [1,...]	| low
offset.storage.replication.factor	| Replication factor used when creating the offset storage topic | 	short	| 3	| [1,...]	| low
plugin.path	| List of paths separated by commas (,) that contain plugins (connectors, converters, transformations). The list should consist of top level directories that include any combination of: a) directories immediately containing jars with plugins and their dependencies b) uber-jars with plugins and their dependencies c) directories immediately containing the package directory structure of classes of plugins and their dependencies Note: symlinks will be followed to discover dependencies or plugins. Examples: plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins,/opt/connectors	| list |	null	| |	low
reconnect.backoff.max.ms |	The maximum amount of time in milliseconds to wait when reconnecting to a broker that has repeatedly failed to connect. If provided, the backoff per host will increase exponentially for each consecutive connection failure, up to this maximum. After calculating the backoff increase, 20% random jitter is added to avoid connection storms. |	long |	1000 |	[0,...] |	low
reconnect.backoff.ms | The base amount of time to wait before attempting to reconnect to a given host. This avoids repeatedly connecting to a host in a tight loop. This backoff applies to all connection attempts by the client to a broker. | long | 50| 	[0,...]	| low
rest.advertised.host.name	| If this is set, this is the hostname that will be given out to other workers to connect to.	| string	| null |	|	low
rest.advertised.listener	| Sets the advertised listener (HTTP or HTTPS) which will be given to other workers to use.	| string	| null	| |	low
rest.advertised.port	| If this is set, this is the port that will be given out to other workers to connect to. |	int	| null	| |	low
rest.host.name	| Hostname for the REST API. If this is set, it will only bind to this interface.	| string	| null |	|	low
rest.port	| Port for the REST API to listen on.	| int |	8083 | | low
retry.backoff.ms	| The amount of time to wait before attempting to retry a failed request to a given topic partition. This avoids repeatedly sending requests in a tight loop under some failure scenarios.	| long | 	100	| [0,...]	| low
sasl.kerberos.kinit.cmd| 	Kerberos kinit command path.	| string |	/usr/bin/kinit	| |	low
sasl.kerberos.min.time.before.relogin	| Login thread sleep time between refresh attempts.	| long	| 60000	| |	low
sasl.kerberos.ticket.renew.jitter	| Percentage of random jitter added to the renewal time.	| double	| 0.05	| 	| low
sasl.kerberos.ticket.renew.window.factor	| Login thread will sleep until the specified window factor of time from last refresh to ticket's expiry has been reached, at which time it will try to renew the ticket. | 	double	| 0.8 | |		low
ssl.cipher.suites	| A list of cipher suites. This is a named combination of authentication, encryption, MAC and key exchange algorithm used to negotiate the security settings for a network connection using TLS or SSL network protocol. By default all the available cipher suites are supported.	| list | 	null | | low
ssl.client.auth	| Configures kafka broker to request client authentication. The following settings are common:ssl.client.auth=required If set to required client authentication is required;ssl.client.auth=requested This means client authentication is optional. unlike requested , if this option is set client can choose not to provide authentication information about itself;ssl.client.auth=none This means client authentication is not needed. | string	| none | |		low
ssl.endpoint.identification.algorithm	|The endpoint identification algorithm to validate server hostname using server certificate.	| string	|null	| |	low
ssl.keymanager.algorithm	| The algorithm used by key manager factory for SSL connections. Default value is the key manager factory algorithm configured for the Java Virtual Machine.	| string	| SunX509 | |	low
ssl.secure.random.implementation	| The SecureRandom PRNG implementation to use for SSL cryptography operations.	| string	| null	| | low
ssl.trustmanager.algorithm	| The algorithm used by trust manager factory for SSL connections. Default value is the trust manager factory algorithm configured for the Java Virtual Machine.	| string	| PKIX | |	low
status.storage.partitions	| The number of partitions used when creating the status storage topic	| int	| 5	 | [1,...]	| low
status.storage.replication.factor	| Replication factor used when creating the status storage topic |	short	| 3	| [1,...] |	low
task.shutdown.graceful.timeout.ms	| Amount of time to wait for tasks to shutdown gracefully. This is the total amount of time, not per task. All task have shutdown triggered, then they are waited on sequentially. |	long	| 5000	| |	low

名称 | 描述 | 类型 | 默认值 | 有效值 | 重要性
:- | :- | :- | :- | :- | :-
config.storage.topic | 存储Connector配置的Kafka主题的名称 | string | | | high
group.id | 标识此工作线程所属的Connect集群组的唯一字符串。| string | | | high
key.converter | 一个转换类，用于在Kafka Connect格式和写入Kafka的序列化数据格式之间进行转换。这将控制写入Kafka或从Kafka读取的消息(record)中的key格式，因为这与Connector无关，所以它允许任何Connector使用任意的序列化格式。常见的格式包括JSON和Avro。 | class | | | high
offset.storage.topic | 存储Connector偏移量(offset)的Kafka主题的名称 | string | | | high
status.storage.topic | 存储connector或任务(task)状态的Kafka主题的名称 | string | | | high
value.converter | 一个转换类，用于在Kafka Connect格式和写入Kafka的序列化数据格式之间进行转换。这将控制写入Kafka或从Kafka读取的消息(record)中的value格式，因为这与Connector无关，所以它允许任何Connector使用任意的序列化格式。常见的格式包括JSON和Avro。| class | | | high
internal.key.converter | 一个转换类，用于在Kafka Connect格式和写入Kafka的序列化格式之间进行转换。这将控制写入Kafka或从Kafka读取的消息中的key格式，因为这与connector无关，所以它允许任何connector使用任意的序列化格式。常见的格式包括JSON和Avro。此设置控制框架使用的内部bookkeeping数据的格式，例如配置和偏移量，因此用户通常可以使用任何有效的转换类实现。 | class | | | low
internal.value.converter | 一个转换类，用于在Kafka Connect格式和写入Kafka的序列化格式之间进行转换。这将控制写入Kafka或从Kafka读取的消息中的value的格式，因为这与connector无关，所以它允许任何connector使用任意的序列化格式。常见的格式包括JSON和Avro。此设置控制框架使用的内部bookkeeping数据的格式，例如配置和偏移量，因此用户通常可以使用任何有效的转换类实现。| class | | | low
bootstrap.servers| 用于建立到Kafka集群的初始连接的主机/端口对列表。客户端将使用所有服务器，而不管在此指定了哪些服务器用于引导 - 此列表仅影响用于发现全套服务器的初始主机。此列表应采用`host1：port1`，`host2：port2`，`...`的形式。由于这些服务器仅用于初始连接以发现完整集群成员资格(可能会动态更改)，因此此列表不需包含完整集合的服务器(如果服务器停机，您可能需要多个服务器)。| list | localhost:9092 | | high
heartbeat.interval.ms | 在使用Kafka的组管理设施时，心跳与团队协调员之间的预期时间。心跳信号用于确保工作线程的会话保持活动状态，并便于在新成员加入或离开组时重新平衡。该值必须设置为低于`session.timeout.ms`，但通常应设置为不高于该值的1/3。它可以调整得更低，以控制正常再次平衡(rebalance)的预期时间。| int | 3000| | high
rebalance.timeout.ms | 重新平衡开始后，每个工作线程加入该组的最大时间。这基本上是所有任务清除任何未决的数据(pending data)和提交偏移量所需时间的限制。如果超时，则工作线程将从组中移除，这将导致偏移提交失败。| int | 60000 | | high
session.timeout.ms | 用于检测工作线程失败的超时时间。工作线程定期发送心跳以表明其对代理仍是存活着的。如果代理在该会话超时到期之前未收到检测信号，代理将从该组中删除该工作线程并启动重新平衡。请注意，该值必须位于代理配置中由`group.min.session.timeout.ms`和`group.max.session.timeout.ms`中配置的允许范围内。 | int | 10000 | | high
ssl.key.password | key存储文件中的私有key的password。这对客户端是个可选项。| password | null | |high
ssl.keystore.location | key存储文件的位置。这对客户端是个可选项，并且可以用于客户端的双向认证。 | string | null | | high
ssl.keystore.password | key存储文件的sotre password。这对客户端是个可选项，只有在配置了ssl.keystore.location时才需要此配置。| password | null | | high
ssl.truststore.location | 信任存储文件(trust store file)的位置。 | string | null | | high
ssl.truststore.password | 信任存储文件的password。如果未设置，对truststore的访问仍然可用，但不会有完整性检查。| password | null | | high
connections.max.idle.ms | 在此配置指定的毫秒数后关闭空闲连接。 | long | 540000 | | medium
receive.buffer.bytes | 读取数据时使用的TCP接收缓冲区(SO_RCVBUF)的大小。如果该值为-1，则将使用操作系统默认值。 | int | 32768 | [ 0,...] | medium
request.timeout.ms | 该配置控制客户端等待请求响应的最长时间。如果在超时之前未收到响应，客户端将在必要时重新发送请求，或者如果重试耗尽，请求失败。 | int | 40000 | [0,...] | medium
sasl.jaas.config | 用于JAAS配置文件使用的格式的SASL连接的JAAS登录上下文参数。[这里](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/LoginConfigFile.html)描述JAAS配置文件格式。该值的格式是：'(=)*;' | password | null | | medium
sasl.kerberos.service.name | Kafka运行的Kerberos principal名称。这可以在Kafka的JAAS配置中定义或在Kafka的配置中定义。 | string | null | | medium
sasl.mechanism | 用于客户端连接的SASL机制。这可以是安全提供者提供的任何机制。GSSAPI是默认机制。 | string | GSSAPI | | medium
security.protocol | 用于与broker沟通的协议。有效值包括：PLAINTEXT，SSL，SASL_PLAINTEXT，SASL_SSL。 | string | PLAINTEXT | | medium
send.buffer.bytes | 发送数据时使用的TCP发送缓冲区的大小(SO_SNDBUF)。如果该值为-1，则将使用操作系统默认值。 | int | 131072 | [0,...] | medium
ssl.enabled.protocols | 启用SSL连接的协议列表。| list | TLSv1.2,TLSv1.1,TLSv1 | | medium
ssl.keystore.type | key存储文件的文件格式。这对客户端是个可选项。 | string | JKS | | medium
ssl.protocol | 用于生成SSLContext的SSL协议。默认设置是TLS，在大多数情况下都很不错。最新的JVM中允许的值是TLS，TLSv1.1和TLSv1.2。较早的JVM可能支持SSL，SSLv2和SSLv3，但由于已知的安全漏洞，它们的使用受到阻碍。 | string | TLS | | medium
ssl.provider | 用于SSL连接的安全提供程序的名称。默认值是JVM的默认安全提供程序。| string | null | | medium
ssl.truststore.type | 信任存储文件(trust store file)的文件格式。| string | JKS | | medium
worker.sync.timeout.ms | 当工作线程与其它工作线程不同步并需要重新同步配置时，需等待此时间后再放弃、离开工作组并再重新加入前等待退回。| int | 3000 | | medium
worker.unsync.backoff.ms | 当工作线程与其它工作线程不同步并且无法在worker.sync.timeout.ms内赶上时，请在重新加入之前将Connect群集保留较长时间。 | int | 300000 | | medium
access.control.allow.methods | 通过设置Access-Control-Allow-Methods头部来设置跨源请求支持的方法。 Access-Control-Allow-Methods头部的默认值允许GET、POST和HEAD的交叉源请求。| string | "" | | low
access.control.allow.origin | 将Access-Control-Allow-Origin头部设置为REST API请求的值。要启用跨源访问，需将其设置为应允许访问该API的应用程序的域，或者设置为"*"来允许从任何位置访问域。默认值只允许从REST API的域进行访问。 | string | "" | | low
client.id | 发出请求时传递给服务器的id字符串。这样做的目的是通过允许将逻辑应用程序名称包含在服务器端请求日志中，从而能够跟踪ip/port之外的请求源。 | string | "" | | low
config.storage.replication.factor | 创建配置存储主题时使用的备份因子 | short | 3 | [1,...] | low
header.converter | 一个头部转换类(HeaderConverter)，用于在Kafka Connect格式和写入Kafka的序列化数据之间进行转换。这将控制写入Kafka或从Kafka读取的消息中的头部值(header value)的格式，并且由于这与connector无关，所以它允许任何connector使用任意的序列化格式。常见的格式包括JSON和Avro。默认情况下，SimpleHeaderConverter用于将头部值序列化为字符串，并通过推断其schema对其进行反序列化。| class |org.apache.kafka.connect.storage.SimpleHeaderConverter | | low
listeners | REST API将侦听的逗号分隔的URI列表。 支持的协议是HTTP和HTTPS。 指定主机名为0.0.0.0以绑定到所有接口。 保持主机名为空以绑定到默认界面。 合法侦听器列表的示例：HTTP：// myhost：8083，HTTPS：// myhost：8084 | list | null| | low
metadata.max.age.ms | 以毫秒为单位的时间段之后，即使我们没有看到任何分区领导改变以主动发现任何新代理或分区，我们也会强制更新元数据。 |  long | 300000 | [0,...] | low
metric.reporters | 用作指标记录的类的列表。实现`org.apache.kafka.common.metrics.MetricsReporter`接口,允许插入将被通知的新的指标创建的类。JmxReporter始终包含在注册JMX统计信息中。 | list | "" | | low
metrics.num.samples | 维持用于计算指标的样本数量。 | int | 2 | [1,...] | low
metrics.recording.level | 指标的最高记录级别。 | string | INFO | [INFO, DEBUG] | low
metrics.sample.window.ms | 计算指标样本的时间窗口。| long | 30000 | [0,...] | low
offset.flush.interval.ms | 尝试为任务提交偏移量的时间间隔。| long | 60000 | low
offset.flush.timeout.ms | 在取消进程并恢复将来尝试提交的偏移数据之前，等待记录刷新的最大毫秒数以及将提交的偏移数据提交到偏移存储的最大毫秒数。| long | 5000 | | low
offset.storage.partitions | 创建偏移量存储主题时使用的分区数量 | int | 25 | [1,...] | low
offset.storage.replication.factor | 创建偏移量存储主题时使用的备份因子 | short | 3 | [1,...] | low
plugin.path | 包含插件(connector，转换类，转换器)的用逗号(,)分隔的路径列表。该列表应该由顶级目录组成，这些目录包括以下任意组合:a)立即包含带插件及其依赖项的jar的目录,b)带插件及其依赖项的uber-jar,c)立即包含插件类及其插件的包目录结构的目录 依赖关系注意：将遵循符号链接来发现依赖关系或插件。 示例：plugin.path=/usr/local/share/java，/usr/local/share/kafka/plugins，/opt/connectors | list | null | | low
reconnect.backoff.max.ms | 重新连接到重复无法连接的代理时等待的最长时间(以毫秒为单位)。如果提供，则每个主机的backoff将按照指数规律增加，对于每个连续的连接故障，达到此最大值。在计算backoff增量后，添加20％的随机抖动以避免连接风暴。 | long | 1000 | [0,...] | low
reconnect.backoff.ms | 尝试重新连接到给定主机之前等待的基本时间。这避免了在紧密循环中重复连接到主机。该backoff适用于客户端向broker的所有连接尝试。 | long | 50 | [0,...]  | low
rest.advertised.host.name | 如果已设置，则这是将要发给其它工作线程连接的主机名。 | string | null | | low
rest.advertised.listener | 设置将提供给其他工作线程使用的侦听器(HTTP或HTTPS)。| string | null | |low
rest.advertised.port | 如果设置了这个，这个端口将被发给其它工作线程连接。| int | null | | low
rest.host.name | REST API的主机名。如果设置了，它只会绑定到这个接口。| string | null | | low
rest.port | REST API可以侦听的端口。 | int | 8083 | |low
retry.backoff.ms | 尝试重试对给定主题分区的失败请求之前等待的时间量。这可以避免在某些故障情况下重复发送请求。 | long | 100 | [0,...] | low
sasl.kerberos.kinit.cmd | Kerberos kinit 命令路径 | string | /usr/bin/kinit | | low
sasl.kerberos.min.time.before.relogin | 登录线程在刷新尝试之间的休眠时间 | long | 60000 | | low
sasl.kerberos.ticket.renew.jitter | 随机抖动增加到更新时间的百分比。 | double | 0.05 | | low
sasl.kerberos.ticket.renew.window.factor | 登录线程将一直处于睡眠状态，直到达到从上次刷新到ticket到期的指定窗口时间因子，届时它将尝试续ticket。 | double | 0.8 | | low
ssl.cipher.suites | 密码套件列表。这是用于使用TLS或SSL网络协议协商网络连接的安全设置的认证，加密，MAC和密钥交换算法的命名组合。默认情况下，所有可用的密码套件都受支持。 | list | null | | low
ssl.client.auth | 配置kafka代理以请求客户端认证。以下设置是常见的：`ssl.client.auth=required`如果设置为必需的,则客户端身份验证是必需的。`ssl.client.auth=requested`这意味着客户端身份验证是可选的。不同于请求，如果这个选项被设置，客户端可以选择不提供关于它自己的认证信息。`ssl.client.auth=none`这意味着不需要客户端身份验证。| string | none | | low
ssl.endpoint.identification.algorithm | 使用服务器证书验证服务器主机名的端点识别算法。| string | null | | low
ssl.keymanager.algorithm | key管理器工厂用于SSL连接的算法。默认值是为Java虚拟机配置的key管理器工厂算法。 | string | SunX509 | | low
ssl.secure.random.implementation | 用于SSL加密操作的SecureRandom PRNG实现。 | string | null | |low
ssl.trustmanager.algorithm | 信任(trust)管理器工厂用于SSL连接的算法。缺省值是为Java虚拟机配置的信任管理器工厂算法。 | string | PKIX | | low
status.storage.partitions | 创建状态存储主题时使用的分区数量 | int | 5 | [1,...] | low
status.storage.replication.factor | 创建状态存储主题时使用的备份因子 | short | 3 | [1,...] | low
task.shutdown.graceful.timeout.ms | 等待任务正常关闭的时间量。这是个总时间，而不是只针对单个任务。所有任务都已关闭触发，然后按顺序等待。| long | 5000 | | low
