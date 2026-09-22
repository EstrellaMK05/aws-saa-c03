---
aliases:
  - Resilient Design Patterns
  - Loose Coupling
tags:
  - aws/saa
  - architecture
---

# Resilient Design Patterns

## Mental Model

**Assume components will fail or slow down. Prevent one failure from spreading, and make unfinished work recoverable.**

## Core

### 1. Stateless application instances

Keep durable application data and shared session state outside individual application instances. Any healthy instance should be able to handle the next request.

This makes replacement and horizontal scaling easier. The external data store still needs its own availability and durability design. Sticky sessions may help route requests, but do not preserve local state after instance loss.

### 2. Loose coupling and buffering

```text
Producer -> SQS queue -> independently scaled workers -> data store
```

The producer can submit work without waiting for processing to finish. The queue buffers bursts and temporary worker outages. Scale workers using a metric that represents demand, such as backlog per worker, while respecting downstream limits.

- Standard SQS delivery is **at least once**: consumers must tolerate duplicate messages.
- Delete a message after successful processing; a visibility timeout temporarily hides a received message from other consumers.
- A dead-letter queue can isolate repeatedly failing messages when configured. It does not repair those messages automatically.
- Queue retention is finite. A queue does not justify leaving failed consumers unattended indefinitely.

### 3. Safe retries and failure isolation

| Pattern | Purpose | Trap to avoid |
|---|---|---|
| Timeout | Stop waiting indefinitely for a dependency | Treating every timeout as proof the operation did not happen |
| Bounded retries with exponential backoff and jitter | Recover from transient failures without synchronized retry storms | Retrying forever or immediately at every layer |
| Idempotency | Repeating a request has no additional business effect | Charging twice because a response was lost |
| Circuit breaker | Temporarily stop calls to a failing dependency and probe for recovery | Confusing it with a mechanism that fixes the dependency |
| Bulkhead / resource isolation | Limit how much one workload or dependency can consume | Sharing one exhausted worker or connection pool across everything |
| Graceful degradation | Preserve essential functions when optional features fail | Making a nonessential recommendation service block checkout |

Retry transient failures selectively. Invalid requests and authorization errors generally need correction, not repeated identical calls. A timeout after a write is ambiguous: the write may have completed before its response was lost.

### 4. Reproducible infrastructure and safe change

**CloudFormation** describes infrastructure in templates and manages related resources as stacks. Infrastructure as code supports repeatable recovery and reduces manual configuration drift.

- Change sets help review proposed resource changes before execution; some changes can replace resources.
- Drift detection identifies supported differences from declared configuration. Detection itself does not repair the workload.
- Templates recreate infrastructure; backups and replication recover application data.
- Rollback and traffic-shifting strategies reduce deployment risk, but database schema compatibility and data changes still need planning.

## Comparisons

| Communication need | Typical choice | Key distinction |
|---|---|---|
| Immediate request and response | Synchronous API | Caller depends on downstream latency and availability |
| Buffered background jobs | SQS | Workers process queued work independently |
| Deliver a notification to multiple subscribers | SNS | Fan-out; SNS plus separate SQS queues can buffer each subscriber's work |
| Route events by content between producers and consumers | EventBridge | Event routing and integration; a queue can buffer downstream processing |
| Coordinate a multi-step workflow | Step Functions | Explicit workflow state, branching, retries and error handling |

Choose from the required interaction, not the fact that every option can “send messages.” This is the architecture-level comparison; detailed service limits belong in integration notes.

## Exam Traps

- **Asynchronous does not mean instant.** Queuing adds processing delay and requires backlog monitoring.
- **A queue does not remove the database bottleneck.** Unbounded worker scaling can overwhelm a downstream system.
- **FIFO does not make every external side effect exactly once.** Consumer failures and retries still require safe processing and appropriate idempotency.
- **More retries can worsen an outage.** Limit attempts, add backoff and avoid retry multiplication across layers.
- **State outside EC2 still needs protection.** Moving sessions into one fragile dependency only moves the failure point.
- **Multi-AZ alone does not prevent cascading application failures.** Timeouts and resource isolation address different risks.
- **CloudFormation is not a data backup tool.** A stack template cannot restore deleted customer records.

## Scenario Check

**Image uploads arrive in bursts; resizing can finish later, and the database has a fixed throughput limit.** Queue resize jobs in SQS and use independently scaled, idempotent workers. Limit processing to downstream capacity. Increasing synchronous web-server capacity alone leaves requests waiting for image processing and can overload the database.

## 30-Second Review

Stateless instances simplify replacement. Queues decouple producers from workers and absorb bursts, but need retention, retries and backlog management. Timeouts, bounded backoff and jitter limit cascading failures. Idempotency makes retries safe. Circuit breakers and isolated resource pools contain unhealthy dependencies. Infrastructure as code recreates resources, while backups restore data. Scale the system according to its slowest required dependency.

## Sources

- [Prefer stateless services](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_mitigate_interaction_failure_stateless.html)
- [Timeouts, retries and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
- [Scaling based on SQS](https://docs.aws.amazon.com/autoscaling/ec2/userguide/scale-sqs-queue-cli.html)
- [SQS at-least-once delivery](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues-at-least-once-delivery.html)
- [CloudFormation best practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [CloudFormation drift detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)

Reviewed: 2026-09-21. Back to [[architecture_overview|Architecture]].
