fluvio
class Record:

The individual record for a given stream.
Record(inner: Record)
def offset(self) -> int:

The offset from the initial offset for a given stream.
def value(self) -> List[int]:

Returns the contents of this Record's value
def value_string(self) -> str:

The UTF-8 decoded value for this record.
def key(self) -> List[int]:

Returns the contents of this Record's key, if it exists
def key_string(self) -> str:

The UTF-8 decoded key for this record.
def timestamp(self) -> int:

Timestamp of this record.
class RecordMetadata:

Metadata of a record send to a topic.
RecordMetadata(inner: RecordMetadata)
def offset(self) -> int:

Return the offset of the sent record in the topic/partition.
def partition_id(self) -> int:

Return the partition index the record was sent to.
class ProduceOutput:

Returned by of TopicProducer.send call allowing access to sent record metadata.
ProduceOutput(inner: ProduceOutput)
def wait(self) -> Optional[RecordMetadata]:

Wait for the record metadata.

This is a blocking call and may only return a RecordMetadata once. Any subsequent call to wait will return a None value.
async def async_wait(self) -> Optional[RecordMetadata]:

Asynchronously wait for the record metadata.

This may only return a RecordMetadata once. Any subsequent call to wait will return a None value.
class Offset:

Describes the location of an event stored in a Fluvio partition.
Offset(inner: Offset)
@classmethod
def absolute(cls, index: int):

Creates an absolute offset with the given index
@classmethod
def beginning(cls):

Creates a relative offset starting at the beginning of the saved log
@classmethod
def end(cls):

Creates a relative offset pointing to the newest log entry
@classmethod
def from_beginning(cls, offset: int):

Creates a relative offset a fixed distance after the oldest log entry
@classmethod
def from_end(cls, offset: int):

Creates a relative offset a fixed distance before the newest log entry
class SmartModuleKind(enum.Enum):

Use of this is to explicitly set the kind of a smartmodule. Not required but needed for legacy SmartModules.
Filter = <SmartModuleKind.Filter: SmartModuleKind.Filter>
Map = <SmartModuleKind.Map: SmartModuleKind.Map>
ArrayMap = <SmartModuleKind.ArrayMap: SmartModuleKind.ArrayMap>
FilterMap = <SmartModuleKind.FilterMap: SmartModuleKind.FilterMap>
Aggregate = <SmartModuleKind.Aggregate: SmartModuleKind.Aggregate>
Inherited Members

enum.Enum
name
value

class ConsumerConfig:
def smartmodule( self, name: str = None, path: str = None, kind: SmartModuleKind = None, params: Dict[str, str] = None, aggregate: List[bytes] = None):

This is a method for adding a smartmodule to a consumer config either using a name of a SmartModule or a path to a wasm binary.

Args:

name: str
path: str
kind: SmartModuleKind
params: Dict[str, str]
aggregate: List[bytes] # This is used for the initial value of an aggregate smartmodule

Raises: "Require either a path or a name for a smartmodule." "Only specify one of path or name not both."

Returns:

None

class PartitionConsumer:

An interface for consuming events from a particular partition

There are two ways to consume events: by "fetching" events and by "streaming" events. Fetching involves specifying a range of events that you want to consume via their Offset. A fetch is a sort of one-time batch operation: you’ll receive all of the events in your range all at once. When you consume events via Streaming, you specify a starting Offset and receive an object that will continuously yield new events as they arrive.
PartitionConsumer(inner: PartitionConsumer)
def stream(self, offset: Offset) -> Iterator[Record]:

Continuously streams events from a particular offset in the consumer’s partition. This returns a Iterator[Record] which is an iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.
async def async_stream(self, offset: Offset) -> AsyncIterator[Record]:

Continuously streams events from a particular offset in the consumer’s partition. This returns a AsyncIterator[Record] which is an iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.
def stream_with_config( self, offset: Offset, config: ConsumerConfig) -> Iterator[Record]:

Continuously streams events from a particular offset with a SmartModule WASM module in the consumer’s partition. This returns a Iterator[Record] which is an iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.

Args: offset: Offset wasm_module_path: str - The absolute path to the WASM file

Example: import os

wmp = os.path.abspath("somefilter.wasm")
config = ConsumerConfig()
config.smartmodule(path=wmp)
for i in consumer.stream_with_config(Offset.beginning(), config): # do something with i

Returns: Iterator[Record]
async def async_stream_with_config( self, offset: Offset, config: ConsumerConfig) -> AsyncIterator[Record]:

Continuously streams events from a particular offset with a SmartModule WASM module in the consumer’s partition. This returns a AsyncIterator[Record] which is an async iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.

Args: offset: Offset wasm_module_path: str - The absolute path to the WASM file

Example: import os

