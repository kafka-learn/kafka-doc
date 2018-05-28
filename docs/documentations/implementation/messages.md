# 5.2 Messages

# 5.2 消息

Messages consist of a variable-length header, a variable length opaque key byte array and a variable length opaque value byte array. The format of the header is described in the following section. Leaving the key and value opaque is the right decision: there is a great deal of progress being made on serialization libraries right now, and any particular choice is unlikely to be right for all uses. Needless to say a particular application using Kafka would likely mandate a particular serialization type as part of its usage. The ```RecordBatch``` interface is simply an iterator over messages with specialized methods for bulk reading and writing to an NIO ```Channel```.

消息由可变长度头部，可变长度且不确定的key字节数组和可变长度且不确定的value字节数组组成。头部格式将在下一节中介绍。让key和value不确定是一个正确的决定：现在在序列化库上取得了很大进展，任何特定的选择都不太可能适合所有用途。更不用说使用Kafka的特定应用程序可能会强制使用特定的序列化类型作为其使用的一部分。```RecordBatch```接口仅仅是一个消息迭代器，它实现了用于批量读取和写入NIO ```Channel```(通道)的专用方法。