# 🗄️ Amazon RDS

> [!summary] Mental Model
> **Amazon RDS = Managed Relational Database**
>
> AWS manages much of the operational work:
> - Backups
> - Patching
> - Failure detection
> - Recovery
>
> ```text
> Application
>      ↓
> Amazon RDS
>      ↓
> Relational Database
> ```

---

# 📌 Core

Amazon Relational Database Service (**RDS**) is a managed service for relational databases.

Supported engines include:

- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Amazon Aurora

AWS manages tasks such as:

- Backups
- Software patching
- Failure detection
- Recovery

> [!tip] 🎯 Exam Clue
> **Managed relational database**
> → Amazon RDS

---

# 🧠 Relational Database

RDS is appropriate when applications require:

- SQL
- Tables
- Relationships
- Transactions
- Structured data

Example:

```text
Customers
    │
    └── Orders
          │
          └── Products
```

> [!tip] 🎯 Exam Clue
> **Relational + SQL + transactions**
> → RDS / Aurora

---

# 🏢 DB Instance

A **DB Instance** is the compute environment where the database engine runs.

You choose:

- DB engine
- Instance class
- Storage
- VPC
- Subnets
- Security Groups
- Availability configuration

Applications connect using a **database endpoint**.

```text
Application
     ↓
RDS Endpoint
     ↓
DB Instance
```

---

# 💾 Storage

Common RDS storage types include:

## General Purpose SSD

Good general-purpose choice.

```text
gp3
```

Suitable for most workloads.

---

## Provisioned IOPS SSD

Designed for workloads requiring:

- High I/O performance
- Predictable performance
- I/O-intensive databases

> [!tip] 🎯 Exam Clue
> **High-performance production database + predictable IOPS**
> → Provisioned IOPS

---

# 📈 Storage Auto Scaling

RDS can automatically increase database storage when available storage is running low.

You configure a **maximum storage threshold**.

```text
Database grows
     ↓
Storage running low
     ↓
RDS Storage Auto Scaling
     ↓
Increase storage
```

> [!tip] 🎯 Exam Clue
> **Database storage requirements are unpredictable**
> → RDS Storage Auto Scaling

---

# 🏥 Multi-AZ

Multi-AZ is primarily for:

# **HIGH AVAILABILITY**

RDS maintains a standby in another Availability Zone.

```text
             AZ-A
          Primary DB
              │
              │ Synchronous
              │ Replication
              ↓
             AZ-B
          Standby DB
```

Replication is **synchronous**.

If the primary fails:

```text
Primary ❌
   ↓
Automatic Failover
   ↓
Standby becomes Primary ✅
```

RDS automatically handles failover.

> [!tip] 🎯 Exam Clue
> **High Availability**
>
> **Automatic failover**
>
> **AZ failure protection**
>
> → Multi-AZ

---

## ⚠️ Multi-AZ Standby

For the classic Multi-AZ DB instance architecture, the standby exists for **failover**.

It is NOT used to scale application reads.

```text
Application
    ↓
 Primary
    ↓
 Standby

Standby ≠ Read Scaling
```

> [!danger] 🚨 VERY IMPORTANT
> **Multi-AZ = HIGH AVAILABILITY**
>
> NOT read scaling.

---

# 📖 Read Replicas

Read Replicas are primarily for:

# **READ SCALING**

```text
                 Primary
               ↙    ↓    ↘
              ↓     ↓     ↓
           Read    Read    Read
          Replica Replica Replica
```

Applications can send read-only queries to the replicas.

Replication from the source is generally **asynchronous**.

Useful for:

- Read-heavy workloads
- Reporting
- Analytics queries
- Reducing load on the primary

> [!tip] 🎯 Exam Clue
> **Read-heavy database**
>
> **Millions of SELECT queries**
>
> **Offload reporting**
>
> **Increase read throughput**
>
> → Read Replica

---

# ⚔️ Multi-AZ vs Read Replica

This is one of the most important RDS distinctions for SAA-C03.

| | Multi-AZ | Read Replica |
|---|---|---|
| Main purpose | **High Availability** | **Read Scaling** |
| Replication | Synchronous | Asynchronous |
| Failover | Automatic | Not the primary purpose |
| Read traffic | Standby generally not used | ✅ |
| AZ failure protection | ✅ | Not primary purpose |
| Reporting workload | ❌ | ✅ |

> [!warning] ⚠️ Exam Trap
> **HA / Failover**
> → Multi-AZ
>
> **Performance / Read Scaling**
> → Read Replica

---

## 🧠 Easy Mental Model

```text
Multi-AZ
→ "What happens if my database dies?"

Read Replica
→ "What happens if I have too many reads?"
```

---

# 🌎 Cross-Region Read Replica

Read Replicas can also exist in another Region.

```text
Region A
Primary RDS
     │
     │ Asynchronous
     ↓
Region B
Read Replica
```

Useful for:

- Reads closer to users in another Region
- Cross-Region replication scenarios
- Disaster recovery architectures

Cross-Region replication can incur data transfer costs.

---

# ⬆️ Promote Read Replica

