 # Tutorial: Write a Kafka Streams Application

 # 教程：编写一个Kafka Streams应用程序
 
 In this guide we will start from scratch on setting up your own project to write a stream processing application using Kafka Streams. It is highly recommended to read the [quickstart](run_demo_app.md) first on how to run a Streams application written in Kafka Streams if you have not done so.

 在本指南中，我们将从头开始设置您自己的项目，并使用Kafka Streams编写流处理应用程序。强烈建议您首先阅读[快速入门指南](run_demo_app.md)，了解如何运行使用Kafka Streams编写的流应用程序（如果您还尚未这样做过）。

## Setting up a Maven Project

## 建立一个Maven项目

We are going to use a Kafka Streams Maven Archetype for creating a Streams project structure with the following commands:

我们将使用Kafka Streams的Maven原型来创建流处理项目结构，其中包含以下命令：

```bash
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.kafka \
    -DarchetypeArtifactId=streams-quickstart-java \
    -DarchetypeVersion=1.1.0 \
    -DgroupId=streams.examples \
    -DartifactId=streams.examples \
    -Dversion=0.1 \
    -Dpackage=myapps
```

You can use a different value for ```groupId```, ```artifactId``` and ```package``` parameters if you like. Assuming the above parameter values are used, this command will create a project structure that looks like this:

如果您愿意，您可以为```groupId```，```artifactId```和```package```参数设置不同的值。假设使用上述参数值，该命令将创建一个如下所示的项目结构：

```bash
> tree streams.examples
streams-quickstart
|-- pom.xml
|-- src
    |-- main
        |-- java
        |   |-- myapps
        |       |-- LineSplit.java
        |       |-- Pipe.java
        |       |-- WordCount.java
        |-- resources
            |-- log4j.properties
```

The ```pom.xml``` file included in the project already has the Streams dependency defined, and there are already several example programs written with Streams library under ```src/main/java```. Since we are going to start writing such programs from scratch, we can now delete these examples:

项目中包含的```pom.xml```文件已经定义了Streams依赖项，并且已经有几个使用```src/main/java```下的Streams库编写的示例程序。由于我们要从头开始编写这样的程序，现在我们可以删除这些例子：

```bash
> cd streams-quickstart
> rm src/main/java/myapps/*.java
```

## Writing a first Streams application: Pipe

## 编写第一个Streams应用程序：管道

It's coding time now! Feel free to open your favorite IDE and import this Maven project, or simply open a text editor and create a java file under ```src/main/java```. Let's name it ```Pipe.java```:

现在开始编码！打开您最喜欢的IDE并导入这个Maven项目，或者直接打开一个文本编辑器并在```src/main/java```下创建一个java文件。我们将其命名为```Pipe.java```：

```Java
package myapps;
 
public class Pipe {
 
    public static void main(String[] args) throws Exception {
 
    }
}
```

We are going to fill in the ```main``` function to write this pipe program. Note that we will not list the import statements as we go since IDEs can usually add them automatically. However if you are using a text editor you need to manually add the imports, and at the end of this section we'll show the complete code snippet with import statement for you.

我们将在```main```方法中编写这个管道程序。请注意，由于IDE通常可以自动添加导入语句，因此我们不会列出导入语句。但是，如果您使用的是文本编辑器，则需要手动添加导入语句，在本节末尾，我们将为您显示带有import语句的完整代码段。

The first step to write a Streams application is to create a ```java.util.Properties``` map to specify different Streams execution configuration values as defined in ```StreamsConfig```. A couple of important configuration values you need to set are: ```StreamsConfig.BOOTSTRAP_SERVERS_CONFIG```, which specifies a list of host/port pairs to use for establishing the initial connection to the Kafka cluster, and ```StreamsConfig.APPLICATION_ID_CONFIG```, which gives the unique identifier of your Streams application to distinguish itself with other applications talking to the same Kafka cluster:

