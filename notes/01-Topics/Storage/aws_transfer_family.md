# 📤 AWS Transfer Family

> [!summary] Mental Model
> **AWS Transfer Family = Managed File Transfer**
>
> ```text
> SFTP / FTPS / FTP / AS2
>           ↓
>   AWS Transfer Family
>           ↓
>       S3 or EFS
> ```

AWS Transfer Family provides fully managed file transfer services without having to manage your own SFTP/FTP servers.

---

# 🎯 Supported Protocols

| Protocol | Key Point |
|---|---|
| **SFTP** | File transfer over SSH |
| **FTPS** | FTP secured with TLS |
| **FTP** | Unencrypted FTP |
| **AS2** | B2B / partner data exchange |

> [!tip] Exam Pattern
> **Managed SFTP server in AWS**
>
> → ✅ AWS Transfer Family

---

# 🗄️ Storage Backends

Transfer Family can use:

- **Amazon S3**
- **Amazon EFS**

```text
             Transfer Family
             /             \
            ↓               ↓
           S3              EFS
        Objects        File System
```

---

# 🪣 Transfer Family + S3

Use S3 when you need:

- Object storage
- High durability
- Lifecycle rules
- Archiving
- Cost-effective storage
- Integration with other AWS services

```text
SFTP Client
    ↓
Transfer Family
    ↓
Amazon S3
    ↓
Lifecycle Rule
    ↓
Archive / Delete
```

> [!tip] Exam Pattern
> **SFTP + automatically delete files after X days**
>
> → Transfer Family + **S3 Lifecycle**

---

# 📂 Transfer Family + EFS

Use EFS when applications require a real shared file system.

```text
SFTP Client
    ↓
Transfer Family
    ↓
Amazon EFS
    ↓
NFS File System
```

Think:

- POSIX-style filesystem
- Shared files
- File-system semantics
- Existing applications that need EFS

---

# 🔐 Security

Transfer Family supports secure managed file-transfer architectures.

For SFTP:

```text
Client
  ↓
SFTP / SSH
  ↓
Transfer Family
  ↓
S3 / EFS
```

Authentication can use different identity-provider options depending on the configuration.

IAM controls Transfer Family access to the underlying AWS resources.

---

# 🌐 Endpoints

Transfer Family supports different endpoint configurations depending on the protocol and access requirements.

Think:

```text
Internet Clients
      ↓
Public Endpoint
      ↓
Transfer Family
```

or

```text
Private / Corporate Network
          ↓
      VPC Endpoint
          ↓
    Transfer Family
```

> [!tip] Exam Pattern
> Need managed file transfer accessible only from private networks
>
> → Think **VPC-hosted Transfer Family endpoint**

---

# ⚙️ Managed Workflows

Transfer Family can automatically process uploaded files.

```text
File Upload
    ↓
Transfer Family
    ↓
Managed Workflow
    ↓
Process File
```

Useful for automated post-upload processing.

---

# 🆚 SFTP vs FTPS vs FTP

```text
SFTP
→ SSH
→ Secure

FTPS
→ FTP + TLS
→ Secure

FTP
→ Unencrypted
```

> [!danger] Exam Trap
> **SFTP is NOT FTP over SSL/TLS.**
>
> SFTP → SSH
>
> FTPS → TLS

---

# 🆚 Transfer Family vs EC2 SFTP Server

## Self-Managed

```text
Client
  ↓
EC2
  ↓
Install SFTP
Patch OS
Scale
Monitor
Configure HA
```

## Managed

```text
Client
  ↓
AWS Transfer Family
  ↓
S3 / EFS
```

> [!tip] Exam Pattern
> **SFTP + least operational overhead**
>
> → ✅ AWS Transfer Family
>
> ❌ EC2 + manually installed SFTP

---

# ♻️ Transfer Family + S3 Lifecycle

Very common exam architecture:

```text
Partner
  ↓
SFTP
  ↓
AWS Transfer Family
  ↓
Encrypted S3 Bucket
  ↓
S3 Lifecycle
  ↓
Transition / Expire
```

> [!important] Exam Pattern
> **SFTP**
> +
> **High Availability**
> +
> **Encryption at Rest**
> +
> **Delete after N days**
> +
> **Least Operational Overhead**
>
> → ✅ Transfer Family + encrypted S3 + S3 Lifecycle

---

# ⚠️ Lifecycle Trap

### S3 Lifecycle

Can:

- Transition objects to cheaper storage classes
- Expire/delete objects automatically

```text
S3 Object
   ↓ 30 days
S3 Lifecycle
   ↓
DELETE
```

### EFS Lifecycle Management

Used primarily to move files between EFS storage classes based on access patterns.

```text
Frequently Accessed
       ↓
Infrequently Accessed
```

> [!danger]
> Requirement:
> **"Automatically DELETE files after 30 days"**
>
> → ✅ S3 Lifecycle
>
> Do not choose EFS Lifecycle Management for expiration.

---

# 🆚 Transfer Family vs DataSync

Do not confuse them.

| Requirement | Service |
|---|---|
| Customers/partners upload using SFTP | **Transfer Family** |
| FTP / FTPS server migration | **Transfer Family** |
| B2B AS2 transfers | **Transfer Family** |
| Automated bulk data movement | **DataSync** |
| On-prem NFS/SMB → AWS storage migration | **DataSync** |

```text
SFTP / FTPS / FTP
→ Transfer Family

Bulk data migration / synchronization
→ DataSync
```

---

# 🧠 Transfer Family in 20 Seconds

```text
AWS Transfer Family
│
├── SFTP → SSH
├── FTPS → TLS
├── FTP → Unencrypted
├── AS2 → B2B
│
├── Storage
│   ├── S3
│   └── EFS
│
└── Managed
    └── No EC2 SFTP server management
```

> [!summary] SAA Memory
> **Managed SFTP → Transfer Family**
>
> **SFTP + S3 → very common architecture**
>
> **Delete after N days → S3 Lifecycle**
>
> **SFTP = SSH**
>
> **FTPS = TLS**
>
> **Bulk migration/sync ≠ Transfer Family → DataSync**