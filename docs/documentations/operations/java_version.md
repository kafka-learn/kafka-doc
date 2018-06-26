# 6.4 Java Version

# 6.4 Java 版本

From a security perspective, we recommend you use the latest released version of JDK 1.8 as older freely available versions have disclosed security vulnerabilities. LinkedIn is currently running JDK 1.8 u5 (looking to upgrade to a newer version) with the G1 collector. If you decide to use the G1 collector (the current default) and you are still on JDK 1.7, make sure you are on u51 or newer. LinkedIn tried out u21 in testing, but they had a number of problems with the GC implementation in that version. LinkedIn's tuning looks like this:

从安全角度来看，我们建议您使用JDK 1.8的最新发布版本，因为较早的免费版本已经披露了安全漏洞。LinkedIn目前正在使用G1收集器运行JDK 1.8 u5（希望升级到更新版本）。如果您决定使用G1收集器（当前的默认值），并且您仍然使用JDK 1.7，请确保您使用的是u51或更新版本。LinkedIn在测试中试用了u21，但是在该版本中，GC实现方面存在一些问题。LinkedIn的调整看起来像这样：

```bash
-Xmx6g -Xms6g -XX:MetaspaceSize=96m -XX:+UseG1GC
-XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M
-XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80
```

For reference, here are the stats on one of LinkedIn's busiest clusters (at peak):

作为参考，如下是LinkedIn最繁忙的集群中的某个代理（在峰值时）的统计数据：

* 60 brokers
* 50k partitions (replication factor 2)
* 800k messages/sec in
* 300 MB/sec inbound, 1 GB/sec+ outbound

The tuning looks fairly aggressive, but all of the brokers in that cluster have a 90% GC pause time of about 21ms, and they're doing less than 1 young GC per second.

该调整看起来相当积极，但该集群中的所有代理都有大约有90%的GC暂停时间，大约是21ms，而且它们每秒钟的年轻代GC不到1个。