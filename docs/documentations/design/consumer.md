# 4.5 The Consumer

# 4.5 消费者

The Kafka consumer works by issuing "fetch" requests to the brokers leading the partitions it wants to consume. The consumer specifies its offset in the log with each request and receives back a chunk of log beginning from that position. The consumer thus has significant control over this position and can rewind it to re-consume data if need be.

kafka消费者通过向希望消费的分区的代理发出"fetch(获取)"请求进行工作。消费者在每个请求的日志中指定其偏移量，并从该位置接收一段反馈日志。因此，消费者对这个偏移位置具有重要的控制权，并且可以重新指向它以在需要时重新消费数据。

## Push vs. pull

## 推送和拉取

An initial question we considered is whether consumers should pull data from brokers or brokers should push data to the consumer. In this respect Kafka follows a more traditional design, shared by most messaging systems, where data is pushed to the broker from the producer and pulled from the broker by the consumer. Some logging-centric systems, such as Scribe and Apache Flume, follow a very different push-based path where data is pushed downstream. There are pros and cons to both approaches. However, a push-based system has difficulty dealing with diverse consumers as the broker controls the rate at which data is transferred. The goal is generally for the consumer to be able to consume at the maximum possible rate; unfortunately, in a push system this means the consumer tends to be overwhelmed when its rate of consumption falls below the rate of production (a denial of service attack, in essence). A pull-based system has the nicer property that the consumer simply falls behind and catches up when it can. This can be mitigated with some kind of backoff protocol by which the consumer can indicate it is overwhelmed, but getting the rate of transfer to fully utilize (but never over-utilize) the consumer is trickier than it seems. Previous attempts at building systems in this fashion led us to go with a more traditional pull model.

最初我们考虑的问题是消费者是否应该从代理拉取数据，或代理是否应该将数据推送给消费者。在这个方面，Kafka采用大多数消息系统常用的更传统的设计，数据从生产者推送给代理，并由消费者从代理中拉取。一些以日志为中心的系统（如Scribe和Apache Flume）采用了一种非常不同的基于推送的方法，即将数据推送到下游。这两种方法都有优缺点。然而，基于推送的系统难以处理不同的消费者，因为代理控制着数据传输的速度。实际目标通常是让消费者能够以最大可能的速度消费；不幸的是，在推送系统中，这意味着当消费者的消费率低于生产速率（本质上是拒绝服务攻击）时，消费者往往会不知所措。而在基于拉取的系统中，消费者只需在落后的时候去赶上。这也可以通过某种退让协议（backoff protocol）来处理，消费者可以通过这种退让协议来表明它已经不堪重负，但获取传输速率以充分利用（但从未过度使用）消费者的实际操作要复杂。先前以这种方式构建系统的一些尝试使得我们采用了更传统的拉取模型。

Another advantage of a pull-based system is that it lends itself to aggressive batching of data sent to the consumer. A push-based system must choose to either send a request immediately or accumulate more data and then send it later without knowledge of whether the downstream consumer will be able to immediately process it. If tuned for low latency, this will result in sending a single message at a time only for the transfer to end up being buffered anyway, which is wasteful. A pull-based design fixes this as the consumer always pulls all available messages after its current position in the log (or up to some configurable max size). So one gets optimal batching without introducing unnecessary latency.

基于拉取的系统的另一个优点是它可以对发送给消费者的数据进行大批量处理。基于推送的系统必须选择立即发送请求或累积更多数据后处理，然后在不知道下游消费者是否能够立即处理它的情况下发送。如果是低延迟，则这将导致一次只发送一条消息，以便传输最终被缓冲，但这是浪费的。基于拉取的设计修复了这种情况，因为消费者总是将所有可用消息拖到日志中的当前位置（或达到某个可配置的最大大小）之后。所以用户可以获得最佳的批量数据而不会引入不必要的延迟。

The deficiency of a naive pull-based system is that if the broker has no data the consumer may end up polling in a tight loop, effectively busy-waiting for data to arrive. To avoid this we have parameters in our pull request that allow the consumer request to block in a "long poll" waiting until data arrives (and optionally waiting until a given number of bytes is available to ensure large transfer sizes).

一个基于拉取的系统的缺陷是，如果代理没有数据，消费者可能会以死循环的方式结束轮询，实际上是忙于等待数据到达。为了避免这种情况，我们在我们的pull请求中设有参数，它允许消费者请求以“长轮询”方式阻塞，等待数据的到达（并且有可选方式的等待，即等待给定数量的字节以确保教大的传输大小）。

You could imagine other possible designs which would be only pull, end-to-end. The producer would locally write to a local log, and brokers would pull from that with consumers pulling from them. A similar type of "store-and-forward" producer is often proposed. This is intriguing but we felt not very suitable for our target use cases which have thousands of producers. Our experience running persistent data systems at scale led us to feel that involving thousands of disks in the system across many applications would not actually make things more reliable and would be a nightmare to operate. And in practice we have found that we can run a pipeline with strong SLAs at large scale without a need for producer persistence.

你可以想象其它可能的设计，这有仅拉取(pull)，端到端(end-to-end)的方式。生产者会向本地日志中写入本地消息，而代理会从消费者那里拉取。一种类似“存储和转发”生产者设计经常被提及。这很有趣，但我们觉得这不太适合我们，因为我们有数千个生产者用例。依据大规模运行持久数据系统的经验，实际上，跨越许多应用程序在系统中涉及数千个磁盘不会使事情变得更加可靠，反而会成为操作的噩梦。实际上，我们发现可以在不需要生产者持续性的情况下大规模地运行带有强大SLA的管道。

