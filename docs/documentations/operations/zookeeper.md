# 6.7 ZooKeeper

## Stable version

## 稳定的版本

The current stable branch is 3.4 and the latest release of that branch is 3.4.9.

目前稳定的分支是3.4，该分支的最新版本是3.4.9。

## Operationalizing ZooKeeper

## 操作 ZooKeeper

Operationally, we do the following for a healthy ZooKeeper installation:

在操作上，我们为健康的ZooKeeper安装执行以下操作：

* Redundancy in the physical/hardware/network layout: try not to put them all in the same rack, decent (but don't go nuts) hardware, try to keep redundant power and network paths, etc. A typical ZooKeeper ensemble has 5 or 7 servers, which tolerates 2 and 3 servers down, respectively. If you have a small deployment, then using 3 servers is acceptable, but keep in mind that you'll only be able to tolerate 1 server down in this case.
* I/O segregation: if you do a lot of write type traffic you'll almost definitely want the transaction logs on a dedicated disk group. Writes to the transaction log are synchronous (but batched for performance), and consequently, concurrent writes can significantly affect performance. ZooKeeper snapshots can be one such a source of concurrent writes, and ideally should be written on a disk group separate from the transaction log. Snapshots are written to disk asynchronously, so it is typically ok to share with the operating system and message log files. You can configure a server to use a separate disk group with the dataLogDir parameter.
* Application segregation: Unless you really understand the application patterns of other apps that you want to install on the same box, it can be a good idea to run ZooKeeper in isolation (though this can be a balancing act with the capabilities of the hardware).
* Use care with virtualization: It can work, depending on your cluster layout and read/write patterns and SLAs, but the tiny overheads introduced by the virtualization layer can add up and throw off ZooKeeper, as it can be very time sensitive
* ZooKeeper configuration: It's java, make sure you give it 'enough' heap space (We usually run them with 3-5G, but that's mostly due to the data set size we have here). Unfortunately we don't have a good formula for it, but keep in mind that allowing for more ZooKeeper state means that snapshots can become large, and large snapshots affect recovery time. In fact, if the snapshot becomes too large (a few gigabytes), then you may need to increase the initLimit parameter to give enough time for servers to recover and join the ensemble.
* Monitoring: Both JMX and the 4 letter words (4lw) commands are very useful, they do overlap in some cases (and in those cases we prefer the 4 letter commands, they seem more predictable, or at the very least, they work better with the LI monitoring infrastructure)
* Don't overbuild the cluster: large clusters, especially in a write heavy usage pattern, means a lot of intracluster communication (quorums on the writes and subsequent cluster member updates), but don't underbuild it (and risk swamping the cluster). Having more servers adds to your read capacity.


* 物理/硬件/网络布局中的冗余：尽量不要将它们全部放在同一个机架上，合适的（但不要太过于苛刻）硬件，尽量保持冗余电源和网络路径等。一个典型的ZooKeeper系统有5个或7台服务器，分别容忍2台和3台服务器挂掉。如果你有一个小的部署，那么使用3台服务器是可以接受的，但请记住，在这种情况下你只能容忍1台服务器挂掉。
* I/O隔离：如果您执行大量写入类型的操作，您几乎肯定会将事务日志记录在专用磁盘组上。写入事务日志是同步的（但为了性能而分批），因此并发写入会显著影响性能。ZooKeeper快照可以是并发写入的一个源，最好应该写在与事务日志分开的磁盘组上。快照以异步方式写入磁盘，因此通常可以与操作系统和消息日志文件共享。您可以将服务器配置为使用具有dataLogDir参数的单独磁盘组中。
* 应用程序隔离：除非您真的了解要安装在同一个框中的其它应用的应用程序模式，否则独立运行ZooKeeper可能是一个好主意（尽管这可能是与硬件功能的平衡）。
* 小心使用虚拟化：它可以工作，这具体取决于你的集群布局和读/写模式和SLA，但虚拟层引入的微小开销可以加起来并抛弃ZooKeeper，因为它可能对时间非常敏感。
* ZooKeeper配置：java确保你给它足够的堆空间（我们通常用3-5G运行它们，但这主要是由于我们在这里有数据集的大小）。不幸的是，我们没有一个好的公式，但请记住，允许更多的ZooKeeper状态意味着快照可能变大，并且大型快照会影响恢复时间。事实上，如果快照变得太大（几千兆字节），那么您可能需要增加initLimit参数，以便为服务器提供足够的时间来恢复和加入到整体中。
* 监控：JMX和4个字母的单词（4lw）命令都非常有用，它们在某些情况下会重叠（在这种情况下，我们更喜欢4个字母的命令，它们看起来更可预测，或者至少它们与LI监测基础设施能工作得更好）
* 不要过度构建集群：特别是在写入繁重的使用模式下，大型集群意味着大量的集群内通信（写入和后续集群成员更新时的法定数量），但不会对其进行低级构建（并且可能会淹没集群）。拥有更多的服务器可增加您的读取容量。

Overall, we try to keep the ZooKeeper system as small as will handle the load (plus standard growth capacity planning) and as simple as possible. We try not to do anything fancy with the configuration or application layout as compared to the official release as well as keep it as self contained as possible. For these reasons, we tend to skip the OS packaged versions, since it has a tendency to try to put things in the OS standard hierarchy, which can be 'messy', for want of a better way to word it.

总的来说，我们尽量保持ZooKeeper系统尽可能小，以便处理负载（加上标准增长容量规划）并尽可能简单。与官方发布相比，我们尽量不对配置或应用程序布局进行任何操作，并尽可能将其保留为自包含。由于这些原因，我们倾向于跳过操作系统打包的版本，因为它倾向于尝试将操作系统标准层次结构中的东西放置在“杂乱”的位置，因为我们想要更好的方式来表达它。