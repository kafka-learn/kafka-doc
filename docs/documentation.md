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
    
    * 3.1 Broker Configs
    
    * 3.2 Topic Configs
    
    * 3.3 Producer Configs
    
    * 3.4 Consumer Configs
    
    * 3.4.1 New Consumer Configs
    
    * 3.4.2 Old Consumer Configs
    
    * 3.5 Kafka Connect Configs
    
    * 3.6 Kafka Streams Configs
    
    * 3.7 AdminClient Configs

4. DESIGN
    
    * 4.1 Motivation
    
    * 4.2 Persistence
    
    * 4.3 Efficiency
    
    * 4.4 The Producer
    
    * 4.5 The Consumer
    
    * 4.6 Message Delivery Semantics
    
    * 4.7 Replication
    
    * 4.8 Log Compaction
    
    * 4.9 Quotas

5. IMPLEMENTATION

    * 5.1 Network Layer
    
    * 5.2 Messages
    
    * 5.3 Message format
    
    * 5.4 Log
    
    * 5.5 Distribution

6. OPERATIONS
    
    * 6.1 Basic Kafka Operations
        
        * Adding and removing topics
        * Modifying topics
        * Graceful shutdown
        * Balancing leadership
        * Checking consumer position
        * Mirroring data between clusters
        * Expanding your cluster
        * Decommissioning brokers
        * Increasing replication factor

    * 6.2 Datacenters
    
    * 6.3 Important Configs

        * Important Client Configs
        * A Production Server Configs

    * 6.4 Java Version
    
    * 6.5 Hardware and OS
        
        * OS
        * Disks and Filesystems
        * Application vs OS Flush Management
        * Linux Flush Behavior
        * Ext4 Notes

    * 6.6 Monitoring

    * 6.7 ZooKeeper
        
        * Stable Version
        * Operationalization

7. SECURITY
    
    * 7.1 Security Overview
    
    * 7.2 Encryption and Authentication using SSL
    
    * 7.3 Authentication using SASL
    
    * 7.4 Authorization and ACLs
    
    * 7.5 Incorporating Security Features in a Running Cluster
    
    * 7.6 ZooKeeper Authentication

        * New Clusters
        * Migrating Clusters
        * Migrating the ZooKeeper Ensemble

8. KAFKA CONNECT
    
    * 8.1 Overview
    
    * 8.2 User Guide
        
        * Running Kafka Connect
        * Configuring Connectors
        * Transformations
        * REST API
    
    * 8.3 Connector Development Guide

9. KAFKA STREAMS
    
    * 9.1 Play with a Streams Application
    
    * 9.2 Write your own Streams Applications
    
    * 9.3 Developer Manual
    
    * 9.4 Core Concepts
    
    * 9.5 Architecture
    
    * 9.6 Upgrade Guide
