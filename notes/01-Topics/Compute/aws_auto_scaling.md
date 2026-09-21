# 📈 AWS Auto Scaling

> [!summary] Mental Model
> **Auto Scaling = automatically adjust capacity based on demand.**
>
> - **Scale Out** → Add capacity
> - **Scale In** → Remove capacity
>
> 🎯 Goal: **Availability + Performance + Cost Optimization**

---

## 🖥️ EC2 Auto Scaling

An **Auto Scaling Group (ASG)** manages a collection of EC2 instances as a logical group.

### ASG Capacity

| Setting | Meaning |
|---|---|
| **Minimum** | Lowest number of instances |
| **Desired** | Number of instances the ASG tries to maintain |
| **Maximum** | Highest number of instances |

```text
Min = 2
Desired = 4
Max = 10

Traffic ↑
   ↓
Scale Out
   ↓
More EC2 instances
```

> [!important] Exam
> **Desired Capacity** = number of instances the ASG currently attempts to maintain.

---

# 🚀 Launch Template

Defines how EC2 instances in the ASG are created.

Can include:

- AMI
- Instance type
- Security Groups
- Key pair
- EBS configuration
- User Data
- IAM Instance Profile

```text
Launch Template
      ↓
Auto Scaling Group
      ↓
EC2   EC2   EC2
```

> [!tip]
> Prefer **Launch Templates** over legacy **Launch Configurations**.
>
> Launch Templates support **versions** and newer EC2 features.

---

# 📊 Scaling Policies

## 🎯 Target Tracking Scaling

Automatically adjusts capacity to maintain a metric around a target value.

Example:

```text
Target CPU = 50%

CPU = 80%
   ↓
Scale Out

CPU = 25%
   ↓
Scale In
```

Common metrics:

- Average CPU Utilization
- ALB Request Count Per Target
- Custom CloudWatch metrics

> [!tip] Mental Model
> **Target Tracking = Thermostat**
>
> "Keep CPU around 50%."

---

## 🪜 Step Scaling

Changes capacity depending on **how much a CloudWatch alarm threshold is exceeded**.

Example:

```text
CPU 60–70% → +1 instance
CPU 70–80% → +2 instances
CPU >80%   → +4 instances
```

> [!tip]
> **Step Scaling = bigger problem → bigger scaling action**

---

## ⏰ Scheduled Scaling

Changes capacity at a **known time**.

Example:

```text
Weekdays

08:45 → Scale Out
17:30 → Scale In
```

Best when traffic patterns are **known and predictable**.

> [!important] Exam Trap
> If the question gives an **exact predictable schedule**, think:
>
> **Scheduled Scaling**

Example:

```text
Traffic always increases at 9 AM
             ↓
Scale Out before 9 AM
             ↓
Scheduled Scaling
```

---

## 🔮 Predictive Scaling

Uses historical data to **forecast future demand** and proactively adjust capacity.

```text
Historical Load
      ↓
Forecast
      ↓
Prepare Capacity
      ↓
Expected Demand
```

> [!tip]
> **Predictive = forecast recurring demand**
>
> **Scheduled = schedule already known**

---

# 🆚 Scaling Policies

| Policy | Use When |
|---|---|
| 🎯 **Target Tracking** | Maintain a metric target |
| 🪜 **Step Scaling** | Different actions based on alarm severity |
| ⏰ **Scheduled Scaling** | Demand occurs at known times |
| 🔮 **Predictive Scaling** | Forecast recurring demand |

### 🧠 Exam Keywords

```text
"Keep CPU around 50%"
        ↓
🎯 Target Tracking

"CPU >70% add 2, >90% add 5"
        ↓
🪜 Step Scaling

"Every Monday at 8 AM"
        ↓
⏰ Scheduled Scaling

"Forecast recurring traffic"
        ↓
🔮 Predictive Scaling
```

---

# 🌡️ Instance Warmup

New instances may need time to:

- Boot
- Start the application
- Load dependencies
- Become ready to serve traffic

**Instance Warmup** prevents new instances from immediately affecting scaling decisions before they are ready.

```text
Launch EC2
    ↓
Boot Application
    ↓
Instance Warmup
    ↓
Ready
```

