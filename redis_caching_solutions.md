# Redis & Caching Interview Solutions for SDE 2

## Table of Contents
1. [Redis Fundamentals Solutions](#redis-fundamentals-solutions)
2. [Redis Data Structures Solutions](#redis-data-structures-solutions)
3. [Caching Strategies Solutions](#caching-strategies-solutions)
4. [Redis Persistence Solutions](#redis-persistence-solutions)
5. [Redis Replication & High Availability Solutions](#redis-replication--high-availability-solutions)
6. [Redis Clustering Solutions](#redis-clustering-solutions)
7. [Performance & Optimization Solutions](#performance--optimization-solutions)
8. [Cache Invalidation & Patterns Solutions](#cache-invalidation--patterns-solutions)
9. [Redis Pub/Sub Solutions](#redis-pubsub-solutions)
10. [Redis Transactions & Lua Scripting Solutions](#redis-transactions--lua-scripting-solutions)
11. [Memory Management Solutions](#memory-management-solutions)
12. [Security & Authentication Solutions](#security--authentication-solutions)
13. [Monitoring & Troubleshooting Solutions](#monitoring--troubleshooting-solutions)
14. [Design Patterns & Architecture Solutions](#design-patterns--architecture-solutions)
15. [Advanced Topics Solutions](#advanced-topics-solutions)

---

## Redis Fundamentals Solutions

### Solution 1: Redis Architecture and Use Cases

**What is Redis:**
- Redis (Remote Dictionary Server) is an in-memory data structure store
- Single-threaded event loop (I/O multiplexing)
- Supports persistence (RDB, AOF)
- Provides data structures: strings, hashes, lists, sets, sorted sets, streams

**How Redis Works Internally:**
- Single-threaded event loop handles all commands
- Non-blocking I/O using epoll/kqueue
- Commands executed sequentially (no race conditions)
- Background threads for persistence and some operations

**Main Use Cases:**
1. **Caching**: Fast data access, reduce database load
2. **Session Storage**: Store user sessions
3. **Real-time Analytics**: Counters, leaderboards, metrics
4. **Message Queues**: Pub/Sub, Lists, Streams
5. **Rate Limiting**: API throttling
6. **Distributed Locks**: Coordination between services
7. **Geospatial Data**: Location-based features

**When to Use Redis vs Database:**
- **Use Redis for:**
  - Fast read/write access needed
  - Temporary/volatile data
  - High-frequency operations
  - Simple data structures
  - Real-time features

- **Use Database for:**
  - Complex queries (JOINs, aggregations)
  - ACID transactions across multiple records
  - Large datasets that don't fit in memory
  - Complex data relationships
  - Long-term data storage

**Limitations:**
- Memory-limited (data must fit in RAM)
- Single-threaded (CPU-bound operations block)
- No complex queries (no JOINs, limited filtering)
- Persistence trade-offs (RDB may lose data, AOF slower)
- Limited data size per key (512MB max)

**Key Points:**
- In-memory, single-threaded architecture
- Excellent for caching and real-time features
- Not a replacement for databases
- Choose based on use case requirements

---

### Solution 2: Redis Data Types and When to Use Them

**Main Redis Data Types:**

1. **Strings**: Simple key-value pairs
   - Use for: Simple values, counters, distributed locks
   - Example: `SET user:1001:name "John"`

2. **Hashes**: Field-value maps
   - Use for: Objects, user profiles, partial updates
   - Example: `HSET user:1001 name "John" email "john@example.com"`

3. **Lists**: Ordered collections
   - Use for: Queues, stacks, timelines
   - Example: `LPUSH queue:orders "order123"`

4. **Sets**: Unordered unique collections
   - Use for: Unique items, tags, membership testing
   - Example: `SADD tags:article:123 "redis" "caching"`

5. **Sorted Sets**: Sets with scores
   - Use for: Leaderboards, rankings, time-series
   - Example: `ZADD leaderboard 100 "player1"`

6. **Bitmaps**: Bit arrays
   - Use for: Boolean flags, analytics, user activity
   - Example: `SETBIT user:1001:activity 100 1`

7. **HyperLogLog**: Cardinality estimation
   - Use for: Unique count approximation
   - Example: `PFADD visitors:2024:01:01 "user1"`

8. **Streams**: Log-like data structure
   - Use for: Event logging, message queues
   - Example: `XADD events * action "login" user "user1"`

**Choosing Data Structures:**

**Strings vs Hashes:**
- Use **Strings** for: Simple values, when you always fetch entire value
- Use **Hashes** for: Objects with multiple fields, partial updates needed
- Memory: Hashes more efficient for objects (no JSON overhead)

**Sets vs Sorted Sets:**
- Use **Sets** for: Unique collections, membership testing, set operations
- Use **Sorted Sets** for: Rankings, ranges, when order matters

**Lists vs Streams:**
- Use **Lists** for: Simple queues, stacks, when order matters
- Use **Streams** for: Event logging, consumer groups, message persistence

**Key Points:**
- Choose based on access patterns
- Consider memory efficiency
- Think about operations needed (ranges, intersections, etc.)

---

### Solution 3: Redis Commands and Operations

**Important Redis Commands:**

**String Commands:**
```redis
SET key value [EX seconds] [PX milliseconds] [NX|XX]
GET key
MGET key1 key2 key3  # Get multiple keys
INCR key  # Atomic increment
DECR key  # Atomic decrement
```

**Hash Commands:**
```redis
HSET key field value
HGET key field
HGETALL key  # Get all fields
HMGET key field1 field2  # Get multiple fields
```

**List Commands:**
```redis
LPUSH key value  # Push to left
RPUSH key value  # Push to right
LPOP key  # Pop from left
RPOP key  # Pop from right
BLPOP key timeout  # Blocking pop
```

**Set Commands:**
```redis
SADD key member
SREM key member
SISMEMBER key member  # Check membership
SINTER key1 key2  # Intersection
SUNION key1 key2  # Union
```

**Sorted Set Commands:**
```redis
ZADD key score member
ZRANGE key start stop [WITHSCORES]
ZRANK key member  # Get rank
ZREVRANGE key start stop  # Reverse range
```

**Atomic Operations:**
- `INCR`, `DECR`: Atomic increment/decrement
- `MULTI/EXEC`: Transaction block
- `SETNX`: Set if not exists (for locks)
- `HSETNX`: Hash set if not exists

**Efficient Multi-Key Operations:**
```redis
# Use MGET instead of multiple GETs
MGET key1 key2 key3

# Use pipeline for multiple operations
# (Client-side batching)
```

**Key Points:**
- Use appropriate commands for data types
- Prefer atomic operations
- Use MGET/pipelining for bulk operations
- Understand command complexity (O(1) vs O(N))

---

## Redis Data Structures Solutions

### Solution 4: Strings and String Operations

**String Caching:**
```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

# Cache a value
r.set('user:1001:name', 'John', ex=3600)  # Expire in 1 hour

# Get cached value
name = r.get('user:1001:name')
```

**Atomic Counter:**
```python
# Increment counter
r.incr('page:views:2024:01:01')

# Increment by specific amount
r.incrby('user:1001:score', 10)

# Decrement
r.decr('counter')
```

**Distributed Lock:**
```python
import time

def acquire_lock(conn, lockname, acquire_timeout=10):
    identifier = str(uuid.uuid4())
    end = time.time() + acquire_timeout
    
    while time.time() < end:
        if conn.set(f'lock:{lockname}', identifier, nx=True, ex=10):
            return identifier
        time.sleep(0.001)
    return False

def release_lock(conn, lockname, identifier):
    pipe = conn.pipeline(True)
    while True:
        try:
            pipe.watch(f'lock:{lockname}')
            if pipe.get(f'lock:{lockname}') == identifier.encode():
                pipe.multi()
                pipe.delete(f'lock:{lockname}')
                pipe.execute()
                return True
            pipe.unwatch()
            break
        except redis.WatchError:
            pass
    return False
```

**String Expiration:**
```python
# Set with expiration
r.setex('key', 3600, 'value')  # Expire in seconds
r.psetex('key', 1000, 'value')  # Expire in milliseconds

# Set expiration on existing key
r.expire('key', 3600)
r.pexpire('key', 1000)

# Check TTL
ttl = r.ttl('key')  # Returns seconds until expiration
```

**Performance Characteristics:**
- GET/SET: O(1) - Very fast
- MGET/MSET: O(N) - Efficient for bulk operations
- String operations are fastest in Redis

**Large String Values:**
- Max string size: 512MB
- For large values, consider splitting or using other structures
- Monitor memory usage

**Key Points:**
- Strings are simplest and fastest
- Use for simple values and counters
- Atomic operations ensure consistency
- Set appropriate TTLs

---

### Solution 5: Hashes for Object Storage

**Storing Objects as Hashes:**
```python
# Store user object
r.hset('user:1001', mapping={
    'name': 'John',
    'email': 'john@example.com',
    'age': '30'
})

# Get all fields
user = r.hgetall('user:1001')

# Get specific field
name = r.hget('user:1001', 'name')

# Get multiple fields
fields = r.hmget('user:1001', 'name', 'email')
```

**Updating Single Field:**
```python
# Update just email
r.hset('user:1001', 'email', 'newemail@example.com')

# Increment numeric field
r.hincrby('user:1001', 'age', 1)
```

**Memory Benefits:**
- Hashes store fields efficiently (no JSON overhead)
- Can update individual fields without rewriting entire object
- More memory-efficient than storing JSON strings

**Comparison: Hash vs JSON String:**
```python
# Hash approach (more efficient)
r.hset('user:1001', 'name', 'John')
r.hset('user:1001', 'email', 'john@example.com')

# JSON string approach (less efficient)
import json
user_data = json.dumps({'name': 'John', 'email': 'john@example.com'})
r.set('user:1001', user_data)
```

**Efficient Retrieval:**
```python
# Get only needed fields (not all)
name_email = r.hmget('user:1001', 'name', 'email')

# Check if field exists
exists = r.hexists('user:1001', 'email')

# Get all field names
fields = r.hkeys('user:1001')
```

**Key Points:**
- Use hashes for objects with multiple fields
- More memory-efficient than JSON strings
- Can update individual fields
- Use HMGET for partial retrieval

---

### Solution 6: Lists and Queue Patterns

**FIFO Queue:**
```python
# Producer: Push to right
r.rpush('queue:orders', 'order1', 'order2', 'order3')

# Consumer: Pop from left
order = r.lpop('queue:orders')

# Check queue length
length = r.llen('queue:orders')
```

**LIFO Stack:**
```python
# Push to left
r.lpush('stack:items', 'item1', 'item2')

# Pop from left
item = r.lpop('stack:items')
```

**Blocking Operations:**
```python
# Block until item available (timeout in seconds)
order = r.blpop('queue:orders', timeout=10)

# Multiple queues
result = r.blpop(['queue:orders', 'queue:payments'], timeout=5)
if result:
    queue_name, value = result
```

**Work Queue Pattern:**
```python
def worker():
    while True:
        # Block until work available
        result = r.blpop('queue:work', timeout=1)
        if result:
            queue, work_item = result
            process_work(work_item)
        else:
            # No work, do other tasks
            pass

def process_work(item):
    try:
        # Process work
        pass
    except Exception as e:
        # On failure, push to retry queue
        r.rpush('queue:retry', item)
```

**Priority Queue:**
```python
# Use multiple queues for priorities
r.rpush('queue:high', 'urgent_task')
r.rpush('queue:normal', 'normal_task')
r.rpush('queue:low', 'low_task')

# Consumer checks high priority first
def get_next_task():
    result = r.blpop(['queue:high', 'queue:normal', 'queue:low'], timeout=1)
    return result[1] if result else None
```

**Key Points:**
- Lists are versatile for queues and stacks
- Use blocking operations for efficient waiting
- Multiple queues for priorities
- Handle failures appropriately

---

### Solution 7: Sets and Set Operations

**Unique Collections:**
```python
# Add members
r.sadd('tags:article:123', 'redis', 'caching', 'database')

# Check membership
is_member = r.sismember('tags:article:123', 'redis')

# Get all members
tags = r.smembers('tags:article:123')

# Remove member
r.srem('tags:article:123', 'caching')
```

**Set Operations:**
```python
# Intersection (common elements)
common = r.sinter('set1', 'set2', 'set3')

# Union (all elements)
all_elements = r.sunion('set1', 'set2')

# Difference (elements in set1 but not set2)
diff = r.sdiff('set1', 'set2')

# Store result
r.sinterstore('result', 'set1', 'set2')
```

**Tagging System:**
```python
# Add tags to article
r.sadd('article:123:tags', 'redis', 'caching')

# Get articles with tag
articles_with_tag = r.smembers('tag:redis:articles')

# Find articles with multiple tags
common_articles = r.sinter('tag:redis:articles', 'tag:caching:articles')
```

**Social Graph Operations:**
```python
# Follow user
r.sadd('user:1001:following', 'user:1002')
r.sadd('user:1002:followers', 'user:1001')

# Get mutual friends
mutual = r.sinter('user:1001:friends', 'user:1002:friends')

# Get all connections
all_connections = r.sunion('user:1001:following', 'user:1001:followers')
```

**Sets vs Sorted Sets:**
- **Sets**: When order doesn't matter, need set operations
- **Sorted Sets**: When you need ranking, ranges, or ordered iteration

**Key Points:**
- Sets for unique collections and membership testing
- Powerful set operations (intersection, union, difference)
- Great for tagging and social graphs
- Use when order doesn't matter

---

### Solution 8: Sorted Sets and Ranking

**Leaderboard Implementation:**
```python
# Add scores
r.zadd('leaderboard', {'player1': 100, 'player2': 200, 'player3': 150})

# Get top 10 players
top_players = r.zrevrange('leaderboard', 0, 9, withscores=True)

# Get player rank (0-based, highest score = rank 0)
rank = r.zrevrank('leaderboard', 'player1')

# Get player score
score = r.zscore('leaderboard', 'player1')
```

**Range Queries:**
```python
# Get players with scores between 100 and 200
players = r.zrangebyscore('leaderboard', 100, 200, withscores=True)

# Get top players by score
top = r.zrevrangebyscore('leaderboard', '+inf', 100, start=0, num=10)

# Count players in score range
count = r.zcount('leaderboard', 100, 200)
```

**Updating Scores:**
```python
# Increment score atomically
r.zincrby('leaderboard', 10, 'player1')

# Add score if member doesn't exist
r.zadd('leaderboard', {'newplayer': 50}, nx=True)

# Update score only if exists
r.zadd('leaderboard', {'player1': 150}, xx=True)
```

**Time-Series Data:**
```python
import time

# Add timestamped events
timestamp = int(time.time())
r.zadd('events:user:1001', {f'event:{timestamp}': timestamp})

# Get events in time range
start_time = int(time.time()) - 3600  # Last hour
end_time = int(time.time())
events = r.zrangebyscore('events:user:1001', start_time, end_time)

# Remove old events (older than 24 hours)
cutoff = int(time.time()) - 86400
r.zremrangebyscore('events:user:1001', 0, cutoff)
```

**Key Points:**
- Sorted sets perfect for rankings and leaderboards
- Efficient range queries
- Atomic score updates
- Great for time-series data

---

### Solution 9: Bitmaps and HyperLogLog

**Bitmaps for Analytics:**
```python
# Track user activity (day 100 of year)
r.setbit('user:1001:activity:2024', 100, 1)

# Check if user was active
was_active = r.getbit('user:1001:activity:2024', 100)

# Count active days
active_days = r.bitcount('user:1001:activity:2024')

# Find common active days
r.bitop('AND', 'common:activity', 'user:1001:activity:2024', 'user:1002:activity:2024')
common_days = r.bitcount('common:activity')
```

**HyperLogLog for Cardinality:**
```python
# Add elements
r.pfadd('visitors:2024:01:01', 'user1', 'user2', 'user3')

# Get approximate count
count = r.pfcount('visitors:2024:01:01')

# Merge multiple HyperLogLogs
r.pfmerge('visitors:week', 'visitors:day1', 'visitors:day2', 'visitors:day3')
week_count = r.pfcount('visitors:week')
```

**When to Use HyperLogLog vs Sets:**
- **HyperLogLog**: When you only need approximate count, memory-efficient
- **Sets**: When you need exact count or membership testing

**Memory Comparison:**
- HyperLogLog: ~12KB regardless of cardinality
- Set: Grows with number of elements
- Use HyperLogLog for large cardinalities (>1000)

**Key Points:**
- Bitmaps for boolean flags and analytics
- HyperLogLog for approximate unique counts
- Very memory-efficient
- Trade accuracy for memory

---

### Solution 10: Streams and Event Sourcing

**Redis Streams Basics:**
```python
# Add event to stream
r.xadd('events', {'action': 'login', 'user': 'user1', 'timestamp': '1234567890'})

# Read events
events = r.xread({'events': '0'}, count=10)

# Read new events (blocking)
events = r.xread({'events': '$'}, block=1000)  # Block 1 second
```

**Consumer Groups:**
```python
# Create consumer group
r.xgroup_create('events', 'processors', id='0', mkstream=True)

# Read as consumer
events = r.xreadgroup('processors', 'consumer1', {'events': '>'}, count=10)

# Acknowledge processing
for stream, messages in events:
    for msg_id, data in messages:
        process_event(data)
        r.xack('events', 'processors', msg_id)
```

**Event Sourcing:**
```python
def append_event(stream_name, event_type, event_data):
    """Append event to stream"""
    event = {
        'type': event_type,
        'data': json.dumps(event_data),
        'timestamp': str(time.time())
    }
    return r.xadd(stream_name, event)

def replay_events(stream_name, from_id='0'):
    """Replay events to rebuild state"""
    state = {}
    events = r.xread({stream_name: from_id})
    
    for stream, messages in events:
        for msg_id, data in messages:
            event_type = data[b'type'].decode()
            event_data = json.loads(data[b'data'].decode())
            apply_event(state, event_type, event_data)
    
    return state
```

**Streams vs Kafka:**
- **Redis Streams**: Simpler, built-in, good for smaller scale
- **Kafka**: More features, better for large scale, more complex

**Key Points:**
- Streams for event logging and queues
- Consumer groups for parallel processing
- Great for event sourcing
- Simpler than Kafka for many use cases

---

## Caching Strategies Solutions

### Solution 11: Cache-Aside Pattern

**Implementation:**
```python
def get_user(user_id):
    # Try cache first
    cache_key = f'user:{user_id}'
    user = r.get(cache_key)
    
    if user:
        return json.loads(user)
    
    # Cache miss - get from database
    user = db.get_user(user_id)
    
    if user:
        # Store in cache
        r.setex(cache_key, 3600, json.dumps(user))
    
    return user

def update_user(user_id, user_data):
    # Update database
    db.update_user(user_id, user_data)
    
    # Invalidate cache
    r.delete(f'user:{user_id}')
    
    # Or update cache
    r.setex(f'user:{user_id}', 3600, json.dumps(user_data))
```

**Advantages:**
- Simple to implement
- Cache failures don't affect database
- Flexible cache invalidation

**Disadvantages:**
- Cache miss penalty (two round trips)
- Potential for stale data
- Need to handle cache invalidation

**Handling Cache Misses:**
- Always check cache first
- Fall back to database on miss
- Populate cache after database read

**Ensuring Consistency:**
- Invalidate cache on updates
- Use appropriate TTL
- Consider write-through for critical data

**Key Points:**
- Most common caching pattern
- Simple but requires careful invalidation
- Good for read-heavy workloads

---

### Solution 12: Write-Through Caching

**Implementation:**
```python
def update_user(user_id, user_data):
    # Update database
    db.update_user(user_id, user_data)
    
    # Update cache immediately
    cache_key = f'user:{user_id}'
    r.setex(cache_key, 3600, json.dumps(user_data))
    
    return user_data

def get_user(user_id):
    # Try cache first
    cache_key = f'user:{user_id}'
    user = r.get(cache_key)
    
    if user:
        return json.loads(user)
    
    # Cache miss - get from database
    user = db.get_user(user_id)
    
    if user:
        # Store in cache
        r.setex(cache_key, 3600, json.dumps(user))
    
    return user
```

**When to Use:**
- When consistency is critical
- When writes are less frequent than reads
- When you can't tolerate stale data

**Handling Write Failures:**
```python
def update_user_safe(user_id, user_data):
    try:
        # Update database first
        db.update_user(user_id, user_data)
        
        # Then update cache
        cache_key = f'user:{user_id}'
        r.setex(cache_key, 3600, json.dumps(user_data))
    except DatabaseError:
        # Database failed - don't update cache
        raise
    except RedisError:
        # Cache failed - log but don't fail
        logger.warning("Cache update failed")
```

**Performance Implications:**
- Every write hits both cache and database
- Higher write latency
- Better read performance (always cache hit)

**Key Points:**
- Ensures cache-database consistency
- Higher write latency
- Good for consistency-critical scenarios

---

### Solution 13: Write-Behind (Write-Back) Pattern

**Implementation:**
```python
import queue
import threading

write_queue = queue.Queue()

def update_user(user_id, user_data):
    # Update cache immediately
    cache_key = f'user:{user_id}'
    r.setex(cache_key, 3600, json.dumps(user_data))
    
    # Queue database write
    write_queue.put((user_id, user_data))
    
    return user_data

def background_writer():
    """Background thread to write to database"""
    while True:
        try:
            user_id, user_data = write_queue.get(timeout=1)
            db.update_user(user_id, user_data)
        except queue.Empty:
            continue
        except Exception as e:
            logger.error(f"Write failed: {e}")
            # Retry logic here

# Start background writer thread
threading.Thread(target=background_writer, daemon=True).start()
```

**Risks:**
- Data loss if cache fails before database write
- Potential inconsistency
- Need to handle failures

**Ensuring Durability:**
- Use Redis persistence (AOF)
- Implement retry logic
- Monitor write queue size
- Handle application crashes

**When Appropriate:**
- High write throughput needed
- Can tolerate eventual consistency
- Writes can be batched
- Not critical data

**Key Points:**
- Highest write performance
- Risk of data loss
- Need careful failure handling
- Use for non-critical, high-volume writes

---

### Solution 14: Read-Through Caching

**Implementation:**
```python
class ReadThroughCache:
    def __init__(self, redis_client, db_client):
        self.redis = redis_client
        self.db = db_client
    
    def get(self, key, loader_func, ttl=3600):
        # Try cache first
        value = self.redis.get(key)
        if value:
            return json.loads(value)
        
        # Cache miss - load from source
        value = loader_func()
        
        if value:
            # Store in cache
            self.redis.setex(key, ttl, json.dumps(value))
        
        return value

# Usage
cache = ReadThroughCache(r, db)

def load_user(user_id):
    return db.get_user(user_id)

user = cache.get(f'user:{user_id}', lambda: load_user(user_id))
```

**Differences from Cache-Aside:**
- Cache handles loading automatically
- Application doesn't know about cache misses
- Simpler application code

**Use Cases:**
- When cache logic should be abstracted
- When using caching libraries
- When you want transparent caching

**Key Points:**
- Cache handles loading
- Simpler application code
- Good for library-based caching

---

### Solution 15: Cache Warming Strategies

**Identifying Data to Warm:**
```python
# Analyze access patterns
def analyze_access_patterns():
    # Track frequently accessed keys
    access_counts = {}
    
    # Monitor for 24 hours
    # Identify top accessed keys
    
    return top_keys

# Warm cache with frequently accessed data
def warm_cache():
    top_keys = analyze_access_patterns()
    
    for key in top_keys:
        # Load from database
        data = db.get(key)
        if data:
            r.setex(key, 3600, json.dumps(data))
```

**Warming Without Impact:**
```python
def warm_cache_gradually():
    """Warm cache gradually to avoid impact"""
    keys_to_warm = get_keys_to_warm()
    
    for key in keys_to_warm:
        try:
            data = db.get(key)
            if data:
                r.setex(key, 3600, json.dumps(data))
            
            # Small delay to avoid overwhelming
            time.sleep(0.01)
        except Exception as e:
            logger.error(f"Failed to warm {key}: {e}")
```

**Handling Failures:**
- Don't fail application if warming fails
- Log failures for monitoring
- Retry failed warm-ups
- Monitor cache hit rates

**Key Points:**
- Pre-populate frequently accessed data
- Warm gradually to avoid impact
- Monitor effectiveness
- Handle failures gracefully

---

## Redis Persistence Solutions

### Solution 16: RDB Persistence

**How RDB Works:**
- Takes point-in-time snapshots
- Saves entire dataset to disk
- Compressed binary format
- Fast to save and load

**Configuration:**
```redis
# redis.conf
save 900 1      # Save if 1+ keys changed in 900 seconds
save 300 10     # Save if 10+ keys changed in 300 seconds
save 60 10000   # Save if 10000+ keys changed in 60 seconds

dbfilename dump.rdb
dir /var/lib/redis
```

**Advantages:**
- Fast backup and restore
- Compact file size
- Good for disaster recovery
- Minimal performance impact

**Disadvantages:**
- May lose data between snapshots
- Not suitable for zero data loss
- Full dataset snapshot (can be slow for large datasets)

**Handling Failures:**
- Regular RDB backups
- Copy RDB files to backup location
- Test restore procedures
- Monitor snapshot success

**Key Points:**
- Good for backups and disaster recovery
- May lose recent data
- Fast save/restore
- Configure appropriate save intervals

---

### Solution 17: AOF (Append-Only File) Persistence

**How AOF Works:**
- Logs every write operation
- Appends to file on disk
- Replays log on restart
- More durable than RDB

**Configuration:**
```redis
# redis.conf
appendonly yes
appendfilename "appendonly.aof"

# Fsync options
appendfsync always    # Every write (safest, slowest)
appendfsync everysec  # Every second (balanced)
appendfsync no        # OS decides (fastest, less safe)
```

**Fsync Options:**
- **always**: Every write synced (safest, ~100 writes/sec)
- **everysec**: Sync every second (balanced, ~1000 writes/sec)
- **no**: OS decides (fastest, may lose 1-2 seconds)

**AOF Rewriting:**
```redis
# Automatic rewriting
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Manual rewriting
BGREWRITEAOF
```

**AOF vs RDB:**
- **AOF**: Better durability, slower recovery, larger files
- **RDB**: Faster recovery, may lose data, smaller files

**Key Points:**
- Better durability than RDB
- Slower recovery (replay log)
- Larger files
- Choose fsync based on requirements

---

### Solution 18: RDB + AOF Combination

**Configuration:**
```redis
# Enable both
save 900 1
appendonly yes
appendfsync everysec
```

**Benefits:**
- RDB for fast recovery
- AOF for durability
- Best of both worlds

**Recovery Process:**
1. Redis loads AOF if exists (more recent)
2. Falls back to RDB if no AOF
3. Replays operations from AOF

**Performance:**
- Slightly higher overhead
- Worth it for critical data
- Monitor disk I/O

**Key Points:**
- Use both for maximum safety
- RDB for speed, AOF for durability
- Slightly higher overhead
- Best for production critical systems

---

### Solution 19: Persistence Trade-offs

**No Persistence:**
- Fastest performance
- Data lost on restart
- Use only for cache

**RDB Only:**
- Fast recovery
- May lose recent data
- Good for backups

**AOF Only:**
- Better durability
- Slower recovery
- Larger files

**RDB + AOF:**
- Maximum durability
- Fast recovery option
- Higher overhead

**Choosing Strategy:**
- **Cache only**: No persistence
- **Can lose some data**: RDB
- **Need durability**: AOF
- **Critical data**: RDB + AOF

**Key Points:**
- Balance durability vs performance
- Choose based on requirements
- Test recovery procedures
- Monitor persistence performance

---

## Redis Replication & High Availability Solutions

### Solution 20: Redis Master-Slave Replication

**Setting Up Replication:**
```redis
# On slave (replica) server
# redis.conf
replicaof <master-ip> <master-port>
replica-read-only yes
```

**How Replication Works:**
1. Slave connects to master
2. Master sends RDB snapshot
3. Master streams write commands
4. Slave applies commands

**Monitoring Replication Lag:**
```redis
# Check replication info
INFO replication

# Key metrics:
# - master_repl_offset: Master's offset
# - slave_repl_offset: Slave's offset
# - Lag = master_repl_offset - slave_repl_offset
```

**Promoting Slave to Master:**
```redis
# On slave
REPLICAOF NO ONE

# Update other slaves to point to new master
REPLICAOF <new-master-ip> <new-master-port>
```

**Handling Failures:**
- Monitor replication status
- Alert on replication lag
- Automate failover (use Sentinel)
- Test failover procedures

**Key Points:**
- Simple replication setup
- Monitor replication lag
- Can promote slave manually
- Use Sentinel for automatic failover

---

### Solution 21: Redis Sentinel for High Availability

**Sentinel Configuration:**
```redis
# sentinel.conf
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

**How Sentinel Works:**
- Monitors master and slaves
- Detects failures
- Elects new master
- Updates configuration

**Quorum:**
- Need majority of Sentinels to agree
- 3 Sentinels: Need 2 to agree
- 5 Sentinels: Need 3 to agree

**Automatic Failover:**
1. Sentinel detects master failure
2. Quorum agrees on failure
3. Sentinel promotes slave to master
4. Updates other slaves
5. Notifies clients

**Limitations:**
- Split-brain possible (network partition)
- Manual configuration updates needed
- No automatic client reconnection

**Key Points:**
- Automatic failover
- Need odd number of Sentinels (3+)
- Quorum prevents split-brain
- Clients need Sentinel-aware libraries

---

### Solution 22: Redis Replication Lag

**Causes:**
- Slow network
- High write load
- Slow slave processing
- Large RDB transfers

**Monitoring:**
```python
def check_replication_lag():
    info = r.info('replication')
    master_offset = info['master_repl_offset']
    slave_offset = info['slave_repl_offset']
    lag = master_offset - slave_offset
    return lag

# Alert if lag > threshold
lag = check_replication_lag()
if lag > 1000000:  # 1MB lag
    alert("High replication lag detected")
```

**Minimizing Lag:**
- Use faster network
- Reduce write load
- Optimize slave processing
- Use multiple slaves for read scaling

**Handling Stale Reads:**
```python
def read_from_replica_allowed():
    lag = check_replication_lag()
    # Only read from replica if lag is acceptable
    return lag < ACCEPTABLE_LAG_THRESHOLD

if read_from_replica_allowed():
    # Read from replica
    value = replica_client.get('key')
else:
    # Read from master
    value = master_client.get('key')
```

**Key Points:**
- Monitor replication lag continuously
- Minimize lag through optimization
- Handle stale reads appropriately
- Alert on high lag

---

### Solution 23: Read Replicas and Load Distribution

**Load Distribution:**
```python
import random

replicas = [
    redis.Redis(host='replica1', port=6379),
    redis.Redis(host='replica2', port=6379),
    redis.Redis(host='replica3', port=6379)
]

def read_from_replica(key):
    # Round-robin or random selection
    replica = random.choice(replicas)
    return replica.get(key)

# Or use consistent hashing
def get_replica(key):
    hash_value = hash(key)
    index = hash_value % len(replicas)
    return replicas[index]
```

**Handling Failures:**
```python
def read_with_failover(key):
    replicas = get_replicas()
    
    for replica in replicas:
        try:
            return replica.get(key)
        except redis.ConnectionError:
            continue
    
    # All replicas failed, try master
    return master_client.get(key)
```

**Consistency:**
- Accept eventual consistency
- Read from master for critical reads
- Monitor replica lag
- Route based on consistency requirements

**Key Points:**
- Distribute reads across replicas
- Handle replica failures
- Accept eventual consistency
- Route based on requirements

---

## Redis Clustering Solutions

### Solution 24: Redis Cluster Setup

**Cluster Configuration:**
```bash
# Create cluster with 6 nodes (3 masters, 3 replicas)
redis-cli --cluster create \
  127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1
```

**How Cluster Works:**
- Divides keyspace into 16384 hash slots
- Each master handles subset of slots
- Replicas replicate master's slots
- Automatic failover

**Key Distribution:**
- Keys mapped to hash slots using CRC16
- Hash slot = CRC16(key) % 16384
- Keys with same hash tag go to same slot

**Adding Nodes:**
```bash
# Add new node
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000

# Reshard to new node
redis-cli --cluster reshard 127.0.0.1:7000
```

**Key Points:**
- Horizontal scaling
- Automatic sharding
- High availability
- Need 3+ masters minimum

---

### Solution 25: Hash Slots and Key Distribution

**Hash Slot Calculation:**
```python
import crc16

def get_hash_slot(key):
    # Extract hash tag if present
    start = key.find('{')
    end = key.find('}', start + 1)
    
    if start != -1 and end != -1:
        key = key[start+1:end]
    
    # Calculate slot
    slot = crc16.crc16xmodem(key.encode()) % 16384
    return slot
```

**Hash Tags:**
```python
# Without hash tag - keys distributed randomly
r.set('user:1001:name', 'John')
r.set('user:1001:email', 'john@example.com')
# May go to different nodes

# With hash tag - keys go to same slot
r.set('user:{1001}:name', 'John')
r.set('user:{1001}:email', 'john@example.com')
# Both go to same node
```

**Key Migration:**
- Automatic during resharding
- Keys migrated slot by slot
- Cluster remains available during migration
- Use MIGRATE command

**Key Points:**
- 16384 hash slots
- Use hash tags for co-location
- Automatic key distribution
- Keys migrate during resharding

---

### Solution 26: Redis Cluster Failover

**Failover Process:**
1. Replica detects master failure
2. Other nodes confirm failure
3. Replica promotes itself to master
4. Updates cluster configuration
5. Clients reconnect to new master

**Preventing Split-Brain:**
- Need majority of masters available
- Minimum 3 masters for cluster
- Quorum prevents split-brain

**Recovery:**
```bash
# Check cluster status
redis-cli --cluster check 127.0.0.1:7000

# Fix cluster if needed
redis-cli --cluster fix 127.0.0.1:7000
```

**Key Points:**
- Automatic failover
- Need quorum to prevent split-brain
- Cluster remains available
- Monitor cluster health

---

### Solution 27: Redis Cluster vs Sentinel

**Sentinel:**
- Single master, multiple replicas
- Manual sharding required
- Simpler setup
- Good for smaller scale

**Cluster:**
- Multiple masters (sharding)
- Automatic sharding
- More complex setup
- Good for larger scale

**When to Use:**
- **Sentinel**: Single dataset, need HA, simpler
- **Cluster**: Large dataset, need scaling, can shard

**Migration:**
- Plan migration carefully
- Test in staging
- Use Redis migration tools
- Monitor during migration

**Key Points:**
- Choose based on scale and requirements
- Sentinel for HA, Cluster for scaling
- Can migrate between them
- Test thoroughly

---

## Performance & Optimization Solutions

### Solution 28: Redis Performance Tuning

**Configuration Tuning:**
```redis
# redis.conf optimizations

# Memory
maxmemory 2gb
maxmemory-policy allkeys-lru

# Persistence (balance durability vs performance)
appendfsync everysec

# Network
tcp-backlog 511
timeout 0

# Clients
maxclients 10000
```

**Optimization Tips:**
- Use appropriate data structures
- Avoid large values
- Use pipelining for bulk operations
- Connection pooling
- Monitor slow queries

**Measuring Performance:**
```python
import time

def benchmark_operation(operation, iterations=10000):
    start = time.time()
    for _ in range(iterations):
        operation()
    end = time.time()
    return (end - start) / iterations * 1000  # ms per operation
```

**Key Points:**
- Tune configuration for workload
- Monitor performance metrics
- Optimize data structures
- Use appropriate persistence settings

---

### Solution 29: Pipelining and Batch Operations

**Pipelining:**
```python
# Without pipelining (slow)
for i in range(1000):
    r.set(f'key:{i}', f'value:{i}')

# With pipelining (fast)
pipe = r.pipeline()
for i in range(1000):
    pipe.set(f'key:{i}', f'value:{i}')
pipe.execute()
```

**Batch Operations:**
```python
# Batch GET
keys = ['key1', 'key2', 'key3', ...]
values = r.mget(keys)

# Batch SET
mapping = {'key1': 'value1', 'key2': 'value2', ...}
r.mset(mapping)
```

**Optimal Batch Size:**
- Depends on network latency
- Typically 50-100 commands per pipeline
- Test to find optimal size
- Monitor performance

**Key Points:**
- Pipelining reduces round trips
- Use for bulk operations
- Find optimal batch size
- Significant performance improvement

---

### Solution 30: Connection Pooling

**Connection Pool Configuration:**
```python
import redis

# Create connection pool
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=50,
    decode_responses=True
)

# Use pool
r = redis.Redis(connection_pool=pool)
```

**Pool Sizing:**
- Size based on concurrent operations
- Too small: Connection waits
- Too large: Resource waste
- Monitor connection usage

**Monitoring:**
```python
# Check pool stats
pool_info = pool.connection_kwargs
active_connections = len([c for c in pool._available_connections if c.is_connected()])
```

**Key Points:**
- Reuse connections
- Size pool appropriately
- Monitor connection usage
- Prevents connection exhaustion

---

### Solution 31: Redis Memory Optimization

**Memory Usage:**
```redis
# Check memory usage
INFO memory

# Key metrics:
# used_memory: Total memory used
# used_memory_peak: Peak memory usage
# mem_fragmentation_ratio: Fragmentation ratio
```

**Optimization Strategies:**
- Use appropriate data structures
- Compress large values
- Set appropriate TTLs
- Use eviction policies
- Monitor memory usage

**Eviction Policies:**
```redis
# redis.conf
maxmemory-policy allkeys-lru    # Evict least recently used
maxmemory-policy allkeys-lfu    # Evict least frequently used
maxmemory-policy volatile-lru   # Evict LRU from keys with TTL
maxmemory-policy noeviction     # Don't evict, return errors
```

**Key Points:**
- Monitor memory continuously
- Use appropriate eviction policy
- Optimize data structures
- Set TTLs appropriately

---

## Cache Invalidation & Patterns Solutions

### Solution 32: Cache Invalidation Strategies

**TTL-Based:**
```python
# Set TTL on write
r.setex('key', 3600, 'value')  # Expires in 1 hour
```

**Event-Based:**
```python
def update_user(user_id, data):
    # Update database
    db.update_user(user_id, data)
    
    # Invalidate cache
    r.delete(f'user:{user_id}')
    
    # Or update cache
    r.setex(f'user:{user_id}', 3600, json.dumps(data))
```

**Pattern-Based Invalidation:**
```python
def invalidate_pattern(pattern):
    """Invalidate all keys matching pattern"""
    keys = r.keys(pattern)
    if keys:
        r.delete(*keys)
```

**Cache Stampede Prevention:**
```python
import time
import threading

lock = threading.Lock()

def get_with_stampede_prevention(key, loader_func):
    # Try cache
    value = r.get(key)
    if value:
        return json.loads(value)
    
    # Acquire lock
    with lock:
        # Double-check cache
        value = r.get(key)
        if value:
            return json.loads(value)
        
        # Load from source
        value = loader_func()
        if value:
            r.setex(key, 3600, json.dumps(value))
    
    return value
```

**Key Points:**
- Multiple invalidation strategies
- Choose based on requirements
- Prevent cache stampede
- Ensure consistency

---

### Solution 33: TTL and Expiration Policies

**Setting TTL:**
```python
# Set with expiration
r.setex('key', 3600, 'value')  # Seconds
r.psetex('key', 1000, 'value')  # Milliseconds

# Set expiration on existing key
r.expire('key', 3600)
r.pexpire('key', 1000)

# Check TTL
ttl = r.ttl('key')  # Returns seconds until expiration
```

**Choosing TTL Values:**
- Based on data change frequency
- Balance freshness vs cache hits
- Shorter for frequently changing data
- Longer for stable data

**Sliding Expiration:**
```python
def get_with_sliding_expiration(key, loader_func, ttl=3600):
    value = r.get(key)
    if value:
        # Extend expiration on access
        r.expire(key, ttl)
        return json.loads(value)
    
    # Load and cache
    value = loader_func()
    if value:
        r.setex(key, ttl, json.dumps(value))
    return value
```

**Key Points:**
- Set appropriate TTLs
- Use sliding expiration for active data
- Monitor expiration patterns
- Balance freshness vs performance

---

### Solution 34: Cache Stampede Prevention

**Problem:**
- Cache expires
- Multiple requests hit database simultaneously
- Database overload

**Solutions:**

**1. Locking:**
```python
def get_with_lock(key, loader_func, lock_timeout=10):
    # Try cache
    value = r.get(key)
    if value:
        return json.loads(value)
    
    # Try to acquire lock
    lock_key = f'lock:{key}'
    lock_id = str(uuid.uuid4())
    
    if r.set(lock_key, lock_id, nx=True, ex=lock_timeout):
        try:
            # Double-check cache
            value = r.get(key)
            if value:
                return json.loads(value)
            
            # Load from source
            value = loader_func()
            if value:
                r.setex(key, 3600, json.dumps(value))
            return value
        finally:
            # Release lock
            if r.get(lock_key) == lock_id.encode():
                r.delete(lock_key)
    else:
        # Lock acquired by another process, wait and retry
        time.sleep(0.1)
        return get_with_lock(key, loader_func, lock_timeout)
```

**2. Probabilistic Early Expiration:**
```python
def get_with_early_refresh(key, loader_func, base_ttl=3600):
    value = r.get(key)
    ttl = r.ttl(key)
    
    if value:
        # Refresh probabilistically when close to expiration
        if ttl < base_ttl * 0.2 and random.random() < 0.1:
            # Refresh in background
            threading.Thread(target=lambda: refresh_cache(key, loader_func)).start()
        return json.loads(value)
    
    # Cache miss - load
    value = loader_func()
    if value:
        r.setex(key, base_ttl, json.dumps(value))
    return value
```

**Key Points:**
- Use locking to prevent stampede
- Early refresh to avoid expiration
- Monitor stampede occurrences
- Test prevention strategies

---

### Solution 35: Distributed Caching Patterns

**Cache Sharding:**
```python
def get_shard(key, num_shards=4):
    """Determine which shard a key belongs to"""
    hash_value = hash(key)
    return hash_value % num_shards

shards = [
    redis.Redis(host='redis1', port=6379),
    redis.Redis(host='redis2', port=6379),
    redis.Redis(host='redis3', port=6379),
    redis.Redis(host='redis4', port=6379)
]

def get(key):
    shard = get_shard(key)
    return shards[shard].get(key)
```

**Cache Coherence:**
- Invalidate on updates
- Use consistent hashing
- Handle node failures
- Monitor cache hit rates

**Key Points:**
- Shard cache across instances
- Ensure cache coherence
- Handle failures gracefully
- Monitor distribution

---

## Redis Pub/Sub Solutions

### Solution 36: Redis Pub/Sub Implementation

**Publisher:**
```python
r.publish('channel:news', 'Breaking news!')
r.publish('channel:sports', 'Game update')
```

**Subscriber:**
```python
pubsub = r.pubsub()
pubsub.subscribe('channel:news', 'channel:sports')

for message in pubsub.listen():
    if message['type'] == 'message':
        channel = message['channel'].decode()
        data = message['data'].decode()
        print(f"Received from {channel}: {data}")
```

**Pattern Subscription:**
```python
pubsub.psubscribe('channel:*')  # Subscribe to all channels matching pattern
```

**Limitations:**
- No message persistence
- No message acknowledgment
- No consumer groups
- Messages lost if no subscribers

**Key Points:**
- Simple pub/sub implementation
- No persistence or acknowledgment
- Good for real-time notifications
- Use Streams for persistent messaging

---

### Solution 37: Pub/Sub vs Streams

**Pub/Sub:**
- Simple, lightweight
- No persistence
- No acknowledgment
- Real-time only

**Streams:**
- Persistent messages
- Consumer groups
- Message acknowledgment
- Can replay messages

**When to Use:**
- **Pub/Sub**: Real-time notifications, can lose messages
- **Streams**: Need persistence, consumer groups, replay

**Key Points:**
- Choose based on requirements
- Pub/Sub for simple real-time
- Streams for persistent messaging
- Streams more feature-rich

---

## Redis Transactions & Lua Scripting Solutions

### Solution 38: Redis Transactions

**Basic Transaction:**
```python
pipe = r.pipeline()
pipe.multi()
pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.execute()
```

**Conditional Transaction:**
```python
def transfer_funds(from_account, to_account, amount):
    pipe = r.pipeline()
    while True:
        try:
            pipe.watch(f'account:{from_account}')
            balance = int(r.get(f'account:{from_account}') or 0)
            
            if balance < amount:
                pipe.unwatch()
                raise ValueError("Insufficient funds")
            
            pipe.multi()
            pipe.decrby(f'account:{from_account}', amount)
            pipe.incrby(f'account:{to_account}', amount)
            pipe.execute()
            break
        except redis.WatchError:
            # Retry on conflict
            continue
```

**Limitations:**
- No rollback on errors in transaction
- Commands queued, not executed immediately
- WATCH for conditional transactions
- Some commands not allowed in transactions

**Key Points:**
- Use MULTI/EXEC for atomicity
- WATCH for conditional transactions
- Handle WatchError for retries
- Understand limitations

---

### Solution 39: Lua Scripting in Redis

**Writing Lua Scripts:**
```lua
-- script.lua
local key = KEYS[1]
local increment = tonumber(ARGV[1])
local current = redis.call('GET', key) or 0
local new_value = tonumber(current) + increment
redis.call('SET', key, new_value)
return new_value
```

**Executing Scripts:**
```python
script = """
local key = KEYS[1]
local increment = tonumber(ARGV[1])
local current = redis.call('GET', key) or 0
local new_value = tonumber(current) + increment
redis.call('SET', key, new_value)
return new_value
"""

sha = r.script_load(script)
result = r.evalsha(sha, 1, 'counter', '10')
```

**Atomic Operations:**
- Scripts execute atomically
- No other commands interleave
- Good for complex operations
- Reduce round trips

**Key Points:**
- Lua scripts for complex atomic operations
- Execute atomically
- Reduce network round trips
- Cache scripts with SCRIPT LOAD

---

### Solution 40: Atomic Operations and Race Conditions

**Atomic Commands:**
```python
# Atomic increment
r.incr('counter')
r.incrby('counter', 10)

# Atomic set if not exists
r.setnx('lock', 'value')

# Atomic compare and set
r.set('key', 'new_value', xx=True)  # Only if exists
r.set('key', 'new_value', nx=True)  # Only if not exists
```

**Preventing Race Conditions:**
- Use atomic commands
- Use transactions (MULTI/EXEC)
- Use WATCH for conditional updates
- Use Lua scripts for complex operations

**Key Points:**
- Use atomic commands when possible
- Transactions for multiple operations
- WATCH for conditional updates
- Prevent race conditions

---

## Memory Management Solutions

### Solution 41: Redis Eviction Policies

**Eviction Policies:**
```redis
# redis.conf
maxmemory 2gb
maxmemory-policy allkeys-lru    # Evict least recently used from all keys
maxmemory-policy allkeys-lfu    # Evict least frequently used from all keys
maxmemory-policy volatile-lru   # Evict LRU from keys with TTL
maxmemory-policy volatile-lfu   # Evict LFU from keys with TTL
maxmemory-policy volatile-ttl    # Evict shortest TTL
maxmemory-policy noeviction     # Return errors, don't evict
```

**Choosing Policy:**
- **allkeys-lru**: General purpose, evicts any key
- **volatile-lru**: Only evicts keys with TTL
- **allkeys-lfu**: Evicts least frequently used
- **noeviction**: Don't evict, return errors

**Monitoring Evictions:**
```python
info = r.info('stats')
evictions = info['evicted_keys']
```

**Key Points:**
- Choose policy based on access patterns
- Monitor eviction rates
- Set appropriate maxmemory
- Balance memory vs performance

---

### Solution 42: Memory Limits and Configuration

**Setting Memory Limits:**
```redis
# redis.conf
maxmemory 2gb
maxmemory-policy allkeys-lru
```

**Handling Memory Pressure:**
- Monitor memory usage
- Set appropriate eviction policy
- Scale horizontally if needed
- Optimize data structures

**Monitoring:**
```python
info = r.info('memory')
used_memory = info['used_memory']
used_memory_peak = info['used_memory_peak']
mem_fragmentation_ratio = info['mem_fragmentation_ratio']
```

**Key Points:**
- Set maxmemory appropriately
- Monitor memory usage
- Handle memory pressure
- Optimize when needed

---

### Solution 43: Memory Fragmentation

**Causes:**
- Frequent allocations/deallocations
- Different sized values
- Memory not reclaimed immediately

**Monitoring:**
```python
info = r.info('memory')
frag_ratio = info['mem_fragmentation_ratio']

if frag_ratio > 1.5:
    # High fragmentation
    # Consider memory defragmentation
    pass
```

**Minimizing:**
- Use consistent value sizes
- Avoid frequent large allocations
- Use memory-efficient data structures
- Monitor fragmentation ratio

**Key Points:**
- Monitor fragmentation ratio
- Optimize allocation patterns
- Use appropriate data structures
- Consider defragmentation if high

---

## Security & Authentication Solutions

### Solution 44: Redis Authentication

**Password Authentication:**
```redis
# redis.conf
requirepass your_strong_password_here
```

**Client Authentication:**
```python
r = redis.Redis(
    host='localhost',
    port=6379,
    password='your_strong_password_here'
)
```

**ACLs (Access Control Lists):**
```redis
# Create user with specific permissions
ACL SETUSER alice on >password ~cached:* +get +set

# User alice can only access keys matching cached:*
# and can only use GET and SET commands
```

**Key Points:**
- Always use authentication in production
- Use strong passwords
- Use ACLs for fine-grained access
- Secure network access

---

### Solution 45: Redis Security Best Practices

**Network Security:**
```redis
# Bind to specific interface
bind 127.0.0.1

# Or use firewall rules
# Don't expose Redis to internet
```

**TLS/SSL:**
```redis
# redis.conf
port 0
tls-port 6380
tls-cert-file /path/to/cert.pem
tls-key-file /path/to/key.pem
```

**Best Practices:**
- Use authentication
- Bind to localhost or private network
- Use TLS for encryption
- Regular security updates
- Monitor access logs

**Key Points:**
- Multiple layers of security
- Don't expose to internet
- Use TLS for encryption
- Regular security audits

---

## Monitoring & Troubleshooting Solutions

### Solution 46: Redis Monitoring

**Key Metrics:**
```python
info = r.info()

# Memory
memory_used = info['used_memory']
memory_peak = info['used_memory_peak']

# Connections
connected_clients = info['connected_clients']

# Commands
total_commands_processed = info['total_commands_processed']
commands_per_sec = info['instantaneous_ops_per_sec']

# Keyspace
keyspace_hits = info['keyspace_hits']
keyspace_misses = info['keyspace_misses']
hit_rate = keyspace_hits / (keyspace_hits + keyspace_misses)
```

**Monitoring Tools:**
- redis-cli INFO
- RedisInsight
- Prometheus + redis_exporter
- Custom monitoring scripts

**Key Points:**
- Monitor key metrics continuously
- Set up alerts
- Use monitoring tools
- Track performance trends

---

### Solution 47: Redis Slow Log Analysis

**Enabling Slow Log:**
```redis
# redis.conf
slowlog-log-slower-than 10000  # Log commands slower than 10ms
slowlog-max-len 128  # Keep last 128 slow commands
```

**Analyzing Slow Log:**
```python
slow_commands = r.slowlog_get(10)  # Get last 10 slow commands

for command in slow_commands:
    duration = command['duration']  # microseconds
    command_args = command['command']
    timestamp = command['start_time']
    
    if duration > 10000:  # Slower than 10ms
        print(f"Slow command: {command_args}, duration: {duration}us")
```

**Common Causes:**
- Large values
- Complex operations (SORT, KEYS)
- Network latency
- High load

**Key Points:**
- Enable slow log
- Analyze regularly
- Optimize slow operations
- Set appropriate threshold

---

### Solution 48: Redis Debugging and Troubleshooting

**Connection Issues:**
```python
try:
    r.ping()
except redis.ConnectionError as e:
    print(f"Connection failed: {e}")
    # Check network, firewall, Redis status
```

**Performance Issues:**
- Check slow log
- Monitor memory usage
- Check replication lag
- Analyze command patterns

**Memory Issues:**
- Check memory usage
- Monitor evictions
- Check fragmentation
- Optimize data structures

**Key Points:**
- Systematic debugging approach
- Use monitoring tools
- Check logs
- Test in isolation

---

## Design Patterns & Architecture Solutions

### Solution 49: Distributed Locks with Redis

**Implementation:**
```python
import uuid
import time

def acquire_lock(conn, lock_name, acquire_timeout=10, lock_timeout=10):
    identifier = str(uuid.uuid4())
    end = time.time() + acquire_timeout
    
    while time.time() < end:
        if conn.set(f'lock:{lock_name}', identifier, nx=True, ex=lock_timeout):
            return identifier
        time.sleep(0.001)
    return False

def release_lock(conn, lock_name, identifier):
    pipe = conn.pipeline(True)
    while True:
        try:
            pipe.watch(f'lock:{lock_name}')
            if pipe.get(f'lock:{lock_name}') == identifier.encode():
                pipe.multi()
                pipe.delete(f'lock:{lock_name}')
                pipe.execute()
                return True
            pipe.unwatch()
            break
        except redis.WatchError:
            pass
    return False
```

**Key Points:**
- Use SET with NX and EX
- Store unique identifier
- Use WATCH for safe release
- Handle lock expiration

---

### Solution 50: Rate Limiting with Redis

**Token Bucket:**
```python
def rate_limit(conn, user_id, limit=100, window=3600):
    key = f'rate_limit:{user_id}'
    current = conn.incr(key)
    
    if current == 1:
        conn.expire(key, window)
    
    return current <= limit
```

**Sliding Window:**
```python
def sliding_window_rate_limit(conn, user_id, limit=100, window=60):
    now = time.time()
    key = f'rate_limit:{user_id}'
    
    pipe = conn.pipeline()
    pipe.zremrangebyscore(key, 0, now - window)
    pipe.zcard(key)
    pipe.zadd(key, {str(now): now})
    pipe.expire(key, window)
    results = pipe.execute()
    
    return results[1] < limit
```

**Key Points:**
- Multiple algorithms available
- Use appropriate algorithm
- Set limits based on requirements
- Monitor rate limit hits

---

### Solution 51: Session Storage with Redis

**Storing Sessions:**
```python
def create_session(user_id):
    session_id = str(uuid.uuid4())
    session_data = {
        'user_id': user_id,
        'created_at': time.time(),
        'last_activity': time.time()
    }
    
    r.setex(f'session:{session_id}', 3600, json.dumps(session_data))
    return session_id

def get_session(session_id):
    data = r.get(f'session:{session_id}')
    if data:
        session = json.loads(data)
        # Update last activity
        session['last_activity'] = time.time()
        r.setex(f'session:{session_id}', 3600, json.dumps(session))
        return session
    return None
```

**Key Points:**
- Store sessions with TTL
- Update activity on access
- Handle session expiration
- Scale horizontally

---

### Solution 52: Caching Layer Architecture

**Multi-Layer Cache:**
```python
class MultiLayerCache:
    def __init__(self):
        self.l1_cache = {}  # In-memory (fastest, smallest)
        self.l2_cache = redis.Redis()  # Redis (fast, larger)
        self.db = Database()  # Database (slowest, largest)
    
    def get(self, key):
        # L1 cache
        if key in self.l1_cache:
            return self.l1_cache[key]
        
        # L2 cache
        value = self.l2_cache.get(key)
        if value:
            self.l1_cache[key] = json.loads(value)
            return json.loads(value)
        
        # Database
        value = self.db.get(key)
        if value:
            self.l2_cache.setex(key, 3600, json.dumps(value))
            self.l1_cache[key] = value
        return value
```

**Key Points:**
- Multiple cache layers
- L1 fastest, L2 larger
- Balance speed vs memory
- Ensure cache coherence

---

### Solution 53: Redis as Message Queue

**Using Lists:**
```python
# Producer
r.rpush('queue:orders', json.dumps(order))

# Consumer
order_data = r.blpop('queue:orders', timeout=10)
if order_data:
    order = json.loads(order_data[1])
    process_order(order)
```

**Using Streams:**
```python
# Producer
r.xadd('queue:orders', {'order': json.dumps(order)})

# Consumer with consumer group
r.xreadgroup('processors', 'consumer1', {'queue:orders': '>'}, count=1)
```

**Key Points:**
- Lists for simple queues
- Streams for persistent queues
- Handle failures appropriately
- Consider dedicated message queues for complex needs

---

### Solution 54: Redis for Real-Time Analytics

**Counters:**
```python
r.incr('page:views:2024:01:01')
r.incrby('user:1001:clicks', 10)
```

**Leaderboards:**
```python
r.zadd('leaderboard', {'player1': 100, 'player2': 200})
top_players = r.zrevrange('leaderboard', 0, 9, withscores=True)
```

**Time-Series:**
```python
timestamp = int(time.time())
r.zadd('events:user:1001', {f'event:{timestamp}': timestamp})
```

**Key Points:**
- Use appropriate data structures
- Aggregate in real-time
- Handle high volume
- Query efficiently

---

### Solution 55: Cache-Aside with Database Integration

**Implementation:**
```python
def get_user(user_id):
    cache_key = f'user:{user_id}'
    
    # Try cache
    user = r.get(cache_key)
    if user:
        return json.loads(user)
    
    # Cache miss - database
    user = db.get_user(user_id)
    if user:
        r.setex(cache_key, 3600, json.dumps(user))
    return user

def update_user(user_id, data):
    # Update database
    db.update_user(user_id, data)
    
    # Invalidate cache
    r.delete(f'user:{user_id}')
```

**Key Points:**
- Check cache first
- Update database on miss
- Invalidate on updates
- Handle failures

---

## Advanced Topics Solutions

### Solution 56: Redis Modules

**Popular Modules:**
- **RedisSearch**: Full-text search
- **RedisGraph**: Graph database
- **RedisTimeSeries**: Time-series data
- **RedisJSON**: JSON data type

**Using Modules:**
```bash
# Load module
redis-server --loadmodule /path/to/module.so

# Or in redis.conf
loadmodule /path/to/module.so
```

**Key Points:**
- Extend Redis functionality
- Many useful modules available
- Test modules thoroughly
- Consider performance impact

---

### Solution 57: Redis Streams Advanced Features

**Consumer Groups:**
```python
# Create consumer group
r.xgroup_create('events', 'processors', id='0', mkstream=True)

# Read as consumer
events = r.xreadgroup('processors', 'consumer1', {'events': '>'}, count=10)

# Acknowledge
for stream, messages in events:
    for msg_id, data in messages:
        process(data)
        r.xack('events', 'processors', msg_id)
```

**Key Points:**
- Consumer groups for parallel processing
- Message acknowledgment
- Handle failures
- Replay capabilities

---

### Solution 58: Redis Cluster Resharding

**Adding Nodes:**
```bash
# Add node
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000

# Reshard
redis-cli --cluster reshard 127.0.0.1:7000
```

**Key Points:**
- Plan resharding carefully
- Monitor during resharding
- Handle failures
- Test in staging first

---

### Solution 59: Redis Backup and Recovery

**Backup:**
```bash
# RDB backup
redis-cli BGSAVE
cp /var/lib/redis/dump.rdb /backup/

# AOF backup
cp /var/lib/redis/appendonly.aof /backup/
```

**Recovery:**
```bash
# Stop Redis
# Copy backup files
# Start Redis
```

**Key Points:**
- Regular backups
- Test recovery
- Multiple backup locations
- Document procedures

---

### Solution 60: Redis Migration and Upgrades

**Migration:**
```bash
# Use redis-dump or custom scripts
# Test thoroughly
# Plan downtime
```

**Upgrades:**
```bash
# Backup first
# Test in staging
# Rolling upgrade if possible
# Monitor after upgrade
```

**Key Points:**
- Plan carefully
- Test thoroughly
- Minimize downtime
- Monitor after changes

---

### Solution 61: Redis vs Other Caching Solutions

**Redis vs Memcached:**
- **Redis**: More data types, persistence, replication
- **Memcached**: Simpler, faster for simple key-value

**Redis vs Hazelcast:**
- **Redis**: Simpler, better for caching
- **Hazelcast**: More features, distributed computing

**Key Points:**
- Compare features
- Consider use case
- Test performance
- Choose appropriately

---

### Solution 62: Redis in Cloud Environments

**AWS ElastiCache:**
- Managed Redis service
- Automatic backups
- Multi-AZ support
- Monitoring included

**Azure Cache for Redis:**
- Managed service
- Enterprise features
- Integration with Azure services

**Key Points:**
- Managed services simplify operations
- Consider costs
- Use cloud-specific features
- Monitor usage

---

### Solution 63: Redis Performance Testing

**Benchmarking:**
```bash
# Use redis-benchmark
redis-benchmark -h localhost -p 6379 -n 100000 -c 50

# Test specific operations
redis-benchmark -h localhost -p 6379 -t set,get -n 100000
```

**Key Points:**
- Benchmark before production
- Test under load
- Identify bottlenecks
- Optimize based on results

---

### Solution 64: Redis High Availability Architecture

**Components:**
- Multiple Redis instances
- Sentinel for failover
- Or Cluster for scaling
- Monitoring and alerts

**Key Points:**
- Design for failures
- Automate failover
- Monitor health
- Test failover procedures

---

### Solution 65: Redis for Microservices

**Patterns:**
- Cache per service
- Shared cache for common data
- Distributed coordination
- Session storage

**Key Points:**
- Isolate when possible
- Share when beneficial
- Handle distributed coordination
- Monitor per service

---

## Summary

This comprehensive guide covers all major Redis and caching topics for SDE 2 interviews:

1. **Fundamentals**: Architecture, data types, commands
2. **Data Structures**: Strings, Hashes, Lists, Sets, Sorted Sets, Bitmaps, Streams
3. **Caching Strategies**: Cache-aside, write-through, write-behind, read-through
4. **Persistence**: RDB, AOF, trade-offs
5. **Replication & HA**: Master-slave, Sentinel, replication lag
6. **Clustering**: Cluster setup, hash slots, failover
7. **Performance**: Tuning, pipelining, connection pooling, memory optimization
8. **Cache Patterns**: Invalidation, TTL, stampede prevention, distributed caching
9. **Pub/Sub**: Messaging, Streams comparison
10. **Transactions**: Transactions, Lua scripting, atomic operations
11. **Memory**: Eviction policies, limits, fragmentation
12. **Security**: Authentication, ACLs, best practices
13. **Monitoring**: Metrics, slow log, troubleshooting
14. **Design Patterns**: Distributed locks, rate limiting, session storage, analytics
15. **Advanced**: Modules, Streams, resharding, backup, migration, cloud deployment

**Total Coverage: 65 Problems with Detailed Solutions**

Each solution includes code examples, configurations, and best practices. Use these as a reference for interview preparation and real-world implementations.