编写Streams应用程序的第一步是创建一个```java.util.Properties```映射来指定```StreamsConfig```中定义的不同Streams执行配置值。您需要设置的几个重要配置值是：```StreamsConfig.BOOTSTRAP_SERVERS_CONFIG```，它指定用于建立初始连接到Kafka集群的主机/端口对列表，以及```StreamsConfig.APPLICATION_ID_CONFIG```，它提供了Streams应用程序与其他应用程序进行区分的唯一标识符，用于与同一个Kafka集群进行数据交流：

```Java
Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-pipe");
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");    // assuming that the Kafka broker this application is talking to runs on local machine with port 9092
// 假设正在与该应用程序对话的Kafka代理在端口为9092的本地机器上运行
```

In addition, you can customize other configurations in the same map, for example, default serialization and deserialization libraries for the record key-value pairs:

另外，您可以在同一个映射中自定义其他配置，例如消息键值对的默认序列化和反序列化库：

```Java
props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
```

For a full list of configurations of Kafka Streams please refer to this table.

有关Kafka Streams的完整配置列表，请参阅此表。

Next we will define the computational logic of our Streams application. In Kafka Streams this computational logic is defined as a ```topology``` of connected processor nodes. We can use a topology builder to construct such a topology,
	
接下来我们将定义Streams应用程序的计算逻辑。在Kafka Streams中，这种计算逻辑被定义为连接处理器节点的```拓扑结构```。我们可以使用拓扑构建器来构建这样的拓扑，

```Java
final StreamsBuilder builder = new StreamsBuilder();
```

And then create a source stream from a Kafka topic named ```streams-plaintext-input``` using this topology builder:

然后使用此拓扑构建器从名为```streams-plaintext-input```的Kafka主题创建源stream：
	
```Java
KStream<String, String> source = builder.stream("streams-plaintext-input");
```

Now we get a ```KStream``` that is continuously generating records from its source Kafka topic ```streams-plaintext-input```. The records are organized as ```String``` typed key-value pairs. The simplest thing we can do with this stream is to write it into another Kafka topic, say it's named ```streams-pipe-output```:

现在我们得到一个```KStream```，它不断从源Kafka主题```streams-plaintext-input```中生成消息。消息按```字符串```类型的键值对来组织。我们可以用这个流做的最简单的事情就是将它写入另一个Kafka主题，比如被命名为```streams-pipe-output```的主题：

```Java
source.to("streams-pipe-output");
```

Note that we can also concatenate the above two lines into a single line as:

请注意，我们也可以将上面两行连接成一行，如下所示：
	
```Java
builder.stream("streams-plaintext-input").to("streams-pipe-output");
```

We can inspect what kind of ```topology``` is created from this builder by doing the following:
	
我们可以通过执行以下操作来检查此构建器创建的```拓扑结构```的类型：

```Java
final Topology topology = builder.build();
```

And print its description to standard output as:

并将其打印到标准输出中，如下：

```Java	
System.out.println(topology.describe());
```

If we just stop here, compile and run the program, it will output the following information:

如果我们停在这里，编译并运行程序，它会输出以下信息：
	
```bash
> mvn clean package
> mvn exec:java -Dexec.mainClass=myapps.Pipe
Sub-topologies:
  Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000(topics: streams-plaintext-input) --> KSTREAM-SINK-0000000001
    Sink: KSTREAM-SINK-0000000001(topic: streams-pipe-output) <-- KSTREAM-SOURCE-0000000000
Global Stores:
  none
```

As shown above, it illustrates that the constructed topology has two processor nodes, a source node ```KSTREAM-SOURCE-0000000000``` and a sink node ```KSTREAM-SINK-0000000001```. ```KSTREAM-SOURCE-0000000000``` continuously read records from Kafka topic ```streams-plaintext-input``` and pipe them to its downstream node ```KSTREAM-SINK-0000000001```; ```KSTREAM-SINK-0000000001``` will write each of its received record in order to another Kafka topic ```streams-pipe-output``` (the ```-->``` and ```<--``` arrows dictates the downstream and upstream processor nodes of this node, i.e. "children" and "parents" within the topology graph). It also illustrates that this simple topology has no global state stores associated with it (we will talk about state stores more in the following sections).