> [!warning] Don't Confuse
> **Instance Warmup**
> → Time for a new instance to become ready
>
> **Warm Pool**
> → Pre-initialized instances waiting for demand

---

# 🧊 Cooldown

A cooldown prevents repeated scaling actions from occurring too quickly after a previous scaling activity.

Especially associated with **Simple Scaling**.

> [!warning] Exam
> Do not automatically associate cooldown with every scaling policy.
>
> **Simple Scaling → Cooldown**
>
> Modern policies such as Target Tracking use their own mechanisms to avoid excessive scaling.

---

# ❤️ Health Checks

An ASG monitors instance health and can automatically replace unhealthy instances.

Health sources include:

- EC2 status checks
- Elastic Load Balancing health checks
- Custom health checks

```text
EC2 becomes unhealthy ❌
          ↓
ASG detects failure
          ↓
Terminate unhealthy EC2
          ↓
Launch replacement
          ↓
Desired Capacity restored
```

> [!important]
> Auto Scaling is not only about handling traffic.
>
> It also maintains the **desired number of healthy instances**.

---

# ⚖️ ASG + Load Balancer

Very common SAA architecture:

```text
               Internet
                  │
                  ▼
                 ALB
            ┌─────┼─────┐
            ▼     ▼     ▼
           EC2   EC2   EC2
            └──── ASG ───┘
             AZ-A    AZ-B
```

### Responsibilities

**ALB**
→ Distributes incoming requests.

**ASG**
→ Adjusts EC2 capacity and replaces unhealthy instances.

> [!tip] Exam Pattern
> **Highly available + elastic web application**
>
> → **ALB + Auto Scaling Group across multiple AZs**

---

# 🪝 Lifecycle Hooks

Lifecycle Hooks allow custom actions when instances are:

- Launching
- Terminating

### Launch Example

```text
ASG launches EC2
       ↓
Lifecycle Hook
       ↓
Install / Configure
       ↓
InService
```

### Termination Example

```text
Scale In
   ↓
Lifecycle Hook
   ↓
Finish Work
Copy Logs
Drain Connections
   ↓
Terminate
```

> [!tip]
> **Lifecycle Hook = pause the instance lifecycle to perform custom work**

### Typical Exam Scenario

> Need to copy logs to S3 before an EC2 instance is terminated.

✅ **Lifecycle Hook**

---

# ♨️ Warm Pools

A **Warm Pool** keeps pre-initialized instances available for applications with long startup times.

Without Warm Pool:

```text
Demand ↑
   ↓
Launch
   ↓
Boot
   ↓
Configure
   ↓
Ready 🐌
```

With Warm Pool:

```text
Pre-initialized EC2
        ↓
Demand ↑
        ↓
Ready ⚡
```

Useful when instance initialization takes a long time.

> [!warning] Don't Confuse
> **Warm Pool**
> → Pre-initialized capacity
>
> **Instance Warmup**
> → Time before new instances participate normally in scaling metrics

---

# 🛑 Standby State

An EC2 instance can temporarily be moved out of service:

```text
InService
    ↓
 Standby
    ↓
Maintenance
    ↓
InService
```

Useful for:

- Maintenance
- Troubleshooting
- Updates

> [!tip]
> Think:
>
> **Standby = temporarily remove an instance from the ASG workload without terminating it**

---

# 🛡️ Instance Scale-In Protection

Prevents specific instances from being terminated during automatic scale-in.

```text
EC2-A 🔒
EC2-B
EC2-C

Scale In
   ↓
B or C can terminate

A is protected
```

> [!tip]
> **Scale-In Protection**
>
> = "Do not terminate this instance during scale-in."

---

# 💀 Termination Policies

Determine **which EC2 instance** should be terminated when the ASG scales in.

```text
Need to Scale In
       ↓
Termination Policy
       ↓
Select EC2
       ↓
Terminate
```

> [!warning] Exam Trap
> **Scaling Policy**
> → WHEN capacity changes
>
> **Termination Policy**
> → WHICH instance is removed

---

## 🗑️ Default Termination Policy

When an Auto Scaling group scales in, the **default termination policy** determines which instance should be terminated while trying to keep the group balanced across Availability Zones.

