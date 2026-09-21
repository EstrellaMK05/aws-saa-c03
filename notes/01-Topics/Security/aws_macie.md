# 🔎 Amazon Macie

> [!summary] Mental Model
> **Amazon Macie = Discover sensitive data in Amazon S3**
>
> Think:
>
> **S3 + Sensitive Data / PII / PHI / Credentials → Macie**

---

# 🎯 Core Purpose

Amazon Macie is a **fully managed data security and data privacy service** focused on discovering sensitive data stored in **Amazon S3**.

```text
Amazon S3
    ↓
Amazon Macie
    ↓
Analyze Objects
    ↓
Sensitive Data Discovery
    ↓
Findings
```

Macie can identify sensitive information such as:

- 👤 Personally Identifiable Information (**PII**)
- 🏥 Personal Health Information (**PHI**)
- 💳 Financial information
- 🔑 Credentials and private keys
- 📄 Other sensitive data

> [!tip] Exam Pattern
> **"Discover sensitive data stored in S3"**
>
> → ✅ **Amazon Macie**

---

# 🔍 Sensitive Data Discovery

Macie analyzes objects stored in S3 to identify sensitive information.

It uses techniques including:

- Machine learning
- Pattern matching
- Managed Data Identifiers
- Custom Data Identifiers

---

# 🧠 Managed Data Identifiers

Built-in detection criteria provided by AWS.

Examples include:

```text
Credit Card Numbers
Passport Numbers
AWS Secret Access Keys
Private Keys
Personal Information
Financial Information
```

```text
S3 Object
   ↓
Managed Data Identifier
   ↓
Sensitive Data Detected
```

> [!tip]
> AWS already knows the sensitive data pattern?
>
> → **Managed Data Identifier**

---

# 🛠️ Custom Data Identifiers

Used when an organization needs to detect its **own specific sensitive data patterns**.

They can use:

- Regular expressions (**regex**)
- Keywords
- Ignore words
- Proximity rules

Example:

```text
Company Employee ID

EMP-123456
EMP-982341
```

You can create a custom identifier to detect that pattern.

> [!tip] Exam Pattern
> Need to detect an **organization-specific sensitive data format**?
>
> → ✅ **Custom Data Identifier**

---

# 🤖 Automated Sensitive Data Discovery

Macie can automatically evaluate S3 data to identify sensitive information without requiring you to manually create individual discovery jobs for every analysis.

```text
S3 Data Estate
      ↓
Automated Discovery
      ↓
Macie analyzes samples
      ↓
Sensitivity information
```

---

# 📋 Sensitive Data Discovery Jobs

You can also create explicit discovery jobs.

A job defines things such as:

- Which S3 buckets to analyze
- Which objects to include
- Which identifiers to use
- When the analysis runs

Jobs can run:

- Once
- Daily
- Weekly
- Monthly

> [!tip]
> Need controlled or scheduled analysis of specific S3 buckets?
>
> → **Sensitive Data Discovery Job**

---

# 🪣 S3 Security Posture

Macie also helps provide visibility into the security and privacy posture of your S3 data.

Mental model:

```text
Macie
│
├── 🔎 Sensitive Data Discovery
└── 🪣 S3 Data Security / Privacy Visibility
```

---

# 🆚 Macie vs GuardDuty

This is an important exam distinction.

| Amazon Macie | Amazon GuardDuty |
|---|---|
| Sensitive data | Threat detection |
| Data privacy | Malicious/suspicious activity |
| Strong focus on S3 data | AWS account/workload threat signals |
| PII / PHI / credentials | Compromised credentials / suspicious behavior |

```text
"Where is my sensitive data?"
          ↓
        Macie

"Is something malicious happening?"
          ↓
      GuardDuty
```

> [!danger] Exam Trap
> **Detect PII in S3**
>
> → Macie
>
> **Detect suspicious or malicious AWS activity**
>
> → GuardDuty

---

# 🆚 Macie vs Inspector

```text
Macie
→ Sensitive DATA

Inspector
→ VULNERABILITIES
```

| Requirement | Service |
|---|---|
| Find PII in S3 | Macie |
| Find sensitive financial data in S3 | Macie |
| Find EC2 vulnerabilities | Inspector |
| Find vulnerable container packages | Inspector |

---

# 🆚 Macie vs GuardDuty vs Inspector

> [!important] SAA Memory Trick
>
> **Macie → DATA**
>
> **GuardDuty → THREATS**
>
> **Inspector → VULNERABILITIES**

```text
Sensitive Data
     ↓
   MACIE

Suspicious Activity
     ↓
  GUARDDUTY

Software Vulnerability
     ↓
  INSPECTOR
```

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> Company needs to discover **PII stored in S3**.
>
> ✅ Amazon Macie

> [!danger] Trap 2
> Company needs to discover **credit card numbers in S3 objects**.
>
> ✅ Amazon Macie

> [!danger] Trap 3
> Company needs to detect a proprietary data format such as internal customer IDs.
>
> ✅ Macie + **Custom Data Identifier**

> [!danger] Trap 4
> Company needs to detect compromised credentials or malicious AWS activity.
>
> ❌ Macie
>
> ✅ GuardDuty

> [!danger] Trap 5
> Company needs to scan EC2 or container workloads for software vulnerabilities.
>
> ❌ Macie
>
> ✅ Inspector

---

# ⚡ Macie in 10 Seconds

```text
Amazon Macie
     ↓
Sensitive Data Discovery
     ↓
Amazon S3
     ↓
PII
PHI
Financial Data
Credentials
```

> [!summary] SAA Memory Trick
> **S3 + Sensitive Data = MACIE**
>
> **PII / PHI / Credit Cards → Macie**
>
> **Custom company data pattern → Custom Data Identifier**
>
> **Threats → GuardDuty**
>
> **Vulnerabilities → Inspector**