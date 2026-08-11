# Databases

## Introduction to Databases
A database is a structured collection of data that supports efficient storage, retrieval, and manipulation. Databases are used to power application state, indexes, analytics, and user-facing features. Key concerns include durability, availability, latency, throughput, consistency, and operational complexity.

Core responsibilities:
- Persisting structured or semi-structured data
- Supporting queries (point reads, range scans, aggregations)
- Ensuring durability and recoverability after failures
- Scaling reads and writes as demand grows

## Types of Databases
1. Relational (RDBMS)
   - Examples: PostgreSQL, MySQL, MariaDB
   - Strengths: Strong consistency, ACID transactions, complex joins, mature tooling
   - Use cases: Financial data, canonical metadata, transactional operations

2. Key-Value Stores
   - Examples: Redis, DynamoDB, RocksDB
   - Strengths: Simple API, extremely low latency, high throughput
   - Use cases: Caches, sessions, feature flags

3. Document Stores
   - Examples: MongoDB, Couchbase
   - Strengths: Flexible schema, JSON-like documents, rich query support
   - Use cases: Content, user profiles, product catalogs

4. Column-Family Stores / Wide-Column
   - Examples: Cassandra, HBase
   - Strengths: High write throughput, horizontal scalability, time-series friendly
   - Use cases: Event logs, metrics, large-scale ingestion

5. Search Engines / Text Indexes
   - Examples: Elasticsearch, OpenSearch, Solr
   - Strengths: Full-text search, faceting, inverted indexes
   - Use cases: Article search, relevance ranking, analytics

6. Graph Databases
   - Examples: Neo4j, JanusGraph
   - Strengths: Traversal, relationships, graph analytics
   - Use cases: Recommendations, social graphs, entity linking

7. Time-Series Databases
   - Examples: InfluxDB, TimescaleDB
   - Strengths: Efficient storage and queries for time-series data
   - Use cases: Monitoring, metrics, event trends

8. NewSQL / Distributed SQL
   - Examples: CockroachDB, YugabyteDB
   - Strengths: SQL semantics with horizontal scaling
   - Use cases: Globally-distributed transactional workloads

## Data Replication
Replication copies data from one node (primary) to one or more replicas to improve availability and read scalability.

Replication topologies:
- Primary-Replica (Master-Slave): A single primary accepts writes and replicates to replicas that serve reads.
- Multi-Primary / Multi-Master: Multiple primaries accept writes and resolve conflicts via consensus or application logic.
- Leaderless (Quorum-based): Writes accepted at any node, consistency enforced via quorums (e.g., Dynamo-style).

Replication modes:
1. Asynchronous replication
   - Primary writes locally and returns success before replicas acknowledge.
   - Pros: Low write latency, high throughput.
   - Cons: Replica lag, potential for data loss if primary fails before replication.

2. Semi-synchronous (or acknowledged) replication
   - Primary waits for at least one (or configurable) replica to acknowledge write before returning success.
   - Pros: Balance between durability and latency; reduces data loss window.
   - Cons: Higher write latency; still possible to lose data if all replicas fail.

3. Synchronous replication
   - Primary waits for acknowledgments from all required replicas (or quorum) before returning success.
   - Pros: Stronger durability and consistency.
   - Cons: Higher latency and lower availability if replicas are slow/unreachable.

Mechanisms used to sync primary → replica:
- Write-Ahead Log (WAL) shipping: Primary appends to a WAL and either streams records or ships periodic segments to replicas. Replicas apply WAL to reach the primary state (Postgres streaming replication is WAL-based).
- Binlog / Logical replication: Primary emits logical change events (e.g., row changes) that replicas consume and apply. Often used for heterogeneous replication or selective replication.
- Consensus-based replication (Raft/Paxos): All replicas participate in a replicated log with leader election; writes go through the consensus protocol (e.g., etcd, CockroachDB). This gives strong consistency and leader-driven replication.
- Snapshot + incremental updates: Replica starts from a snapshot and then applies incremental WAL/binlog to catch up.

Practical example (PostgreSQL streaming replication):
- Primary writes changes to WAL.
- A WAL sender process streams WAL records to standby.
- Standby writes WAL to local disk and replays it to apply changes.
- Replication delay (lag) is measured as last WAL position applied by standby vs primary.

Observability & metrics for replication:
- Replica lag (time and WAL bytes)
- Replication throughput
- Replication error counts and retry rates
- Last applied LSN/SCN/offset on replicas

Failure modes and mitigation:
- Lag spikes → read staleness: Use `read_from_replicas=false` for critical reads, or use semi-sync to reduce window.
- Split-brain in multi-primary → use consensus or external coordinator.
- Network partitions → prefer leader election with quorum and automatic failover tooling.

## Data Partitioning (Sharding)
Partitioning distributes data across multiple nodes to scale writes and storage capacity.

Common partitioning strategies:
1. Range partitioning
   - Key ranges map to shards (e.g., time-based ranges).
   - Pros: Good for range scans and time-series.
   - Cons: Hot shards when traffic skews to recent ranges.

2. Hash partitioning
   - Hash(key) % N → shard.
   - Pros: Even distribution; simple.
   - Cons: Poor locality for range queries; re-sharding when N changes is expensive.

3. List partitioning
   - Assign specific key values to particular shards.
   - Pros: Useful for geographic or tenant-based isolation.
   - Cons: Manual management, potential imbalance.

4. Consistent hashing
   - Hash ring with virtual nodes; minimizes movement when nodes change.
   - Pros: Smooth scaling, lower re-sharding cost.
   - Cons: More complex, possible uneven load without virtual nodes.

