---
aliases: [Containers, Amazon ECS, Amazon EKS, AWS Fargate, Amazon ECR]
tags: [aws/saa, compute]
---

# Containers — ECS, EKS, Fargate and ECR

## Mental Model

**ECR stores the image. ECS or EKS orchestrates containers. A compute option supplies the resources on which they run.**

Do not compare an image registry, an orchestrator and a compute engine as if they performed the same job.

## Core

### Service roles in the architecture

| Service | Job | Decision clue |
|---|---|---|
| ECR | Store and distribute container images | Private image registry, scanning and image lifecycle policies |
| ECS | AWS-native container orchestration | Tasks and services without a Kubernetes requirement |
| EKS | Managed Kubernetes | Existing Kubernetes tooling, APIs or ecosystem |
| Fargate | Managed compute for supported ECS/EKS workloads | Avoid provisioning and patching worker servers |
| EC2 capacity | Instances hosting containers | Host control, specialized hardware or requirements incompatible with Fargate |

ECS also offers **Managed Instances** for supported EC2 capabilities with infrastructure operations handled by AWS. EKS **Auto Mode** manages more of the data plane than standard EKS. Therefore, “EC2-based containers always require all host operations to be manual” is too broad. First learn the ECS/EKS/Fargate distinction, then check the compute option named in the question.

### ECS vocabulary

- **Task definition:** versioned blueprint for containers, CPU/memory, image, ports, roles and related settings.
- **Task:** running instantiation of a task definition; useful for a one-off job or as part of a service.
- **Service:** maintains desired task count and replaces stopped/unhealthy tasks; can integrate with a load balancer.
- **Cluster:** logical grouping for workloads and capacity.
- **Capacity provider:** defines how ECS uses/manages a capacity source; strategies can distribute tasks across appropriate providers.

For a conventional ECS-on-EC2 design, distinguish **service scaling** (number of tasks) from **cluster capacity scaling** (available instances). More desired tasks do not help when no suitable host capacity exists. Fargate removes that host-fleet management layer but still has quotas and task sizing constraints.

### Permissions: two roles, two purposes

| ECS role | Who uses it? | Example |
|---|---|---|
| Task role | Application code in the task | Read an S3 object or write DynamoDB records |
| Task execution role | ECS/Fargate agent for task startup and related operations | Pull ECR images, publish logs, retrieve configured startup secrets |
| EC2 container-instance role | ECS agent on a conventional EC2 container host | Register/manage the host's interaction with ECS |

If application code fetches a secret at runtime, it needs suitable application/task permissions. Do not give every task broad permissions through one host role. For EKS, use supported workload IAM mechanisms, such as Pod Identity or IAM roles for service accounts, according to configuration support.

### Networking, storage and availability

- ECS Fargate uses **awsvpc** networking: tasks receive ENIs and security groups. For ALB/NLB integration, use **IP targets** for these tasks.
- In private subnets, pulling images and reaching AWS APIs requires a working NAT or suitable endpoint configuration. Private ECR pulls can require both ECR endpoints and an S3 path for image layers.
- Unlike Lambda, an ECS Fargate task can have a public IP when configured in an appropriate public-subnet design. A subnet label alone still does not establish connectivity.
- Spread service tasks across AZs and retain sufficient healthy capacity. A task's temporary filesystem is not durable application storage.
- Use compatible persistent storage, such as EFS for shared files, when the workload needs it. Check compute/OS support rather than assuming all volume types work everywhere.
- Fargate Spot is suitable for compatible interruptible ECS workloads, not a guarantee for an irreplaceable task.

## Comparisons

| Requirement | Starting point |
|---|---|
| AWS-native container service, minimal worker-server operations | ECS + Fargate |
| Kubernetes compatibility | EKS, with suitable compute |
| Direct host access or specialized unsupported Fargate capability | Appropriate EC2-based container capacity |
| Short event-triggered function | [[aws_lambda|Lambda]], if its execution model fits |
| Queue and schedule finite jobs | [[aws_batch|Batch]] on supported container compute |

## Exam Traps

- **ECS is not EC2; Fargate is not an orchestrator.** ECS can use different capacity options.
- **EKS's managed control plane does not mean every worker and application responsibility disappears.** Check the selected compute mode.
- **Task execution role ≠ task role.** Image pull permission does not grant application access to S3 data.
- **More tasks ≠ more EC2 host capacity.** Both layers may need scaling.
- **A container image does not preserve runtime data.** Store important state externally.
- **ECR stores images; it does not run them.**

## Scenario Check

**A task starts successfully but receives AccessDenied when its application reads S3.** Review the task role and relevant resource policies. Adding S3 permissions only to the task execution role does not grant them to the application.

## 30-Second Review

ECR stores images; ECS orchestrates AWS-native tasks; EKS provides Kubernetes; Fargate supplies managed container compute. ECS services maintain task count, while EC2 capacity may need separate scaling. Task roles authorize application calls; execution roles support startup and logs. Fargate uses task ENIs and IP targets. Keep persistent data external, provide network access, and distribute replicas across AZs.

## Sources

- [ECS overview and compute options](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [EKS overview](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [ECS task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)
- [ECS task role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
- [ECS task execution role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)
- [Fargate networking](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-task-networking.html)
- [ECR VPC endpoints](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