### 🔄 Default Termination Flow

```text
Scale-In Triggered
       ↓
1️⃣ AZ with the MOST instances
   that has an instance not protected from scale-in
       ↓
   Tie between AZs?
       ↓
   Prefer AZ containing instances
   using the OLDEST launch template
       ↓
2️⃣ Within selected AZ:
   Find unprotected instances using
   the OLDEST launch template
       ↓
   Only one?
   → TERMINATE IT 🗑️
       ↓
3️⃣ Multiple candidates?
   → Choose instance closest to
     the NEXT BILLING HOUR
       ↓
4️⃣ Still multiple candidates?
   → RANDOM selection
```

> [!important] Exam Pattern
> **Auto Scaling + Scale-In + Default Termination Policy**
>
> Think in this order:
>
> **AZ Balance → Oldest Launch Template → Next Billing Hour → Random**

---

### 🧠 Memory Trick

```text
BALANCE
   ↓
OLDEST
   ↓
BILLING
   ↓
RANDOM
```

**B → O → B → R**

---

### Example

```text
AZ-A                    AZ-B
────────────────        ────────────────
EC2 → LT v1 🗑️         EC2 → LT v3
EC2 → LT v2             EC2 → LT v3
EC2 → LT v3
```

AZ-A has more instances.

Within AZ-A:

```text
LT v1 ← oldest
LT v2
LT v3
```

→ The instance using the **oldest launch template** is preferred for termination.

---

> [!warning] Don't Confuse

The default termination policy does NOT primarily select:

❌ Instance with the least user sessions  
❌ Instance running for the longest time  
❌ A random instance immediately

Random selection is only a **later tie-breaker** after the previous criteria.

---

### 🛡️ Scale-In Protection

Instances protected from scale-in are excluded from automatic scale-in termination.

```text
EC2
 ↓
Scale-In Protection = ON
 ↓
Not selected for automatic scale-in
```

---

### Scaling Policy vs Termination Policy

| Policy | Determines |
|---|---|
| **Scaling Policy** | **WHEN** capacity changes |
| **Termination Policy** | **WHICH** instance is terminated |
| **Scale-In Protection** | Which instances should **NOT** be terminated by scale-in |

> [!summary] SAA Memory
> **Default Scale-In:**
>
> **Balance AZs → Oldest Launch Template → Billing Hour → Random**

--- 

# ⚖️ Availability Zone Rebalancing

Auto Scaling attempts to maintain balanced capacity across enabled Availability Zones.

Example:

```text
AZ-A             AZ-B

EC2
EC2              EC2

        ↓

ASG launches capacity
in AZ-B to rebalance
```

🎯 Goal:

**Improve availability across AZs.**

---

# 🔄 Instance Refresh

Gradually replaces EC2 instances in an ASG.

Common scenario:

```text
New AMI
   ↓
Update Launch Template
   ↓
Instance Refresh
   ↓
Old EC2 → New EC2
Old EC2 → New EC2
Old EC2 → New EC2
```

Useful for rolling out:

- New AMIs
- Launch Template changes
- Application updates
- OS updates

> [!tip] Exam
> Need to replace an existing ASG fleet after changing the AMI?
>
> ✅ **Instance Refresh**

---

# 📦 Application Auto Scaling

Auto Scaling is **not limited to EC2**.

Application Auto Scaling can scale resources from services such as:

- ECS services
- DynamoDB provisioned capacity
- Aurora Replicas
- Lambda Provisioned Concurrency
- SageMaker resources

Common scaling policies include:

- Target Tracking
- Step Scaling
- Scheduled Scaling

> [!warning] Exam Trap
> **EC2 Auto Scaling**
> → EC2 instances / Auto Scaling Groups
>
> **Application Auto Scaling**
> → scalable resources from other AWS services

---

# 📊 Monitoring

## Amazon CloudWatch

Provides:

- Metrics
- Alarms
- Scaling triggers

```text
CloudWatch Metric
       ↓
CloudWatch Alarm
       ↓
Scaling Policy
       ↓
Auto Scaling Group
```

---

## Amazon SNS

Can send notifications about Auto Scaling events.

Examples:

- Instance launched
- Instance terminated
- Launch failure
- Termination failure

---

## AWS CloudTrail

Tracks Auto Scaling **API activity**.

> [!tip]
> **CloudWatch**
> → Metrics / Alarms
>
> **CloudTrail**
> → API activity

---

# 🧠 High-Value Exam Traps

> [!danger] Trap 1 — Predictable Traffic
> Traffic increases every day at a specific known time.
>
> ✅ **Scheduled Scaling**

---

> [!danger] Trap 2 — Maintain CPU
> Keep average CPU utilization around 50%.
>
> ✅ **Target Tracking**

---

> [!danger] Trap 3 — Different Scaling Amounts
> Add 2 instances at 70% CPU and 5 instances at 90%.
>
> ✅ **Step Scaling**

---

> [!danger] Trap 4 — Slow Startup
> EC2 instances require a long initialization process.
>
> ✅ **Warm Pool**

---

> [!danger] Trap 5 — Work Before Termination
> Copy logs or finish processing before EC2 terminates.
>
> ✅ **Lifecycle Hook**

---

> [!danger] Trap 6 — Protect an Instance
> A specific EC2 must not be removed during automatic scale-in.
>
> ✅ **Instance Scale-In Protection**

---

> [!danger] Trap 7 — New AMI
> Replace existing ASG instances after changing the AMI.
>
> ✅ **Instance Refresh**

---

> [!danger] Trap 8 — Highly Available Web Tier
> Application must automatically handle changing traffic and AZ failures.
>
> ✅ **ALB + ASG across multiple AZs**

---

> [!danger] Trap 9 — Unhealthy EC2
> An EC2 instance fails a health check.
>
> ✅ **ASG replaces the unhealthy instance**

---

# 🆚 Quick Comparisons

| Requirement | Solution |
|---|---|
| Maintain CPU around 50% | 🎯 Target Tracking |
| Scale differently based on severity | 🪜 Step Scaling |
| Scale every day at 9 AM | ⏰ Scheduled Scaling |
| Forecast recurring demand | 🔮 Predictive Scaling |
| Reduce slow EC2 startup | ♨️ Warm Pool |
| Run logic before termination | 🪝 Lifecycle Hook |
| Temporarily remove EC2 for maintenance | 🛑 Standby |
| Prevent EC2 termination during scale-in | 🛡️ Scale-In Protection |
| Choose which EC2 gets terminated | 💀 Termination Policy |
| Replace fleet with new AMI | 🔄 Instance Refresh |
| Distribute traffic | ⚖️ ALB |
| Automatically adjust EC2 capacity | 📈 ASG |

---

# ⚡ Auto Scaling in 30 Seconds

```text
EC2 Auto Scaling Group
│
├── Min / Desired / Max
│
├── 🚀 Launch Template
│
├── 📊 Scaling Policies
│   ├── 🎯 Target Tracking → maintain target
│   ├── 🪜 Step → severity-based scaling
│   ├── ⏰ Scheduled → known schedule
│   └── 🔮 Predictive → forecast demand
│
├── ❤️ Health Checks → replace unhealthy EC2
├── 🪝 Lifecycle Hooks → custom lifecycle actions
├── ♨️ Warm Pool → pre-initialized EC2
├── 🛑 Standby → maintenance
├── 🛡️ Scale-In Protection → don't terminate
├── 💀 Termination Policy → which EC2 dies
├── ⚖️ AZ Rebalancing → balance across AZs
└── 🔄 Instance Refresh → replace fleet gradually
```

---

> [!summary] SAA Memory Trick
> **WHEN should capacity change?**
> → 📊 Scaling Policy
>
> **WHICH instance should terminate?**
> → 💀 Termination Policy
>
> **WHAT should happen before launch/termination completes?**
> → 🪝 Lifecycle Hook
>
> **HOW do I avoid slow instance startup?**
> → ♨️ Warm Pool
>
> **HOW do I temporarily remove an instance?**
> → 🛑 Standby
>
> **HOW do I prevent a specific instance from scaling in?**
> → 🛡️ Scale-In Protection
>
> **HOW do I roll out a new AMI across the ASG?**
> → 🔄 Instance Refresh