如上所示，它说明构建的拓扑具有两个处理器节点，源节点```KSTREAM-SOURCE-0000000000```和汇聚节点```KSTREAM-SINK-0000000001```。```KSTREAM-SOURCE-0000000000```连续读取来自Kafka主题```streams-plaintext-input```的消息并将它们传送到其下游节点```KSTREAM-SINK-0000000001```; ```KSTREAM-SINK-0000000001```会将其接收到的每条消息写入另一个Kafka主题```streams-pipe-output```（```-->```和```<--```箭头指示该节点的下游和上游处理器节点，即在拓扑图中的“子节点”和“父节点”）。它还说明，这种简单的拓扑没有与之相关联的全局状态存储（我们将在后面的章节中更多地讨论状态存储）。

Note that we can always describe the topology as we did above at any given point while we are building it in the code, so as a user you can interactively "try and taste" your computational logic defined in the topology until you are happy with it. Suppose we are already done with this simple topology that just pipes data from one Kafka topic to another in an endless streaming manner, we can now construct the Streams client with the two components we have just constructed above: the configuration map and the topology object (one can also construct a ```StreamsConfig``` object from the ```props``` map and then pass that object to the constructor, ```KafkaStreams``` have overloaded constructor functions to takes either type).
	
请注意，当我们在代码中构建拓扑结构的时候，总是可以像上面那样在任何给定的点上描述它。因此作为用户，您可以交互式地“尝试并品尝”拓扑中定义的计算逻辑，直到您满意为止。假设我们已经完成了这个只以一种无尽的流式方式将数据从一个Kafka主题通过管道传输到另一个主题的简单的拓扑结构，我们现在就可以使用我们刚刚构建的两个组件：配置映射和拓扑对象来构建Streams客户端（也可以从```props```映射构造一个```StreamsConfig```对象，然后将该对象传递给构造函数，```KafkaStreams```已经重载了构造函数以接受其中的任一类型）。

```Java
final KafkaStreams streams = new KafkaStreams(topology, props);
```

By calling its ```start()``` function we can trigger the execution of this client. The execution won't stop until ```close()``` is called on this client. We can, for example, add a shutdown hook with a countdown latch to capture a user interrupt and close the client upon terminating this program:

通过调用它的```start()```函数，我们可以触发这个客户端的执行。在此客户端被调用```close()```之前，执行不会停止。例如，我们可以添加一个带有CountDownLatch的关闭钩子来捕获用户中断，并在终止该程序时关闭客户端：

```Java
final CountDownLatch latch = new CountDownLatch(1);
 
// attach shutdown handler to catch control-c
// 附加关闭处理程序来捕获control-c
Runtime.getRuntime().addShutdownHook(new Thread("streams-shutdown-hook") {
    @Override
    public void run() {
        streams.close();
        latch.countDown();
    }
});
 
try {
    streams.start();
    latch.await();
} catch (Throwable e) {
    System.exit(1);
}
System.exit(0);
```

The complete code so far looks like this:

到目前为止，完整的代码如下所示：

```Java	
package myapps;
 
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
 
import java.util.Properties;
import java.util.concurrent.CountDownLatch;
 
public class Pipe {
 
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-pipe");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
 
        final StreamsBuilder builder = new StreamsBuilder();
 
        builder.stream("streams-plaintext-input").to("streams-pipe-output");
 
        final Topology topology = builder.build();
 
        final KafkaStreams streams = new KafkaStreams(topology, props);
        final CountDownLatch latch = new CountDownLatch(1);
 
        // attach shutdown handler to catch control-c
        Runtime.getRuntime().addShutdownHook(new Thread("streams-shutdown-hook") {
            @Override
            public void run() {
                streams.close();
                latch.countDown();
            }
        });
 
        try {
            streams.start();
            latch.await();
        } catch (Throwable e) {
            System.exit(1);
        }
        System.exit(0);
    }
}
```

If you already have the Kafka broker up and running at ```localhost:9092```, and the topics ```streams-plaintext-input``` and ```streams-pipe-output``` created on that broker, you can run this code in your IDE or on the command line, using Maven:

