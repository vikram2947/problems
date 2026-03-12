# Kafka Interview Problems for SDE 2

## Table of Contents
1. [Producer Problems](#producer-problems)
2. [Consumer Problems](#consumer-problems)
3. [Partitioning & Replication](#partitioning--replication)
4. [Consumer Groups & Rebalancing](#consumer-groups--rebalancing)
5. [Offset Management](#offset-management)
6. [Exactly-Once Semantics](#exactly-once-semantics)
7. [Performance & Optimization](#performance--optimization)
8. [Fault Tolerance & High Availability](#fault-tolerance--high-availability)
9. [Schema Registry & Serialization](#schema-registry--serialization)
10. [Stream Processing](#stream-processing)
11. [Monitoring & Troubleshooting](#monitoring--troubleshooting)
12. [Security & Authentication](#security--authentication)
13. [Design & Architecture](#design--architecture)
14. [Kafka Internals & Advanced Concepts](#kafka-internals--advanced-concepts)
15. [Testing & Operations](#testing--operations)

---

## Producer Problems

### Problem 1: Idempotent Producer Implementation
**Scenario**: You need to implement an idempotent producer that ensures no duplicate messages are sent even if the producer retries due to network failures.

**Requirements**:
- Implement a producer that can handle retries without duplicates
- Handle producer ID (PID) and sequence number management
- Ensure exactly-once semantics

**Questions**:
1. How does Kafka achieve idempotency at the producer level?
2. What happens if a producer crashes and restarts?
3. How do you configure idempotent producers?

---

### Problem 2: Custom Partitioner Implementation
**Scenario**: You need to route messages to partitions based on a custom business logic (e.g., route all messages from a specific customer to the same partition).

**Requirements**:
- Implement a custom partitioner
- Ensure messages with the same key always go to the same partition
- Handle null keys appropriately

**Questions**:
1. How does the default partitioner work?
2. What happens when you add/remove partitions?
3. How to ensure key-based partitioning maintains order?

---

### Problem 3: Producer Batching and Compression
**Scenario**: You need to optimize producer throughput by implementing batching and compression.

**Requirements**:
- Configure producer to batch messages efficiently
- Implement compression (gzip, snappy, lz4, zstd)
- Balance between latency and throughput

**Questions**:
1. How does `linger.ms` affect batching?
2. What's the trade-off between compression ratio and CPU usage?
3. How to choose the right batch size?

---

### Problem 4: Producer Error Handling and Retries
**Scenario**: Implement robust error handling for producer failures.

**Requirements**:
- Handle retriable vs non-retriable errors
- Implement exponential backoff
- Handle timeout scenarios

**Questions**:
1. Which errors are retriable in Kafka?
2. How does `max.in.flight.requests.per.connection` affect ordering?
3. What happens when `retries` is set to Integer.MAX_VALUE?

---

## Consumer Problems

### Problem 5: Consumer Group Rebalancing
**Scenario**: Design a system where multiple consumers process messages from a topic, and new consumers can join/leave dynamically.

**Requirements**:
- Handle consumer group rebalancing gracefully
- Minimize downtime during rebalancing
- Implement cooperative rebalancing

**Questions**:
1. What are the different rebalancing strategies?
2. How does cooperative rebalancing differ from eager rebalancing?
3. What causes rebalancing and how to minimize it?

---

### Problem 6: Manual Offset Management
**Scenario**: Implement a consumer that commits offsets manually at specific checkpoints.

**Requirements**:
- Commit offsets after processing batches
- Handle offset commit failures
- Implement at-least-once semantics

**Questions**:
1. When should you use manual vs automatic offset commits?
2. How to handle duplicate processing with manual commits?
3. What happens if offset commit fails?

---

### Problem 7: Consumer Lag Monitoring
**Scenario**: Build a monitoring system to track consumer lag and alert when lag exceeds thresholds.

**Requirements**:
- Calculate consumer lag for each partition
- Set up alerts for high lag
- Handle lag spikes gracefully

**Questions**:
1. How is consumer lag calculated?
2. What causes high consumer lag?
3. How to reduce consumer lag?

---

### Problem 8: Consumer with Multiple Topics
**Scenario**: Implement a consumer that subscribes to multiple topics with different processing logic for each.

**Requirements**:
- Subscribe to multiple topics
- Route messages based on topic
- Handle different schemas per topic

**Questions**:
1. How does Kafka handle subscriptions to multiple topics?
2. Can you have different consumer groups for different topics?
3. How to ensure ordering across multiple topics?

---

## Partitioning & Replication

### Problem 9: Partition Count Optimization
**Scenario**: Determine the optimal number of partitions for a topic handling 1M messages/sec.

**Requirements**:
- Calculate partition count based on throughput
- Consider consumer parallelism
- Handle partition growth over time

**Questions**:
1. What factors determine the number of partitions?
2. What are the downsides of too many partitions?
3. How to change partition count for an existing topic?

---

### Problem 10: Replication Factor and ISR
**Scenario**: Configure replication for a critical topic that must survive broker failures.

**Requirements**:
- Set appropriate replication factor
- Understand In-Sync Replicas (ISR)
- Handle under-replicated partitions

**Questions**:
1. What is the minimum replication factor for high availability?
2. How does Kafka determine if a replica is in-sync?
3. What happens when ISR size drops below minimum?

---

### Problem 11: Leader Election and Unclean Leader Election
**Scenario**: Configure leader election behavior for a topic that prioritizes availability over consistency.

**Requirements**:
- Understand leader election process
- Configure `unclean.leader.election.enable`
- Handle split-brain scenarios

**Questions**:
1. When does leader election occur?
2. What is unclean leader election and when is it acceptable?
3. How to prevent data loss during leader election?

---

## Consumer Groups & Rebalancing

### Problem 12: Static vs Dynamic Membership
**Scenario**: Compare static and dynamic consumer group membership strategies.

**Requirements**:
- Implement both membership types
- Understand when to use each
- Handle membership changes

**Questions**:
1. What are the advantages of static membership?
2. How does dynamic membership work?
3. When should you use static membership?

---

### Problem 13: Consumer Group Coordinator
**Scenario**: Understand the role of the group coordinator in consumer group management.

**Requirements**:
- Explain coordinator election
- Handle coordinator failures
- Understand group metadata

**Questions**:
1. How is the group coordinator selected?
2. What happens if the coordinator fails?
3. How does the coordinator track consumer group state?

---

## Offset Management

### Problem 14: Offset Reset Strategies
**Scenario**: Implement a consumer that can handle different offset reset scenarios.

**Requirements**:
- Handle `earliest` vs `latest` offset reset
- Implement custom offset reset logic
- Handle offset out of range errors

**Questions**:
1. When does offset reset occur?
2. What's the difference between `earliest` and `latest`?
3. How to implement custom offset reset?

---

### Problem 15: Offset Storage (Internal vs External)
**Scenario**: Compare storing offsets in Kafka vs external storage (e.g., database).

**Requirements**:
- Implement both approaches
- Understand trade-offs
- Handle offset migration

**Questions**:
1. What are the advantages of storing offsets externally?
2. How does Kafka store offsets internally?
3. When should you use external offset storage?

---

## Exactly-Once Semantics

### Problem 16: Transactional Producer
**Scenario**: Implement a transactional producer that ensures exactly-once semantics.

**Requirements**:
- Enable transactions
- Handle transaction boundaries
- Coordinate with transactional consumers

**Questions**:
1. How do transactions work in Kafka?
2. What is the transaction coordinator?
3. How to handle transaction timeouts?

---

### Problem 17: Exactly-Once Processing
**Scenario**: Implement a consumer-producer pipeline with exactly-once semantics.

**Requirements**:
- Use transactional producer
- Use transactional consumer
- Handle idempotent processing

**Questions**:
1. How to achieve exactly-once semantics end-to-end?
2. What are the performance implications?
3. How to handle failures in transactional processing?

---

## Performance & Optimization

### Problem 18: Producer Throughput Optimization
**Scenario**: Optimize producer to achieve maximum throughput for a high-volume topic.

**Requirements**:
- Tune batch size and linger time
- Optimize compression
- Configure acks appropriately

**Questions**:
1. How does `acks` setting affect throughput?
2. What's the optimal batch size?
3. How to balance latency vs throughput?

---

### Problem 19: Consumer Throughput Optimization
**Scenario**: Optimize consumer to process messages as fast as possible.

**Requirements**:
- Increase fetch size
- Optimize processing logic
- Parallelize consumption

**Questions**:
1. How does `fetch.min.bytes` and `fetch.max.wait.ms` affect throughput?
2. How to parallelize consumer processing?
3. What are common bottlenecks in consumer processing?

---

### Problem 20: Topic-Level Configuration Tuning
**Scenario**: Optimize topic-level configurations for a specific use case.

**Requirements**:
- Tune retention policies
- Configure segment size
- Optimize index intervals

**Questions**:
1. How does `log.segment.bytes` affect performance?
2. What's the impact of `log.retention.hours`?
3. How to optimize for different use cases (streaming vs batch)?

---

## Fault Tolerance & High Availability

### Problem 21: Broker Failure Handling
**Scenario**: Design a Kafka cluster that can survive broker failures.

**Requirements**:
- Configure replication
- Handle broker failures gracefully
- Minimize impact on producers/consumers

**Questions**:
1. How many broker failures can a cluster survive?
2. What happens to partitions when a broker fails?
3. How to detect and handle broker failures?

---

### Problem 22: Network Partition Handling
**Scenario**: Handle network partitions in a Kafka cluster.

**Requirements**:
- Understand split-brain scenarios
- Configure `min.insync.replicas`
- Handle partition unavailability

**Questions**:
1. What happens during a network partition?
2. How does `min.insync.replicas` prevent data loss?
3. How to detect and recover from network partitions?

---

### Problem 23: Data Loss Prevention
**Scenario**: Implement safeguards to prevent data loss in critical scenarios.

**Requirements**:
- Configure replication properly
- Use appropriate acks
- Handle unclean leader election

**Questions**:
1. What are common causes of data loss?
2. How to prevent data loss?
3. How to detect data loss?

---

## Schema Registry & Serialization

### Problem 24: Schema Evolution
**Scenario**: Handle schema changes in a Kafka topic without breaking consumers.

**Requirements**:
- Implement backward/forward compatible schemas
- Use Schema Registry
- Handle schema versioning

**Questions**:
1. What is schema evolution?
2. What are backward and forward compatibility?
3. How does Schema Registry handle schema changes?

---

### Problem 25: Avro Serialization
**Scenario**: Implement Avro serialization/deserialization with Schema Registry.

**Requirements**:
- Configure Avro serializer
- Handle schema registry failures
- Implement schema caching

**Questions**:
1. Why use Avro over JSON?
2. How does Schema Registry cache work?
3. How to handle schema registry unavailability?

---

## Stream Processing

### Problem 26: Kafka Streams Application
**Scenario**: Build a Kafka Streams application that processes a stream of events.

**Requirements**:
- Transform messages
- Aggregate data
- Join streams

**Questions**:
1. What is Kafka Streams?
2. How does Kafka Streams achieve exactly-once processing?
3. How to handle stateful operations?

---

### Problem 27: Stream-Table Join
**Scenario**: Implement a stream-table join in Kafka Streams.

**Requirements**:
- Join a stream with a KTable
- Handle late-arriving data
- Maintain join state

**Questions**:
1. What is the difference between KStream and KTable?
2. How does stream-table join work?
3. How to handle out-of-order events?

---

### Problem 28: Windowing and Aggregation
**Scenario**: Implement time-based windowing and aggregation in Kafka Streams.

**Requirements**:
- Create tumbling windows
- Aggregate data within windows
- Handle window boundaries

**Questions**:
1. What are the different window types?
2. How does Kafka Streams handle late data?
3. How to choose the right window size?

---

## Monitoring & Troubleshooting

### Problem 29: Consumer Lag Alerting
**Scenario**: Build a monitoring system that alerts when consumer lag exceeds thresholds.

**Requirements**:
- Monitor consumer lag metrics
- Set up alerts
- Handle false positives

**Questions**:
1. How to monitor consumer lag?
2. What are normal vs abnormal lag values?
3. How to reduce false positives in alerts?

---

### Problem 30: Broker Metrics Monitoring
**Scenario**: Monitor key broker metrics and set up dashboards.

**Requirements**:
- Monitor throughput metrics
- Track replication metrics
- Alert on critical issues

**Questions**:
1. What are the key broker metrics to monitor?
2. How to monitor partition leadership?
3. What metrics indicate broker health issues?

---

### Problem 31: Debugging Message Ordering Issues
**Scenario**: Debug why messages are not being processed in order.

**Requirements**:
- Understand ordering guarantees
- Identify ordering violations
- Fix ordering issues

**Questions**:
1. When is ordering guaranteed in Kafka?
2. What breaks message ordering?
3. How to ensure ordering across partitions?

---

## Security & Authentication

### Problem 32: SASL Authentication Setup
**Scenario**: Configure SASL authentication for Kafka brokers and clients.

**Requirements**:
- Set up SASL/PLAIN authentication
- Configure clients to authenticate
- Handle authentication failures

**Questions**:
1. What are the different SASL mechanisms?
2. How does SASL/PLAIN work?
3. How to secure credentials?

---

### Problem 33: ACL Authorization
**Scenario**: Implement fine-grained access control using ACLs.

**Requirements**:
- Configure ACLs for topics
- Set up user permissions
- Audit access

**Questions**:
1. What are ACLs in Kafka?
2. How to configure ACLs?
3. How to audit ACL changes?

---

### Problem 34: SSL/TLS Encryption
**Scenario**: Enable SSL/TLS encryption for Kafka communication.

**Requirements**:
- Configure SSL on brokers
- Enable SSL on clients
- Handle certificate management

**Questions**:
1. How does SSL work in Kafka?
2. What's the difference between one-way and two-way SSL?
3. How to rotate SSL certificates?

---

## Design & Architecture

### Problem 35: Multi-Datacenter Kafka Setup
**Scenario**: Design a Kafka setup spanning multiple datacenters.

**Requirements**:
- Configure replication across datacenters
- Handle network latency
- Design disaster recovery

**Questions**:
1. How to replicate data across datacenters?
2. What are the challenges of multi-DC setup?
3. How to handle datacenter failures?

---

### Problem 36: Topic Design for Microservices
**Scenario**: Design Kafka topics for a microservices architecture.

**Requirements**:
- Design topic naming conventions
- Determine topic granularity
- Handle schema evolution

**Questions**:
1. How many topics should a microservice have?
2. What's the best topic naming strategy?
3. How to handle shared topics?

---

### Problem 37: Event Sourcing with Kafka
**Scenario**: Implement event sourcing pattern using Kafka as the event store.

**Requirements**:
- Store events in Kafka
- Replay events for state reconstruction
- Handle event versioning

**Questions**:
1. How does Kafka support event sourcing?
2. How to replay events?
3. How to handle schema evolution in event sourcing?

---

### Problem 38: CQRS with Kafka
**Scenario**: Implement CQRS (Command Query Responsibility Segregation) using Kafka.

**Requirements**:
- Separate command and query topics
- Maintain read models
- Handle eventual consistency

**Questions**:
1. How does Kafka support CQRS?
2. How to maintain read models?
3. How to handle consistency in CQRS?

---

### Problem 39: Dead Letter Queue Pattern
**Scenario**: Implement a dead letter queue for messages that fail processing.

**Requirements**:
- Route failed messages to DLQ
- Retry failed messages
- Monitor DLQ

**Questions**:
1. What is a dead letter queue?
2. When should messages go to DLQ?
3. How to handle messages in DLQ?

---

### Problem 40: Kafka Connect Integration
**Scenario**: Use Kafka Connect to integrate Kafka with external systems.

**Requirements**:
- Set up source connectors
- Configure sink connectors
- Handle connector failures

**Questions**:
1. What is Kafka Connect?
2. How do source and sink connectors work?
3. How to handle connector failures and restarts?

---

## Kafka Internals & Advanced Concepts

### Problem 46: KRaft (Kafka Raft) Migration
**Scenario**: Understand the migration from Zookeeper to KRaft and its implications.

**Requirements**:
- Understand KRaft architecture
- Compare Zookeeper vs KRaft
- Plan migration strategy
- Handle KRaft-specific configurations

**Questions**:
1. What is KRaft and why was it introduced?
2. What are the advantages of KRaft over Zookeeper?
3. How does the migration process work?
4. What are the key differences in cluster management?

---

### Problem 47: Kafka Storage Internals
**Scenario**: Understand how Kafka stores data internally on disk.

**Requirements**:
- Understand log segments and segment files
- Explain index files (offset and time indexes)
- Understand how reads and writes work
- Handle segment rolling and cleanup

**Questions**:
1. How does Kafka store messages on disk?
2. What are log segments and how do they work?
3. How do index files improve read performance?
4. How does segment rolling work?

---

### Problem 48: Message Headers and Metadata
**Scenario**: Use message headers to pass metadata, implement tracing, and route messages.

**Requirements**:
- Add custom headers to messages
- Extract headers in consumers
- Use headers for distributed tracing
- Implement header-based routing

**Questions**:
1. What are message headers and when to use them?
2. How to add and read headers?
3. How to use headers for tracing (OpenTracing, etc.)?
4. What's the performance impact of headers?

---

### Problem 49: Kafka Controller and Cluster Coordination
**Scenario**: Understand how the Kafka controller manages cluster state and coordinates operations.

**Requirements**:
- Understand controller responsibilities
- Handle controller failures
- Understand metadata propagation
- Explain partition leadership management

**Questions**:
1. What is the Kafka controller and what does it do?
2. How is the controller elected?
3. What happens when the controller fails?
4. How does the controller manage partition leadership?

---

### Problem 50: Kafka Protocol Internals
**Scenario**: Understand the Kafka wire protocol and how clients communicate with brokers.

**Requirements**:
- Understand request/response protocol
- Explain produce and fetch protocols
- Handle protocol versions
- Understand protocol efficiency

**Questions**:
1. How does the Kafka protocol work?
2. What are the main protocol requests?
3. How does batching work at the protocol level?
4. How to handle protocol version mismatches?

---

### Problem 51: Timestamp Handling and Out-of-Order Events
**Scenario**: Handle message timestamps and process out-of-order events correctly.

**Requirements**:
- Understand log append time vs create time
- Handle out-of-order events
- Implement time-based processing
- Use timestamps for windowing

**Questions**:
1. What are the timestamp types in Kafka?
2. How to handle out-of-order events?
3. When to use log append time vs create time?
4. How do timestamps affect stream processing?

---

### Problem 52: Kafka Admin API
**Scenario**: Programmatically manage Kafka topics, partitions, and configurations.

**Requirements**:
- Create and delete topics programmatically
- Alter topic configurations
- Manage partitions
- Handle admin operations asynchronously

**Questions**:
1. How to use Kafka Admin API?
2. What operations can be performed programmatically?
3. How to handle admin operation failures?
4. What are the best practices for admin operations?

---

### Problem 53: Backpressure Handling
**Scenario**: Handle situations where consumers cannot keep up with producer throughput.

**Requirements**:
- Detect backpressure
- Implement backpressure strategies
- Throttle producers when needed
- Scale consumers dynamically

**Questions**:
1. What causes backpressure in Kafka?
2. How to detect backpressure?
3. What strategies can handle backpressure?
4. How to prevent backpressure?

---

### Problem 54: Kafka Log Retention Internals
**Scenario**: Understand how log retention policies work internally.

**Requirements**:
- Understand time-based retention
- Understand size-based retention
- Handle retention cleanup process
- Optimize retention for different use cases

**Questions**:
1. How does time-based retention work?
2. How does size-based retention work?
3. When does cleanup occur?
4. How to optimize retention policies?

---

### Problem 55: Leader-Follower Replication Protocol
**Scenario**: Understand how replication works at the protocol level between leader and followers.

**Requirements**:
- Understand fetch requests from followers
- Explain how followers catch up
- Handle replication lag
- Understand high watermark and LEO

**Questions**:
1. How do followers replicate data from leaders?
2. What is the high watermark?
3. What is LEO (Log End Offset)?
4. How does a follower catch up after lagging?

---

## Testing & Operations

### Problem 56: Producer and Consumer Interceptors
**Scenario**: Implement interceptors to add cross-cutting concerns like logging, metrics, and tracing.

**Requirements**:
- Implement producer interceptor
- Implement consumer interceptor
- Add custom logic before/after message processing
- Handle interceptor failures gracefully

**Questions**:
1. What are interceptors and when to use them?
2. How to implement producer interceptors?
3. How to implement consumer interceptors?
4. What are common use cases for interceptors?

---

### Problem 57: Kafka Streams State Stores and Interactive Queries
**Scenario**: Use state stores in Kafka Streams and expose them via interactive queries.

**Requirements**:
- Understand different state store types (RocksDB, in-memory)
- Implement stateful operations with state stores
- Expose state stores via REST API
- Handle state store failures and recovery

**Questions**:
1. What are state stores in Kafka Streams?
2. What are the different types of state stores?
3. How to implement interactive queries?
4. How does state store recovery work?

---

### Problem 58: Outbox Pattern Implementation
**Scenario**: Implement the outbox pattern to ensure reliable event publishing from database transactions.

**Requirements**:
- Implement transactional outbox table
- Publish events from outbox to Kafka
- Ensure exactly-once event publishing
- Handle outbox cleanup

**Questions**:
1. What is the outbox pattern and why is it needed?
2. How does it ensure consistency between database and Kafka?
3. How to implement the outbox pattern?
4. How to handle outbox cleanup and deduplication?

---

### Problem 59: Testing Kafka Applications
**Scenario**: Write comprehensive tests for Kafka producers, consumers, and stream processing applications.

**Requirements**:
- Unit test producers and consumers
- Integration test with embedded Kafka
- Test Kafka Streams applications
- Mock Kafka for unit tests

**Questions**:
1. How to unit test Kafka producers?
2. How to integration test Kafka consumers?
3. How to test Kafka Streams applications?
4. What testing tools are available?

---

### Problem 60: Capacity Planning and Sizing
**Scenario**: Plan Kafka cluster capacity for a given workload and growth projections.

**Requirements**:
- Calculate broker requirements
- Estimate disk storage needs
- Plan for network bandwidth
- Design for future growth

**Questions**:
1. How to calculate broker count?
2. How to estimate disk storage requirements?
3. How to plan network capacity?
4. What factors affect Kafka cluster sizing?

---

## Advanced Topics

### Problem 41: Compacted Topics
**Scenario**: Use log compaction to maintain the latest value for each key.

**Requirements**:
- Configure topic compaction
- Understand compaction process
- Handle compaction lag

**Questions**:
1. What is log compaction?
2. When should you use compacted topics?
3. How does compaction work?

---

### Problem 42: Quotas and Throttling
**Scenario**: Implement quotas to limit producer/consumer throughput.

**Requirements**:
- Configure producer quotas
- Set consumer quotas
- Handle quota violations

**Questions**:
1. What are quotas in Kafka?
2. How to configure quotas?
3. What happens when quotas are exceeded?

---

### Problem 43: MirrorMaker 2.0 Setup
**Scenario**: Set up MirrorMaker 2.0 for cross-cluster replication.

**Requirements**:
- Configure MirrorMaker connectors
- Handle replication lag
- Design failover strategy

**Questions**:
1. What is MirrorMaker 2.0?
2. How does it differ from MirrorMaker 1.0?
3. How to handle replication failures?

---

### Problem 44: Kafka on Kubernetes
**Scenario**: Deploy and manage Kafka on Kubernetes.

**Requirements**:
- Deploy Kafka brokers on K8s
- Handle stateful sets
- Configure service discovery

**Questions**:
1. What are the challenges of running Kafka on K8s?
2. How to handle persistent storage?
3. How to scale Kafka on K8s?

---

### Problem 45: Performance Testing
**Scenario**: Design a performance testing strategy for Kafka.

**Requirements**:
- Measure throughput
- Test latency
- Identify bottlenecks

**Questions**:
1. How to measure Kafka performance?
2. What tools are available for performance testing?
3. How to identify performance bottlenecks?

---

### Problem 46: KRaft (Kafka Raft) Migration
**Scenario**: Understand the migration from Zookeeper to KRaft and its implications.

**Requirements**:
- Understand KRaft architecture
- Compare Zookeeper vs KRaft
- Plan migration strategy
- Handle KRaft-specific configurations

**Questions**:
1. What is KRaft and why was it introduced?
2. What are the advantages of KRaft over Zookeeper?
3. How does the migration process work?
4. What are the key differences in cluster management?

---

### Problem 47: Kafka Storage Internals
**Scenario**: Understand how Kafka stores data internally on disk.

**Requirements**:
- Understand log segments and segment files
- Explain index files (offset and time indexes)
- Understand how reads and writes work
- Handle segment rolling and cleanup

**Questions**:
1. How does Kafka store messages on disk?
2. What are log segments and how do they work?
3. How do index files improve read performance?
4. How does segment rolling work?

---

### Problem 48: Message Headers and Metadata
**Scenario**: Use message headers to pass metadata, implement tracing, and route messages.

**Requirements**:
- Add custom headers to messages
- Extract headers in consumers
- Use headers for distributed tracing
- Implement header-based routing

**Questions**:
1. What are message headers and when to use them?
2. How to add and read headers?
3. How to use headers for tracing (OpenTracing, etc.)?
4. What's the performance impact of headers?

---

### Problem 49: Kafka Controller and Cluster Coordination
**Scenario**: Understand how the Kafka controller manages cluster state and coordinates operations.

**Requirements**:
- Understand controller responsibilities
- Handle controller failures
- Understand metadata propagation
- Explain partition leadership management

**Questions**:
1. What is the Kafka controller and what does it do?
2. How is the controller elected?
3. What happens when the controller fails?
4. How does the controller manage partition leadership?

---

### Problem 50: Kafka Protocol Internals
**Scenario**: Understand the Kafka wire protocol and how clients communicate with brokers.

**Requirements**:
- Understand request/response protocol
- Explain produce and fetch protocols
- Handle protocol versions
- Understand protocol efficiency

**Questions**:
1. How does the Kafka protocol work?
2. What are the main protocol requests?
3. How does batching work at the protocol level?
4. How to handle protocol version mismatches?

---

### Problem 51: Timestamp Handling and Out-of-Order Events
**Scenario**: Handle message timestamps and process out-of-order events correctly.

**Requirements**:
- Understand log append time vs create time
- Handle out-of-order events
- Implement time-based processing
- Use timestamps for windowing

**Questions**:
1. What are the timestamp types in Kafka?
2. How to handle out-of-order events?
3. When to use log append time vs create time?
4. How do timestamps affect stream processing?

---

### Problem 52: Kafka Admin API
**Scenario**: Programmatically manage Kafka topics, partitions, and configurations.

**Requirements**:
- Create and delete topics programmatically
- Alter topic configurations
- Manage partitions
- Handle admin operations asynchronously

**Questions**:
1. How to use Kafka Admin API?
2. What operations can be performed programmatically?
3. How to handle admin operation failures?
4. What are the best practices for admin operations?

---

### Problem 53: Backpressure Handling
**Scenario**: Handle situations where consumers cannot keep up with producer throughput.

**Requirements**:
- Detect backpressure
- Implement backpressure strategies
- Throttle producers when needed
- Scale consumers dynamically

**Questions**:
1. What causes backpressure in Kafka?
2. How to detect backpressure?
3. What strategies can handle backpressure?
4. How to prevent backpressure?

---

### Problem 54: Kafka Log Retention Internals
**Scenario**: Understand how log retention policies work internally.

**Requirements**:
- Understand time-based retention
- Understand size-based retention
- Handle retention cleanup process
- Optimize retention for different use cases

**Questions**:
1. How does time-based retention work?
2. How does size-based retention work?
3. When does cleanup occur?
4. How to optimize retention policies?

---

### Problem 55: Leader-Follower Replication Protocol
**Scenario**: Understand how replication works at the protocol level between leader and followers.

**Requirements**:
- Understand fetch requests from followers
- Explain how followers catch up
- Handle replication lag
- Understand high watermark and LEO

**Questions**:
1. How do followers replicate data from leaders?
2. What is the high watermark?
3. What is LEO (Log End Offset)?
4. How does a follower catch up after lagging?
