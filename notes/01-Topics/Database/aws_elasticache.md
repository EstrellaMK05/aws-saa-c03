# ⚡ Amazon ElastiCache

> [!summary] Mental Model
> **ElastiCache = In-Memory Cache**
>
> Put frequently accessed data in memory to reduce database load and improve latency.
>
> ```text
> Application
>     ↓
> ElastiCache ⚡
>     ↓ cache miss
> Database
> ```

---

# 🎯 Core Purpose

Amazon ElastiCache is a managed in-memory data store/cache.

Common use cases:

- Database caching
- Session storage
- Frequently accessed data
- Low-latency reads
- Reduce database load
- Leaderboards
- Real-time applications

```text
Without Cache

App → Database
App → Database
App → Database

With Cache

App → ElastiCache ⚡
          ↓ cache miss
       Database
```

> [!tip] Exam Pattern
> **Database has heavy read traffic**
> +
> **Same data is repeatedly requested**
> +
> **Need very low latency**
>
> → ✅ Amazon ElastiCache

---

# 🧠 Engines

ElastiCache supports:

- **Valkey**
- **Redis OSS**
- **Memcached**

For SAA, the most important decision is usually:

```text
Need HA / replication / advanced data structures?
        ↓
Valkey / Redis OSS

Need simple distributed cache?
        ↓
Memcached
```

---

# 🔴 Valkey / Redis OSS

Think:

- Replication
- High Availability
- Automatic failover
- Read replicas
- Complex data structures
- Pub/Sub
- Sorted sets
- Backup and restore

```text
Application
     ↓
Primary
  ↙     ↘
Replica Replica
```

> [!tip] Exam Pattern
> **Highly available cache**
>
> → ✅ Valkey / Redis OSS

### Session Storage

Very common SAA pattern:

```text
User
 ↓
ALB
 ↓
EC2 ──┐
EC2 ──┼──→ ElastiCache
EC2 ──┘      Sessions
```

The session is not tied to a specific EC2 instance.

> [!tip]
> **Distributed application + shared session state**
>
> → Think **ElastiCache Valkey / Redis OSS**

---

# 🏆 Leaderboards

Valkey / Redis OSS supports sorted sets.

```text
Player → Score
A      → 9500
B      → 8200
C      → 7600
```

> [!tip] Exam Pattern
> **Gaming leaderboard / ranking**
>
> → Valkey / Redis OSS

---

# 🟦 Memcached

Think:

**Simple distributed cache**

Characteristics:

- Simple key-value/object caching
- Multi-threaded
- Easy horizontal scaling
- No native replication-based HA like Valkey/Redis OSS
- No Pub/Sub
- No sorted sets

```text
Application
   ↓
Memcached Cluster
├── Node
├── Node
└── Node
```

> [!tip] Exam Pattern
> **Simple cache**
> +
> **Need to scale horizontally**
> +
> **No advanced features required**
>
> → ✅ Memcached

---

# 🆚 Valkey / Redis OSS vs Memcached

| Feature | Valkey / Redis OSS | Memcached |
|---|---|---|
| In-memory cache | ✅ | ✅ |
| Simple caching | ✅ | ✅ |
| Replication | ✅ | ❌ |
| High Availability | ✅ | ❌ |
| Automatic Failover | ✅ | ❌ |
| Backup / Restore | ✅ | Limited / deployment-dependent |
| Pub/Sub | ✅ | ❌ |
| Sorted Sets | ✅ | ❌ |
| Complex Data Types | ✅ | ❌ |
| Multi-threaded | ❌ | ✅ |

> [!summary] Memory
> **Need HA / replication / advanced features**
> → Valkey / Redis OSS
>
> **Simple cache**
> → Memcached

---

# 🗄️ Database Caching Pattern

Common architecture:

```text
             ┌── Cache Hit ──→ Return Data
             │
Application → ElastiCache
             │
             └── Cache Miss
                    ↓
                 Database
                    ↓
               Update Cache
                    ↓
                Return Data
```

This reduces:

- Database reads
- Database load
- Response latency

