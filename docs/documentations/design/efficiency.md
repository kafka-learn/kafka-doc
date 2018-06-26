# 4.3 Efficiency

# 4.3 效率

We have put significant effort into efficiency. One of our primary use cases is handling web activity data, which is very high volume: each page view may generate dozens of writes. Furthermore, we assume each message published is read by at least one consumer (often many), hence we strive to make consumption as cheap as possible.

我们已经为效率付出了很大的努力。我们的主要用例之一是处理网络活动数据，该数据体量很大：每个页面视图可能会产生上百个写入操作。此外，我们假设发布的每条消息都被至少一个消费者（通常是很多消费者）读取，因此我们努力使消费尽可能的便宜。

We have also found, from experience building and running a number of similar systems, that efficiency is a key to effective multi-tenant operations. If the downstream infrastructure service can easily become a bottleneck due to a small bump in usage by the application, such small changes will often create problems. By being very fast we help ensure that the application will tip-over under load before the infrastructure. This is particularly important when trying to run a centralized service that supports dozens or hundreds of applications on a centralized cluster as changes in usage patterns are a near-daily occurrence.

我们还发现，从建立和运行一系列类似系统的经验来看，效率是实现multi-tenant(多租户)有效运营的关键。如果下游基础设施服务由于应用程序使用量小而容易成为瓶颈，那么这些小的更改往往会产生问题。通过非常快的速度，我们可以帮助确保在基础架构之前应用程序在负载下崩溃。当尝试运行支持集中式的、成百上千个应用程序的集中式服务时，这一点尤其重要，因为使用模式的变化几乎每天都在发生。

We discussed disk efficiency in the previous section. Once poor disk access patterns have been eliminated, there are two common causes of inefficiency in this type of system: too many small I/O operations, and excessive byte copying.

我们在前一节讨论了磁盘效率。一旦消除了较差的磁盘访问模式，在这种类型的系统中存在两种低效率的常见原因：太多的比较小的I/O操作和过多的字节复制。

The small I/O problem happens both between the client and the server and in the server's own persistent operations.

较小的I/O问题发生在客户端和服务器之间以及服务器自己的持久操作中。

To avoid this, our protocol is built around a "message set" abstraction that naturally groups messages together. This allows network requests to group messages together and amortize the overhead of the network roundtrip rather than sending a single message at a time. The server in turn appends chunks of messages to its log in one go, and the consumer fetches large linear chunks at a time.

为了避免这种情况，我们的协议建立在一个“message set(消息集)”抽象的基础上，自动将消息分组在一起。这允许网络请求将消息分组在一起，并分摊网络往返的开销，而不是一次发送单个消息。服务器依次将消息块附加到其日志中，并且消费者一次获取大型线性块数据。

This simple optimization produces orders of magnitude speed up. Batching leads to larger network packets, larger sequential disk operations, contiguous memory blocks, and so on, all of which allows Kafka to turn a bursty stream of random message writes into linear writes that flow to the consumers.

这个简单的优化产生了数量级的加速。批处理导致更大的网络数据包，更大的顺序磁盘操作，连续的内存块等等，所有这些都允许Kafka将随机消息突发流的写入转换为流向消费者的线性写入。

The other inefficiency is in byte copying. At low message rates this is not an issue, but under load the impact is significant. To avoid this we employ a standardized binary message format that is shared by the producer, the broker, and the consumer (so data chunks can be transferred without modification between them).

另一个低效率是在字节复制中。在低消息速率下，这不是问题，但是在负载情况下，这种影响是显著的。为了避免这种情况，我们使用由生产者，代理和消费者共享的标准的二进制消息格式（因此数据块可以在它们之间进行无需修改的传输）。

