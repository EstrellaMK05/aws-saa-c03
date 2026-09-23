---
aliases: [AWS DMS, Database Migration, DMS Schema Conversion]
tags: [aws/saa, database]
---

# AWS DMS — Database Migration

## Mental Model

**Prepare a compatible target schema, copy existing data, then capture changes until a controlled cutover.**

Moving data and converting the application/database design are separate jobs.

## Core

### Migration building blocks

| Component / operation | Purpose |
|---|---|
| Source and target endpoints | Identify databases/data stores and connection details |
| Replication compute | Runs supported migration/replication work; provisioned and serverless options have different support |
| Full load | Copy existing selected data |
| Change data capture (CDC) | Apply ongoing supported source changes |
| Mapping/transformation rules | Select and transform supported objects/data |
| Validation and monitoring | Check migrated data, failures, latency and progress |

Supported sources and targets vary by engine, version, migration type and DMS mode. Do not assume all listed endpoints support every direction or feature.

### Homogeneous vs heterogeneous

- **Homogeneous:** same or compatible engine. Schema and feature checks still matter, even when no large conversion is needed.
- **Heterogeneous:** different engines, such as Oracle to Aurora PostgreSQL. Assess and convert schema/code, resolve unsupported objects, then migrate data.
- **DMS Schema Conversion** performs supported assessment/schema conversion. **AWS SCT** is another tool found in many migration scenarios. Conversion can produce action items requiring manual work.

The data-replication task alone does not faithfully recreate every index, stored procedure, trigger, user, job and application query. Even when basic target tables can be created, do not treat that as a complete schema migration.

### Minimal-downtime migration

1. Assess engine/data-type compatibility and prepare target schema/security.
2. Enable required source logging and retain logs long enough for CDC.
3. Configure network paths, credentials and supported replication settings.
4. Run full load plus CDC while the source continues accepting writes.
5. Validate data and application behavior; monitor replication lag.
6. Quiesce source writes, let changes catch up, validate and redirect the application.
7. Keep a deliberate rollback/failback plan; later target writes do not automatically return to the old source.

“Minimal downtime” is not “no cutover work.” DMS replicates data; the team still coordinates writes, endpoints and application readiness.

### Operational considerations

Replication failures can come from missing logs, inadequate permissions, incompatible data types, insufficient network capacity or under-sized replication/target resources. Large objects have mode-specific handling and can materially affect performance/completeness. Measure and validate rather than assuming a successful task status proves every application object migrated correctly.

Multi-AZ for supported DMS replication compute protects the migration service's availability. It does not automatically make the source or target database Multi-AZ.

## Comparisons

| Requirement | Mechanism |
|---|---|
| Move database data while retaining source writes | Full load + CDC |
| Convert supported schema between engines | DMS Schema Conversion / SCT plus manual remediation |
| Copy a compatible database backup | Native backup/restore or snapshot workflow may be simpler |
| Copy ordinary NFS files or S3 objects | DataSync, not database CDC |
| Long-term HA for a production database | Suitable native HA/replication architecture; a migration task is not the complete solution |

## Exam Traps

- **Schema conversion does not move all data; data replication does not convert all application logic.**
- **CDC needs source prerequisites and logs.** It cannot recover changes that the source no longer retains.
- **Full load alone misses later source changes.**
- **CDC is not necessarily synchronous or zero-RPO replication.**
- **Target writes after cutover complicate rollback.** Plan reconciliation or supported reverse replication.
- **DMS Multi-AZ protects DMS compute, not every database in the migration.**

## Scenario Check

**An Oracle application must move to Aurora PostgreSQL with little downtime while orders continue arriving.** Assess/convert schema and incompatible application code, then use supported full load plus CDC. Validate and perform a controlled cutover. DMS alone does not automatically rewrite Oracle procedures and every SQL query.

## 30-Second Review

DMS copies data and captures supported ongoing changes. Schema Conversion or SCT prepares a compatible schema, with manual remediation where necessary. Full load plus CDC reduces downtime, but still needs logs, permissions, connectivity, validation and a cutover. Monitor lag and large-object behavior. Migration-compute HA does not make endpoints highly available, and rollback after new target writes requires planning.

## Sources

- [DMS overview](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [CDC](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Task.CDC.html)
- [DMS Schema Conversion](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_SchemaConversion.html)
- [Schema conversion with SCT](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.SCT.html)
- [DMS validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html)

Reviewed: 2026-09-22. Back to [[database_overview|Databases]].