5. Directory-based (lookup) partitioning
   - A central mapping (or service) maps key → shard.
   - Pros: Flexible, can implement hot-key routing.
   - Cons: Centralized lookup can be a bottleneck; needs high availability.

Sharding concerns:
- Re-sharding (splitting/merging shards) requires data movement and careful coordination.
- Hot keys: single keys that get disproportionate traffic; mitigate with request-level caching, write coalescing, or per-key workers.
- Cross-shard transactions: Support is limited; either avoid multi-shard transactions or use two-phase commit (costly).
- Secondary indexes across shards: More complex — may need global index or fan-out queries.

## Trade-Offs in Databases
Key trade-offs often follow CAP theorem and operational constraints.

1. CAP Theorem
   - Consistency, Availability, Partition tolerance — you can guarantee at most two simultaneously in the presence of network partitions.
   - Design choice depends on tolerance for stale reads vs. availability under partition.

2. Consistency vs Latency
   - Strong consistency (synchronous replication, consensus) increases write latency.
   - Eventual consistency allows low-latency writes but increases complexity for correctness.

3. Availability vs Durability
   - Prioritizing availability (respond during partitions) risks data loss or divergence.
   - Durability-focused systems may refuse writes if quorum can't be obtained.

4. Operational Complexity vs Performance
   - Distributed systems (sharding, consensus) scale well but are operationally harder (monitoring, backups, re-sharding).
   - Simpler single-node setups are easier to operate but limited in scale.

5. Read Scaling vs Write Scaling
   - Replication helps read scaling but doesn’t improve write throughput (writes still hit primary).
   - Partitioning (sharding) increases write throughput by distributing writes across shards.

6. Secondary Indexes & Query Flexibility vs Storage/Write Cost
   - More indexes improve query latency but increase write amplification and storage usage.

## How primary → replica sync happens (detailed)
There are several implementation patterns; the choice affects durability, latency, and failover behavior.

1. WAL / Physical replication (byte-level)
   - Primary records changes to a write-ahead log.
   - Replicas receive WAL segments or a WAL stream and apply them.
   - Guarantees binary-equivalent state (fast to apply, simple replay).
   - Typical in DB engines like PostgreSQL.

2. Logical replication / Change Events
   - Primary emits logical change events (INSERT/UPDATE/DELETE) or a logical representation of row changes.
   - Replicas or downstream consumers apply these changes in order.
   - Pros: Selective replication, cross-engine replication, schema evolution handling.
   - Example: MySQL binlog, Debezium CDC streams.

3. Streaming replication with acknowledgement modes
   - Modes:
     - `async`: primary returns immediately; replicas eventually catch up.
     - `semi-sync`: primary waits for at least one replica to persist the change.
     - `sync`: primary waits for required replicas/quorum.
   - Impact: stronger acks increase write latency but reduce data-loss risk.

4. Replicated log (consensus: Raft/Paxos)
   - Leader appends entry to a replicated log; followers replicate the entry.
   - A write is considered committed when a quorum acknowledges (Raft) and can be applied.
   - Provides strong consistency and safe leader failover.
   - Used by etcd, CockroachDB, Yugabyte.

5. Snapshot & incremental catch-up
   - When a replica joins, it starts from a consistent snapshot and then applies incremental WAL or binlog until caught up.
   - Snapshots speed initial provisioning but require transfer bandwidth.

6. Asynchronous replication via message bus
   - Primary writes changes to a durable message bus (Kafka). Replicas subscribe and apply changes.
   - Offers replayability, audit trail, and decoupling. Useful for analytics and cross-system replication.

Practical considerations for sync:
- Idempotency: Ensure applying the same change twice is safe or detect duplicates.
- Ordering: Maintain order for operations on the same key to avoid anomalies.
- Schema changes: Coordinate DDL propagation so replicas remain compatible.
- Backpressure & flow control: Prevent replicas from being overwhelmed (throttling, batching).
- Security: Encrypt replication traffic and authenticate nodes.

## Practical Patterns & Examples
- PostgreSQL streaming replication (executor WAL → standby apply; supports async/semi-sync)
- MySQL replication via binlog (logical replication; replicas replay statements or row events)
- MongoDB replica sets (oplog applied by secondaries; configurable write concern)
- Cassandra (Gossip + hinted handoff + commit log replication; eventual consistency with tunable consistency levels)
- CockroachDB (Raft consensus per range; strongly consistent distributed SQL)
- Using Kafka + Debezium for CDC: database changes → Kafka topics → downstream consumers or replica appliers

## Monitoring & Operations
Essential metrics:
- Replica lag (time & bytes)
- Write throughput and replication throughput
- Number of failed replication attempts
- Snapshot/restore durations for new replicas
- Consistency/health checks (read-your-writes tests)

Operational practices:
- Run at least 3 replicas (one primary + two replicas) for high availability
- Use semi-sync/sync for critical tables and async for less-critical workloads
- Automate failover with tools that understand replication state
- Test backups and restore regularly; verify replication after restore
- Plan and test re-sharding/migration operations ahead of traffic

## Summary and Recommendations
- Choose the database type that matches access patterns: transactions → RDBMS; low-latency key access → key-value; flexible schema → document store; analytics/search → search engine.
- Use replication for read scaling and availability; choose sync/semi-sync/async based on durability needs.
- Use partitioning to scale writes; pick a sharding key that minimizes hot keys and supports primary queries.
- Monitor replication lag and design client read policies to avoid stale reads when necessary.
- Favor idempotent consumers and change-data-capture patterns for cross-system replication and auditing.

