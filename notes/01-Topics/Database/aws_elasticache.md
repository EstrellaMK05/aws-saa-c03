---
aliases: [Amazon ElastiCache, ElastiCache, Database Caching]
tags: [aws/saa, database]
---

# Amazon ElastiCache

## Mental Model

**Serve reusable data from memory to reduce database work. Decide explicitly how stale or disposable that data may be.**

A cache normally needs application integration and a strategy for misses, updates, expiration and failure.

## Core

### Engines and deployment modes

| Option | Fit | Important qualification |
|---|---|---|
| Valkey / Redis OSS | Rich data structures, sorted sets, sessions and replication/failover configurations | Choose appropriate shards, replicas and durability settings |
| Node-based Memcached | Simple distributed key-value caching | Nodes partition cached data; no native primary/replica failover model like Valkey/Redis OSS |
| ElastiCache Serverless | Reduce capacity-management work | Automatically scales and replicates across AZs; supported engines include Memcached |

Do not extend node-based Memcached limitations to every Serverless deployment. Also avoid the outdated universal claim that Valkey/Redis uses only one thread: execution and I/O threading capabilities depend on engine/version.

### Cache patterns

| Pattern | Flow | Trade-off |
|---|---|---|
| Cache-aside / lazy loading | Read cache; on miss query database and populate cache | Caches useful data, but first access is slower and cached values can become stale |
| Write-through application pattern | Update database and cache on the write path | Improves freshness when implemented correctly; adds write work and may cache unused data |
| TTL | Expire cached entries after a chosen interval | Bounds age, but expiration/misses increase backend load |
| Invalidation | Remove/update entries when underlying data changes | Requires correct handling of races, failures and all write paths |

ElastiCache is not automatically a transparent SQL query-result cache. “Write-through” also does not make database and cache updates one atomic distributed transaction.

Avoid a **cache stampede** when many clients rebuild the same missing/expired entries: consider staggered expirations, request coalescing and controlled refresh. Plan for database load when the cache is cold or unavailable.

### Scaling and HA

- **More memory/larger nodes:** increase available capacity on nodes.
- **More shards:** distribute the dataset and write/processing load for compatible clustered workloads.
- **Read replicas:** offload eligible reads and provide failover targets; replicas do not create independent writers for the same shard.
- **Multi-AZ failover:** improves availability, but ordinary asynchronous replication may lose recent writes.

For sessions, first decide whether losing a recent update or forcing reauthentication is acceptable. Moving sessions off EC2 solves instance-local state, but does not prove session data can never be lost.

### Durability is a configuration decision

Current node-based **ElastiCache for Valkey supports optional durability using a Multi-AZ transaction log** in supported versions/configurations. Synchronous durable writes acknowledge after persistence; asynchronous durable writes trade lower latency for possible recent-write loss.

This is different from simply enabling ordinary replica failover. Durable storage also does not prevent intentional TTL expiry, eviction or application deletion; configure those policies for the data's role. Compare with MemoryDB when selecting a durable in-memory primary database.

### Security and monitoring

Use VPC/security-group controls, supported TLS/encryption and suitable authentication/RBAC. Engine and deployment support differs; do not assume every security mechanism is universal.

Monitor hit rate, memory, evictions, CPU, connections and replication lag where available. Low hit rate may indicate poor keys, unsuitable cached data or expiry settings rather than insufficient node capacity.

## Comparisons

| Need | Starting point |
|---|---|
| Repeated application/SQL result reads with tolerated staleness | ElastiCache |
| Additional fresh-enough SQL query execution capacity | RDS/Aurora read replicas |
| DynamoDB-specific eventual read acceleration | [[aws_dynamodb|DAX]] |
| Too many relational database connections | RDS Proxy |
| Sorted-set leaderboard | Valkey/Redis-compatible data structures |
| Durable in-memory primary data | MemoryDB or a suitable supported durable ElastiCache configuration; compare guarantees |

## Exam Traps

- **Multi-AZ alone does not promise zero lost cache writes.**
- **Replication is not sharding.** One duplicates data; the other partitions it.
- **More replicas do not scale one primary's write workload.**
- **A TTL does not guarantee immediate freshness after a database update.**
- **Memcached Serverless has managed Multi-AZ replication**, unlike the classic node-based Memcached model.
- **Durability does not disable eviction or expiration.**

## Scenario Check

**An application repeatedly reads product descriptions and accepts brief staleness.** Cache-aside with suitable invalidation/TTL can reduce database load. If the requirement instead says every read must reflect the latest committed inventory change, an ordinary eventual cache is not sufficient just because it is fast.

## 30-Second Review

ElastiCache accelerates reusable data, but applications must handle misses and freshness. Cache-aside fills on demand; write-through updates on writes; TTL and invalidation control staleness. Shards scale datasets/writes, replicas support reads/failover. Ordinary asynchronous HA can lose recent updates. Serverless and node-based engines have different capabilities. Durable Valkey options exist, but expiry and eviction still need deliberate configuration.

## Sources

- [ElastiCache architecture and deployment options](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.corecomponents.html)
- [Multi-AZ failover](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/AutoFailover.html)
- [Caching strategies](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Strategies.html)
- [Durability](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/durability.html)
- [Durability and eviction options](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Durability.Options.html)

Reviewed: 2026-09-22. Back to [[database_overview|Databases]].