---

# 🆚 ElastiCache vs RDS Read Replica

Both can reduce read pressure, but they solve different problems.

### Read Replica

```text
Application
     ↓
Read Replica
     ↓
Database Data
```

Still querying a relational database.

### ElastiCache

```text
Application
     ↓
Memory Cache ⚡
```

Frequently accessed data is served from memory.

> [!tip] Exam Pattern
> **Repeated reads + extremely low latency**
>
> → ElastiCache
>
> **Need additional SQL read capacity**
>
> → RDS Read Replica

---

# 🆚 ElastiCache vs DAX

Very important.

```text
RDS / Aurora / Generic DB
        ↓
    ElastiCache

DynamoDB
   ↓
  DAX
```

| Requirement | Think |
|---|---|
| Cache for RDS/Aurora | ElastiCache |
| General application cache | ElastiCache |
| DynamoDB-specific cache | DAX |

> [!danger] Exam Trap
> **DynamoDB + microsecond read cache**
>
> → DAX
>
> Don't automatically choose ElastiCache.

---

# 🏗️ High Availability

For HA requirements, think **Valkey / Redis OSS replication + Multi-AZ**.

```text
        Primary
       AZ-A
         ↓
     Replication
         ↓
       Replica
       AZ-B
```

If the primary fails:

```text
Primary ❌
    ↓
Automatic Failover
    ↓
Replica → Primary
```

> [!tip] Exam Pattern
> **Session cache cannot be lost during instance failure**
> +
> **Automatic failover**
>
> → Valkey / Redis OSS with Multi-AZ

---

# ☁️ ElastiCache Serverless

ElastiCache also provides a Serverless deployment option.

Think:

- No node sizing
- Automatic scaling
- Unpredictable workloads
- Reduced capacity-management overhead

```text
Application
     ↓
ElastiCache Serverless
     ↓
Automatic Scaling
```

> [!tip] Exam Pattern
> **Unpredictable cache workload**
> +
> **Least operational overhead**
>
> → ElastiCache Serverless

---

# 🔐 Security

ElastiCache supports security features such as:

- Encryption in transit
- Encryption at rest
- IAM authentication for supported engines/configurations
- AUTH
- RBAC
- VPC security controls

> [!important]
> ElastiCache is normally accessed by applications inside a VPC.
>
> Use Security Groups to control network access.

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> RDS database overloaded by repeated reads.
>
> → ✅ ElastiCache

---

> [!danger] Trap 2
> Need additional relational SQL read capacity.
>
> → ✅ RDS Read Replica
>
> Not necessarily ElastiCache.

---

> [!danger] Trap 3
> DynamoDB needs a cache.
>
> → ✅ DAX

---

> [!danger] Trap 4
> Need HA + automatic failover for cache.
>
> → ✅ Valkey / Redis OSS

---

> [!danger] Trap 5
> Need a simple distributed object cache with horizontal scaling.
>
> → ✅ Memcached

---

> [!danger] Trap 6
> Shared session state for multiple EC2 instances.
>
> → ✅ ElastiCache Valkey / Redis OSS

```text
ALB
 ↓
ASG
├── EC2
├── EC2
└── EC2
     ↓
ElastiCache
Shared Sessions
```

---

# 🧠 ElastiCache in 20 Seconds

```text
Amazon ElastiCache
       ↓
In-Memory Cache ⚡

Repeated DB Reads
→ ElastiCache

HA / Replication / Failover
→ Valkey / Redis OSS

Simple Cache
→ Memcached

Sessions
→ Valkey / Redis OSS

Leaderboard
→ Valkey / Redis OSS

DynamoDB Cache
→ DAX

SQL Read Scaling
→ RDS Read Replica

Unpredictable Cache Capacity
→ ElastiCache Serverless
```

> [!summary] SAA Memory
> **CACHE → ElastiCache**
>
> **HA CACHE → Valkey / Redis OSS**
>
> **SIMPLE CACHE → Memcached**
>
> **DYNAMODB CACHE → DAX**
>
> **SQL READ SCALE → Read Replica**