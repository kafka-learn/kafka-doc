# 6.5 Hardware and OS

# 6.5 硬件和操作系统

We are using dual quad-core Intel Xeon machines with 24GB of memory.

我们正在使用具有24GB内存的四核英特尔Xeon处理器。

You need sufficient memory to buffer active readers and writers. You can do a back-of-the-envelope estimate of memory needs by assuming you want to be able to buffer for 30 seconds and compute your memory need as write_throughput*30.

您需要足够的内存来缓存活动的读写。您可以通过假设您希望能够缓冲30秒并计算您的内存需求为write_throughput*30来进行内存需求的后期估计。

The disk throughput is important. We have 8x7200 rpm SATA drives. In general disk throughput is the performance bottleneck, and more disks is better. Depending on how you configure flush behavior you may or may not benefit from more expensive disks (if you force flush often then higher RPM SAS drives may be better).

磁盘吞吐量很重要。我们有8x7200转的SATA硬盘。一般来说，磁盘吞吐量是性能瓶颈，更多的磁盘会更好。根据配置刷新行为的方式，您可能会也可能不会从更昂贵的磁盘中受益（如果您经常强制刷新，那么更高RPM的SAS驱动器可能会更好）。

## OS

## 操作系统

Kafka should run well on any unix system and has been tested on Linux and Solaris.

Kafka应该在任何Unix系统上运行良好，并且已经在Linux和Solaris上进行了测试。

We have seen a few issues running on Windows and Windows is not currently a well supported platform though we would be happy to change that.

我们已经看到在Windows上运行的一些问题，并且目前在Windows上还不能很好的支持，尽管我们很乐意改变这一点。

It is unlikely to require much OS-level tuning, but there are two potentially important OS-level configurations:

这不太需要很多操作系统级的调整，但是有两个潜在的重要的操作系统级配置：

