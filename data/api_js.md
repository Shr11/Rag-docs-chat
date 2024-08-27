@fluvio/client
@fluvio/client
Fluvio Client for Node.js
Node.js binding for Fluvio streaming platform.

Build Status Github All Releases License Docs
Installation on NPM

npm install @fluvio/client

Fluvio client is native module. It is written using Rust. The client will install the platform specific native .node module.

The Fluvio client uses TypeScript for typed definitions.
Documentation

Fluvio client uses Typedoc to generate the client API documentation.
Pre-Requisites

Fluvio should be up and running to use the Node fluvio client.

Click the link for installation instructions on the Fluvio GitHub repository.
Example usage

import Fluvio, { RecordSet } from '@fluvio/client';

// define the name of the topic to use
const TOPIC_NAME = "my-topic"

// define the partition where the topic
// records will be stored;
const PARTITION = 0

// Instantiate a new fluvio client
const fluvio = await Fluvio.connect();

//// Fluvio Admin Client

// Create a new Fluvio Admin Client to manage
// topics and partitions
const admin = await fluvio.admin();

// Create a new topic
await admin.createTopic(TOPIC_NAME)

//// Topic Producer

// Create a topic producer for the topic;
const producer = await fluvio.topicProducer(TOPIC_NAME);

// Send a new topic record
producer.sendRecord("stringified data", PARTITION)

//// Partition Consumer

// Instantiate a new topic listener;
const consumer = await fluvio.partitionConsumer(TOPIC_NAME, PARTITION)

// Listen for new topics sent by a topic producer;
await consumer.stream(async (data: string) => {
// handle data record
})

Please look at ./examples and ./tests folder for more detailed examples.
Contributing

If you'd like to contribute to the project, please read our Contributing guide.
License

This project is licensed under the Apache license. Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in Fluvio Client by you, shall be licensed as Apache, without any additional terms or conditions.
Installation on NPM
Documentation
Pre-Requisites
Example usage
Contributing
License
@fluvio/client

    Encryption
    ErrorCode
    OffsetFrom
    SmartModuleType
    CustomSpuSpec
    FluvioAdmin
    Offset
    PartitionConsumer
    SpuConfig
    SpuGroupSpec
    TopicProducer
    TopicSpec
    default
    Batch
    BatchHeader
    ConsumerConfig
    DefaultRecord
    Endpoint
    EnvVar
    FetchablePartitionResponse
    FluvioClient
    IngressAddr
    IngressPort
    Options
    PartitionMap
    PartitionMaps
    PartitionSpec
    PartitionSpecMetadata
    PartitionSpecStatus
    Record
    RecordSet
    ReplicaStatus
    ReplicationConfig
    StorageConfig
    Topic
    TopicReplicaParam
    KeyValue
    DEFAULT_HOST
    DEFAULT_ID
    DEFAULT_IGNORE_RACK_ASSIGNMENT
    DEFAULT_MIN_ID
    DEFAULT_OFFSET
    DEFAULT_OFFSET_FROM
    DEFAULT_PARTITIONS
    DEFAULT_PORT
    DEFAULT_PRIVATE_PORT
    DEFAULT_PUBLIC_PORT
    DEFAULT_REPLICAS
    DEFAULT_REPLICATION_FACTOR
    DEFAULT_TOPIC
    PartitionConsumerJS
    arrayBufferToString
    stringToArrayBuffer

Generated using TypeDoc
