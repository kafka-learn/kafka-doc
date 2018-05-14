# KAFKA CONNECT

## Overview

## 概况

Kafka Connect is a tool for scalably and reliably streaming data between Apache Kafka and other systems. It makes it simple to quickly define connectors that move large collections of data into and out of Kafka. Kafka Connect can ingest entire databases or collect metrics from all your application servers into Kafka topics, making the data available for stream processing with low latency. An export job can deliver data from Kafka topics into secondary storage and query systems or into batch systems for offline analysis.

Kafka Connect是一个可扩展的、可靠的在Apache Kafka和其他系统之间传输数据的工具。它使得快速定义能将大量数据传入和传出Kafka的Connectors变得很简单。Kafka Connect可以摄取整个数据库或从应用程序服务器收集指标到Kafka主题中，使数据可用于低延迟的流处理。一个导出作业可以将来自Kafka主题的数据传送到辅助存储和查询系统或批量系统中进行离线分析。

Kafka Connect features include:

Kafka Connect 的特点包括：

* **A common framework for Kafka connectors** - Kafka Connect standardizes integration of other data systems with Kafka, simplifying connector development, deployment, and management

* **Distributed and standalone modes** - scale up to a large, centrally managed service supporting an entire organization or scale down to development, testing, and small production deployments

* **REST interface** - submit and manage connectors to your Kafka Connect cluster via an easy to use REST API

* **Automatic offset management** - with just a little information from connectors, Kafka Connect can manage the offset commit process automatically so connector developers do not need to worry about this error prone part of connector development

* **Distributed and scalable by default** - Kafka Connect builds on the existing group management protocol. More workers can be added to scale up a Kafka Connect cluster.

* **Streaming/batch integration** - leveraging Kafka's existing capabilities, Kafka Connect is an ideal solution for bridging streaming and batch data systems


* **Kafka connect的通用框架** - Kafka Connect将其它数据系统与Kafka的集成标准化，简化了connector的开发、部署和管理

* **分布式和独立模式** - 扩展到支持整个组织的大型集中管理服务，或者扩展到开发、测试和小型生产部署

* **REST 接口** - 通过使用简单的REST API来向Kafka Connect群集提交和管理connector

* **自动偏移(automatic offset)管理** - 只需connector的一些信息，Kafka Connect就可以自动管理偏移提交过程，因此connector开发人员无需担心这个错误会成为connector开发中的一部分

* **默认分布式和可扩展** - Kafka Connect基于现有的组管理协议构建。可以添加更多工作线程来扩展Kafka Connect群集。

* **流/批量的整合** - 利用Kafka现有的功能，Kafka Connect是桥接流和批量数据系统的理想解决方案

## User Guide

## 用户指南

The quickstart provides a brief example of how to run a standalone version of Kafka Connect. This section describes how to configure, run, and manage Kafka Connect in more detail.

快速开始部分给出了如何运行独立模式的Kafka Connect的简要示例。本节会介绍如何更详细地配置、运行和管理Kafka Connect。

### Running Kafka Connect

### 运行 Kafka Connect

Kafka Connect currently supports two modes of execution: standalone (single process) and distributed.

Kafka Connect目前支持两种运行模式：独立的（单进程）和分布式的。

In standalone mode all work is performed in a single process. This configuration is simpler to setup and get started with and may be useful in situations where only one worker makes sense (e.g. collecting log files), but it does not benefit from some of the features of Kafka Connect such as fault tolerance. You can start a standalone process with the following command:

在独立(standalone)模式下，所有工作线程都在一个进程中执行。这种配置更容易设置并开始使用，并且在只有一个工作线程起作用的情况下可能更有用(例如收集日志文件)，但它不会从Kafka Connect的某些功能(例如容错功能)中受益。您可以使用以下命令启动一个独立模式的进程：

```bash
> bin/connect-standalone.sh config/connect-standalone.properties connector1.properties [connector2.properties ...]
```

The first parameter is the configuration for the worker. This includes settings such as the Kafka connection parameters, serialization format, and how frequently to commit offsets. The provided example should work well with a local cluster running with the default configuration provided by ```config/server.properties```. It will require tweaking to use with a different configuration or production deployment. All workers (both standalone and distributed) require a few configs:

第一个参数是工作线程的配置。这包括诸如Kafka连接参数，序列化格式以及提交偏移的频率等设置。提供的示例可以在使用由```config/server.properties```提供的默认配置运行的本地群集上正常工作。它需要调整以配合不同的配置或生产部署。所有工作线程（包括独立的和分布式的模式）都需要一些配置：

* bootstrap.servers - List of Kafka servers used to bootstrap connections to Kafka

    bootstrap.servers - 用于引导与Kafka连接的服务器列表

* key.converter - Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the keys in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro.

    key.converter - 转换器类，用于在Kafka Connect格式和写入Kafka的序列化表单之间进行转换。这将控制写入Kafka或从Kafka读取的消息中的key格式，因为这与connector无关，所以它允许任何connector使用任意的序列化格式。常见的格式包括JSON和Avro。

* value.converter - Converter class used to convert between Kafka Connect format and the serialized form that is written to Kafka. This controls the format of the values in messages written to or read from Kafka, and since this is independent of connectors it allows any connector to work with any serialization format. Examples of common formats include JSON and Avro.
    
    value.converter - 转换器类，用于在Kafka Connect格式和写入Kafka的序列化表单之间进行转换。这将控制写入Kafka或从Kafka读取的消息中的值的格式，因为这与connecotr无关，所以它允许任何connector使用任何序列化格式。常见格式的例子包括JSON和Avro。

The important configuration options specific to standalone mode are:

独立模式下的重要配置选项是：

* offset.storage.file.filename - File to store offset data in

    offset.storage.file.filename - 用来存储写入数据偏移的文件名

The parameters that are configured here are intended for producers and consumers used by Kafka Connect to access the configuration, offset and status topics. For configuration of Kafka source and Kafka sink tasks, the same parameters can be used but need to be prefixed with ```consumer.``` and ```producer.``` respectively. The only parameter that is inherited from the worker configuration is ```bootstrap.servers```, which in most cases will be sufficient, since the same cluster is often used for all purposes. A notable exeption is a secured cluster, which requires extra parameters to allow connections. These parameters will need to be set up to three times in the worker configuration, once for management access, once for Kafka sinks and once for Kafka sources.

此处配置的参数被Kafka Connect使用的生产者和消费者用来访问配置、数据偏移和各种状态的主题。对于Kafka的source和sink任务配置，可以使用相同的参数，但需要分别以```consumer.```和```producer.```为前缀。从工作线程配置继承来的唯一参数是```bootstrap.servers```，在大多数情况下这是足够的，因为同一个群集通常用于所有操作目标。一个值得注意的例外是一个安全集群，它需要额外的参数来允许连接。这些参数需要在工作线程的配置中设置为三次，一次用于管理访问，一次用于Kafka sink，还有一次用Kafka source。

The remaining parameters are connector configuration files. You may include as many as you want, but all will execute within the same process (on different threads).

其余参数是connector配置文件。你可以配置尽可能多的，但都会在同一个进程内（在不同的线程上）执行。

Distributed mode handles automatic balancing of work, allows you to scale up (or down) dynamically, and offers fault tolerance both in the active tasks and for configuration and offset commit data. Execution is very similar to standalone mode:

分布式模式处理工作线程是自平衡的，允许动态的扩展(或缩小)，并提供包括配置的active任务以和偏移量提交数据的容错能力。其执行与独立模式非常相似：

```bash
> bin/connect-distributed.sh config/connect-distributed.properties
```

The difference is in the class which is started and the configuration parameters which change how the Kafka Connect process decides where to store configurations, how to assign work, and where to store offsets and task statues. In the distributed mode, Kafka Connect stores the offsets, configs and task statuses in Kafka topics. It is recommended to manually create the topics for offset, configs and statuses in order to achieve the desired the number of partitions and replication factors. If the topics are not yet created when starting Kafka Connect, the topics will be auto created with default number of partitions and replication factor, which may not be best suited for its usage.

不同之处在于启动的类以及一些配置参数，这些参数包括了Kafka Connect处理过程如何决定存储配置位置、如何分配工作、哪里存储偏移量和任务状态。在分布式模式下，Kafka Connect将偏移量、配置和任务状态存储在Kafka主题中。建议手动创建偏移量，配置和状态的主题以实现所需的分区数量和备份因子。如果在启动Kafka Connect时还未创建主题，则会使用默认的分区数和备份因子自动创建主题，这可能不是Kafka Connedct的最佳使用。

