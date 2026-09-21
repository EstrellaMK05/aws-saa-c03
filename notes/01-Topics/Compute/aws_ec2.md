# 🖥️ Amazon EC2

> [!summary] Mental Model
> **Amazon EC2 = resizable virtual servers in AWS.**
>
> You choose:
>
> - 🧠 Compute
> - 💾 Storage
> - 🌐 Networking
> - 🔐 Security
> - 💰 Purchasing model
>
> EC2 gives you **full control of the operating system**.

---

# 🏗️ EC2 Basics

An EC2 instance is a virtual machine running inside an AWS Availability Zone.

```text
Region
│
├── AZ-A
│   ├── EC2
│   └── EC2
│
└── AZ-B
    ├── EC2
    └── EC2
```

When launching an instance, you typically choose:

- AMI
- Instance type
- VPC / Subnet
- Security Groups
- Storage
- IAM Role
- User Data
- Purchasing option

---

# 📀 Amazon Machine Image — AMI

An **AMI** is a template used to launch EC2 instances.

Contains information such as:

- Operating System
- Installed software
- Configuration
- Block device mapping

```text
AMI
 ↓
Launch
 ↓
EC2 Instance
```

You can create custom AMIs from configured EC2 instances.

> [!tip] Exam Pattern
> Need to launch many EC2 instances with the **same OS and preinstalled software**?
>
> ✅ Create a **custom AMI**

---

# 🧬 Instance Types

General naming:

```text
m7i.large
│ │   │
│ │   └── Size
│ └────── Generation / attributes
└──────── Instance family
```

## Common Families

| Family | Optimized For |
|---|---|
| **T** | Burstable workloads |
| **M** | General purpose |
| **C** | Compute intensive |
| **R** | Memory intensive |
| **I** | Storage / high IOPS |
| **G / P** | GPU workloads |

### 🧠 Exam Keywords

```text
Balanced workload
      ↓
M family

CPU intensive
      ↓
C family

Large in-memory dataset
      ↓
R family

Machine Learning / GPU
      ↓
P / G family
```

---

# 💰 EC2 Purchasing Options

## 💵 On-Demand

Pay for compute capacity without long-term commitment.

Best for:

- Short-term workloads
- Unpredictable workloads
- Testing
- Applications that cannot be interrupted

```text
Use EC2
  ↓
Pay as you go
```

> [!tip]
> **Unpredictable + short-term + no commitment**
>
> → On-Demand

---

# 💳 Savings Plans

Commit to a consistent amount of compute usage for **1 or 3 years** in exchange for lower prices.

Useful for predictable long-term compute usage.

### Compute Savings Plans

More flexible.

Can apply across eligible:

- EC2 instance families
- Regions
- Operating systems
- AWS Fargate
- AWS Lambda

### EC2 Instance Savings Plans

Less flexible but can provide greater savings.

Commitment is tied more closely to:

- Instance family
- Region

> [!tip]
> **Need flexibility across compute services**
>
> → Compute Savings Plans

---

# 🏦 Reserved Instances

Provide discounts compared with On-Demand pricing in exchange for a **1-year or 3-year commitment**.

Best for:

- Stable workloads
- Predictable usage
- Long-running EC2 instances

Types include:

- Standard Reserved Instances
- Convertible Reserved Instances

> [!warning]
> Reserved Instances are primarily a **billing discount model**.
>
> They do not automatically create or reserve a running EC2 instance.

---

# 🎟️ Capacity Reservations

Reserve EC2 compute capacity in a specific Availability Zone.

```text
Capacity Reservation
        ↓
AZ-A capacity guaranteed
```

Useful when:

> "The company must guarantee EC2 capacity in a specific AZ."

> [!danger] Don't Confuse
> **Reserved Instance**
> → 💰 Pricing discount
>
> **Capacity Reservation**
> → 🖥️ Capacity guarantee

---

# 💸 Spot Instances

Use spare EC2 capacity at a large discount.

AWS can interrupt the instance when capacity is needed.

Best for:

- Batch processing
- Distributed workloads
- CI/CD
- Data processing
- Fault-tolerant workloads
- Stateless applications

Not ideal for:

- Critical databases
- Workloads that cannot tolerate interruption

```text
Spare AWS Capacity
       ↓
Spot Instance
       ↓
Cheap 💰
       ↓
Can be interrupted ⚠️
```

