---
aliases: [Amazon FSx, FSx, aws_fsx]
tags: [aws/saa, storage]
---

# Amazon FSx — Specialized Filesystems

## Mental Model

**Choose the filesystem technology the application requires, then choose a deployment that meets availability and performance needs.**

The FSx family is not one interchangeable filesystem.

## Core

| Service | Main capability | Strong clue |
|---|---|---|
| FSx for Windows File Server | Native Windows files, SMB, AD integration | Windows shares, NTFS permissions, Windows application compatibility |
| FSx for Lustre | Parallel high-performance filesystem | HPC, simulations and large-scale dataset processing |
| FSx for NetApp ONTAP | ONTAP data management; NFS, SMB and block protocols | Existing NetApp environment, SnapMirror, multiprotocol requirements |
| FSx for OpenZFS | Managed ZFS-based NFS with snapshots/cloning | Existing ZFS or applications requiring its capabilities |

### Windows File Server

Integrate with supported Active Directory options and preserve required DNS/network access. Select Multi-AZ when automatic failover across AZs is required. A Windows-compatible share is not automatically permissioned correctly: AD identity, share permissions and file ACLs matter.

SMB access is not restricted to Windows clients if another client supports the protocol and authentication. For generic Linux NFS shared files without Windows requirements, compare [[aws_efs|EFS]].

### Lustre and S3

Lustre provides fast parallel file access to compute clients. Supported S3 data-repository associations connect object datasets to filesystem processing. Configure import/export behavior and verify needed outputs reach S3; a linked bucket is not proof every new file has already been exported.

- **Scratch:** temporary processing where failed/lost data can be recreated; no durable replication of the scratch filesystem's data.
- **Persistent:** replicated storage and failure recovery for longer-lived workloads; still evaluate deployment failure scope and backups.

Lustre deployments do not become Multi-AZ simply because they are called persistent. S3 integration and backup capabilities also depend on deployment/version configuration.

### ONTAP and OpenZFS

ONTAP fits NetApp migrations and workloads needing its multiprotocol/data-management model. Common SAA protocols include NFS, SMB and iSCSI; confirm additional protocol support against the selected configuration. Storage tiering does not turn its capacity pool into an ordinary customer S3 bucket that applications browse directly.

OpenZFS fits NFS workloads that benefit from ZFS snapshots, clones and compatible semantics. Choose the supported deployment and performance configuration; do not assume every FSx family has identical resilience options.

## Comparisons

| Requirement | Better starting point |
|---|---|
| Elastic general-purpose shared NFS | EFS |
| Windows-native SMB and AD | FSx for Windows |
| Parallel HPC access to S3-backed datasets | FSx for Lustre |
| NetApp replication and protocol compatibility | FSx for ONTAP |
| ZFS-native capabilities and NFS | FSx for OpenZFS |
| Application needs a block device rather than files | EBS, or a specifically required supported block protocol |

## Exam Traps

- **“High performance” alone does not select Lustre.** Look for parallel filesystem/HPC requirements.
- **“NFS” alone does not select EFS.** ONTAP or OpenZFS may be required by compatibility constraints.
- **Persistent Lustre is not synonymous with cross-AZ failover.**
- **S3 integration is not instant synchronization of every change.** Configure and verify data movement.
- **Snapshots and HA solve different problems.** Replication can preserve availability while a backup preserves an older valid state.
- **FSx File Gateway and FSx for Windows are different offerings.** A gateway's availability status does not imply the filesystem service is discontinued.

## Scenario Check

**A company processes a large S3 dataset using parallel Linux HPC jobs and needs filesystem access.** Evaluate FSx for Lustre with a supported S3 association. Use scratch only if data can be recreated; preserve required outputs before removing temporary resources.

## 30-Second Review

FSx Windows means SMB and AD; Lustre means parallel HPC and optional S3 integration; ONTAP means NetApp and multiprotocol data management; OpenZFS means ZFS capabilities over NFS. Choose deployment resilience separately. Scratch data must be recreatable. Persistent does not universally mean Multi-AZ. Verify S3 exports, permissions and backups instead of assuming a managed filesystem handles every recovery need.

## Sources

- [FSx for Windows](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)
- [Lustre deployment options](https://docs.aws.amazon.com/fsx/latest/LustreGuide/using-fsx-lustre.html)
- [Lustre S3 associations](https://docs.aws.amazon.com/fsx/latest/LustreGuide/create-dra-linked-data-repo.html)
- [FSx for ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html)
- [FSx for OpenZFS](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/what-is-fsx.html)

Reviewed: 2026-09-22. Filename retained for existing vault compatibility; the service name is **FSx**. Back to [[storage_overview|Storage]].