In particular, the following configuration parameters, in addition to the common settings mentioned above, are critical to set before starting your cluster:

除了上面提到的常用设置之外，在启动集群之前，以下参数的设置是至关重要的：

* group.id (default ```connect-cluster```) - unique name for the cluster, used in forming the Connect cluster group; note that this **must not conflict** with consumer group IDs

* config.storage.topic (default ```connect-configs```) - topic to use for storing connector and task configurations; note that this should be a single partition, highly replicated, compacted topic. You may need to manually create the topic to ensure the correct configuration as auto created topics may have multiple partitions or be automatically configured for deletion rather than compaction

* offset.storage.topic (default ```connect-offsets```) - topic to use for storing offsets; this topic should have many partitions, be replicated, and be configured for compaction

* status.storage.topic (default ```connect-status```) - topic to use for storing statuses; this topic can have multiple partitions, and should be replicated and configured for compaction


* group.id(默认为```connect-cluster```) - 集群的唯一名称，用于形成Connect集群组；请注意，这**不得与消费者组的Id相冲突**

* config.storage.topic（默认为```connect-configs```） - 用于存储connector和任务配置的主题；请注意，这应该是y一个单分区的，高度备份的，压缩的主题。您可能需要手动创建主题以确保正确的配置，因为自动创建的主题可能有多个分区，或者会自动配置删除而不是压缩形式的主题

* offset.storage.topic（默认为```connect-offsets```） - 用于存储偏移量的主题；这个主题应该有许多分区，被备份，并被配置为压缩

* status.storage.topic（默认为```connect-status```） - 用于存储状态的主题；此主题可以有多个分区，并且应该进行备份和配置以进行压缩

Note that in distributed mode the connector configurations are not passed on the command line. Instead, use the REST API described below to create, modify, and destroy connectors.

请注意，在分布式模式下，connector配置不会在命令行上传递。相反，请使用下面描述的REST API来创建、修改和销毁connector。

### Configuring Connectors

### 配置Connector

Connector configurations are simple key-value mappings. For standalone mode these are defined in a properties file and passed to the Connect process on the command line. In distributed mode, they will be included in the JSON payload for the request that creates (or modifies) the connector.

Connector的配置是简单的键值映射。对于独立(standalone)模式，这些配置在属性文件中定义并在命令行上传递给Connect进程。在分布式模式下，它们将被包含在创建(或修改)connector的请求的JSON负载中。

Most configurations are connector dependent, so they can't be outlined here. However, there are a few common options:

大多数配置都依赖于connector，所以在这里不能一一概述。但是，有一些相同的选项：

* ```name``` - Unique name for the connector. Attempting to register again with the same name will fail.sxx
* ```connector.class``` - The Java class for the connector
* ```tasks.max``` - The maximum number of tasks that should be created for this connector. The connector may create fewer tasks if it cannot achieve this level of parallelism.
* ```key.converter``` - (optional) Override the default key converter set by the worker.
* ```value.converter``` - (optional) Override the default value converter set by the worker.


* ```name``` - connector的唯一名称。如果尝试使用相同的名称注册则会失败
* ```connector.class``` -  connector的Java类
* ```tasks.max``` - 应该为connector创建一个最大任务数。如果无法达到此级别的并行，connector可能会创建更少的任务
* ```key.converter``` - (可选的)覆盖由工作线程设置的默认key convertor。
* ```value.converter``` - (可选的)覆盖由工作线程设置的默认value convertor。

The connector.class config supports several formats: the full name or alias of the class for this connector. If the connector is org.apache.kafka.connect.file.FileStreamSinkConnector, you can either specify this full name or use FileStreamSink or FileStreamSinkConnector to make the configuration a bit shorter.

connector.class 配置支持多种格式：该connector类的全名或别名。如果connector是org.apache.kafka.connect.file.FileStreamSinkConnector，则可以指定此全名，也可以使用FileStreamSink或FileStreamSinkConnector来缩短配置。

Sink connectors also have a few additional options to control their input. Each sink connector must set one of the following:

sink connector 还有一些额外的选项来控制其输入。每个sink connector都必须设置如下配置中的一个：

* ```topics``` - A comma-separated list of topics to use as input for this connector
* ```topics.regex``` - A Java regular expression of topics to use as input for this connector


* ```topics``` - 以逗号分隔的主题列表，并作为该connector的输入
* ```topics.regex```  - 用Java正则表达式表示的一系列主题，并作为该connector的输入

### Transformations

### 转换器

Connectors can be configured with transformations to make lightweight message-at-a-time modifications. They can be convenient for data massaging and event routing.

可以使用转换器对connector进行配置，以实现轻量级的消息一次性的修改。这可以方便地进行数据传输和事件路由。

A transformation chain can be specified in the connector configuration.

可以在connector的配置中指定一个转换器链。

* ```transforms``` - List of aliases for the transformation, specifying the order in which the transformations will be applied.
* ```transforms.$alias.type``` - Fully qualified class name for the transformation.
* ```transforms.$alias.$transformationSpecificConfig``` Configuration properties for the transformation


* ```transforms``` - 转换器的别名列表，指定了转换器使用的顺序。
* ```transforms.$alias.type``` - 转换器的完整类名
* ```transforms.$alias.$transformationSpecificConfig``` 转换器的配置属性

For example, lets take the built-in file source connector and use a transformation to add a static field.

例如，可以添加一个内置的文件source connector，然后再使用一个转换器来添加一个静态字段。

Throughout the example we'll use schemaless JSON data format. To use schemaless format, we changed the following two lines in ```connect-standalone.properties``` from true to false:

在整个例子中，我们将使用无模式的JSON数据格式。要使用无模式格式，我们将```connect-standalone.properties```中的以下两行从true更改为false：

```
key.converter.schemas.enable
value.converter.schemas.enable
```

The file source connector reads each line as a String. We will wrap each line in a Map and then add a second field to identify the origin of the event. To do this, we use two transformations:

文件source connector将每行读取为一个String。我们会将每一行包装在一个Map中，然后添加第二个字段来标识事件的来源。为此，我们使用两个转换器：

* **HoistField** to place the input line inside a Map
* **InsertField** to add the static field. In this example we'll indicate that the record came from a file connector


* **HoistField** 将输入行放入Map中
* **InsertField** 添加静态字段。在这个例子中，我们将指出记录record来自一个文件connector

After adding the transformations, ```connect-file-source.properties``` file looks as following:

当添加转换器后, ```connect-file-source.properties```文件显示如下：

```
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=test.txt
topic=connect-test
transforms=MakeMap, InsertSource
transforms.MakeMap.type=org.apache.kafka.connect.transforms.HoistField$Value
transforms.MakeMap.field=line
transforms.InsertSource.type=org.apache.kafka.connect.transforms.InsertField$Value
transforms.InsertSource.static.field=data_source
transforms.InsertSource.static.value=test-file-source
```

All the lines starting with ```transforms``` were added for the transformations. You can see the two transformations we created: "InsertSource" and "MakeMap" are aliases that we chose to give the transformations. The transformation types are based on the list of built-in transformations you can see below. Each transformation type has additional configuration: HoistField requires a configuration called "field", which is the name of the field in the map that will include the original String from the file. InsertField transformation lets us specify the field name and the value that we are adding.

所有以```transforms```开始的行都表示添加的转换器。您可以看到我们创建的两个转换器：“InsertSource”和“MakeMap”是我们选择的转换器的别名。转换类型基于您可以在下面看到的内置转换列表。每种转换器都有附加的配置：HoistField需要一个名为“field”的配置，它是映射中将包含文件中原始String的字段的名称。InsertField转换器可以让我们指定字段名称和我们添加的值。

When we ran the file source connector on my sample file without the transformations, and then read them using ```kafka-console-consumer.sh```, the results were:

当我们不使用转换器，直接使用文件source connector运行样例时，然后使用```kafka-console-consumer.sh```读取它们时，结果如下：

```
"foo"
"bar"
"hello world"
```

We then create a new file connector, this time after adding the transformations to the configuration file. This time, the results will be:

然后，我们创建一个新的文件connector。 这一次将转换器添加到配置文件中，结果将如下所示：

```bash
{"line":"foo","data_source":"test-file-source"}
{"line":"bar","data_source":"test-file-source"}
{"line":"hello world","data_source":"test-file-source"}
```

You can see that the lines we've read are now part of a JSON map, and there is an extra field with the static value we specified. This is just one example of what you can do with transformations.

您可以看到我们读取的行现在是JSON映射的一部分，并且还有一个带有我们指定的静态值的额外字段。这只是使用转换器后的一个例子。

