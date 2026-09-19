# 🌌 Amazon Aurora

> [!summary] Mental Model
> **Amazon Aurora = AWS-optimized relational database compatible with MySQL/PostgreSQL**
>
> Think:
>
> **Relational + High Performance + High Availability + Read Scaling**
>
> ```text
>                Aurora Cluster
>                     │
>          ┌──────────┴──────────┐
>          ↓                     ↓
>      Writer DB           Aurora Replicas
>          │                     │
>          └──────────┬──────────┘
>                     ↓
>             Shared Cluster Storage
> ```

---

# 📌 Core

Amazon Aurora is a relational database engine built for AWS.

Compatible with:

- **MySQL**
- **PostgreSQL**

Aurora is part of **Amazon RDS**, but uses a different distributed storage architecture.

Key characteristics:

- Managed relational database
- MySQL/PostgreSQL compatible
- High availability
- Automatic storage scaling
- Read scaling with Aurora Replicas
- Automatic failover
- Multi-AZ distributed storage

> [!tip] 🎯 Exam Clue
> **High-performance managed relational DB + MySQL/PostgreSQL compatibility**
> → Amazon Aurora

---

# 🏗️ Aurora Cluster Architecture

An Aurora database normally consists of:

```text
Aurora DB Cluster
│
├── Writer / Primary
│
├── Aurora Replica
├── Aurora Replica
├── Aurora Replica
│
└── Shared Cluster Storage
```

The instances share the same distributed storage layer.

This differs from traditional RDS replication where each DB instance maintains its own storage.

---

# 💾 Aurora Storage

Aurora automatically maintains:

**6 copies of the data across 3 Availability Zones**

```text
              Aurora Storage

        AZ-A       AZ-B       AZ-C
         │          │          │
       Copy       Copy       Copy
       Copy       Copy       Copy

              6 copies total
```

Storage is:

- Distributed
- Multi-AZ
- Self-healing
- Automatically scalable

Aurora continuously detects storage failures and repairs them using healthy copies.

> [!tip] 🎯 Exam Clue
> **6 copies across 3 AZs**
> → Aurora

---

# ✍️ Writer Instance

The **Writer / Primary Instance** handles:

- `INSERT`
- `UPDATE`
- `DELETE`
- DDL
- Read operations if needed

```text
Application
     ↓
Writer Endpoint
     ↓
Primary Instance
     ↓
Shared Storage
```

Normally there is **one writer** in an Aurora cluster.

---

# 📖 Aurora Replicas

Aurora Replicas are used primarily for:

- Read scaling
- High availability
- Failover targets

Aurora supports up to:

# **15 Aurora Replicas**

```text
                 Writer
                   │
          Shared Storage
          ↙       ↓       ↘
      Replica   Replica   Replica
```

Because Aurora Replicas share the cluster storage, replication lag is typically very low.

> [!tip] 🎯 Exam Clue
> **Aurora + increase read capacity**
> → Add Aurora Replicas

---

# 🏥 Failover

If the writer fails:

```text
Writer ❌
   ↓
Aurora detects failure
   ↓
Promote Aurora Replica
   ↓
New Writer ✅
```

Aurora can automatically promote an existing replica.

You can configure **failover priority** for Aurora Replicas.

> [!tip] 🎯 Exam Clue
> **Aurora HA + faster failover**
> → Aurora Replica in another AZ

---

# 🔗 Aurora Endpoints

This is VERY important for SAA.

Aurora abstracts database connections using different **endpoints**.

Main types:

1. Cluster / Writer Endpoint
2. Reader Endpoint
3. Instance Endpoint
4. Custom Endpoint

---

# ✍️ Cluster / Writer Endpoint

Connects to the **current primary/writer instance**.

```text
Application
     ↓
Cluster Endpoint
     ↓
Current Writer
```

Use for:

- Reads + Writes
- `INSERT`
- `UPDATE`
- `DELETE`
- DDL

If failover occurs:

```text
Old Writer ❌

Replica
   ↓
Promoted
   ↓
New Writer

Cluster Endpoint
       ↓
New Writer
```

The application continues using the same cluster endpoint.