A Read Replica can be **promoted** to become an independent database.

```text
Read Replica
     ↓
   Promote
     ↓
Standalone DB
```

After promotion, it no longer acts as a replica of the original database.

> [!tip] 🎯 Exam Clue
> **Create independent DB from Read Replica**
> → Promote Read Replica

---

# 💾 Backups

RDS supports:

1. **Automated Backups**
2. **Manual Snapshots**

---

# 🔄 Automated Backups

RDS automatically performs backups and maintains transaction logs.

Allows:

## Point-In-Time Recovery — PITR

```text
Database
   ↓
"Restore database to 10:37 AM yesterday"
   ↓
New DB Instance
```

> [!tip] 🎯 Exam Clue
> **Restore database to a specific point in time**
> → Automated Backups / PITR

---

# 📸 Manual Snapshots

Manual snapshots are initiated manually.

```text
RDS
 ↓
Snapshot
 ↓
Restore
 ↓
New DB Instance
```

Useful for:

- Long-term backups
- Before major changes
- Database copies

Snapshots can also be copied:

- Across Regions
- Across AWS accounts

---

# 🧠 Backup vs Snapshot

| | Automated Backup | Manual Snapshot |
|---|---|---|
| Created automatically | ✅ | ❌ |
| PITR | ✅ | ❌ |
| Manually created | ❌ | ✅ |
| Restore creates new DB | ✅ | ✅ |
| Useful long-term copy | Possible depending on retention | ✅ |

> [!warning] ⚠️ Exam Trap
> Restoring an RDS backup/snapshot creates a **new DB instance**.
>
> It does not overwrite the existing DB.

---

# 🔐 Encryption

RDS supports encryption at rest using **AWS KMS**.

Encryption can protect:

- Database storage
- Automated backups
- Snapshots
- Read Replicas

Encryption in transit can use **SSL/TLS**.

```text
Application
    │
    │ TLS
    ↓
Encrypted RDS
    │
    │ KMS
    ↓
Storage
```

---

# 🔑 IAM Database Authentication

For supported engines, applications can authenticate using **IAM database authentication** instead of relying only on traditional database passwords.

```text
IAM
 ↓
Temporary Authentication Token
 ↓
RDS
```

Useful for reducing long-lived database credentials.

> [!tip] 🎯 Exam Clue
> **Avoid storing DB passwords in application**
> → IAM DB Authentication
>
> or, depending on the scenario:
> → Secrets Manager

---

# 🔑 Secrets Manager + RDS

AWS Secrets Manager can securely store database credentials.

It can also support **automatic credential rotation**.

```text
Application
     ↓
Secrets Manager
     ↓
DB Credentials
     ↓
RDS
```

> [!tip] 🎯 Exam Clue
> **Store + automatically rotate database password**
> → AWS Secrets Manager

---

# 🔌 RDS Proxy

RDS Proxy sits between applications and the database.

```text
Applications
     ↓
  RDS Proxy
     ↓
Connection Pool
     ↓
     RDS
```

It pools and reuses database connections.

Useful for:

- Large numbers of connections
- Serverless applications
- Lambda
- Connection spikes
- Improving resilience during DB failovers

> [!tip] 🎯 Exam Clue
> **Lambda opens too many DB connections**
>
> **Connection exhaustion**
>
> **Frequent opening/closing DB connections**
>
> → RDS Proxy

---

# 🧠 Lambda + RDS

A common architecture:

```text
API Gateway
     ↓
Lambda
     ↓
RDS Proxy
     ↓
RDS
```

Why?

Lambda can scale very quickly:

```text
10 Lambdas
   ↓
100 Lambdas
   ↓
1,000 Lambdas
   ↓
Thousands of DB connections 💥
```

RDS Proxy pools those connections.

---

# 🌐 Networking

RDS is deployed inside a VPC.

Typical production architecture:

```text
              VPC

Public Subnets
┌─────────────────────┐
│         ALB         │
└─────────────────────┘
          ↓

Private App Subnets
┌─────────────────────┐
│      EC2 / ECS      │
└─────────────────────┘
          ↓

Private DB Subnets
┌─────────────────────┐
│         RDS         │
└─────────────────────┘
```

Best practice:

> Database should normally be in **private subnets**.

---

# 🗂️ DB Subnet Group

A DB Subnet Group defines the subnets that RDS can use inside a VPC.

For high availability, it should cover multiple Availability Zones.

```text
DB Subnet Group
│
├── Private Subnet AZ-A
└── Private Subnet AZ-B
```

---

# 🛡️ Security Groups

RDS uses Security Groups to control network access.

Example:

```text
Application SG
      ↓
TCP 5432
      ↓
RDS SG
```

Instead of:

```text
0.0.0.0/0 → PostgreSQL ❌
```

prefer:

```text
Application-SG → RDS-SG ✅
```

> [!tip] 🎯 Exam Clue
> **Allow application servers to access RDS**
> → Reference application Security Group in RDS Security Group

---

# 📊 Monitoring

RDS integrates with:

## CloudWatch

Monitor metrics such as:

