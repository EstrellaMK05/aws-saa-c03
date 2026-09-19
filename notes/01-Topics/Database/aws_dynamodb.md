# ⚡ Amazon DynamoDB

> [!summary] Mental Model
> **DynamoDB = Serverless NoSQL database for massive scale + low latency**
>
> ```text
> Application
>     ↓
> DynamoDB
>     ↓
> Key-Value / Document Data
> ```
>
> No servers to manage.
> Designed for **single-digit millisecond performance at scale**.

---

# 📌 Core

Amazon DynamoDB is a fully managed **NoSQL** database service.

Key characteristics:

- Serverless / fully managed
- Key-value and document database
- Automatic scaling
- Built-in high availability
- Data replicated across multiple AZs
- Encryption at rest
- Very low latency
- Schemaless except for the primary key

> [!tip] 🎯 Exam Clue
> **Massive scale + NoSQL + very low latency + minimal operations**
> → DynamoDB

---

# 🧱 Core Components

```text
Table
  ↓
Items
  ↓
Attributes
```

### Table

Collection of data.

Example:

```text
Users
```

### Item

A single record.

```json
{
  "UserId": "123",
  "Name": "Gabriela",
  "Country": "Peru"
}
```

### Attribute

Individual data element.

```text
UserId
Name
Country
```

> [!note]
> DynamoDB is **schemaless** except for attributes that form the primary key.

---

# 🔑 Primary Key

Every item must be uniquely identified by a **Primary Key**.

Two possibilities:

1. Partition Key
2. Partition Key + Sort Key

---

## 🔹 Simple Primary Key

Only a **Partition Key**.

```text
UserId
```

Example:

```text
UserId = 123
```

Each partition key value must uniquely identify an item.

---

## 🔹 Composite Primary Key

Uses:

```text
Partition Key + Sort Key
```

Example:

```text
UserId    OrderDate
123       2026-01-01
123       2026-02-01
123       2026-03-01
```

The combination must be unique.

This allows multiple related items under the same partition key.

---

# 🧩 Partition Key

The Partition Key determines how DynamoDB distributes data across physical partitions.

```text
Partition Key
     ↓
Hash Function
     ↓
Partition
```

Good partition keys have **high cardinality**.

Examples:

```text
UserId
OrderId
CustomerId
```

These distribute requests across many values.

---

# 🔥 Hot Partition

A **Hot Partition** occurs when too much traffic is concentrated on a small number of partition key values.

Bad example:

```text
Partition Key = Country

Peru       → 5%
Chile      → 5%
USA        → 80% 🔥
Others     → 10%
```

Most traffic goes to:

```text
USA
```

→ Uneven workload distribution.

Possible result:

- Throttling
- Performance issues
- Uneven capacity usage

> [!tip] 🎯 Exam Clue
> **Uneven traffic / throttling on one partition**
> → Hot Partition
>
> **Solution**
> → Choose a high-cardinality partition key

> [!warning] ⚠️ Exam Trap
> DynamoDB may have enough overall capacity but still experience problems from a **poor partition-key access pattern**.

---

# 🔃 Sort Key

A Sort Key organizes items that share the same Partition Key.

Example:

```text
CustomerId = 123

OrderDate
├── 2026-01-01
├── 2026-02-01
└── 2026-03-01
```

Useful for:

- Time ranges
- Grouped data
- Ordered queries

Example:

```text
CustomerId = 123
AND
OrderDate BETWEEN Jan AND Mar
```

---

# 🔎 Secondary Indexes

Secondary indexes provide additional ways to query your data.

Two types:

1. **Global Secondary Index — GSI**
2. **Local Secondary Index — LSI**

---

# 🌎 Global Secondary Index — GSI

A GSI can use a **different Partition Key and Sort Key** from the base table.

Example:

Base table:

```text
PK: CustomerId
SK: OrderId
```

GSI:

```text
PK: ProductId
SK: OrderDate
```

Now you can efficiently query by:

```text
ProductId
```

instead of only `CustomerId`.

### Important

A GSI:

- Can have a different Partition Key
- Can have a different Sort Key
- Can be created **after table creation**
- Has separate capacity characteristics
- Supports alternative access patterns

> [!tip] 🎯 Exam Clue
> **Need to query using a completely different key**
> → GSI

---

# 📍 Local Secondary Index — LSI

An LSI uses:

```text
SAME Partition Key
DIFFERENT Sort Key
```

Example:

Base table:

```text
PK: CustomerId
SK: OrderId
```

LSI:

```text
PK: CustomerId
SK: OrderDate
```

### Important

An LSI:

- Must use the same Partition Key
- Uses a different Sort Key
- Must be created **when the table is created**

> [!warning] ⚠️ Exam Trap
> You CANNOT add an LSI later.

---

# ⚔️ GSI vs LSI

