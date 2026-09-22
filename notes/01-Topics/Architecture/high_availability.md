---
aliases:
  - High Availability
  - Multi-AZ Architecture
tags:
  - aws/saa
  - architecture
---

# High Availability

## Mental Model

**Remove a component or an AZ. Can the remaining system still serve users?**

Redundancy is useful only when traffic can reach healthy resources, data remains accessible, and surviving resources have enough capacity.

## Core

### Protect every critical tier

| Tier | Typical design | What to verify |
|---|---|---|
| Traffic distribution | Load balancer with targets across AZs | Application health checks and healthy capacity |
| Compute | Auto Scaling group across multiple AZs | Desired capacity actually produces redundant instances; replacement takes time |
| Session state | External store accessible to all application instances | Store availability and acceptable loss of session data |
| Relational database | Appropriate RDS Multi-AZ or Aurora deployment | Failover behavior, replica placement and connection retries |
| Shared files | EFS Regional when shared Linux file access is needed | EFS One Zone has a different failure scope |
| Network dependencies | Resilient egress, routing and connectivity | Avoid a single zonal dependency for all AZs |

An ASG configured for multiple AZs with **only one running instance** is not a continuously redundant application tier. Scaling after failure also depends on startup time, quotas and available capacity.

### Load balancing and replacement are different jobs

- **Load balancer:** uses target health to decide where to send requests.
- **Auto Scaling:** maintains desired instance capacity and replaces instances it considers unhealthy.
- EC2 Auto Scaling uses EC2 health checks by default. **Enable ELB health checks on the ASG** when application health reported by the load balancer should trigger replacement.
- Allow startup time with suitable health-check settings. Applications must also reconnect or retry appropriately during database failover.

### Database deployment matters

| Deployment | Main use | Important distinction |
|---|---|---|
| RDS Multi-AZ DB instance | HA with a synchronous standby | Standby does not serve reads |
| RDS Multi-AZ DB cluster | HA with a writer and two readable instances across three AZs | Readable instances; engine and Region support differ from the DB instance option |
| RDS read replica | Read scaling; sometimes part of a DR design | Generally asynchronous; promotion and application redirection are separate concerns |
| Aurora with replicas across AZs | Fast failover to an existing replica and read scaling | Shared distributed storage does not remove the benefit of redundant compute |

See [[aws_rds|RDS]] and [[aws_aurora|Aurora]] for service detail.

### Network dependencies count

For **zonal NAT gateways**, one gateway in each relevant AZ with same-AZ routing avoids making all egress depend on one AZ. AWS also offers **Regional NAT gateways**; distinguish the deployment mode instead of memorizing “every NAT gateway is single-AZ.” Choose based on the question's architecture and requirements. See [[aws_vpc|VPC]].

## Comparisons

| Design | Failure it addresses | Limitation |
|---|---|---|
| Several instances in one AZ | Individual instance failure | Shared AZ failure remains |
| Complete Multi-AZ deployment | Instance and AZ failures | Regional failure remains |
| Cross-Region deployment | Regional failures, when fully prepared | Replication, routing and operational complexity |
| Backups without running recovery capacity | Historical recovery | Service waits for restore and rebuild |

## Exam Traps

- **An ALB does not create or replace EC2 instances.** That is the ASG's job.
- **EC2 status checks can pass while the application is broken.** ELB application checks must be integrated with ASG replacement when required.
- **“Multi-AZ never supports reads” is too broad.** It describes the standby of a Multi-AZ DB instance, not a Multi-AZ DB cluster.
- **Sticky sessions do not make local session data resilient.** If the instance fails, its local state may disappear.
- **A cache replica is not a backup.** Check persistence and replication behavior before treating a cache as durable session storage.
- **Failover is not necessarily interruption-free.** Connections can break and recovery takes time.
- **Redundancy without spare capacity can still cause an outage.** Remaining AZs must carry the workload or scale in time.

## Scenario Check

**The web process hangs, but its EC2 instance still passes status checks. The ALB marks it unhealthy, yet the ASG does not replace it.** Enable ELB health checks for the ASG. Increasing maximum capacity alone does not change which instances the ASG considers unhealthy.

## 30-Second Review

HA requires healthy alternatives across failure domains, enough surviving capacity, and resilient dependencies. ALB routes; ASG replaces. Enable ELB health checks when application failure should cause replacement. Keep session state outside individual instances. RDS Multi-AZ DB instance standbys are not readable, but Multi-AZ DB clusters have readable instances. Multi-AZ protects against AZ failure; Regional recovery needs a separate design.

## Sources

- [EC2 Auto Scaling health checks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-checks-overview.html)
- [RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [RDS Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html)
- [Aurora high availability](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)
- [Regional NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html)

Reviewed: 2026-09-21. Back to [[architecture_overview|Architecture]].