> [!tip] 🎯 Exam Clue
> **Write operations**
> → Cluster / Writer Endpoint

---

# 📖 Reader Endpoint

Provides load balancing across available **Aurora Replicas**.

```text
Application
     ↓
Reader Endpoint
     ↓
 ┌───────┬───────┬───────┐
 ↓       ↓       ↓
Replica Replica Replica
```

Use for:

- `SELECT`
- Reporting
- Read-heavy workloads

The Reader Endpoint **cannot perform writes**.

> [!tip] 🎯 Exam Clue
> **Read scaling + automatic load balancing**
> → Reader Endpoint

---

# 🎯 Instance Endpoint

Connects directly to **one specific DB instance**.

```text
Application
     ↓
Instance Endpoint
     ↓
Replica #2
```

Useful for:

- Troubleshooting
- Performance diagnosis
- Connecting to a specific instance

> [!tip] 🎯 Exam Clue
> **Connect to one specific Aurora instance**
> → Instance Endpoint

---

# 🧩 Custom Endpoint

A Custom Endpoint represents a **group of selected DB instances**.

Example:

```text
Aurora Cluster

High Capacity
├── Replica A
└── Replica B

Low Capacity
├── Replica C
└── Replica D
```

Create:

```text
Production Endpoint
        ↓
Replica A + B

Reporting Endpoint
        ↓
Replica C + D
```

Aurora load balances connections among the instances in each group.

> [!tip] 🎯 Exam Clue
> **Different workloads must use different groups/types of Aurora instances**
> → Custom Endpoint

---

# 🧠 Endpoint Cheat Sheet

| Requirement | Endpoint |
|---|---|
| Write operations | **Cluster / Writer** |
| Load-balanced reads | **Reader** |
| Specific DB instance | **Instance** |
| Specific group of instances | **Custom** |

> [!warning] ⚠️ Exam Trap
> **Writer Endpoint**
> → Current primary
>
> **Reader Endpoint**
> → Load balances reads
>
> **Instance Endpoint**
> → One specific instance
>
> **Custom Endpoint**
> → Selected group of instances

---

# 🚨 Custom Endpoint Exam Scenario

Suppose:

```text
High-capacity replicas
→ Production queries

Low-capacity replicas
→ Reporting
```

Don't use the normal Reader Endpoint because it load balances across the available reader replicas without separating them according to your workload requirement.

Instead:

```text
Production
    ↓
Custom Endpoint A
    ↓
High-Capacity Replicas


Reporting
    ↓
Custom Endpoint B
    ↓
Low-Capacity Replicas
```

> [!danger] 🎯 Exam Clue
> **Route workloads based on instance capacity/configuration**
> → Custom Endpoints

---

# 📈 Aurora Auto Scaling

Aurora Auto Scaling can automatically adjust the number of Aurora Replicas according to workload.

```text
Read Traffic ↑
      ↓
Aurora Auto Scaling
      ↓
Add Replicas
```

Then:

```text
Read Traffic ↓
      ↓
Remove Replicas
```

Useful for variable read workloads.

> [!tip] 🎯 Exam Clue
> **Automatically scale Aurora read capacity**
> → Aurora Auto Scaling

---

# 🌍 Aurora Global Database

Aurora Global Database spans **multiple AWS Regions**.

```text
              Region A
          Primary Cluster
                 │
                 │ Replication
                 ↓
              Region B
         Secondary Cluster
                 │
                 ↓
              Region C
         Secondary Cluster
```

Designed for:

- Global applications
- Low-latency global reads
- Cross-Region disaster recovery
- Protection from Region-level failures

> [!tip] 🎯 Exam Clue
> **Aurora + global reads + cross-Region DR**
> → Aurora Global Database

---

# 🌎 Global Database Mental Model

```text
Normal Aurora
→ Multi-AZ / Regional HA

Aurora Global Database
→ Multi-Region
```

> [!warning] ⚠️ Exam Trap
> **Multi-AZ ≠ Multi-Region**
>
> Multi-AZ protects against **AZ failures**.
>
> Global Database helps protect against **Regional failures**.

---

# ⚡ Aurora Serverless

Aurora Serverless automatically adjusts database capacity based on demand.

