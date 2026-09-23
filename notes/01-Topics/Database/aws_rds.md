---
aliases: [Amazon RDS, Relational Database Service]
tags: [aws/saa, database]
---

# Amazon RDS

## Mental Model

**AWS manages database infrastructure operations; you still design schema, queries, permissions, capacity and recovery.**

Choose the engine and deployment type before applying an exam mnemonic.

## Core

RDS supports relational engines including MySQL, PostgreSQL, MariaDB, Oracle, SQL Server and Db2; Aurora is also part of RDS with a distinct architecture. Compatibility, licensing and features differ by engine/version/Region.

### Availability and read scaling

| Deployment | Replication / topology | Read and failover behavior |
|---|---|---|
| Single-AZ DB instance | One primary instance in one AZ | No Multi-AZ standby to take over |
| Multi-AZ DB instance | Primary plus synchronous standby in another AZ | Automatic failover; standby does not serve reads |
| Multi-AZ DB cluster | One writer plus two readable instances across three AZs; semisynchronous | HA plus read capacity; supported MySQL/PostgreSQL configurations |
| Read replica | Generally asynchronous from its source | Offload reads; lag and promotion behavior depend on engine |

**“Multi-AZ does not serve reads” applies to the classic DB instance standby, not every Multi-AZ deployment.**

Applications must route eligible reads to the relevant read endpoint. Replication does not automatically move application SQL off the writer. After failover, existing connections can break; use endpoint DNS appropriately and implement retries/reconnection.

Cross-Region replicas can support regional reads or DR for supported engines. Promotion creates an independent writable database; it is not the same operation as automatic standby failover. Include application redirection and replication lag in recovery planning.

### Solve the actual bottleneck

| Bottleneck | Evaluate |
|---|---|
| CPU/memory | Query/index tuning and instance sizing |
| Storage I/O | Suitable gp3 or provisioned-IOPS storage and supported limits |
| Storage capacity | Storage autoscaling with a maximum threshold |
| Read demand | Read replicas or [[aws_elasticache|cache]] according to freshness/access patterns |
| Connection churn | RDS Proxy and application pooling |

Storage autoscaling increases capacity; it does not shrink storage or automatically add CPU, solve inefficient SQL, or scale writes horizontally. General EBS specifications are not universal RDS storage limits.

### Backups and restore

Automated backups plus transaction logs support PITR within the enabled retention window, typically up to 35 days for supported RDS configurations. A manual snapshot is a chosen recovery point retained until deleted.

Restoring creates a **new DB instance or cluster**, according to deployment type. Validate data, networking, parameters and application endpoints before cutover. A successful restore job is not the entire recovery process.

Snapshots can be copied/shared where supported. Cross-account encrypted snapshot workflows require appropriate customer-managed KMS keys and permissions; an AWS-managed key is not a general cross-account sharing solution. Back up before destructive changes and test recovery.

### Security and connection management

- Use private network placement where appropriate, a DB subnet group spanning required AZs, and database-port access from application security groups.
- KMS encryption at rest and TLS in transit solve different problems. Protect key access during recovery.
- **IAM database authentication** uses temporary authentication tokens for supported engines/configurations. Database users and SQL privileges still matter.
- **Secrets Manager** stores and rotates supported database credentials; it is not the same mechanism as IAM token authentication.
- **RDS Proxy** pools/reuses connections and helps applications handle connection spikes/failover. It does not cache query results or create unlimited database capacity. Engine support and session pinning affect its benefits.

### Operations and control

| Tool / feature | Purpose |
|---|---|
| CloudWatch metrics | CPU, connections, storage and latency indicators |
| Enhanced Monitoring | Guest OS/process-level metrics |
| CloudWatch Database Insights | Database load, waits and SQL performance analysis according to mode/support |
| CloudTrail | Control-plane API audit; not a substitute for database SQL audit logs |
| Blue/Green Deployments | Test supported engine/configuration changes before controlled switchover |
| RDS Custom | Supported workloads requiring more OS/database customization within a managed-service support boundary |

Ordinary RDS does not provide normal SSH/administrator access to the underlying host. For unsupported customization, evaluate a database on EC2 with its additional operational burden.

## Comparisons

| Requirement | Choose |
|---|---|
| Engine-specific relational compatibility | RDS engine matching the workload |
| Aurora's MySQL/PostgreSQL-compatible architecture | [[aws_aurora|Aurora]] |
| Key-value access without relational joins | [[aws_dynamodb|DynamoDB]] |
| Large warehouse analytics | [[aws_redshift|Redshift]] |
| Undo logical corruption | PITR / snapshot restore, not standby failover |

## Exam Traps

- **Read replicas do not add writer capacity.**
- **Multi-AZ DB cluster readers are readable; a Multi-AZ DB instance standby is not.**
- **Synchronous HA does not undo a legitimate but mistaken DELETE.**
- **RDS Proxy is a connection pool, not a cache.**
- **A backup restore creates a new resource.** Plan application cutover.
- **IAM authorization to manage RDS resources is not automatically permission to execute SQL.**
- **Blue/green reduces risk; it does not guarantee zero downtime or support every engine/change.**

## Scenario Check

**A company needs AZ failover and wants its standby capacity to serve reporting reads.** A compatible Multi-AZ DB cluster can satisfy both. The standby of a classic Multi-AZ DB instance cannot; another option may combine a Multi-AZ DB instance with separate read replicas, depending on engine and cost requirements.

## 30-Second Review

RDS manages relational infrastructure. Classic Multi-AZ uses a nonreadable synchronous standby; Multi-AZ DB clusters have two readable instances. Replicas scale reads, storage autoscaling grows disk, and Proxy pools connections. None automatically scales writes. PITR restores a new resource. Use private access, appropriate database authentication and encryption. Monitor SQL load separately from host metrics and API audit.

## Sources

- [Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html)
- [Read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)
- [PITR](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
- [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)
- [IAM database authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)
- [Database Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html)

Reviewed: 2026-09-22. Back to [[database_overview|Databases]].
