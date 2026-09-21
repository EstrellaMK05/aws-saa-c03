# 🪣 Amazon S3

> [!summary] Mental Model
> **S3 = regional object storage**
> Highly durable, scalable, and designed for storing objects inside buckets.

---

## 📌 Core

- **Object storage**
- Bucket names must be **globally unique**
- Bucket Region **cannot be changed** after creation
- S3 is a **regional service**
- Strong read-after-write consistency for:
  - `PUT`
  - Overwrite
  - `DELETE`
  - `GET`
  - `HEAD`
  - `LIST`

---

## 🗄️ Storage Classes

| Storage Class | Best For | Key Point |
|---|---|---|
| **S3 Standard** | Frequently accessed data | Multi-AZ |
| **Intelligent-Tiering** | Unknown/changing access | Automatic tiering |
| **Standard-IA** | Infrequent access | Multi-AZ + retrieval fee |
| **One Zone-IA** | Infrequent + recreatable | Single AZ, cheaper |
| **Glacier Instant Retrieval** | Archive + immediate access | Millisecond retrieval |
| **Glacier Flexible Retrieval** | Archive | Minutes to hours |
| **Glacier Deep Archive** | Long-term archive | Lowest-cost archival option |

> [!tip] 🎯 Exam Clues
> **Unknown access pattern** → Intelligent-Tiering  
> **Infrequent + HA** → Standard-IA  
> **Infrequent + recreatable** → One Zone-IA  
> **Long-term archive** → Glacier / Deep Archive

---

## 🔄 Versioning

- Disabled by default
- Keeps **multiple versions** of an object
- Protects against accidental overwrite/delete
- `DELETE` without `VersionId` → creates a **delete marker**
- Previous versions still exist
- Permanent deletion → specify the `VersionId`

> [!warning] ⚠️ Exam Trap
> A normal `DELETE` on a versioned bucket **does not permanently delete the object**.
> It creates a **delete marker**.

---

## 🌎 Replication

### CRR — Cross-Region Replication
Replicates objects to a bucket in **another Region**.

### SRR — Same-Region Replication
Replicates objects within the **same Region**.

### Requirements

- Versioning enabled on **source and destination**
- S3 needs permission to replicate

### Existing vs New Objects

**New objects**
→ CRR / SRR replication rule

**Existing objects**
→ **S3 Batch Replication**

**SSE-KMS objects**
→ KMS replication must be explicitly configured

> [!warning] ⚠️ Exam Trap
> **Transfer Acceleration ≠ Replication**
>
> CRR → copy objects across Regions  
> Transfer Acceleration → accelerate client ↔ S3 transfers

---

## ♻️ Lifecycle

Lifecycle rules can:

- Transition objects between storage classes
- Expire/delete objects
- Manage noncurrent versions
- Abort incomplete multipart uploads

> [!tip] 🎯 Exam Clue
> **Predictable change in access pattern over time**
> → S3 Lifecycle Policy

---

# 🔒 Security

## Object Lock

Provides **WORM** protection:

**Write Once, Read Many**

### Governance Mode

- Prevents deletion/overwrite
- Authorized users can **bypass retention**

### Compliance Mode

- Cannot be overwritten/deleted during retention
- **Even root cannot bypass retention**

> [!danger] ⚠️ Remember
> **Governance** → privileged users may bypass  
> **Compliance** → nobody can bypass during retention

### Retention Period

- Has an **expiration date**
- Protects object until retention expires

### Legal Hold

- **No expiration date**
- Remains until explicitly removed
- Independent of retention period
- Requires `s3:PutObjectLegalHold`

> [!important]
> **Retention Period + Legal Hold are independent.**
>
> If either protection is still active → the object remains protected.

---

## 🔐 Encryption

### SSE-S3

- S3 manages encryption keys
- Default server-side encryption for new objects

### SSE-KMS

- Uses AWS KMS keys
- More control over key permissions
- KMS activity can be audited with CloudTrail

**Reading an SSE-KMS object requires:**

` s3:GetObject `  
+
` kms:Decrypt `

### DSSE-KMS

- Two independent layers of server-side encryption
- Useful for strict compliance/security requirements

### SSE-C

- Customer provides encryption key
- AWS performs encryption/decryption
- AWS **does not store the customer key**

### Client-Side Encryption

→ Encrypt data **before uploading** to S3

## 🔐 Client-Side vs Server-Side Encryption

