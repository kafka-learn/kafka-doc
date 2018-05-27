# Documentation

# 文档

## Kafka 1.1 Documentation

## Kafka 1.1 文档

Prior releases: [0.7.x](http://kafka.apache.org/07/documentation.html), [0.8.0](http://kafka.apache.org/08/documentation.html), [0.8.1.X](http://kafka.apache.org/081/documentation.html), [0.8.2.X](http://kafka.apache.org/082/documentation.html), [0.9.0.X](http://kafka.apache.org/090/documentation.html), [0.10.0.X](http://kafka.apache.org/0100/documentation.html), [0.10.1.X](http://kafka.apache.org/0101/documentation.html), [0.10.2.X](http://kafka.apache.org/0102/documentation.html), [0.11.0.X](http://kafka.apache.org/0110/documentation.html), [1.0.X](http://kafka.apache.org/10/documentation.html)

先前的版本：[0.7.x](http://kafka.apache.org/07/documentation.html), [0.8.0](http://kafka.apache.org/08/documentation.html), [0.8.1.X](http://kafka.apache.org/081/documentation.html), [0.8.2.X](http://kafka.apache.org/082/documentation.html), [0.9.0.X](http://kafka.apache.org/090/documentation.html), [0.10.0.X](http://kafka.apache.org/0100/documentation.html), [0.10.1.X](http://kafka.apache.org/0101/documentation.html), [0.10.2.X](http://kafka.apache.org/0102/documentation.html), [0.11.0.X](http://kafka.apache.org/0110/documentation.html), [1.0.X](http://kafka.apache.org/10/documentation.html)

1. GETTING STARTED

    * [1.1 Introduction](./introduction.md)

    * [1.2 Use Cases](./usercases.md)

    * [1.3 Quick Start](./quickstart.md)

    * 1.4 Ecosystem

    * 1.5 Upgrading

2. APIS

    * [2.1 Producer API](./documentations/apis.md#producer-api)

    * [2.2 Consumer API](./documentations/apis.md#consumer-api)

    * [2.3 Streams API](./documentations/apis.md#streams-api)

    * [2.4 Connect API](./documentations/apis.md#connect-api)

    * [2.5 AdminClient API](./documentations/apis.md#adminclient-api)

    * [2.6 Legacy APIs](./documentations/apis.md#legacy-apis)

3. CONFIGURATION

    * [3.1 Broker Configs](./documentations/configs/broker.md)

    * [3.2 Topic Configs](./documentations/configs/topic.md)

    * [3.3 Producer Configs](./documentations/configs/producer.md)

    * [3.4 Consumer Configs](./documentations/configs/consumer.md)

        * [3.4.1 New Consumer Configs](./documentations/configs/consumer.md)

        * [3.4.2 Old Consumer Configs](./documentations/configs/consumer.md)

    * [3.5 Kafka Connect Configs](./documentations/configs/connect.md)

    * [3.6 Kafka Streams Configs](./documentations/configs/streams.md)

    * [3.7 AdminClient Configs](./documentations/configs/admin_client.md)

4. DESIGN

    * [4.1 Motivation](./documentations/design/motivation.md)

    * [4.2 Persistence](./documentations/design/persistence.md)

    * [4.3 Efficiency](./documentations/design/efficiency.md)

    * [4.4 The Producer](./documentations/design/producer.md)

    * [4.5 The Consumer](./documentations/design/consumer.md)

    * [4.6 Message Delivery Semantics](./documentations/design/message_delivery_semantics.md)

    * [4.7 Replication](./documentations/design/replication.md)

    * [4.8 Log Compaction](./documentations/design/log_compaction.md)

    * [4.9 Quotas](./documentations/design/quotas.md)

5. IMPLEMENTATION

    * [5.1 Network Layer](./documentations/implementation/network_layer.md)

    * [5.2 Messages](./documentations/implementation/messages.md)

    * [5.3 Message format](./documentations/implementation/message_format.md)

    * [5.4 Log](./documentations/implementation/log.md)

    * [5.5 Distribution](./documentations/implementation/distribution.md)

6. OPERATIONS

    * [6.1 Basic Kafka Operations](./documentations/operations/basic_kafka.md)

        * Adding and removing topics
        * Modifying topics
        * Graceful shutdown
        * Balancing leadership
        * Checking consumer position
        * Mirroring data between clusters
        * Expanding your cluster
        * Decommissioning brokers
        * Increasing replication factor

    * [6.2 Datacenters](./documentations/operations/datacenters.md)

    * [6.3 Important Configs](./documentations/operations/kafka_configuration.md)

        * Important Client Configs
        * A Production Server Configs

    * [6.4 Java Version](./documentations/operations/java_version.md)

    * [6.5 Hardware and OS](./documentations/operations/hardware_os.md)

        * OS
        * Disks and Filesystems
        * Application vs OS Flush Management
        * Linux Flush Behavior
        * Ext4 Notes

    * [6.6 Monitoring](./documentations/operations/monitoring.md)

    * [6.7 ZooKeeper](./documentations/operations/zookeeper.md)

        * Stable Version
        * Operationalization

7. SECURITY

    * [7.1 Security Overview](./documentations/security/overview.md)

    * [7.2 Encryption and Authentication using SSL](./documentations/security/encryption_authentication.md)

    * [7.3 Authentication using SASL](./documentations/security/authentication_sasl.md)

    * [7.4 Authorization and ACLs](./documentations/security/authorization_acls.md)

    * [7.5 Incorporating Security Features in a Running Cluster](./documentations/security/incorporating_security_features.md)

    * [7.6 ZooKeeper Authentication](./documentations/security/zookeeper_authentication.md)

        * New Clusters
        * Migrating Clusters
        * Migrating the ZooKeeper Ensemble

8. KAFKA CONNECT

    * [8.1 Overview](./documentations/kafka_connect.md#概况)

    * [8.2 User Guide](./documentations/kafka_connect.md#用户指南)

        * [Running Kafka Connect](./documentations/kafka_connect.md#运行-kafka-connect)
        * [Configuring Connectors](./documentations/kafka_connect.md#配置connector)
        * [Transformations](./documentations/kafka_connect.md#转换器)
        * [REST API](./documentations/kafka_connect.md#rest-api)

    * [8.3 Connector Development Guide](./documentations/kafka_connect.md#connector-开发指南)

9. KAFKA STREAMS

    * [9.1 Play with a Streams Application](./documentations/kafka_streams/run_demo_app.md)

    * [9.2 Write your own Streams Applications](./documentations/kafka_streams/tutorial_write_app.md)

    * [9.3 Developer Manual](./documentations/kafka_streams/developer_guide.md)

    * [9.4 Core Concepts](./documentations/kafka_streams/concepts.md)

    * [9.5 Architecture](./documentations/kafka_streams/architecture.md)

    * [9.6 Upgrade Guide](./documentations/kafka_streams/upgrade.md)