## Consumer Position

## 消费位置

Keeping track of what has been consumed is, surprisingly, one of the key performance points of a messaging system.

令人惊讶的是，跟踪已被消费的是消息传递系统的关键性能点之一。

Most messaging systems keep metadata about what messages have been consumed on the broker. That is, as a message is handed out to a consumer, the broker either records that fact locally immediately or it may wait for acknowledgement from the consumer. This is a fairly intuitive choice, and indeed for a single machine server it is not clear where else this state could go. Since the data structures used for storage in many messaging systems scale poorly, this is also a pragmatic choice--since the broker knows what is consumed it can immediately delete it, keeping the data size small.

大多数消息传递系统都保留关于代理上消费的消息的元数据。也就是说，当消息发送给消费者时，代理要么立即在本地记录，要么等待消费者的确认。这是一个相当直观的选择，实际上对于单台服务器来说，并不清楚这种状态可以发生在哪里。由于许多消息传递系统中用于存储的数据结构规模较小，因此这也是一个实用的选择 - 由于代理知道消耗的是什么，它可以立即将其删除，从而保持较小的数据量。

What is perhaps not obvious is that getting the broker and consumer to come into agreement about what has been consumed is not a trivial problem. If the broker records a message as consumed immediately every time it is handed out over the network, then if the consumer fails to process the message (say because it crashes or the request times out or whatever) that message will be lost. To solve this problem, many messaging systems add an acknowledgement feature which means that messages are only marked as sent not consumed when they are sent; the broker waits for a specific acknowledgement from the consumer to record the message as consumed. This strategy fixes the problem of losing messages, but creates new problems. First of all, if the consumer processes the message but fails before it can send an acknowledgement then the message will be consumed twice. The second problem is around performance, now the broker must keep multiple states about every single message (first to lock it so it is not given out a second time, and then to mark it as permanently consumed so that it can be removed). Tricky problems must be dealt with, like what to do with messages that are sent but never acknowledged.

不明显的问题可能是，让代理和消费者就已经消费的东西达成一致并不是一个小问题。如果代理将消息记录为在每次通过网络发送时立即使用，那么如果消费者未能处理该消息（比如因为崩溃或请求超时等等），则该消息将丢失。为了解决这个问题，许多消息传递系统增加了一个确认功能，这意味着消息在被发送时只被标记为发送而不被标记为已消费；代理等待消费者的特殊确认后将消息记录标记为已消费。这个策略解决了丢失信息的问题，但却产生了新的问题。首先，如果消费者在发送确认之前处理消息失败，则该消息将被消费两次。第二个是关于性能的问题，代理现在必须保持每个消息的多个状态（首先锁定它，以便它不会再次发出，然后将其标记为被消费了以便将其删除）。一些棘手的问题必须得到处理，比如如何处理发送但未被确认的消息。

Kafka handles this differently. Our topic is divided into a set of totally ordered partitions, each of which is consumed by exactly one consumer within each subscribing consumer group at any given time. This means that the position of a consumer in each partition is just a single integer, the offset of the next message to consume. This makes the state about what has been consumed very small, just one number for each partition. This state can be periodically checkpointed. This makes the equivalent of message acknowledgements very cheap.

Kafka处理这个有些不同。我们的主题被分成一组完全有序的分区，每个分区在任何给定的时间都由每个订阅消费者组中的一个消费者使用。这意味着消费者在每个分区中的位置只是一个整数，即要消费的下一个消息的偏移量。这使得状态变得非常小，每个分区只有一个数字。这个状态可以定期检查。这使得消息的确认非常方便。

There is a side benefit of this decision. A consumer can deliberately rewind back to an old offset and re-consume data. This violates the common contract of a queue, but turns out to be an essential feature for many consumers. For example, if the consumer code has a bug and is discovered after some messages are consumed, the consumer can re-consume those messages once the bug is fixed.

这种设计有一个好处。消费者可以退回到旧的偏移量并重新使用数据。这违反了队列的共同使用模式，但是对于许多消费者来说这是一个重要特征。例如，如果消费者代码有一个错误，并且该错误是在部分消息被消费后才被发现，那么消费者可以在错误修复后重新使用这些消息。

## Offline Data Load

## 离线数据加载

Scalable persistence allows for the possibility of consumers that only periodically consume such as batch data loads that periodically bulk-load data into an offline system such as Hadoop or a relational data warehouse.

可伸缩持久性允许消费者仅定期的进行数据消费，例如批量加载的数据，定期将数据批量加载到离线系统（如Hadoop或关系数据仓库）中。

In the case of Hadoop we parallelize the data load by splitting the load over individual map tasks, one for each node/topic/partition combination, allowing full parallelism in the loading. Hadoop provides the task management, and tasks which fail can restart without danger of duplicate data—they simply restart from their original position.

对于Hadoop的情况，我们通过在单个映射任务上分割负载来并行化数据负载，每个节点/主题/分区组合都有一个负载，从而在负载中实现完全并行化。Hadoop提供任务管理，失败的任务可以重新启动而没有重复数据的危险 - 它们只需从原始位置重新启动即可。