---
aliases:
  - Backup and Restore
  - AWS Backup
tags:
  - aws/saa
  - architecture
---

# Backups and Restore — Recover a Valid Data State

## Mental Model

**Replication gives another live copy. Backups give a way back in time.**

A recovery point is useful only if it survives the incident, remains decryptable, and can be restored within the business requirements.

## Core

### Match protection to the failure

| Failure | Protection to consider | Why |
|---|---|---|
| Instance or AZ failure | Redundant live resources and appropriate replication | Continue service or fail over quickly |
| Accidental overwrite or logical corruption | Versioning, snapshots or supported point-in-time recovery (PITR) | Recover a state from before the mistake |
| Regional disaster | Usable copies in another Region | A backup available only in the affected Region may not be accessible |
| Compromised production credentials | Separate account controls and appropriate immutable retention | Reduce the ability of one compromised identity to destroy all recovery copies |
| Accidental or malicious backup deletion | AWS Backup Vault Lock where supported and appropriate | Enforce backup retention rules |

Cross-Region and cross-account copies solve different isolation problems. A cross-account copy in the same Region does not remove the Regional dependency.

### AWS Backup

AWS Backup centralizes protection for supported resources:

- A **backup plan** defines schedules, retention and optional copy rules.
- **Resource assignments** select what to protect, including supported tag-based selection.
- **Backup vaults** organize recovery points and provide access controls.
- **AWS Organizations backup policies** support central management across accounts.
- **Cross-Region / cross-account copies** improve isolation when supported by the resource type and configuration.

Check resource support, destination permissions and encryption requirements. Do not assume every resource supports every AWS Backup feature.

### Retention, frequency and restore testing

- **Frequency** helps determine how much data can be lost; **retention** determines how far back you can recover.
- PITR can provide finer recovery points than periodic snapshots for supported services and enabled configurations. The recoverable window and latest restorable time still matter.
- A restore commonly creates a replacement resource. Plan endpoint changes, permissions, validation and traffic cutover.
- Schedule restore tests. AWS Backup restore testing can automate supported restores; validate application correctness separately.

### Vault Lock: the distinction worth remembering

| Mode | Key distinction |
|---|---|
| Governance | Users with sufficient permissions can remove the lock |
| Compliance | After its grace period, the lock cannot be removed while protected recovery points remain; retention is enforced even against privileged users |

Locked backups still expire according to their configured retention. Immutability does not mean eternal storage or proof that the backed-up data is correct.

## Comparisons

| Option | Strength | Limitation |
|---|---|---|
| Live replica | Fast access to a recent copy | Can replicate logical mistakes |
| Snapshot / periodic backup | Historical recovery point | Changes after that point may be lost |
| PITR | Recover within a supported historical window | Requires restore time and a valid chosen timestamp |
| Infrastructure as code | Recreate resource configuration | Does not contain your live application data |
| Immutable backup retention | Resist premature backup deletion | Does not prevent production downtime or replace restore testing |

Related: [[disaster_recovery]], [[aws_s3|S3]], [[aws_rds|RDS]], [[aws_dynamodb|DynamoDB]].

## Exam Traps

- **Longer retention does not mean lower RPO.** Keeping daily backups for a year does not make them minute-by-minute backups.
- **A successful backup job does not prove a successful recovery.** Test decryption, restore, application data and dependencies.
- **Cross-Region is not cross-account.** Choose the isolation boundary the question requires.
- **Encryption needs a recovery plan for keys and permissions.** An unreadable backup cannot restore service.
- **Multi-AZ is not historical recovery.** It does not undo a valid but mistaken DELETE operation.
- **S3 Versioning alone is not immutable retention.** Versions can still be deleted by sufficiently authorized identities; use the appropriate retention controls when immutability is required.

## Scenario Check

**An operator deletes valid records, and the live replica receives the deletion.** Restore a known-good backup or use supported PITR to a time before the operation, then validate and reconcile necessary later changes. Failing over to the replica would expose the same logical damage.

## 30-Second Review

Backups protect historical states; replicas protect access to recent data. Match copy location to the disaster: another Region for Regional failure, another account for administrative isolation. Frequency affects RPO; retention controls recovery history. AWS Backup centralizes policies and recovery points. Vault Lock enforces retention. Keep keys accessible, test restores, and include application validation and cutover in recovery time.

## Sources

- [What is AWS Backup?](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [AWS Backup feature availability by resource](https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-feature-availability.html)
- [AWS Backup Vault Lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)
- [AWS Backup restore testing](https://docs.aws.amazon.com/aws-backup/latest/devguide/restore-testing.html)
- [Cross-account backups](https://docs.aws.amazon.com/aws-backup/latest/devguide/create-cross-account-backup.html)

Reviewed: 2026-09-21. Back to [[architecture_overview|Architecture]].
