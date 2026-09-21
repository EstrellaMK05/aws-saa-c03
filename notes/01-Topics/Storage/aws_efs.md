# 📁 Amazon EFS

> [!summary] Mental Model
> **Amazon EFS = Managed Shared File Storage**
>
> Think:
>
> **Multiple compute instances need to access the SAME files**
>
> → Amazon EFS

---

# 🎯 Core Purpose

Amazon Elastic File System (EFS) provides **managed, elastic file storage**.

It uses the **NFS protocol** and can be mounted by multiple compute resources simultaneously.

```text
EC2 ──┐
EC2 ──┼──→ Amazon EFS
EC2 ──┘       │
              └── Shared Files
```

Common use cases:

- Shared application files
- Web content
- Content management systems
- Home directories
- Container persistent storage
- Shared data across multiple EC2 instances

> [!tip] Exam Pattern
> **Multiple EC2 instances need concurrent access to the same files**
>
> → ✅ Amazon EFS

---

# 🌎 Regional EFS

A Regional EFS file system stores data redundantly across multiple Availability Zones.

```text
          Amazon EFS
        Regional File System
              │
       ┌──────┼──────┐
       ↓      ↓      ↓
     AZ-A    AZ-B    AZ-C
      │       │       │
     EC2     EC2     EC2
```

Designed for:

- High availability
- Multi-AZ architectures
- Shared file storage

> [!important] Exam Pattern
> **Shared file system + EC2 across multiple AZs + High Availability**
>
> → ✅ Regional Amazon EFS

---

# 🏠 EFS One Zone

EFS One Zone stores data within **one Availability Zone**.

```text
AZ-A
 │
 ├── EC2
 │
 └── EFS One Zone
```

Advantages:

- Lower cost

Tradeoff:

- Less resilient than Regional EFS

> [!warning] Exam
> **Cost optimization + data can tolerate AZ-level risk**
>
> → EFS One Zone
>
> **High Availability across AZs**
>
> → Regional EFS

---

# 🌐 Mount Targets

To access EFS from a VPC, clients connect through **mount targets**.

A mount target:

- Exists in a subnet
- Has an IP address
- Uses Security Groups
- Provides network access to EFS

Recommended architecture:

```text
VPC
│
├── AZ-A
│   ├── EC2
│   └── EFS Mount Target
│
├── AZ-B
│   ├── EC2
│   └── EFS Mount Target
│
└── AZ-C
    ├── EC2
    └── EFS Mount Target
```

> [!tip]
> Create a mount target in each AZ where clients need to access EFS.

---

# 🔌 NFS

Amazon EFS uses **Network File System (NFS)**.

Important port:

```text
NFS
 ↓
TCP 2049
```

Typical Security Group rule:

```text
EFS Security Group

Inbound:
TCP 2049
Source: EC2 Security Group
```

> [!danger] Exam Pattern
> EC2 cannot mount EFS because network access is blocked.
>
> Check:
>
> → **TCP 2049**
>
> → Security Groups

---

# 💾 Storage Classes

## EFS Standard

For frequently accessed files requiring low latency.

```text
Frequently Accessed
       ↓
EFS Standard
```

---

## EFS Infrequent Access — IA

Cost-optimized for files accessed infrequently.

```text
Less Frequently Accessed
         ↓
      EFS IA
```

---

## EFS Archive

For data accessed only rarely.

```text
Rarely Accessed
      ↓
EFS Archive
```

> [!tip] Memory
> **Standard → Active**
>
> **IA → Infrequent**
>
> **Archive → Rare**

---

# ♻️ Lifecycle Management

EFS Lifecycle Management can automatically move files between storage classes based on access patterns.

```text
Frequently Used
      ↓
EFS Standard
      ↓
Less Frequently Used
      ↓
EFS IA
      ↓
Rarely Used
      ↓
EFS Archive
```

This reduces storage costs without requiring applications to move files manually.

---

# ⚡ Performance Mode

