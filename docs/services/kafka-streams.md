# Kafka Streams

## Overview
Kafka Streams is a client library for building streaming applications that process data in Apache Kafka. It enables stateful transformations, aggregations, and joins directly on data flowing through Kafka topics without requiring a separate stream processing cluster.

## Use Cases

### In Google News System
1. **Real-time Deduplication**: Identify duplicate articles using windowed aggregations
2. **Content Enrichment**: Add metadata, classification, and tags to articles in real-time
3. **Aggregations**: Count articles per topic, track trending stories
4. **Joins**: Correlate articles with user profiles for personalization
5. **Stateful Processing**: Track article versions and update history
6. **Time-windowed Analytics**: Calculate moving averages of engagement metrics

## How It Works

### Core Architecture

```
Input Topic (raw-articles)
        ↓
  Kafka Streams App
  (Embedded in your service)
        ├─ Stream Processing Topology
        │  ├─ Filter: Remove low-quality articles
        │  ├─ Map: Normalize and enrich content
        │  ├─ Aggregate: Group by topic with 5-min windows
        │  └─ Join: Correlate with user interests
        ↓
  State Stores (Local + Remote)
        ├─ In-Memory Cache
        └─ RocksDB (Persistent)
        ↓
Output Topic (processed-articles)
```

### Processing Topology Example

```
Source: raw-articles-topic
    ↓
[Filter] Remove spam/low-quality
    ↓
[Map] Parse and normalize
    ↓
[Branch]
    ├─→ [Aggregate] Count by topic (5-min window) → topic-counts-store
    │       ↓
    │   trending-topics-topic
    │
    └─→ [Join] With user-interests (KStream-KTable join)
            ↓
        user-personalized-articles
```

## Configuration Options

### Key Parameters

| Parameter | Description | Recommended Value |
|-----------|-------------|-------------------|
| `application.id` | Unique app identifier for state | `article-processor-v1` |
| `bootstrap.servers` | Kafka brokers | `broker1:9092,broker2:9092` |
| `default.key.serde` | Key serialization | StringSerde |
| `default.value.serde` | Value serialization | JsonSerde |
| `state.dir` | Local state store location | `/var/kafka-streams` |
| `num.stream.threads` | Parallelism within app | 8-16 |
| `cache.max.bytes.buffering` | Local cache size | 10 MB |
| `commit.interval.ms` | State commit frequency | 30,000 ms |

### Topology Settings

```properties
# High throughput
num.stream.threads=16
cache.max.bytes.buffering=50000000

# High consistency
commit.interval.ms=5000
state.cleanup.delay.ms=600000

# Interactive queries
rocksdb.config.setter=org.apache.kafka.streams.state.internals.RocksDBConfigSetter
```

## Effective Use Cases

### Ideal Scenarios
✅ Real-time transformations with sub-second latency
✅ Stateful processing (aggregations, windowing, joins)
✅ Processing volume: 100K to 1M events/second per instance
✅ No external state management needed
✅ Single Kafka cluster as both input and output
✅ Event-time semantics important

### Not Suitable For
❌ Cross-cluster federation or data movement
❌ Complex SQL queries (use Flink or Spark)
❌ Transaction support across multiple systems
❌ Strict ordering across independent streams
❌ ML model serving (use specialized platforms)

## Performance Characteristics

### Throughput
- Per stream thread: 500K-2M events/second
- Scales linearly with number of threads and app instances

### Latency
- Processing latency: 1-10ms (in-memory operations)
- End-to-end latency: 100-1000ms (includes Kafka broker)
- State store access: <1ms (RocksDB)

### State Management
- Local state with changelog topic for recovery
- Automatic rebalancing on app restart or scaling

## Integration with Google News System

### Real-time Article Deduplication Pipeline

```python
from kafka import KafkaConsumer, KafkaProducer
from kafka.streams import KafkaStreams, StreamsBuilder
from kafka.streams.kstream import KStream
from kafka.streams.state import WindowStore

builder = StreamsBuilder()

# Source stream of raw articles
articles = builder.stream("raw-articles")

# Filter low-quality content
quality_articles = articles.filter(
    lambda key, article: article['word_count'] > 50
)

# Aggregate by content hash (5-minute tumbling window)
article_dedup = quality_articles \
    .map(lambda key, article: (article['content_hash'], article)) \
    .groupByKey() \
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5))) \
    .aggregate(
        initializer=lambda: [],
        adder=lambda key, value, agg: agg + [value],
        materialized=Materialized.as("article-dedup-store")
    )

# Output only first article per cluster
deduplicated = article_dedup \
    .mapValues(lambda articles: articles[0]) \
    .toStream() \
    .map(lambda key, value: (value['article_id'], value))

deduplicated.to("deduplicated-articles")

topology = builder.build()
streams = KafkaStreams(topology, config)
streams.start()
```

### Real-time Trending Topics

```python
# Time-windowed aggregation (1-minute windows)
trending = articles \
    .selectKey(lambda k, v: v['topic']) \
    .groupByKey() \
    .windowedBy(TimeWindows.of(Duration.ofMinutes(1))) \
    .count(Materialized.as("topic-count-store")) \
    .toStream() \
    .filter(lambda key, count: count > 100)  # Trending threshold

trending.to("trending-topics")
```

## Operational Considerations

### Deployment Strategy
- Run multiple instances for fault tolerance (replication factor 3)
- Scale horizontally by adding stream threads or app instances
- Assign one app instance per Kafka partition (optimal)

### State Management Best Practices
- Set appropriate retention for changelog topics (7-30 days)
- Use persistent state stores (RocksDB) for larger state
- Enable interactive queries for debugging
- Monitor state store size to plan capacity

### Monitoring Metrics
- Processing rate (records/sec per thread)
- Processing latency (min, avg, max, p99)
- State store size and restore time
- Consumer lag of source topics
- Task rebalancing frequency

### Failure Recovery
- Changelog topics store state mutations
- On restart, state is restored from changelog
- Recovery time = changelog size / consumer speed

## State Store Types

### In-Memory Store
```
Use: Temporary aggregations, fast lookups
Persistence: None (lost on restart)
Restore Time: Instant
Size Limit: RAM available
```

### RocksDB Store
```
Use: Large state, requiring persistence
Persistence: Local disk + changelog topic
Restore Time: Minutes to hours (depends on size)
Size Limit: Disk space available
```

### Window Store
```
Use: Time-windowed aggregations
Retention: Window duration + grace period
Example: 5-minute windows with 10-min grace period
```

## Comparison with Alternatives

| Feature | Kafka Streams | Flink | Spark Streaming |
|---------|---------------|-------|-----------------|
| **Latency** | 1-10ms | 1-5ms | 500-2000ms |
| **Setup** | Simple (library) | Complex (cluster) | Complex (cluster) |
| **State** | Built-in (RocksDB) | Flexible backends | External (checkpoint) |
| **Cost** | Low (Kafka only) | Medium | Medium to High |
| **Learning Curve** | Easy | Steep | Medium |
| **Use Case** | Kafka-native | General streaming | Batch + streaming |

## Cost Estimation

For processing 1M articles/day with Kafka Streams:
- **Application Servers**: 2-4 instances × $200/month = $400-800/month
- **Storage (changelog topics)**: 50 GB retention = $50/month
- **Monitoring & Observability**: $100/month
- **Total**: ~$550-950/month (no separate cluster needed)

