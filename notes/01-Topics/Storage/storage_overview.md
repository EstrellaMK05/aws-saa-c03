---
aliases: [Storage, Storage Decision Map]
tags: [aws/saa, storage]
---

# Storage — Decision Map

## Mental Model

**First choose the access model: blocks, files or objects. Then choose durability, latency, throughput and cost.**

Stored capacity does not tell you how quickly applications must read or write it.

## Core

| Requirement | Start with | Check |
|---|---|---|
| Persistent disk for EC2 | [[aws_ebs|EBS]] | AZ, IOPS, throughput and instance limits |
| Temporary host-local scratch data | EC2 instance store | Data must survive elsewhere if the host fails |
| Shared Linux/NFS files across compute clients | [[aws_efs|EFS]] | Regional vs One Zone, throughput and permissions |
| Windows SMB and Active Directory | [[aws_efx|FSx for Windows File Server]] | Deployment type and AD connectivity |
| HPC parallel filesystem for processing datasets | [[aws_efx|FSx for Lustre]] | Scratch vs persistent and S3 export |
| NetApp or OpenZFS compatibility | [[aws_efx|FSx ONTAP or OpenZFS]] | Required protocols and filesystem capabilities |
| API-accessed objects, backups or data lake | [[aws_s3|S3]] | Access frequency, restore delay and retention |
| Existing on-premises file/block/tape interface with cloud backing | [[aws_storage_gateway|Storage Gateway]] | Protocol and where the full data resides |
| Transfer existing datasets or synchronize changes | [[aws_datasync|DataSync]] | Network capacity, metadata and verification |
| Partners retain SFTP/FTPS/AS2 workflows | [[aws_transfer_family|Transfer Family]] | Protocol/backend compatibility and identity |

## Comparisons

| Dimension | Question to ask |
|---|---|
| Capacity | How much data is stored, and for how long? |
| IOPS | How many operations per second? |
| Throughput | How many bytes per second? |
| Latency | How long does each operation take? |
| Availability | Can clients access data through the stated failure? |
| Durability | Will data remain preserved? |
| Recovery | Can we restore an earlier valid state? |

Small random database operations often emphasize IOPS/latency. Large sequential transfers emphasize throughput. Always evaluate both service and client/network limits.

## Exam Traps

- **Durability is not availability.** One Zone storage has a different failure scope from Regional storage.
- **Replication is not historical recovery.** Keep appropriate backups/version history for logical mistakes.
- **Shared block access is not a managed shared filesystem.** EBS Multi-Attach requires coordinated writes and stays within one AZ.
- **Cheap storage can have expensive access.** Include retrieval, minimum-duration, request and transfer charges.
- **An archive class may require restoration before reading.** The word “Glacier” alone does not identify retrieval latency.
- **Moving data and providing ongoing access are different requirements.** DataSync transfers; Gateway presents an interface/cache; Transfer Family serves transfer protocols.

## Scenario Check

**Hundreds of ECS tasks need common persistent files, with a small stored dataset but frequent writes.** EFS fits shared file access. Select throughput separately: provision known sustained needs or use Elastic for variable demand. “Less than 1 TB” alone does not specify required MB/s.

## 30-Second Review

EBS is a disk; EFS is shared NFS; FSx supplies specialized filesystems; S3 stores objects. Match protocol before performance. Distinguish capacity, IOPS, throughput and latency. Gateway supports ongoing hybrid access, DataSync moves datasets, and Transfer Family supports partner protocols. Include retrieval and retention costs, check AZ resilience, and keep recoverable copies of important data.

## Study Order

1. [[aws_s3]] → [[aws_ebs]] → [[aws_efs]]
2. [[aws_efx|FSx]]
3. [[aws_storage_gateway]] → [[aws_datasync]] → [[aws_transfer_family]]

Related: [[backup_and_restore]], [[disaster_recovery]].

## Sources

- [SAA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html)
- Official service references appear in the linked notes.

Reviewed: 2026-09-22.