Several widely-applicable data and routing transformations are included with Kafka Connect:

Kafka Connect包含几个广泛适用的数据和路由转换器：

* InsertField - Add a field using either static data or record metadata
* ReplaceField - Filter or rename fields
* MaskField - Replace field with valid null value for the type (0, empty string, etc)
* ValueToKey
* HoistField - Wrap the entire event as a single field inside a Struct or a Map
* ExtractField - Extract a specific field from Struct and Map and include only this field in results
* SetSchemaMetadata - modify the schema name or version
* TimestampRouter - Modify the topic of a record based on original topic and timestamp. Useful when using a sink that needs to write to different tables or indexes based on timestamps
* RegexRouter - modify the topic of a record based on original topic, replacement string and a regular expression

Details on how to configure each transformation are listed below:

下面列出了有关如何配置每个转换器的详细信息：

**org.apache.kafka.connect.transforms.InsertField**

Insert field(s) using attributes from the record metadata or a configured static value.

Insert字段来自记录元数据或配置中的一个静态值

Use the concrete transformation type designed for the record key

使用为记录键设计的具体转换器类型

(```org.apache.kafka.connect.transforms.InsertField$Key```) or value

(```org.apache.kafka.connect.transforms.InsertField$Value```).

NAME | DESCRIPTION	| TYPE	| DEFAULT |	VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | --- 
offset.field |	Field name for Kafka offset - only applicable to sink connectors. Suffix with ! to make this a required field, or ? to keep it optional (the default).	| string	| null|		| medium
partition.field	| Field name for Kafka partition. Suffix with ! to make this a required field, or ? to keep it optional (the default).	| string	| null	|	| medium
static.field	| Field name for static data field. Suffix with ! to make this a required field, or ? to keep it optional (the default).	| string	| null	|	| medium
static.value	| Static field value, if field name configured.	| string | 	null |	|	medium
timestamp.field	| Field name for record timestamp. Suffix with ! to make this a required field, or ? to keep it optional (the default).	| string	| null	|	| medium
topic.field	| Field name for Kafka topic. Suffix with ! to make this a required field, or ? to keep it optional (the default).	| string	| null	|	| medium

**org.apache.kafka.connect.transforms.ReplaceField**

Filter or rename fields.

Use the concrete transformation type designed for the record key 

(```org.apache.kafka.connect.transforms.ReplaceField$Key```) or value 

(```org.apache.kafka.connect.transforms.ReplaceField$Value```).

NAME	| DESCRIPTION	| TYPE	| DEFAULT	| VALID VALUES	| IMPORTANCE
--- | --- | --- | --- | --- | --- 
blacklist |	Fields to exclude. This takes precedence over the whitelist.	|list| "" | | medium
renames	| Field rename mappings.	| list	| ""	|list of colon-delimited pairs, e.g. foo:bar,abc:xyz	| medium
whitelist |	Fields to include. If specified, only these fields will be used.| list| ""|	| medium


**org.apache.kafka.connect.transforms.MaskField**

Mask specified fields with a valid null value for the field type (i.e. 0, false, empty string, and so on).

Use the concrete transformation type designed for the record key 

(```org.apache.kafka.connect.transforms.MaskField$Key```) or value 

(```org.apache.kafka.connect.transforms.MaskField$Value```).

NAME	| DESCRIPTION |	TYPE |	DEFAULT	| VALID VALUES	| IMPORTANCE
--- | --- | --- | --- | --- | ---
fields	| Names of fields to mask.	| list	|	| non-empty list |	high

**org.apache.kafka.connect.transforms.ValueToKey**

Replace the record key with a new key formed from a subset of fields in the record value.

NAME	| DESCRIPTION |	TYPE |	DEFAULT	| VALID VALUES	| IMPORTANCE
--- | --- | --- | --- | --- | ---
fields	| Field names on the record value to extract as the record key.	| list | | non-empty list	| high

**org.apache.kafka.connect.transforms.HoistField**

Wrap data using the specified field name in a Struct when schema present, or a Map in the case of schemaless data.

Use the concrete transformation type designed for the record key

(```org.apache.kafka.connect.transforms.HoistField$Key```) or value 

(```org.apache.kafka.connect.transforms.HoistField$Value```).

NAME |	DESCRIPTION	| TYPE	| DEFAULT | VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
field	| Field name for the single field that will be created in the resulting Struct or Map.	| string |	|	| medium

**org.apache.kafka.connect.transforms.ExtractField**

Extract the specified field from a Struct when schema present, or a Map in the case of schemaless data. Any null values are passed through unmodified.

Use the concrete transformation type designed for the record key 

(```org.apache.kafka.connect.transforms.ExtractField$Key```) or value 

(```org.apache.kafka.connect.transforms.ExtractField$Value```).

NAME |	DESCRIPTION	| TYPE	| DEFAULT| VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
field	| Field name to extract.|	string| |	|medium

**org.apache.kafka.connect.transforms.SetSchemaMetadata**

Set the schema name, version or both on the record's key 

(```org.apache.kafka.connect.transforms.SetSchemaMetadata$Key```) or value 

(```org.apache.kafka.connect.transforms.SetSchemaMetadata$Value```) schema.

NAME |	DESCRIPTION	| TYPE	| DEFAULT	| VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
schema.name	| Schema name to set.	| string	| null | | high
schema.version	| Schema version to set. |	int	| null | | high

**org.apache.kafka.connect.transforms.TimestampRouter**

Update the record's topic field as a function of the original topic value and the record timestamp.

This is mainly useful for sink connectors, since the topic field is often used to determine the equivalent entity name in the destination system(e.g. database table or search index name).

NAME |	DESCRIPTION	| TYPE	| DEFAULT |	VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
timestamp.format	| Format string for the timestamp that is compatible with java.text.SimpleDateFormat.	| string	|yyyyMMdd |	|	high
topic.format	| Format string which can contain ${topic} and ${timestamp} as placeholders for the topic and timestamp, respectively.	| string	| ${topic}-${timestamp}		| | high

**org.apache.kafka.connect.transforms.RegexRouter**

Update the record topic using the configured regular expression and replacement string.

Under the hood, the regex is compiled to a ```java.util.regex.Pattern```. If the pattern matches the input topic, ```java.util.regex.Matcher#replaceFirst()``` is used with the replacement string to obtain the new topic.

NAME |	DESCRIPTION	| TYPE	| DEFAULT	| VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
regex	| Regular expression to use for matching.	| string|	|	valid regex	| high
replacement	| Replacement string.	| string		|	| | high


**org.apache.kafka.connect.transforms.Flatten**

Flatten a nested data structure, generating names for each field by concatenating the field names at each level with a configurable delimiter character. Applies to Struct when schema present, or a Map in the case of schemaless data. The default delimiter is '.'.

Use the concrete transformation type designed for the record key 

(```org.apache.kafka.connect.transforms.Flatten$Key```) or value 

(```org.apache.kafka.connect.transforms.Flatten$Value```).

NAME |	DESCRIPTION	| TYPE	| DEFAULT	| VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
delimiter	| Delimiter to insert between field names from the input record when generating field names for the output record	| string	| .	|	| medium

**org.apache.kafka.connect.transforms.Cast**

Cast fields or the entire key or value to a specific type, e.g. to force an integer field to a smaller width. Only simple primitive types are supported -- integers, floats, boolean, and string.

Use the concrete transformation type designed for the record key 

(```org.apache.kafka.connect.transforms.Cast$Key```) or value 

(```org.apache.kafka.connect.transforms.Cast$Value```).

NAME |	DESCRIPTION	| TYPE	| DEFAULT |	VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
spec	| List of fields and the type to cast them to of the form field1:type,field2:type to cast fields of Maps or Structs. A single type to cast the entire value. Valid types are int8, int16, int32, int64, float32, float64, boolean, and string.	| list | |	list of colon-delimited pairs, e.g. foo:bar,abc:xyz	| high

**org.apache.kafka.connect.transforms.TimestampConverter**

Convert timestamps between different formats such as Unix epoch, strings, and Connect Date/Timestamp types.Applies to individual fields or to the entire value.

Use the concrete transformation type designed for the record key 

(```org.apache.kafka.connect.transforms.TimestampConverter$Key```) or value 

(```org.apache.kafka.connect.transforms.TimestampConverter$Value```).

NAME |	DESCRIPTION	| TYPE	| DEFAULT | VALID VALUES | IMPORTANCE
--- | --- | --- | --- | --- | ---
target.type	| The desired timestamp representation: string, unix, Date, Time, or Timestamp	| string	| | |	high
field	| The field containing the timestamp, or empty if the entire value is a timestamp	| string	| ""	|	| high
format	| A SimpleDateFormat-compatible format for the timestamp. Used to generate the output when type=string or used to parse the input if the input is a string.	| string	| ""	|	| medium