* File descriptor limits: Kafka uses file descriptors for log segments and open connections. If a broker hosts many partitions, consider that the broker needs at least (number_of_partitions)*(partition_size/segment_size) to track all log segments in addition to the number of connections the broker makes. We recommend at least 100000 allowed file descriptors for the broker processes as a starting point.
* Max socket buffer size: can be increased to enable high-performance data transfer between data centers as [described here](https://www.psc.edu/index.php/networking/641-tcp-tune).


* 文件描述符限制：Kafka为日志段和打开的连接使用文件描述符。如果代理承载多个分区，请考虑代理至少需要（number_of_partitions）*（partition_size / segment_size）来跟踪代理所做的连接数量以外的所有日志段。 我们推荐至少100000个允许的代理进程文件描述符作为起点。
* 最大套接字缓冲区大小：可以增大以实现数据中心之间的高性能数据传输，如[此处所述](https://www.psc.edu/index.php/networking/641-tcp-tune)。

## Disks and Filesystem

## 磁盘和文件系统

We recommend using multiple drives to get good throughput and not sharing the same drives used for Kafka data with application logs or other OS filesystem activity to ensure good latency. You can either RAID these drives together into a single volume or format and mount each drive as its own directory. Since Kafka has replication the redundancy provided by RAID can also be provided at the application level. This choice has several tradeoffs.

我们建议使用多个驱动器以获得良好的吞吐量，而不是与应用程序日志或其它操作系统文件系统活动共享用于Kafka数据的相同驱动器，以确保良好的延迟。您可以将这些驱动器一起RAID成单个卷或格式，并将每个驱动器安装为自己的目录。由于Kafka具有复制功能，RAID提供的冗余也可以在应用程序级别提供。这个选择有几个折衷。

If you configure multiple data directories partitions will be assigned round-robin to data directories. Each partition will be entirely in one of the data directories. If data is not well balanced among partitions this can lead to load imbalance between disks.

如果您配置多个数据目录，则会将分区循环分配到数据目录。每个分区将完全位于其中一个数据目录中。如果分区间的数据不均衡，则可能导致磁盘之间的负载不均衡。

RAID can potentially do better at balancing load between disks (although it doesn't always seem to) because it balances load at a lower level. The primary downside of RAID is that it is usually a big performance hit for write throughput and reduces the available disk space.

RAID可以更好地平衡磁盘之间的负载（尽管看起来并不总是如此），因为它可以在较低的级别上平衡负载。RAID的主要缺点是写入吞吐量通常会造成很大的性能下降，并且会减少可用的磁盘空间。

Another potential benefit of RAID is the ability to tolerate disk failures. However our experience has been that rebuilding the RAID array is so I/O intensive that it effectively disables the server, so this does not provide much real availability improvement.

RAID的另一个潜在好处是可以容忍磁盘故障。但是我们的经验是，重建RAID阵列的I/O密集程度非常高，因此它可以有效地禁用服务器，所以这不会提供实际的可用性改进。

## Application vs. OS Flush Management

## 应用程序 vs. 操作系统的flush管理

Kafka always immediately writes all data to the filesystem and supports the ability to configure the flush policy that controls when data is forced out of the OS cache and onto disk using the flush. This flush policy can be controlled to force data to disk after a period of time or after a certain number of messages has been written. There are several choices in this configuration.

Kafka总是立即将所有数据写入文件系统，并支持配置刷新策略的功能，该策略控制何时使用刷新将数据从OS缓存中移出到磁盘上。可以控制该刷新策略以在一段时间之后或写入一定数量的消息之后强制数据到磁盘。这种配置有几种选择。

Kafka must eventually call fsync to know that data was flushed. When recovering from a crash for any log segment not known to be fsync'd Kafka will check the integrity of each message by checking its CRC and also rebuild the accompanying offset index file as part of the recovery process executed on startup.

Kafka最终必须调用fsync才能知道数据已被刷新。从任何未知的fsync'd日志段的崩溃中恢复时，Kafka将通过检查每个消息的CRC来检查其完整性，并在启动时执行的恢复过程中重建随附的偏移量索引文件。

Note that durability in Kafka does not require syncing data to disk, as a failed node will always recover from its replicas.

请注意，Kafka中的持久性不需要将数据同步到磁盘，因为失败的节点将始终从其副本中恢复。

We recommend using the default flush settings which disable application fsync entirely. This means relying on the background flush done by the OS and Kafka's own background flush. This provides the best of all worlds for most uses: no knobs to tune, great throughput and latency, and full recovery guarantees. We generally feel that the guarantees provided by replication are stronger than sync to local disk, however the paranoid still may prefer having both and application level fsync policies are still supported.

我们建议使用完全禁用应用程序fsync的默认刷新设置。这意味着依靠操作系统和Kafka自己的后台刷新完成的背景刷新。这为大多数用途提供了最好的环境：无需调节旋钮，极大的吞吐量和延迟以及完全恢复保证。我们一般认为复制提供的保证比同步到本地磁盘更强，但偏执狂仍然可能更喜欢同时支持fsync和应用程序级别的策略。

The drawback of using application level flush settings is that it is less efficient in its disk usage pattern (it gives the OS less leeway to re-order writes) and it can introduce latency as fsync in most Linux filesystems blocks writes to the file whereas the background flushing does much more granular page-level locking.

使用应用程序级别刷新设置的缺点是它的磁盘使用模式效率较低（它使操作系统在重新排序时没有什么回旋余地），并且它可能会引入延迟，因为大多数Linux文件系统块中的fsync会写入文件，背景刷新可以实现更细致的页面级锁定。

In general you don't need to do any low-level tuning of the filesystem, but in the next few sections we will go over some of this in case it is useful.

一般而言，您不需要对文件系统进行任何低级调整，但在接下来的几节中，我们会在其中介绍其中的一些内容以防万一它有用。

## Understanding Linux OS Flush Behavior

## 理解Linux操作系统的flush行为

In Linux, data written to the filesystem is maintained in [pagecache](https://en.wikipedia.org/wiki/Page_cache) until it must be written out to disk (due to an application-level fsync or the OS's own flush policy). The flushing of data is done by a set of background threads called pdflush (or in post 2.6.32 kernels "flusher threads").

在Linux中，写入文件系统的数据在[pagecache](https://en.wikipedia.org/wiki/Page_cache)中保存，直到它必须写入磁盘（由于应用程序级别的fsync或操作系统自己的flush 政策）。 数据的刷新是通过一组称为pdflush的后台线程（或后2.6.32内核中的“刷新线程”）完成的。

Pdflush has a configurable policy that controls how much dirty data can be maintained in cache and for how long before it must be written back to disk. This policy is described [here](http://web.archive.org/web/20160518040713/http://www.westnet.com/~gsmith/content/linux-pdflush.htm). When Pdflush cannot keep up with the rate of data being written it will eventually cause the writing process to block incurring latency in the writes to slow down the accumulation of data.

Pdflush有一个可配置的策略，用于控制在缓存中可以维护多少脏数据以及在必须将数据写回到磁盘之前多长时间。 这项政策的描述[这里](http://web.archive.org/web/20160518040713/http://www.westnet.com/~gsmith/content/linux-pdflush.htm)。当Pdflush无法跟上正在写入的数据速率时，它最终会导致写入过程阻止写入中的等待时间，从而减慢数据的积累。

You can see the current state of OS memory usage by doing

您可以通过执行如下命令来查看操作系统内存使用情况的当前状态

```bash
> cat /proc/meminfo
```

The meaning of these values are described in the link above.

这些值的含义在上面的链接中有描述。

Using pagecache has several advantages over an in-process cache for storing data that will be written out to disk:

使用pagecache与存储进入磁盘的数据存储的进程内缓存相比有几个优点：

* The I/O scheduler will batch together consecutive small writes into bigger physical writes which improves throughput.
* The I/O scheduler will attempt to re-sequence writes to minimize movement of the disk head which improves throughput.
* It automatically uses all the free memory on the machine


* I/O调度程序会将连续的小写入批量转换为更大的物理写入，从而提高吞吐量。
* I/O调度程序将尝试重新排序写入，以尽量减少磁盘磁头的移动，从而提高吞吐量。
* 它会自动使用机器上的所有可用内存

## Filesystem Selection

## 文件系统的选择

Kafka uses regular files on disk, and as such it has no hard dependency on a specific filesystem. The two filesystems which have the most usage, however, are EXT4 and XFS. Historically, EXT4 has had more usage, but recent improvements to the XFS filesystem have shown it to have better performance characteristics for Kafka's workload with no compromise in stability.

Kafka在磁盘上使用常规文件，因此它不依赖于特定的文件系统。然而，使用最多的两个文件系统是EXT4和XFS。从历史上看，EXT4有更多的用途，但最近对XFS文件系统的改进表明它具有更好的卡夫卡工作负载性能特性，而且不会影响稳定性。

Comparison testing was performed on a cluster with significant message loads, using a variety of filesystem creation and mount options. The primary metric in Kafka that was monitored was the "Request Local Time", indicating the amount of time append operations were taking. XFS resulted in much better local times (160ms vs. 250ms+ for the best EXT4 configuration), as well as lower average wait times. The XFS performance also showed less variability in disk performance.

使用各种文件系统创建和安装选项，在具有重要消息加载的群集上执行比较测试。Kafka中监控的主要指标是“请求本地时间”，表示所附加操作的时间量。XFS导致当地时间好得多（160ms比250ms +最好的EXT4配置），以及更低的平均等待时间。 XFS性能也表现出较小的磁盘性能变化。

### General Filesystem Notes

### 一般的文件系统节点

For any filesystem used for data directories, on Linux systems, the following options are recommended to be used at mount time:

对于用于数据目录的任何文件系统，在Linux系统上，建议在安装时使用以下选项：

* noatime: This option disables updating of a file's atime (last access time) attribute when the file is read. This can eliminate a significant number of filesystem writes, especially in the case of bootstrapping consumers. Kafka does not rely on the atime attributes at all, so it is safe to disable this.


* noatime：该选项禁止在读取文件时更新文件的atime（最后访问时间）属性。这可以消除大量的文件系统写入，特别是在引导用户的情况下。Kafka根本不依赖atime属性，因此禁用这个属性是安全的。

### XFS Notes

### XFS 节点

The XFS filesystem has a significant amount of auto-tuning in place, so it does not require any change in the default settings, either at filesystem creation time or at mount. The only tuning parameters worth considering are:

XFS文件系统具有大量的自动调整功能，因此无需在文件系统创建时或挂载时对默认设置进行任何更改。唯一值得考虑的调整参数是：

* largeio: This affects the preferred I/O size reported by the stat call. While this can allow for higher performance on larger disk writes, in practice it had minimal or no effect on performance.
* nobarrier: For underlying devices that have battery-backed cache, this option can provide a little more performance by disabling periodic write flushes. However, if the underlying device is well-behaved, it will report to the filesystem that it does not require flushes, and this option will have no effect.


* largeio：这会影响统计调用报告的首选I/O大小。虽然这可以在更大的磁盘写入时实现更高的性能，但实际上它对性能的影响很小或没有影响。
* nobarrier：对于具有电池支持缓存的底层设备，此选项可通过禁用定期写入刷新来提供更高的性能。但是，如果底层设备运行良好，则会向文件系统报告不需要刷新，此选项不起作用。

### EXT4 Notes

### EXT4 节点

EXT4 is a serviceable choice of filesystem for the Kafka data directories, however getting the most performance out of it will require adjusting several mount options. In addition, these options are generally unsafe in a failure scenario, and will result in much more data loss and corruption. For a single broker failure, this is not much of a concern as the disk can be wiped and the replicas rebuilt from the cluster. In a multiple-failure scenario, such as a power outage, this can mean underlying filesystem (and therefore data) corruption that is not easily recoverable. The following options can be adjusted:

EXT4是Kafka数据目录文件系统的一个可用选择，但要获得最高的性能，但需要调整多个安装选项。另外，这些选项在故障情况下通常是不安全的，并且会导致更多的数据丢失和 损坏。对于单个代理故障，这不是什么大问题，因为可以擦除磁盘并从集群重建副本。在诸如停电等多故障情况下，这可能意味着不容易恢复的底层文件系统（因此是数据）。以下选项可以调整：

* data=writeback: Ext4 defaults to data=ordered which puts a strong order on some writes. Kafka does not require this ordering as it does very paranoid data recovery on all unflushed log. This setting removes the ordering constraint and seems to significantly reduce latency.
* Disabling journaling: Journaling is a tradeoff: it makes reboots faster after server crashes but it introduces a great deal of additional locking which adds variance to write performance. Those who don't care about reboot time and want to reduce a major source of write latency spikes can turn off journaling entirely.
* commit=num_secs: This tunes the frequency with which ext4 commits to its metadata journal. Setting this to a lower value reduces the loss of unflushed data during a crash. Setting this to a higher value will improve throughput.
* nobh: This setting controls additional ordering guarantees when using data=writeback mode. This should be safe with Kafka as we do not depend on write ordering and improves throughput and latency.
* delalloc: Delayed allocation means that the filesystem avoid allocating any blocks until the physical write occurs. This allows ext4 to allocate a large extent instead of smaller pages and helps ensure the data is written sequentially. This feature is great for throughput. It does seem to involve some locking in the filesystem which adds a bit of latency variance.


* data=writeback：Ext4默认为data=ordered，这会在某些写入操作上产生强大的顺序。Kafka不需要这样的排序，因为它对所有未刷新的日志进行非常偏执的数据恢复。此设置消除了排序约束，似乎显着减少了延迟。
* 禁用日志功能：日志功能是一种折衷：在服务器崩溃后，重新启动会更快，但会引入大量额外的锁定，从而增加写入性能的差异。那些不关心重启时间并希望减少写入延迟尖峰的主要来源的人可以完全关闭日志记录。
* commit=num_secs：调整ext4向其元数据日志提交的频率。将其设置为较低的值可减少崩溃期间未刷新数据的丢失。将其设置为更高的值将提高吞吐量。
* nobh：当使用data=回写模式时，此设置控制额外的订购保证。这对Kafka应该是安全的，因为我们不依赖写入顺序并提高吞吐量和延迟。
* delalloc：延迟分配意味着文件系统避免分配任何数据块，直到发生物理写入。这允许ext4分配很大的范围而不是较小的页面，并有助于确保数据顺序写入。此功能对于吞吐量非常有用。它似乎涉及到文件系统中的一些锁定，这会增加一些延迟差异。