wmp = os.path.abspath("somefilter.wasm")
config = ConsumerConfig()
config.smartmodule(path=wmp)
async for i in await consumer.async_stream_with_config(Offset.beginning(), config): # do something with i

Returns: AsyncIterator[Record]
class MultiplePartitionConsumer:

An interface for consuming events from multiple partitions

There are two ways to consume events: by "fetching" events and by "streaming" events. Fetching involves specifying a range of events that you want to consume via their Offset. A fetch is a sort of one-time batch operation: you’ll receive all of the events in your range all at once. When you consume events via Streaming, you specify a starting Offset and receive an object that will continuously yield new events as they arrive.
MultiplePartitionConsumer(inner: MultiplePartitionConsumer)
def stream(self, offset: Offset) -> Iterator[Record]:

Continuously streams events from a particular offset in the consumer’s partitions. This returns a Iterator[Record] which is an iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.
async def async_stream(self, offset: Offset) -> AsyncIterator[Record]:

Continuously streams events from a particular offset in the consumer’s partitions. This returns a AsyncIterator[Record] which is an async iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.
def stream_with_config( self, offset: Offset, config: ConsumerConfig) -> Iterator[Record]:

Continuously streams events from a particular offset with a SmartModule WASM module in the consumer’s partitions. This returns a Iterator[Record] which is an iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.

Args: offset: Offset wasm_module_path: str - The absolute path to the WASM file

Example: import os

wmp = os.path.abspath("somefilter.wasm")
config = ConsumerConfig()
config.smartmodule(path=wmp)
for i in consumer.stream_with_config(Offset.beginning(), config): # do something with i

Returns: Iterator[Record]
async def async_stream_with_config( self, offset: Offset, config: ConsumerConfig) -> AsyncIterator[Record]:

Continuously streams events from a particular offset with a SmartModule WASM module in the consumer’s partitions. This returns a AsyncIterator[Record] which is an async iterator.

Streaming is one of the two ways to consume events in Fluvio. It is a continuous request for new records arriving in a partition, beginning at a particular offset. You specify the starting point of the stream using an Offset and periodically receive events, either individually or in batches.

Args: offset: Offset wasm_module_path: str - The absolute path to the WASM file

Example: import os

wmp = os.path.abspath("somefilter.wasm")
config = ConsumerConfig()
config.smartmodule(path=wmp)
async for i in await consumer.async_stream_with_config(Offset.beginning(), config): # do something with i

Returns: AsyncIterator[Record]
class TopicProducer:

An interface for producing events to a particular topic.

A TopicProducer allows you to send events to the specific topic it was initialized for. Once you have a TopicProducer, you can send events to the topic, choosing which partition each event should be delivered to.
TopicProducer(inner: TopicProducer)
def send_string(self, buf: str) -> ProduceOutput:

Sends a string to this producer’s topic
async def async_send_string(self, buf: str) -> ProduceOutput:

Sends a string to this producer’s topic
def send(self, key: List[int], value: List[int]) -> ProduceOutput:

Sends a key/value record to this producer's Topic.

The partition that the record will be sent to is derived from the Key.
async def async_send(self, key: List[int], value: List[int]) -> ProduceOutput:

Async sends a key/value record to this producer's Topic.

The partition that the record will be sent to is derived from the Key.
def flush(self) -> None:

Send all the queued records in the producer batches.
async def async_flush(self) -> None:

Async send all the queued records in the producer batches.
def send_all(self, records: List[Tuple[bytes, bytes]]) -> List[ProduceOutput]:

Sends a list of key/value records as a batch to this producer's Topic.
Parameters

    records: The list of records to send

async def async_send_all(self, records: List[Tuple[bytes, bytes]]) -> List[ProduceOutput]:

Async sends a list of key/value records as a batch to this producer's Topic.
Parameters

    records: The list of records to send

class PartitionSelectionStrategy:

Stragegy to select partitions
PartitionSelectionStrategy(inner: FluvioConfig)
@classmethod
def with_all(cls, topic: str):

select all partitions of one topic
@classmethod
def with_multiple(cls, topic: List[Tuple[str, int]]):

select multiple partitions of multiple topics
class FluvioConfig:

Configuration for Fluvio client
FluvioConfig(inner: FluvioConfig)
@classmethod
def load(cls):

get current cluster config from default profile
@classmethod
def new(cls, addr: str):

Create a new cluster configuration with no TLS.
def set_endpoint(self, endpoint: str):

set endpoint
def set_use_spu_local_address(self, val: bool):

set wheather to use spu local address
def disable_tls(self):

disable tls for this config
def set_anonymous_tls(self):

set the config to use anonymous tls
def set_inline_tls(self, domain: str, key: str, cert: str, ca_cert: str):

specify inline tls parameters
def set_tls_file_paths(self, domain: str, key_path: str, cert_path: str, ca_cert_path: str):

