# 🗄️ Amazon FSx

> [!summary] Mental Model
> **Amazon FSx = Fully Managed File Systems**
>
> Choose the file system based on the workload:
>
> **Windows / SMB → FSx for Windows**
>
> **HPC / ML / S3 processing → FSx for Lustre**
>
> **NetApp / Multi-protocol → FSx for ONTAP**
>
> **ZFS / NFS / snapshots & cloning → FSx for OpenZFS**

---

# 🎯 Core Purpose

Amazon FSx provides fully managed file systems based on popular file-system technologies.

```text
Amazon FSx
│
├── 🪟 FSx for Windows File Server
├── ⚡ FSx for Lustre
├── 🔵 FSx for NetApp ONTAP
└── 📂 FSx for OpenZFS
```

---

# 🪟 FSx for Windows File Server

Fully managed **native Windows file system**.

Uses:

- Microsoft Windows Server
- SMB protocol
- Windows file shares
- Active Directory integration

```text
Windows EC2
     ↓
    SMB
     ↓
FSx for Windows
```

Common use cases:

- Windows applications
- Windows file shares
- Home directories
- Lift-and-shift Windows workloads
- Applications requiring SMB
- Active Directory environments

> [!tip] Exam Pattern
> **Windows + SMB + Active Directory**
>
> → ✅ **FSx for Windows File Server**

AWS provides native Windows compatibility and SMB support. :chatgpt-content-reference{index="2"}

---

# ⚡ FSx for Lustre

A high-performance distributed file system designed for **compute-intensive workloads**.

Think:

```text
HPC
ML
Financial Modeling
Video Processing
Large-scale Data Processing
        ↓
FSx for Lustre
```

FSx for Lustre is designed for very high throughput and parallel access across many clients. :chatgpt-content-reference{index="3"}

> [!tip] Exam Pattern
> **High Performance Computing (HPC)**
> +
> **Massive parallel file access**
>
> → ✅ **FSx for Lustre**

---

# 🪣 FSx for Lustre + S3

This is VERY important for SAA.

FSx for Lustre can integrate with **Amazon S3** for high-performance processing of datasets stored in S3.

```text
Amazon S3
Long-term Dataset
      ↓
FSx for Lustre
      ↓
High-Speed File Access
      ↓
EC2 / HPC / ML
```

FSx for Lustre can present S3 data through a high-performance filesystem and can transfer data between the filesystem and S3. :chatgpt-content-reference{index="4"}

> [!danger] Exam Pattern
> **S3 dataset**
> +
> **HPC / ML / high-performance processing**
>
> → ✅ **FSx for Lustre**

---

# 🔵 FSx for NetApp ONTAP

Fully managed storage based on **NetApp ONTAP**.

Supports multiple protocols, including:

- NFS
- SMB
- iSCSI
- NVMe :chatgpt-content-reference{index="5"}


```text
Linux ── NFS ──┐
Windows ─ SMB ──┼── FSx for ONTAP
Apps ── iSCSI ──┘
```

Best when:

- Migrating existing NetApp workloads
- Need ONTAP features
- Need multi-protocol access
- Hybrid/on-prem NetApp environments

> [!tip] Exam Pattern
> **Existing NetApp ONTAP environment**
>
> → ✅ **FSx for NetApp ONTAP**

> [!tip]
> **Need NFS + SMB + iSCSI from the same storage platform**
>
> → Think **FSx for ONTAP**

---

# 📂 FSx for OpenZFS

Fully managed storage built on **OpenZFS**.

Uses the NFS protocol and provides features such as:

- Snapshots
- Data cloning
- Compression
- NFS access
- ZFS data management capabilities :chatgpt-content-reference{index="6"}


```text
Existing ZFS / Linux File Server
             ↓
      FSx for OpenZFS
```

Best for:

- Migrating ZFS workloads
- Linux-based file servers
- Applications requiring ZFS features
- Low-latency NFS workloads

