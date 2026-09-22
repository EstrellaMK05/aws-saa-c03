# 📜 AWS Artifact

> [!summary] Mental Model
> **AWS Artifact = AWS Compliance Documents + Agreements**
>
> ```text
> Need AWS Compliance Evidence?
>           ↓
>      AWS Artifact
>       /        \
>   Reports     Agreements
> ```

---

# 🎯 Core Purpose

AWS Artifact provides on-demand access to AWS:

- Security reports
- Compliance reports
- Certifications
- Agreements

Common examples:

- SOC reports
- PCI reports
- ISO certifications
- Business Associate Addendum (BAA)

> [!tip] Exam Pattern
> **Download AWS compliance / audit documentation**
>
> → ✅ AWS Artifact

---

# 📄 Artifact Reports

Used to obtain AWS security and compliance documentation.

```text
Auditor / Compliance Team
           ↓
      AWS Artifact
           ↓
    Compliance Reports
```

Examples:

- SOC reports
- PCI reports
- ISO certifications

> [!tip] Exam Pattern
> **An auditor requests AWS SOC / PCI / ISO documentation**
>
> → ✅ AWS Artifact

---

# 🤝 Artifact Agreements

AWS Artifact can also be used to manage agreements with AWS.

You can:

- Review agreements
- Accept agreements
- Track agreements
- Terminate certain agreements

Example:

```text
Company handles PHI
       ↓
Needs AWS BAA
       ↓
AWS Artifact Agreements
```

> [!tip] Exam Pattern
> **Review or accept a Business Associate Addendum (BAA)**
>
> → ✅ AWS Artifact Agreements

---

# 🏢 AWS Organizations Integration

Artifact agreements can be managed for multiple accounts in an AWS Organization.

```text
Management Account
       ↓
AWS Artifact Agreement
       ↓
AWS Organization
├── Account A
├── Account B
└── Account C
```

> [!tip] Exam Pattern
> **Compliance agreement covering multiple AWS accounts**
>
> → AWS Artifact + AWS Organizations

---

# 💰 Pricing

AWS Artifact reports and agreements are available at no additional charge.

---

# 🆚 Artifact vs CloudTrail

```text
AWS Artifact
→ COMPLIANCE DOCUMENTS

AWS CloudTrail
→ API ACTIVITY / AUDIT LOGS
```

Examples:

```text
"Who deleted this S3 bucket?"
→ CloudTrail

"Give the auditor the AWS SOC report"
→ Artifact
```

> [!summary]
> **Artifact = DOCUMENTS**
>
> **CloudTrail = ACTIVITY**

---

# 🆚 Artifact vs AWS Config

```text
AWS Artifact
→ AWS compliance reports / certifications

AWS Config
→ Resource configuration / compliance
```

Examples:

```text
"Obtain AWS PCI compliance documentation"
→ Artifact

"Are our S3 buckets compliant with our configuration rules?"
→ AWS Config
```

> [!summary]
> **Artifact = AWS COMPLIANCE DOCUMENTATION**
>
> **Config = YOUR RESOURCE COMPLIANCE**

---

# 🆚 Artifact vs Audit Manager

```text
AWS Artifact
→ Obtain AWS compliance documents

AWS Audit Manager
→ Collect and organize evidence
  for your audits
```

> [!danger] Exam Trap
> **Download AWS SOC / PCI / ISO reports**
>
> → AWS Artifact
>
> **Automate evidence collection for your own audit**
>
> → AWS Audit Manager

---

# 🆚 Artifact vs Security Hub

```text
AWS Artifact
→ Compliance documentation

AWS Security Hub
→ Security findings and security posture
```

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> **Auditor requests AWS SOC report**
>
> → AWS Artifact

> [!danger] Trap 2
> **Download PCI / ISO compliance documentation**
>
> → AWS Artifact

> [!danger] Trap 3
> **Accept or review a BAA**
>
> → AWS Artifact Agreements

> [!danger] Trap 4
> **Track AWS API calls**
>
> → CloudTrail
>
> NOT Artifact.

> [!danger] Trap 5
> **Evaluate resource configuration compliance**
>
> → AWS Config
>
> NOT Artifact.

> [!danger] Trap 6
> **Automated evidence collection for audits**
>
> → AWS Audit Manager
>
> NOT Artifact.

> [!danger] Trap 7
> AWS Artifact does **not** automatically make your workloads compliant.
>
> It provides AWS compliance documentation and agreements.

---

# 🧠 AWS Artifact in 20 Seconds

```text
AWS COMPLIANCE DOCUMENTS
→ Artifact

SOC REPORT
→ Artifact

PCI REPORT
→ Artifact

ISO CERTIFICATION
→ Artifact

BAA / AGREEMENTS
→ Artifact Agreements

API AUDIT LOGS
→ CloudTrail

RESOURCE COMPLIANCE
→ AWS Config

AUTOMATED AUDIT EVIDENCE
→ Audit Manager
```

> [!summary] SAA Memory
> **ARTIFACT = DOCUMENTS**
>
> **CLOUDTRAIL = ACTIVITY**
>
> **CONFIG = CONFIGURATION COMPLIANCE**
>
> **AUDIT MANAGER = AUDIT EVIDENCE**