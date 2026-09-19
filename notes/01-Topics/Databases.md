## Quick Decision

- Traditional relational DB → RDS
- AWS optimized relational DB → Aurora
- Key-value / NoSQL → DynamoDB
- Cache → ElastiCache
- Data Warehouse / Analytics → Redshift

---

# RDS

Managed relational database.

Supports engines such as:
- MySQL
- PostgreSQL
- MariaDB
- Oracle
- SQL Server

💡 Traditional relational / OLTP → RDS


## Multi-AZ

Used for **High Availability**.

- Synchronous replication
- Standby in another AZ
- Automatic failover

💡 HA / AZ failure → Multi-AZ

⚠️ Multi-AZ is NOT mainly for read scaling.


## Read Replica

Used to scale **READS**.

- Asynchronous replication
- Applications can send read queries to replicas

💡 High read traffic → Read Replica

⚠️ Read Replica ≠ Multi-AZ


## RDS Proxy

Manages and pools database connections.

Useful for:
- Lambda
- Many concurrent connections
- Preventing DB connection exhaustion

💡 Lambda + too many DB connections → RDS Proxy


## Backups / PITR

Point-in-Time Recovery allows restoring the database to a specific point in time.

💡 Accidental deletion at 3:14 PM → PITR

---

# Aurora

AWS relational database compatible with:

- MySQL
- PostgreSQL

Designed for higher availability, scalability and performance than traditional RDS engines.

💡 MySQL/PostgreSQL compatible + AWS optimized → Aurora


## Aurora Replicas

Used for:
- Read scaling
- High availability

Aurora supports up to 15 read replicas.


## Aurora Endpoints

### Writer Endpoint
Used for writes.

### Reader Endpoint
Distributes read connections across Aurora Replicas.

💡 Multiple replicas + distribute SELECT queries → Reader Endpoint


## Aurora Serverless

Automatically adjusts database capacity.

Good for:
- Unpredictable workloads
- Variable traffic
- Infrequent workloads

💡 Relational + unpredictable demand → Aurora Serverless


## Aurora Global Database

Used for multi-region architectures.

- Low-latency global reads
- Cross-region disaster recovery

💡 Aurora + global reads / multi-region DR → Aurora Global Database


## Aurora Backtrack

Move an Aurora database back to an earlier point without performing a traditional restore.

💡 "Go back 10 minutes" → Backtrack

---

# DynamoDB

Serverless NoSQL key-value/document database.

- Very low latency
- Automatically scales
- No servers to manage

💡 Key-value + massive scale + low latency → DynamoDB


## Capacity Modes

### On-Demand
Good for unpredictable traffic.

💡 Unknown / unpredictable requests → On-Demand

### Provisioned
Good when capacity requirements are predictable.


## DynamoDB Accelerator — DAX

In-memory cache for DynamoDB.

Provides microsecond read performance.

💡 DynamoDB + repeated reads + microseconds → DAX


## Global Tables

Multi-region, active-active DynamoDB.

- Local reads
- Local writes
- Automatic multi-region replication

💡 Multi-region READ + WRITE → Global Tables


## TTL

Automatically expires items.

Uses a Unix epoch timestamp attribute.

💡 Expire sessions/items automatically → TTL


## DynamoDB Streams

Captures item-level changes.

Can trigger Lambda when an item is:
- Inserted
- Updated
- Deleted

💡 DynamoDB change → trigger Lambda → DynamoDB Streams


## Consistency

### Eventually Consistent
May not immediately return the latest data.

### Strongly Consistent
Returns the latest successful write.

💡 MUST immediately read latest value → Strongly Consistent Read


## GSI vs LSI

### GSI — Global Secondary Index

- Can use a DIFFERENT partition key
- Can be created after the table already exists

💡 Need new query pattern later → GSI


### LSI — Local Secondary Index

- SAME partition key as base table
- Different sort key
- Must be created WITH the table

💡 Same partition key + different sort key + known at table creation → LSI


⚠️ Exam Trap

GSI → can be added later  
LSI → must exist when table is created

---

# ElastiCache

In-memory caching service.

Used to:
- Reduce database load
- Improve response time
- Cache frequently accessed data

💡 Repeated relational DB reads → ElastiCache


## Redis / Valkey

Supports:
- Replication
- Multi-AZ
- Automatic failover
- More advanced data structures

💡 Cache + High Avai. + failover → Redis / Valkey


## Memcached

Simpler distributed cache.

- No replication
- No Multi-AZ automatic failover

💡 Simple cache → Memcached

⚠️ Need HA/failover → Redis/Valkey, NOT Memcached


# Redshift

Data warehouse for analytics / OLAP.

Good for:
- Large datasets
- Complex analytical queries
- Aggregations
- BI/reporting

💡 Analytics over billions of rows → Redshift

⚠️ Redshift = OLAP
⚠️ RDS/Aurora = OLTP


## Redshift Serverless

Automatically manages compute capacity.

Good for:
- Intermittent analytics
- Unpredictable analytical workloads

💡 Analytics + unpredictable/intermittent usage → Redshift Serverless


## Redshift Spectrum

Queries data directly in S3 without loading it into Redshift tables.

💡 Redshift + query S3 directly → Spectrum

Can join:

`Redshift Tables + S3 Data`


# Database Migration

## DMS — Database Migration Service

Moves data between databases.

For minimal downtime:

**Full Load + CDC**

CDC = Change Data Capture

💡 Database keeps receiving writes during migration → DMS + CDC


## SCT — Schema Conversion Tool

Used when migrating between different database engines.

Example:

`Oracle → Aurora PostgreSQL`

Use:

**SCT + DMS**

- SCT → converts schema
- DMS → moves data

💡 Heterogeneous database migration → SCT + DMS

## RDS Custom

RDS → Managed relational DB, no OS-level customization.

RDS Custom → Need OS / database environment customization
             BUT still want AWS to manage much of the infrastructure.

EC2 + Database → Maximum control, maximum management overhead.

Exam clue:
"Customize underlying OS/database + minimize operational overhead"
→ RDS Custom

# 🔥 Exam Decision Map

HA problem?
→ RDS Multi-AZ

Too many READS?
→ Read Replica

Too many CONNECTIONS?
→ RDS Proxy

Repeated data?
→ ElastiCache

Relational + unpredictable compute?
→ Aurora Serverless

NoSQL?
→ DynamoDB

DynamoDB cache?
→ DAX

DynamoDB multi-region active-active?
→ Global Tables

Analytics / Data Warehouse?
→ Redshift

Query S3 from Redshift?
→ Spectrum

Same DB engine migration?
→ DMS

Different DB engines?
→ SCT + DMS