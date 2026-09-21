# 🌉 AWS Storage Gateway

> [!summary] Mental Model
> **Storage Gateway = Hybrid On-Premises ↔ AWS Storage**
>
> Provides on-premises applications access to AWS storage while supporting hybrid storage architectures.

---

# 📁 File Gateway

Provides a file interface to cloud storage.

Supports:

- SMB
- NFS
- Local cache for frequently/recently accessed data
- Amazon S3 as cloud storage

```text
On-Prem Application
       ↓
    SMB / NFS
       ↓
  File Gateway
  Local Cache
       ↓
   Amazon S3
```

> [!tip] Exam Pattern
> **On-premises**
> +
> **SMB/NFS**
> +
> **Local cache**
> +
> **S3 durable storage**
>
> → ✅ **File Gateway**

---

# 🪣 File Gateway + S3 Lifecycle

Objects stored through File Gateway can use S3 capabilities such as lifecycle policies.

```text
On-Prem
   ↓ SMB
File Gateway
   ↓
S3 Standard
   ↓ Lifecycle
S3 Glacier
```

> [!tip] Exam Pattern
> Recent files need fast access locally but old files must be archived.
>
> → **File Gateway + S3 Lifecycle**

---

# 📼 Tape Gateway

Think:

**Virtual tapes / backup / archive**

```text
Existing Backup Software
        ↓
    Tape Gateway
        ↓
Virtual Tape Library
        ↓
 AWS Cloud Storage
```

> [!tip] Exam Pattern
> **Replace physical tape infrastructure**
>
> → ✅ Tape Gateway

---

# 🆚 File Gateway vs DataSync

| Requirement | Service |
|---|---|
| SMB/NFS access to cloud storage | **File Gateway** |
| Local cache | **File Gateway** |
| Hybrid file storage | **File Gateway** |
| Bulk data transfer | **DataSync** |
| Migration to AWS | **DataSync** |
| Scheduled/recurring data transfer | **DataSync** |

> [!danger] Exam Trap
> **DataSync transfers data.**
>
> **File Gateway provides ongoing file access + local cache.**

---

# 🆚 Transfer Family vs File Gateway vs DataSync

```text
External users / partners
using SFTP
        ↓
Transfer Family

On-Prem applications
using SMB/NFS + local cache
        ↓
File Gateway

Bulk migration / synchronization
        ↓
DataSync
```

| Keyword | Think |
|---|---|
| SFTP / FTPS | **Transfer Family** |
| SMB/NFS + local cache | **File Gateway** |
| Bulk migration/sync | **DataSync** |
| Virtual tapes / backup | **Tape Gateway** |

---

# ⚡ Storage Gateway in 20 Seconds

```text
Hybrid On-Prem + AWS
        ↓
Storage Gateway

SMB/NFS + S3 + Cache
→ File Gateway

Virtual Tapes / Backup
→ Tape Gateway
```

> [!summary] SAA Memory
> **SMB/NFS + CACHE → File Gateway**
>
> **TAPES → Tape Gateway**
>
> **MIGRATE/SYNC → DataSync**
>
> **SFTP → Transfer Family**