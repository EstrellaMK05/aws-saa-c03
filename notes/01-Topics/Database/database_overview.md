---
aliases: [Database Decision Map, Databases Overview]
tags: [aws/saa, database]
---

# Databases — Choose by Access Pattern

## Mental Model

**Choose the data model and consistency requirements first. Then solve availability, scaling, recovery and cost separately.**

A cache, a replica, a connection pool and a backup fix different problems.

## Core

| Requirement | Start with | Confirm |
|---|---|---|
| Relational schema, joins and SQL transactions | [[aws_rds|RDS]] / [[aws_aurora|Aurora]] | Engine compatibility and deployment topology |
| MySQL/PostgreSQL compatibility with Aurora's distributed storage | [[aws_aurora|Aurora]] | Feature/version support and total cost |
| Predictable key-based access at large scale | [[aws_dynamodb|DynamoDB]] | Keys, indexes, item size and consistency |
| Repeated application reads with submillisecond cache access | [[aws_elasticache|ElastiCache]] | Staleness, invalidation and loss tolerance |
| DynamoDB-specific acceleration of eventual reads | [[aws_dynamodb|DAX]] | Client integration and cache consistency |
| Large analytical aggregations and warehouse workloads | [[aws_redshift|Redshift]] | Data layout, compute mode and S3 integration |
| Migrate existing database data with limited downtime | [[aws_dms|DMS]] | Schema compatibility, CDC support and cutover |

### Recognize specialized alternatives

- **DocumentDB:** document workloads requiring supported MongoDB-compatible APIs. Compatibility is not identical support for every MongoDB feature; assess the actual application.
- **Neptune:** graph relationships and traversals, such as fraud connections or social graphs. “Data has relationships” alone is not enough; SQL can model relationships too.
- **MemoryDB:** durable in-memory database workloads with a Multi-AZ transaction log. Compare required API, durability and latency with supported ElastiCache durability configurations instead of assuming every in-memory service is disposable.

These are selection clues, not migration guarantees. Keep detailed implementation study focused on the main services above.

## Comparisons

| Symptom / requirement | Appropriate mechanism |
|---|---|
| Instance or AZ outage | Suitable Multi-AZ deployment with failover |
| Too many SQL reads | Read replicas or caching, depending on queries and freshness |
| Too many connections | RDS Proxy / connection pooling |
| Writer CPU or I/O saturation | Query/index optimization and suitable compute/storage; readers do not scale writes |
| Incorrect DELETE or UPDATE | PITR or another known-good recovery point |
| Regional outage | Cross-Region design with measured RPO/RTO |
| Variable database compute | Supported serverless capacity, within limits and latency requirements |
| Growing data volume | Storage scaling; separate from compute and query scaling |

## Exam Traps

- **NoSQL does not mean no transactions.** DynamoDB supports ACID transactions in supported configurations.
- **Multi-AZ is not one topology.** RDS DB instance and DB cluster deployments have different read capabilities.
- **A read replica is not a backup.** Valid but unwanted writes can propagate.
- **Connection pooling is not query-result caching.**
- **Serverless is not limitless or automatically cheapest.** Evaluate capacity limits, idle behavior and steady demand.
- **HA is not zero data loss.** Read the exact replication/durability mode.

## Scenario Check

**Lambda opens thousands of short-lived connections to a relational database, while query CPU is otherwise acceptable.** RDS Proxy addresses connection management. A read replica helps only if reads can be redirected; a cache helps repeated results; neither automatically solves the stated connection churn.

## 30-Second Review

RDS/Aurora fit relational workloads; DynamoDB fits designed key-based access; ElastiCache accelerates reusable data; Redshift analyzes large datasets. Choose availability, read scaling, connection management and backup independently. A replica cannot undo a bad write. Serverless still has limits. Match consistency and recovery requirements to the exact deployment, then optimize cost among designs that meet them.

## Study Order

1. [[aws_rds]] → [[aws_aurora]]
2. [[aws_dynamodb]] → [[aws_elasticache]]
3. [[aws_redshift]] → [[aws_dms]]

Related: [[architecture_overview]], [[backup_and_restore]], [[disaster_recovery]].

## Sources

- [SAA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html)
- [DocumentDB compatibility](https://docs.aws.amazon.com/documentdb/latest/devguide/compatibility.html)
- [DocumentDB and Neptune comparison](https://aws.amazon.com/compare/documentdb-and-neptune/)
- [MemoryDB failure recovery](https://docs.aws.amazon.com/memorydb/latest/devguide/autofailover.html)

Reviewed: 2026-09-22. Use this index with the detailed notes; older summaries elsewhere in the vault may still contain simplified rules.