```text
Application Traffic
       ↓
Aurora Serverless
       ↓
Capacity scales
up / down
```

Instead of choosing a fixed DB instance size, Aurora adjusts capacity according to workload.

Useful for:

- Variable workloads
- Unpredictable workloads
- Applications where database demand changes significantly

> [!tip] 🎯 Exam Clue
> **Aurora + unpredictable/intermittent database workload**
> → Aurora Serverless

---

# ⚔️ Provisioned vs Serverless

## Aurora Provisioned

You choose DB instance classes.

```text
db.r...
db.t...
etc.
```

Good for:

- Predictable workloads
- Consistent database usage
- More explicit capacity planning

## Aurora Serverless

Capacity adjusts automatically.

Good for:

- Variable traffic
- Unpredictable workloads
- Minimal capacity management

> [!tip] 🧠 Mental Model
> **Predictable workload**
> → Provisioned
>
> **Unpredictable / variable**
> → Serverless

---

# ⏪ Aurora Backtrack

Aurora Backtrack allows you to move the database back to an earlier point in time **without restoring a backup into a new cluster**.

Example:

```text
10:00  Database OK
  ↓
10:15  Bad UPDATE 💥
  ↓
10:20  Detect problem
  ↓
Backtrack
  ↓
10:14  Database restored
```

Useful for quickly recovering from:

- Accidental DELETE
- Incorrect UPDATE
- Application mistakes

> [!tip] 🎯 Exam Clue
> **Undo database mistake quickly without restoring backup**
> → Aurora Backtrack

> [!warning]
> Backtrack is **not a replacement for backups**.

---

# ⚔️ Backtrack vs PITR

## Backtrack

```text
Existing Aurora Cluster
       ↓
Rewind
       ↓
Earlier State
```

Fast rollback of the existing cluster.

## Point-In-Time Restore

```text
Backup
   ↓
Restore
   ↓
New DB Cluster
```

> [!tip] 🎯 Exam Clue
> **Quickly rewind Aurora**
> → Backtrack
>
> **Restore database from backup**
> → PITR

---

# 💾 Backups

Aurora automatically backs up its cluster volume.

Supports:

- Automated backups
- Point-In-Time Recovery
- Manual snapshots

Backup retention can be configured.

Snapshots can be retained independently.

---

# 📸 Database Cloning

Aurora supports fast **database cloning**.

```text
Production Aurora
       ↓
     Clone
       ↓
Test / Development Aurora
```

Initially, the clone uses a **copy-on-write** model.

This avoids immediately making a full physical copy of all data.

Useful for:

- Testing
- Development
- Creating database copies quickly

> [!tip] 🎯 Exam Clue
> **Quickly create test copy of large Aurora database**
> → Aurora Database Cloning

---

# 🔐 Security

Aurora supports encryption at rest using **AWS KMS**.

Can protect:

- Cluster storage
- Backups
- Snapshots
- Replicas

Encryption in transit can use:

```text
SSL / TLS
```

---

# 🔑 IAM Database Authentication

Aurora MySQL and Aurora PostgreSQL can support IAM database authentication.

Instead of:

```text
Username + Long-Lived Password
```

you can use:

```text
IAM
 ↓
Temporary Authentication Token
 ↓
Aurora
```

> [!tip] 🎯 Exam Clue
> **Authenticate to Aurora without long-lived DB password**
> → IAM Database Authentication

---

# 🔑 Secrets Manager

Secrets Manager can store and automatically rotate database credentials.

```text
Application
     ↓
Secrets Manager
     ↓
Credentials
     ↓
Aurora
```

> [!tip] 🎯 Exam Clue
> **Store + automatically rotate Aurora password**
> → Secrets Manager

---

# 🔌 RDS Proxy + Aurora

RDS Proxy can sit between applications and Aurora.

```text
Lambda / Application
        ↓
     RDS Proxy
        ↓
      Aurora
```

Benefits:

- Connection pooling
- Reuse database connections
- Handle connection spikes
- Improve application resilience

Especially useful with:

```text
Lambda → Aurora
```

> [!tip] 🎯 Exam Clue
> **Lambda creates too many Aurora connections**
> → RDS Proxy