### REST API

Since Kafka Connect is intended to be run as a service, it also provides a REST API for managing connectors. The REST API server can be configured using the ```listeners``` configuration option. This field should contain a list of listeners in the following format: ```protocol://host:port,protocol2://host2:port2```. Currently supported protocols are ```http``` and ```https```. For example:

由于Kafka Connect旨在作为一个服务，因此它还提供了用于管理connector的REST API。REST API服务器可以使用```listeners```配置选项进行配置。该字段应包含以下格式的侦听器列表：```protocol://host:port，protocol2://host2:port2```。目前支持的协议是```http```和```https```。 例如：

```
listeners=http://localhost:8080,https://localhost:8443
```

By default, if no ```listeners``` are specified, the REST server runs on port 8083 using the HTTP protocol. When using HTTPS, the configuration has to include the SSL configuration. By default, it will use the ```ssl.*``` settings. In case it is needed to use different configuration for the REST API than for connecting to Kafka brokers, the fields can be prefixed with ```listeners.https```. When using the prefix, only the prefixed options will be used and the ```ssl.*``` options without the prefix will be ignored. Following fields can be used to configure HTTPS for the REST API:

默认情况下，如果没有指定```listeners```，则REST服务器会使用HTTP协议在端口8083上运行。当使用HTTPS时，必须包含SSL配置。默认情况下，它将使用```ssl。*```设置。如果需要为REST API使用不同的配置，而不是连接到Kafka代理，则这些字段可以加上```listeners.https```前缀。当使用前缀定义配置时，只会使用带前缀的选项，而不带前缀的```ssl。*```选项将被忽略。以下字段可用于为REST API配置HTTPS：

* ```ssl.keystore.location```
* ```ssl.keystore.password```
* ```ssl.keystore.type```
* ```ssl.key.password```
* ```ssl.truststore.location```
* ```ssl.truststore.password```
* ```ssl.truststore.type```
* ```ssl.enabled.protocols```
* ```ssl.provider```
* ```ssl.protocol```
* ```ssl.cipher.suites```
* ```ssl.keymanager.algorithm```
* ```ssl.secure.random.implementation```
* ```ssl.trustmanager.algorithm```
* ```ssl.endpoint.identification.algorithm```
* ```ssl.client.auth```

The REST API is used not only by users to monitor / manage Kafka Connect. It is also used for the Kafka Connect cross-cluster communication. Requests received on the follower nodes REST API will be forwarded to the leader node REST API. In case the URI under which is given host reachable is different from the URI which it listens on, the configuration options rest.advertised.host.name, rest.advertised.port and rest.advertised.listener can be used to change the URI which will be used by the follower nodes to connect with the leader. When using both HTTP and HTTPS listeners, the ```rest.advertised.listener``` option can be also used to define which listener will be used for the cross-cluster communication. When using HTTPS for communication between nodes, the same ```ssl.*``` or ```listeners.https``` options will be used to configure the HTTPS client.

Rest API不仅被用户用来监视/管理Kafka Connect。它也用于Kafka Connect跨集群通信。在follower节点的Rest API上接受到的请求会被转发到leader节点的Rest API上。如果给定主机可达的URI与它所监听的URI不同，配置选项rest.advertised.host.name，rest.advertised.port和rest.advertised.listener可用于更改follower节点与leader连接使用的URI。同时使用HTTP和HTTPS的listener时，还可以使用```rest.advertised.listener```选项来定义哪个listener将用于跨集群通信。当使用HTTPS进行节点之间的通信时，将使用相同的```ssl.*```或```listeners.https```选项来配置HTTPS客户端。

The following are the currently supported REST API endpoints:

以下是当前支持的REST API端点：

* ```GET /connectors``` - return a list of active connectors
* ```POST /connectors``` - create a new connector; the request body should be a JSON object containing a string name field and an object config field with the connector configuration parameters
* ```GET /connectors/{name}``` - get information about a specific connector
* ```GET /connectors/{name}/config``` - get the configuration parameters for a specific connector
* ```PUT /connectors/{name}/config``` - update the configuration parameters for a specific connector
* ```GET /connectors/{name}/status``` - get current status of the connector, including if it is running, failed, paused, etc., which worker it is assigned to, error information if it has failed, and the state of all its tasks
* ```GET /connectors/{name}/tasks``` - get a list of tasks currently running for a connector
* ```GET /connectors/{name}/tasks/{taskid}/status``` - get current status of the task, including if it is running, failed, paused, etc., which worker it is assigned to, and error information if it has failed
* ```PUT /connectors/{name}/pause``` - pause the connector and its tasks, which stops message processing until the connector is resumed
* ```PUT /connectors/{name}/resume``` - resume a paused connector (or do nothing if the connector is not paused)
* ```POST /connectors/{name}/restart``` - restart a connector (typically because it has failed)
* ```POST /connectors/{name}/tasks/{taskId}/restart``` - restart an individual task (typically because it has failed)
* ```DELETE /connectors/{name}``` - delete a connector, halting all tasks and deleting its configuration


* ```GET /connectors``` - 返回活动connector列表
* ```POST /connectors``` - 创建一个新的connector；请求主体应该是包含字符串名称字段的JSON对象，以及包含connector配置参数的对象配置字段
* ```GET /connectors/{name}``` - 获取特定connector的相关信息
* ```GET /connectors/{name}/config``` - 获取特定connector的配置参数
* ```PUT /connectors/{name}/config``` - 更新特定connector的配置参数
* ```GET /connectors/{name}/status``` - 获取connector的当前状态，包括connector是否正在运行、失败、暂停等，分配给哪个工作线程，错误信息(如果运行失败)以及所有任务的状态
* ```GET /connectors/{name}/tasks``` - 获取当前connector运行的任务列表
* ```GET /connectors/{name}/tasks/{taskid}/status``` - 获取任务的当前状态，包括如果它正在运行、失败、暂停等，分配给哪个工作线程，以及失败信息(如果失败)
* ```PUT /connectors/{name}/pause``` - 暂停connector及其任务，停止消息处理直到connector恢复
* ```PUT /connectors/{name}/resume``` - 恢复一个暂停状态的connector(如果connector不是暂停状态，则不执行任何操作)
* ```POST /connectors/{name}/restart``` - 重新启动connector(通常是因为它失败了)
* ```POST /connectors/{name}/tasks/{taskId}/restart``` - 重新启动一个单独的任务(通常是因为它失败了)
* ```DELETE /connectors/{name}``` - 删除connector，停止所有任务并删除其配置

Kafka Connect also provides a REST API for getting information about connector plugins:

Kafka Connect还提供了用于获取有关connector插件信息的REST API:

* ```GET /connector-plugins``` - return a list of connector plugins installed in the Kafka Connect cluster. Note that the API only checks for connectors on the worker that handles the request, which means you may see inconsistent results, especially during a rolling upgrade if you add new connector jars
* ```PUT /connector-plugins/{connector-type}/config/validate``` - validate the provided configuration values against the configuration definition. This API performs per config validation, returns suggested values and error messages during validation.


* ```GET /connector-plugins``` - 返回安装在Kafka Connect集群中的connector插件列表。请注意，该API仅检查能处理请求的工作线程上的connector，这意味着您可能会看到不一致的结果，尤其是在滚动升级期间，如果添加新的connector jar
* ```PUT /connector-plugins/{connector-type}/config/validate``` - 根据配置定义验证提供的配置值。此API执行每个配置验证，在验证期间返回建议值和错误消息

## Connector Development Guide

## Connector 开发指南

This guide describes how developers can write new connectors for Kafka Connect to move data between Kafka and other systems. It briefly reviews a few key concepts and then describes how to create a simple connector.

本指南介绍开发人员如何为Kafka Connect编写新connector以在Kafka和其他系统之间移动数据。它简要回顾了几个关键概念，然后描述了如何创建一个简单的connector。

### Core Concepts and APIs

### 核心概念与API

#### Connectors and Tasks

#### Connector与任务(Task)

To copy data between Kafka and another system, users create a ```Connector``` for the system they want to pull data from or push data to. Connectors come in two flavors: ```SourceConnectors``` import data from another system (e.g. ```JDBCSourceConnector``` would import a relational database into Kafka) and ```SinkConnectors``` export data (e.g. ```HDFSSinkConnector``` would export the contents of a Kafka topic to an HDFS file).

