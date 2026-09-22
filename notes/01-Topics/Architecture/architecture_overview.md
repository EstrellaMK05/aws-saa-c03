---
aliases:
  - Architecture
  - Resilient Architectures
tags:
  - aws/saa
  - architecture
---

# Architecture — Decision Map

## Mental Model

Design for a **specific failure**, then choose the least expensive architecture that meets the business requirements.

**Failure scope → acceptable data loss → acceptable downtime → capacity → cost.**

## Core

| Requirement in the question | Start with | Check before choosing |
|---|---|---|
| Survive an instance or Availability Zone failure | [[high_availability|Multi-AZ high availability]] | Every critical tier and dependency must survive; remaining capacity must be sufficient |
| Recover after a Regional outage | [[disaster_recovery|Cross-Region disaster recovery]] | Data replication, application recovery, traffic routing, RPO and RTO |
| Recover from accidental deletion or corruption | [[backup_and_restore|Backups and point-in-time recovery]] | Retention, isolation, encryption keys and tested restores |
| Absorb bursts without losing accepted work | [[resilient_design_patterns|Queues and independent workers]] | Retention, duplicate processing, retries and backlog |
| Add application capacity easily | [[resilient_design_patterns|Stateless horizontal scaling]] | External session state and database capacity |
| Keep a failing dependency from taking down everything | [[resilient_design_patterns|Failure isolation]] | Timeouts, bounded retries, circuit breakers and separate resource pools |

### Read the constraints first

1. Identify the failure scope: server, AZ, Region, account, or logical data damage.
2. Extract **RPO** (acceptable data loss) and **RTO** (acceptable recovery time).
3. Check consistency, latency, throughput and data residency requirements.
4. Prefer managed services when they meet the requirements and reduce operational effort.
5. Compare cost only among solutions that satisfy the requirements.

### Reference architecture

An internet-facing ALB distributes traffic to an Auto Scaling group across multiple AZs. Application instances keep durable data outside the instances. The database has an appropriate Multi-AZ deployment. A queue separates slow background work from requests. Backups provide historical recovery points; a separate cross-Region strategy addresses Regional failure when required.

Related service notes: [[aws_auto_scaling|Auto Scaling]], [[aws_ec2|EC2]], [[aws_rds|RDS]], [[aws_aurora|Aurora]], [[aws_vpc|VPC]], [[aws_s3|S3]].

## Comparisons

| Concept | Question it answers | Does not automatically provide |
|---|---|---|
| High availability | Can the service continue through expected component failures? | Regional recovery or historical data recovery |
| Fault tolerance | Can it continue through a defined failure without interruption? | Protection from every possible failure |
| Disaster recovery | How do we restore service after a disaster? | Zero downtime or zero data loss |
| Durability | How reliably is stored data preserved? | Immediate access to that data |
| Scalability | Can capacity grow with demand? | Automatic scaling or redundancy |
| Elasticity | Can capacity expand and contract with demand? | Elimination of downstream bottlenecks |

## Exam Traps

- **Multi-AZ and Multi-Region answer different failure scopes.** Do not add a second Region unless the requirements justify it.
- **A managed service does not remove architecture choices.** Deployment mode, backups, scaling limits and application behavior still matter.
- **The strongest individual component does not determine application availability.** A required single-AZ dependency can still stop the workload.
- **Read replicas, backups and standby databases are not interchangeable.** Identify whether the question asks for read scale, historical recovery, or failover.
- **High durability is not high availability.** Preserved data can temporarily be inaccessible.

## Scenario Check

**An application must survive an AZ failure at the lowest practical cost.** Start with a complete Multi-AZ design. Multi-Region active-active adds cost and complexity beyond the stated failure scope. Backups alone require restoration and do not keep the application serving traffic during the failure.

## 30-Second Review

Start with the failure, not the service name. Multi-AZ handles AZ failures; cross-Region DR handles Regional disasters; backups handle historical recovery. Stateless applications scale and recover more easily. Queues absorb bursts, and failure isolation limits cascading outages. Check every dependency and remaining capacity. Choose the lowest-cost option that actually meets RPO, RTO and workload requirements.

## Study Order

1. [[high_availability]]
2. [[disaster_recovery]]
3. [[backup_and_restore]]
4. [[resilient_design_patterns]]

## Sources

- [SAA-C03 Domain 2: Design Resilient Architectures](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03-domain2.html)
- [AWS Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)

Reviewed: 2026-09-21. Focus: SAA-C03 architecture decisions, not implementation runbooks.