---

# ⚔️ Aurora vs Standard RDS

Both are managed relational databases.

## Standard RDS

Supports engines such as:

- MySQL
- PostgreSQL
- MariaDB
- Oracle
- SQL Server

## Aurora

AWS-designed relational engine compatible with:

- MySQL
- PostgreSQL

Aurora emphasizes:

- Distributed storage
- Read scaling
- Fast failover
- Up to 15 Aurora Replicas
- Global Database
- Serverless options

> [!tip] 🎯 Exam Clue
> **Need Oracle / SQL Server**
> → Standard RDS
>
> **MySQL/PostgreSQL compatible + AWS-optimized HA/scaling**
> → Aurora

---

# ⚔️ Aurora vs DynamoDB

## Aurora

→ Relational  
→ SQL  
→ Joins  
→ Transactions  
→ MySQL/PostgreSQL compatibility

## DynamoDB

→ NoSQL  
→ Key-value/document  
→ Massive horizontal scale  
→ Serverless

> [!tip] 🎯 Exam Clue
> **Relational SQL**
> → Aurora
>
> **Massive key-value NoSQL**
> → DynamoDB

---

# ⚔️ Aurora vs Redshift

## Aurora

Primarily:

**OLTP**

```text
Orders
Payments
Users
Application Transactions
```

## Redshift

Primarily:

**OLAP**

```text
Data Warehouse
Analytics
BI
Large Aggregations
```

> [!tip] 🧠 Mental Model
> **Aurora → Run the application**
>
> **Redshift → Analyze the business**

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| MySQL/PostgreSQL compatible AWS DB | **Aurora** |
| 6 copies across 3 AZs | **Aurora Storage** |
| Aurora writes | **Cluster/Writer Endpoint** |
| Load-balanced reads | **Reader Endpoint** |
| Connect specific instance | **Instance Endpoint** |
| Route workloads to selected replicas | **Custom Endpoint** |
| Read scaling | **Aurora Replicas** |
| Up to 15 read replicas | **Aurora** |
| Automatic read scaling | **Aurora Auto Scaling** |
| Unpredictable DB workload | **Aurora Serverless** |
| Multi-Region Aurora | **Global Database** |
| Region-level DR | **Global Database** |
| Quickly rewind DB | **Backtrack** |
| Quickly create test DB copy | **Database Cloning** |
| Too many DB connections | **RDS Proxy** |
| Rotate DB credentials | **Secrets Manager** |
| SQL/relational | **Aurora/RDS** |
| NoSQL | **DynamoDB** |
| Data warehouse | **Redshift** |

---

# 🚨 Most Important Exam Distinctions

```text
Writer Endpoint
→ WRITE

Reader Endpoint
→ READ + LOAD BALANCING

Instance Endpoint
→ ONE INSTANCE

Custom Endpoint
→ SELECTED GROUP
```

```text
Aurora Replica
→ READ SCALING
→ FAILOVER TARGET

Aurora Auto Scaling
→ Automatically add/remove replicas
```

```text
Multi-AZ Aurora
→ REGIONAL HA

Global Database
→ MULTI-REGION
```

```text
Provisioned Aurora
→ Predictable workload

Aurora Serverless
→ Variable/unpredictable workload
```

```text
Backtrack
→ REWIND existing cluster

PITR
→ RESTORE from backup
```

---

> [!abstract] 🧠 Amazon Aurora in 30 Seconds
> **Type:** Managed relational database
>
> **Compatible:** MySQL + PostgreSQL
>
> **Storage:** 6 copies / 3 AZs
>
> **Writer:** Cluster Endpoint
>
> **Reads:** Reader Endpoint
>
> **Specific instance:** Instance Endpoint
>
> **Selected replicas:** Custom Endpoint
>
> **Read Scaling:** Aurora Replicas
>
> **Replicas:** Up to 15
>
> **Automatic read scaling:** Aurora Auto Scaling
>
> **Variable workload:** Aurora Serverless
>
> **Multi-Region:** Global Database
>
> **Quick rewind:** Backtrack
>
> **Fast test copy:** Database Cloning
>
> **Connection pooling:** RDS Proxy