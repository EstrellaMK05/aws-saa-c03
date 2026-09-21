# 🔐 AWS Identity and Access Management (IAM)

> [!summary] Mental Model
> **IAM = WHO can do WHAT on WHICH AWS resource, under WHICH conditions.**
>
> ```text
> Principal
>    ↓
> Authentication
>    ↓
> IAM Policies
>    ↓
> Authorization
>    ↓
> AWS Resource
> ```
>
> 🎯 Core principle: **Least Privilege**

---

# 🌎 IAM Basics

AWS IAM is a **global service** used to securely control access to AWS resources.

IAM controls:

- **Authentication** → Who are you?
- **Authorization** → What are you allowed to do?

> [!important]
> IAM is **global**, not regional.

---

# 👤 IAM Identities

## 👑 Root User

The root user is created when the AWS account is created.

It has unrestricted access to the account.

Best practices:

- Enable MFA
- Do not use for daily tasks
- Do not create/use root access keys
- Use IAM identities for normal administration

> [!danger] Exam
> **Root User = maximum privileges**
>
> Protect it with **MFA** and avoid using it for everyday operations.

---

## 👤 IAM Users

Represents an individual identity inside an AWS account.

Can have:

- Console password
- Access keys
- IAM policies
- MFA

```text
IAM User
   │
   ├── Console Password
   └── Access Key + Secret Key
```

A newly created IAM user has:

```text
NO permissions by default
```

> [!warning]
> For workforce/human access across AWS accounts, prefer:
>
> **IAM Identity Center**
>
> rather than creating many IAM users.

---

# 👥 IAM Groups

A group is a collection of IAM users.

```text
Developers Group
       │
       ├── Alice
       ├── Bob
       └── Charlie
```

Attach policies to the group:

```text
Developers
    ↓
Policy
    ↓
Permissions inherited by users
```

> [!important]
> Groups:
>
> - Contain **users**
> - Cannot contain other groups
> - Are **not identities**
> - Cannot be assumed
> - Cannot log in

---

# 🎭 IAM Roles

An IAM Role is an identity with permissions but **without long-term credentials**.

A principal assumes the role and receives **temporary credentials**.

```text
Principal
   ↓
AssumeRole
   ↓
IAM Role
   ↓
Temporary Credentials
   ↓
AWS Resource
```

Roles are commonly used by:

- EC2
- Lambda
- ECS
- AWS services
- Cross-account users
- Federated users

> [!tip] Exam
> Application running on AWS needs AWS credentials?
>
> ❌ Hard-code access keys
>
> ✅ **Use an IAM Role**

---

# 🆚 User vs Group vs Role

| IAM Entity | Purpose |
|---|---|
| 👤 User | Individual identity |
| 👥 Group | Collection of users |
| 🎭 Role | Temporary assumed identity |
| 👑 Root | Full AWS account identity |

### 🧠 Memory Trick

```text
User
→ WHO you are

Group
→ ORGANIZE users

Role
→ WHO you temporarily become
```

---

# 🤖 Service Roles

AWS services can assume IAM roles to perform actions on your behalf.

Example:

```text
Lambda
   ↓
Execution Role
   ↓
S3 / DynamoDB / CloudWatch
```

Example:

```text
EC2
 ↓
IAM Role
 ↓
S3
```

> [!tip]
> **Compute service needs AWS permissions**
>
> → IAM Role

---

# 🔗 Service-Linked Roles

A **Service-Linked Role** is predefined for a specific AWS service.

```text
AWS Service
     ↓
Service-Linked Role
     ↓
Actions required by service
```

AWS defines the permissions needed by that service.

---

# 📜 IAM Policies

Policies are JSON documents that define permissions.