### Server-Side Encryption

```text
Plaintext
   ↓
   AWS
   ↓
Encryption
   ↓
S3
---

## 🛡️ Access Control

Objects are **private by default**.

### Bucket Policy

Resource-based policy.

Supports:

- `Allow`
- Explicit `Deny`
- Conditions

**Require HTTPS**

`aws:SecureTransport`

**Require specific VPC Endpoint**

`aws:SourceVpce`

### ACL

- Legacy access control mechanism
- Disabled by default for new buckets with **Bucket Owner Enforced**
- Prefer **IAM + Bucket Policies**

### Block Public Access

→ Prevent accidental public exposure

---

## 🔗 Presigned URLs

Provides **temporary access** to private S3 objects.

- Temporary upload/download
- User doesn't need AWS credentials
- Uses permissions of the principal that generated the URL

> [!tip] 🎯 Exam Clue
> **Temporary access to private S3 object without AWS credentials**
> → Presigned URL

---

# ⚡ Performance & Delivery

## 🌐 CloudFront + S3

Private content distribution:

`User → CloudFront → OAC → Private S3 Bucket`

### OAC — Origin Access Control

Allows CloudFront to access a **private S3 bucket**.

### CloudFront

→ Global content delivery  
→ Edge caching  
→ Reduces load on origin

---

## 🚀 S3 Transfer Acceleration

Accelerates **long-distance transfers**.

`Client → Edge Location → AWS Global Network → S3`

Useful for:

- Large uploads/downloads
- Geographically distant clients

Does **NOT**:

- ❌ Replicate objects
- ❌ Cache objects like CloudFront

> [!tip] 🎯 Exam Clue
> Users around the world uploading large files to **one S3 bucket**
> → **S3 Transfer Acceleration**

---

## 🧩 Multipart Upload

- Upload large objects in **parts**
- Failed parts can be retried independently
- Improves reliability for large uploads

> [!tip] 🎯 Exam Clue
> **Large object + unreliable network**
> → Multipart Upload

---

# 📢 Events & Operations

## S3 Event Notifications

Can react to events such as:

- Object creation
- Object deletion

Targets include:

- Lambda
- SQS
- SNS
- EventBridge integration

---

## 🔑 MFA Delete

Adds MFA protection for:

- Permanently deleting object versions
- Changing bucket versioning state

---

## 📋 S3 Inventory

Generates scheduled reports of:

- Objects
- Metadata

Useful for:

→ Auditing  
→ Analysis of large buckets

---

# 🌐 Networking

## Gateway VPC Endpoint

- Supports **S3 and DynamoDB**
- Private VPC access
- No NAT/Internet required
- Preferred for normal private S3 access from a VPC

## Interface Endpoint

- PrivateLink-based
- S3 also supports Interface Endpoints
- Usually more expensive than Gateway Endpoint

> [!tip] 🎯 Exam Clue
> **EC2 private subnet → S3 + lowest cost**
> → Gateway VPC Endpoint

---

# 🧠 Quick Exam Traps

| If you see... | Think... |
|---|---|
| Replication across Regions | **CRR** |
| Replicate existing objects | **S3 Batch Replication** |
| Faster global uploads to S3 | **Transfer Acceleration** |
| Global cached content | **CloudFront** |
| Temporary private access | **Presigned URL** |
| WORM | **Object Lock** |
| No expiration protection | **Legal Hold** |
| Retention nobody can bypass | **Compliance Mode** |
| Versioned object + DELETE | **Delete Marker** |
| Read SSE-KMS object | `s3:GetObject` + `kms:Decrypt` |
| Predictable hot → cold transition | **Lifecycle Policy** |
| Unknown access pattern | **Intelligent-Tiering** |
| Private EC2 → S3, lowest cost | **Gateway Endpoint** |

---

> [!abstract] 🧠 S3 in 20 Seconds

> > **Storage:** Standard → IA → Glacier  
> **Unknown pattern:** Intelligent-Tiering  
> **Replication:** CRR / SRR  
> **Existing replication:** Batch Replication  
> **Protection:** Versioning + Object Lock  
> **Temporary access:** Presigned URL  
> **Global delivery:** CloudFront  
> **Global uploads:** Transfer Acceleration  
> **Private VPC access:** Gateway Endpoint  
> **Encryption:** SSE-S3 / SSE-KMS  
> **Automation:** Lifecycle + Event Notifications