如果您已经在```localhost:9092```上启动并运行了Kafka代理，并且在该代理上创建了主题```streams-plaintext-input```和```streams-pipe-output```，则可以在IDE或命令行上使用Maven运行此代码：

```bash
        > mvn clean package
        > mvn exec:java -Dexec.mainClass=myapps.Pipe
``` 

For detailed instructions on how to run a Streams application and observe its computing results, please read the [Play with a Streams Application](run_demo_app.md) section. We will not talk about this in the rest of this section.

有关如何运行Streams应用程序并观察计算结果的详细说明，请阅读[Play with a Streams应用程序](run_demo_app.md)部分。本节的其余部分我们不会谈论这一点。

## Writing a second Streams application: Line Split

## 编写第二个Streams应用程序：行分割

We have learned how to construct a Streams client with its two key components: the ```StreamsConfig``` and ```Topology```. Now let's move on to add some real processing logic by augmenting the current topology. We can first create another program by first copy the existing ```Pipe.java``` class:

我们已经学会了如何通过两个关键组件：```StreamsConfig```和```Topology```构建Streams客户端。现在让我们继续通过增加当前拓扑来添加一些实际的处理逻辑。我们可以先复制现有的```Pipe.java```类来创建另一个程序：

```bash
        > cp src/main/java/myapps/Pipe.java src/main/java/myapps/LineSplit.java
``` 

And change its class name as well as the application id config to distinguish with the original program:

并更改其类名以及应用程序ID配置以与原始程序区分开来：

```Java
public class LineSplit {
 
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-linesplit");
        // ...
    }
}
```

Since each of the source stream's record is a ```String``` typed key-value pair, let's treat the value string as a text line and split it into words with a ```FlatMapValues``` operator:

由于每个源stream的消息都是一个```String```类型的键值对，因此让我们将值字符串视为文本行，并使用```FlatMapValues```运算符将其分成单词：

```Java	
KStream<String, String> source = builder.stream("streams-plaintext-input");
KStream<String, String> words = source.flatMapValues(new ValueMapper<String, Iterable<String>>() {
            @Override
            public Iterable<String> apply(String value) {
                return Arrays.asList(value.split("\\W+"));
            }
        });
```

The operator will take the ```source``` stream as its input, and generate a new stream named ```words``` by processing each record from its source stream in order and breaking its value string into a list of words, and producing each word as a new record to the output ```words``` stream. This is a stateless operator that does not need to keep track of any previously received records or processed results. Note if you are using JDK 8 you can use lambda expression and simplify the above code as:

该运算符将把```源```stream作为输入，并通过按顺序处理源stream中的每条消息，将其值字符串分解为一个单词列表，生成以每个单词作为输出```单词```的新消息，从而生成一个名为```words```的新stream。这是一个无需跟踪以前收到的任何消息或处理结果的无状态运算符。请注意，如果您使用的是JDK 8，则可以使用lambda表达式来简化上面的代码：
	
```Java
KStream<String, String> source = builder.stream("streams-plaintext-input");
KStream<String, String> words = source.flatMapValues(value -> Arrays.asList(value.split("\\W+")));
```

And finally we can write the word stream back into another Kafka topic, say ```streams-linesplit-output```. Again, these two steps can be concatenated as the following (assuming lambda expression is used):
	
最后，我们可以将单词流写回另一个Kafka主题，比如说```stream-linesplit-output```。同样的，这两个步骤可以如下所示连接（假设使用lambda表达式）：

```Java
KStream<String, String> source = builder.stream("streams-plaintext-input");
source.flatMapValues(value -> Arrays.asList(value.split("\\W+")))
      .to("streams-linesplit-output");
```

If we now describe this augmented topology as ```System.out.println(topology.describe())```, we will get the following:

如果我们现在将此扩展拓扑描述为```System.out.println(topology.describe())```，我们将得到以下结果：
	
```bash
> mvn clean package
> mvn exec:java -Dexec.mainClass=myapps.LineSplit
Sub-topologies:
  Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000(topics: streams-plaintext-input) --> KSTREAM-FLATMAPVALUES-0000000001
    Processor: KSTREAM-FLATMAPVALUES-0000000001(stores: []) --> KSTREAM-SINK-0000000002 <-- KSTREAM-SOURCE-0000000000
    Sink: KSTREAM-SINK-0000000002(topic: streams-linesplit-output) <-- KSTREAM-FLATMAPVALUES-0000000001
  Global Stores:
    none
```

