---
aliases: [Amazon DynamoDB, DynamoDB, DAX]
tags: [aws/saa, database]
---

# Amazon DynamoDB

## Mental Model

**Design the keys around the questions the application asks. DynamoDB scales key-based access; it does not make every scan efficient.**

It is a managed key-value/document database with Regional Multi-AZ resilience. Flexible attributes do not eliminate data-model design.

## Core

### Keys and access patterns

- A simple primary key uses a partition key; a composite primary key uses partition key plus sort key. The complete primary key uniquely identifies an item.
- The partition key distributes data; the sort key orders related items for that partition-key value.
- **GetItem:** retrieve by complete primary key. **Query:** equality on one partition-key value, with optional sort-key conditions. **Scan:** read through table/index contents.
- Filters are applied after reading candidate items. They do not save the read capacity already consumed by those items.
- High-cardinality keys help only when actual traffic is distributed. One extremely popular key can remain hot even in a table with millions of other keys. Consider appropriate sharding, caching or access-pattern changes.
- Maximum item size is **400 KB**, including attribute names. Keep large objects in S3 and store references/metadata where appropriate.

### Secondary indexes and consistency

| Property | GSI | LSI |
|---|---|---|
| Partition key | Can differ from base table | Same as base table |
| Sort key | Optional/different | Alternate sort key |
| Creation | With table or later | At table creation only |
| Read consistency | Eventual only | Eventual or strong |
| Provisioned capacity | Separate index capacity | Shares table capacity |
| Item-collection constraint | Different partitioning model | 10 GB per partition-key item collection, including relevant index data |

A GSI can support another access path, but its propagation is asynchronous. In provisioned mode, insufficient GSI write capacity can throttle related base-table writes. Project attributes according to query needs rather than copying every attribute without considering cost.

Base-table and LSI reads can request strong consistency. Ordinary reads default to eventual consistency. Strong reads are not a global snapshot of an entire changing dataset, and they do not make GSI reads strong.

### Capacity and cost

| Mode | Fit | Caveat |
|---|---|---|
| On-demand | Unknown or variable traffic with minimal capacity management | Still subject to quotas, configured limits, scaling behavior and hot-key constraints |
| Provisioned with optional Auto Scaling | Forecastable usage where capacity planning is worthwhile | Scaling is not instantaneous; plan sudden peaks and index capacity |

For standard nontransactional reads, one RCU supports one strongly consistent read/second up to 4 KB, or two eventual reads/second of that size. One WCU supports one standard write/second up to 1 KB. Round item sizes up to the applicable boundary; transactional operations require more capacity.

Example: 10 strongly consistent reads/second of 6 KB items require 20 RCUs for individual reads. Do not use these simple per-item calculations blindly for every batch/query API.

### Conditional writes, transactions and events

Use conditional writes to enforce a condition atomically, such as “create only if absent” or “update only if version matches.” Use transactions when multiple supported operations must succeed/fail together. A BatchWrite request is not an all-or-nothing transaction.

DynamoDB Streams retains item-change records for **24 hours**. Ordering is maintained for changes to each item; it is not one global ordering across the table. Lambda event source mappings can process records more than once, so downstream effects should be idempotent.

### DAX

DAX caches supported DynamoDB access patterns for very low-latency repeated **eventually consistent** reads. Strongly consistent and transactional reads pass through to DynamoDB without the same cache acceleration. Writes made directly to DynamoDB can leave cached values stale until expiration; plan client/write paths and acceptable freshness.

DAX does not fix hot writes, an unsuitable key design, or a requirement to cache strongly consistent reads with microsecond latency.

### Global tables

| Mode | Behavior | Trade-off |
|---|---|---|
| MREC — default | Asynchronous cross-Region replication; concurrent item conflicts use last-writer-wins | Lower-latency regional writes; replication lag means nonzero RPO risk |
| MRSC | Cross-Region synchronous durability and supported strong reads across replicas | Higher latency, quorum/topology/Region constraints; RPO zero support |

In MREC, a strong read in one Region can still miss a recent write made in another Region. MREC transactions are atomic in their originating Region, not replicated as a single atomic unit everywhere.

MRSC requires a supported three-Region topology, using three replicas or two replicas plus a witness. TTL and LSIs are not supported. Verify other feature constraints before choosing it; “global tables always eventual” is outdated.

### Recovery, expiration and access

PITR supports a configured recovery window up to 35 days. On-demand backups provide chosen recovery points. Restoration creates a **new table**, requiring validation and application cutover; recheck settings/integrations rather than assuming everything is recreated identically.

TTL uses an expiration timestamp in Unix epoch seconds and deletes expired items asynchronously, typically within days. Applications must enforce expiration when serving sessions/tokens; TTL is not an exact-time authorization mechanism.

Use IAM and applicable resource policies for access, encryption for data protection, and a DynamoDB gateway VPC endpoint when its private-access scope fits. An endpoint does not replace IAM authorization.

## Comparisons

| Need | Mechanism |
|---|---|
| New key-based query pattern | Suitable GSI / redesigned keys |
| Latest committed regional table value | Supported strongly consistent read |
| Multi-item atomic changes | Transactions |
| Automatic cleanup of expired items | TTL, with application-side expiry checks |
| React to item changes | Streams + consumer |
| Regional disaster resilience | Appropriate global-table mode plus application routing |
| Complex relational joins | [[aws_rds|RDS]] / [[aws_aurora|Aurora]] |

DynamoDB offers PartiQL, a SQL-compatible query language; it does not thereby become a relational engine with arbitrary joins.

## Exam Traps

- **GSI reads cannot be made strongly consistent by setting a flag.**
- **A filter does not undo read-capacity consumption.**
- **On-demand does not eliminate hot partitions.**
- **TTL does not delete exactly at the timestamp.**
- **DAX can pass strong reads through, but does not accelerate them from its eventual cache.**
- **A batch operation is not necessarily a transaction.**
- **Global replication is not backup history.** Bad writes can propagate.

## Scenario Check

**An application must immediately read a just-written item through a GSI.** The GSI cannot provide strong consistency. Use a suitable base-table/LSI read path or redesign the access pattern. Increasing capacity or adding DAX does not change GSI consistency.

## 30-Second Review

DynamoDB performance starts with keys and distributed traffic. Query targets keys; Scan and post-read filters can consume substantial capacity. GSIs allow new access paths but only eventual reads; LSIs are created with the table and support strong reads. DAX accelerates eventual caching. TTL is delayed cleanup. Global-table consistency depends on MREC versus MRSC. Backups restore a new table.

## Sources

- [Secondary indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)
- [Read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
- [Capacity modes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html)
- [DAX consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.consistency.html)
- [Global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/V2globaltables_HowItWorks.html)
- [TTL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [PITR](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Point-in-time-recovery.html)

Reviewed: 2026-09-22. Back to [[database_overview|Databases]].