specify paths to tls files
def set_client_id(self, client_id: str):

set client id
def unset_client_id(self):

remove the configured client id from config
class Fluvio:

An interface for interacting with Fluvio streaming.
Fluvio(inner: Fluvio)
@classmethod
def connect(cls):

Tries to create a new Fluvio client using the current profile from ~/.fluvio/config
@classmethod
def connect_with_config(cls, config: FluvioConfig):

Creates a new Fluvio client using the given configuration
def partition_consumer(self, topic: str, partition: int) -> PartitionConsumer:

Creates a new PartitionConsumer for the given topic and partition

Currently, consumers are scoped to both a specific Fluvio topic and to a particular partition within that topic. That means that if you have a topic with multiple partitions, then in order to receive all of the events in all of the partitions, you will need to create one consumer per partition.
def multi_partition_consumer(self, topic: str) -> MultiplePartitionConsumer:

Creates a new MultiplePartitionConsumer for the given topic and its all partitions

Currently, consumers are scoped to both a specific Fluvio topic and to its all partitions within that topic.
def multi_topic_partition_consumer( self, selections: List[Tuple[str, int]]) -> MultiplePartitionConsumer:

Creates a new MultiplePartitionConsumer for the given topics and partitions

Currently, consumers are scoped to a list of Fluvio topic and partition tuple.
def topic_producer(self, topic: str) -> TopicProducer:

Creates a new TopicProducer for the given topic name.

Currently, producers are scoped to a specific Fluvio topic. That means when you send events via a producer, you must specify which partition each event should go to.
class PartitionMap:
PartitionMap(inner: PartitionMap)
@classmethod
def new(cls, partition: int, replicas: List[int]):
class TopicSpec:
TopicSpec(inner: TopicSpec)
@classmethod
def new_assigned(cls, partition_maps: List[PartitionMap]):
@classmethod
def new_computed(cls, partitions: int, replication: int, ignore: bool):
class CommonCreateRequest:
CommonCreateRequest(inner: CommonCreateRequest)
@classmethod
def new(cls, name: str, dry_run: bool, timeout: int):
class MetadataTopicSpec:
MetadataTopicSpec(inner: MetadataTopicSpec)
def name(self) -> str:
class MessageMetadataTopicSpec:
MessageMetadataTopicSpec(inner: MessageMetadataTopicSpec)
def is_update(self) -> bool:
def is_delete(self) -> bool:
def metadata_topic_spec(self) -> MetadataTopicSpec:
class MetaUpdateTopicSpec:
MetaUpdateTopicSpec(inner: MetaUpdateTopicSpec)
def all(self) -> List[MetadataTopicSpec]:
def changes(self) -> List[MessageMetadataTopicSpec]:
def epoch(self) -> int:
class SmartModuleSpec:
SmartModuleSpec(inner: SmartModuleSpec)
@classmethod
def new(cls, path: str):
class MetadataSmartModuleSpec:
MetadataSmartModuleSpec(inner: MetadataSmartModuleSpec)
def name(self) -> str:
class MessageMetadataSmartModuleSpec:
MessageMetadataSmartModuleSpec(inner: MessageMetadataSmartModuleSpec)
def is_update(self) -> bool:
def is_delete(self) -> bool:
def metadata_smart_module_spec(self) -> MetadataSmartModuleSpec:
class MetaUpdateSmartModuleSpec:
MetaUpdateSmartModuleSpec(inner: MetaUpdateSmartModuleSpec)
def all(self) -> List[MetadataSmartModuleSpec]:
def changes(self) -> List[MessageMetadataSmartModuleSpec]:
def epoch(self) -> int:
class MetadataPartitionSpec:
MetadataPartitionSpec(inner: MetadataPartitionSpec)
def name(self) -> str:
class FluvioAdmin:
FluvioAdmin(inner: FluvioAdmin)
def connect():
def connect_with_config(config: FluvioConfig):
def create_topic(self, topic: str, dry_run: bool, spec: TopicSpec):
def create_topic_with_config( self, topic: str, req: CommonCreateRequest, spec: TopicSpec):
def delete_topic(self, topic: str):
def all_topics(self) -> List[MetadataTopicSpec]:
def list_topics(self, filters: List[str]) -> List[MetadataTopicSpec]:
def list_topics_with_params( self, filters: List[str], summary: bool) -> List[MetadataTopicSpec]:
def watch_topic(self) -> Iterator[MetadataTopicSpec]:
def create_smart_module(self, name: str, path: str, dry_run: bool):
def delete_smart_module(self, name: str):
def list_smart_modules(self, filters: List[str]) -> List[MetadataSmartModuleSpec]:
def watch_smart_module(self) -> Iterator[MetadataSmartModuleSpec]:
def list_partitions(self, filters: List[str]) -> List[MetadataPartitionSpec]:
