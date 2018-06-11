# 5.3 Message Format

# 5.3 消息格式

Messages (aka Records) are always written in batches. The technical term for a batch of messages is a record batch, and a record batch contains one or more records. In the degenerate case, we could have a record batch containing a single record. Record batches and records have their own headers. The format of each is described below for Kafka version 0.11.0 and later (message format version v2, or magic=2). [Click here](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol#AGuideToTheKafkaProtocol-Messagesets) for details about message formats 0 and 1.

消息（即Record）总是批量写入的。一批消息的技术术语是记录批次（record batch），一个record batch包含一个或多个record。在退化情况下，我们可以拥有包含单个record的record batch。record batch和record都有自己的头部。在Kafka 0.11.0版本及更高版本（消息格式版本v2，或魔数=2）中描述了每种格式。[点击此处](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol#AGuideToTheKafkaProtocol-Messagesets)了解有关消息格式0和1的详细信息。

## 5.3.1 Record Batch

The following is the on-disk format of a RecordBatch.

以下是RecordBatch的磁盘格式。

```
baseOffset: int64
batchLength: int32
partitionLeaderEpoch: int32
magic: int8 (current magic value is 2)
crc: int32
attributes: int16
    bit 0~2:
        0: no compression
        1: gzip
        2: snappy
        3: lz4
    bit 3: timestampType
    bit 4: isTransactional (0 means not transactional)
    bit 5: isControlBatch (0 means not a control batch)
    bit 6~15: unused
lastOffsetDelta: int32
firstTimestamp: int64
maxTimestamp: int64
producerId: int64
producerEpoch: int16
baseSequence: int32
records: [Record]
```

Note that when compression is enabled, the compressed record data is serialized directly following the count of the number of records.

请注意，当启用压缩时，压缩的record数据会在record数量的计数后直接进行序列化。

The CRC covers the data from the attributes to the end of the batch (i.e. all the bytes that follow the CRC). It is located after the magic byte, which means that clients must parse the magic byte before deciding how to interpret the bytes between the batch length and the magic byte. The partition leader epoch field is not included in the CRC computation to avoid the need to recompute the CRC when this field is assigned for every batch that is received by the broker. The CRC-32C (Castagnoli) polynomial is used for the computation.

CRC描述了从属性到批处理结束的数据（即CRC后的所有字节）。它位于魔术字节之后，这意味着客户端必须在决定如何解释批处理长度和魔术字节之间的字节之前解析魔术字节。在CRC计算中，不包括分区(partion)leader的epoch字段，以避免在为代理接收的每个批次分配此字段时需要重新计算CRC。CRC-32C（Castagnoli）多项式用于计算。

On compaction: unlike the older message formats, magic v2 and above preserves the first and last offset/sequence numbers from the original batch when the log is cleaned. This is required in order to be able to restore the producer's state when the log is reloaded. If we did not retain the last sequence number, for example, then after a partition leader failure, the producer might see an OutOfSequence error. The base sequence number must be preserved for duplicate checking (the broker checks incoming Produce requests for duplicates by verifying that the first and last sequence numbers of the incoming batch match the last from that producer). As a result, it is possible to have empty batches in the log when all the records in the batch are cleaned but batch is still retained in order to preserve a producer's last sequence number. One oddity here is that the baseTimestamp field is not preserved during compaction, so it will change if the first record in the batch is compacted away.

压缩：与旧的消息格式不同，当清理日志时，magic v2及以上版本会保留原始批次的第一个和最后一个偏移/序列号。这是为了能够在日志重新加载时恢复生产者的状态。例如，如果我们没有保留最后的序列号，那么在分区引导失败后，生产者（producer）可能会看到OutOfSequence错误。所以，必须保留基本序列号以进行重复检查（代理通过验证传入批次的第一个和最后一个序列号与来自该生产者的最后一个序列号相匹配来检查传入的Produce对重复的请求）。因此，当批次中的所有记录都被清除但批次仍被保留以保留生产者的最后序列号时，可能在日志中有空批次。奇怪的是，baseTimestamp字段在压缩过程中不会保留，所以如果批处理中的第一条记录被压缩，它将会改变。

### 5.3.1.1 Control Batches

A control batch contains a single record called the control record. Control records should not be passed on to applications. Instead, they are used by consumers to filter out aborted transactional messages.

一个控制批次（control batch）包含单个记录(record)时，其称为控制记录（control record）。control record不应该传递给应用程序。相反，它们被消费者用来过滤掉中止的交易消息。

The key of a control record conforms to the following schema:

control record的key符合以下设计：

```
version: int16 (current version is 0)
type: int16 (0 indicates an abort marker, 1 indicates a commit)
```

The schema for the value of a control record is dependent on the type. The value is opaque to clients.

control record的值（value）的设计取决于key类型。对客户端来说value是不确定的。

## 5.3.2 Record

Record level headers were introduced in Kafka 0.11.0. The on-disk format of a record with Headers is delineated below.

record层次的头部在Kafka 0.11.0版本中引入。带有头部的record的磁盘格式如下所示。

```
length: varint
attributes: int8
    bit 0~7: unused
timestampDelta: varint
offsetDelta: varint
keyLength: varint
key: byte[]
valueLen: varint
value: byte[]
Headers => [Header]

```

### 5.3.2.1 Record Header

```
headerKeyLength: varint
headerKey: String
headerValueLength: varint
Value: byte[]

```

We use the same varint encoding as Protobuf. More information on the latter can be found [here](https://developers.google.com/protocol-buffers/docs/encoding#varints). The count of headers in a record is also encoded as a varint.

我们使用与Protobuf相同的varint编码。关于后者的更多信息可以在[这里](https://developers.google.com/protocol-buffers/docs/encoding#varints)找到。record中头部的数量也被编码为varint。