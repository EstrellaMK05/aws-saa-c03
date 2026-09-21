# 🛡️ AWS Shield

> [!summary] Mental Model
> **AWS Shield = DDoS Protection**
>
> Think:
>
> **DDoS / SYN Flood / UDP Reflection → AWS Shield**

---

# 🎯 Core Purpose

AWS Shield protects AWS applications against **Distributed Denial of Service (DDoS)** attacks.

```text
Attackers
   ↓↓↓↓↓
DDoS Traffic
   ↓
AWS Shield
   ↓
AWS Resource
```

Common DDoS attack types include:

- Network volumetric attacks
- UDP reflection attacks
- TCP SYN floods
- Application-layer request floods

---

# 🛡️ Shield Standard

AWS Shield Standard provides automatic protection against common DDoS attacks.

Key characteristics:

- Automatically enabled
- No additional charge
- Protects against common network and transport layer attacks
- No subscription required

```text
AWS Customer
     ↓
Shield Standard
     ↓
Automatic Basic DDoS Protection
```

> [!tip] Exam Pattern
> **Basic DDoS protection included automatically with AWS**
>
> → ✅ **Shield Standard**

---

# 🛡️ Shield Advanced

Shield Advanced provides **enhanced DDoS protection** for important or critical applications.

Think:

```text
Critical Application
        +
Advanced DDoS Protection
        ↓
AWS Shield Advanced
```

Key capabilities:

- Advanced DDoS detection and mitigation
- Greater visibility into DDoS events
- Protection for supported AWS resources
- Integration with AWS WAF
- AWS Shield Response Team (**SRT**) support
- Additional cost / subscription

> [!tip] Exam Pattern
> **Application is experiencing serious/frequent DDoS attacks**
>
> → ✅ **Shield Advanced**

---

# 🌐 Protected Resources

Shield Advanced can protect resources such as:

- Amazon EC2
- Elastic Load Balancing
- Amazon CloudFront
- Amazon Route 53
- AWS Global Accelerator

Example:

```text
Internet
   ↓
Shield Advanced
   ↓
CloudFront / ALB
   ↓
EC2
```

---

# 🚨 Common DDoS Attacks

## TCP SYN Flood

Attackers send large numbers of TCP SYN requests.

```text
Attacker
 ↓↓↓↓↓
SYN SYN SYN SYN SYN
 ↓
Target
 ↓
Connection resources exhausted
```

Think:

> **SYN Flood → DDoS → Shield**

---

## UDP Reflection Attack

Attackers abuse UDP services to generate large amounts of traffic toward the victim.

```text
Attackers
    ↓
UDP Reflection
    ↓↓↓↓↓
Victim
```

Think:

> **UDP Reflection → DDoS → Shield**

---

# 🆚 Shield Standard vs Shield Advanced

| | Shield Standard | Shield Advanced |
|---|---|---|
| DDoS Protection | ✅ | ✅ |
| Automatically included | ✅ | ❌ |
| Additional cost | ❌ | ✅ |
| Enhanced protection | ❌ | ✅ |
| Advanced visibility | ❌ | ✅ |
| Shield Response Team | ❌ | ✅ |

> [!tip] Memory Trick
> **Standard → Automatic basic protection**
>
> **Advanced → Critical workloads / enhanced DDoS protection**

---

# 🆚 Shield vs WAF

This is the **most important comparison for SAA**.

| AWS Shield | AWS WAF |
|---|---|
| DDoS protection | Web application firewall |
| Volumetric attacks | HTTP/HTTPS requests |
| SYN floods | SQL injection |
| UDP reflection | XSS |
| DDoS mitigation | Allow / Block / Count requests |

```text
DDoS
 ↓
SHIELD

Malicious HTTP Request
 ↓
WAF
```

> [!danger] Exam Trap
> **SYN Flood / UDP Reflection / DDoS**
>
> → ✅ Shield
>
> **SQL Injection / XSS**
>
> → ✅ WAF
>
> **Excessive HTTP requests**
>
> → ✅ WAF Rate-Based Rule

---

# 🤝 Shield + WAF

Shield and WAF can work together.

```text
Internet
   ↓
Shield
   │
   │ DDoS Protection
   ↓
WAF
   │
   │ HTTP/HTTPS Filtering
   ↓
CloudFront / ALB
   ↓
Application
```

Shield Advanced integrates with AWS WAF for application-layer DDoS protection.

> [!important]
> Shield and WAF are **complementary**, not mutually exclusive.

---

# 🆚 Shield vs Firewall Manager

```text
Shield
   ↓
Protect against DDoS

Firewall Manager
   ↓
Centrally manage security policies
across accounts/resources
```

> [!danger] Exam Trap
> **Prevent/mitigate DDoS**
>
> → Shield
>
> **Centrally manage WAF / Shield / security policies across AWS accounts**
>
> → Firewall Manager

---

# 🆘 Shield Response Team — SRT

Shield Advanced provides access to the **AWS Shield Response Team (SRT)** for assistance with DDoS events.

```text
DDoS Event
    ↓
Shield Advanced
    ↓
SRT
    ↓
AWS DDoS Experts
```

> [!tip] Exam Keyword
> **AWS experts helping during DDoS attacks**
>
> → Shield Advanced + **SRT**

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> Company is experiencing **DDoS attacks**.
>
> ✅ AWS Shield

> [!danger] Trap 2
> Company needs **enhanced DDoS protection** for a critical application.
>
> ✅ Shield Advanced

> [!danger] Trap 3
> Need protection against **TCP SYN floods or UDP reflection attacks**.
>
> ✅ Shield

> [!danger] Trap 4
> Need protection against **SQL injection or XSS**.
>
> ❌ Shield
>
> ✅ WAF

> [!danger] Trap 5
> Need to block clients making excessive HTTP requests.
>
> → WAF Rate-Based Rule

> [!danger] Trap 6
> Need centralized security policy management across multiple AWS accounts.
>
> → AWS Firewall Manager

---

# 🧠 Security Services Comparison

```text
Macie
→ Sensitive DATA

GuardDuty
→ THREATS / suspicious activity

Inspector
→ VULNERABILITIES

WAF
→ WEB REQUESTS

Shield
→ DDoS

Firewall Manager
→ CENTRALIZED SECURITY MANAGEMENT
```

| Requirement | Service |
|---|---|
| Find PII in S3 | Macie |
| Detect suspicious AWS activity | GuardDuty |
| Find software vulnerabilities | Inspector |
| Filter HTTP/HTTPS requests | WAF |
| Protect against DDoS | Shield |
| Centrally manage security policies | Firewall Manager |

---

# ⚡ Shield in 10 Seconds

```text
AWS Shield
    ↓
DDoS Protection
    │
    ├── Standard
    │    └── Automatic + no additional charge
    │
    └── Advanced
         ├── Enhanced protection
         ├── Advanced visibility
         ├── WAF integration
         └── SRT
```

> [!summary] SAA Memory Trick
> **DDoS → SHIELD**
>
> **Basic / automatic → Shield Standard**
>
> **Critical application / enhanced DDoS → Shield Advanced**
>
> **SQLi / XSS / HTTP filtering → WAF**
>
> **Multi-account security management → Firewall Manager**