---
aliases: [Compute, Compute Decision Map]
tags: [aws/saa, compute]
---

# Compute — Choose the Execution Model

## Mental Model

**Start with the workload: what runs, for how long, with how much control, and with what tolerance for interruption?**

Then choose capacity, scaling, availability and cost controls. A service name alone does not solve all five.

## Core

| Requirement | Start with | Confirm |
|---|---|---|
| OS access, custom software or specialized hardware | [[aws_ec2|EC2]] | Instance family, storage, patching and HA |
| Replace unhealthy VMs and adjust fleet size | [[aws_auto_scaling|EC2 Auto Scaling]] | Health checks, startup time and scaling metric |
| Route web requests by host/path | [[aws_elb|ALB]] | Healthy targets, TLS and network access |
| TCP/UDP traffic or fixed per-AZ load-balancer IPs | [[aws_elb|NLB]] | Protocol, target type and deployment across AZs |
| Short event-driven functions without managing servers | [[aws_lambda|Lambda]] | Invocation model, timeout, concurrency and retries |
| Containers without managing worker servers | [[aws_containers|ECS with Fargate]] | Networking, IAM, task size and storage |
| Kubernetes APIs and ecosystem are required | [[aws_containers|EKS]] | Compute option and remaining operational responsibilities |
| Queued jobs with scheduling, dependencies and resource requirements | [[aws_batch|AWS Batch]] | Compute environment, retries and interruption tolerance |
| Deploy a conventional web app with managed environment operations | [[aws_elastic_beanstalk|Elastic Beanstalk]] | Environment type, deployment policy and persistent data |

### Four decisions that appear across services

1. **Execution:** VM, container, function or batch job?
2. **Resilience:** enough healthy capacity across AZs; durable state outside replaceable compute.
3. **Scaling:** reactive demand, known schedule or forecast; protect downstream services.
4. **Cost:** right-size first; commit for predictable usage; use interruptible capacity only when work can recover.

See [[architecture_overview|Architecture]] for failure scopes and [[resilient_design_patterns|Resilient Design Patterns]] for queues and idempotency.

## Comparisons

| Layer | Examples | Responsibility |
|---|---|---|
| Application routing | ALB, NLB | Distribute traffic |
| Container orchestration | ECS, EKS | Schedule and maintain container workloads |
| Compute capacity | EC2, Fargate | Supply resources on which work executes |
| Fleet scaling | EC2 Auto Scaling | Maintain and change instance capacity |
| Job scheduling | Batch | Place queued work on compatible compute |
| Application platform | Beanstalk | Manage deployment and environment resources |

These layers can work together: **ALB → ECS service → Fargate tasks** is one architecture, not three competing answers.

## Exam Traps

- **Serverless does not mean unlimited scale or zero configuration.** IAM, networking, quotas and downstream capacity still matter.
- **A traffic spike does not automatically imply Lambda.** Check runtime compatibility, duration, state and latency first.
- **A container image is packaging.** It can run on different compute services with different lifecycle rules.
- **A price discount does not necessarily reserve capacity.** Separate billing commitments from capacity guarantees.
- **Running a single instance/task is still a single compute replica**, even when a service can scale automatically.

## Scenario Check

**An existing containerized API must run continuously without managing worker servers; Kubernetes is not required.** ECS with Fargate is a strong fit. Lambda would require a compatible function execution model; EKS introduces Kubernetes requirements the question does not ask for.

## 30-Second Review

EC2 gives OS control. ASG maintains the VM fleet; ELB distributes traffic. Lambda runs short event-driven functions. ECS orchestrates containers; EKS provides Kubernetes; Fargate supplies managed container compute. Batch schedules jobs. Beanstalk manages application environments. Match runtime, state, interruption tolerance and operational effort before optimizing price. Managed compute still needs resilient data, suitable permissions and capacity planning.

## Study Order

1. [[aws_ec2]] → [[aws_elb]] → [[aws_auto_scaling]]
2. [[aws_lambda]] → [[aws_containers]]
3. [[aws_batch]] → [[aws_elastic_beanstalk]]

## Sources

- [SAA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html)
- Service-specific documentation is linked in each note.

Reviewed: 2026-09-21. Scope: SAA-C03 decisions, not an exhaustive feature catalog.
