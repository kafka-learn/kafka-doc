# 4.9 Quotas

# 4.9 配额

Kafka cluster has the ability to enforce quotas on requests to control the broker resources used by clients. Two types of client quotas can be enforced by Kafka brokers for each group of clients sharing a quota:

Kafka集群可以对请求执行配额以控制客户端使用的代理资源。Kafka代理可以对两类客户端配额中的每组客户端提供配额：

1. Network bandwidth quotas define byte-rate thresholds (since 0.9)
2. Request rate quotas define CPU utilization thresholds as a percentage of network and I/O threads (since 0.11)


1. 网络带宽配额定义了字节速率阈值（从0.9版本开始）
2. 请求速率配额将CPU利用率阈值定义为网络和I/O线程的百分比（从0.11版本开始）

## Why are quotas necessary?

## 为什么需要配额

It is possible for producers and consumers to produce/consume very high volumes of data or generate requests at a very high rate and thus monopolize broker resources, cause network saturation and generally DOS other clients and the brokers themselves. Having quotas protects against these issues and is all the more important in large multi-tenant clusters where a small set of badly behaved clients can degrade user experience for the well behaved ones. In fact, when running Kafka as a service this even makes it possible to enforce API limits according to an agreed upon contract.

生产者和消费者可能以非常高的速度生产/消费大量数据或产生请求，从而垄断代理资源，导致网络饱和，并且通常会对其它客户端和代理本身产生影响。拥有配额可以防范这些问题，并且在大型多租户集群中更重要，其中一小部分表现不佳的客户端可能会降低用户体验良好的用户体验。事实上，在将Kafka作为服务运行时，甚至可以根据约定的协议强制执行API限制。

## Client groups

## 客户端组

The identity of Kafka clients is the user principal which represents an authenticated user in a secure cluster. In a cluster that supports unauthenticated clients, user principal is a grouping of unauthenticated users chosen by the broker using a configurable ```PrincipalBuilder```. Client-id is a logical grouping of clients with a meaningful name chosen by the client application. The tuple (user, client-id) defines a secure logical group of clients that share both user principal and client-id.

Kafka客户端的身份是代表安全集群中经过身份验证的用户的用户主体。在支持未经身份验证的客户端的集群中，用户主体是代理使用可配置的```PrincipalBuilder```选择的未经身份验证的用户的分组。客户端ID（Client-id）是由客户端应用程序选择的具有有意义名称的客户端的逻辑分组。元组（user，client-id）定义了共享用户主体和客户端ID的一个安全的客户端逻辑组。

Quotas can be applied to (user, client-id), user or client-id groups. For a given connection, the most specific quota matching the connection is applied. All connections of a quota group share the quota configured for the group. For example, if (user="test-user", client-id="test-client") has a produce quota of 10MB/sec, this is shared across all producer instances of user "test-user" with the client-id "test-client".

配额可以应用于（用户，客户端ID），用户或客户端组。对于给定的连接，将采用与连接匹配的最具体的配额。配额组的所有连接都共享为该组配置的配额。例如，如果（user =“test-user”，client-id=“test-client”）的生产配额为10MB/sec，则在用户“test-user”的所有生产者实例中与客户端ID“test-client”中共享。

## Quota Configuration

## 配额配置