## General Purpose

Recommended for most workloads.

Characteristics:

- Lowest per-operation latency
- Default performance mode
- Suitable for most applications

```text
Most Workloads
     ↓
General Purpose
```

---

## Max I/O

Previous-generation performance mode designed for highly parallelized workloads that can tolerate higher latency.

> [!warning]
> AWS currently recommends **General Purpose** for new EFS file systems.
>
> Do not automatically choose Max I/O just because the question says
> **"high performance."**

---

# 🚀 Throughput Modes

Do NOT confuse **Performance Mode** with **Throughput Mode**.

```text
Performance Mode
→ latency / I/O characteristics

Throughput Mode
→ amount of throughput available
```

---

## ⚡ Elastic Throughput

Automatically scales throughput according to workload activity.

Best for:

- Unpredictable workloads
- Spiky workloads
- Difficult-to-forecast throughput

```text
Unpredictable / Spiky
        ↓
Elastic Throughput
```

> [!tip]
> Elastic is the recommended/default throughput mode for most new workloads.

---

## 🎛️ Provisioned Throughput

You specify the throughput required independently of file system size.

Best when:

- Throughput requirements are known
- You need predictable throughput independent of stored capacity

```text
Known Throughput Requirement
           ↓
Provisioned Throughput
```

---

## 💥 Bursting Throughput

Throughput scales with the amount of data stored.

Uses burst credits to temporarily exceed baseline throughput.

```text
More Storage
    ↓
More Baseline Throughput
    +
Burst Credits
```

---

# 🧠 Throughput Memory Trick

```text
Elastic
→ AUTO / unpredictable

Provisioned
→ I KNOW what I need

Bursting
→ STORAGE SIZE + credits
```

---

# 🔐 Security

Amazon EFS supports:

- Encryption at rest using AWS KMS
- Encryption in transit using TLS
- Security Groups
- IAM authorization
- EFS File System Policies
- POSIX permissions

```text
EC2
 ↓
Security Group
 ↓
Mount Target
 ↓
EFS
 ↓
File Permissions
```

---

# 🚪 EFS Access Points

EFS Access Points provide an application-specific entry point into an EFS file system.

They can enforce:

- Root directory
- POSIX user/group identity
- File system permissions

```text
              EFS
               │
       ┌───────┴───────┐
       ↓               ↓
Access Point A     Access Point B
       ↓               ↓
 Application A     Application B
```

> [!tip] Exam Pattern
> Multiple applications share EFS but require **different directories or identities**.
>
> → ✅ EFS Access Points

---

# 📈 Elastic Capacity

EFS automatically grows and shrinks as files are added and removed.

```text
More Files
   ↓
EFS grows automatically

Delete Files
   ↓
EFS shrinks automatically
```

You don't normally provision filesystem storage capacity beforehand.

---

# 🆚 EFS vs EBS

This is VERY important for SAA.

| EFS | EBS |
|---|---|
| File storage | Block storage |
| NFS | Block device |
| Shared access | Usually attached to an EC2 instance |
| Multi-AZ Regional option | Volume belongs to one AZ |
| Automatically scales storage | Provisioned volume capacity |
| Multiple clients | EC2 disk-like storage |

### Mental Model

```text
EC2 ─┐
EC2 ─┼── EFS
EC2 ─┘
      SHARED

EC2 ───── EBS
           DISK
```

> [!danger] Exam Pattern
> **Multiple EC2 instances across AZs need the same files**
>
> → EFS
>
> **EC2 needs persistent block storage**
>
> → EBS

---

# 🆚 EFS vs S3

| EFS | S3 |
|---|---|
| File system | Object storage |
| NFS | S3 API |
| Mount like filesystem | Access objects |
| Directories/files semantics | Buckets/objects |
| Shared filesystem workloads | Massive object storage |

```text
Need a FILE SYSTEM
      ↓
     EFS

Need OBJECT STORAGE
      ↓
      S3
```