> [!danger] Exam
> **Fault tolerant + flexible + cheapest EC2**
>
> → Spot Instances

---

# 🆚 Purchasing Models

| Requirement | Think |
|---|---|
| Short / unpredictable workload | On-Demand |
| Stable long-term usage | Savings Plans / RI |
| Cheapest fault-tolerant compute | Spot |
| Guarantee capacity in an AZ | Capacity Reservation |

---

# 🚀 Spot Fleet

A Spot Fleet can launch capacity across different:

- Instance types
- Availability Zones
- Purchase options

Goal:

```text
Required Capacity
       ↓
Multiple EC2 pools
       ↓
Optimize price / availability
```

Useful for flexible, fault-tolerant workloads.

---

# 📍 Placement Groups

Control how EC2 instances are physically placed.

Three strategies:

```text
Cluster
Spread
Partition
```

---

## ⚡ Cluster Placement Group

Places instances **close together** inside a single AZ.

```text
AZ

EC2 EC2 EC2 EC2
 ↔   ↔   ↔
High-speed network
```

Best for:

- HPC
- Low network latency
- High network throughput

> [!tip]
> **Lowest latency between EC2 instances**
>
> → Cluster Placement Group

### Tradeoff

Instances are close together, so failure isolation is lower.

---

## 🛡️ Spread Placement Group

Places instances on **distinct underlying hardware**.

```text
EC2      EC2      EC2
 │        │        │
Rack A   Rack B   Rack C
```

Best for:

- Small number of critical instances
- Maximum failure isolation

> [!tip]
> **Critical EC2 instances must not share underlying hardware**
>
> → Spread Placement Group

---

## 🧱 Partition Placement Group

Divides instances into logical partitions.

Instances in different partitions do not share the same underlying hardware.

```text
Partition 1    Partition 2    Partition 3

EC2 EC2        EC2 EC2        EC2 EC2
```

Best for:

- Large distributed systems
- Hadoop
- Cassandra
- HBase

> [!tip]
> **Large distributed workload + failure isolation**
>
> → Partition Placement Group

---

# 🧠 Placement Group Memory Trick

```text
CLUSTER
→ CLOSE
→ Performance

SPREAD
→ SEPARATE
→ Maximum isolation

PARTITION
→ GROUPS
→ Large distributed systems
```

---

# 🌐 Elastic Network Interface — ENI

An ENI is a virtual network interface attached to an EC2 instance.

Can contain:

- Private IPv4 addresses
- Public IPv4 address
- Elastic IP
- IPv6 addresses
- Security Groups
- MAC address

```text
EC2
 │
 └── ENI
      ├── Private IP
      ├── Security Groups
      └── MAC Address
```

Additional ENIs can be attached to supported EC2 instances.

> [!tip] Exam Pattern
> Need to move a **network interface/private IP** between EC2 instances?
>
> → Think **ENI**

---

# 🌍 Public IP vs Elastic IP

## Public IPv4

Can change when an instance is stopped and started.

## Elastic IP

Static public IPv4 address associated with your AWS account.

```text
Elastic IP
    ↓
Static Public IPv4
```

> [!warning]
> Avoid depending heavily on Elastic IPs for scalable architectures.
>
> Prefer services such as:
>
> - Load Balancers
> - Route 53

---

# 🔐 Security Groups

Security Groups act as **stateful virtual firewalls** for EC2 networking.

```text
Internet
   ↓
Security Group
   ↓
EC2
```

Characteristics:

- Stateful
- Allow rules only
- Applied to ENIs

> [!tip]
> **Security Group = instance/ENI-level network security**
>
> **NACL = subnet-level network security**

---

# 🎭 IAM Roles for EC2

Applications running on EC2 should use IAM Roles to access AWS services.

```text
EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
S3 / DynamoDB / SQS / etc.
```

> [!danger] Exam Trap
> EC2 application needs AWS API access:
>
> ❌ Hard-code access keys
>
> ❌ Store access keys in source code
>
> ✅ Attach an **IAM Role**

---

# 📜 User Data

EC2 User Data can run scripts when an instance launches.

Common uses:

- Install software
- Configure applications
- Download dependencies
- Start services

Example:

```text
Launch EC2
    ↓
User Data
    ↓
Install Web Server
    ↓
Start Application
```

