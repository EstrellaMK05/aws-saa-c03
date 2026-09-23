---
aliases: [Amazon Aurora, Aurora]
tags: [aws/saa, database]
---

# Amazon Aurora

## Mental Model

**Relational compute sits above shared distributed storage. Scale the writer, readers and storage according to different needs.**

Aurora is MySQL/PostgreSQL-compatible, not automatically identical to every engine feature or extension.

## Core

### Cluster architecture

A conventional Aurora cluster has one writer and up to 15 Aurora Replicas. Its storage maintains six copies across three AZs and grows automatically. Readers share the distributed cluster storage and can serve as failover targets.

Multi-AZ storage does not remove the value of an existing replica in another AZ: redundant compute can reduce recovery work when the writer fails. Configure failover priorities and application reconnection. Adding replicas scales reads, not independent writers.

### Endpoints

| Endpoint | Destination | Important detail |
|---|---|---|
| Cluster / writer | Current writer | Reads and writes; stable name across writer failover |
| Reader | Available Aurora Replicas | Balances **connections**, not individual queries |
| Instance | One specific instance | Useful for diagnostics; does not follow the writer role automatically |
| Custom | Selected group of instances | Separate workloads onto appropriate replica groups |

If there are **no Aurora Replicas**, the reader endpoint connects to the writer, which can accept writes. Therefore, a reader endpoint is not an authorization boundary that always enforces read-only access. Use database privileges for that requirement.

Connection pools may keep many queries on the same selected reader. The writer endpoint does not inspect SELECT statements and automatically distribute them to replicas.

### Three kinds of scaling

| Mechanism | Changes |
|---|---|
| Storage growth | Cluster storage capacity |
| Aurora Auto Scaling | Number of supported reader replicas |
| Aurora Serverless v2 | Compute capacity of configured serverless instances, measured in ACUs |

Serverless v2 scales within configured bounds; it still needs sensible HA placement, capacity limits and workload testing. Supported versions/configurations can pause at zero ACUs when configured accordingly. Resume latency and pause-blocking features matter: do not memorize either “v2 never pauses” or “every serverless cluster always scales to zero.”

For predictable sustained demand, compare provisioned and serverless costs rather than assuming a winner from the label alone. **Aurora Standard** charges for I/O; **I/O-Optimized** changes the cost model and can fit I/O-heavy workloads. Compare total compute, storage and I/O cost.

### Global Database

Global Database replicates to secondary clusters in other Regions for regional reads and disaster recovery. Cross-Region replication is asynchronous. Planned switchovers and unplanned failovers have different data-loss conditions; do not promise zero RPO for any regional outage.

Global write forwarding, where supported, forwards secondary-Region writes to the primary writer. It does not turn the deployment into independent active-active writers in every Region. Recover the application, credentials and traffic routing as well as the database.

### Recovery and copies

| Feature | Purpose | Boundary |
|---|---|---|
| PITR / snapshots | Historical recovery | Restore a new cluster and validate/cut over |
| Backtrack | Rewind an existing supported Aurora MySQL cluster | Not PostgreSQL; must be enabled/configured and meet version/feature restrictions |
| Clone | Fast development/test copy using copy-on-write | Changes consume additional storage; not a replacement for independent backups |
| Replica failover | Recover from a failed writer | Does not undo propagated logical mistakes |

Backtrack affects the cluster's data state, not just one mistaken row. Plan the impact on valid later writes; keep backups even when Backtrack is enabled.

Use KMS encryption, TLS, appropriate IAM database authentication or Secrets Manager, and RDS Proxy where connection pooling fits. See [[aws_rds]] for these shared concepts.

## Comparisons

| Requirement | Starting point |
|---|---|
| Read scale within a Region | Aurora Replicas and reader/custom endpoints |
| Variable relational compute | Aurora Serverless v2 |
| Relational reads near global users and cross-Region DR | Aurora Global Database |
| Oracle or SQL Server compatibility | Appropriate RDS engine, not Aurora |
| Native multi-Region key-value writes | [[aws_dynamodb|DynamoDB global tables]], if the data model fits |
| Large BI/warehouse workloads | [[aws_redshift|Redshift]] |

## Exam Traps

- **Reader endpoint balances connections, not each SQL query.**
- **With no replicas, the reader endpoint can reach the writer.**
- **Backtrack is an Aurora MySQL capability with restrictions.** Do not apply it to Aurora PostgreSQL.
- **Six storage copies are not six query-serving instances.**
- **Serverless v2 compute scaling differs from adding replicas.**
- **Global write forwarding is not independent local writing in every Region.**
- **A database failover time is not the application's complete RTO.**

## Scenario Check

**A production workload and a heavy reporting workload must use different reader instance groups.** Create appropriate custom endpoints and route each application accordingly. The generic reader endpoint does not partition those workloads by intent. An instance endpoint can pin to one reader, but does not provide a selected group of readers.

## 30-Second Review

Aurora separates compute from shared Multi-AZ storage. One writer handles writes; replicas scale reads and support failover. Writer, reader, instance and custom endpoints have different routing roles; reader routing is per connection. Serverless v2 scales compute, while Auto Scaling adjusts readers. Global Database adds cross-Region replication. Backtrack is restricted to supported MySQL clusters; backups remain necessary.

## Sources

- [Aurora high availability](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)
- [Reader endpoint behavior](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html)
- [Serverless v2 pause/resume](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html)
- [Global Database recovery](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-disaster-recovery.html)
- [Backtrack](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html)
- [Storage configurations](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html)

Reviewed: 2026-09-22. Back to [[database_overview|Databases]].
