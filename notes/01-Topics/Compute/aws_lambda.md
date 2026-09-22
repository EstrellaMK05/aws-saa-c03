---
aliases: [AWS Lambda, Lambda]
tags: [aws/saa, compute]
---

# AWS Lambda

## Mental Model

**An event triggers a function. AWS supplies execution environments; you design permissions, state, retries and downstream access.**

An accepted event is not proof of successful processing. Automatic scaling is not unlimited concurrency.

## Core

### Execution, resources and state

- A standard function invocation can run for **up to 900 seconds (15 minutes)**. A single uninterrupted two-hour process belongs on suitable container/VM/batch compute.
- Increasing configured memory also increases allocated CPU. More memory can lower duration enough to improve total cost; measure instead of assuming the smallest setting is cheapest.
- Keep authoritative state in services such as S3, DynamoDB or a database. Execution environments may be reused but reuse is not guaranteed.
- `/tmp` is temporary local storage: useful for scratch data or a reusable cache, not the only durable copy. EFS provides shared persistent files with appropriate VPC and permission configuration.
- Package code as a ZIP or compatible container image. Lambda layers share dependencies for ZIP functions; image-based functions include dependencies in the image. Packaging as a container does not remove invocation limits.

This note focuses on standard Lambda function scenarios. Durable workflows can coordinate multiple steps/invocations over a longer period; do not confuse total workflow duration with one continuously running invocation. Newer Lambda execution products have their own constraints.

### Invocation and failure handling

| Model | Examples | What happens on failure? |
|---|---|---|
| Synchronous | API Gateway, ALB, direct request/response | Caller receives the error; caller/integrating service decides whether to retry |
| Asynchronous invocation | S3 notifications, SNS, EventBridge targets | Lambda queues the event and manages asynchronous processing/retries |
| Event source mapping | SQS, Kinesis, DynamoDB Streams | Lambda reads/polls records; retry, batching and checkpoint behavior depend on the source |

For asynchronous **function errors**, Lambda retries twice by default. Throttling/system errors have different retry behavior, governed by the event's age and configuration. Configure an on-failure destination or appropriate DLQ rather than silently losing exhausted/expired events. Destinations provide invocation records; an async DLQ captures failed events. This mechanism is not the SQS redrive policy.

Duplicates are possible. Use idempotent processing, especially for payments, writes and notifications. A timeout does not prove a downstream write failed.

### SQS and streams

- For SQS, successfully processed messages are deleted; failed messages become available again after the visibility timeout.
- Configure **partial batch responses** and implement the response correctly to retry failed SQS items without unnecessarily retrying successful items.
- Configure the **source queue's DLQ/redrive policy** for repeatedly failing SQS messages. Do not substitute the Lambda asynchronous-invocation DLQ.
- Set visibility timeout appropriately; AWS recommends at least six times the function timeout, plus any batching window.
- Limit event-source concurrency where supported and coordinate it with function concurrency to protect downstream systems.
- For Kinesis/DynamoDB Streams, processing follows shard/checkpoint semantics. A bad record can delay progress; use supported retry limits, partial failure handling or batch splitting as appropriate. Ordering is per shard, not global across all shards.

### Concurrency and startup

For ordinary single-request execution environments:

`approximate concurrency = requests per second × average duration in seconds`

Example: 200 requests/second × 0.5 seconds ≈ 100 concurrent executions. This is a sizing estimate; service quotas, scaling rates and burst behavior also apply.

| Control | Main purpose | Key limitation |
|---|---|---|
| Reserved concurrency | Allocate concurrency to a function and cap its maximum | Does not pre-initialize environments or remove cold starts; zero throttles the function |
| Provisioned concurrency | Prepare initialized environments for a version/alias | Additional cost; excess traffic can spill over to on-demand execution if capacity permits |
| SnapStart | Reduce initialization using snapshots for supported runtimes/configurations | Compatibility restrictions; cannot combine with provisioned concurrency on the same function version |

Provisioned concurrency consumes account concurrency capacity. It is not itself a maximum concurrency setting, and callers must invoke the configured version/alias to use it.

### Permissions and networking

- **Execution role:** what function code may do, such as reading S3 or writing logs.
- **Invocation permissions / resource policy:** who or what may invoke the function. Granting the execution role S3 access does not authorize S3 to invoke Lambda.
- To reach private VPC resources, configure suitable subnets and security groups.
- A VPC-connected function does **not** gain public IPv4 internet access merely by selecting a public subnet. A common IPv4 design uses private subnets and a public NAT gateway; supported AWS services can use VPC endpoints.
- For connection-heavy RDS workloads, consider **RDS Proxy** to pool and manage connections; it does not make the database infinitely scalable.
- Store rotating secrets in Secrets Manager; encrypted environment variables alone do not provide rotation.

### Deployment and observability

Published **versions** are immutable code/configuration snapshots. An **alias** provides a stable name and can shift a proportion of traffic between two versions for gradual rollout.

CloudWatch tracks invocations, errors, duration, throttles and concurrency; logs need appropriate permissions. Distributed tracing helps identify latency across dependencies. Monitor event age or queue backlog as well as function errors: a quiet function can have work waiting upstream.

## Comparisons

| Requirement | Choice |
|---|---|
| Simple direct HTTP endpoint for a function | Function URL, with appropriate authentication/permissions |
| API routing, authorization and management features | [[aws_apigateway|API Gateway]]; features differ by API type |
| CloudFront request/response customization | Lambda@Edge or CloudFront Functions, according to capabilities; see [[aws_cloudfront|CloudFront]] |
| Long-running container process | [[aws_containers|ECS/Fargate]] or compatible EC2 compute |
| Scheduled resource-intensive jobs with dependencies | [[aws_batch|Batch]] |
| Buffer asynchronous bursts | SQS plus controlled Lambda consumption |

## Exam Traps

- **SQS processing is asynchronous at the application level, but uses an event source mapping**, not Lambda's asynchronous invocation queue.
- **Reserved concurrency does not solve cold starts.** Provisioned concurrency does not automatically cap the function.
- **A successful HTTP acceptance response is not a successful business operation.** Track processing completion.
- **SQS FIFO does not remove the need for idempotent side effects.** Failures and retries still happen.
- **Public subnet ≠ public IP for Lambda.** Check actual egress and private-service paths.
- **Automatic scaling can exhaust database connections.** Control concurrency and use appropriate connection management.
- **A large burst alone does not make Lambda the right runtime.** Check compatibility, latency, duration and quotas.

## Scenario Check

**One message in an SQS batch fails; the others already updated records successfully.** Use partial batch responses plus idempotent processing. Otherwise, retrying the batch can repeat successful work. Send persistently failing messages through the queue's redrive policy for investigation.

## 30-Second Review

Lambda executes event-driven functions; standard invocations last at most 15 minutes. Sync, async and polling integrations retry differently. Make side effects idempotent and handle partial SQS failures. Reserved concurrency allocates and caps capacity; provisioned concurrency prepares environments. VPC attachment needs explicit networking. Roles govern function actions; invocation policies govern callers. Keep durable state externally and protect downstream capacity.

## Sources

- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [Retry behavior](https://docs.aws.amazon.com/lambda/latest/dg/invocation-retries.html)
- [Asynchronous errors and retries](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-error-handling.html)
- [SQS integration](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
- [SQS mapping configuration](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html)
- [Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
- [SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html)
- [VPC internet access](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc-internet.html)
- [Lambda permissions](https://docs.aws.amazon.com/lambda/latest/dg/lambda-permissions.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