要在Kafka和另一个系统之间复制数据，用户需要为他们想要从中提取数据或将数据推送到的系统创建一个```Connector```。Connector有两种类型：```SourceConnectors```，从另一个系统导入数据(例如，```JDBCSourceConnector```会将关系数据库导入到Kafka中)；```SinkConnector```，导出数据(例如```HDFSSinkConnector```会将Kafka主题的内容导出到HDFS文件中)。

Connectors do not perform any data copying themselves: their configuration describes the data to be copied, and the ```Connector``` is responsible for breaking that job into a set of ```Tasks``` that can be distributed to workers. These Tasks also come in two corresponding flavors: ```SourceTask``` and ```SinkTask```.

Connectors 不会执行任何数据复制操作：它的配置描述了要复制的数据，```Connector```负责将这个任务分解为一组```Tasks```，其可以分发给工作线程。这些任务(Task)也有两种相应的风格：```SourceTask```和```SinkTask```。

With an assignment in hand, each ```Task``` must copy its subset of the data to or from Kafka. In Kafka Connect, it should always be possible to frame these assignments as a set of input and output streams consisting of records with consistent schemas. Sometimes this mapping is obvious: each file in a set of log files can be considered a stream with each parsed line forming a record using the same schema and offsets stored as byte offsets in the file. In other cases it may require more effort to map to this model: a JDBC connector can map each table to a stream, but the offset is less clear. One possible mapping uses a timestamp column to generate queries incrementally returning new data, and the last queried timestamp can be used as the offset.

注备好分配任务后，每个```Task```必须将数据的子集复制到Kafka或从Kafka复制。在Kafka Connect中，应始终可以将这些分配框架化为一组输入和输出流，这些输入和输出流由具有一致模式的记录组成。 有时候这种映射是显而易见的：一组日志文件中的每个文件可以被认为是一个流，每条解析的行使用相同的模式和偏移量作为字节偏移量存储在文件中。 在其他情况下，可能需要更多努力来映射到此模型：一个JDBC connector可将每个表映射到流，但偏移量不太清晰。一个可能的映射使用时间戳列来生成增量返回新数据的查询，最后查询的时间戳可以用作偏移量。

#### Streams and Records

#### 流与记录

Each stream should be a sequence of key-value records. Both the keys and values can have complex structure -- many primitive types are provided, but arrays, objects, and nested data structures can be represented as well. The runtime data format does not assume any particular serialization format; this conversion is handled internally by the framework.

每个流应该是一系列键值记录（key-value record）。键和值都可以具有复杂的结构 - 许多基本类型都提供了的，并且也可以表示数组，对象和嵌套数据结构。运行时数据格式不承担任何特定的序列化格式；此转换由框架内部处理。

In addition to the key and value, records (both those generated by sources and those delivered to sinks) have associated stream IDs and offsets. These are used by the framework to periodically commit the offsets of data that have been processed so that in the event of failures, processing can resume from the last committed offsets, avoiding unnecessary reprocessing and duplication of events.

除了key(键)和value(值)之外，记录(包括source和传送到sink的记录)都有相关的流Id和偏移量。框架会定期的提交已处理数据的偏移量，以便在发生故障时，可以从最后一次提交的偏移量恢复处理，避免不必要的重新处理和事件拷贝。

#### Dynamic Connectors

#### 动态 Connector

Not all jobs are static, so ```Connector``` implementations are also responsible for monitoring the external system for any changes that might require reconfiguration. For example, in the ```JDBCSourceConnector``` example, the ```Connector``` might assign a set of tables to each ```Task```. When a new table is created, it must discover this so it can assign the new table to one of the ```Tasks``` by updating its configuration. When it notices a change that requires reconfiguration (or a change in the number of ```Tasks```), it notifies the framework and the framework updates any corresponding Tasks

并非所有的任务都是静态的，因此```Connector```实现还负责监视外部系统是否有可能需要重新配置的更改。例如，在```JDBCSourceConnector```示例中，```Connector```可能会为每个```Task```分配一组表。当创建新表时，必须能发现它，以便通过更新其配置来将新表分配给其中一个```Tasks```。当发现需要重新配置的变更(或```Tasks```数量的变化)时，它会通知框架更新相应的任务。

### Developing a Simple Connector

### 开发一个简单的 Connector

Developing a connector only requires implementing two interfaces, the ```Connector``` and ```Task```. A simple example is included with the source code for Kafka in the ```file``` package. This connector is meant for use in standalone mode and has implementations of a ```SourceConnector```/```SourceTask``` to read each line of a file and emit it as a record and a ```SinkConnector```/```SinkTask``` that writes each record to a file.

开发一个Connector只需要实现两个接口，即```Connector```和```Task```。一个简单的例子包含在Kafka源代码的```file```包中。这个Connector用于独立模式，并且具有一个```SourceConnector```/```SourceTask```来读取文件的每一行并将它作为一个记录和一个```SinkConnector```/```SinkTask```将每条记录写入文件。

The rest of this section will walk through some code to demonstrate the key steps in creating a connector, but developers should also refer to the full example source code as many details are omitted for brevity.

本节的其余部分将通过一些代码演示创建Connector的关键步骤，但开发人员还应参考完整的示例源代码，因为为简洁起见，省略了许多细节。

#### Connector Example

#### Connctor 样例

We'll cover the ```SourceConnector``` as a simple example. ```SinkConnector``` implementations are very similar. Start by creating the class that inherits from ```SourceConnector``` and add a couple of fields that will store parsed configuration information (the filename to read from and the topic to send data to):

我们将以一个简单的例子来介绍```SourceConnector```。```SinkConnector```的实现也非常相似。首先创建一个从```SourceConnector```继承的类并添加一些字段来存储分析的配置信息（要读取的文件名以及要发送数据的主题）：

```java
public class FileStreamSourceConnector extends SourceConnector {
    private String filename;
    private String topic;
```

The easiest method to fill in is ```taskClass()```, which defines the class that should be instantiated in worker processes to actually read the data:

类中最简单的方法是```taskClass()```，它定义了应该在工作进程中实例化以实际读取数据的类：

```java
@Override
public Class<? extends Task> taskClass() {
    return FileStreamSourceTask.class;
}
```

We will define the ```FileStreamSourceTask``` class below. Next, we add some standard lifecycle methods, ```start()``` and ```stop()``` :

我们将定义```FileStreamSourceTask```类。接着，我们添加一些标准的生命周期方法，```start()```和```stop()```:

```java
@Override
public void start(Map<String, String> props) {
    // The complete version includes error handling as well.
    filename = props.get(FILE_CONFIG);
    topic = props.get(TOPIC_CONFIG);
}
 
@Override
public void stop() {
    // Nothing to do since no background monitoring is required.
}
```

Finally, the real core of the implementation is in ```taskConfigs()```. In this case we are only handling a single file, so even though we may be permitted to generate more tasks as per the ```maxTasks``` argument, we return a list with only one entry:

最后，真正核心的实现是在```taskConfigs()```方法中。在此，我们只处理单个文件，所以即使被允许按照```maxTasks```参数生成更多任务，但返回的是只包含一个条目的列表：

```java
@Override
public List<Map<String, String>> taskConfigs(int maxTasks) {
    ArrayList<Map<String, String>> configs = new ArrayList<>();
    // Only one input stream makes sense.
    Map<String, String> config = new HashMap<>();
    if (filename != null)
        config.put(FILE_CONFIG, filename);
    config.put(TOPIC_CONFIG, topic);
    configs.add(config);
    return configs;
}
```

Although not used in the example, ```SourceTask``` also provides two APIs to commit offsets in the source system: ```commit``` and ```commitRecord```. The APIs are provided for source systems which have an acknowledgement mechanism for messages. Overriding these methods allows the source connector to acknowledge messages in the source system, either in bulk or individually, once they have been written to Kafka. The ```commit``` API stores the offsets in the source system, up to the offsets that have been returned by ```poll```. The implementation of this API should block until the commit is complete. The ```commitRecord``` API saves the offset in the source system for each ```SourceRecord``` after it is written to Kafka. As Kafka Connect will record offsets automatically, ```SourceTasks``` are not required to implement them. In cases where a connector does need to acknowledge messages in the source system, only one of the APIs is typically required.

