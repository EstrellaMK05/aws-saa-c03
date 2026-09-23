---
aliases: [Amazon EFS, Elastic File System]
tags: [aws/saa, storage]
---

# Amazon EFS — Shared NFS Files

## Mental Model

**Many clients mount the same persistent filesystem. Capacity grows with files; throughput is a separate decision.**

EFS suits shared Linux/POSIX file access, including compatible EC2, ECS and Lambda workloads.

## Core

### Availability and access

- **Regional:** redundant storage across AZs; use for shared files in a resilient Multi-AZ application.
- **One Zone:** data in one AZ; lower-cost option when that failure risk and recovery plan are acceptable.
- Mount targets provide private network access. For Regional EFS, place a mount target in each client AZ to favor local access; one per AZ is sufficient, not one per client subnet.
- Allow NFS **TCP 2049** from appropriate client security groups; check routing and network ACLs too.
- On-premises clients can access EFS over suitable private connectivity, such as VPN or Direct Connect, with correct routing/DNS.

### Four independent choices

| Dimension | Choices | Question answered |
|---|---|---|
| Filesystem type | Regional / One Zone | What failure scope is tolerated? |
| Storage class | Standard / IA / Archive, where supported | How frequently are these files accessed? |
| Performance mode | General Purpose / legacy Max I/O | What latency and I/O characteristics apply? |
| Throughput mode | Elastic / Provisioned / Bursting | How much data can move per second? |

**General Purpose is the current recommendation.** Max I/O is a previous-generation mode with higher per-operation latency; “hundreds of clients” alone does not justify it.

| Throughput mode | Choose when | Important behavior |
|---|---|---|
| Elastic | Demand is spiky, variable or difficult to predict | Automatically adjusts with activity; no burst-credit management |
| Provisioned | You know the required sustained throughput | Specify throughput independent of stored size and burst credits |
| Bursting | Size-based baseline and burst credits fit the workload | Baseline depends on data in Standard storage; sustained demand can exhaust credits |

All clients share filesystem throughput. The amount stored is not a measurement of read/write demand.

### Storage classes and lifecycle

Standard fits frequently accessed files. IA and Archive reduce storage cost for colder data, with access charges and different latency/economic trade-offs. Archive is supported on Regional filesystems using Elastic throughput.

Lifecycle policies move eligible files between classes, including a configured return to Standard on access. They **do not implement “delete after 30 days.”** EFS Archive remains filesystem-accessible; do not import S3 Glacier restore-job rules into EFS.

### Security and protection

Use KMS encryption at rest, TLS in transit, appropriate IAM/filesystem policies and POSIX permissions. Network reachability does not automatically grant file access.

**Access points** can enforce an application-specific root directory and POSIX identity. Combine them with authorization rules if applications must be restricted to their assigned entry points; merely creating an access point does not remove all alternate access paths.

Use AWS Backup for historical recovery. EFS replication can support a separate recovery filesystem, including cross-Region designs; it is asynchronous and is not a replacement for backup history. See [[backup_and_restore]].

## Comparisons

| Need | Choose |
|---|---|
| Generic shared NFS files | EFS |
| Persistent block disk | [[aws_ebs|EBS]] |
| Native Windows SMB/AD | [[aws_efx|FSx for Windows]] |
| Specialized HPC parallel filesystem | [[aws_efx|FSx for Lustre]] |
| API-accessed objects | [[aws_s3|S3]] |

## Exam Traps

- **Performance mode ≠ throughput mode.** General Purpose can be paired with an appropriate throughput mode.
- **Under 1 TB does not mean low throughput demand.** Frequent reads of the same files can move far more data than the stored capacity.
- **Provisioned and Elastic both decouple throughput from dataset size.** Use predictability and the answer choices to distinguish them.
- **One Zone is not Multi-AZ resilience**, even if clients run in several AZs.
- **A mount target is a network entry point, not a separate copy of the filesystem.**
- **EFS lifecycle is tiering, not expiration.**

## Scenario Check — ECS Outputs

**Hundreds of ECS tasks write roughly 20 MB each, share persistent output/state files, and retain less than 1 TB.** EFS addresses shared filesystem access. If the choices contrast Bursting with a known sustained throughput requirement, Provisioned avoids dependence on size/credits. If demand is unpredictable and Elastic is offered, evaluate Elastic. Neither 20 MB nor task count alone specifies the required MB/s without a time interval and access pattern.

## 30-Second Review

EFS supplies shared NFS files over TCP 2049. Regional protects across AZs; One Zone accepts an AZ dependency. Choose General Purpose, then select Elastic for variable demand, Provisioned for known throughput, or Bursting for size-based performance. Access points control application entry identities/directories with appropriate policies. Lifecycle tiers files; backups preserve history. Small datasets can still need high throughput.

## Sources

- [EFS overview](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
- [Performance and throughput](https://docs.aws.amazon.com/efs/latest/ug/performance.html)
- [Storage features](https://docs.aws.amazon.com/efs/latest/ug/features.html)
- [Mount targets](https://docs.aws.amazon.com/efs/latest/ug/manage-fs-access-create-delete-mount-targets.html)
- [Access points](https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html)
- [EFS with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/efs-volumes.html)

Reviewed: 2026-09-22. Back to [[storage_overview|Storage]].