| | GSI | LSI |
|---|---|---|
| Partition Key | Can be different | **Same as table** |
| Sort Key | Can be different | Different |
| Create after table creation | ✅ | ❌ |
| Alternative partition key | ✅ | ❌ |

> [!tip] 🧠 Mental Model
> **GSI = GLOBAL → Different Partition Key**
>
> **LSI = LOCAL → Same Partition Key**

---

# 🔍 Query

`Query` efficiently retrieves items based on a Partition Key.

Example:

```text
CustomerId = 123
```

Optionally:

```text
CustomerId = 123
AND
OrderDate > 2026-01-01
```

```text
Partition Key
     ↓
Specific Partition
     ↓
Matching Items
```

> [!tip] 🎯 Exam Clue
> **Know the Partition Key**
> → Query

---

# 🔦 Scan

`Scan` reads every item in a table or index.

```text
Table
 ↓
Item
 ↓
Item
 ↓
Item
 ↓
Item
 ↓
Filter Results
```

This can consume significantly more resources.

> [!warning] ⚠️ Exam Trap
> **Query is generally preferred over Scan.**
>
> Query → targeted access
>
> Scan → examine the entire table/index

---

# ⚔️ Query vs Scan

| | Query | Scan |
|---|---|---|
| Uses Partition Key | ✅ | ❌ required |
| Reads targeted items | ✅ | ❌ |
| Reads entire table/index | ❌ | ✅ |
| Efficient | ✅ | Less efficient |
| Preferred | ✅ | When necessary |

> [!tip] 🎯 Exam Clue
> **Application repeatedly scans huge DynamoDB table**
> → Consider redesigning access pattern / using an index.

---

# 📊 Capacity Modes

DynamoDB has two main capacity modes:

1. **On-Demand**
2. **Provisioned**

---

# ⚡ On-Demand Capacity

DynamoDB automatically handles request capacity.

You pay per request.

Best for:

- Unpredictable workloads
- New applications
- Traffic spikes
- Unknown capacity requirements

```text
Low Traffic
    ↓
Huge Spike
    ↓
Low Traffic

DynamoDB adjusts automatically
```

> [!tip] 🎯 Exam Clue
> **Unpredictable workload**
> → On-Demand

---

# 📐 Provisioned Capacity

You configure expected:

```text
RCU
→ Read Capacity Units

WCU
→ Write Capacity Units
```

Best for:

- Predictable traffic
- Stable workloads
- Capacity you can forecast

Can be combined with:

**DynamoDB Auto Scaling**

> [!tip] 🎯 Exam Clue
> **Predictable workload**
> → Provisioned Capacity

---

# ⚔️ On-Demand vs Provisioned

| | On-Demand | Provisioned |
|---|---|---|
| Capacity planning | Minimal | Required |
| Unpredictable traffic | ✅ | Less ideal |
| Predictable traffic | ✅ | ✅ |
| Pay model | Per request | Capacity |
| Auto Scaling | Built in | Can configure |

Mental model:

```text
Unknown / Spiky
→ On-Demand

Predictable
→ Provisioned
```

---

# 📖 Read Consistency

DynamoDB supports different consistency models.

---

## Eventually Consistent Read

Default for standard DynamoDB reads.

A read may briefly return older data after a successful write.

```text
WRITE
 ↓
Replica propagation
 ↓
READ

Possibly slightly stale
```

Advantages:

- Lower cost
- Higher read efficiency

---

## Strongly Consistent Read

Returns the latest successful write.

Example:

```text
GetItem(
  ConsistentRead = true
)
```

> [!tip] 🎯 Exam Clue
> **Must return latest successfully written value**
>
> **Cannot tolerate stale data**
>
> → Strongly Consistent Read

---

# ⚔️ Eventually vs Strongly Consistent

| | Eventually | Strongly |
|---|---|---|
| Default | ✅ | ❌ |
| May return stale data | ✅ | ❌ |
| Latest successful write | Not guaranteed immediately | ✅ |
| Cost | Lower | Higher |

> [!warning] ⚠️ Exam Trap
> `GetItem` does NOT automatically mean strongly consistent.
>
> Set:
>
> `ConsistentRead = true`

---

# 🚀 DynamoDB Accelerator — DAX

DAX is a fully managed **in-memory cache for DynamoDB**.

```text
Application
    ↓
   DAX
    ↓
DynamoDB
```

DAX can provide **microsecond read latency**.

Useful for:

- Read-heavy applications
- Repeated reads
- Hot/read-heavy items
- Extremely low read latency

```text
Request
   ↓
DAX

Cache HIT
→ Return immediately

Cache MISS
→ DynamoDB
```

> [!tip] 🎯 Exam Clue
> **DynamoDB + microsecond read latency**
> → DAX

---