As we can see above, a new processor node ```KSTREAM-FLATMAPVALUES-0000000001``` is injected into the topology between the original source and sink nodes. It takes the source node as its parent and the sink node as its child. In other words, each record fetched by the source node will first traverse to the newly added ```KSTREAM-FLATMAPVALUES-0000000001``` node to be processed, and one or more new records will be generated as a result. They will continue traverse down to the sink node to be written back to Kafka. Note this processor node is "stateless" as it is not associated with any stores (i.e. ```(stores: [])```).

正如我们上面看到的，一个新的处理器节点```KSTREAM-FLATMAPVALUES-0000000001```被注入到原始源节点和汇聚节点之间的拓扑中。它将源节点作为其父节点，将汇聚节点作为其子节点。换句话说，源节点获取的每个消息将首先遍历新加入的```KSTREAM-FLATMAPVALUES-0000000001```节点并依次得到处理，并且最终将生成一个或多个新消息。他们将继续往下通过汇聚节点回写给Kafka。注意这个处理器节点是“无状态的”，因为它不与任何存储器相关联（即：```(stores: [])```）。

The complete code looks like this (assuming lambda expression is used):

完整的代码如下所示（假设使用lambda表达式）：

```Java
package myapps;
 
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.kstream.KStream;
 
import java.util.Arrays;
import java.util.Properties;
import java.util.concurrent.CountDownLatch;
 
public class LineSplit {
 
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-linesplit");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
 
        final StreamsBuilder builder = new StreamsBuilder();
 
        KStream<String, String> source = builder.stream("streams-plaintext-input");
        source.flatMapValues(value -> Arrays.asList(value.split("\\W+")))
              .to("streams-linesplit-output");
 
        final Topology topology = builder.build();
        final KafkaStreams streams = new KafkaStreams(topology, props);
        final CountDownLatch latch = new CountDownLatch(1);
 
        // ... same as Pipe.java above
        // ...与上面的Pipe.java相同
    }
}
```

## Writing a third Streams application: Wordcount

## 编写第三个Streams应用程序：单词计数

Let's now take a step further to add some "stateful" computations to the topology by counting the occurrence of the words split from the source text stream. Following similar steps let's create another program based on the ```LineSplit.java``` class:

现在让我们进一步通过计算源文本流中分词的出现来向拓扑中添加一些“有状态”的计算。按照类似的步骤，我们创建另一个基于```LineSplit.java```类的程序：

```Java
public class WordCount {
 
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-wordcount");
        // ...
    }
}
```

In order to count the words we can first modify the ```flatMapValues``` operator to treat all of them as lower case (assuming lambda expression is used):

为了计算单词的出现次数，我们首先可以修改```flatMapValues```运算符，将单词全部转换为小写字母（假设使用lambda表达式）：

```Java	
source.flatMapValues(new ValueMapper<String, Iterable<String>>() {
            @Override
            public Iterable<String> apply(String value) {
                return Arrays.asList(value.toLowerCase(Locale.getDefault()).split("\\W+"));
            }
        });
```

In order to do the counting aggregation we have to first specify that we want to key the stream on the value string, i.e. the lower cased word, with a ```groupBy``` operator. This operator generate a new grouped stream, which can then be aggregated by a ```count``` operator, which generates a running count on each of the grouped keys:

为了进行计数聚合，我们必须首先指定我们想要使用```groupBy```运算符来将值字符串上的流（即小写字母）键入。该运算符生成一个新的分组流，然后可以由一个```count```运算符汇总，该运算符在每个分组的键上生成一个运行中的计数值：

