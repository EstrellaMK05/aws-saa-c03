---
aliases: [AWS Batch, Batch Computing]
tags: [aws/saa, compute]
---

# AWS Batch

## Mental Model

**Submit finite jobs; Batch queues, schedules and places them on suitable compute.**

Think rendering, simulations, scientific processing and large collections of independent data-processing jobs. Interactive request/response serving is a different problem.

## Core

| Component | Purpose |
|---|---|
| Job definition | Describes how to run: image, command, resource requirements, roles and supported retry/timeout settings |
| Job | A submitted unit of work |
| Job queue | Holds work awaiting placement; priorities influence scheduling |
| Compute environment | Compatible compute resources on which jobs execute |

Batch supports different compute integrations, including ECS-based EC2/Fargate and EKS-based environments. Features vary by compute type: do not assume a Fargate environment supports every EC2 job capability.

### Design decisions

- Choose managed compute environments when you want Batch to manage the relevant provisioning and scaling.
- Use suitable EC2 instances for specialized requirements, such as GPUs that are not supported by Fargate.
- Use Spot only when jobs tolerate interruption. Checkpoint to durable storage and make retries safe.
- Use job dependencies when one job must finish before another starts. Array jobs support many related executions with different indexes/inputs.
- Configure retry behavior for recoverable failures. A retry can repeat already-completed external side effects.
- Store inputs and results durably, commonly in S3; temporary container/host files are not the final system of record.

### A typical pattern

`Input in S3 → job submission → Batch queue → compatible compute → output in S3`

EventBridge or another service can trigger submission. Step Functions can coordinate a broader workflow that includes Batch jobs and other services. Batch itself handles job scheduling and compute placement, not every business-process step.

## Comparisons

| Option | Prefer when |
|---|---|
| Lambda | Short event-driven functions with suitable limits and minimal runtime management |
| ECS service | Continuously running container application with a desired replica count |
| ECS standalone task | Run a container task without needing the full Batch scheduling model |
| AWS Batch | Queued finite jobs, resource-aware scheduling, dependencies and retries |
| Step Functions | Coordinate steps across functions, jobs and other services |

Batch jobs are not subject to Lambda's 15-minute invocation limit. Actual job/compute limits and interruption behavior still depend on configuration; “Batch runs forever” is not a useful rule.

## Exam Traps

- **Batch is not an instant-start interactive API backend.** Jobs can wait for scheduling, images and capacity.
- **A queue does not guarantee compute is available.** Check resource requirements, quotas, permissions and compute-environment health.
- **Batch does not make Spot noninterruptible.** Preserve progress when repeating the full job would be unacceptable.
- **Fargate does not support every EC2 capability.** Match specialized hardware and job features to supported compute.
- **Retrying a job is not exactly-once processing.** Prevent duplicate output or charges.

## Scenario Check

**Thousands of independent containerized simulations each take 45 minutes. The company wants managed scheduling and can restart failed jobs.** Consider AWS Batch with appropriate compute, potentially Spot to reduce cost. A single standard Lambda invocation cannot run a 45-minute simulation; an ECS service is aimed at maintaining running tasks rather than scheduling this collection of finite jobs.

## 30-Second Review

Batch runs finite queued jobs on compatible compute. Definitions describe execution; jobs are submissions; queues hold work; environments supply resources. Choose compute for hardware and feature needs. Use dependencies, retries and durable outputs. Spot lowers cost only when interruption is acceptable. Lambda suits shorter functions, ECS services suit continuous applications, and Step Functions coordinates broader workflows.

## Sources

- [What is AWS Batch?](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html)
- [Job definition parameters](https://docs.aws.amazon.com/batch/latest/userguide/job_definition_parameters.html)
- [Compute environments](https://docs.aws.amazon.com/batch/latest/userguide/compute_environments.html)
- [Array jobs](https://docs.aws.amazon.com/batch/latest/userguide/array_jobs.html)
- [Automated job retries](https://docs.aws.amazon.com/batch/latest/userguide/job_retries.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
