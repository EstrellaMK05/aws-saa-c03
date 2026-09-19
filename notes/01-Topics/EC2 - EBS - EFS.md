# EC2

Amazon EC2 provides virtual servers in AWS.

---

## EC2 Purchasing Options

### On-Demand
- No long-term commitment
- Pay for what you use
- Flexible, but more expensive for long-running predictable workloads

### Standard Reserved Instances
- Predictable, steady workloads
- 1 or 3-year commitment
- High discount
- Less flexible

💡 Predictable 24/7 workload + maximum discount → Standard RI

### Compute Savings Plans
- Commitment to a consistent amount of compute usage
- More flexible than Standard RI
- Can change instance family, size, OS, tenancy and Region

💡 Predictable usage + need flexibility → Compute Savings Plans

### Spot Instances
- Cheapest EC2 option
- AWS can interrupt the instance
- Good for fault-tolerant workloads

💡 Can be interrupted/restarted → Spot

⚠️ Critical workload that cannot be interrupted → NOT Spot

---

## Placement Groups

### Cluster Placement Group
- Instances placed close together
- Low network latency
- High network throughput
- Good for tightly coupled / HPC workloads

💡 Tightly coupled + lowest latency → Cluster

### Spread Placement Group
- Instances placed on distinct underlying hardware
- Reduces correlated hardware failure
- Good for a small number of critical instances

💡 Critical instances + isolate hardware failure → Spread

---

# Auto Scaling Group (ASG)

Automatically adds/removes EC2 instances based on demand.

## Target Tracking Scaling

Maintain a target metric.

Example:
`Average CPU ≈ 50%`

💡 Maintain CPU around X% → Target Tracking

## Scheduled Scaling

Scale at a known time.

💡 Every Friday at 6 PM → Scheduled Scaling

## Predictive Scaling

Uses historical patterns to forecast future demand.

💡 Recurring traffic pattern + forecast → Predictive Scaling

## Default Instance Warmup

Prevents new instances from affecting scaling metrics before they are ready.

💡 New instances take time to initialize → Instance Warmup

## Lifecycle Hooks

Pause an instance during launch or termination to perform custom actions.

💡 Need configuration before instance receives traffic → Lifecycle Hook

## Standby

Temporarily remove an instance from service without terminating it.

💡 Maintenance + keep instance in ASG → Standby

---

# EC2 Storage

## Instance Store

- Physical storage attached to the host
- Very high performance
- Temporary / ephemeral
- Data can be lost when the instance stops or terminates

💡 Ultra-fast TEMPORARY storage → Instance Store

⚠️ Need persistent data → EBS

---

# EBS

Persistent block storage for EC2.

## io2

- High and consistent IOPS
- Critical I/O-intensive workloads
- Persistent storage

💡 Database + high consistent IOPS → io2

## Move EBS Across AZs

EBS volumes are tied to an AZ.

To move:

`EBS → Snapshot → New EBS volume in target AZ`

💡 Snapshot is regional, EBS volume is AZ-specific.

## EBS Snapshots

- First snapshot → full baseline
- Subsequent snapshots → incremental (changed blocks)

## Encrypt Existing Unencrypted EBS

`Volume → Snapshot → Encrypted snapshot copy → New encrypted volume`

---

# EFS

Shared managed file system.

- Multiple EC2 instances can access it simultaneously
- Multi-AZ
- Automatically scales

💡 Multiple Linux EC2 instances need the SAME filesystem → EFS

---

# Load Balancers

## ALB — Application Load Balancer

Layer 7

- HTTP / HTTPS
- Path-based routing
- Host-based routing

Examples:
`/api/* → Target A`
`/images/* → Target B`

💡 HTTP/HTTPS + intelligent routing → ALB

## NLB — Network Load Balancer

Layer 4

- TCP / UDP / TLS
- Ultra-low latency
- Very high performance
- Static IP support

💡 TCP + ultra-low latency + static IP → NLB

---

# Health Checks

An EC2 instance can be healthy at the infrastructure level while the application is broken.

Use ELB health checks with an Auto Scaling Group to detect application failures.

💡 EC2 healthy but HTTP application unhealthy → ELB Health Check