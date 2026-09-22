---
aliases: [EC2 Auto Scaling, Auto Scaling]
tags: [aws/saa, compute]
---

# EC2 Auto Scaling

## Mental Model

**Maintain the desired healthy capacity, then adjust that desired capacity when demand changes.**

The load balancer routes requests. The Auto Scaling group (ASG) launches and replaces instances.

## Core

### Capacity and launch configuration

- **Minimum:** lower bound; **desired:** target capacity; **maximum:** upper bound for normal scaling policies.
- An ASG uses a **launch template** for instance configuration and selected subnets/AZs for placement.
- Capacity usually means instance count; mixed groups can use weighted capacity units or supported vCPU/memory-based settings. Check the question's units.
- Use multiple AZs and enough running capacity to survive failure. Configuring two AZs with desired capacity of one does not create two healthy replicas.
- Maintain stateless application instances; keep durable state outside the replaceable fleet.

### Select a scaling policy

| Policy | Trigger | Good fit |
|---|---|---|
| Target tracking | Deviation from a target metric | Keep average CPU near 50% or ALB requests per target near a chosen value |
| Step scaling | Size of alarm threshold breach | Add more capacity for a larger overload |
| Scheduled scaling | Known time | Increase capacity before a known 9 AM event |
| Predictive scaling | Forecast from recurring historical load | Prepare capacity ahead of expected demand; combine with dynamic scaling for unexpected changes |
| Simple scaling | Alarm followed by cooldown | Recognize the legacy pattern; do not apply its cooldown behavior to every policy |

For target tracking, use a metric that reflects utilization and changes appropriately with capacity. **Total request count or raw queue length is not the same as load per worker.**

For an SQS worker fleet:

`target backlog per instance ≈ acceptable queue latency / average processing time per message`

Example: 100 seconds acceptable waiting time / 10 seconds per message = about 10 queued messages per worker. This is a planning estimate, not a latency guarantee; processing parallelism and variability matter.

### Health and readiness

EC2 health checks are the default. Enable **ELB health checks on the ASG** when an unhealthy application target should be replaced even though EC2 status checks pass.

| Setting / feature | What it does |
|---|---|
| Health-check grace period | Gives a newly InService instance time before configured health failures trigger replacement; not universal termination protection |
| Instance warmup | Accounts for a new instance's initialization in scaling calculations |
| Cooldown | Delays subsequent simple-scaling actions so the previous change can take effect |
| Warm pool | Keeps pre-initialized instances available to reduce startup work |
| Lifecycle hook | Temporarily pauses launch/termination for custom work, subject to timeouts |
| Deregistration delay | Gives in-flight target requests time to finish after deregistration |

Warmup does not configure the ALB health endpoint. A warm pool is not a fleet of targets already serving traffic.

### Changes and termination

- **Instance Refresh:** roll out a new AMI/template configuration gradually with health/capacity controls. Merely updating the template does not replace existing instances.
- **Standby:** temporarily remove an instance from active service for maintenance; choose how desired capacity should change.
- **Scale-in protection:** protects from ASG scale-in selection. It does not block health-based replacement, manual termination or Spot interruption.
- **Termination policy:** chooses which eligible instance to remove, not when demand requires scaling.

For the default policy, AZ balance matters first. Among eligible candidates, outdated configurations are preferred: legacy launch configurations, a noncurrent template, then an older version of the current template. Billing-hour proximity and random selection can break remaining ties. Mixed instances groups also consider purchase-option ratios and allocation strategy. **Do not reduce this to “terminate the oldest instance.”**

## Comparisons

| Service | Scales / manages |
|---|---|
| EC2 Auto Scaling | EC2 fleet capacity and replacement |
| Application Auto Scaling | Supported resources such as ECS service task count, DynamoDB provisioned capacity and Lambda provisioned concurrency |
| Elastic Load Balancing | Traffic distribution; does not itself launch application instances |
| CloudWatch | Metrics and alarms that support scaling decisions |
| CloudTrail | API activity and audit history |

See [[aws_elb]], [[aws_containers]] and [[high_availability]].

## Exam Traps

- **ELB health does not trigger ASG replacement by default.** Enable its use explicitly.
- **Scale-in protection is not protection from all termination.** Design workers to recover unfinished work.
- **Maximum capacity is not proof capacity can launch.** Quotas, available instances, subnet addresses and startup failures can prevent scaling.
- **A lifecycle hook has a timeout.** It cannot hold a Spot instance indefinitely.
- **Scheduled scaling needs a known schedule; predictive scaling learns recurring demand.**
- **Adding workers can overload a database.** Scale against useful demand and downstream limits.

## Scenario Check

**A queue grows while worker CPU remains low because jobs wait on I/O.** Scale using backlog per worker and acceptable processing latency, not CPU alone. Check the downstream bottleneck before adding more workers.

## 30-Second Review

ASG maintains desired healthy capacity; ALB routes. Target tracking maintains a metric, step scaling reacts by severity, scheduled scaling uses known times, and predictive scaling forecasts demand. Enable ELB health checks for application-based replacement. Distinguish warmup, grace period, cooldown and warm pools. Instance Refresh rolls out configuration changes. Scale-in protection is limited; queues and durable state make replacement safe.

## Sources

- [Scaling policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html)
- [Health checks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-checks-overview.html)
- [SQS-based scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/scale-sqs-queue-cli.html)
- [Default instance warmup](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-default-instance-warmup.html)
- [Termination policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-termination-policies.html)
- [Scale-in protection](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-instance-protection.html)
- [Instance Refresh](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
