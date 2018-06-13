# 4.7 Replication

# 4.7 备份

Kafka replicates the log for each topic's partitions across a configurable number of servers (you can set this replication factor on a topic-by-topic basis). This allows automatic failover to these replicas when a server in the cluster fails so messages remain available in the presence of failures.

Kafka通过可配置数量的服务器为每个主题的分区备份日志（您可以对每一个主题设置此备份因子）。当集群中的服务器出现故障时，Kafka可以自动将日志转移到这些副本中，以便在服务器出现故障时仍能保持消息可用。

Other messaging systems provide some replication-related features, but, in our (totally biased) opinion, this appears to be a tacked-on thing, not heavily used, and with large downsides: slaves are inactive, throughput is heavily impacted, it requires fiddly manual configuration, etc. Kafka is meant to be used with replication by default—in fact we implement un-replicated topics as replicated topics where the replication factor is one.

其它消息系统提供了一些与备份相关的功能，但是，在我们（完全有偏见的）看来，这看起来是一个附加的东西，没有被大量使用，并且有很多的缺点：从属服务器是处于非活跃状态的，吞吐量受到严重影响，需要进行繁琐的手动配置等。Kafka默认就使用备份策略 - 事实上，我们将未备份的主题作为一个备份主题是通过将备份因子设为1实现的。

The unit of replication is the topic partition. Under non-failure conditions, each partition in Kafka has a single leader and zero or more followers. The total number of replicas including the leader constitute the replication factor. All reads and writes go to the leader of the partition. Typically, there are many more partitions than brokers and the leaders are evenly distributed among brokers. The logs on the followers are identical to the leader's log—all have the same offsets and messages in the same order (though, of course, at any given time the leader may have a few as-yet unreplicated messages at the end of its log).

备份的单位是主题分区(topic partition)。在没有错误的情况下，Kafka中的每个分区都有一个单独的leader和零个或多个follower。包括leader在内的备份（replicas）总数构成了复制因子。所有读取和写入操作都需分区的leader处理。通常，分区数量比代理数量要多，并且leader会在代理中分布。follower的日志与leader的日志相同 - 都具有相同的顺序的偏移量和消息（当然，在任何给定时间，leader日志的末尾可能有几条尚未复制的消息）。

Followers consume messages from the leader just as a normal Kafka consumer would and apply them to their own log. Having the followers pull from the leader has the nice property of allowing the follower to naturally batch together log entries they are applying to their log.

follower就像普通的Kafka消费者一样消费leader的消息，并将这些消息应用到自己的日志中。让follower从leader中获取消息有一个很好的特点，这可以让follower自然地将应用到它们日志中的日志条目分组在一起。

As with most distributed systems automatically handling failures requires having a precise definition of what it means for a node to be "alive". For Kafka node liveness has two conditions

与大多数分布式系统一样，自动处理故障需要精确定义节点“活着（alive）”意味着什么。对于Kafka节点而言，其活着有两个条件

