# Persistence

# 持久性

## Don't fear the filesystem!

## 不要害怕文件系统！

Kafka relies heavily on the filesystem for storing and caching messages. There is a general perception that "disks are slow" which makes people skeptical that a persistent structure can offer competitive performance. In fact disks are both much slower and much faster than people expect depending on how they are used; and a properly designed disk structure can often be as fast as the network.

Kafka高度依赖文件系统来存储和缓存消息。人们普遍认为“磁盘是缓慢的”，这使得人们对“持久化结构”能提供具有竞争力的性能持有怀疑的态度。事实上，磁盘比人们预想的要快得多，也慢得多，这取决于如何使用它们；一个合理的磁盘结构设计，其速度往往可以和网络速度想媲美。

The key fact about disk performance is that the throughput of hard drives has been diverging from the latency of a disk seek for the last decade. As a result the performance of linear writes on a [JBOD](http://en.wikipedia.org/wiki/Non-RAID_drive_architectures) configuration with six 7200rpm SATA RAID-5 array is about 600MB/sec but the performance of random writes is only about 100k/sec—a difference of over 6000X. These linear reads and writes are the most predictable of all usage patterns, and are heavily optimized by the operating system. A modern operating system provides read-ahead and write-behind techniques that prefetch data in large block multiples and group smaller logical writes into large physical writes. A further discussion of this issue can be found in this [ACM Queue article](http://queue.acm.org/detail.cfm?id=1563874); they actually find that [sequential disk access can in some cases be faster than random memory access!](http://deliveryimages.acm.org/10.1145/1570000/1563874/jacobs3.jpg) 

一个关于磁盘性能的关键事实是：在过去的十年里，磁盘驱动器的吞吐量和磁盘寻道(seek)延迟是相背离的。结果是：在[JBOD](http://en.wikipedia.org/wiki/Non-RAID_drive_architectures)上配置的6个7200rpm SATA RAID-5的磁盘阵列线性写的速度大概是600MB/sec，但是随机写的速度大约只有100k/sec，两者相差超过6000倍。线性读和写在大多数应用场景下是可预测的，且通过操作系统做了大量的优化。 现代操作系统提供了预读和写滞后技术，预先读取多个大块的数据，并将较小的逻辑写组合成一个大的物理写。关于这个问题的更多讨论可以在[ACM Queue article](http://queue.acm.org/detail.cfm?id=1563874)找到。他们发现，在某些情况下，[顺序访问磁盘的速度比随机访问内存的数据要更快!](http://deliveryimages.acm.org/10.1145/1570000/1563874/jacobs3.jpg) 

To compensate for this performance divergence, modern operating systems have become increasingly aggressive in their use of main memory for disk caching. A modern OS will happily divert all free memory to disk caching with little performance penalty when the memory is reclaimed. All disk reads and writes will go through this unified cache. This feature cannot easily be turned off without using direct I/O, so even if a process maintains an in-process cache of the data, this data will likely be duplicated in OS pagecache, effectively storing everything twice.

为了补偿这个性能上的差异，现代操作系统对使用内存做磁盘缓存的方式越来越积极。现代操作系统非常乐意将空闲内存用于磁盘缓存，即使当内存回收的时候会付出一点性能上的代价。所有磁盘的读和写将会通过这个统一的缓存。在没有使用直接I/O的情况下，不能简单的关闭这个功能。所以，即使一个进程维护着一个进程内数据缓存，这些数据仍然会在操作系统页缓存中被复制，从而有效的存储两次。

Furthermore, we are building on top of the JVM, and anyone who has spent any time with Java memory usage knows two things:

此外，由于Kafka是建立在JVM上的，所以任何一个花时间在Java内存使用上的人都应该清楚以下两件事情：

1. The memory overhead of objects is very high, often doubling the size of the data stored (or worse).

2. Java garbage collection becomes increasingly fiddly and slow as the in-heap data increases.


1. 对象的内存开销是非常高的，通常是所存储的数据的两倍（或者更多）。

2. 由于堆内数据增多，Java垃圾回收会变得繁琐和缓慢。

As a result of these factors using the filesystem and relying on pagecache is superior to maintaining an in-memory cache or other structure—we at least double the available cache by having automatic access to all free memory, and likely double again by storing a compact byte structure rather than individual objects. Doing so will result in a cache of up to 28-30GB on a 32GB machine without GC penalties. Furthermore, this cache will stay warm even if the service is restarted, whereas the in-process cache will need to be rebuilt in memory (which for a 10GB cache may take 10 minutes) or else it will need to start with a completely cold cache (which likely means terrible initial performance). This also greatly simplifies the code as all logic for maintaining coherency between the cache and filesystem is now in the OS, which tends to do so more efficiently and more correctly than one-off in-process attempts. If your disk usage favors linear reads then read-ahead is effectively pre-populating this cache with useful data on each disk read.

由于这些因素，使用文件系统并依赖页缓存将优于缓存在内存中或其他结构——Kafka通过自动访问所有空闲内存使得可用缓存至少提升两倍，并且通过存储紧凑的字节结构而不是单个对象使得可用缓存再次提高一倍。这将使得一台32GB内存的机器拥有高达28-30GB的缓存并无需垃圾回收(GC)。此外，即使服务重启，该缓存仍然保持可用，但进程中的缓存将需要在内存中重建（10GB的缓存可能需要10分钟），否则将需要从一个完全冷却的缓存启动（这意味着可怕的初始化性能）。这也大大简化了代码，因为在缓存和文件系统之间维持一致性的逻辑现在都在操作系统中，这比在一次性进程内做尝试更有效也更正确。如果您的磁盘支持线性读，那么预读将有效地将每个磁盘上有用的数据预填充到缓存里。

This suggests a design which is very simple: rather than maintain as much as possible in-memory and flush it all out to the filesystem in a panic when we run out of space, we invert that. All data is immediately written to a persistent log on the filesystem without necessarily flushing to disk. In effect this just means that it is transferred into the kernel's pagecache.

这带来了一个非常简单的设计：当内存空间耗尽时，将数据全部刷新(flush)到文件系统中，而不是尽可能的将数据维持在内存中。我们将其倒置，所有数据立刻写入到文件系统上的持久化日志中而不需要刷新到磁盘中。事实上，这仅仅意味着数据被转移到内核的页缓存中。

This style of pagecache-centric design is described in an [article](http://varnish-cache.org/wiki/ArchitectNotes) on the design of Varnish here (along with a healthy dose of arrogance).

这种以页缓存为中心的设计风格在一篇关于Varnish的设计的[文章](http://varnish-cache.org/wiki/ArchitectNotes)中有描述（伴随着健康的傲慢态度）。

## Constant Time Suffices

## 常数时间就足够了

The persistent data structure used in messaging systems are often a per-consumer queue with an associated BTree or other general-purpose random access data structures to maintain metadata about messages. BTrees are the most versatile data structure available, and make it possible to support a wide variety of transactional and non-transactional semantics in the messaging system. They do come with a fairly high cost, though: Btree operations are O(log N). Normally O(log N) is considered essentially equivalent to constant time, but this is not true for disk operations. Disk seeks come at 10 ms a pop, and each disk can do only one seek at a time so parallelism is limited. Hence even a handful of disk seeks leads to very high overhead. Since storage systems mix very fast cached operations with very slow physical disk operations, the observed performance of tree structures is often superlinear as data increases with fixed cache--i.e. doubling your data makes things much worse than twice as slow.

消息系统中使用的持久化数据结构通常是一个消费者队列，一个Btree或者是其他随机访问数据结构，以维护消息的元数据。Btree是最通用的数据结构，使得在消息系统中支持各种事务的或者非事务的语义成为可能。虽然Btree的操作是O(log N)，但它们的成本相当高。通常O(log N)被认为基本上等同于常数时间，但对于磁盘操作来说是不正确的。磁盘查找（seeks）时间是10ms，且每个磁盘每次只能进行一次查找，所以并行是被限制的。因此，少量的磁盘查找也会带来非常高的开销。由于存储系统将非常快的缓存操作和非常慢的物理磁盘操作结合，所以当数据随着固定缓存增加而增加时，观察到的树型结构的性能往往是超线性(superlinear)的。即，您的数据翻倍将使得效率下降两倍或者更慢。

Intuitively a persistent queue could be built on simple reads and appends to files as is commonly the case with logging solutions. This structure has the advantage that all operations are O(1) and reads do not block writes or each other. This has obvious performance advantages since the performance is completely decoupled from the data size—one server can now take full advantage of a number of cheap, low-rotational speed 1+TB SATA drives. Though they have poor seek performance, these drives have acceptable performance for large reads and writes and come at 1/3 the price and 3x the capacity.

直观上，持久化队列可以构建在简单的读取和追加到文件之上，就和通用的日志解决方案一样。这种结构的有点是所有操作都是O(1)，并且读和写不会相互阻塞。这具有明显的性能优势，因为性能和数据大小完全无关——现在一个服务器可以充分利用一些便宜的，低转速的1+TB SATA驱动器。尽管查找性能很糟糕，但对于大量的读和写来说，这些驱动器的性能是可以接受的，并且仅仅1/3的价格，却得到了3倍的容量。

Having access to virtually unlimited disk space without any performance penalty means that we can provide some features not usually found in a messaging system. For example, in Kafka, instead of attempting to delete messages as soon as they are consumed, we can retain messages for a relatively long period (say a week). This leads to a great deal of flexibility for consumers, as we will describe.

可以在没有任何性能损失的情况下访问几乎无限的磁盘空间，这意味着我们能提供一些其他常见的消息系统没有的特性。例如，在Kafka中，我们能够在相对较长的时间内保留消息（例如一个星期），而不是试图在消息被消费后尽快地删除。正如我们将要描述的那样，这为消费者带来了很大的灵活性。