> [!tip] Exam Pattern
> **Existing ZFS workload**
>
> → ✅ **FSx for OpenZFS**

---

# 🧠 The Most Important Comparison

| Requirement | Solution |
|---|---|
| Windows file shares / SMB | **FSx for Windows** |
| Active Directory + Windows files | **FSx for Windows** |
| HPC | **FSx for Lustre** |
| ML / massive parallel processing | **FSx for Lustre** |
| High-performance processing of S3 data | **FSx for Lustre** |
| Existing NetApp environment | **FSx for ONTAP** |
| Multi-protocol NFS + SMB + iSCSI | **FSx for ONTAP** |
| Existing ZFS environment | **FSx for OpenZFS** |
| ZFS snapshots / cloning | **FSx for OpenZFS** |

---

# 🆚 EFS vs FSx

This is important for SAA.

## Amazon EFS

```text
Linux / NFS
+
General Shared Files
+
Multiple EC2
        ↓
       EFS
```

## Amazon FSx

```text
Specific File-System Requirement
        ↓
Windows / Lustre / ONTAP / ZFS
        ↓
        FSx
```

| Requirement | Think |
|---|---|
| General-purpose managed NFS shared storage | EFS |
| Native Windows / SMB | FSx for Windows |
| HPC / parallel filesystem | FSx for Lustre |
| NetApp | FSx for ONTAP |
| ZFS | FSx for OpenZFS |

> [!danger] Exam Trap
> **Multiple Linux EC2 instances need a simple shared NFS filesystem**
>
> → EFS
>
> **High-performance parallel processing / HPC**
>
> → FSx for Lustre

---

# 🆚 EBS vs EFS vs FSx vs S3

| Storage | Mental Model |
|---|---|
| **EBS** | EC2 block disk |
| **EFS** | Shared NFS filesystem |
| **FSx** | Specialized managed filesystem |
| **S3** | Object storage |

```text
EC2 Disk
→ EBS

Shared Linux Files
→ EFS

Windows / HPC / NetApp / ZFS
→ FSx

Objects
→ S3
```

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> Windows applications require a shared filesystem using **SMB**.
>
> ✅ FSx for Windows File Server

---

> [!danger] Trap 2
> Windows file shares must integrate with **Active Directory**.
>
> ✅ FSx for Windows File Server

---

> [!danger] Trap 3
> Application performs **HPC or massively parallel processing**.
>
> ✅ FSx for Lustre

---

> [!danger] Trap 4
> Large dataset is stored in **S3** and must be processed using a high-performance filesystem.
>
> ✅ FSx for Lustre

---

> [!danger] Trap 5
> Company wants to migrate existing **NetApp ONTAP** workloads.
>
> ✅ FSx for NetApp ONTAP

---

> [!danger] Trap 6
> Application requires **NFS + SMB + iSCSI**.
>
> ✅ FSx for NetApp ONTAP

---

> [!danger] Trap 7
> Company wants to migrate existing **ZFS** workloads.
>
> ✅ FSx for OpenZFS

---

> [!danger] Trap 8
> Multiple Linux EC2 instances simply need shared NFS storage.
>
> ❌ Don't automatically choose FSx.
>
> ✅ Think **EFS**

---

# ⚡ FSx in 20 Seconds

```text
Amazon FSx
│
├── 🪟 Windows
│   ├── SMB
│   └── Active Directory
│
├── ⚡ Lustre
│   ├── HPC
│   ├── ML
│   ├── Parallel Processing
│   └── S3 Integration
│
├── 🔵 ONTAP
│   ├── NetApp
│   └── NFS + SMB + iSCSI
│
└── 📂 OpenZFS
    ├── ZFS
    ├── NFS
    ├── Snapshots
    └── Cloning
```

> [!summary] SAA Memory Trick
> **Windows → FSx Windows**
>
> **HPC → Lustre**
>
> **S3 + HPC → Lustre**
>
> **NetApp → ONTAP**
>
> **ZFS → OpenZFS**
>
> **Generic shared NFS → EFS**