- CPU
- Connections
- Storage
- IOPS
- Latency

---

## Enhanced Monitoring

Provides deeper operating-system-level metrics for the DB instance.

---

## Database Insights

Provides deeper database performance monitoring and troubleshooting capabilities.

---

## CloudTrail

Tracks RDS API activity.

```text
CloudWatch
→ Metrics

Enhanced Monitoring
→ OS metrics

Database Insights
→ Database performance analysis

CloudTrail
→ API activity
```

---

# 🔄 Blue/Green Deployments

RDS Blue/Green Deployments allow you to create a staging environment that mirrors production.

```text
BLUE
Production
   │
   ↓
GREEN
Staging
```

You can test changes in Green before switching production over.

Useful for:

- Database upgrades
- Configuration changes
- Safer production changes
- Reduced downtime

> [!tip] 🎯 Exam Clue
> **Test database changes before production + minimize downtime**
> → RDS Blue/Green Deployment

---

# 📈 Vertical vs Horizontal Scaling

## Vertical Scaling

Increase the DB instance class:

```text
db.t3
  ↓
db.m6
  ↓
db.r6
```

More:

- CPU
- RAM

---

## Read Scaling

Add:

```text
Read Replicas
```

Mental model:

```text
Need more CPU/RAM
→ Bigger DB Instance

Too many reads
→ Read Replicas

Need HA
→ Multi-AZ
```

---

# ⚔️ RDS vs DynamoDB

## RDS

→ Relational  
→ SQL  
→ Joins  
→ Transactions  
→ Structured schema

## DynamoDB

→ NoSQL  
→ Key-value/document  
→ Massive horizontal scale  
→ Very low latency

> [!tip] 🎯 Exam Clue
> **Relational + complex SQL**
> → RDS
>
> **Massive scale + key-value access**
> → DynamoDB

---

# ⚔️ RDS vs Redshift

## RDS

Designed primarily for:

**OLTP**

```text
INSERT
UPDATE
DELETE
SELECT individual records
```

Examples:

- Orders
- Users
- Payments
- Application databases

## Redshift

Designed primarily for:

**OLAP / Analytics**

```text
SELECT SUM(...)
GROUP BY
Large analytical queries
```

Examples:

- Data warehouse
- BI
- Reporting
- Analytics

> [!tip] 🧠 Mental Model
> **RDS → Run the application**
>
> **Redshift → Analyze the business**

---

# ⚔️ RDS vs ElastiCache

ElastiCache can sit in front of RDS to reduce repeated database reads.

```text
Application
     ↓
ElastiCache
     ↓
RDS
```

If frequently requested data is cached:

```text
Cache HIT
→ Return from cache

Cache MISS
→ Query RDS
```

> [!tip] 🎯 Exam Clue
> **Same database queries repeatedly + need very low latency**
> → ElastiCache

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| Managed relational database | **RDS** |
| High Availability | **Multi-AZ** |
| Automatic DB failover | **Multi-AZ** |
| Read-heavy workload | **Read Replica** |
| Reporting overloads primary | **Read Replica** |
| Cross-Region reads | **Cross-Region Read Replica** |
| Restore specific time | **PITR** |
| Long-term manual backup | **Snapshot** |
| Automatically growing storage | **Storage Auto Scaling** |
| Too many DB connections | **RDS Proxy** |
| Lambda → RDS connection spikes | **RDS Proxy** |
| Rotate DB credentials | **Secrets Manager** |
| DB encryption at rest | **KMS** |
| Safe DB upgrades | **Blue/Green Deployment** |
| Relational SQL workload | **RDS** |
| NoSQL massive scale | **DynamoDB** |
| Data warehouse / OLAP | **Redshift** |
| Repeated reads / caching | **ElastiCache** |

---

# 🚨 MOST IMPORTANT EXAM TRAP

```text
                 RDS
                  │
       ┌──────────┴──────────┐
       ↓                     ↓
    Multi-AZ            Read Replica
       ↓                     ↓
HIGH AVAILABILITY        READ SCALING
       ↓                     ↓
Synchronous            Asynchronous
       ↓                     ↓
Automatic Failover     Read Queries
```

> [!danger] Do NOT confuse them
> **Multi-AZ ≠ Read Scaling**
>
> **Read Replica ≠ Automatic HA replacement**

---

> [!abstract] 🧠 Amazon RDS in 30 Seconds
> **Purpose:** Managed relational database
>
> **HA:** Multi-AZ
>
> **Read Scaling:** Read Replica
>
> **Multi-AZ replication:** Synchronous
>
> **Read Replica replication:** Asynchronous
>
> **Specific recovery time:** PITR
>
> **Manual backup:** Snapshot
>
> **Growing DB:** Storage Auto Scaling
>
> **Connection pooling:** RDS Proxy
>
> **Credentials:** Secrets Manager / IAM DB Auth
>
> **Encryption:** KMS
>
> **Safer upgrades:** Blue/Green
>
> **OLTP:** RDS
>
> **OLAP:** Redshift
>
> **NoSQL:** DynamoDB