尽管没有在示例中使用，但```SourceTask```还提供了两个API来提交源系统中的偏移量：```commit```和```commitRecord```。这些API是为具有消息确认机制的源系统提供的。覆盖这些方法允许source Connector在源系统中写入Kafka后，批量或单独确认消息。```commit```API将偏移量存储在源系统中，直到```poll```返回的偏移量。这个API的实现应该是阻塞的，直到提交完成。```commitRecord```API在写入Kafka后，能将每个```SourceRecord```的源文件的偏移量保存在源系统中。由于Kafka Connect会自动记录偏移量，因此```SourceTasks```不需要执行它们。在Connector确实需要确认源系统中的消息的情况下，通常只需要其中一个API。

Even with multiple tasks, this method implementation is usually pretty simple. It just has to determine the number of input tasks, which may require contacting the remote service it is pulling data from, and then divvy them up. Because some patterns for splitting work among tasks are so common, some utilities are provided in ```ConnectorUtils``` to simplify these cases.

即使有多个任务，这个方法实现通常也很简单。它只需确定输入任务的数量，这可能需要联系远程服务，然后将它们分解。由于某些用于在任务之间分割工作的模式非常普遍，因此```ConnectorUtils```中提供了一些实用程序来简化这些情况。

Note that this simple example does not include dynamic input. See the discussion in the next section for how to trigger updates to task configs.

请注意，这个简单的示例不包含动态输入。有关如何触发对任务配置的更新，请参阅下一节中的讨论。

#### Task Example - Source Task

#### 任务样例 - Source Task

Next we'll describe the implementation of the corresponding ```SourceTask```. The implementation is short, but too long to cover completely in this guide. We'll use pseudo-code to describe most of the implementation, but you can refer to the source code for the full example.

接下来我们将描述相应```SourceTask```的实现。该实现很简单，但在本指南中如果完全的讲解，将涵盖很多的内容。我们将使用伪代码来描述大部分实现，但您可以参考完整示例的源代码。

Just as with the connector, we need to create a class inheriting from the appropriate base ```Task``` class. It also has some standard lifecycle methods:

就像Connector一样，我们需要创建一个从基类```Task```继承的类。它也有一些标准的生命周期方法：

```java
public class FileStreamSourceTask extends SourceTask {
    String filename;
    InputStream stream;
    String topic;
 
    @Override
    public void start(Map<String, String> props) {
        filename = props.get(FileStreamSourceConnector.FILE_CONFIG);
        stream = openOrThrowError(filename);
        topic = props.get(FileStreamSourceConnector.TOPIC_CONFIG);
    }
 
    @Override
    public synchronized void stop() {
        stream.close();
    }
```

These are slightly simplified versions, but show that these methods should be relatively simple and the only work they should perform is allocating or freeing resources. There are two points to note about this implementation. First, the ```start()``` method does not yet handle resuming from a previous offset, which will be addressed in a later section. Second, the ```stop()``` method is synchronized. This will be necessary because ```SourceTasks``` are given a dedicated thread which they can block indefinitely, so they need to be stopped with a call from a different thread in the Worker.

这些都是稍微简化的版本，但是显示这些方法应该相对简单，他们应该执行的唯一工作是分配或释放资源。这个实现有两点需要注意。 首先，```start()```方法还没有处理从先前的偏移量恢复，这将在后面的章节中解决。 其次，```stop()```方法是同步的。 这是必要的，因为``SourceTasks``有一个专用的线程，它们可以无限期地阻塞，所以它们需要被来自Worker中不同线程的调用停止。

Next, we implement the main functionality of the task, the ```poll()``` method which gets events from the input system and returns a ```List<SourceRecord>```:

接下来，我们实现了任务的主要功能，```poll()```方法从输入系统获取事件并返回一个```List <SourceRecord>```：

```java
@Override
public List<SourceRecord> poll() throws InterruptedException {
    try {
        ArrayList<SourceRecord> records = new ArrayList<>();
        while (streamValid(stream) && records.isEmpty()) {
            LineAndOffset line = readToNextLine(stream);
            if (line != null) {
                Map<String, Object> sourcePartition = Collections.singletonMap("filename", filename);
                Map<String, Object> sourceOffset = Collections.singletonMap("position", streamOffset);
                records.add(new SourceRecord(sourcePartition, sourceOffset, topic, Schema.STRING_SCHEMA, line));
            } else {
                Thread.sleep(1);
            }
        }
        return records;
    } catch (IOException e) {
        // Underlying stream was killed, probably as a result of calling stop. Allow to return
        // null, and driving thread will handle any shutdown if necessary.
    }
    return null;
}
```

Again, we've omitted some details, but we can see the important steps: the ```poll()``` method is going to be called repeatedly, and for each call it will loop trying to read records from the file. For each line it reads, it also tracks the file offset. It uses this information to create an output ```SourceRecord``` with four pieces of information: the source partition (there is only one, the single file being read), source offset (byte offset in the file), output topic name, and output value (the line, and we include a schema indicating this value will always be a string). Other variants of the ```SourceRecord``` constructor can also include a specific output partition, a key, and headers.

同样的，我们已经省略了一些细节，但我们可以看到重要的步骤：```poll()```方法将被重复调用，并且对于每个调用，它将循环尝试从文件中读取记录。 对于它读取的每一行，它也会跟踪文件偏移量。 它使用这些信息创建一个带有四个信息的输出```SourceRecord```：源分区（只有一个，单个文件正在被读取），源偏移量（文件中的字节偏移量），输出主题名称和输出值（ 行，并且我们包含一个表明这个值总是一个字符串的模式）。```SourceRecord```构造函数的其他变体还可以包含特定的输出分区（partion），key和header(头部)。

Note that this implementation uses the normal Java ```InputStream``` interface and may sleep if data is not available. This is acceptable because Kafka Connect provides each task with a dedicated thread. While task implementations have to conform to the basic ```poll()``` interface, they have a lot of flexibility in how they are implemented. In this case, an NIO-based implementation would be more efficient, but this simple approach works, is quick to implement, and is compatible with older versions of Java.

请注意，该实现使用普通的Java```InputStream```接口，如果数据未到达，则会一直等待。这是可以接受的，因为Kafka Connect为每项任务提供专用线程。虽然任务实现必须符合基本的```poll()```接口，但它们在如何实现方面有很大的灵活性。 在这种情况下，基于NIO的实现将更加高效，但这种简单的方法很有效，实现起来很快，并且与旧版本的Java兼容。

#### Sink Tasks

The previous section described how to implement a simple ```SourceTask```. Unlike ```SourceConnector``` and ```SinkConnector```, ```SourceTask``` and ```SinkTask``` have very different interfaces because ```SourceTask``` uses a pull interface and ```SinkTask``` uses a push interface. Both share the common lifecycle methods, but the ```SinkTask``` interface is quite different:

上一节描述了如何实现一个简单的```SourceTask```。 与```SourceConnector```和```SinkConnector```不同，```SourceTask```和```SinkTask```具有非常不同的接口，因为```SourceTask```使用一个pull接口而```SinkTask```使用push接口。两者都有共同的生命周期方法，但```SinkTask```接口有很大不同：

```java
public abstract class SinkTask implements Task {
    public void initialize(SinkTaskContext context) {
        this.context = context;
    }
 
    public abstract void put(Collection<SinkRecord> records);
 
    public void flush(Map<TopicPartition, OffsetAndMetadata> currentOffsets) {
}
```

The SinkTask documentation contains full details, but this interface is nearly as simple as the ```SourceTask```. The ```put()``` method should contain most of the implementation, accepting sets of ```SinkRecords```, performing any required translation, and storing them in the destination system. This method does not need to ensure the data has been fully written to the destination system before returning. In fact, in many cases internal buffering will be useful so an entire batch of records can be sent at once, reducing the overhead of inserting events into the downstream data store. The ```SinkRecords``` contain essentially the same information as ```SourceRecords```: Kafka topic, partition, offset, the event key and value, and optional headers.

SinkTask文档包含完整的细节，但该接口几乎与```SourceTask```一样简单。```put()```方法应包含大部分实现，接收```SinkRecords```集合，执行任何必需的转换，并将它们存储在目标系统中。此方法无需确保数据在返回之前已完全写入目标系统。事实上，在很多情况下，内部缓冲将很有用，因此可以一次发送一大批记录，从而减少将事件插入下游数据存储的开销。 ```SinkRecords```包含与```SourceRecords```基本相同的信息：Kafka主题、分区、偏移量、事件的可以和value、可选标题。

The flush() method is used during the offset commit process, which allows tasks to recover from failures and resume from a safe point such that no events will be missed. The method should push any outstanding data to the destination system and then block until the write has been acknowledged. The ```offsets``` parameter can often be ignored, but is useful in some cases where implementations want to store offset information in the destination store to provide exactly-once delivery. For example, an HDFS connector could do this and use atomic move operations to make sure the ```flush()``` operation atomically commits the data and offsets to a final location in HDFS.

