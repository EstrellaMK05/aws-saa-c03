# ⏱️ RPO & RTO — Disaster Recovery

> [!summary] Mental Model
> **RPO = How much DATA can I lose?**
>
> **RTO = How much TIME can I be down?**

---

# 💾 RPO — Recovery Point Objective

**Maximum acceptable amount of data loss measured in time.**

```text
Last Recovery Point          Failure
        │                       │
        ▼                       ▼
────────●───────────────────────X──────
        │<------ RPO ---------->│
              Data Loss
```

Example:

```text
RPO = 1 hour
```

The business accepts losing up to **1 hour of data**.

```text
RPO = 1 second
```

The architecture must replicate data almost continuously.

> [!tip] Memory Trick
> **RPO → Point → DATA**
>
> "To what point in time can I recover?"

---

# ⏱️ RTO — Recovery Time Objective

**Maximum acceptable downtime after a failure.**

```text
Failure                    Service Restored
   │                              │
   ▼                              ▼
───X──────────────────────────────●────
   │<----------- RTO ------------>│
               Downtime
```

Example:

```text
RTO = 4 hours
```

The application can remain unavailable for up to **4 hours**.

```text
RTO < 1 minute
```

The architecture must recover extremely quickly.

> [!tip] Memory Trick
> **RTO → Time → DOWNTIME**
>
> "How quickly must I recover?"

---

# 🧠 RPO vs RTO

| | RPO | RTO |
|---|---|---|
| Measures | 💾 Data loss | ⏱️ Downtime |
| Question | How much data can I lose? | How long can I be down? |
| Lower value means | More frequent replication | Faster recovery |
| Example | RPO = 1 minute | RTO = 5 minutes |

### ⚡ Fast Recognition

```text
"Maximum acceptable data loss"
             ↓
            RPO

"Maximum acceptable downtime"
             ↓
            RTO
```

---

# 🌍 Example — Aurora Global Database

Requirement:

```text
Relational Database
+
Multi-Region DR
+
RPO ≈ seconds
+
RTO < minutes
```

Think:

👉 **Amazon Aurora Global Database**

```text
Primary Region
┌──────────────────┐
│ Aurora Cluster   │
│     Writer       │
└────────┬─────────┘
         │
         │ Cross-Region
         │ Replication
         ▼
Secondary Region
┌──────────────────┐
│ Aurora Cluster   │
│    Replicas      │
└──────────────────┘
```

If the primary Region fails:

```text
Region Failure ❌
      ↓
Secondary Region
      ↓
Promote / Failover
      ↓
Application Recovery
```

> [!important] Exam Pattern
> **Relational DB + multi-Region + very low RPO/RTO**
>
> → Think **Aurora Global Database**
>
> Standard **RDS Multi-AZ** provides high availability **within a Region**, not multi-Region disaster recovery.

---

# 🏗️ DR Strategies and RPO/RTO

General exam mental model:

```text
Cheaper / Slower Recovery
          │
          ▼
Backup & Restore
          ↓
Pilot Light
          ↓
Warm Standby
          ↓
Multi-Site / Active-Active
          │
          ▼
Expensive / Faster Recovery
```

| DR Strategy | RPO/RTO | Cost |
|---|---|---|
| 📦 Backup & Restore | Highest | $ |
| 🔥 Pilot Light | Lower | $$ |
| ♨️ Warm Standby | Low | $$$ |
| 🌎 Multi-Site / Active-Active | Lowest | $$$$ |

> [!warning]
> Do **not memorize exact RPO/RTO numbers** for these strategies unless the question provides them.
>
> Focus on the relationship:
>
> **Lower RPO/RTO → More infrastructure already running → Higher cost**

---

# 🆚 Pilot Light vs Warm Standby

## 🔥 Pilot Light

Only the **critical core components** are continuously running in the DR Region.

```text
Primary Region             DR Region

Full Application           Critical Core
EC2 EC2 EC2                Database ✅
Database                   Minimal Infra
ALB                        App capacity ❌
```

During disaster:

```text
Failure
  ↓
Start / Scale infrastructure
  ↓
Recover application
```

---

## ♨️ Warm Standby

A **complete but reduced-capacity environment** is already running.

```text
Primary Region             DR Region

Full Capacity              Reduced Capacity
██████████                 ███
```

During disaster:

```text
Failure
  ↓
Scale DR environment
  ↓
Redirect traffic
```

> [!tip] Exam
> **Pilot Light**
> → Only critical/core infrastructure running
>
> **Warm Standby**
> → Entire application running at reduced capacity

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> **"Maximum acceptable data loss"**
>
> → 💾 **RPO**

> [!danger] Trap 2
> **"Maximum acceptable downtime"**
>
> → ⏱️ **RTO**

> [!danger] Trap 3
> Very low RPO generally requires:
>
> → Frequent / continuous replication

> [!danger] Trap 4
> Very low RTO generally requires:
>
> → Infrastructure already available or quickly promotable

> [!danger] Trap 5
> **RDS Multi-AZ**
>
> → Regional High Availability
>
> ≠ Multi-Region Disaster Recovery

---

# ⚡ RPO & RTO in 10 Seconds

```text
RPO
 ↓
💾 DATA LOSS
"How far back?"

RTO
 ↓
⏱️ DOWNTIME
"How long to recover?"
```

> [!summary] SAA Memory Trick
> **RPO = DATA**
>
> **RTO = TIME**
>
> ↓ RPO → Less acceptable data loss
>
> ↓ RTO → Less acceptable downtime
>
> **Very low RPO + very low RTO**
> → More replication + more infrastructure ready
> → 💰 Higher cost