1. A node must be able to maintain its session with ZooKeeper (via ZooKeeper's heartbeat mechanism)
2. If it is a slave it must replicate the writes happening on the leader and not fall "too far" behind


1. 节点必须能够维护与ZooKeeper的会话（通过ZooKeeper的心跳机制）
2. 如果它是一个从属节点，它必须要备份leader上的写操作，该从属节点不能落后leader“太远”

We refer to nodes satisfying these two conditions as being "in sync" to avoid the vagueness of "alive" or "failed". The leader keeps track of the set of "in sync" nodes. If a follower dies, gets stuck, or falls behind, the leader will remove it from the list of in sync replicas. The determination of stuck and lagging replicas is controlled by the replica.lag.time.max.ms configuration.

我们将满足这两个条件的节点称为处于“同步（in sync）”中的，以避免“活着”或“失败”的模糊性。leader跟踪“同步“节点集。如果一个follower死亡，阻塞或落后于leader，leader会将其从同步副本列表中删除。确定一个副本是阻塞或落后于leader是由replica.lag.time.max.ms配置控制的。

In distributed systems terminology we only attempt to handle a "fail/recover" model of failures where nodes suddenly cease working and then later recover (perhaps without knowing that they have died). Kafka does not handle so-called "Byzantine" failures in which nodes produce arbitrary or malicious responses (perhaps due to bugs or foul play).

在分布式系统术语中，我们只尝试处理节点突然停止工作然后恢复（可能不知道它们已经死亡）的“失败/恢复”模式。Kafka不处理所谓的“Byzantine”故障，在该故障中，节点会产生随意的或恶意的响应（可能是因为bug或不合理行为）。

We can now more precisely define that a message is considered committed when all in sync replicas for that partition have applied it to their log. Only committed messages are ever given out to the consumer. This means that the consumer need not worry about potentially seeing a message that could be lost if the leader fails. Producers, on the other hand, have the option of either waiting for the message to be committed or not, depending on their preference for tradeoff between latency and durability. This preference is controlled by the acks setting that the producer uses. Note that topics have a setting for the "minimum number" of in-sync replicas that is checked when the producer requests acknowledgment that a message has been written to the full set of in-sync replicas. If a less stringent acknowledgement is requested by the producer, then the message can be committed, and consumed, even if the number of in-sync replicas is lower than the minimum (e.g. it can be as low as just the leader).

现在我们可以更准确地定义，当该分区的所有同步副本将消息应用于其日志时，就认为该消息已被提交。只有提交的消息会被分发给消费者。这意味着消费者不必担心由于leader失败会看到可能丢失的消息。另一方面，生产者可以选择是否等待消息的提交，这具体取决于其在延迟与持久性之间的权衡。此首选项由生产者使用的acks设置控制。请注意，主题有一个对同步副本集的“最小数量的”设置，该设置会在生产者请求确认消息已写入完整的同步副本集时检查。如果生产者请求是不严格的确认，则即使同步副本的数量低于最小值（例如，它可以仅只有leader），该消息也可以被提交并消费。

The guarantee that Kafka offers is that a committed message will not be lost, as long as there is at least one in sync replica alive, at all times.

Kafka提供的保证是，只要至少有一个同步副本处于活着的状态，提交的消息就不会丢失。

Kafka will remain available in the presence of node failures after a short fail-over period, but may not remain available in the presence of network partitions.

在短暂的故障转移期间，Kafka在节点故障时仍能保持可用状态，但在网络分区存在时可能无法保持可用状态。

## Replicated Logs: Quorums, ISRs, and State Machines (Oh my!)

## 复制日志：Quorums，ISR和状态机（当然，还有我们自己的）

At its heart a Kafka partition is a replicated log. The replicated log is one of the most basic primitives in distributed data systems, and there are many approaches for implementing one. A replicated log can be used by other systems as a primitive for implementing other distributed systems in the [state-machine style](https://en.wikipedia.org/wiki/State_machine_replication).

Kafka分区的核心是一个备份日志（replicated log）。备份日志是分布式数据系统中最基本的原语之一，并且其有很多实现方法。备份日志可以被其它系统用来当作在[状态机模式](https://en.wikipedia.org/wiki/State_machine_replication)下实现分布式系统的原语。

A replicated log models the process of coming into consensus on the order of a series of values (generally numbering the log entries 0, 1, 2, ...). There are many ways to implement this, but the simplest and fastest is with a leader who chooses the ordering of values provided to it. As long as the leader remains alive, all followers need to only copy the values and ordering the leader chooses.

备份日志模拟了按照一系列值（通常编号为日志条目 0,1,2，...）的顺序到达的处理过程。有很多方法可以实现这一点，但最简单和最快的方法是leader将这些值排序后直接提供。只要leader还活着，所有的follower只需要复制这些值和leader选择的命令。

Of course if leaders didn't fail we wouldn't need followers! When the leader does die we need to choose a new leader from among the followers. But followers themselves may fall behind or crash so we must ensure we choose an up-to-date follower. The fundamental guarantee a log replication algorithm must provide is that if we tell the client a message is committed, and the leader fails, the new leader we elect must also have that message. This yields a tradeoff: if the leader waits for more followers to acknowledge a message before declaring it committed then there will be more potentially electable leaders.

如果leader没有失败，我们当然就不需要follower！但当leader死亡时，我们需要从follower中选择一个成为新的leader。但follower本身可能会落后于leader或崩溃掉，所以我们必须确保选择一个最新的follower。日志复制算法必须提供的基本保证是，如果我们告诉消费者一个消息被提交了，并且leader失败了，我们选择出的新leader也必须拥有这个消息。这产生了一个折衷：如果leader在宣布提交消息之前等待更多的follower确认此消息，那么将会有更多潜在的可选leader。

If you choose the number of acknowledgements required and the number of logs that must be compared to elect a leader such that there is guaranteed to be an overlap, then this is called a Quorum.

如果您选择所需的follower数量与日志数量进行对比来选举出leader以确保重叠，那么这称为仲裁(Quorum)。

A common approach to this tradeoff is to use a majority vote for both the commit decision and the leader election. This is not what Kafka does, but let's explore it anyway to understand the tradeoffs. Let's say we have 2_f_+1 replicas. If _f_+1 replicas must receive a message prior to a commit being declared by the leader, and if we elect a new leader by electing the follower with the most complete log from at least _f_+1 replicas, then, with no more than _f_ failures, the leader is guaranteed to have all committed messages. This is because among any _f_+1 replicas, there must be at least one replica that contains all committed messages. That replica's log will be the most complete and therefore will be selected as the new leader. There are many remaining details that each algorithm must handle (such as precisely defined what makes a log more complete, ensuring log consistency during leader failure or changing the set of servers in the replica set) but we will ignore these for now.

这种权衡的一种常见方法是对消息提交决定和leader选举都采用多数投票方式。这不是由Kafka来处理的，但无论如何我们还是要探索它以理解这种权衡。假设我们有 2_f_+1 个副本。如果 _f_+1 个副本必须在leader声明提交消息之前接收到此消息，并且如果我们通过从至少 _f_+1 副本中选择拥有最完整日志的follower来选择新的leader，则当副本失败数目不超过 _f_ 个时，leader能保证拥有所有提交的消息。这是因为在任何 _f_+1 个副本中，至少有一个副本包含所有提交的消息。该备份的日志将是最完整的，因此将被选为新leader。每种算法都必须处理许多其他细节（例如，精确定义什么使得日志更加完整，确保leader失败期间的日志一致性或更改副本集中的服务器集），但现在我们将忽略这些细节。

This majority vote approach has a very nice property: the latency is dependent on only the fastest servers. That is, if the replication factor is three, the latency is determined by the faster slave not the slower one.

这种多数投票方式有一个非常好的属性：延迟仅取决于最快的服务器。也就是说，如果复制因子是3，则等待时间由较快的从属者而不是较慢的从属者确定。

There are a rich variety of algorithms in this family including ZooKeeper's [Zab](http://web.archive.org/web/20140602093727/http://www.stanford.edu/class/cs347/reading/zab.pdf), [Raft](https://api.media.atlassian.com/file/223578e2-b74c-44fc-aff4-3f9d74ff03a8/binary?token=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiI1M2U1NGU4NC00NmY5LTRhNjAtYjQ4Yi1kOWQxN2Y2OTVlZmQiLCJhY2Nlc3MiOnsidXJuOmZpbGVzdG9yZTpmaWxlOjIyMzU3OGUyLWI3NGMtNDRmYy1hZmY0LTNmOWQ3NGZmMDNhOCI6WyJyZWFkIl19LCJleHAiOjE1Mjg4NTg2MTgsIm5iZiI6MTUyODg1NTU1OH0.yOUqGAdC6MIwpPOXbesABWJJ0gJBhvHePAaBEbmBm20&client=53e54e84-46f9-4a60-b48b-d9d17f695efd&name=raft.pdf&dl=true), and [Viewstamped Replication](http://pmg.csail.mit.edu/papers/vr-revisited.pdf). The most similar academic publication we are aware of to Kafka's actual implementation is [PacificA](https://www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fpubs%2Fdefault.aspx%3Fid%3D66814) from Microsoft.

该系列算法有多种实现，包括ZooKeeper的[Zab](http://web.archive.org/web/20140602093727/http://www.stanford.edu/class/cs347/reading/zab.pdf)，[Raft](https://api.media.atlassian.com/file/223578e2-b74c-44fc-aff4-3f9d74ff03a8/binary?token=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiI1M2U1NGU4NC00NmY5LTRhNjAtYjQ4Yi1kOWQxN2Y2OTVlZmQiLCJhY2Nlc3MiOnsidXJuOmZpbGVzdG9yZTpmaWxlOjIyMzU3OGUyLWI3NGMtNDRmYy1hZmY0LTNmOWQ3NGZmMDNhOCI6WyJyZWFkIl19LCJleHAiOjE1Mjg4NTg2MTgsIm5iZiI6MTUyODg1NTU1OH0.yOUqGAdC6MIwpPOXbesABWJJ0gJBhvHePAaBEbmBm20&client=53e54e84-46f9-4a60-b48b-d9d17f695efd&name=raft.pdf&dl=true)和[Viewstamped Replication](http://pmg.csail.mit.edu/papers/vr-revisited.pdf)。我们知道Kafka实际实现采用的最类似的学术出版物是微软的[PacificA](https://www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fpubs%2Fdefault.aspx%3Fid%3D66814)。

The downside of majority vote is that it doesn't take many failures to leave you with no electable leaders. To tolerate one failure requires three copies of the data, and to tolerate two failures requires five copies of the data. In our experience having only enough redundancy to tolerate a single failure is not enough for a practical system, but doing every write five times, with 5x the disk space requirements and 1/5th the throughput, is not very practical for large volume data problems. This is likely why quorum algorithms more commonly appear for shared cluster configuration such as ZooKeeper but are less common for primary data storage. For example in HDFS the namenode's high-availability feature is built on a [majority-vote-based journal](http://blog.cloudera.com/blog/2012/10/quorum-based-journaling-in-cdh4-1/), but this more expensive approach is not used for the data itself.

多数投票方式的不利之处在于，经过几次失败后会导致没有可选的leader。要容忍一次故障需要三份数据，容忍两次故障需要五份数据。根据我们的经验，仅仅有足够数量的冗余来容忍单个故障对于实际系统来说是不够的，但对于大容量数据问题而言，每次写入操作进行5次（磁盘空间要求是5倍，吞吐量是1/5）并不是很实用。这可能是为什么仲裁算法更常用于共享群集配置（如ZooKeeper），而不常见于主数据存储的原因。例如在HDFS中，namenode的高可用性功能基于[多数投票日志](http://blog.cloudera.com/blog/2012/10/quorum-based-journaling-in-cdh4-1/)，但这种更昂贵的方法不适用于数据本身。

Kafka takes a slightly different approach to choosing its quorum set. Instead of majority vote, Kafka dynamically maintains a set of in-sync replicas (ISR) that are caught-up to the leader. Only members of this set are eligible for election as leader. A write to a Kafka partition is not considered committed until all in-sync replicas have received the write. This ISR set is persisted to ZooKeeper whenever it changes. Because of this, any replica in the ISR is eligible to be elected leader. This is an important factor for Kafka's usage model where there are many partitions and ensuring leadership balance is important. With this ISR model and _f_+1 replicas, a Kafka topic can tolerate _f_ failures without losing committed messages.

Kafka采用稍微不同的方法来选择其仲裁集。Kafka不采用多数投票方式，而是动态地维护一组被引导到leader的同步备份集（ISR）。只有该集合中的成员才有资格当选为leader。写入Kafka分区的消息在所有同步副本都收到该写入操作之前不会被视为已提交。只要ISR发生变化，其就会被持久化到ZooKeeper。正因为如此，ISR中的任何备份都有资格当选为leader。这是Kafka有很多分区模式的重要因素，并且确保这种leader平衡很重要。有了这个ISR模型和 _f_+1 个备份，Kafka主题可以容忍 _f_ 个故障而不会丢失提交的消息。

For most use cases we hope to handle, we think this tradeoff is a reasonable one. In practice, to tolerate _f_ failures, both the majority vote and the ISR approach will wait for the same number of replicas to acknowledge before committing a message (e.g. to survive one failure a majority quorum needs three replicas and one acknowledgement and the ISR approach requires two replicas and one acknowledgement). The ability to commit without the slowest servers is an advantage of the majority vote approach. However, we think it is ameliorated by allowing the client to choose whether they block on the message commit or not, and the additional throughput and disk space due to the lower required replication factor is worth it.

对于我们希望处理的大多数用例，我们认为这种折衷权衡是合理的。在实践中，为了容忍 _f_ 个失败，多数投票方式和ISR方法都会在提交消息之前等待相同数量的副本进行确认（例如，为了在一次失败后生存下来，大多数备份数需要三个副本和一次确认，并且ISR方法需要两个副本和一个确认）。在没有最慢服务器的情况下也能提交是多数投票方法的一个优点。但是，我们认为通过允许客户端选择是否阻止消息提交来改善它，并且由于所需备份因子较低而产生的额外吞吐量和磁盘空间是值得的。

Another important design distinction is that Kafka does not require that crashed nodes recover with all their data intact. It is not uncommon for replication algorithms in this space to depend on the existence of "stable storage" that cannot be lost in any failure-recovery scenario without potential consistency violations. There are two primary problems with this assumption. First, disk errors are the most common problem we observe in real operation of persistent data systems and they often do not leave data intact. Secondly, even if this were not a problem, we do not want to require the use of fsync on every write for our consistency guarantees as this can reduce performance by two to three orders of magnitude. Our protocol for allowing a replica to rejoin the ISR ensures that before rejoining, it must fully re-sync again even if it lost unflushed data in its crash.

另一个重要的设计区别是，Kafka不要求崩溃的节点在恢复后能拥有所有完好无损的数据。在这个领域的复制算法依赖于存在“稳定存储”的情况并不少见，这种“稳定存储”在没有潜在的一致性违反的情况下，在任何故障恢复场景中都不会丢失。这个假设有两个主要问题。首先，磁盘错误是我们在一致性数据系统的实际操作中观察到的最常见的问题，并且它们通常不会使数据保持原样。其次，即使这不是问题，我们也不希望在每次写入时都要求使用fsync来保证一致性，因为这会将性能降低两到三个数量级。我们允许副本重新加入ISR的协议确保了在副本重新加入之前，即使它在崩溃时丢失了未刷新的数据，它也必须再次重新同步。

## Unclean leader election: What if they all die?

## 不清晰的leader选举：如果所有备份都死亡？

Note that Kafka's guarantee with respect to data loss is predicated on at least one replica remaining in sync. If all the nodes replicating a partition die, this guarantee no longer holds.

请注意，Kafka关于数据丢失的保证取决于至少有一个副本保持同步。如果备份分区的所有节点都死亡，则此保证不再成立。

However a practical system needs to do something reasonable when all the replicas die. If you are unlucky enough to have this occur, it is important to consider what will happen. There are two behaviors that could be implemented:

然而，当所有副本死亡时，实际系统需要做一些合理的事情。如果你不幸发生这种情况，重要的是要考虑会发生什么。有两种行为可以实施：

1. Wait for a replica in the ISR to come back to life and choose this replica as the leader (hopefully it still has all its data).
2. Choose the first replica (not necessarily in the ISR) that comes back to life as the leader.


1. 等待ISR中的一个备份恢复并选择这个备份作为leader（希望它仍然拥有其所有数据）。
2. 选择第一个恢复的备份（不一定在ISR中）作为leader。

This is a simple tradeoff between availability and consistency. If we wait for replicas in the ISR, then we will remain unavailable as long as those replicas are down. If such replicas were destroyed or their data was lost, then we are permanently down. If, on the other hand, a non-in-sync replica comes back to life and we allow it to become leader, then its log becomes the source of truth even though it is not guaranteed to have every committed message. By default from version 0.11.0.0, Kafka chooses the first strategy and favor waiting for a consistent replica. This behavior can be changed using configuration property unclean.leader.election.enable, to support use cases where uptime is preferable to consistency.

这是可用性和一致性之间的简单折衷。如果我们在ISR中等待副本，那么只要这些副本是死亡的，我们就一直保持不可用状态。如果这些副本被毁坏或者他们的数据丢失了，那么我们会永久失败。另一方面，如果一个非同步副本恢复了，并且我们允许它成为leader，那么它的日志成为真实的数据来源，即使它不能保证拥有每一个提交的消息。默认情况下，从0.11.0.0版本开始，Kafka选择第一种策略，并倾向于等待一致的副本。此行为可以使用配置属性unclean.leader.election.enable进行更改，以支持正常运行时间比一致性更好的用例。

This dilemma is not specific to Kafka. It exists in any quorum-based scheme. For example in a majority voting scheme, if a majority of servers suffer a permanent failure, then you must either choose to lose 100% of your data or violate consistency by taking what remains on an existing server as your new source of truth.

这种困境并不只是存在于Kafka。它存在于任何基于仲裁(quorum-based)的设计中。例如，在多数投票方式中，如果大多数服务器遭遇永久性故障，那么您必须选择丢失100％的数据，或者通过将现有服务器上剩下的内容作为新的真实数据来源来破坏一致性。

## Availability and Durability Guarantees

## 可用性和持久性保证

When writing to Kafka, producers can choose whether they wait for the message to be acknowledged by 0,1 or all (-1) replicas. Note that "acknowledgement by all replicas" does not guarantee that the full set of assigned replicas have received the message. By default, when acks=all, acknowledgement happens as soon as all the current in-sync replicas have received the message. For example, if a topic is configured with only two replicas and one fails (i.e., only one in sync replica remains), then writes that specify acks=all will succeed. However, these writes could be lost if the remaining replica also fails. Although this ensures maximum availability of the partition, this behavior may be undesirable to some users who prefer durability over availability. Therefore, we provide two topic-level configurations that can be used to prefer message durability over availability:

往Kafka写入数据时，生产者可以选择是否等待消息被0,1或全部（-1）副本确认。请注意，“所有副本确认”并不保证已分配副本能收到该消息。默认情况下，当 acks=all 时，只要所有当前的同步副本收到消息，确认就会发生。例如，如果一个主题配置了只有两个副本并且一个失败（即只有一个同步副本保留），那么指定 acks=all 的写入将会成功执行。但是，如果剩余副本失败，则这些写入操作可能会丢失。虽然这确保了分区的最大可用性，但对于偏好持用性而非可用性的用户而言，这种行为可能是不期望发生的。因此，我们提供了两个主题级配置，可用于优先考虑消息的可用性：

1. Disable unclean leader election - if all replicas become unavailable, then the partition will remain unavailable until the most recent leader becomes available again. This effectively prefers unavailability over the risk of message loss. See the previous section on Unclean Leader Election for clarification.
2. Specify a minimum ISR size - the partition will only accept writes if the size of the ISR is above a certain minimum, in order to prevent the loss of messages that were written to just a single replica, which subsequently becomes unavailable. This setting only takes effect if the producer uses acks=all and guarantees that the message will be acknowledged by at least this many in-sync replicas. This setting offers a trade-off between consistency and availability. A higher setting for minimum ISR size guarantees better consistency since the message is guaranteed to be written to more replicas which reduces the probability that it will be lost. However, it reduces availability since the partition will be unavailable for writes if the number of in-sync replicas drops below the minimum threshold.


1. 禁用不清晰的leader选举 - 如果所有副本都不可用，那么分区将保持不可用状态，直到最近的leader再次可用。这有效地避免了消息丢失的风险。有关说明请参阅上一节“不清晰的leader选举”。
2. 指定ISR的最小值 - 如果ISR的大小超过某个最小值，分区将只接受写入操作，以防止仅写入单个副本的消息丢失，而随后将不可用。此设置仅在生产者使用 acks=all 并且保证该消息至少被许多同步副本确认时才会生效。此设置提供了一致性和可用性之间的折衷。对于ISR的最小值的更高设置可以保证更好的一致性，因为可以保证将消息写入更多副本，从而降低丢失概率。但是，它会降低可用性，因为如果同步副本的数量低于最小阈值，则分区将无法写入。

## Replica Management

## 备份管理

The above discussion on replicated logs really covers only a single log, i.e. one topic partition. However a Kafka cluster will manage hundreds or thousands of these partitions. We attempt to balance partitions within a cluster in a round-robin fashion to avoid clustering all partitions for high-volume topics on a small number of nodes. Likewise we try to balance leadership so that each node is the leader for a proportional share of its partitions.

以上关于备份日志的讨论确实只涉及单个日志，即一个主题分区。然而，Kafka集群将管理数百或数千个这样的分区。我们试图以轮询（round-robin）方式平衡集群内的分区，以避免在少数节点上集中高容量主题的分区。同样，我们试图平衡leader处理，使每个节点都有相同机会成为leader。

It is also important to optimize the leadership election process as that is the critical window of unavailability. A naive implementation of leader election would end up running an election per partition for all partitions a node hosted when that node failed. Instead, we elect one of the brokers as the "controller". This controller detects failures at the broker level and is responsible for changing the leader of all affected partitions in a failed broker. The result is that we are able to batch together many of the required leadership change notifications which makes the election process far cheaper and faster for a large number of partitions. If the controller fails, one of the surviving brokers will become the new controller.

优化leader选举过程也很重要，因为这是不可用的关键窗口。leader选举的执行最终会针对该节点失败时托管的节点的每个分区运行一次选举。相反，我们选择其中一个代理作为“控制者”。该控制器检测代理级别的故障，并负责更改故障代理中所有受影响的分区的负责人。结果是我们可以将许多所需的领导层变更通知批量化，这使得大量分区的选举过程更便宜，更快捷。如果控制器失败，其中一个幸存的代理将成为新的控制者。