---
aliases: [AWS Transfer Family, Managed File Transfer]
tags: [aws/saa, storage]
---

# AWS Transfer Family

## Mental Model

**Keep the partner's file-transfer protocol; let AWS manage the transfer endpoint and place data in the supported storage backend.**

Choose protocol, identity, network exposure and storage together.

## Core

| Protocol | Meaning | Storage distinction |
|---|---|---|
| SFTP | File transfer over SSH | Supported S3 or EFS server configurations |
| FTPS | FTP secured with TLS | Supported S3 or EFS server configurations |
| FTP | Unencrypted FTP | Supported configurations; restrict to appropriate private networking |
| AS2 | B2B message exchange, signing/encryption and delivery receipts | S3 backend; not EFS |

Transfer Family also has browser-based and connector capabilities. Learn the server/protocol/backend distinction first; do not assume every feature supports every protocol.

### S3 vs EFS

- **S3:** object workflows, downstream event processing, lifecycle transitions and expiration. Configure encryption and retention for the business requirement.
- **EFS:** shared filesystem semantics and POSIX ownership/permissions for applications reading the same files.
- A transfer endpoint does not make object storage behave exactly like every POSIX filesystem operation.

### Identity and network access

Choose supported identity-provider options for the protocol: service-managed users or suitable external/custom integration where supported. Authentication identifies the user; their associated IAM/storage permissions determine access. EFS also needs correct POSIX identities and permissions.

Endpoint support varies by protocol. Use appropriate VPC-hosted endpoints and routing/security controls for private access or supported address restrictions. Do not assume a public endpoint supports every protocol or that “VPC-hosted” always means internet-inaccessible.

Separate **encryption in transit** from **encryption at rest**. SFTP protects the transfer using SSH; backend encryption and KMS permissions are separate settings.

### Processing and retention

Managed workflows can perform supported post-upload steps such as copying, tagging or custom processing. Check protocol support: AS2 messages do not execute workflows merely because a workflow is attached to the server.

For S3, use lifecycle for age-based transitions/expiration and Object Lock for immutable retention. On a versioned bucket, include noncurrent versions in the deletion design. EFS lifecycle tiers files; it does not implement automatic age-based deletion.

## Comparisons

| Requirement | Choice |
|---|---|
| Partners must keep SFTP, with minimal server administration | Transfer Family |
| On-premises applications need SMB/NFS with local cache | [[aws_storage_gateway|S3 File Gateway]] |
| Copy an existing file-server dataset and periodic changes | [[aws_datasync|DataSync]] |
| Temporary browser/client access to an S3 object without an SFTP requirement | Consider an S3 presigned URL |
| Custom transfer-server behavior outside managed capabilities | Evaluate self-managed compute and its operational cost |

## Exam Traps

- **SFTP is not FTP over TLS.** SSH → SFTP; TLS → FTPS.
- **AS2 does not use EFS as its backend.** The broad “S3 or EFS” statement is protocol-dependent.
- **Encryption does not grant authorization.** Identity, IAM and backend permissions still matter.
- **S3 lifecycle expiration does not delete all retained versions automatically.** Check versioning and Object Lock.
- **EFS lifecycle does not mean delete after N days.**
- **Transfer Family is not a local cache or general dataset-synchronization engine.**

## Scenario Check

**Partners upload over SFTP; files must be encrypted at rest and removed after an allowed retention period with little server administration.** Use Transfer Family with S3, appropriate encryption/permissions and lifecycle rules covering relevant versions. Do not choose EFS lifecycle for expiration, and do not delete data earlier than Object Lock/compliance rules allow.

## 30-Second Review

Transfer Family manages file-transfer endpoints. SFTP uses SSH; FTPS uses TLS; AS2 supports B2B exchanges with S3, not EFS. Choose a compatible backend, identity provider and endpoint exposure. S3 lifecycle can expire eligible objects; EFS lifecycle only tiers files. Permissions and encryption are separate. DataSync migrates datasets, while Storage Gateway provides ongoing hybrid interfaces and cache.

## Sources

- [Transfer Family overview](https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html)
- [AS2 capabilities](https://docs.aws.amazon.com/transfer/latest/userguide/create-b2b-server.html)
- [AS2 server and workflow restrictions](https://docs.aws.amazon.com/transfer/latest/userguide/create-as2-transfer-server.html)
- [Endpoint types](https://docs.aws.amazon.com/transfer/latest/userguide/create-server-in-vpc.html)

Reviewed: 2026-09-22. Back to [[storage_overview|Storage]].
