---
aliases: [Amazon S3, Simple Storage Service]
tags: [aws/saa, storage]
---

# Amazon S3 — Objects, Protection and Lifecycle

## Mental Model

**Store objects by key; access them through APIs. Choose the storage class by access frequency and retrieval requirements.**

S3 provides durable object storage, not a normal shared POSIX filesystem or EC2 block device.

## Core

### Buckets and consistency

This note focuses on general-purpose buckets. A bucket has a fixed Region; moving data to another Region requires another bucket/data transfer. The default global namespace requires unique names across accounts/Regions within an AWS partition. Account-regional namespaces are also available, so “all bucket names always use one global naming scope” is too broad.

S3 provides strong consistency for object writes, overwrites, deletes and listing. This does not make cross-Region replication synchronous or invalidate cached copies in CloudFront.

### Storage-class decisions

| Class | Access model | Cost/recovery distinction |
|---|---|---|
| Standard | Frequent, immediate access | Regional resilience; no minimum storage duration |
| Intelligent-Tiering | Unknown/changing access | Monitoring/tiering charges; optional archive tiers require restore |
| Standard-IA | Infrequent but immediate | Multi-AZ; retrieval charges; 30-day minimum duration |
| One Zone-IA | Infrequent, recreatable data | One AZ; retrieval charges; 30-day minimum duration |
| Glacier Instant Retrieval | Rare but millisecond access | 90-day minimum duration; retrieval charges |
| Glacier Flexible Retrieval | Archive with minutes-to-hours restore | Restore first; 90-day minimum duration |
| Glacier Deep Archive | Long-term archive with hours of restore tolerance | Restore first; 180-day minimum duration |
| Express One Zone | Latency-sensitive object access near compute | Directory buckets in one AZ; different feature/availability trade-offs |

Minimum durations are billing conditions, not locks preventing deletion. Small-object minimum billable sizes, transition requests and retrieval fees can dominate savings. Intelligent-Tiering does not automatically optimize every tiny object; objects below its tiering size threshold remain in the frequent tier.

### Lifecycle and versioning

Lifecycle can transition eligible objects, expire current/noncurrent versions and abort incomplete multipart uploads. Transition eligibility depends on class, object size and age; a lifecycle rule is not permission to ignore minimum storage charges.

With versioning enabled, a DELETE without a version ID normally adds a delete marker. The older data remains and is billable. Deleting a specific version is permanent unless retention protection blocks it. Suspending versioning does not erase existing history.

For “delete after N days,” check **noncurrent versions**, delete markers, retention and replication too. Expiring only the current version of a versioned object is not complete data erasure.

### Replication

- **CRR:** another Region; **SRR:** same Region. General-purpose bucket replication requires versioning and suitable permissions.
- Live replication rules apply to eligible new writes; use **Batch Replication** for existing eligible objects.
- Replication is asynchronous. S3 Replication Time Control addresses a defined replication-time requirement; ordinary CRR is not a zero-RPO guarantee.
- SSE-KMS replication needs explicit configuration and suitable source/destination KMS permissions.
- Delete-marker behavior is configurable; do not assume every deletion replicates. Permanent deletion of a specific source version is not replicated as deletion of the destination version.

### Access and encryption

Keep objects private using IAM/resource policies and Block Public Access. New general-purpose buckets normally use Bucket owner enforced Object Ownership with ACLs disabled. Prefer policies over legacy ACLs.

| Mechanism | What it does |
|---|---|
| SSE-S3 | S3-managed server-side encryption; baseline protection for new object uploads |
| SSE-KMS | KMS key control/auditing; reading requires S3 and KMS authorization |
| DSSE-KMS | Two layers of server-side encryption for suitable requirements |
| SSE-C | Customer supplies encryption key material; configuration/support restrictions apply |
| Client-side encryption | Data is encrypted before S3 receives it |
| TLS / SecureTransport policy condition | Protect/require transport encryption |

Changing bucket default encryption does not retroactively re-encrypt every existing object. S3 Bucket Keys can reduce KMS request costs for supported SSE-KMS use cases. Encryption does not itself prevent a policy from authorizing public access.

**Presigned URLs** delegate the signer's permitted access for a limited time. They are bearer credentials and may expire earlier when underlying temporary credentials expire; explicit denies still apply.

### Object Lock

Object Lock protects **object versions** with WORM retention and requires versioning. Governance retention can be bypassed with specific permissions and the appropriate bypass request. Compliance retention cannot be shortened/bypassed during its term, including by root. A legal hold has no expiration and persists until an authorized user removes it.

Retention and legal hold are independent. A new version or delete marker does not destroy the protected version. Object Lock can be enabled on eligible existing buckets; it is not limited to bucket creation.

## Comparisons

| Requirement | Mechanism |
|---|---|
| Retry parts of a large upload | Multipart upload; clean up abandoned parts |
| Download part of an object | Range GET |
| Accelerate distant clients' transfers into/out of a bucket | Transfer Acceleration |
| Cache content globally | [[aws_cloudfront|CloudFront]] |
| Private S3 origin behind CloudFront | OAC with a suitable bucket policy; website endpoints use a different model |
| Low-cost private S3 access from a VPC | Gateway endpoint when its connectivity scope fits |
| Private access through supported hybrid network paths | Consider an S3 interface endpoint; evaluate DNS and cost |
| Scheduled object/metadata report | S3 Inventory |
| Act on object events | Notifications or EventBridge integration |

Event notifications can arrive more than once and are not globally ordered. Direct S3 notifications do not target SQS FIFO; route through EventBridge when that integration is required. Avoid writing results to the same triggering location without filters, which can create a loop.

## Exam Traps

- **Glacier Instant Retrieval does not need a restore job; Flexible/Deep Archive do.**
- **Strong consistency does not mean synchronous replication.**
- **Lifecycle expiration is not Object Lock.** One deletes eligible data; the other prevents protected version deletion.
- **A delete marker is not permanent deletion of all versions.**
- **SSE-KMS needs KMS permissions as well as S3 access.**
- **Transfer Acceleration does not create a replica or act as CloudFront caching.**
- **One Zone classes do not protect against destruction of that AZ.**

## Scenario Check

**Compliance records must be immediately readable, infrequently accessed, and impossible to delete for a fixed term.** Choose a class with immediate access and appropriate economics, plus Object Lock compliance retention. Glacier Deep Archive would miss immediate access, while versioning alone does not prevent privileged version deletion.

## 30-Second Review

S3 stores objects with strong consistency. Choose classes by retrieval delay and total cost. Versioning preserves history; Object Lock protects versions. Lifecycle transitions or expires eligible data. Replication is asynchronous; existing objects need Batch Replication. KMS encryption needs key permissions. Presigned URLs delegate temporary access. CloudFront caches, Transfer Acceleration speeds transfers, and endpoints provide private network paths.

## Sources

- [Bucket overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html)
- [Storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [Lifecycle transitions](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html)
- [Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [What is replicated](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-what-is-isnot-replicated.html)
- [Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [Server-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/serv-side-encryption.html)
- [Event notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html)

Reviewed: 2026-09-22. Back to [[storage_overview|Storage]].