Quota configuration may be defined for (user, client-id), user and client-id groups. It is possible to override the default quota at any of the quota levels that needs a higher (or even lower) quota. The mechanism is similar to the per-topic log config overrides. User and (user, client-id) quota overrides are written to ZooKeeper under ```/config/users``` and client-id quota overrides are written under ```/config/clients```. These overrides are read by all brokers and are effective immediately. This lets us change quotas without having to do a rolling restart of the entire cluster. See [here](http://kafka.apache.org/documentation/#quotas) for details. Default quotas for each group may also be updated dynamically using the same mechanism.

可以为（用户，客户端ID），用户和客户端组定义配额配置。可以在任何需要更高（或更低）配额的配额级别上覆盖默认配额。该机制与每个主题日志配置覆盖类似。用户和（用户，客户端ID）配额覆盖写入```/ config/users```下的ZooKeeper，客户端配额覆盖写在```/config/clients```下。这些覆盖可以被所有代理阅读，并立即生效。这使我们可以更改配额，而无需执行整个集群的滚动重新启动。有关详细信息，请参见[这里](http://kafka.apache.org/documentation/#quotas)。每个组的默认配额也可以使用相同的机制动态更新。

The order of precedence for quota configuration is:

配额配置的优先顺序是：

1. /config/users/&lt;user&gt;/clients/&lt;client-id&gt;
2. /config/users/&lt;user&gt;/clients/&lt;default&gt;
3. /config/users/&lt;user&gt;
4. /config/users/&lt;default&gt;/clients/&lt;client-id&gt;
5. /config/users/&lt;default&gt;/clients/&lt;default&gt;
6. /config/users/&lt;default&gt;
7. /config/clients/&lt;client-id&gt;
8. /config/clients/&lt;default&gt;

Broker properties (quota.producer.default, quota.consumer.default) can also be used to set defaults of network bandwidth quotas for client-id groups. These properties are being deprecated and will be removed in a later release. Default quotas for client-id can be set in Zookeeper similar to the other quota overrides and defaults.

代理属性（quota.producer.default，quota.consumer.default）也可用于为客户端组设置网络带宽配额的默认值。这些属性已被弃用，并将在以后的版本中删除。客户端ID的默认配额可以在Zookeeper中设置，类似于其他配额覆盖和默认值。

## Network Bandwidth Quotas

## 网络带宽配额

Network bandwidth quotas are defined as the byte rate threshold for each group of clients sharing a quota. By default, each unique client group receives a fixed quota in bytes/sec as configured by the cluster. This quota is defined on a per-broker basis. Each group of clients can publish/fetch a maximum of X bytes/sec per broker before clients are throttled.

网络带宽配额定义为每组共享配额的客户端的字节速率阈值。默认情况下，每个不同的客户端组都会收到由集群配置的固定配额（以字节/秒为单位）。此配额是以每个代理为基础定义的。在客户端被限制之前，每个客户端组可以发布/获取每个代理的最大X字节/秒。

## Request Rate Quotas

## 请求率配额

Request rate quotas are defined as the percentage of time a client can utilize on request handler I/O threads and network threads of each broker within a quota window. A quota of ```n%``` represents ```n%``` of one thread, so the quota is out of a total capacity of ```((num.io.threads + num.network.threads) * 100)%```. Each group of clients may use a total percentage of upto ```n%``` across all I/O and network threads in a quota window before being throttled. Since the number of threads allocated for I/O and network threads are typically based on the number of cores available on the broker host, request rate quotas represent the total percentage of CPU that may be used by each group of clients sharing the quota.

请求速率配额定义为客户端可以在请求处理程序上使用的时间百分比处理器配额窗口中每个代理的I/O线程和网络线程。```n％```的配额代表一个线程的```n％```，所以配额超出了总容量```（（num.io.threads + num.network.threads ）* 100）％```。在被限制之前，每组客户端可以在配额窗口中的所有I/O和网络线程中使用高达```n％```的总百分比。由于为I/O和网络线程分配的线程数通常基于代理主机上可用的内核数，因此请求速率限额表示可由共享配额的每组客户端使用的CPU的总百分比。

## Enforcement

## 约束

By default, each unique client group receives a fixed quota as configured by the cluster. This quota is defined on a per-broker basis. Each client can utilize this quota per broker before it gets throttled. We decided that defining these quotas per broker is much better than having a fixed cluster wide bandwidth per client because that would require a mechanism to share client quota usage among all the brokers. This can be harder to get right than the quota implementation itself!

默认情况下，每个不同的客户端组都会收到由集群配置的固定配额。此配额是以每个代理为基础定义的。每个客户可以在受到限制之前利用每个代理的配额。我们决定为每个代理定义这些配额，这比每个客户端拥有固定的集群带宽好得多，因为这需要一种机制来在所有代理中共享客户端配额使用。这可能比配额实施本身更难获得！

How does a broker react when it detects a quota violation? In our solution, the broker does not return an error rather it attempts to slow down a client exceeding its quota. It computes the amount of delay needed to bring a guilty client under its quota and delays the response for that time. This approach keeps the quota violation transparent to clients (outside of client-side metrics). This also keeps them from having to implement any special backoff and retry behavior which can get tricky. In fact, bad client behavior (retry without backoff) can exacerbate the very problem quotas are trying to solve.

代理在检测到配额违规时如何反应？在我们的解决方案中，代理不会返回错误，而是尝试减慢超过配额的客户端速度。它计算将违规客户置于配额之下所需的延迟时间，同时计算该延迟的响应时间。这种方法使配额违规对客户端透明（客户端指标除外）。这也使它们不必执行任何特殊的退避和重试行为，这可能会变得棘手。事实上，糟糕的客户行为（无退避情况下的重试）可能会加剧配额试图解决的问题。

Byte-rate and thread utilization are measured over multiple small windows (e.g. 30 windows of 1 second each) in order to detect and correct quota violations quickly. Typically, having large measurement windows (for e.g. 10 windows of 30 seconds each) leads to large bursts of traffic followed by long delays which is not great in terms of user experience.

字节速率和线程利用率是在多个小窗口（例如每个窗口为1秒的30个窗口）上测量的，以便快速检测和纠正配额违规。通常，具有较大的测量窗口（例如每个30秒的10个窗口）容易导致大量的业务量突发，然后会产生长时间的延迟，这在用户体验方面不是很好。