```Java	
KTable<String, Long> counts =
source.flatMapValues(new ValueMapper<String, Iterable<String>>() {
            @Override
            public Iterable<String> apply(String value) {
                return Arrays.asList(value.toLowerCase(Locale.getDefault()).split("\\W+"));
            }
        })
      .groupBy(new KeyValueMapper<String, String, String>() {
           @Override
           public String apply(String key, String value) {
               return value;
           }
        })
      // Materialize the result into a KeyValueStore named "counts-store".
      // 将结果物化到名为“counts-store”的KeyValueStore中。
      // The Materialized store is always of type <Bytes, byte[]> as this is the format of the inner most store.
      // 物化存储始终是<Bytes，byte []>类型，因为这是最内层存储的格式。
      .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>> as("counts-store"));
```

Note that the ```count``` operator has a ```Materialized``` parameter that specifies that the running count should be stored in a state store named ```counts-store```. This ```Counts``` store can be queried in real-time, with details described in the Developer Manual.

请注意，```count```运算符具有```Materialized```参数，该参数指定运行计数值应存储在名为```counts-store```的状态存储器中。此```Counts```存储器支持实时查询，详情请参阅开发者手册。

We can also write the ```counts``` KTable's changelog stream back into another Kafka topic, say ```streams-wordcount-output```. Because the result is a changelog stream, the output topic ```streams-wordcount-output``` should be configured with log compaction enabled. Note that this time the value type is no longer ```String``` but ```Long```, so the default serialization classes are not viable for writing it to Kafka anymore. We need to provide overridden serialization methods for ```Long``` types, otherwise a runtime exception will be thrown:

我们还可以将```计数值```KTable的更新日志流写回到另一个Kafka主题中，例如```streams-wordcount-output```。由于结果是更新日志流，因此应该启用日志压缩来配置输出主题```streams-wordcount-output```。请注意，这次值类型不再是```String```而是```Long```，所以默认的序列化类不再可用于将它写入Kafka。我们需要为```Long```类型提供重写的序列化方法，否则将引发runtime exception：

```Java	
counts.toStream().to("streams-wordcount-output", Produced.with(Serdes.String(), Serdes.Long()));
```

Note that in order to read the changelog stream from topic ```streams-wordcount-output```, one needs to set the value deserialization as ```org.apache.kafka.common.serialization.LongDeserializer```. Details of this can be found in the [Play with a Streams Application](run_demo_app.md) section. Assuming lambda expression from JDK 8 can be used, the above code can be simplified as:

请注意，为了从```streams-wordcount-output```主题中读取更新日志流，需要将值反序列化设置为```org.apache.kafka.common.serialization.LongDeserializer```。有关详细信息，请参见[使用Streams应用程序](run_demo_app.md)部分。假设可以使用来自JDK 8的lambda表达式，上面的代码可以简化为：

```Java	
KStream<String, String> source = builder.stream("streams-plaintext-input");
source.flatMapValues(value -> Arrays.asList(value.toLowerCase(Locale.getDefault()).split("\\W+")))
      .groupBy((key, value) -> value)
      .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"))
      .toStream()
      .to("streams-wordcount-output", Produced.with(Serdes.String(), Serdes.Long()));
```

If we again describe this augmented topology as ```System.out.println(topology.describe())```, we will get the following:

如果我们将这种扩展拓扑描述为```System.out.println(topology.describe())```，我们将得到以下结果：

```bash
> mvn clean package
> mvn exec:java -Dexec.mainClass=myapps.WordCount
Sub-topologies:
  Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000(topics: streams-plaintext-input) --> KSTREAM-FLATMAPVALUES-0000000001
    Processor: KSTREAM-FLATMAPVALUES-0000000001(stores: []) --> KSTREAM-KEY-SELECT-0000000002 <-- KSTREAM-SOURCE-0000000000
    Processor: KSTREAM-KEY-SELECT-0000000002(stores: []) --> KSTREAM-FILTER-0000000005 <-- KSTREAM-FLATMAPVALUES-0000000001
    Processor: KSTREAM-FILTER-0000000005(stores: []) --> KSTREAM-SINK-0000000004 <-- KSTREAM-KEY-SELECT-0000000002
    Sink: KSTREAM-SINK-0000000004(topic: Counts-repartition) <-- KSTREAM-FILTER-0000000005
  Sub-topology: 1
    Source: KSTREAM-SOURCE-0000000006(topics: Counts-repartition) --> KSTREAM-AGGREGATE-0000000003
    Processor: KSTREAM-AGGREGATE-0000000003(stores: [Counts]) --> KTABLE-TOSTREAM-0000000007 <-- KSTREAM-SOURCE-0000000006
    Processor: KTABLE-TOSTREAM-0000000007(stores: []) --> KSTREAM-SINK-0000000008 <-- KSTREAM-AGGREGATE-0000000003
    Sink: KSTREAM-SINK-0000000008(topic: streams-wordcount-output) <-- KTABLE-TOSTREAM-0000000007
Global Stores:
  none
```