> [!tip]
> **Bootstrap EC2 at launch**
>
> → User Data

---

# 🪪 Instance Metadata

EC2 instances can access information about themselves through the **Instance Metadata Service (IMDS)**.

Examples:

- Instance ID
- Networking information
- IAM role credentials
- Other instance metadata

```text
Application
    ↓
IMDS
    ↓
Instance Metadata
```

---

# 🔐 IMDSv2

IMDSv2 uses a **session-oriented token**.

It provides stronger protection than IMDSv1.

```text
Request Token
     ↓
Receive Token
     ↓
Request Metadata
using Token
```

> [!important] Exam
> Need stronger protection against unauthorized access to instance metadata?
>
> → **Require IMDSv2**

---

# ⏯️ EC2 Instance States

Common states:

```text
Pending
   ↓
Running
   ↓
Stopping
   ↓
Stopped
   ↓
Terminated
```

---

# 🛑 Stop vs Terminate

## Stop

Instance can be started again.

Typically:

```text
EC2 stopped
   ↓
EBS persists
```

Compute charges stop, although attached resources such as EBS can still incur charges.

---

## Terminate

Deletes the EC2 instance.

Root EBS behavior depends on the:

```text
DeleteOnTermination
```

setting.

> [!warning]
> **Stop ≠ Terminate**

---

# 😴 EC2 Hibernation

Hibernation preserves the contents of RAM.

```text
Running EC2
    ↓
Hibernate
    ↓
RAM saved to encrypted EBS
    ↓
Start
    ↓
RAM restored
```

Useful when:

- Application startup takes a long time
- In-memory state should be preserved

> [!tip]
> **Need to preserve RAM across stop/start**
>
> → EC2 Hibernation

---

# 🩺 EC2 Status Checks

EC2 performs status checks to identify problems.

## System Status Check

Checks AWS infrastructure supporting the instance.

Examples:

- Physical host problems
- Network problems
- Power problems

Mental model:

```text
SYSTEM check
→ AWS problem
```

---

## Instance Status Check

Checks the operating system / instance.

Examples:

- OS configuration
- Memory exhaustion
- File system issues
- Networking configuration

Mental model:

```text
INSTANCE check
→ Your VM / OS problem
```

---

# 🔧 Recovering an EC2 Instance

For infrastructure-related failures, EC2 recovery can move the instance to healthy hardware.

```text
Underlying Host Failure
        ↓
EC2 Recovery
        ↓
Healthy Hardware
```

> [!tip]
> **System Status Check failure**
>
> → Think AWS infrastructure / recovery
>
> **Instance Status Check failure**
>
> → Think OS / instance troubleshooting

---

# 📊 Monitoring

Amazon CloudWatch provides EC2 metrics.

Common metrics:

- CPU utilization
- Network
- Disk operations
- Status checks

> [!warning] Exam Trap
> EC2 does **not provide OS memory utilization as a standard CloudWatch metric**.
>
> For metrics such as:
>
> - Memory utilization
> - Disk space utilization
>
> → Install/configure the **CloudWatch Agent**

---

# 💾 EC2 Storage

EC2 commonly works with:

```text
EC2
├── EBS
├── Instance Store
└── EFS
```

## EBS

Persistent block storage.

```text
EC2 stopped
    ↓
EBS data persists
```

## Instance Store

Temporary storage physically attached to the host.

```text
Very Fast
   +
Ephemeral
```

Data can be lost when the instance is stopped, terminated, or underlying hardware fails.

> [!danger] Exam
> **Temporary + very high-performance local storage**
>
> → Instance Store
>
> **Persistent block storage**
>
> → EBS

---

# 🏷️ Dedicated Instances vs Dedicated Hosts

## Dedicated Instance

Instance runs on hardware dedicated to one AWS account.

Less control over the physical host.

## Dedicated Host

Entire physical server dedicated to your account.

Provides visibility/control over host placement.

Useful for:

- Licensing requirements
- Compliance
- Server-bound software licenses

> [!tip]
> **Need physical server visibility/control or BYOL tied to sockets/cores**
>
> → Dedicated Host

---

# 🆚 EC2 vs Lambda vs ECS/Fargate

| Requirement | Think |
|---|---|
| Full OS control | EC2 |
| Long-running VM workload | EC2 |
| Event-driven short execution | Lambda |
| Containers | ECS/EKS |
| Containers without managing servers | Fargate |

