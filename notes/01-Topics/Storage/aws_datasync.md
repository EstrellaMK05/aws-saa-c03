---
aliases: [AWS DataSync, Storage Migration]
tags: [aws/saa, storage]
---

# AWS DataSync — Move and Synchronize Data

## Mental Model

**Automate moving existing files or objects between supported storage locations.**

DataSync transfers datasets; it does not provide an ongoing mounted filesystem or replace network connectivity.

## Core

- Define source/destination **locations** and a **task** describing what to copy and how to verify it.
- Supported combinations include on-premises file/object storage and AWS storage such as S3, EFS and FSx. Confirm the exact endpoint/protocol combination.
- An **agent** is needed for certain locations, especially on-premises/self-managed storage. Do not assume every AWS-to-AWS transfer needs an agent.
- Use schedules, include/exclude filters, bandwidth controls and supported incremental transfers to fit migration windows.
- DataSync validates transferred data; verification modes and metadata preservation depend on task/location support.
- IAM/storage permissions, credentials, routing and DNS must all work. Direct Connect or VPN provides connectivity; DataSync provides the transfer workflow.

### Migration pattern

1. Measure data size, change rate and usable bandwidth.
2. Run the initial copy while the application continues operating when appropriate.
3. Repeat incremental transfers to reduce the remaining delta.
4. Quiesce writes or use an application-consistent cutover procedure.
5. Transfer final changes, verify content/permissions and switch clients.

A file transfer is not automatically a transactionally consistent live database migration. Use suitable database-native backup/replication or migration tooling for that requirement.

### Throughput reasoning

`ideal transfer time = data bits / usable bits per second`

Actual time is longer because of protocol overhead, small-file metadata operations, source/destination performance and ongoing changes. Installing faster transfer software cannot eliminate a physically insufficient network link.

### Offline-transfer questions

Older study material uses Snowball/Snowball Edge for large transfers over inadequate networks. Preserve the **offline physical-transfer concept**, but check product availability: AWS no longer offers Snow Family devices to new customers. Current options can include DataSync online, AWS Data Transfer Terminal or partner solutions, depending on location and requirements. Existing customers have different eligibility.

## Comparisons

| Requirement | Choice |
|---|---|
| Bulk copy / scheduled synchronization | DataSync |
| Ongoing SMB/NFS local cache backed by S3 | [[aws_storage_gateway|S3 File Gateway]] |
| Partners connect through SFTP/FTPS | [[aws_transfer_family|Transfer Family]] |
| Continually replicate eligible S3 writes between buckets | [[aws_s3|S3 replication]] |
| Dedicated private network connection | Direct Connect; combine with an appropriate transfer service |
| Object uploads from distant clients | Consider S3 Transfer Acceleration |

## Exam Traps

- **Transfer service ≠ connectivity service.** DataSync still needs a working path.
- **Synchronization ≠ zero-RPO replication.** Schedules and transfer time leave a recovery gap.
- **Copying files ≠ application consistency.** Coordinate actively changing applications.
- **Metadata support differs by source and destination.** Verify permissions and ownership after moving between file and object storage.
- **“No agent ever” and “agent always” are both too broad.** Check the locations.
- **Do not recommend unavailable hardware solely from an old exam mnemonic.**

## Scenario Check

**A company must migrate an on-premises NFS dataset to EFS, then copy nightly changes until cutover.** DataSync fits the initial and scheduled incremental transfers. Storage Gateway would address ongoing hybrid access instead; Transfer Family would provide partner transfer protocols rather than this dataset migration workflow.

## 30-Second Review

DataSync moves files and objects between supported locations. Configure locations, tasks, permissions, connectivity and verification; deploy an agent where required. Use initial plus incremental copies and a controlled final cutover. It is not a filesystem cache, a dedicated network link or automatically database-consistent replication. Match bandwidth to the migration window and verify current offline-transfer availability.

## Sources

- [DataSync overview](https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html)
- [Create transfer tasks](https://docs.aws.amazon.com/datasync/latest/userguide/create-task-how-to.html)
- [Data verification](https://docs.aws.amazon.com/datasync/latest/userguide/configure-data-verification-options.html)
- [Metadata and incremental transfers](https://docs.aws.amazon.com/datasync/latest/userguide/configure-metadata.html)
- [Snowball Edge availability change](https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html)

Reviewed: 2026-09-22. Back to [[storage_overview|Storage]].
