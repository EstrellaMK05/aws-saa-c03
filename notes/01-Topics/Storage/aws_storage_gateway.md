---
aliases: [AWS Storage Gateway, Volume Gateway, S3 File Gateway, Tape Gateway]
tags: [aws/saa, storage]
---

# AWS Storage Gateway — Hybrid Storage Interfaces

## Mental Model

**Keep an existing file, block or tape interface while integrating with AWS storage. Local cache/buffers bridge the network.**

Choose the gateway by the application protocol and where the complete dataset must reside.

## Core

| Gateway | Application interface | Where data lives / use case |
|---|---|---|
| S3 File Gateway | NFS or SMB file shares | Files stored as S3 objects; local cache supports frequently used data |
| Volume Gateway — cached | iSCSI block volumes | Primary data stored in AWS; working data cached locally |
| Volume Gateway — stored | iSCSI block volumes | Full primary dataset local; asynchronous cloud backups as EBS snapshots |
| Tape Gateway | iSCSI virtual tape library | Preserve compatible backup-software workflows while replacing physical tapes |

### S3 File Gateway

Applications use file shares; the gateway maps files to S3 objects and uploads asynchronously. This is useful when applications need NFS/SMB plus ongoing local cached access, rather than being rewritten to call S3 APIs.

Direct changes to the backing bucket may need a cache refresh before the gateway sees them. Avoid treating multiple independent writers/caches as a coordinated distributed filesystem.

S3 lifecycle can archive colder objects, but objects moved to restore-required archive classes cannot be read normally through the share until restored and made accessible appropriately. Verify archival policy against application access needs.

### Volume Gateway

**Cached = most data in AWS; stored = all primary data local.** Both present block devices, not browseable S3 file shares. EBS snapshots support volume recovery; cached-volume cloud backing is not an ordinary bucket of user-visible files.

Stored volumes fit applications requiring low-latency access to the entire local dataset while maintaining cloud backups. Cached volumes reduce the amount of local capacity needed when the active working set is smaller.

### Tape Gateway

Compatible backup software writes virtual tapes. Archiving ejected tapes provides long-term retention in supported archival storage. Retrieve a tape before using archived content again; archival retrieval is not immediate local-disk access.

### Operations and current availability

Size local cache/upload buffers, provide appropriate network bandwidth, and monitor pending uploads. A successful local write does not prove the data has already reached AWS; consider gateway/site failure during that interval.

**FSx File Gateway is no longer available to new customers.** This is separate from S3 File Gateway and from FSx for Windows File Server. Recognize the older offering in existing-environment questions without recommending it as universally available for a new deployment.

## Comparisons

| Requirement | Service |
|---|---|
| Ongoing on-premises NFS/SMB with S3 backing and cache | S3 File Gateway |
| On-premises block interface with cloud storage/backup | Volume Gateway |
| Preserve virtual-tape backup workflows | Tape Gateway |
| Copy or synchronize a dataset | [[aws_datasync|DataSync]] |
| Partner uploads over SFTP | [[aws_transfer_family|Transfer Family]] |
| Direct shared filesystem in AWS | [[aws_efs|EFS]] or [[aws_efx|FSx]] according to protocol |

## Exam Traps

- **NFS/SMB → File Gateway; iSCSI disk → Volume Gateway; VTL → Tape Gateway.**
- **Cached and stored are not synonyms.** Identify whether the whole primary dataset stays on premises.
- **Volume Gateway does not expose user files as S3 objects.** That is the S3 File Gateway model.
- **Cache is not unlimited offline storage or an independent backup.** Consider misses, unuploaded data and failure recovery.
- **DataSync is not a persistent local cache appliance.**

## Scenario Check

**An on-premises application uses iSCSI and requires low-latency access to its entire dataset, with cloud backups.** Choose Volume Gateway stored volumes. Cached volumes prioritize reducing local capacity; File Gateway supplies the wrong interface.

## 30-Second Review

Storage Gateway preserves hybrid application interfaces. S3 File Gateway translates NFS/SMB into S3 objects with cache. Volume Gateway uses iSCSI: cached keeps the working set local, stored keeps all primary data local with cloud snapshots. Tape Gateway replaces tape infrastructure. Uploads can be asynchronous. Match protocol and data location, then check bandwidth, cache and archive retrieval behavior.

## Sources

- [S3 File Gateway](https://docs.aws.amazon.com/filegateway/latest/files3/WhatIsFileGateway.html)
- [Volume Gateway architecture](https://docs.aws.amazon.com/storagegateway/latest/vgw/StorageGatewayConcepts.html)
- [Tape Gateway](https://docs.aws.amazon.com/storagegateway/latest/tgw/WhatIsStorageGateway.html)
- [FSx File Gateway availability](https://docs.aws.amazon.com/filegateway/latest/filefsxw/create-file-gateway.html)

Reviewed: 2026-09-22. Back to [[storage_overview|Storage]].