flush() 方法在偏移提交过程中使用，它允许任务从失败中恢复并从安全点恢复，从而不会丢失任何事件。该方法应该将任何未完成的数据推送到目标系统，然后阻塞，直到写入已被确认。```offsets```参数通常可以忽略，但在某些情况下，实现需要在目标存储中存储偏移量信息以提供精确的一次传送。 例如，一个HDFS connector可以做到这一点，并使用原子移动操作来确保```flush()```操作自动将数据和偏移量提交到HDFS中的最终位置。

#### Resuming from Previous Offsets

#### 从先前的偏移恢复

The SourceTask implementation included a stream ID (the input filename) and offset (position in the file) with each record. The framework uses this to commit offsets periodically so that in the case of a failure, the task can recover and minimize the number of events that are reprocessed and possibly duplicated (or to resume from the most recent offset if Kafka Connect was stopped gracefully, e.g. in standalone mode or due to a job reconfiguration). This commit process is completely automated by the framework, but only the connector knows how to seek back to the right position in the input stream to resume from that location.

SourceTask实现包括每个记录的流ID(输入文件名)和偏移量(在文件中的位置)。该框架使用它来定期提交偏移量，以便在出现故障的情况下，该任务可以恢复并最小化重新处理并可能重复的事件数量(或者，如果Kafka Connect正常停止，则从最近的偏移量恢复，例如 在独立模式下或由于作业重新配置)。 这个提交过程由框架完全自动化，但只有connector知道如何找回输入流中的正确位置才能从该位置恢复。

To correctly resume upon startup, the task can use the ```SourceContext``` passed into its ```initialize()``` method to access the offset data. In ```initialize()```, we would add a bit more code to read the offset (if it exists) and seek to that position:

为了在启动时正确恢复，任务可以使用```SourceContext```传递到它的```initialize()```方法来访问偏移量数据。 在```initialize()```中，我们会添加更多的代码来读取偏移量(如果存在的话)并寻找该位置：

```java
stream = new FileInputStream(filename);
Map<String, Object> offset = context.offsetStorageReader().offset(Collections.singletonMap(FILENAME_FIELD, filename));
if (offset != null) {
    Long lastRecordedOffset = (Long) offset.get("position");
    if (lastRecordedOffset != null)
        seekToOffset(stream, lastRecordedOffset);
}
```

Of course, you might need to read many keys for each of the input streams. The ```OffsetStorageReader``` interface also allows you to issue bulk reads to efficiently load all offsets, then apply them by seeking each input stream to the appropriate position.

当然，您可能需要为每个输入流读取多个key。```OffsetStorageReader```接口还允许您发出批量读取以有效加载所有偏移量，然后通过将每个输入流应用到适当的位置来应用它们。

### Dynamic Input/Output Streams

### 动态的输入/输出流

Kafka Connect is intended to define bulk data copying jobs, such as copying an entire database rather than creating many jobs to copy each table individually. One consequence of this design is that the set of input or output streams for a connector can vary over time.

Kafka Connect旨在定义批量数据复制作业，例如复制整个数据库，而不是创建许多作业以分别复制每个表。这种设计的一个结果是，connector的一组输入或输出流可随时间变化。

Source connectors need to monitor the source system for changes, e.g. table additions/deletions in a database. When they pick up changes, they should notify the framework via the ```ConnectorContext``` object that reconfiguration is necessary. For example, in a ```SourceConnector```:

soruce connector 需要监视源系统的变化，例如 数据库中的表增加/删除。当他们选择更改时，他们应通过```ConnectorContext```对象通知框架需要重新配置。例如，在一个```SourceConnector```中：

```java
if (inputsChanged())
    this.context.requestTaskReconfiguration();
```

The framework will promptly request new configuration information and update the tasks, allowing them to gracefully commit their progress before reconfiguring them. Note that in the ```SourceConnector``` this monitoring is currently left up to the connector implementation. If an extra thread is required to perform this monitoring, the connector must allocate it itself.

该框架将及时请求新的配置信息并更新任务，使他们能够在重新配置之前正常地提交进度。 请注意，在```SourceConnector```中，此监视当前由connector实现决定。 如果执行此监视需要额外的线程，则connector必须自行分配它。

Ideally this code for monitoring changes would be isolated to the ```Connector``` and tasks would not need to worry about them. However, changes can also affect tasks, most commonly when one of their input streams is destroyed in the input system, e.g. if a table is dropped from a database. If the ```Task``` encounters the issue before the ```Connector```, which will be common if the ```Connector``` needs to poll for changes, the ```Task``` will need to handle the subsequent error. Thankfully, this can usually be handled simply by catching and handling the appropriate exception.

理想情况下，这个用于监控变化的代码将被隔离到```Connector```上，任务不需要担心。然而，改变也会影响任务，最常见的是当其输入流之一在输入系统中被破坏时，例如，如果一个表从数据库中删除。 如果```Task```在```Connector```之前遇到问题，如果```Connector```需要轮询修改，那么```Task```将需要处理后续错误。但这通常可以通过捕捉和处理适当的例外来处理。

SinkConnectors usually only have to handle the addition of streams, which may translate to new entries in their outputs (e.g., a new database table). The framework manages any changes to the Kafka input, such as when the set of input topics changes because of a regex subscription. ```SinkTasks``` should expect new input streams, which may require creating new resources in the downstream system, such as a new table in a database. The trickiest situation to handle in these cases may be conflicts between multiple ```SinkTasks``` seeing a new input stream for the first time and simultaneously trying to create the new resource. ```SinkConnectors```, on the other hand, will generally require no special code for handling a dynamic set of streams.

SinkConnectors 通常只需要处理流的添加，这可能会转化为输出中的新条目（例如新的数据库表）。 该框架管理对Kafka输入的任何更改，例如，由于正则表达式订阅而导致输入主题集发生更改时。```SinkTasks```应该期待新的输入流，这可能需要在下游系统中创建新的资源，比如数据库中的新表。 在这些情况下处理最棘手的情况可能是多个```SinkTasks```第一次看到一个新输入流并同时尝试创建新资源。 另一方面，```SinkConnectors```通常不需要特殊的代码来处理一组动态的流。

### Connect Configuration Validation

### Connect 配置验证

Kafka Connect allows you to validate connector configurations before submitting a connector to be executed and can provide feedback about errors and recommended values. To take advantage of this, connector developers need to provide an implementation of ```config()``` to expose the configuration definition to the framework.

Kafka Connect允许您在提交要执行的connector之前验证connector配置，并可提供有关错误和建议值的反馈。为了利用这一点，connector开发人员需要提供```config()```的实现来将配置定义公开给框架。

The following code in ```FileStreamSourceConnector``` defines the configuration and exposes it to the framework.

在```FileStreamSourceConnector```中的以下代码定义了配置并将其公开给框架。

```java
private static final ConfigDef CONFIG_DEF = new ConfigDef()
    .define(FILE_CONFIG, Type.STRING, Importance.HIGH, "Source filename.")
    .define(TOPIC_CONFIG, Type.STRING, Importance.HIGH, "The topic to publish data to");
 
public ConfigDef config() {
    return CONFIG_DEF;
}
```

ConfigDef class is used for specifying the set of expected configurations. For each configuration, you can specify the name, the type, the default value, the documentation, the group information, the order in the group, the width of the configuration value and the name suitable for display in the UI. Plus, you can provide special validation logic used for single configuration validation by overriding the Validator class. Moreover, as there may be dependencies between configurations, for example, the valid values and visibility of a configuration may change according to the values of other configurations. To handle this, ```ConfigDef``` allows you to specify the dependents of a configuration and to provide an implementation of Recommender to get valid values and set visibility of a configuration given the current configuration values.

ConfigDef类用于指定一组预期的配置。 对于每个配置，您可以指定名称，类型，默认值，文档，组信息，组中的顺序，配置值的宽度以及适合在UI中显示的名称。 另外，您可以通过覆盖Validator类来提供用于单个配置验证的特殊验证逻辑。 此外，由于配置之间可能存在依赖关系，例如，配置的有效值和可见性可能会根据其他配置的值而改变。 为了处理这个问题，可以使用```ConfigDef```来指定配置的依赖项，并提供Recommender的实现来获取有效值，并在给定当前配置值的情况下设置配置的可见性。

Also, the ```validate()``` method in ```Connector``` provides a default validation implementation which returns a list of allowed configurations together with configuration errors and recommended values for each configuration. However, it does not use the recommended values for configuration validation. You may provide an override of the default implementation for customized configuration validation, which may use the recommended values.