As we can see above, the topology now contains two disconnected sub-topologies. The first sub-topology's sink node ```KSTREAM-SINK-0000000004``` will write to a repartition topic ```Counts-repartition```, which will be read by the second sub-topology's source node ```KSTREAM-SOURCE-0000000006```. The repartition topic is used to "shuffle" the source stream by its aggregation key, which is in this case the value string. In addition, inside the first sub-topology a stateless ```KSTREAM-FILTER-0000000005``` node is injected between the grouping ```KSTREAM-KEY-SELECT-0000000002``` node and the sink node to filter out any intermediate record whose aggregate key is empty.

如上所述，拓扑结构现在包含两个断开的子拓扑。第一个子拓扑的汇聚节点```KSTREAM-SINK-0000000004```将写入一个重分区主题```Counts-repartition```，它将由第二个子拓扑的源节点```KSTREAM-SOURCE-0000000006```读取。重分区主题用于通过其聚合键“混洗”源流，在这种情况下，聚合键为值字符串。此外，在第一个子拓扑结构内部，在分组```KSTREAM-KEY-SELECT-0000000002```节点和汇聚节点之间注入无状态的```KSTREAM-FILTER-0000000005```节点，以过滤出聚合键为空的任意中间消息。  

In the second sub-topology, the aggregation node ```KSTREAM-AGGREGATE-0000000003``` is associated with a state store named ```Counts``` (the name is specified by the user in the ```count``` operator). Upon receiving each record from its upcoming stream source node, the aggregation processor will first query its associated ```Counts``` store to get the current count for that key, augment by one, and then write the new count back to the store. Each updated count for the key will also be piped downstream to the ```KTABLE-TOSTREAM-0000000007``` node, which interpret this update stream as a record stream before further piping to the sink node ```KSTREAM-SINK-0000000008``` for writing back to Kafka.

在第二个子拓扑中，聚合节点```KSTREAM-AGGREGATE-0000000003```与名为```Counts```的状态存储器（名称由用户在```count```运算符中指定）相关联。在从即将到来的流源节点接收到每个消息时，聚合处理器将首先查询其关联的```Counts```存储器以获得该键的当前计数值，并将其增加1，然后将新计数值写回存储器。每个更新的键计数值将被传送到```KTABLE-TOSTREAM-0000000007```节点，此节点将该更新流解释为消息流，然后再传输到汇聚节点```KSTREAM-SINK-0000000008```以写回Kafka。

The complete code looks like this (assuming lambda expression is used):

完整的代码如下所示（假设使用lambda表达式）：

```Java
package myapps;
 
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.kstream.KStream;
 
import java.util.Arrays;
import java.util.Locale;
import java.util.Properties;
import java.util.concurrent.CountDownLatch;
 
public class WordCount {
 
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-wordcount");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
 
        final StreamsBuilder builder = new StreamsBuilder();
 
        KStream<String, String> source = builder.stream("streams-plaintext-input");
        source.flatMapValues(value -> Arrays.asList(value.toLowerCase(Locale.getDefault()).split("\\W+")))
              .groupBy((key, value) -> value)
              .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"))
              .toStream()
              .to("streams-wordcount-output", Produced.with(Serdes.String(), Serdes.Long());
 
        final Topology topology = builder.build();
        final KafkaStreams streams = new KafkaStreams(topology, props);
        final CountDownLatch latch = new CountDownLatch(1);
 
        // ... same as Pipe.java above
    }
}
```