---

# 🧠 High-Value Exam Traps

> [!danger] Trap 1 — Cheapest Fault-Tolerant Compute
> Workload can tolerate interruptions.
>
> ✅ **Spot Instances**

---

> [!danger] Trap 2 — Guaranteed Capacity
> Must guarantee EC2 capacity in a specific AZ.
>
> ✅ **Capacity Reservation**

---

> [!danger] Trap 3 — High Network Performance
> HPC instances require extremely low latency between each other.
>
> ✅ **Cluster Placement Group**

---

> [!danger] Trap 4 — Maximum Isolation
> Small number of critical EC2 instances must use separate hardware.
>
> ✅ **Spread Placement Group**

---

> [!danger] Trap 5 — Distributed System
> Large Hadoop/Cassandra cluster requires hardware failure isolation.
>
> ✅ **Partition Placement Group**

---

> [!danger] Trap 6 — Bootstrap
> Install/configure software when EC2 launches.
>
> ✅ **User Data**

---

> [!danger] Trap 7 — AWS Credentials
> EC2 application needs access to S3.
>
> ✅ **IAM Role**
>
> ❌ Access keys in code

---

> [!danger] Trap 8 — Preserve RAM
> Application must resume with previous in-memory state.
>
> ✅ **EC2 Hibernation**

---

> [!danger] Trap 9 — Memory Monitoring
> Need RAM utilization from EC2.
>
> ✅ **CloudWatch Agent**

---

> [!danger] Trap 10 — Static Public IPv4
> EC2 needs a persistent public IPv4 address.
>
> ✅ **Elastic IP**

---

> [!danger] Trap 11 — Metadata Security
> Need stronger protection for EC2 Instance Metadata.
>
> ✅ **IMDSv2**

---

> [!danger] Trap 12 — Host Licensing
> Software license is tied to physical cores/sockets.
>
> ✅ **Dedicated Host**

---

# 🆚 Quick Comparison

| Requirement | Solution |
|---|---|
| Short unpredictable workload | On-Demand |
| Stable long-term workload | Savings Plans / RI |
| Fault-tolerant cheapest compute | Spot |
| Guarantee AZ capacity | Capacity Reservation |
| High-performance networking | Cluster Placement Group |
| Maximum hardware isolation | Spread Placement Group |
| Large distributed cluster | Partition Placement Group |
| Bootstrap instance | User Data |
| EC2 → AWS permissions | IAM Role |
| Preserve RAM | Hibernation |
| Static public IPv4 | Elastic IP |
| OS memory metrics | CloudWatch Agent |
| Physical host control | Dedicated Host |
| Secure instance metadata | IMDSv2 |

---

# ⚡ EC2 in 30 Seconds

```text
Amazon EC2
│
├── 📀 AMI → instance template
├── 🧬 Instance Type → CPU / RAM / network
│
├── 💰 Pricing
│   ├── On-Demand → flexible
│   ├── Savings Plans / RI → predictable
│   ├── Spot → cheap + interruptible
│   └── Capacity Reservation → guaranteed capacity
│
├── 📍 Placement Groups
│   ├── Cluster → performance
│   ├── Spread → isolation
│   └── Partition → distributed systems
│
├── 🌐 ENI → network interface
├── 🔐 Security Group → stateful firewall
├── 🎭 IAM Role → AWS permissions
├── 📜 User Data → bootstrap
├── 🪪 IMDSv2 → secure metadata
├── 😴 Hibernation → preserve RAM
└── 📊 CloudWatch Agent → OS metrics
```

---

> [!summary] SAA Memory Trick
> **Need cheap interruptible compute?**
> → 💸 Spot
>
> **Need guaranteed EC2 capacity?**
> → 🎟️ Capacity Reservation
>
> **Need instances physically close?**
> → ⚡ Cluster
>
> **Need instances physically separated?**
> → 🛡️ Spread
>
> **Need large distributed failure domains?**
> → 🧱 Partition
>
> **Need software installed at launch?**
> → 📜 User Data
>
> **Need AWS permissions from EC2?**
> → 🎭 IAM Role
>
> **Need RAM preserved?**
> → 😴 Hibernation
>
> **Need memory/disk-space metrics?**
> → 📊 CloudWatch Agent