The message log maintained by the broker is itself just a directory of files, each populated by a sequence of message sets that have been written to disk in the same format used by the producer and consumer. Maintaining this common format allows optimization of the most important operation: network transfer of persistent log chunks. Modern unix operating systems offer a highly optimized code path for transferring data out of pagecache to a socket; in Linux this is done with the [sendfile system call](http://man7.org/linux/man-pages/man2/sendfile.2.html).

由代理维护的消息日志本身就是一个文件目录，每个文件都由一系列消息集合填充，这些消息集合以生产者和消费者使用的相同格式写入磁盘。保持这种通用格式可以优化最重要的操作：持久日志块的网络传输。现代unix操作系统提供高度优化的代码路径，用于将数据从页面缓存传输到套接字；在Linux系统中，这是通过[sendfile系统调用](http://man7.org/linux/man-pages/man2/sendfile.2.html)完成的。

To understand the impact of sendfile, it is important to understand the common data path for transfer of data from file to socket:

要理解sendfile的影响，需要了理解数据从文件传输到套接字的公共数据路径：

1. The operating system reads data from the disk into pagecache in kernel space
2. The application reads the data from kernel space into a user-space buffer
3. The application writes the data back into kernel space into a socket buffer
4. The operating system copies the data from the socket buffer to the NIC buffer where it is sent over the network


1. 操作系统从磁盘读取数据到内核空间的pagecache中
2. 应用程序从内核空间读取数据到用户空间缓冲区中
3. 应用程序将数据写回内核空间的套接字缓冲区中
4. 操作系统将数据从套接字缓冲区复制到通过网络发送的NIC缓冲区

This is clearly inefficient, there are four copies and two system calls. Using sendfile, this re-copying is avoided by allowing the OS to send the data from pagecache to the network directly. So in this optimized path, only the final copy to the NIC buffer is needed.

这显然是低效的，有四次拷贝操作和两个系统调用。使用sendfile，通过允许操作系统将数据从页面缓存直接发送到网络，可以避免重新拷贝。所以在这个优化的路径中，只需要最终拷贝到NIC缓冲区。

We expect a common use case to be multiple consumers on a topic. Using the zero-copy optimization above, data is copied into pagecache exactly once and reused on each consumption instead of being stored in memory and copied out to user-space every time it is read. This allows messages to be consumed at a rate that approaches the limit of the network connection.

我们期望一个常见的用例在一个主题上成为多个消费者。使用上面的零拷贝优化，数据被复制到pagecache中一次，并在每次使用时重用，而不是存储在内存中，并且每次读取时都将其复制到用户空间。这允许消息以接近网络连接限制的速率消耗。

This combination of pagecache and sendfile means that on a Kafka cluster where the consumers are mostly caught up you will see no read activity on the disks whatsoever as they will be serving data entirely from cache.

这种页面缓存和发送文件的结合意味着，在消费者大多被捕获的Kafka集群中，您将看不到磁盘上的读取活动，因为它们将完全从缓存中提供数据。

For more background on the sendfile and zero-copy support in Java, see this [article](https://www.ibm.com/developerworks/linux/library/j-zerocopy/).

有关sendfile和Java中零拷贝支持的更多背景信息，请参阅此篇[文章](https://www.ibm.com/developerworks/linux/library/j-zerocopy/)。

## End-to-end Batch Compression

## 端到端的批量压缩

In some cases the bottleneck is actually not CPU or disk but network bandwidth. This is particularly true for a data pipeline that needs to send messages between data centers over a wide-area network. Of course, the user can always compress its messages one at a time without any support needed from Kafka, but this can lead to very poor compression ratios as much of the redundancy is due to repetition between messages of the same type (e.g. field names in JSON or user agents in web logs or common string values). Efficient compression requires compressing multiple messages together rather than compressing each message individually.

在一些场景中，如在数据中心之间进行数据复制，系统的瓶颈往往不是CPU或者磁盘，而是网络带宽的限制，尤其是在需要通过广域网进行数据管道流传输的时候变得更加明显。当然，用户可以一次压缩一条消息，而不需要Kafka提供任何支持，但是这会导致压缩比非常低，因为冗余的多少是由于相同类型消息之间的重复（例如，JSON格式的字段名称或Web日志中的用户代理或公共字符串值）。有效的压缩需要将多个消息压缩在一起，而不是单独压缩每个消息。

Kafka supports this with an efficient batching format. A batch of messages can be clumped together compressed and sent to the server in this form. This batch of messages will be written in compressed form and will remain compressed in the log and will only be decompressed by the consumer.

Kakfa以高效的批处理格式支持这一点。一批消息可以压缩在一起并以这种形式发送到服务器。这批消息将以压缩格式写入，并将保持压缩在日志中，并仅由消费者进行解压。

Kafka supports GZIP, Snappy and LZ4 compression protocols. More details on compression can be found [here](https://cwiki.apache.org/confluence/display/KAFKA/Compression).

Kafka支持GZIP，Snappy和LZ4压缩协议。有关压缩的更多细节可以在[这里](https://cwiki.apache.org/confluence/display/KAFKA/Compression)找到。