Basic structure:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::example-bucket/*"
}
```

Important elements:

| Element | Meaning |
|---|---|
| `Effect` | Allow or Deny |
| `Action` | AWS API operation |
| `Resource` | Target resource |
| `Condition` | Optional restrictions |
| `Principal` | Who receives access in resource-based policies |

---

# 👤 Identity-Based Policies

Attached to:

- Users
- Groups
- Roles

They answer:

> **What can this identity do?**

```text
IAM Role
   ↓
Identity Policy
   ↓
Allow s3:GetObject
```

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

---

# 🪣 Resource-Based Policies

Attached directly to a resource.

Examples:

- S3 Bucket Policy
- SQS Queue Policy
- KMS Key Policy
- SNS Topic Policy
- Lambda Resource Policy

They answer:

> **Who can access this resource?**

```text
Principal
    ↓
Resource Policy
    ↓
S3 Bucket
```

Example:

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/AppRole"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

> [!tip] Exam
> Resource-based policies are especially important for:
>
> **Cross-account access**

---

# 🆚 Identity vs Resource Policy

| Policy | Attached To | Main Question |
|---|---|---|
| Identity-Based | User / Group / Role | What can this identity do? |
| Resource-Based | Resource | Who can access this resource? |

```text
Identity Policy
     ↓
"What can YOU do?"

Resource Policy
     ↓
"Who can access ME?"
```

---

# 📦 Managed vs Inline Policies

## Managed Policy

Reusable policy that can be attached to multiple identities.

```text
Managed Policy
   ├── User A
   ├── User B
   └── Role C
```

Types include:

- AWS managed policies
- Customer managed policies

---

## Inline Policy

Embedded directly into **one identity**.

```text
Role
 └── Inline Policy
```

Strong one-to-one relationship.

> [!tip]
> **Managed Policy**
> → reusable
>
> **Inline Policy**
> → embedded in one identity

---

# 🚧 Permissions Boundary

A **Permissions Boundary** defines the **maximum permissions** an IAM user or role can receive.

It does **NOT grant permissions** by itself.

Example:

```text
Identity Policy
Allow:
S3 + DynamoDB + EC2

Permissions Boundary
Allow maximum:
S3 + DynamoDB

Effective Permissions
        ↓
S3 + DynamoDB
```

EC2 is not allowed because it exceeds the boundary.

> [!danger] VERY IMPORTANT EXAM TRAP
> A Permissions Boundary:
>
> ❌ Does NOT grant permissions
>
> ✅ Defines the **maximum possible permissions**

### Mental Model

```text
Identity Policy
       ∩
Permissions Boundary
       ↓
Effective Permissions
```

---

# ⚖️ IAM Policy Evaluation

AWS evaluates permissions before allowing a request.

Basic mental model:

```text
Request
   ↓
Explicit Deny?
   │
   ├── YES → ❌ DENY
   │
   └── NO
        ↓
Explicit Allow?
   │
   ├── YES → ✅ ALLOW
   │
   └── NO → ❌ IMPLICIT DENY
```

### Golden Rule

> [!danger]
> **Explicit DENY always wins.**

Priority:

```text
1️⃣ Explicit Deny
       ↓
2️⃣ Explicit Allow
       ↓
3️⃣ Implicit Deny
```

Example:

```text
Policy A → Allow s3:*
Policy B → Deny s3:DeleteObject

Result:

GetObject     ✅
PutObject     ✅
DeleteObject  ❌
```

---

# 🚫 Implicit vs Explicit Deny

## Implicit Deny

No policy grants permission.

```text
No Allow
   ↓
DENY
```

AWS permissions are denied by default.

---

## Explicit Deny

A policy explicitly contains:

```json
"Effect": "Deny"
```

This overrides an Allow.

```text
Allow + Explicit Deny
        ↓
       DENY
```

---

# 🏢 AWS Organizations SCP

A **Service Control Policy (SCP)** defines the maximum available permissions for accounts or OUs in AWS Organizations.

It does **not grant permissions**.

```text
SCP
 ↓
Account Maximum Permissions
 ↓
IAM Policies
 ↓
User / Role
```

> [!warning] Don't Confuse
> **IAM Policy**
> → Grants permissions to identities
>
> **Permissions Boundary**
> → Maximum permissions for a user/role
>
> **SCP**
> → Maximum permissions for accounts/OUs

### Exam Mental Model

```text
SCP
 ∩
Permissions Boundary
 ∩
IAM Permissions
        ↓
Effective Permissions
```

And:

```text
Explicit Deny
      ↓
Always wins
```

---

# 🔑 AWS Security Token Service (STS)

AWS STS provides **temporary security credentials**.

Temporary credentials contain:

```text
Access Key ID
+
Secret Access Key
+
Session Token
```

Common use cases:

- Assume IAM Roles
- Cross-account access
- Federation
- Temporary privileged access

---

# 🎭 STS AssumeRole

`AssumeRole` allows a principal to temporarily assume an IAM Role.

```text
User / Role
     ↓
sts:AssumeRole
     ↓
Target IAM Role
     ↓
Temporary Credentials
```

Common scenario:

```text
Account A
User
 ↓
AssumeRole
 ↓
Account B
Role
 ↓
Resources
```

> [!tip] Exam
> **Cross-account access without sharing credentials**
>
> → IAM Role + STS `AssumeRole`

---

# 🤝 Role Trust Policy

An IAM Role has a **Trust Policy** defining **who can assume the role**.

```text
Principal
   ↓
Trust Policy
   ↓
IAM Role
```

Think of two questions:

```text
Trust Policy
→ WHO can assume this role?

Permissions Policy
→ WHAT can this role do?
```

> [!danger] Exam Trap
> Giving a role S3 permissions does NOT automatically allow another account to assume it.
>
> The **trust relationship** must allow the principal.

---

# 🔐 MFA

Multi-Factor Authentication adds another authentication factor.

Strongly recommended for:

- Root user
- Administrators
- Privileged users

```text
Password
   +
MFA Device
   ↓
Authentication
```

STS can also require MFA when assuming roles.

---

# 🌐 Federation

Federation allows users from an external identity system to access AWS without creating permanent IAM users for everyone.

Examples:

- Corporate Active Directory
- SAML 2.0
- OpenID Connect
- External Identity Providers

```text
Corporate Identity
       ↓
Identity Provider
       ↓
Federation
       ↓
IAM Role
       ↓
Temporary AWS Credentials
```

## 🏢 SAML 2.0 Federation + Active Directory

Used when a company wants employees to access AWS using their **existing corporate credentials**, commonly stored in **Microsoft Active Directory**.

```text
Employee
   ↓
On-Premises Active Directory
   ↓
AD FS (Identity Provider)
   ↓
SAML 2.0 Assertion
   ↓
AWS STS
   ↓
Temporary Credentials
   ↓
IAM Role
   ↓
AWS Resources
```

### 🔑 Key Components

- **Active Directory** → Stores corporate identities and credentials
- **AD FS** → Acts as the Identity Provider (IdP)
- **SAML 2.0** → Federation protocol
- **AWS STS** → Provides temporary AWS credentials
- **IAM Role** → Defines AWS permissions

> [!tip] Exam Pattern
> **On-premises Active Directory + existing employee credentials + AWS access**
>
> → ✅ **SAML 2.0 Federation**
>
> If **AD FS** appears in the answers, it is a strong indicator.

> [!warning] Don't Confuse
> **SAML Federation**
> → Corporate/workforce identities (e.g., Active Directory)
>
> **Web Identity Federation / OIDC**
> → Web/mobile identities and external identity providers
>
> **IAM Users**
> → AWS identities with long-term credentials
---

# 🏢 AssumeRoleWithSAML

Used when users authenticate through a **SAML 2.0 identity provider**.

```text
Corporate Directory
        ↓
SAML IdP
        ↓
AssumeRoleWithSAML
        ↓
Temporary AWS Credentials
```

Typical scenario:

> Employees need AWS access using existing corporate identities.

---

# 🌐 AssumeRoleWithWebIdentity

Used with web identity providers / OIDC.

Examples:

- Amazon Cognito
- Google
- Other OIDC providers

```text
User
 ↓
Web Identity Provider
 ↓
AssumeRoleWithWebIdentity
 ↓
Temporary AWS Credentials
```

---

# 🎟️ GetSessionToken

Returns temporary credentials for an IAM user.

Common use:

**MFA-protected programmatic API access**

```text
IAM User
   +
MFA
   ↓
GetSessionToken
   ↓
Temporary Credentials
```

---

# 🏢 IAM Identity Center

AWS IAM Identity Center is the modern solution for managing **workforce access** to AWS accounts and applications.

Previously:

**AWS Single Sign-On (AWS SSO)**

```text
Corporate Users
       ↓
IAM Identity Center
       ↓
 ┌─────┼─────┐
 ▼     ▼     ▼
AWS   AWS   AWS
Acct  Acct  Acct
```

Identity sources can include:

- Identity Center directory
- Active Directory
- External Identity Provider

> [!tip] Exam
> Company has many employees and multiple AWS accounts?
>
> ✅ **IAM Identity Center**

---

# 🎫 Permission Sets

IAM Identity Center uses **Permission Sets** to define access.

Example:

```text
Developer Group
       ↓
Permission Set
"ReadOnly"
       ↓
AWS Account
       ↓
IAM Role created in account
```

Think:

```text
Permission Set
      ↓
Template for permissions
      ↓
Role in target AWS account
```

---

# 🏷️ ABAC

**Attribute-Based Access Control** uses attributes/tags to determine access.

Examples:

```text
Department = Finance
Project = Unicorn
Environment = Dev
```

Instead of creating many policies:

```text
Principal Tag:
Project = Unicorn

Resource Tag:
Project = Unicorn

        ↓

Access Allowed
```

> [!tip]
> **RBAC**
> → permissions based on roles
>
> **ABAC**
> → permissions based on attributes/tags

Useful at scale when many resources and identities exist.

---

# 🔎 IAM Access Analyzer

IAM Access Analyzer helps identify resources that are accessible by external principals and validates IAM policies.

Can help detect unintended access involving resources such as:

- S3 buckets
- KMS keys
- SQS queues
- IAM roles
- Lambda functions
- Secrets Manager secrets

```text
Resource Policy
      ↓
IAM Access Analyzer
      ↓
External Access?
      ↓
Finding
```

It also provides policy validation:

```text
IAM Policy
    ↓
Access Analyzer
    ↓
Errors
Warnings
Suggestions
```

> [!tip] Exam
> Need to identify resources shared with an **external account or public principal**?
>
> ✅ **IAM Access Analyzer**

---

# 🏷️ Policy Conditions

Conditions restrict when a permission applies.

Examples:

- Source IP
- MFA
- VPC Endpoint
- Tags
- Time
- Organization

Example:

```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

Mental model:

```text
Allow Action
     +
Condition must match
     ↓
Access
```

---

# 🔑 IAM Roles for EC2

Applications running on EC2 should use IAM Roles instead of stored access keys.

```text
EC2
 ↓
Instance Profile / IAM Role
 ↓
Temporary Credentials
 ↓
AWS API
```

> [!danger] Exam Trap
> Application on EC2 needs access to S3:
>
> ❌ Store access keys in code
>
> ❌ Store access keys in user data
>
> ❌ Store access keys on disk
>
> ✅ **Attach IAM Role to EC2**

---

# ⚡ IAM Roles for Lambda

Lambda functions use an **Execution Role**.

```text
Lambda
   ↓
Execution Role
   ↓
AWS Services
```

Example:

```text
Lambda
 ↓
Execution Role
 ↓
s3:GetObject
 ↓
S3
```

The role determines what the function can access.

---

# 🎟️ iam:PassRole

`iam:PassRole` allows a principal to pass an IAM Role to an AWS service.

Example:

```text
Developer
   ↓
Creates Lambda
   ↓
Passes Execution Role
   ↓
Lambda assumes Role
```

The developer may need:

```text
iam:PassRole
```

> [!danger] Exam Trap
> **PassRole ≠ AssumeRole**
>
> `AssumeRole`
> → YOU become the role.
>
> `PassRole`
> → You give the role to an AWS service.

---

# 🆚 Authentication vs Authorization

| Concept | Question |
|---|---|
| Authentication | Who are you? |
| Authorization | What can you do? |

```text
Login
 ↓
Authentication
 ↓
Policies evaluated
 ↓
Authorization
```

## 🏢 Active Directory Federation — Directory Service

### AD Connector

Used to connect AWS services to an **existing on-premises Active Directory**.

```text
Existing Corporate AD
        ↓
   AD Connector
        ↓
     IAM Roles
        ↓
AWS Management Console

---

# 🧠 High-Value Exam Traps

> [!danger] Trap 1 — Explicit Deny
> An Allow and an explicit Deny both apply.
>
> ✅ **DENY wins**

---

> [!danger] Trap 2 — Permissions Boundary
> Need to prevent developers from ever exceeding a certain permission level.
>
> ✅ **Permissions Boundary**

---

> [!danger] Trap 3 — Cross-Account
> Account A needs temporary access to resources in Account B.
>
> ✅ **IAM Role + STS AssumeRole**

---

> [!danger] Trap 4 — Compute Credentials
> EC2/Lambda needs AWS API access.
>
> ✅ **IAM Role**
>
> ❌ Hard-coded access keys

---

> [!danger] Trap 5 — Workforce Access
> Hundreds of employees need access to multiple AWS accounts.
>
> ✅ **IAM Identity Center**

---

> [!danger] Trap 6 — External Access
> Need to discover S3/KMS/etc. resources shared externally.
>
> ✅ **IAM Access Analyzer**

---

> [!danger] Trap 7 — Who Can Assume Role?
> Need to control which principal can assume a role.
>
> ✅ **Trust Policy**

---

> [!danger] Trap 8 — What Can Role Do?
> Need to control actions after the role is assumed.
>
> ✅ **Permissions Policy**

---

> [!danger] Trap 9 — Give Role to AWS Service
> Developer creates Lambda and assigns an execution role.
>
> ✅ **iam:PassRole**

---

> [!danger] Trap 10 — MFA for Programmatic Access
> Need temporary credentials protected by MFA.
>
> ✅ **STS GetSessionToken**

---

# 🆚 Quick Comparisons

| Requirement | Solution |
|---|---|
| Individual AWS identity | IAM User |
| Organize IAM users | IAM Group |
| Temporary permissions | IAM Role |
| Temporary credentials | STS |
| Cross-account access | AssumeRole |
| Define who assumes role | Trust Policy |
| Define what role can do | Permissions Policy |
| Maximum permissions for user/role | Permissions Boundary |
| Maximum permissions for AWS account/OU | SCP |
| Workforce multi-account access | IAM Identity Center |
| Find unintended external access | IAM Access Analyzer |
| Pass role to AWS service | iam:PassRole |
| MFA temporary API credentials | GetSessionToken |

---

# ⚡ IAM in 30 Seconds

```text
IAM
│
├── 👤 User
│   └── Long-term identity
│
├── 👥 Group
│   └── Collection of users
│
├── 🎭 Role
│   └── Temporary assumed identity
│
├── 📜 Policies
│   ├── Identity-Based → what identity can do
│   └── Resource-Based → who can access resource
│
├── 🚧 Permissions Boundary
│   └── Maximum identity permissions
│
├── ⚖️ Evaluation
│   ├── Explicit Deny → DENY
│   ├── Explicit Allow → ALLOW
│   └── No Allow → DENY
│
├── 🔑 STS
│   └── Temporary credentials
│
├── 🤝 AssumeRole
│   └── Cross-account / temporary access
│
├── 🏢 Identity Center
│   └── Workforce + multi-account access
│
└── 🔎 Access Analyzer
    └── External access + policy validation
```

---

> [!summary] SAA Memory Trick
> **WHO are you?**
> → 👤 User / 🎭 Role
>
> **WHAT can you do?**
> → 📜 IAM Policy
>
> **WHO can assume the role?**
> → 🤝 Trust Policy
>
> **WHAT is the maximum you can ever do?**
> → 🚧 Permissions Boundary
>
> **Need temporary credentials?**
> → 🔑 STS
>
> **Need cross-account access?**
> → 🎭 AssumeRole
>
> **Need workforce access across many accounts?**
> → 🏢 IAM Identity Center
>
> **Need to find external resource access?**
> → 🔎 IAM Access Analyzer
>
> **Allow + Explicit Deny?**
> → ❌ DENY