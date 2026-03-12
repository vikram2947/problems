# Kafka Interview Solutions for SDE 2

## Table of Contents
1. [Producer Solutions](#producer-solutions)
2. [Consumer Solutions](#consumer-solutions)
3. [Partitioning & Replication Solutions](#partitioning--replication-solutions)
4. [Consumer Groups & Rebalancing Solutions](#consumer-groups--rebalancing-solutions)
5. [Offset Management Solutions](#offset-management-solutions)
6. [Exactly-Once Semantics Solutions](#exactly-once-semantics-solutions)
7. [Performance & Optimization Solutions](#performance--optimization-solutions)
8. [Fault Tolerance & High Availability Solutions](#fault-tolerance--high-availability-solutions)
9. [Schema Registry & Serialization Solutions](#schema-registry--serialization-solutions)
10. [Stream Processing Solutions](#stream-processing-solutions)
11. [Monitoring & Troubleshooting Solutions](#monitoring--troubleshooting-solutions)
12. [Security & Authentication Solutions](#security--authentication-solutions)
13. [Design & Architecture Solutions](#design--architecture-solutions)
14. [Kafka Internals & Advanced Concepts Solutions](#kafka-internals--advanced-concepts-solutions)
15. [Testing & Operations Solutions](#testing--operations-solutions)

---

## Producer Solutions

### Solution 1: Idempotent Producer Implementation

**How Kafka Achieves Idempotency:**
- Kafka assigns each producer a unique Producer ID (PID) and Sequence Number per partition
- Broker tracks the sequence number for each PID-partition pair
- If a duplicate request arrives (same PID, partition, sequence number), broker rejects it
- This ensures exactly-once semantics at the partition level

**Configuration:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("enable.idempotence", "true");  // Enables idempotent producer
props.put("acks", "all");  // Required for idempotence
props.put("max.in.flight.requests.per.connection", "5");  // Can be up to 5 with idempotence
props.put("retries", Integer.MAX_VALUE);  // Recommended with idempotence

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
```

**What Happens on Producer Restart:**
- Producer gets a new PID from the broker
- Old sequence numbers are invalidated
- New messages get new sequence numbers
- No duplicates occur because PID changes

**Key Points:**
- Idempotence only works within a single producer session
- If producer crashes and restarts, it gets a new PID
- Messages are idempotent per partition, not globally
- Requires `acks=all` and `retries` configured

---

### Solution 2: Custom Partitioner Implementation

**Default Partitioner Behavior:**
- If key is null: uses round-robin across available partitions
- If key exists: `hash(key) % partition_count` determines partition
- Ensures same key always goes to same partition (unless partition count changes)

**Custom Partitioner Implementation:**
```java
public class CustomerPartitioner implements Partitioner {
    
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, 
                        Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        
        if (keyBytes == null) {
            // Round-robin for null keys
            return new Random().nextInt(numPartitions);
        }
        
        // Extract customer ID from key
        String customerId = (String) key;
        
        // Business logic: route by customer ID
        // Example: route all messages from customer "CUST_001" to partition 0
        if (customerId.startsWith("CUST_001")) {
            return 0;
        }
        
        // Default: hash-based partitioning
        return Math.abs(customerId.hashCode()) % numPartitions;
    }
    
    @Override
    public void close() {
        // Cleanup if needed
    }
    
    @Override
    public void configure(Map<String, ?> configs) {
        // Configure partitioner if needed
    }
}

// Usage
props.put("partitioner.class", "com.example.CustomerPartitioner");
```

**Handling Partition Changes:**
- When partitions are added/removed, keys may map to different partitions
- This breaks ordering guarantees
- Solution: Use consistent hashing or avoid changing partition count

**Key Points:**
- Custom partitioners must be deterministic
- Same key must always map to same partition
- Handle null keys appropriately
- Consider partition count changes

---

### Solution 3: Producer Batching and Compression

**Batching Configuration:**
```java
props.put("batch.size", "16384");  // 16KB batch size
props.put("linger.ms", "10");  // Wait up to 10ms to fill batch
props.put("buffer.memory", "33554432");  // 32MB buffer
props.put("compression.type", "snappy");  // or "gzip", "lz4", "zstd"
```

**How `linger.ms` Works:**
- Producer waits up to `linger.ms` milliseconds before sending a batch
- If batch fills before `linger.ms`, it sends immediately
- Balances latency vs throughput
- `linger.ms=0` = send immediately (low latency, lower throughput)
- `linger.ms=10` = wait up to 10ms (higher throughput, slightly higher latency)

**Compression Trade-offs:**

| Compression | Ratio | CPU | Speed |
|------------|-------|-----|-------|
| None | 1x | Low | Fastest |
| Gzip | High | High | Slow |
| Snappy | Medium | Medium | Fast |
| Lz4 | Medium-High | Low | Very Fast |
| Zstd | High | Medium | Fast |

**Choosing Batch Size:**
- Larger batches = better throughput but higher latency
- Smaller batches = lower latency but lower throughput
- Rule of thumb: 16KB-64KB for most use cases
- Consider network MTU (typically 1500 bytes)

**Key Points:**
- Batching improves throughput significantly
- Compression reduces network bandwidth
- Balance latency requirements with throughput needs
- Monitor producer metrics to tune

---

### Solution 4: Producer Error Handling and Retries

**Retriable vs Non-Retriable Errors:**

**Retriable Errors:**
- `LEADER_NOT_AVAILABLE`
- `NOT_LEADER_FOR_PARTITION`
- `REQUEST_TIMED_OUT`
- `NETWORK_EXCEPTION`
- `CORRUPT_MESSAGE_EXCEPTION`

**Non-Retriable Errors:**
- `INVALID_TOPIC_EXCEPTION`
- `RECORD_TOO_LARGE`
- `INVALID_CONFIG`
- `UNKNOWN_TOPIC_OR_PARTITION` (if topic doesn't exist)

**Error Handling Implementation:**
```java
public class ProducerCallback implements Callback {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (exception == null) {
            System.out.println("Message sent successfully: " + metadata);
        } else {
            if (exception instanceof RetriableException) {
                // Retriable error - Kafka will retry automatically
                System.err.println("Retriable error: " + exception.getMessage());
            } else {
                // Non-retriable error - handle manually
                System.err.println("Non-retriable error: " + exception.getMessage());
                // Log to dead letter queue, alert, etc.
            }
        }
    }
}

// Usage
producer.send(record, new ProducerCallback());
```

**Exponential Backoff:**
- Kafka handles retries internally with exponential backoff
- Configured via `retry.backoff.ms` (default: 100ms)
- Backoff increases exponentially: 100ms, 200ms, 400ms, etc.

**`max.in.flight.requests.per.connection` Impact:**
- Controls how many unacknowledged requests can be in flight
- With idempotence: can be up to 5 (maintains ordering)
- Without idempotence: must be 1 to guarantee ordering
- Higher values = better throughput but potential reordering

**Key Points:**
- Configure appropriate retry count
- Handle non-retriable errors separately
- Use callbacks for error handling
- Monitor producer error rates

---

## Consumer Solutions

### Solution 5: Consumer Group Rebalancing

**Rebalancing Strategies:**

1. **Range Assignor (Default):**
   - Divides partitions into ranges
   - Example: 3 partitions, 2 consumers → C1: [0,1], C2: [2]
   - Can lead to uneven distribution

2. **RoundRobin Assignor:**
   - Distributes partitions in round-robin fashion
   - Better balance but may break key-based ordering

3. **Sticky Assignor:**
   - Minimizes partition reassignment
   - Tries to keep existing assignments
   - Reduces rebalancing overhead

4. **Cooperative Sticky Assignor (Recommended):**
   - Incremental rebalancing
   - Consumers can continue processing during rebalance
   - Reduces stop-the-world pauses

**Cooperative Rebalancing Configuration:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("partition.assignment.strategy", 
    "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
props.put("enable.auto.commit", "false");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));
```

**Causes of Rebalancing:**
- Consumer joins group
- Consumer leaves group (graceful shutdown or crash)
- Consumer doesn't send heartbeat in time (`session.timeout.ms`)
- New partitions added to subscribed topics
- Coordinator changes

**Minimizing Rebalancing:**
- Use cooperative rebalancing
- Increase `session.timeout.ms` (but balance with failure detection)
- Increase `heartbeat.interval.ms`
- Ensure consumers send heartbeats regularly
- Use static membership (Kafka 2.3+)

**Key Points:**
- Rebalancing pauses consumption
- Cooperative rebalancing reduces pause time
- Configure timeouts appropriately
- Monitor rebalancing frequency

---

### Solution 6: Manual Offset Management

**Manual Commit Implementation:**
```java
Properties props = new Properties();
props.put("enable.auto.commit", "false");  // Disable auto-commit
props.put("group.id", "my-group");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        
        for (ConsumerRecord<String, String> record : records) {
            // Process record
            processRecord(record);
        }
        
        // Commit offsets after processing batch
        try {
            consumer.commitSync();  // Synchronous commit
            // or
            consumer.commitAsync();  // Asynchronous commit
        } catch (CommitFailedException e) {
            // Handle commit failure
            System.err.println("Commit failed: " + e);
        }
    }
} finally {
    consumer.close();
}
```

**Synchronous vs Asynchronous Commit:**

**Synchronous (`commitSync`):**
- Blocks until commit completes
- Guarantees commit before continuing
- Slower but safer
- Use for critical offsets

**Asynchronous (`commitAsync`):**
- Non-blocking
- Better performance
- May fail silently
- Use with callback for error handling

**Commit with Callback:**
```java
consumer.commitAsync(new OffsetCommitCallback() {
    @Override
    public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, 
                          Exception exception) {
        if (exception != null) {
            System.err.println("Commit failed: " + exception);
            // Retry or handle error
        }
    }
});
```

**Handling Commit Failures:**
- If commit fails, offsets aren't updated
- Next poll may return same records (at-least-once)
- Implement idempotent processing
- Consider retry logic for commit failures

**Key Points:**
- Manual commits give more control
- Commit after processing, not before
- Handle commit failures appropriately
- Balance between sync and async commits

---

### Solution 7: Consumer Lag Monitoring

**Calculating Consumer Lag:**
```java
// Get consumer group metadata
ConsumerGroupDescription groupDesc = adminClient
    .describeConsumerGroups(Collections.singletonList("my-group"))
    .all()
    .get()
    .get("my-group");

// Get partition offsets
Map<TopicPartition, OffsetSpec> partitionSpecs = new HashMap<>();
for (TopicPartition tp : consumer.assignment()) {
    partitionSpecs.put(tp, OffsetSpec.latest());
}

Map<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo> endOffsets = 
    adminClient.listOffsets(partitionSpecs).all().get();

// Get consumer group offsets
Map<TopicPartition, OffsetAndMetadata> committedOffsets = 
    consumer.committed(consumer.assignment());

// Calculate lag
for (TopicPartition tp : consumer.assignment()) {
    long endOffset = endOffsets.get(tp).offset();
    long committedOffset = committedOffsets.get(tp).offset();
    long lag = endOffset - committedOffset;
    
    System.out.println("Partition " + tp + " lag: " + lag);
}
```

**Using Kafka Consumer Lag Metrics:**
```java
// Enable JMX metrics
props.put("metric.reporters", "org.apache.kafka.common.metrics.JmxReporter");

// Access metrics via JMX
// records-lag-max: Maximum lag across all partitions
// records-lag-avg: Average lag
// records-lag: Lag per partition
```

**Alerting Implementation:**
```java
public class LagMonitor {
    private static final long LAG_THRESHOLD = 10000;
    
    public void checkLag(KafkaConsumer<String, String> consumer) {
        Map<TopicPartition, Long> lags = getConsumerLag(consumer);
        
        for (Map.Entry<TopicPartition, Long> entry : lags.entrySet()) {
            if (entry.getValue() > LAG_THRESHOLD) {
                alert("High lag detected: " + entry.getKey() + 
                      " lag = " + entry.getValue());
            }
        }
    }
}
```

**Causes of High Lag:**
- Consumer processing slower than producer
- Consumer crashes/restarts
- Network issues
- Under-provisioned consumers
- Slow downstream systems

**Reducing Lag:**
- Increase consumer instances
- Optimize processing logic
- Increase `fetch.min.bytes` and `fetch.max.wait.ms`
- Parallelize processing
- Scale downstream systems

**Key Points:**
- Monitor lag continuously
- Set appropriate thresholds
- Alert on sustained high lag
- Investigate root causes

---

### Solution 8: Consumer with Multiple Topics

**Multiple Topic Subscription:**
```java
consumer.subscribe(Arrays.asList("topic1", "topic2", "topic3"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
    for (ConsumerRecord<String, String> record : records) {
        String topic = record.topic();
        
        // Route based on topic
        switch (topic) {
            case "topic1":
                processTopic1(record);
                break;
            case "topic2":
                processTopic2(record);
                break;
            case "topic3":
                processTopic3(record);
                break;
        }
    }
}
```

**Different Consumer Groups:**
```java
// Consumer 1 for topic1
KafkaConsumer<String, String> consumer1 = new KafkaConsumer<>(props1);
consumer1.subscribe(Arrays.asList("topic1"));

// Consumer 2 for topic2
KafkaConsumer<String, String> consumer2 = new KafkaConsumer<>(props2);
consumer2.subscribe(Arrays.asList("topic2"));
```

**Ordering Across Topics:**
- Kafka doesn't guarantee ordering across topics
- Each topic maintains ordering within partitions
- If cross-topic ordering needed:
  - Use single topic with routing key
  - Implement application-level ordering
  - Use Kafka Streams for complex ordering

**Key Points:**
- One consumer can subscribe to multiple topics
- Each topic maintains its own partition ordering
- Use different consumer groups for independent processing
- Cross-topic ordering requires application logic

---

## Partitioning & Replication Solutions

### Solution 9: Partition Count Optimization

**Calculating Partition Count:**
```
Partitions needed = (Producer Throughput × Replication Factor) / Consumer Throughput

Example:
- Producer: 1M msgs/sec
- Consumer: 50K msgs/sec per partition
- Replication Factor: 3
- Partitions = (1,000,000 × 3) / 50,000 = 60 partitions
```

**Factors to Consider:**
- Throughput requirements (producer and consumer)
- Consumer parallelism (one consumer per partition)
- Replication factor
- Broker capacity
- Zookeeper/Kafka controller limits

**Downsides of Too Many Partitions:**
- More file handles (each partition = multiple files)
- More memory overhead
- Slower leader election
- More metadata to manage
- Zookeeper/Kafka controller overhead

**Changing Partition Count:**
```bash
# Increase partitions (can only increase, not decrease)
kafka-topics.sh --alter --topic my-topic \
  --partitions 10 \
  --bootstrap-server localhost:9092
```

**Best Practices:**
- Start with fewer partitions, scale up as needed
- Rule of thumb: 1-2 partitions per broker initially
- Consider consumer parallelism needs
- Monitor partition-level metrics

**Key Points:**
- More partitions = more parallelism
- But also more overhead
- Balance based on use case
- Can only increase, not decrease

---

### Solution 10: Replication Factor and ISR

**Replication Factor Configuration:**
```bash
# Create topic with replication factor 3
kafka-topics.sh --create --topic my-topic \
  --partitions 6 \
  --replication-factor 3 \
  --bootstrap-server localhost:9092
```

**Minimum Replication Factor:**
- For high availability: replication factor ≥ 3
- Can survive (RF-1)/2 broker failures
- RF=3 can survive 1 broker failure
- RF=5 can survive 2 broker failures

**In-Sync Replicas (ISR):**
- Replicas that are caught up with the leader
- Determined by `replica.lag.time.max.ms` (default: 10 seconds)
- If replica falls behind, removed from ISR
- Leader only writes to ISR replicas

**Checking ISR:**
```bash
kafka-topics.sh --describe --topic my-topic \
  --bootstrap-server localhost:9092

# Output shows ISR for each partition
# Topic: my-topic Partition: 0 Leader: 1 Replicas: 1,2,3 Isr: 1,2,3
```

**Under-Replicated Partitions:**
- When ISR size < replication factor
- Indicates broker issues or network problems
- Monitor via JMX: `UnderReplicatedPartitions`
- Investigate broker health, network, disk I/O

**Key Points:**
- Higher replication factor = better availability
- ISR ensures consistency
- Monitor ISR size
- Handle under-replicated partitions promptly

---

### Solution 11: Leader Election and Unclean Leader Election

**Leader Election Process:**
1. Broker detects leader failure (via Zookeeper/KRaft)
2. Controller selects new leader from ISR
3. New leader starts serving requests
4. Followers catch up to new leader

**Unclean Leader Election:**
- When no ISR replicas available
- Kafka can elect a non-ISR replica as leader
- May result in data loss (non-ISR replica may be behind)
- Controlled by `unclean.leader.election.enable`

**Configuration:**
```bash
# Disable unclean leader election (default: false)
unclean.leader.election.enable=false

# If enabled, may lose data but maintain availability
unclean.leader.election.enable=true
```

**Trade-offs:**

**`unclean.leader.election.enable=false`:**
- No data loss
- Partition unavailable if no ISR replicas
- Better for data consistency

**`unclean.leader.election.enable=true`:**
- Maintains availability
- Risk of data loss
- Better for high availability requirements

**Preventing Data Loss:**
- Use `min.insync.replicas` (e.g., 2)
- Ensure replication factor > min.insync.replicas
- Monitor ISR size
- Keep brokers healthy

**Key Points:**
- Leader election happens automatically
- Unclean election trades consistency for availability
- Use `min.insync.replicas` to prevent data loss
- Monitor leader elections

---

## Consumer Groups & Rebalancing Solutions

### Solution 12: Static vs Dynamic Membership

**Static Membership (Kafka 2.3+):**
- Consumer instance has persistent ID
- Can restart without triggering rebalance
- Reduces rebalancing overhead
- Better for long-running consumers

**Configuration:**
```java
props.put("group.instance.id", "consumer-1");  // Static member ID
props.put("session.timeout.ms", "45000");
```

**Dynamic Membership:**
- Default behavior
- Consumer ID changes on restart
- Triggers rebalance on restart
- Better for ephemeral consumers

**When to Use Static Membership:**
- Long-running consumers
- Frequent restarts (deployments, health checks)
- Want to minimize rebalancing
- Can manage member IDs

**When to Use Dynamic Membership:**
- Ephemeral consumers
- Ad-hoc processing
- Don't want to manage IDs
- Default for most cases

**Key Points:**
- Static membership reduces rebalancing
- Requires managing member IDs
- Use for production long-running consumers
- Dynamic is simpler for development

---

### Solution 13: Consumer Group Coordinator

**Coordinator Selection:**
- Selected based on hash of group.id
- Same group always uses same coordinator
- Coordinator manages group state

**Finding Coordinator:**
```java
// Coordinator is determined automatically
// Same group.id always uses same coordinator broker
```

**Coordinator Failover:**
- If coordinator fails, new coordinator elected
- Consumers reconnect to new coordinator
- Group state recovered from internal topic `__consumer_offsets`
- Minimal impact on consumers

**Group Metadata:**
- Stored in `__consumer_offsets` topic
- Includes: member assignments, offsets, group state
- Replicated across brokers
- Survives coordinator failures

**Key Points:**
- Coordinator manages group lifecycle
- Automatically selected and failed over
- State persisted in internal topic
- Transparent to applications

---

## Offset Management Solutions

### Solution 14: Offset Reset Strategies

**Offset Reset Scenarios:**
- Consumer starts with no committed offset
- Committed offset is out of range (deleted)
- Consumer group is reset

**Reset Strategies:**

**`earliest`:**
- Start from beginning of partition
- Process all available messages
- Use for reprocessing

**`latest`:**
- Start from end of partition
- Only process new messages
- Use for real-time processing

**Configuration:**
```java
props.put("auto.offset.reset", "earliest");  // or "latest", "none"
```

**Custom Offset Reset:**
```java
// Seek to specific offset
consumer.seek(new TopicPartition("my-topic", 0), 100L);

// Seek to beginning
consumer.seekToBeginning(Collections.singletonList(
    new TopicPartition("my-topic", 0)));

// Seek to end
consumer.seekToEnd(Collections.singletonList(
    new TopicPartition("my-topic", 0)));
```

**Handling Offset Out of Range:**
- If `auto.offset.reset=none`, throws exception
- Otherwise uses configured strategy
- Monitor for offset resets

**Key Points:**
- `earliest` = from beginning
- `latest` = from end (default)
- `none` = throw exception
- Can manually seek to specific offsets

---

### Solution 15: Offset Storage (Internal vs External)

**Internal Storage (Kafka):**
- Stored in `__consumer_offsets` topic
- Managed automatically
- Simple to use
- Default approach

**External Storage (Database):**
```java
public class DatabaseOffsetStore {
    public void saveOffset(String groupId, TopicPartition partition, long offset) {
        // Save to database
        db.execute("INSERT INTO offsets (group_id, topic, partition, offset) " +
                   "VALUES (?, ?, ?, ?) ON DUPLICATE KEY UPDATE offset = ?",
                   groupId, partition.topic(), partition.partition(), offset, offset);
    }
    
    public long getOffset(String groupId, TopicPartition partition) {
        // Retrieve from database
        return db.query("SELECT offset FROM offsets WHERE group_id = ? " +
                       "AND topic = ? AND partition = ?",
                       groupId, partition.topic(), partition.partition());
    }
}

// Use in consumer
consumer.subscribe(Arrays.asList("my-topic"));
consumer.poll(Duration.ZERO);  // Get assignment
for (TopicPartition partition : consumer.assignment()) {
    long offset = offsetStore.getOffset("my-group", partition);
    consumer.seek(partition, offset);
}
```

**Advantages of External Storage:**
- Can share offsets across different systems
- More control over offset management
- Can implement custom logic
- Easier to audit and debug

**Advantages of Internal Storage:**
- Simpler
- Automatic management
- Integrated with consumer groups
- No additional infrastructure

**When to Use External:**
- Need to share offsets across systems
- Custom offset logic required
- Audit requirements
- Multi-system coordination

**Key Points:**
- Internal is simpler, external gives more control
- Choose based on requirements
- External requires more implementation
- Both support exactly-once semantics

---

## Exactly-Once Semantics Solutions

### Solution 16: Transactional Producer

**Transaction Configuration:**
```java
props.put("transactional.id", "my-transactional-id");
props.put("enable.idempotence", "true");  // Required
props.put("acks", "all");  // Required

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Initialize transactions
producer.initTransactions();

try {
    producer.beginTransaction();
    
    // Send multiple messages atomically
    producer.send(new ProducerRecord<>("topic1", "key1", "value1"));
    producer.send(new ProducerRecord<>("topic2", "key2", "value2"));
    
    // Commit transaction
    producer.commitTransaction();
} catch (Exception e) {
    // Abort on error
    producer.abortTransaction();
    throw e;
}
```

**Transaction Coordinator:**
- Manages transaction state
- One coordinator per transactional.id
- Tracks transaction lifecycle
- Ensures atomicity

**Transaction Lifecycle:**
1. `initTransactions()` - gets PID from coordinator
2. `beginTransaction()` - starts new transaction
3. `send()` - messages added to transaction
4. `commitTransaction()` - atomically commits all messages
5. `abortTransaction()` - rolls back all messages

**Handling Timeouts:**
```java
props.put("transaction.timeout.ms", "60000");  // 60 seconds

// If transaction times out, coordinator aborts it
// Producer must handle ProducerFencedException
```

**Key Points:**
- Requires `transactional.id`
- All messages in transaction commit atomically
- Coordinator manages transaction state
- Handle timeouts appropriately

---

### Solution 17: Exactly-Once Processing

**Transactional Consumer:**
```java
props.put("isolation.level", "read_committed");  // Only read committed messages
props.put("enable.auto.commit", "false");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("input-topic"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
    producer.beginTransaction();
    
    try {
        for (ConsumerRecord<String, String> record : records) {
            // Process and produce to output topic
            String processed = process(record.value());
            producer.send(new ProducerRecord<>("output-topic", record.key(), processed));
        }
        
        // Commit consumer offsets and producer messages atomically
        Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
        for (TopicPartition partition : records.partitions()) {
            offsets.put(partition, new OffsetAndMetadata(
                records.records(partition).get(records.records(partition).size() - 1).offset() + 1));
        }
        
        producer.sendOffsetsToTransaction(offsets, consumer.groupMetadata());
        producer.commitTransaction();
    } catch (Exception e) {
        producer.abortTransaction();
        throw e;
    }
}
```

**End-to-End Exactly-Once:**
1. Use transactional producer
2. Use `read_committed` consumer
3. Send offsets to transaction
4. Commit atomically

**Performance Implications:**
- Higher latency (coordination overhead)
- Lower throughput (more round trips)
- More complex error handling
- Use only when needed

**Key Points:**
- Requires both transactional producer and consumer
- Offsets sent as part of transaction
- Atomic commit ensures exactly-once
- Performance trade-off for correctness

---

## Performance & Optimization Solutions

### Solution 18: Producer Throughput Optimization

**Optimization Configuration:**
```java
props.put("batch.size", "65536");  // 64KB batches
props.put("linger.ms", "10");  // Wait 10ms for batching
props.put("compression.type", "snappy");  // Fast compression
props.put("acks", "1");  // Faster than "all", still durable
props.put("buffer.memory", "67108864");  // 64MB buffer
props.put("max.in.flight.requests.per.connection", "5");  // With idempotence
```

**`acks` Impact:**
- `acks=0`: Fire and forget, highest throughput, no guarantee
- `acks=1`: Leader acknowledgment, good throughput, some durability
- `acks=all`: All replicas, lower throughput, highest durability

**Optimal Batch Size:**
- Start with 16KB-32KB
- Increase if latency allows
- Monitor producer metrics
- Consider network MTU (1500 bytes)

**Key Points:**
- Batch size and linger.ms critical
- Compression improves network efficiency
- acks=1 good balance
- Monitor and tune based on metrics

---

### Solution 19: Consumer Throughput Optimization

**Optimization Configuration:**
```java
props.put("fetch.min.bytes", "1048576");  // 1MB minimum fetch
props.put("fetch.max.wait.ms", "500");  // Wait up to 500ms
props.put("max.partition.fetch.bytes", "10485760");  // 10MB per partition
props.put("max.poll.records", "1000");  // Process up to 1000 records per poll
```

**Parallel Processing:**
```java
// Use thread pool for parallel processing
ExecutorService executor = Executors.newFixedThreadPool(10);

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
    for (ConsumerRecord<String, String> record : records) {
        executor.submit(() -> processRecord(record));
    }
    
    // Commit after all processing completes
    consumer.commitSync();
}
```

**Common Bottlenecks:**
- Slow processing logic
- Synchronous I/O operations
- Under-provisioned consumers
- Network latency
- Small fetch sizes

**Key Points:**
- Increase fetch sizes
- Parallelize processing
- Optimize processing logic
- Scale consumers horizontally

---

### Solution 20: Topic-Level Configuration Tuning

**Retention Configuration:**
```bash
# Set retention to 7 days
kafka-configs.sh --alter --entity-type topics --entity-name my-topic \
  --add-config retention.ms=604800000 \
  --bootstrap-server localhost:9092

# Or retention by size
kafka-configs.sh --alter --entity-type topics --entity-name my-topic \
  --add-config retention.bytes=1073741824 \
  --bootstrap-server localhost:9092
```

**Segment Configuration:**
```bash
# Larger segments = fewer files, but longer retention
kafka-configs.sh --alter --entity-type topics --entity-name my-topic \
  --add-config segment.bytes=1073741824 \
  --bootstrap-server localhost:9092
```

**Index Intervals:**
```bash
# More frequent indexing = faster seeks, more overhead
kafka-configs.sh --alter --entity-type topics --entity-name my-topic \
  --add-config index.interval.bytes=4096 \
  --bootstrap-server localhost:9092
```

**Use Case Optimizations:**

**Streaming (Low Latency):**
- Smaller segments
- Shorter retention
- More frequent indexing

**Batch Processing:**
- Larger segments
- Longer retention
- Less frequent indexing

**Key Points:**
- Tune based on use case
- Balance retention vs storage
- Optimize segment size
- Monitor disk usage

---

## Fault Tolerance & High Availability Solutions

### Solution 21: Broker Failure Handling

**Cluster Design:**
- Minimum 3 brokers for high availability
- Replication factor ≥ 3
- Distribute replicas across brokers
- Monitor broker health

**Surviving Failures:**
- RF=3 can survive 1 broker failure
- RF=5 can survive 2 broker failures
- General: can survive (RF-1)/2 failures

**Partition Impact:**
- When broker fails, partitions it leads become unavailable
- New leader elected from ISR
- Followers catch up to new leader
- Temporary unavailability during election

**Monitoring:**
```bash
# Check under-replicated partitions
kafka-topics.sh --describe --under-replicated-partitions \
  --bootstrap-server localhost:9092

# Check broker status
kafka-broker-api-versions.sh --bootstrap-server localhost:9092
```

**Key Points:**
- Design for expected failures
- Monitor broker health
- Ensure adequate replication
- Test failure scenarios

---

### Solution 22: Network Partition Handling

**Split-Brain Scenario:**
- Network partition divides cluster
- Each side thinks other is down
- Can lead to two leaders (with unclean election)
- Data inconsistency possible

**Prevention:**
```bash
# Require minimum ISR replicas
kafka-configs.sh --alter --entity-type topics --entity-name my-topic \
  --add-config min.insync.replicas=2 \
  --bootstrap-server localhost:9092
```

**How `min.insync.replicas` Works:**
- Producer requires at least N replicas in ISR
- If ISR < min.insync.replicas, producer fails
- Prevents writes during network partition
- Ensures consistency

**Recovery:**
- When partition heals, brokers reconnect
- Out-of-sync replicas catch up
- ISR restored
- Normal operation resumes

**Key Points:**
- Network partitions are rare but critical
- Use min.insync.replicas to prevent inconsistency
- Monitor ISR size
- Design for partition scenarios

---

### Solution 23: Data Loss Prevention

**Common Causes:**
- Unclean leader election
- Insufficient replication
- Producer with acks=0
- Disk failures without replication
- Bugs in application logic

**Prevention Strategies:**
1. **Replication:**
   ```bash
   # Use replication factor ≥ 3
   --replication-factor 3
   ```

2. **Producer Acks:**
   ```java
   props.put("acks", "all");  // Wait for all replicas
   ```

3. **Unclean Leader Election:**
   ```bash
   unclean.leader.election.enable=false
   ```

4. **Min ISR:**
   ```bash
   min.insync.replicas=2  # With RF=3
   ```

5. **Monitoring:**
   - Monitor under-replicated partitions
   - Alert on ISR size drops
   - Track producer errors

**Detection:**
- Compare message counts across replicas
- Monitor offset gaps
- Audit logs
- Compare producer vs consumer counts

**Key Points:**
- Multiple layers of protection
- Configure appropriately
- Monitor continuously
- Test failure scenarios

---

## Schema Registry & Serialization Solutions

### Solution 24: Schema Evolution

**Compatibility Types:**

**Backward Compatible:**
- New schema can read old data
- Add optional fields
- Remove required fields (not recommended)

**Forward Compatible:**
- Old schema can read new data
- Remove optional fields
- Add required fields (not recommended)

**Full Compatible:**
- Both backward and forward compatible
- Only add optional fields
- Safest approach

**Schema Registry Configuration:**
```bash
# Set compatibility level
curl -X PUT http://localhost:8081/config/my-topic-value \
  -H "Content-Type: application/json" \
  -d '{"compatibility": "BACKWARD"}'
```

**Schema Versioning:**
```java
// Producer with schema
Schema schema = new Schema.Parser().parse(schemaString);
GenericRecord record = new GenericData.Record(schema);
record.put("field1", "value1");

// Schema Registry handles versioning automatically
```

**Handling Schema Changes:**
1. Update schema with backward compatibility
2. Deploy new consumers (can read old data)
3. Deploy new producers (write new format)
4. Old consumers eventually retire

**Key Points:**
- Plan for schema evolution
- Use appropriate compatibility
- Test schema changes
- Version schemas properly

---

### Solution 25: Avro Serialization

**Avro Producer:**
```java
props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://localhost:8081");

KafkaProducer<String, GenericRecord> producer = new KafkaProducer<>(props);

Schema schema = new Schema.Parser().parse(schemaString);
GenericRecord record = new GenericData.Record(schema);
record.put("name", "John");
record.put("age", 30);

producer.send(new ProducerRecord<>("my-topic", "key", record));
```

**Schema Registry Caching:**
- Client caches schemas locally
- Reduces registry calls
- Handles schema lookups efficiently
- Configurable cache size

**Handling Registry Unavailability:**
- Cache allows operation without registry
- New schemas fail if registry down
- Existing schemas work from cache
- Implement retry logic

**Why Avro:**
- Compact binary format
- Schema evolution support
- Language agnostic
- Better than JSON for Kafka

**Key Points:**
- Avro is efficient and evolvable
- Schema Registry manages versions
- Caching improves performance
- Handle registry failures

---

## Stream Processing Solutions

### Solution 26: Kafka Streams Application

**Basic Stream Processing:**
```java
StreamsBuilder builder = new StreamsBuilder();

// Read from source topic
KStream<String, String> source = builder.stream("input-topic");

// Transform
KStream<String, String> transformed = source
    .mapValues(value -> value.toUpperCase())
    .filter((key, value) -> value.length() > 10);

// Write to sink topic
transformed.to("output-topic");

// Build and start
KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

**Exactly-Once Processing:**
```java
props.put("processing.guarantee", "exactly_once_v2");
// Ensures exactly-once semantics
```

**Stateful Operations:**
```java
// Aggregate
KTable<String, Long> counts = source
    .groupByKey()
    .count();

// Join
KStream<String, String> joined = stream1.join(
    stream2,
    (value1, value2) -> value1 + "-" + value2,
    JoinWindows.of(Duration.ofMinutes(5))
);
```

**Key Points:**
- Kafka Streams for stream processing
- Supports exactly-once
- Stateful operations supported
- Integrated with Kafka

---

### Solution 27: Stream-Table Join

**KStream vs KTable:**
- **KStream**: Append-only stream of events
- **KTable**: Changelog stream (latest value per key)

**Stream-Table Join:**
```java
// Create KTable from topic
KTable<String, String> table = builder.table("lookup-topic");

// Join stream with table
KStream<String, String> enriched = stream.leftJoin(
    table,
    (streamValue, tableValue) -> streamValue + "|" + tableValue
);
```

**Handling Late Data:**
- Kafka Streams handles out-of-order events
- Configurable grace period
- State stores maintain join state

**Key Points:**
- KTable maintains latest value per key
- Joins enrich streams with lookup data
- Handles late-arriving data
- Stateful operation

---

### Solution 28: Windowing and Aggregation

**Tumbling Windows:**
```java
KStream<String, String> windowed = source
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
    .count()
    .toStream();
```

**Window Types:**
- **Tumbling**: Non-overlapping, fixed-size
- **Hopping**: Overlapping, fixed-size
- **Sliding**: Based on record timestamps
- **Session**: Activity-based

**Late Data:**
```java
// Configure grace period
TimeWindows.of(Duration.ofMinutes(5))
    .grace(Duration.ofSeconds(30));
```

**Key Points:**
- Windows group events by time
- Different window types for different needs
- Grace period handles late data
- Stateful aggregation

---

## Monitoring & Troubleshooting Solutions

### Solution 29: Consumer Lag Alerting

**Monitoring Implementation:**
```java
public class LagMonitor {
    public void monitorLag(String groupId) {
        Map<TopicPartition, Long> lags = getConsumerLag(groupId);
        
        for (Map.Entry<TopicPartition, Long> entry : lags.entrySet()) {
            if (entry.getValue() > THRESHOLD) {
                alert(entry.getKey(), entry.getValue());
            }
        }
    }
}
```

**Normal vs Abnormal Lag:**
- Normal: Lag < 1000 messages
- Warning: Lag 1000-10000
- Critical: Lag > 10000
- Varies by use case

**Reducing False Positives:**
- Use time-based thresholds (sustained lag)
- Consider partition count
- Account for processing time
- Set appropriate thresholds

**Key Points:**
- Monitor continuously
- Set realistic thresholds
- Alert on sustained lag
- Investigate root causes

---

### Solution 30: Broker Metrics Monitoring

**Key Metrics:**
- **Throughput**: Messages/sec in/out
- **Latency**: Request processing time
- **Replication**: Under-replicated partitions
- **Disk**: Disk usage, I/O
- **Network**: Bytes in/out
- **JVM**: GC, heap usage

**Monitoring Tools:**
- JMX metrics
- Kafka Exporter + Prometheus
- Confluent Control Center
- Custom dashboards

**Critical Alerts:**
- Under-replicated partitions > 0
- Disk usage > 80%
- Broker down
- High request latency
- GC pauses

**Key Points:**
- Monitor key metrics
- Set up dashboards
- Alert on critical issues
- Proactive monitoring

---

### Solution 31: Debugging Message Ordering Issues

**Ordering Guarantees:**
- Within a partition: guaranteed
- Across partitions: not guaranteed
- With key: same key → same partition → ordered

**Common Causes:**
- Multiple partitions without keys
- Producer retries with max.in.flight > 1 (without idempotence)
- Consumer rebalancing
- Multiple producers

**Debugging:**
```java
// Add timestamps and sequence numbers
record.put("timestamp", System.currentTimeMillis());
record.put("sequence", sequenceNumber++);

// Log partition assignment
consumer.assignment().forEach(tp -> 
    logger.info("Assigned partition: " + tp));

// Check ordering in consumer
long lastOffset = -1;
for (ConsumerRecord r : records) {
    if (r.offset() <= lastOffset) {
        logger.error("Out of order: " + r);
    }
    lastOffset = r.offset();
}
```

**Fixes:**
- Use keys for ordering
- Enable idempotence
- Use single partition if needed
- Handle rebalancing gracefully

**Key Points:**
- Ordering only within partition
- Use keys for related messages
- Enable idempotence
- Test ordering scenarios

---

## Security & Authentication Solutions

### Solution 32: SASL Authentication Setup

**SASL/PLAIN Configuration (Broker):**
```properties
# server.properties
listeners=SASL_PLAINTEXT://localhost:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# JAAS config
listener.name.sasl_plaintext.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="admin" \
  password="admin-secret" \
  user_admin="admin-secret" \
  user_producer="producer-secret" \
  user_consumer="consumer-secret";
```

**Client Configuration:**
```java
props.put("security.protocol", "SASL_PLAINTEXT");
props.put("sasl.mechanism", "PLAIN");
props.put("sasl.jaas.config", 
    "org.apache.kafka.common.security.plain.PlainLoginModule required " +
    "username=\"producer\" " +
    "password=\"producer-secret\";");
```

**Other SASL Mechanisms:**
- **SCRAM-SHA-256/512**: More secure, password-based
- **GSSAPI**: Kerberos integration
- **OAUTHBEARER**: OAuth 2.0

**Key Points:**
- SASL provides authentication
- PLAIN is simple but less secure
- Use SCRAM or GSSAPI for production
- Secure credential storage

---

### Solution 33: ACL Authorization

**ACL Configuration:**
```bash
# Allow producer to write
kafka-acls.sh --add --allow-principal User:producer \
  --operation Write --topic my-topic \
  --bootstrap-server localhost:9092

# Allow consumer to read
kafka-acls.sh --add --allow-principal User:consumer \
  --operation Read --topic my-topic \
  --group my-group \
  --bootstrap-server localhost:9092

# List ACLs
kafka-acls.sh --list --topic my-topic \
  --bootstrap-server localhost:9092
```

**ACL Operations:**
- Read, Write, Create, Delete, Alter, Describe
- Cluster operations
- Group operations

**Best Practices:**
- Principle of least privilege
- Use groups for common permissions
- Regular audits
- Document ACLs

**Key Points:**
- Fine-grained access control
- Per-topic, per-group permissions
- Regular audits required
- Document access patterns

---

### Solution 34: SSL/TLS Encryption

**Broker SSL Configuration:**
```properties
# server.properties
listeners=SSL://localhost:9093
security.inter.broker.protocol=SSL
ssl.keystore.location=/path/to/kafka.server.keystore.jks
ssl.keystore.password=test1234
ssl.key.password=test1234
ssl.truststore.location=/path/to/kafka.server.truststore.jks
ssl.truststore.password=test1234
```

**Client SSL Configuration:**
```java
props.put("security.protocol", "SSL");
props.put("ssl.truststore.location", "/path/to/client.truststore.jks");
props.put("ssl.truststore.password", "test1234");
props.put("ssl.keystore.location", "/path/to/client.keystore.jks");
props.put("ssl.keystore.password", "test1234");
props.put("ssl.key.password", "test1234");
```

**One-way vs Two-way SSL:**
- **One-way**: Client verifies server (common)
- **Two-way**: Mutual authentication (more secure)

**Certificate Management:**
- Use CA-signed certificates
- Implement certificate rotation
- Monitor expiration
- Secure key storage

**Key Points:**
- SSL encrypts data in transit
- Two-way SSL for mutual auth
- Proper certificate management critical
- Monitor expiration

---

## Design & Architecture Solutions

### Solution 35: Multi-Datacenter Kafka Setup

**Replication Strategies:**

1. **MirrorMaker 2.0:**
   ```bash
   # Replicate topics across datacenters
   # Handles offset translation
   # Supports active-active
   ```

2. **Confluent Replicator:**
   - Enterprise solution
   - Better monitoring
   - Schema Registry replication

**Challenges:**
- Network latency
- Bandwidth costs
- Consistency vs availability
- Failover complexity

**Disaster Recovery:**
- Active-passive setup
- Regular failover testing
- Monitor replication lag
- Automated failover scripts

**Key Points:**
- Use MirrorMaker or Replicator
- Consider latency and bandwidth
- Plan failover procedures
- Test regularly

---

### Solution 36: Topic Design for Microservices

**Naming Conventions:**
```
{service-name}.{domain}.{event-type}
Example: orders.payment.payment-processed
```

**Topic Granularity:**
- One topic per event type (recommended)
- Easier schema evolution
- Independent scaling
- Clear ownership

**Shared Topics:**
- Use for cross-cutting concerns
- Document ownership
- Coordinate schema changes
- Consider separate topics

**Key Points:**
- Consistent naming
- One event type per topic
- Clear ownership
- Document conventions

---

### Solution 37: Event Sourcing with Kafka

**Event Store:**
```java
// Store events in Kafka
producer.send(new ProducerRecord<>("events", 
    eventId, 
    serialize(event)));

// Replay events
consumer.seekToBeginning(consumer.assignment());
while (true) {
    ConsumerRecords records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord record : records) {
        Event event = deserialize(record.value());
        applyEvent(event);  // Rebuild state
    }
}
```

**Event Versioning:**
- Use Schema Registry
- Backward compatible schemas
- Version events
- Handle migrations

**Key Points:**
- Kafka as event store
- Replay for state reconstruction
- Version events properly
- Handle schema evolution

---

### Solution 38: CQRS with Kafka

**Command Topics:**
```java
// Write commands
producer.send(new ProducerRecord<>("commands", command));
```

**Query Topics:**
```java
// Read models built from events
KTable<String, Order> orderView = builder.table("orders-view");
```

**Eventual Consistency:**
- Acceptable in CQRS
- Monitor lag
- Handle consistency requirements
- Design for async updates

**Key Points:**
- Separate command and query
- Build read models from events
- Accept eventual consistency
- Monitor and handle lag

---

### Solution 39: Dead Letter Queue Pattern

**DLQ Implementation:**
```java
try {
    processMessage(record);
} catch (Exception e) {
    // Send to DLQ
    ProducerRecord<String, String> dlqRecord = new ProducerRecord<>(
        "dlq-topic",
        record.key(),
        record.value()
    );
    dlqRecord.headers().add("error", e.getMessage().getBytes());
    dlqRecord.headers().add("original-topic", record.topic().getBytes());
    dlqRecord.headers().add("original-partition", 
        String.valueOf(record.partition()).getBytes());
    dlqRecord.headers().add("original-offset", 
        String.valueOf(record.offset()).getBytes());
    
    producer.send(dlqRecord);
}
```

**DLQ Processing:**
- Monitor DLQ size
- Investigate failures
- Retry or fix messages
- Alert on DLQ growth

**Key Points:**
- Route failures to DLQ
- Include error context
- Monitor and process DLQ
- Prevent DLQ growth

---

### Solution 40: Kafka Connect Integration

**Source Connector:**
```json
{
  "name": "jdbc-source",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:postgresql://localhost:5432/mydb",
    "table.whitelist": "users",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "postgres-"
  }
}
```

**Sink Connector:**
```json
{
  "name": "jdbc-sink",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "connection.url": "jdbc:postgresql://localhost:5432/mydb",
    "topics": "my-topic",
    "table.name.format": "kafka_${topic}"
  }
}
```

**Failure Handling:**
- Connectors retry on failure
- Checkpoint offsets
- Monitor connector status
- Handle schema mismatches

**Key Points:**
- Connect integrates external systems
- Source and sink connectors
- Handles failures and restarts
- Monitor connector health

---

## Advanced Topics Solutions

### Solution 41: Compacted Topics

**Configuration:**
```bash
kafka-topics.sh --create --topic my-compacted-topic \
  --partitions 3 \
  --replication-factor 3 \
  --config cleanup.policy=compact \
  --bootstrap-server localhost:9092
```

**How Compaction Works:**
- Keeps latest value per key
- Removes older values
- Runs periodically
- Maintains head and tail of log

**Use Cases:**
- Change data capture
- State stores
- Latest value per key
- Event sourcing

**Compaction Lag:**
- Monitor compaction lag
- Adjust `min.cleanable.dirty.ratio`
- Ensure compaction runs regularly

**Key Points:**
- Keeps latest value per key
- Useful for state management
- Monitor compaction lag
- Configure appropriately

---

### Solution 42: Quotas and Throttling

**Producer Quota:**
```bash
kafka-configs.sh --alter --add-config \
  'producer_byte_rate=1048576' \
  --entity-type clients \
  --entity-name producer-client \
  --bootstrap-server localhost:9092
```

**Consumer Quota:**
```bash
kafka-configs.sh --alter --add-config \
  'consumer_byte_rate=1048576' \
  --entity-type clients \
  --entity-name consumer-client \
  --bootstrap-server localhost:9092
```

**Quota Violations:**
- Broker throttles client
- Client receives throttle exceptions
- Monitor quota usage
- Adjust as needed

**Key Points:**
- Prevent resource exhaustion
- Per-client quotas
- Throttling on violation
- Monitor and adjust

---

### Solution 43: MirrorMaker 2.0 Setup

**Configuration:**
```properties
# mm2.properties
clusters = source, target
source.bootstrap.servers = source-kafka:9092
target.bootstrap.servers = target-kafka:9092

# Replication
source->target.enabled = true
source->target.topics = .*
```

**Features:**
- Offset translation
- Topic configuration replication
- Consumer group replication
- Monitoring metrics

**Key Points:**
- Cross-cluster replication
- Handles offset translation
- Replicates configs and groups
- Monitor replication lag

---

### Solution 44: Kafka on Kubernetes

**StatefulSet Deployment:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka
spec:
  serviceName: kafka
  replicas: 3
  template:
    spec:
      containers:
      - name: kafka
        image: confluentinc/cp-kafka:latest
        volumeMounts:
        - name: data
          mountPath: /var/lib/kafka/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 100Gi
```

**Challenges:**
- Persistent storage
- Service discovery
- Network policies
- Resource management

**Key Points:**
- Use StatefulSets
- Persistent volumes required
- Service discovery critical
- Consider operators (Strimzi)

---

### Solution 45: Performance Testing

**Tools:**
- **kafka-producer-perf-test**: Producer performance
- **kafka-consumer-perf-test**: Consumer performance
- **kafka-run-class**: General testing

**Producer Test:**
```bash
kafka-producer-perf-test.sh \
  --topic test-topic \
  --num-records 1000000 \
  --record-size 1024 \
  --throughput 10000 \
  --producer-props bootstrap.servers=localhost:9092
```

**Consumer Test:**
```bash
kafka-consumer-perf-test.sh \
  --topic test-topic \
  --messages 1000000 \
  --bootstrap-server localhost:9092
```

**Metrics to Measure:**
- Throughput (messages/sec, MB/sec)
- Latency (p50, p95, p99)
- CPU and memory usage
- Network utilization

**Key Points:**
- Use built-in tools
- Measure key metrics
- Test under load
- Identify bottlenecks

---

## Kafka Internals & Advanced Concepts Solutions

### Solution 46: KRaft (Kafka Raft) Migration

**What is KRaft:**
- KRaft (Kafka Raft) is the new consensus protocol replacing Zookeeper
- Introduced in Kafka 2.8.0, production-ready in 3.3.0
- Uses Raft consensus algorithm for metadata management
- Eliminates Zookeeper dependency

**Advantages over Zookeeper:**
- **Simpler architecture**: No separate Zookeeper cluster
- **Better scalability**: Can handle millions of partitions
- **Faster controller failover**: Milliseconds vs seconds
- **Simpler operations**: One less system to manage
- **Better metadata performance**: More efficient metadata handling

**Migration Process:**
```bash
# 1. Upgrade to Kafka 3.3+ with KRaft support
# 2. Start brokers in migration mode
process.roles=broker,controller
controller.quorum.voters=1@localhost:9093,2@localhost:9094,3@localhost:9095

# 3. Migrate metadata from Zookeeper to KRaft
kafka-metadata-migration.sh --bootstrap-server localhost:9092 \
  --zookeeper localhost:2181

# 4. Verify migration
# 5. Remove Zookeeper dependency
```

**Key Differences:**
- Metadata stored in KRaft metadata log, not Zookeeper
- Controller uses Raft for consensus
- No Zookeeper session management
- Different configuration properties

**Key Points:**
- KRaft is the future of Kafka
- Simpler and more scalable
- Migration requires careful planning
- Test thoroughly before production

---

### Solution 47: Kafka Storage Internals

**How Kafka Stores Data:**
- Each partition is a log (ordered sequence of records)
- Log divided into segments (files on disk)
- Each segment: `.log` file (data) + `.index` file (offset index) + `.timeindex` file (time index)

**Log Segments:**
```
topic-partition/
  ├── 00000000000000000000.log    # Segment file (data)
  ├── 00000000000000000000.index   # Offset index
  ├── 00000000000000000000.timeindex  # Time index
  ├── 00000000000000100000.log
  ├── 00000000000000100000.index
  └── ...
```

**Segment File Structure:**
- Append-only log file
- Records stored sequentially
- Format: offset, record size, record data
- Immutable once written

**Index Files:**
- **Offset Index**: Maps relative offset to file position
  - Enables fast seeks to specific offsets
  - Sparse index (not every offset indexed)
- **Time Index**: Maps timestamp to offset
  - Enables time-based seeks
  - Used for retention policies

**How Reads Work:**
1. Consumer requests offset N
2. Broker finds segment containing offset N using index
3. Uses offset index to find file position
4. Reads from that position
5. Returns records starting from offset N

**Segment Rolling:**
- New segment created when:
  - Segment size exceeds `log.segment.bytes`
  - Time exceeds `log.segment.ms`
- Old segments cleaned up based on retention policy

**Key Points:**
- Segments enable efficient storage and retrieval
- Indexes enable fast seeks
- Append-only design ensures high throughput
- Segments are immutable

---

### Solution 48: Message Headers and Metadata

**Adding Headers:**
```java
ProducerRecord<String, String> record = new ProducerRecord<>("my-topic", "key", "value");

// Add headers
record.headers().add("correlation-id", "12345".getBytes());
record.headers().add("trace-id", "abc-def-ghi".getBytes());
record.headers().add("source", "service-a".getBytes());

producer.send(record);
```

**Reading Headers:**
```java
for (ConsumerRecord<String, String> record : records) {
    // Get specific header
    Header correlationId = record.headers().lastHeader("correlation-id");
    if (correlationId != null) {
        String id = new String(correlationId.value());
    }
    
    // Iterate all headers
    for (Header header : record.headers()) {
        System.out.println(header.key() + ": " + new String(header.value()));
    }
}
```

**Distributed Tracing:**
```java
// Producer: Add trace context
record.headers().add("traceparent", traceContext.getTraceParent().getBytes());
record.headers().add("tracestate", traceContext.getTraceState().getBytes());

// Consumer: Extract and propagate
Header traceparent = record.headers().lastHeader("traceparent");
if (traceparent != null) {
    TraceContext context = TraceContext.fromString(new String(traceparent.value()));
    // Continue trace
}
```

**Header-Based Routing:**
```java
// Route based on header
String route = new String(record.headers().lastHeader("route").value());
String targetTopic = "topic-" + route;
producer.send(new ProducerRecord<>(targetTopic, record.key(), record.value()));
```

**Performance Impact:**
- Headers add small overhead (~100 bytes per header)
- Serialization/deserialization overhead
- Minimal impact for reasonable header counts
- Use headers for metadata, not large data

**Key Points:**
- Headers for metadata, tracing, routing
- Small performance overhead
- Useful for distributed systems
- Don't store large data in headers

---

### Solution 49: Kafka Controller and Cluster Coordination

**Controller Responsibilities:**
- Manages partition leadership
- Handles broker failures
- Coordinates partition reassignments
- Manages topic creation/deletion
- Maintains cluster metadata

**Controller Election:**
- In Zookeeper: First broker to create `/controller` ephemeral node
- In KRaft: Raft-based election
- Only one active controller at a time
- Other brokers act as standby controllers

**Controller Failure:**
- New controller elected automatically
- Controller state recovered from:
  - Zookeeper (legacy)
  - KRaft metadata log (KRaft mode)
- Minimal impact on cluster operations
- Temporary unavailability during failover

**Partition Leadership Management:**
```java
// Controller maintains:
// - Partition → Leader mapping
// - Partition → ISR list
// - Partition → Replica assignment

// On broker failure:
// 1. Controller detects failure
// 2. Selects new leader from ISR
// 3. Updates metadata
// 4. Notifies brokers
```

**Metadata Propagation:**
- Controller updates metadata
- Metadata propagated to all brokers
- Brokers cache metadata locally
- Clients fetch metadata from brokers

**Key Points:**
- Controller is critical but can failover quickly
- Manages cluster state
- Coordinates operations
- Single point of coordination (not failure)

---

### Solution 50: Kafka Protocol Internals

**Request/Response Protocol:**
- Binary protocol over TCP
- Request: API key, version, correlation ID, client ID, request data
- Response: Correlation ID, response data
- Correlation ID matches request to response

**Main Protocol Requests:**
- **Produce**: Send messages to broker
- **Fetch**: Read messages from broker
- **Metadata**: Get cluster metadata
- **Offset**: Get offsets for partitions
- **FindCoordinator**: Find group coordinator
- **JoinGroup**: Join consumer group
- **SyncGroup**: Sync group assignments
- **Heartbeat**: Keep consumer alive
- **LeaveGroup**: Leave consumer group
- **OffsetCommit**: Commit offsets
- **OffsetFetch**: Fetch committed offsets

**Produce Protocol:**
```
ProduceRequest:
  - Topic → Partition → RecordSet
  - Acks (0, 1, all)
  - Timeout

ProduceResponse:
  - Topic → Partition → Error, BaseOffset, Timestamp
```

**Fetch Protocol:**
```
FetchRequest:
  - Topic → Partition → FetchOffset, MaxBytes
  - MaxWaitTime
  - MinBytes

FetchResponse:
  - Topic → Partition → Error, HighWatermark, RecordSet
```

**Batching at Protocol Level:**
- Multiple records batched in single request
- Reduces network overhead
- Improves throughput
- Controlled by batch.size, linger.ms

**Protocol Versions:**
- Each API has version
- Backward compatible
- New features in newer versions
- Clients negotiate supported versions

**Key Points:**
- Binary protocol for efficiency
- Request/response pattern
- Batching improves performance
- Version negotiation for compatibility

---

### Solution 51: Timestamp Handling and Out-of-Order Events

**Timestamp Types:**
- **CreateTime**: Timestamp when record created (producer)
- **LogAppendTime**: Timestamp when record appended to log (broker)

**Configuration:**
```bash
# Use create time (default)
log.message.timestamp.type=CreateTime

# Use log append time
log.message.timestamp.type=LogAppendTime
```

**Producer Timestamp:**
```java
// Set timestamp explicitly
long timestamp = System.currentTimeMillis();
ProducerRecord<String, String> record = new ProducerRecord<>(
    "topic", null, timestamp, "key", "value");

// Or let producer set it
ProducerRecord<String, String> record = new ProducerRecord<>("topic", "key", "value");
// Producer sets timestamp automatically
```

**Handling Out-of-Order Events:**
```java
// In Kafka Streams
KStream<String, String> stream = builder.stream("events");

stream
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5))
        .grace(Duration.ofSeconds(30)))  // Grace period for late events
    .aggregate(...);
```

**Time-Based Processing:**
- Use event time (CreateTime) for business logic
- Use processing time (LogAppendTime) for system metrics
- Handle late events with grace periods
- Consider watermark strategies

**Key Points:**
- Two timestamp types serve different purposes
- CreateTime for business logic
- LogAppendTime for system metrics
- Handle out-of-order with grace periods

---

### Solution 52: Kafka Admin API

**Admin Client Setup:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");

AdminClient admin = AdminClient.create(props);
```

**Create Topic:**
```java
NewTopic newTopic = new NewTopic("my-topic", 3, (short) 3);
CreateTopicsResult result = admin.createTopics(Collections.singletonList(newTopic));

// Wait for completion
result.all().get();
```

**Delete Topic:**
```java
DeleteTopicsResult result = admin.deleteTopics(Collections.singletonList("my-topic"));
result.all().get();
```

**Alter Topic Configuration:**
```java
ConfigEntry retentionConfig = new ConfigEntry("retention.ms", "604800000");
AlterConfigOp alterOp = new AlterConfigOp(retentionConfig, AlterConfigOp.OpType.SET);

Map<ConfigResource, Collection<AlterConfigOp>> configs = new HashMap<>();
configs.put(new ConfigResource(ConfigResource.Type.TOPIC, "my-topic"), 
            Collections.singletonList(alterOp));

AlterConfigsResult result = admin.incrementalAlterConfigs(configs);
result.all().get();
```

**List Topics:**
```java
ListTopicsResult result = admin.listTopics();
Set<String> topics = result.names().get();
```

**Describe Topics:**
```java
DescribeTopicsResult result = admin.describeTopics(Collections.singletonList("my-topic"));
TopicDescription description = result.allTopicNames().get().get("my-topic");
```

**Async Operations:**
```java
CreateTopicsResult result = admin.createTopics(Collections.singletonList(newTopic));

result.all().whenComplete((v, e) -> {
    if (e != null) {
        System.err.println("Failed to create topic: " + e);
    } else {
        System.out.println("Topic created successfully");
    }
});
```

**Best Practices:**
- Use async operations for better performance
- Handle exceptions appropriately
- Check operation results
- Don't perform admin operations in tight loops

**Key Points:**
- Programmatic cluster management
- Supports async operations
- Handle failures appropriately
- Use for automation and tooling

---

### Solution 53: Backpressure Handling

**Detecting Backpressure:**
```java
// Monitor consumer lag
Map<TopicPartition, Long> lags = getConsumerLag(consumer);
for (Map.Entry<TopicPartition, Long> entry : lags.entrySet()) {
    if (entry.getValue() > THRESHOLD) {
        // Backpressure detected
        handleBackpressure(entry.getKey(), entry.getValue());
    }
}
```

**Strategies:**

1. **Scale Consumers:**
   - Add more consumer instances
   - Increase parallelism
   - Scale horizontally

2. **Throttle Producers:**
   ```java
   // Reduce producer throughput
   props.put("batch.size", "8192");  // Smaller batches
   props.put("linger.ms", "100");  // Longer wait
   props.put("max.in.flight.requests.per.connection", "1");  // Reduce concurrency
   ```

3. **Optimize Processing:**
   - Optimize consumer processing logic
   - Parallelize processing
   - Use async processing
   - Batch processing

4. **Increase Resources:**
   - More CPU/memory for consumers
   - Faster network
   - Optimize downstream systems

**Prevention:**
- Right-size consumer capacity
- Monitor lag continuously
- Set up alerts
- Plan for peak loads
- Test under load

**Key Points:**
- Monitor lag to detect backpressure
- Multiple strategies available
- Scale consumers or throttle producers
- Prevent with proper capacity planning

---

### Solution 54: Kafka Log Retention Internals

**Time-Based Retention:**
```bash
# Retain messages for 7 days
log.retention.hours=168
# or
log.retention.ms=604800000
```

**How It Works:**
- Broker checks segment files periodically
- Compares segment's last modified time to retention period
- Deletes segments older than retention period
- Runs every `log.retention.check.interval.ms` (default: 5 minutes)

**Size-Based Retention:**
```bash
# Retain up to 10GB per partition
log.retention.bytes=10737418240
```

**How It Works:**
- Calculates total size of partition
- Deletes oldest segments if size exceeds limit
- Maintains at least one segment per partition

**Cleanup Process:**
1. Identify deletable segments
2. Delete `.log`, `.index`, `.timeindex` files
3. Update partition metadata
4. Log deletion events

**Optimization:**
- Balance retention vs storage costs
- Consider segment size
- Monitor disk usage
- Set appropriate retention policies

**Key Points:**
- Time-based: delete old segments
- Size-based: delete when size limit reached
- Cleanup runs periodically
- Balance retention vs storage

---

### Solution 55: Leader-Follower Replication Protocol

**Fetch Requests:**
- Followers send FetchRequest to leader
- Request includes: topic, partition, fetch offset, max bytes
- Leader responds with records starting from fetch offset

**Replication Process:**
```
1. Follower sends FetchRequest(offset=N)
2. Leader responds with records from offset N
3. Follower appends records to local log
4. Follower updates LEO (Log End Offset)
5. Follower sends next FetchRequest(offset=N+records)
```

**High Watermark (HW):**
- Highest offset that is replicated to all ISR replicas
- Consumers can only read up to HW
- Prevents reading uncommitted data
- Updated when all ISR replicas acknowledge

**LEO (Log End Offset):**
- Highest offset in a replica's log
- Each replica has its own LEO
- Leader's LEO = latest written offset
- Follower's LEO = last replicated offset

**Catching Up:**
```
Leader LEO: 1000
Follower LEO: 500 (lagging)

1. Follower sends FetchRequest(offset=500)
2. Leader sends records 500-1000
3. Follower appends records, updates LEO to 1000
4. Follower now in sync
```

**Replication Lag:**
- Difference between leader LEO and follower LEO
- Normal: < 1000 records
- High lag: > 10000 records (investigate)
- Causes: slow network, slow disk, high load

**Key Points:**
- Followers fetch from leaders
- High watermark ensures consistency
- LEO tracks replication progress
- Monitor replication lag

---

## Testing & Operations Solutions

### Solution 56: Producer and Consumer Interceptors

**What are Interceptors:**
- Interceptors allow adding cross-cutting concerns
- Execute before/after message processing
- Useful for logging, metrics, tracing, validation
- Can modify or filter messages

**Producer Interceptor:**
```java
public class MetricsProducerInterceptor implements ProducerInterceptor<String, String> {
    private Counter messageCounter;
    
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        // Called before sending
        messageCounter.increment();
        
        // Can modify record
        record.headers().add("sent-at", String.valueOf(System.currentTimeMillis()).getBytes());
        
        return record;
    }
    
    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        // Called after acknowledgment
        if (exception != null) {
            // Handle error
        }
    }
    
    @Override
    public void close() {
        // Cleanup
    }
    
    @Override
    public void configure(Map<String, ?> configs) {
        // Initialize metrics, etc.
    }
}

// Configuration
props.put("interceptor.classes", "com.example.MetricsProducerInterceptor");
```

**Consumer Interceptor:**
```java
public class TracingConsumerInterceptor implements ConsumerInterceptor<String, String> {
    @Override
    public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
        // Called before processing
        for (ConsumerRecord<String, String> record : records) {
            Header traceId = record.headers().lastHeader("trace-id");
            if (traceId != null) {
                // Start trace span
                startTrace(new String(traceId.value()));
            }
        }
        return records;
    }
    
    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
        // Called after commit
    }
    
    @Override
    public void close() {
        // Cleanup
    }
    
    @Override
    public void configure(Map<String, ?> configs) {
        // Initialize
    }
}

// Configuration
props.put("interceptor.classes", "com.example.TracingConsumerInterceptor");
```

**Common Use Cases:**
- **Metrics**: Count messages, measure latency
- **Tracing**: Add trace IDs, propagate context
- **Logging**: Log message flow
- **Validation**: Validate messages before processing
- **Enrichment**: Add metadata to messages

**Key Points:**
- Interceptors for cross-cutting concerns
- Can modify messages
- Handle failures gracefully
- Don't add heavy processing

---

### Solution 57: Kafka Streams State Stores and Interactive Queries

**State Store Types:**

1. **RocksDB State Store** (default):
   - Persistent, disk-backed
   - Good for large state
   - Slower than in-memory

2. **In-Memory State Store**:
   - Fast, non-persistent
   - Good for small state
   - Lost on restart

**Creating State Store:**
```java
StreamsBuilder builder = new StreamsBuilder();

// Materialize state store
KTable<String, Long> counts = builder.stream("input-topic")
    .groupByKey()
    .count(Materialized.as("count-store"));  // Named state store

// Or explicitly create store
StoreBuilder<KeyValueStore<String, Long>> storeBuilder = Stores.keyValueStoreBuilder(
    Stores.persistentKeyValueStore("my-store"),
    Serdes.String(),
    Serdes.Long()
);

builder.addStateStore(storeBuilder);
```

**Interactive Queries:**
```java
public class InteractiveQueryService {
    private KafkaStreams streams;
    
    public Long getCount(String key) {
        // Get state store
        ReadOnlyKeyValueStore<String, Long> store = streams.store(
            StoreQueryParameters.fromNameAndType("count-store", 
                QueryableStoreTypes.keyValueStore())
        );
        
        // Query store
        return store.get(key);
    }
    
    // Expose via REST API
    @GetMapping("/count/{key}")
    public Long getCount(@PathVariable String key) {
        return getCount(key);
    }
}
```

**State Store Recovery:**
- State stores are backed by changelog topics
- On restart, state is rebuilt from changelog
- Recovery time depends on state size
- Can take time for large state

**Key Points:**
- State stores enable stateful operations
- RocksDB for large state, in-memory for small
- Interactive queries expose state via API
- State recovered from changelog topics

---

### Solution 58: Outbox Pattern Implementation

**What is Outbox Pattern:**
- Ensures reliable event publishing from database transactions
- Writes events to database table (outbox) in same transaction
- Separate process reads outbox and publishes to Kafka
- Ensures exactly-once event publishing

**Database Schema:**
```sql
CREATE TABLE outbox (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    aggregate_id VARCHAR(255) NOT NULL,
    event_type VARCHAR(255) NOT NULL,
    payload TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    published BOOLEAN DEFAULT FALSE
);
```

**Writing to Outbox:**
```java
@Transactional
public void createOrder(Order order) {
    // Save order
    orderRepository.save(order);
    
    // Write event to outbox in same transaction
    OutboxEvent event = new OutboxEvent();
    event.setAggregateId(order.getId());
    event.setEventType("OrderCreated");
    event.setPayload(serialize(order));
    outboxRepository.save(event);
    // Both operations in same transaction
}
```

**Publishing from Outbox:**
```java
@Component
public class OutboxPublisher {
    @Scheduled(fixedDelay = 1000)
    public void publishEvents() {
        List<OutboxEvent> events = outboxRepository.findUnpublished();
        
        for (OutboxEvent event : events) {
            try {
                // Publish to Kafka
                producer.send(new ProducerRecord<>("events", 
                    event.getAggregateId(), 
                    event.getPayload()));
                
                // Mark as published
                event.setPublished(true);
                outboxRepository.save(event);
            } catch (Exception e) {
                // Retry later
            }
        }
    }
}
```

**Deduplication:**
- Use idempotent producer
- Check if event already published
- Use event ID as Kafka message key

**Cleanup:**
```java
@Scheduled(fixedDelay = 3600000)  // Hourly
public void cleanupPublishedEvents() {
    // Delete events published more than 7 days ago
    outboxRepository.deletePublishedEventsOlderThan(7, TimeUnit.DAYS);
}
```

**Key Points:**
- Ensures consistency between DB and Kafka
- Write to outbox in same transaction
- Separate process publishes events
- Handle deduplication and cleanup

---

### Solution 59: Testing Kafka Applications

**Unit Testing Producer:**
```java
@Test
public void testProducer() {
    // Mock producer
    KafkaProducer<String, String> mockProducer = mock(KafkaProducer.class);
    
    // Test producer logic
    MyProducerService service = new MyProducerService(mockProducer);
    service.sendMessage("key", "value");
    
    // Verify
    verify(mockProducer).send(any(ProducerRecord.class), any(Callback.class));
}
```

**Integration Testing with Embedded Kafka:**
```java
@SpringBootTest
public class KafkaIntegrationTest {
    @ClassRule
    public static EmbeddedKafkaBroker embeddedKafka = new EmbeddedKafkaBroker(1)
        .brokerProperty("listeners", "PLAINTEXT://localhost:9092")
        .brokerProperty("auto.create.topics.enable", "true");
    
    @Test
    public void testProducerConsumer() {
        // Create producer
        KafkaProducer<String, String> producer = createProducer();
        
        // Send message
        producer.send(new ProducerRecord<>("test-topic", "key", "value"));
        
        // Create consumer
        KafkaConsumer<String, String> consumer = createConsumer();
        consumer.subscribe(Collections.singletonList("test-topic"));
        
        // Consume message
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(5));
        assertEquals(1, records.count());
    }
}
```

**Testing Kafka Streams:**
```java
@Test
public void testKafkaStreams() {
    StreamsBuilder builder = new StreamsBuilder();
    builder.stream("input-topic")
        .mapValues(v -> v.toUpperCase())
        .to("output-topic");
    
    Topology topology = builder.build();
    
    // Create test driver
    TopologyTestDriver testDriver = new TopologyTestDriver(topology, props);
    
    // Send input
    testDriver.pipeInput("input-topic", "key", "hello");
    
    // Read output
    ProducerRecord<String, String> output = testDriver.readOutput("output-topic", 
        new StringDeserializer(), new StringDeserializer());
    
    assertEquals("HELLO", output.value());
}
```

**Testing Tools:**
- **EmbeddedKafka**: Embedded Kafka broker for integration tests
- **TopologyTestDriver**: Test Kafka Streams without broker
- **MockProducer/MockConsumer**: Mock clients for unit tests
- **Testcontainers**: Docker-based Kafka for tests

**Key Points:**
- Unit test with mocks
- Integration test with embedded Kafka
- Test Kafka Streams with TopologyTestDriver
- Use appropriate testing tools

---

### Solution 60: Capacity Planning and Sizing

**Broker Count Calculation:**
```
Brokers needed = max(
    (Total Partitions / Partitions per Broker),
    (Replication Factor + 1) / 2,  // For fault tolerance
    (Throughput / Broker Capacity)
)

Example:
- 1000 partitions
- 200 partitions per broker max
- Replication factor: 3
- Brokers = max(1000/200, (3+1)/2, ...) = max(5, 2, ...) = 5 brokers minimum
```

**Disk Storage Estimation:**
```
Storage per Partition = (Message Size × Messages per Day × Retention Days) / Replication Factor

Total Storage = Sum(Storage per Partition) × Replication Factor

Example:
- Message size: 1KB
- Messages per day: 10M
- Retention: 7 days
- Partitions: 100
- Replication: 3

Storage = (1KB × 10M × 7 × 100) / 3 = ~2.3TB per broker
```

**Network Bandwidth:**
```
Bandwidth = (Message Size × Messages per Second) × Replication Factor

Example:
- Message size: 1KB
- Messages/sec: 100K
- Replication: 3

Bandwidth = 1KB × 100K × 3 = 300MB/s = 2.4 Gbps
```

**Factors Affecting Sizing:**
- **Throughput**: Messages per second
- **Retention**: How long to keep data
- **Replication**: Number of replicas
- **Partition count**: More partitions = more overhead
- **Message size**: Larger messages = more storage/bandwidth
- **Compression**: Reduces storage and bandwidth
- **Growth**: Plan for 2-3x growth

**Key Points:**
- Calculate brokers based on partitions, throughput, fault tolerance
- Estimate storage based on retention and throughput
- Plan network bandwidth for replication
- Account for growth and overhead

---

## Summary

This comprehensive guide covers all major Kafka topics for SDE 2 interviews:

1. **Producers**: Idempotency, partitioning, batching, error handling
2. **Consumers**: Groups, rebalancing, offset management, lag
3. **Replication**: ISR, leader election, fault tolerance
4. **Exactly-Once**: Transactions, end-to-end guarantees
5. **Performance**: Optimization, tuning, monitoring
6. **Security**: Authentication, authorization, encryption
7. **Stream Processing**: Kafka Streams, joins, windows
8. **Architecture**: Multi-DC, event sourcing, CQRS
9. **Advanced**: Compaction, quotas, MirrorMaker, K8s
10. **Internals**: KRaft, storage, protocol, controller, timestamps, Admin API, backpressure, retention, replication protocol
11. **Testing & Operations**: Interceptors, state stores, outbox pattern, testing, capacity planning

Each solution includes code examples, configurations, and best practices. Use these as a reference for interview preparation and real-world implementations.

**Total Coverage: 60 Problems with Detailed Solutions**