# ⚠️ DAX and Strong Consistency

DAX is designed around **eventually consistent reads**.

It is NOT the solution when an application requires strongly consistent reads.

> [!danger] 🚨 Exam Trap
> **Microsecond DynamoDB reads**
> → DAX
>
> **Must always read latest value**
> → Strongly Consistent Read
>
> NOT DAX.

---

# ⚔️ DAX vs ElastiCache

## DAX

Purpose-built cache for:

```text
DynamoDB
```

Minimal application changes because it is DynamoDB-compatible.

## ElastiCache

General-purpose cache.

Can cache data from:

- RDS
- APIs
- Applications
- Other data sources

> [!tip] 🧠 Mental Model
> **Cache specifically for DynamoDB**
> → DAX
>
> **General application/database cache**
> → ElastiCache

---

# 🌊 DynamoDB Streams

DynamoDB Streams capture item-level changes.

Events include:

```text
INSERT
MODIFY
REMOVE
```

Architecture:

```text
DynamoDB
    ↓
DynamoDB Stream
    ↓
Lambda
    ↓
Action
```

Useful for:

- Event-driven applications
- Notifications
- Replication workflows
- Materialized views
- Auditing/change processing

Stream records are retained for **24 hours**.

> [!tip] 🎯 Exam Clue
> **Run Lambda whenever DynamoDB item changes**
> → DynamoDB Streams + Lambda

---

# 🆚 DynamoDB Streams vs Kinesis Data Streams

Both can be used for streaming architectures, but:

### DynamoDB Streams

→ Changes occurring to DynamoDB items

### Kinesis Data Streams

→ General-purpose real-time streaming data

Mental model:

```text
DynamoDB Change Events
→ DynamoDB Streams

General Streaming Data
→ Kinesis
```

---

# 🌎 Global Tables

DynamoDB Global Tables provide a **multi-Region database**.

```text
       Region A
      DynamoDB
         ↕
     Replication
         ↕
       Region B
      DynamoDB
```

Useful for:

- Globally distributed applications
- Multi-Region availability
- Low-latency regional access
- Disaster recovery

Global Tables support writes across Regions.

> [!tip] 🎯 Exam Clue
> **DynamoDB active-active / multi-Region application**
> → Global Tables

---

# 🧠 Global Tables vs Read Replica

Don't think of Global Tables like RDS Read Replicas.

```text
RDS Read Replica
→ primarily READ scaling

DynamoDB Global Tables
→ multi-Region replication + writes
```

> [!tip] 🎯 Exam Clue
> **Multi-Region DynamoDB writes**
> → Global Tables

---

# 🔄 DynamoDB Transactions

DynamoDB supports **ACID transactions**.

Multiple operations can succeed or fail as one unit.

```text
Operation A
Operation B
Operation C
     ↓
Transaction
     ↓
ALL succeed ✅

or

ALL fail ❌
```

Useful for scenarios such as:

```text
Create Order
+
Decrease Inventory
+
Record Payment State
```

> [!tip] 🎯 Exam Clue
> **Multiple DynamoDB changes must succeed or fail together**
> → DynamoDB Transaction

---

# 💾 Backups

DynamoDB supports:

- On-Demand Backups
- Point-In-Time Recovery (PITR)

---

# 📸 On-Demand Backup

Creates a full backup of a DynamoDB table.

Does not consume table provisioned throughput.

Useful for:

- Long-term backups
- Compliance
- Manual recovery points

---

# ⏱️ Point-In-Time Recovery — PITR

Provides continuous backups.

Allows restoring a table to a point within the previous:

**35 days**

```text
Table
  ↓
Accidental Delete 💥
  ↓
PITR
  ↓
Restore previous state
```

> [!tip] 🎯 Exam Clue
> **Recover DynamoDB from accidental write/delete**
> → PITR

---

# ⚠️ Restore Behavior

A DynamoDB backup restore creates a:

# **NEW TABLE**

It does not overwrite the existing table.

```text
Backup
   ↓
Restore
   ↓
New DynamoDB Table
```

> [!warning] ⚠️ Exam Trap
> DynamoDB restore does NOT overwrite the existing table.

---

# ⏳ Time To Live — TTL

TTL allows DynamoDB to automatically remove expired items.

You define an attribute containing an expiration timestamp.

Example:

```json
{
  "SessionId": "abc123",
  "expiresAt": 1780000000
}
```

Useful for:

- Sessions
- Temporary data
- Expiring records
- Old application data

> [!tip] 🎯 Exam Clue
> **Automatically delete expired DynamoDB items**
> → TTL

---

# 🔐 Encryption

DynamoDB encrypts data at rest.

AWS KMS can be used for encryption key management.

Encryption in transit uses HTTPS/TLS.

---

# 🔒 IAM

Access to DynamoDB can be controlled using IAM.

