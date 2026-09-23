---
aliases: [Amazon EBS, Elastic Block Store]
tags: [aws/saa, storage]
---

# Amazon EBS — Persistent Block Storage

## Mental Model

**EBS is a persistent disk in one AZ. EC2 supplies compute; the volume supplies blocks.**

You configure the filesystem/database on top. Persistence does not replace backups or protect against every deletion setting.

## Core

### Choose by the I/O pattern

| Type | Main fit | Distinction |
|---|---|---|
| gp3 | General-purpose SSD, boot and common application/database disks | Configure capacity, IOPS and throughput separately within supported limits |
| gp2 | Earlier general-purpose SSD | Performance is more closely tied to size and, for smaller volumes, credits |
| io2 | Demanding databases requiring sustained IOPS and low latency | Higher durability/performance requirements; verify instance support |
| io1 | Earlier provisioned-IOPS SSD | Recognize existing workloads; consider io2 capabilities |
| st1 | Large sequential, frequently accessed throughput workloads | HDD; not a boot volume |
| sc1 | Infrequent sequential access with low storage cost priority | HDD; not latency-sensitive random I/O or a boot volume |

Do not buy more GiB merely to obtain gp3 performance. Check the EC2 instance's EBS bandwidth/IOPS ceilings: a faster volume cannot bypass them.

### Lifecycle and modification

- Attach a volume to a compatible instance in the **same AZ**.
- Stopping an EBS-backed instance normally retains its volumes. Termination follows each volume's **DeleteOnTermination** setting.
- Elastic Volumes can change supported size/type/performance settings while attached. Increasing volume capacity may also require extending the partition/filesystem inside the OS.
- Volumes cannot simply be shrunk in place. Plan a new smaller volume and migrate data if needed.
- Encryption uses KMS and covers encrypted-volume data, associated snapshots and traffic between the instance and volume. Recovery still requires usable keys and permissions.

### Snapshots and recovery

Snapshots are incremental backups managed by EBS. Deleting an older snapshot does not invalidate later snapshots; AWS retains blocks still needed by remaining snapshots.

Create a new volume from a snapshot in another AZ to move data across AZs. For another Region, copy the snapshot there first. Snapshots are not ordinary objects in your own S3 bucket.

Coordinate writes or use appropriate application-aware procedures for consistent database backups. Restored volumes can experience initialization overhead; **Fast Snapshot Restore** can supply fully initialized volumes for enabled snapshots/AZs at additional cost. Choose it for a stated immediate-performance requirement, not every restore.

### Multi-Attach

Supported io1/io2 configurations can attach one volume to multiple compatible Nitro instances **within the same AZ**. The application/clustered filesystem must coordinate writes. Mounting a normal ext4/XFS filesystem read-write from multiple independent hosts risks corruption. Multi-Attach does not turn EBS into EFS or protect the volume from an AZ outage.

## Comparisons

| Storage | Best mental model | Key boundary |
|---|---|---|
| EBS | Persistent block disk | AZ-scoped attachment |
| Instance store | Fast temporary local disk | Data can disappear with stop/termination/host loss |
| EFS | Shared NFS files | Managed concurrent filesystem access |
| S3 | Objects accessed through APIs | Not a normal attached block device |

## Exam Traps

- **IOPS and throughput are different.** Many small operations and large sequential transfers have different bottlenecks.
- **gp3 performance does not require increasing storage size.** Respect configuration ratios and service limits.
- **st1/sc1 cannot be boot volumes.** Choose SSD for low-latency random I/O.
- **EBS is not directly attachable across AZs.** Restore a snapshot into the required AZ.
- **Persistent does not mean retained on termination.** Check DeleteOnTermination.
- **Growing the volume does not automatically grow every filesystem.** Complete guest-side steps.
- **A snapshot is a recovery point, not live failover.** Restore and application startup take time.

## Scenario Check

**A database needs more disk IOPS without more capacity, and its current instance supports the required performance.** Adjust gp3 IOPS if the requirement fits its capabilities, or evaluate io2 for stricter sustained performance/durability needs. Increasing volume size blindly or selecting HDD does not address the actual requirement.

## 30-Second Review

EBS provides persistent block storage in one AZ. gp3 separates capacity from performance; io2 fits demanding sustained I/O; st1/sc1 fit sequential HDD workloads. Check instance limits and DeleteOnTermination. Snapshots support recovery into new volumes and other AZs. Multi-Attach needs coordinated writes and stays in one AZ. Extend the filesystem after increasing capacity when required.

## Sources

- [Volume types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- [I/O characteristics](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-io-characteristics.html)
- [Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html)
- [Snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)
- [Modify a volume](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-modify-volume.html)
- [Fast Snapshot Restore](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-fast-snapshot-restore.html)

Reviewed: 2026-09-22. Back to [[storage_overview|Storage]].
