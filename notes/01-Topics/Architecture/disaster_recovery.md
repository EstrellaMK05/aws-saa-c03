---
aliases:
  - Disaster Recovery
  - RPO and RTO
tags:
  - aws/saa
  - architecture
---

# Disaster Recovery — RPO, RTO and Strategies

## Mental Model

**RPO = how much recent data can we lose?**
**RTO = how long can the service be unavailable?**

These are business targets. A design meets them only when its measured recovery behavior is good enough.

## Core

### RPO and RTO are independent

| Objective | Meaning | Example |
|---|---|---|
| Recovery Point Objective (RPO) | Maximum acceptable data loss, measured in time | RPO of 15 minutes means recovering data to within 15 minutes before disruption |
| Recovery Time Objective (RTO) | Maximum acceptable time to restore service after disruption | RTO of 1 hour means the required service must be restored within that hour |

Hourly snapshots alone cannot satisfy a 5-minute RPO in the worst case. A current replica may satisfy a small RPO but still miss a short RTO if infrastructure rebuild, promotion, validation and traffic switching take too long.

RTO concerns the **usable application**, not just a database restore job. RPO concerns the **recoverable data state**, not simply whether a backup job ran.

### The four strategies

These are common **cross-Region DR** patterns. Relative cost and recovery speed depend on implementation; the names do not guarantee fixed RPO/RTO values.

| Strategy | What exists before a disaster? | What happens during recovery? | Trade-off |
|---|---|---|---|
| Backup and restore | Backups, templates and required artifacts | Provision infrastructure, restore data, validate, route traffic | Usually lowest idle cost; longest recovery work |
| Pilot light | Essential core, commonly replicated data; application capacity is not fully running | Start or deploy missing application components and scale | Less running capacity; more recovery steps |
| Warm standby | A complete, functional application at reduced capacity | Scale up and redirect traffic | Higher idle cost; less startup work |
| Multi-site active-active | Multiple sites actively serve production traffic | Route around the failed site and handle extra load | High cost and complexity; potentially very short recovery |

**Pilot light: start missing parts. Warm standby: scale an already functional system.**

### Design the entire recovery path

1. **Data:** backup or replication method, lag, retention and a usable recovery point.
2. **Infrastructure:** reproducible networks, compute, configuration and deployment artifacts.
3. **Dependencies:** accessible IAM roles, KMS keys, secrets, certificates and required external services.
4. **Capacity:** quotas, startup time and capacity available in the recovery location.
5. **Cutover:** health checks, database promotion where needed, application validation and traffic routing.
6. **Operations:** tested runbook, controlled failback and reconciliation of writes after recovery.

Infrastructure as code helps recreate resources; it does not automatically restore application data. See [[backup_and_restore]] and [[resilient_design_patterns]].

## Comparisons

| Mechanism | Helps with | Does not automatically solve |
|---|---|---|
| RDS Multi-AZ | Regional HA across AZs | Regional disaster recovery |
| Cross-Region database replication | Low data lag in another Region | Application deployment, historical recovery or traffic cutover |
| Route 53 failover routing | Directing new DNS resolutions toward a healthy endpoint | Copying data, creating capacity or instantly moving existing connections |
| Backups / PITR | Recovering an earlier valid data state | Keeping production continuously available |
| Active-active applications | Serving traffic in more than one location | Conflict-free writes or zero data loss by default |

### Service selection clues

- **Aurora Global Database:** consider for compatible relational workloads needing cross-Region replication and rapid recovery. Cross-Region replication is asynchronous; distinguish planned switchover from unplanned failover. Database recovery does not equal application recovery. See [[aws_aurora|Aurora]].
- **DynamoDB global tables:** consider for supported multi-Region key-value workloads. Consistency mode matters: MREC and MRSC have different guarantees and constraints. Do not assume every global table is eventually consistent or that every deployment gives zero data loss. See [[aws_dynamodb|DynamoDB]].
- **Cross-Region backups:** useful when the business accepts restore time and needs lower idle cost. They can also complement faster replication-based strategies.

## Exam Traps

- **Replication is not a substitute for backups.** Bad writes or deletion can propagate to replicas.
- **Active-active does not inherently mean RPO = 0.** Data guarantees depend on the storage and replication design.
- **A small RTO does not imply a small RPO.** Fast recovery can still recover older data.
- **The least expensive strategy must still meet both targets.** Backup and restore is not correct merely because it costs less.
- **DNS failover is not instant.** Detection and resolver/client caching affect traffic movement.
- **Do not memorize universal recovery times for strategy names.** Workload size, automation and testing determine actual results.
- **A regional standby can be healthy but inaccessible.** Keys, permissions and other dependencies must work during the disaster.

## Scenario Check

**The question states that a small but fully functional application already runs in the recovery Region. It must increase capacity during a disaster.** This is warm standby. If only the essential core were running and application servers still needed to start, it would be pilot light.

**A workload requires an RPO of 5 minutes. Daily backups are proposed.** Reject backups alone: a disaster shortly before the next backup could lose almost a day of changes. Faster backup/PITR or replication capabilities must meet the stated objective, and recovery must be tested.

## 30-Second Review

RPO limits lost data; RTO limits time to restore usable service. Backup and restore rebuilds, pilot light starts missing components, warm standby scales a functional system, and active-active already serves traffic in multiple locations. Strategy names do not guarantee times or zero loss. Recover data, infrastructure, dependencies and traffic together. Keep backups alongside replication and test recovery before trusting the targets.

## Sources

- [AWS disaster recovery strategies](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)
- [Aurora Global Database disaster recovery](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-disaster-recovery.html)
- [DynamoDB global table consistency modes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/V2globaltables_HowItWorks.html)
- [Route 53 DNS failover](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)

Reviewed: 2026-09-21. Back to [[architecture_overview|Architecture]].