并且，```Connector```中的```validate()```方法提供了一个默认的验证实现，它返回一个允许的配置列表，以及每个配置的配置错误和推荐值。 但是，它不使用建议的值进行配置验证。 您可以提供自定义配置验证的默认实现覆盖，该自定义配置验证可能使用推荐值。

### Working with Schemas

### 使用Schema类

The FileStream connectors are good examples because they are simple, but they also have trivially structured data -- each line is just a string. Almost all practical connectors will need schemas with more complex data formats.

FileStream connector是很好的例子，因为它们很简单，但它们也具有简单的结构化数据 - 每行只是一个字符串。几乎所有的实用的connector都需要具有更复杂数据格式的模式。

To create more complex data, you'll need to work with the Kafka Connect ```data``` API. Most structured records will need to interact with two classes in addition to primitive types: ```Schema``` and ```Struct```.

要创建更复杂的数据，您需要使用Kafka Connect的```data```API。大多数结构化记录除了基本类型外，还需要与两个类进行交互：```Schema```和```Struct```。

The API documentation provides a complete reference, but here is a simple example creating a ```Schema``` and ```Struct```:

API文档提供了一个完整的参考，但是这里有一个简单的例子来创建```Schema```和```Struct```。

```java
Schema schema = SchemaBuilder.struct().name(NAME)
    .field("name", Schema.STRING_SCHEMA)
    .field("age", Schema.INT_SCHEMA)
    .field("admin", new SchemaBuilder.boolean().defaultValue(false).build())
    .build();
 
Struct struct = new Struct(schema)
    .put("name", "Barbara Liskov")
    .put("age", 75);
```

If you are implementing a source connector, you'll need to decide when and how to create schemas. Where possible, you should avoid recomputing them as much as possible. For example, if your connector is guaranteed to have a fixed schema, create it statically and reuse a single instance.

如果您正在实现source connector，则需要确定何时以及如何创建模式。在可能的情况下，应尽可能避免重新计算它们。例如，如果您的connector保证有固定模式，请静态创建并重用单个实例。

However, many connectors will have dynamic schemas. One simple example of this is a database connector. Considering even just a single table, the schema will not be predefined for the entire connector (as it varies from table to table). But it also may not be fixed for a single table over the lifetime of the connector since the user may execute an ```ALTER TABLE``` command. The connector must be able to detect these changes and react appropriately.

但是，许多connector将具有动态模式。一个简单的例子就是数据库connector。考虑到即使只有一个表，schema也不会为整个connector预定义(因为它随表到表而变化)。但是在connector的生命周期中，它也可能不是固定的，因为用户可以执行```ALTER TABLE```命令。connector必须能够检测到这些变化并做出适当的反应。

Sink connectors are usually simpler because they are consuming data and therefore do not need to create schemas. However, they should take just as much care to validate that the schemas they receive have the expected format. When the schema does not match -- usually indicating the upstream producer is generating invalid data that cannot be correctly translated to the destination system -- sink connectors should throw an exception to indicate this error to the system.

Sink Connector通常更简单，因为它们正在消耗数据，因此不需要创建模式。但是，他们应该同样仔细地验证他们收到的模式具有预期的格式。当schema不匹配时 - 通常指示上游生产者正在生成无法正确转换到目标系统的无效数据 - sink connector应抛出异常以向系统指示此错误。

### Kafka Connect Administration

### Kafka Connect 管理

Kafka Connect's [REST layer](#rest-api) provides a set of APIs to enable administration of the cluster. This includes APIs to view the configuration of connectors and the status of their tasks, as well as to alter their current behavior (e.g. changing configuration and restarting tasks).

Kafka Connect的[REST层](#rest-api)提供了一组API来启用群集管理。这包括查看connector配置和任务状态的API，以及改变其当前行为（例如更改配置和重新启动任务）的API。

When a connector is first submitted to the cluster, the workers rebalance the full set of connectors in the cluster and their tasks so that each worker has approximately the same amount of work. This same rebalancing procedure is also used when connectors increase or decrease the number of tasks they require, or when a connector's configuration is changed. You can use the REST API to view the current status of a connector and its tasks, including the id of the worker to which each was assigned. For example, querying the status of a file source (using ```GET /connectors/file-source/status```) might produce output like the following:

当connector首次提交到群集时，工作人员将重新平衡群集中的全部connector及其任务，以便每个工作线程的工作量大致相同。当connector增加或减少它们需要的任务数量或connector的配置发生变化时，也使用相同的重新平衡过程。您可以使用REST API来查看connector及其任务的当前状态，包括每个connector分配的工作者的ID。 例如，查询文件源的状态（使用```GET /connectors/file-source/status```）可能会产生如下输出：

```json
{
"name": "file-source",
"connector": {
    "state": "RUNNING",
    "worker_id": "192.168.1.208:8083"
},
"tasks": [
    {
    "id": 0,
    "state": "RUNNING",
    "worker_id": "192.168.1.209:8083"
    }
]
}
```

Connectors and their tasks publish status updates to a shared topic (configured with ```status.storage.topic```) which all workers in the cluster monitor. Because the workers consume this topic asynchronously, there is typically a (short) delay before a state change is visible through the status API. The following states are possible for a connector or one of its tasks:

Connector及其任务将状态更新发布到群集中所有工作人员监视的共享主题(使用```status.storage.topic```配置)。由于工作人员异步使用此主题，因此状态更改通过状态API可见之前通常会有(短)延迟。以下状态可用于connector或其中的一个任务：

* **UNASSIGNED**: The connector/task has not yet been assigned to a worker.
* **RUNNING**: The connector/task is running.
* **PAUSED**: The connector/task has been administratively paused.
* **FAILED**: The connector/task has failed (usually by raising an exception, which is reported in the status output).


* **UNASSIGNED**: connector/task 还没有分派到工作线程中
* **RUNNING**: connector/task 正在运行
* **PAUSED**: connector/task 被暂停了
* **FAILED**: connector/task 运行失败(通常在状态输出中抛出一个异常)

In most cases, connector and task states will match, though they may be different for short periods of time when changes are occurring or if tasks have failed. For example, when a connector is first started, there may be a noticeable delay before the connector and its tasks have all transitioned to the RUNNING state. States will also diverge when tasks fail since Connect does not automatically restart failed tasks. To restart a connector/task manually, you can use the restart APIs listed above. Note that if you try to restart a task while a rebalance is taking place, Connect will return a 409 (Conflict) status code. You can retry after the rebalance completes, but it might not be necessary since rebalances effectively restart all the connectors and tasks in the cluster.

在大多数情况下，connector和任务状态都会匹配，但在发生更改或任务失败时，connector和任务状态在短时间内可能会有所不同。 例如，首次启动connector时，connector及其任务全部转换为RUNNING状态之前可能会有明显的延迟。由于Connect不会自动重启失败的任务，因此任务失败时状态也会发生变化。要手动重新启动connector/任务，可以使用上面列出的重新启动API。请注意，如果尝试在重新调整平衡时重新启动任务，connect将返回409(冲突)状态码。您可以在重新调整平衡完成后重试，但可能没有必要，因为重新调整平衡后会重新启动群集中的所有connector和任务。

It's sometimes useful to temporarily stop the message processing of a connector. For example, if the remote system is undergoing maintenance, it would be preferable for source connectors to stop polling it for new data instead of filling logs with exception spam. For this use case, Connect offers a pause/resume API. While a source connector is paused, Connect will stop polling it for additional records. While a sink connector is paused, Connect will stop pushing new messages to it. The pause state is persistent, so even if you restart the cluster, the connector will not begin message processing again until the task has been resumed. Note that there may be a delay before all of a connector's tasks have transitioned to the PAUSED state since it may take time for them to finish whatever processing they were in the middle of when being paused. Additionally, failed tasks will not transition to the PAUSED state until they have been restarted.

暂时停止connector的消息处理有时很有用。例如，如果远程系统正在进行维护，则source connector最好停止轮询它以获取新数据，而不是使用例外垃圾邮件填充日志。对于这个用例，Connect提供了一个暂停/恢复API。source connector暂停时，Connect将停止轮询它以获取其他记录。当sink connector暂停时，Connect将停止向其发送新消息。暂停状态是持久的，因此即使重新启动群集，connector也不会再次开始消息处理，直到任务恢复。请注意，在所有connector的任务已转换到PAUSED状态之前可能会有延迟，因为它们可能需要一些时间才能完成暂停时的中间任何处理。另外，失败的任务在重新启动之前不会转换到PAUSED状态。