Permissions can restrict:

- Tables
- Actions
- Items
- Attributes

Example:

```text
Application
    ↓
IAM Role
    ↓
DynamoDB
```

---

# 🌐 VPC Endpoint

Private resources can access DynamoDB using a:

# **Gateway VPC Endpoint**

```text
Private EC2
    ↓
Route Table
    ↓
Gateway Endpoint
    ↓
DynamoDB
```

No:

- NAT Gateway
- Internet Gateway
- Public Internet

required for this access path.

> [!tip] 🎯 Exam Clue
> **Private EC2 → DynamoDB + lowest-cost private connectivity**
> → Gateway VPC Endpoint

---

# 📊 Monitoring

## CloudWatch

Provides metrics such as:

- Read/write usage
- Throttled requests
- Latency
- Capacity consumption

## CloudTrail

Records DynamoDB API activity.

Mental model:

```text
CloudWatch
→ Performance / Metrics

CloudTrail
→ Who did what?
```

---

# ⚔️ DynamoDB vs RDS

| | DynamoDB | RDS |
|---|---|---|
| Type | NoSQL | Relational |
| SQL | ❌ | ✅ |
| Joins | ❌ | ✅ |
| Schema | Flexible | Structured |
| Scaling | Horizontal / automatic | Traditionally more constrained |
| Server management | Serverless/managed | Managed DB instances |
| Massive scale | ✅ | Depends on architecture |

> [!tip] 🎯 Exam Clue
> **SQL + joins + relational data**
> → RDS
>
> **Key-value + massive scale + low latency**
> → DynamoDB

---

# ⚔️ DynamoDB vs S3

## DynamoDB

→ Database  
→ Millisecond access  
→ Key-based queries  
→ Frequent updates

## S3

→ Object storage  
→ Files/objects  
→ Images/videos/backups/data lake

```text
Application Records
→ DynamoDB

Files / Objects
→ S3
```

---

# 🔥 Important: Partition Key Design

Good:

```text
UserId
OrderId
DeviceId
```

Bad when traffic is concentrated:

```text
Status = "ACTIVE"

Country = "USA"

Date = "2026-09-19"
```

A good key distributes requests across many partitions.

> [!danger] 🚨 Exam Trap
> **High-cardinality Partition Key**
> → Better workload distribution
>
> **Low-cardinality / extremely popular key**
> → Potential Hot Partition

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| Serverless NoSQL | **DynamoDB** |
| Key-value database | **DynamoDB** |
| Massive scale + low latency | **DynamoDB** |
| Unpredictable traffic | **On-Demand** |
| Predictable traffic | **Provisioned** |
| Latest successful write required | **Strongly Consistent Read** |
| Different Partition Key for query | **GSI** |
| Same PK + different Sort Key | **LSI** |
| Add index after table creation | **GSI** |
| Uneven traffic / throttling | **Hot Partition** |
| Know Partition Key | **Query** |
| Read entire table | **Scan** |
| Microsecond reads | **DAX** |
| Strong consistency required | **NOT DAX** |
| React to item changes | **DynamoDB Streams** |
| Multi-Region active-active | **Global Tables** |
| Multiple atomic operations | **Transactions** |
| Restore last 35 days | **PITR** |
| Automatically expire items | **TTL** |
| Private VPC → DynamoDB | **Gateway Endpoint** |

---

# 🚨 Most Important Exam Distinctions

```text
GSI
→ Different Partition Key allowed
→ Can add later

LSI
→ Same Partition Key
→ Must create with table
```

```text
Query
→ Targeted using Partition Key

Scan
→ Reads entire table/index
```

```text
On-Demand
→ Unpredictable workload

Provisioned
→ Predictable workload
```

```text
Eventually Consistent
→ May briefly be stale

Strongly Consistent
→ Latest successful write
```

```text
DAX
→ Microsecond cached reads

Streams
→ Capture changes

Global Tables
→ Multi-Region
```

---

> [!abstract] 🧠 DynamoDB in 30 Seconds
> **Type:** Serverless NoSQL
>
> **Primary Key:** Partition Key [+ Sort Key]
>
> **Good PK:** High cardinality
>
> **Bad distribution:** Hot Partition
>
> **GSI:** Different PK, can add later
>
> **LSI:** Same PK, create with table
>
> **Query:** Targeted
>
> **Scan:** Entire table
>
> **Unknown traffic:** On-Demand
>
> **Predictable traffic:** Provisioned
>
> **Latest data:** Strongly Consistent Read
>
> **Microsecond reads:** DAX
>
> **Item changes:** DynamoDB Streams
>
> **Multi-Region:** Global Tables
>
> **ACID:** Transactions
>
> **Recovery:** PITR
>
> **Expiration:** TTL
>
> **Private access:** Gateway Endpoint