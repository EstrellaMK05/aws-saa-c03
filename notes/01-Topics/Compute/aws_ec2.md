---
aliases: [Amazon EC2, EC2]
tags: [aws/saa, compute]
---

# Amazon EC2

## Mental Model

**Rent a virtual server in one AZ. You control the guest OS, application and configuration; AWS manages the physical infrastructure.**

Choose the right machine, protect its data, and make replacement repeatable.

## Core

### Launch and right-size

- **AMI:** OS/software image and launch information. AMIs are Regional; copy an AMI for use in another Region. Check processor architecture compatibility.
- **Launch template:** versioned launch settings such as AMI, instance type, storage, security groups, instance profile and user data.
- **User data:** bootstrap commands; Linux user-data scripts normally run on first launch, not every reboot by default. Avoid secrets in scripts.
- **Instance family:** match the bottleneck, not just the lowest hourly price.

| Family / capability | Typical fit |
|---|---|
| M — general purpose | Balanced CPU and memory |
| C — compute optimized | CPU-heavy processing |
| R — memory optimized | Large in-memory datasets |
| I — storage optimized | High local storage I/O |
| G / P — accelerated computing | Compatible GPU workloads |
| T — burstable | Low baseline CPU with occasional bursts; understand credits and Unlimited charges |
| Graviton / Arm-based types | Potential price-performance benefits when OS, binaries and dependencies support Arm |

### Purchasing: price and capacity are separate

| Option | What you obtain | Exam distinction |
|---|---|---|
| On-Demand | Usage without a long-term commitment | Good for uncertain duration; no advance capacity reservation |
| Compute Savings Plans | Discount for a 1- or 3-year eligible compute spend commitment, measured in dollars/hour | Flexible across eligible EC2, Fargate and Lambda usage; no capacity reservation |
| EC2 Instance Savings Plans | Discount tied to an instance family in a Region | Less flexible than Compute Savings Plans; no capacity reservation |
| Regional Reserved Instance | Discount for matching EC2 usage | Does not reserve capacity |
| Zonal Reserved Instance | Discount plus capacity reservation for the matching configuration in one AZ | Exception to “RIs are only discounts” |
| On-Demand Capacity Reservation | Matching EC2 capacity in an AZ once active | No intrinsic discount; unused reserved capacity is billable; eligible discounts may apply |
| Spot | Discounted spare capacity that can be interrupted | Use recoverable, flexible workloads; capacity is not guaranteed |

Standard and Convertible RIs have different modification/exchange flexibility. Do not memorize discount percentages as universal guarantees.

### Spot architecture

Checkpoint progress to durable storage, make retries safe, and diversify compatible instance types and AZs. An ASG mixed instances policy can combine an On-Demand baseline with Spot capacity. Price-capacity-optimized allocation considers price and available capacity.

Spot interruption notices are best effort and normally give two minutes for stop/terminate; hibernation starts immediately. **Do not depend on a warning arriving to preserve the only copy of data.** Capacity Rebalancing can proactively replace at-risk instances, but cannot guarantee uninterrupted capacity.

### Storage and lifecycle

| Operation / storage | What to remember |
|---|---|
| Reboot | Normally retains private/public addresses and instance-store data |
| Stop/start an EBS-backed instance | EBS persists; RAM is lost; auto-assigned public IPv4 normally changes |
| Hibernate, when supported and configured | Saves RAM to an encrypted EBS root volume; requires suitable configuration and space |
| Terminate | Instance cannot restart; EBS deletion follows each volume's DeleteOnTermination setting |
| Instance store | Host-local temporary data; lost on stop, hibernate, termination or relevant host failure |
| EBS | Persistent block volume in one AZ; attach to compatible instances in that AZ |
| EFS | Shared network file storage; see [[aws_efs|EFS]] |

Stopped instances can still incur storage and other resource charges. “Persistent EBS” does not mean “survives every termination setting.”

### Security, networking and operations

- Attach an **IAM role through an instance profile** for temporary AWS credentials. Require **IMDSv2** to strengthen metadata access; it does not replace least privilege.
- Security groups are stateful, allow-only controls on ENIs. Network access and IAM permissions answer different questions.
- An ENI belongs to an AZ. A detachable secondary ENI can move between compatible instances in that AZ; the primary ENI cannot simply be detached for failover.
- An Elastic IP is a static public IPv4 address. Use load-balancer/DNS abstractions for fleets; public IPv4 addresses can incur charges.
- Systems Manager Session Manager can provide managed access without opening inbound SSH, given the agent, permissions and connectivity.
- **System status failure:** investigate underlying infrastructure; supported EC2 recovery can help. **Instance status failure:** investigate guest OS/networking. Supported instances also expose attached-EBS health status; optional EC2 application status checks monitor configured HTTP/HTTPS endpoints.
- CloudWatch's standard EC2 metrics do not include guest memory utilization or filesystem free space. Use the CloudWatch agent or custom metrics.

## Comparisons

| Placement / tenancy | Choose when | Trade-off |
|---|---|---|
| Cluster placement group | Tightly coupled, low-latency HPC | One AZ; performance over failure isolation |
| Spread placement group | Small set of critical instances needs separate hardware | Strict placement limits; not a large-fleet strategy |
| Partition placement group | Distributed systems need groups on separate racks | Instances within one partition can share hardware |
| Dedicated Instances | Dedicated single-account hardware tenancy | Less host placement/licensing visibility |
| Dedicated Hosts | Physical host control or eligible socket/core-based BYOL | Manage host capacity and licensing requirements |

## Exam Traps

- **Regional RI ≠ zonal RI.** Only the zonal scope includes a capacity reservation.
- **On-Demand is not fault tolerance.** Avoiding Spot interruption does not prevent host or AZ failure.
- **Cluster placement is not Multi-AZ HA.** Its performance benefit comes with a shared AZ failure scope.
- **A stopped instance is not completely free.** EBS and other allocated resources can remain billable.
- **Changing a launch template does not update existing machines.** Use an appropriate rollout such as [[aws_auto_scaling|Instance Refresh]].
- **An IAM role cannot fix a blocked network path**, and a security group cannot grant S3 API permissions.

## Scenario Check

**A long-running workload needs an EC2 discount and matching capacity in a specific AZ.** A zonal RI can provide both. A Regional RI or Savings Plan alone cannot satisfy the capacity requirement. An active matching Capacity Reservation is another capacity mechanism, with discount eligibility evaluated separately.

## 30-Second Review

EC2 means OS control and responsibility. Match instance resources to the bottleneck. Separate discounts from capacity: Savings Plans and Regional RIs discount usage; zonal RIs also reserve capacity. Spot needs recoverable work. EBS persists according to lifecycle settings; instance store is temporary. Use roles, IMDSv2 and appropriate network controls. Placement groups trade performance against failure isolation; multiple AZs address AZ failures.

## Sources

- [Regional and zonal RIs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/reserved-instances-scope.html)
- [Capacity Reservations](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-capacity-reservations.html)
- [Savings Plans types](https://docs.aws.amazon.com/savingsplans/latest/userguide/plan-types.html)
- [Spot interruption notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-instance-termination-notices.html)
- [Instance lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
- [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
- [EC2 status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html)
- [EC2 user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