---

# 🆚 EFS vs Instance Store

```text
EFS
→ Shared + Persistent

EBS
→ Block + Persistent

Instance Store
→ Local + Ephemeral
```

---

# 🔄 EFS Replication

EFS supports replication to another EFS file system, including cross-Region replication.

Useful for:

- Disaster recovery
- Business continuity
- Regional resilience

```text
Region A
  EFS
   │
   │ Replication
   ▼
Region B
  EFS
```

> [!warning]
> **Multi-AZ availability**
> → Regional EFS
>
> **Cross-Region disaster recovery**
> → EFS Replication

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1 — Shared Storage
> Multiple EC2 instances need simultaneous access to the same files.
>
> ✅ EFS

---

> [!danger] Trap 2 — Multi-AZ Shared Storage
> EC2 instances across multiple AZs need a highly available shared filesystem.
>
> ✅ Regional EFS

---

> [!danger] Trap 3 — NFS
> Application requires managed NFS storage.
>
> ✅ EFS

---

> [!danger] Trap 4 — NFS Port
> EC2 cannot connect to EFS.
>
> Check:
>
> ✅ TCP **2049**

---

> [!danger] Trap 5 — Infrequent Files
> Reduce cost for files accessed infrequently.
>
> ✅ EFS IA + Lifecycle Management

---

> [!danger] Trap 6 — Spiky Throughput
> Throughput is unpredictable and changes significantly.
>
> ✅ Elastic Throughput

---

> [!danger] Trap 7 — Known Throughput
> Application requires a specific throughput independent of filesystem size.
>
> ✅ Provisioned Throughput

---

> [!danger] Trap 8 — Different Application Directories
> Applications sharing EFS need isolated root directories / POSIX identities.
>
> ✅ EFS Access Points

---

> [!danger] Trap 9 — Shared vs Block
> Multiple EC2 instances need shared files.
>
> ❌ EBS
>
> ✅ EFS

---

# 🆚 Quick Comparison

| Requirement | Solution |
|---|---|
| Shared filesystem | EFS |
| Persistent EC2 block storage | EBS |
| Object storage | S3 |
| Temporary local EC2 storage | Instance Store |
| Multi-AZ shared filesystem | Regional EFS |
| Lower-cost single-AZ filesystem | EFS One Zone |
| Frequently accessed files | EFS Standard |
| Infrequently accessed files | EFS IA |
| Rarely accessed files | EFS Archive |
| Unpredictable throughput | Elastic Throughput |
| Known throughput requirement | Provisioned Throughput |
| Throughput based on storage size | Bursting Throughput |
| Application-specific EFS entry point | EFS Access Point |
| NFS connectivity | TCP 2049 |

---

# ⚡ EFS in 30 Seconds

```text
Amazon EFS
│
├── 📁 Managed File Storage
├── 🔌 NFS → TCP 2049
├── 👥 Shared by multiple clients
│
├── 🌎 Regional
│   └── Multi-AZ
│
├── 🏠 One Zone
│   └── Lower cost / single AZ
│
├── 💾 Storage Classes
│   ├── Standard
│   ├── IA
│   └── Archive
│
├── ⚡ Performance
│   └── General Purpose
│
├── 🚀 Throughput
│   ├── Elastic → unpredictable
│   ├── Provisioned → known requirement
│   └── Bursting → storage size
│
├── 🚪 Access Points
│   └── App-specific access
│
└── 🔐 Security
    ├── KMS at rest
    ├── TLS in transit
    └── Security Groups
```

> [!summary] SAA Memory Trick
> **Shared Files + Multiple EC2 → EFS**
>
> **EFS = NFS = TCP 2049**
>
> **Multi-AZ → Regional EFS**
>
> **Single AZ / cheaper → EFS One Zone**
>
> **Spiky throughput → Elastic**
>
> **Known throughput → Provisioned**
>
> **EC2 disk → EBS**
>
> **Objects → S3**