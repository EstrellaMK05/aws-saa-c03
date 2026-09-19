# Serverless & Integration

## Quick Decision

- Run code without servers → Lambda
- REST API → API Gateway
- Queue / buffer / decouple → SQS
- Pub/Sub / Fanout → SNS
- Event bus / intelligent routing → EventBridge
- Workflow / orchestration → Step Functions
- Real-time streaming + replay → Kinesis Data Streams
- Deliver streaming data to S3/Redshift/etc. → Data Firehose
- Application user authentication → Cognito

---

# Lambda

Serverless compute service.

- Event-driven
- Automatically scales
- Pay for execution
- No servers to manage

💡 Run code in response to events → Lambda


## Invocation Types

### Synchronous

Caller waits for the response.

Example:

`API Gateway → Lambda → Response`

💡 "Do this and tell me the result"


### Asynchronous

Caller sends the event but does NOT wait for the function result.

Example:

`S3 Event → Lambda`

Lambda handles the event and retry behavior.

💡 "Do this, I'll continue"


### Poll-Based

Lambda reads events using an Event Source Mapping.

Example:

`SQS → Lambda`
`Kinesis → Lambda`

---

## Lambda Concurrency

### Reserved Concurrency

Reserves AND limits concurrency for a function.

💡 Limit Lambda / protect capacity for critical function
→ Reserved Concurrency


### Provisioned Concurrency

Keeps execution environments initialized.

Reduces cold starts.

💡 Consistent low startup latency → Provisioned Concurrency

⚠️ Reserved ≠ reduce cold starts
⚠️ Provisioned → cold start optimization


## Lambda + SQS

Lambda can process SQS messages in batches.

If only some messages fail:

→ Partial Batch Response

This prevents successful messages from being unnecessarily processed again.

💡 1 message fails in batch of 10
→ Report partial batch item failures


## Async Failure

For asynchronous Lambda invocations, use:

**Lambda Destinations**

Can route successful or failed invocation results.

💡 Async Lambda fails after retries → On-Failure Destination


## Lambda in VPC

A Lambda inside private subnets does NOT automatically have Internet access.

For outbound IPv4 Internet:

`Lambda → NAT Gateway → Internet Gateway`

💡 Private Lambda needs public Internet → NAT Gateway


---

# SQS

Message queue used to:

- Decouple applications
- Buffer workloads
- Handle traffic spikes
- Process work asynchronously

💡 Queue / buffer / decouple → SQS


## SQS Standard

- Very high throughput
- At-least-once delivery
- Best-effort ordering

⚠️ Duplicate messages are possible.

Consumer should be:

**IDEMPOTENT**

💡 Duplicate must not cause duplicate charge/order
→ Idempotent consumer


## SQS FIFO

FIFO = First-In-First-Out

- Ordering
- Deduplication

💡 Strict ordering + deduplication → FIFO


### Message Group

Messages in the same message group are processed in order.


### Content-Based Deduplication

SQS can generate the deduplication ID from the message body.

💡 Producer may send same message repeatedly and does not provide deduplication ID
→ Content-based deduplication

Deduplication interval:
**5 minutes**


## Visibility Timeout

After a consumer receives a message, SQS temporarily hides it from other consumers.

If processing takes longer than the visibility timeout, the message can become visible again.

💡 Processing takes 5 min but visibility = 30 sec
→ Increase Visibility Timeout

⚠️ Visibility Timeout does NOT guarantee duplicate prevention.


## Delay Queue

New messages remain invisible for a configured period BEFORE they can be consumed.

💡 Don't process new message for 5 minutes
→ Delay Queue

⚠️ Delay = BEFORE processing
⚠️ Visibility Timeout = DURING processing


## Long Polling

Waits for messages instead of constantly returning empty responses.

Reduces:
- Empty receives
- API calls
- Cost

💡 Many empty ReceiveMessage calls → Long Polling


## Dead-Letter Queue — DLQ

Stores messages that repeatedly fail processing.

Use with:

`maxReceiveCount`

💡 Poison message / repeated failures → DLQ


## SQS + Auto Scaling

Workers processing SQS messages should scale based on queue backlog.

Example metric:

`ApproximateNumberOfMessagesVisible`

💡 Thousands of pending jobs but CPU is low
→ Scale based on SQS backlog


---

# SNS

Pub/Sub messaging service.

One message can be delivered to multiple subscribers.

💡 Fanout → SNS


## SNS + SQS Fanout

Common architecture:

              ┌→ SQS → Billing
`Order → SNS ─┼→ SQS → Inventory`
              └→ SQS → Analytics

Each system:
- Gets its own copy
- Processes at its own pace
- Has durable buffering

💡 Every consumer needs EVERY event
→ SNS + separate SQS queues


⚠️ One SQS queue + multiple consumers

Consumers COMPETE for messages.

They do NOT each receive a copy.


## SNS Filter Policy

Allows subscriptions to receive only matching messages.

💡 One SNS topic but consumers need different event types
→ Subscription Filter Policy


---

# EventBridge

Serverless event bus.

Used for:
- Event-driven architectures
- Content-based routing
- AWS service events
- Custom application events
- SaaS integrations

💡 Route based on event JSON/content → EventBridge


## EventBridge Rules

Match events based on properties and route them to targets.

Example:

`source = orders`
`type = OrderCreated`

→ Lambda / SQS / Step Functions / etc.


## EventBridge Archive & Replay

### Archive
Retains matching events.

### Replay
Reprocesses historical archived events.

💡 Consumer had a bug and historical events must be processed again
→ Archive + Replay


## EventBridge Scheduler

Runs actions based on schedules.

Examples:
- Cron
- One-time schedules
- Recurring schedules

💡 Run Lambda every day at 2 AM → EventBridge Scheduler


---

# Step Functions

Serverless workflow orchestration.

Used to coordinate:
- Lambda
- AWS services
- Retries
- Error handling
- Workflow state

💡 Multiple steps + dependencies + retries → Step Functions


## Standard Workflow

- Durable
- Long-running
- Up to 1 year
- Good for auditable/business workflows

💡 Long / durable workflow → Standard


## Express Workflow

- High volume
- Short duration
- Up to 5 minutes
- Good for event processing

💡 Huge volume + short workflow → Express


## Parallel State

Runs multiple branches at the same time.

Example:

        ┌→ Lambda A
`Start ┼→ Lambda B → Continue`
        └→ Lambda C

💡 Independent tasks simultaneously → Parallel


## Map State

Runs the same processing logic for multiple items.

Example:

`[image1, image2, image3...] → Process each image`

💡 Process collection of items → Map


---

# Kinesis Data Streams

Real-time streaming service.

Used for:
- High-volume streams
- Near real-time processing
- Multiple consumers
- Replay

Examples:
- Clickstreams
- IoT telemetry
- Application events

💡 Streaming + multiple consumers + replay → Kinesis Data Streams


## Partition Key

Determines which shard receives a record.

Records with the SAME partition key go to the same shard.

💡 Need ordering for same user/customer/device
→ Same Partition Key

⚠️ Ordering is within a shard, not globally across the entire stream.


## Enhanced Fan-Out

Provides dedicated read throughput to each registered consumer.

💡 Multiple consumers competing for shard read throughput
→ Enhanced Fan-Out


---

# Amazon Data Firehose

Managed service for delivering streaming data to destinations.

Can:
- Buffer
- Batch
- Transform
- Deliver

Common destinations include:
- S3
- Redshift
- OpenSearch

💡 Stream data → automatically buffer/batch → S3
→ Data Firehose

⚠️ Kinesis Data Streams → custom real-time stream processing + replay
⚠️ Data Firehose → managed DELIVERY to destination


---

# API Gateway

Managed API service.

Used with Lambda for serverless APIs.

Provides:
- HTTP/REST APIs
- Authentication integration
- Throttling
- Routing
- Caching

💡 Serverless REST API → API Gateway + Lambda


## Throttling

Limits API request rate.

💡 Protect backend from too many requests → API Gateway Throttling


## API Gateway Cache

Caches responses to avoid invoking the backend repeatedly.

💡 Same API data requested repeatedly
→ API Gateway Cache

Can reduce:
- Lambda invocations
- Database queries
- Latency


---

# Cognito

Authentication and identity service for application users.


## Cognito User Pool

User directory and authentication.

Handles:
- Sign-up
- Sign-in
- Tokens

💡 Web/mobile users need login → User Pool


## Cognito Identity Pool

Provides temporary AWS credentials to users.

💡 Application user needs temporary access to AWS resources
→ Identity Pool


⚠️ User Pool → Authentication / tokens
⚠️ Identity Pool → Temporary AWS credentials


---

# 🔥 Integration Decision Map

Need to BUFFER?
→ SQS

Need FANOUT?
→ SNS

Need FANOUT + durable independent consumers?
→ SNS + SQS

Need complex event ROUTING?
→ EventBridge

Need WORKFLOW?
→ Step Functions

Need real-time STREAM + REPLAY?
→ Kinesis Data Streams

Need managed stream DELIVERY to S3?
→ Data Firehose

Need serverless API?
→ API Gateway + Lambda

Need app user authentication?
→ Cognito


# ⚠️ My Exam Traps

## SQS Duplicates

SQS Standard = at-least-once.

Visibility Timeout does NOT prevent all duplicates.

→ Make consumer IDEMPOTENT.


## Delay vs Visibility

Delay Queue
→ BEFORE message is first available

Visibility Timeout
→ AFTER consumer receives message


## One SQS vs SNS + SQS

One SQS + 3 consumers
→ Consumers compete

SNS → 3 SQS queues
→ Every system gets its own copy


## Kinesis vs SQS

SQS
→ Queue / jobs / decoupling

Kinesis
→ Continuous stream / multiple consumers / replay


## Kinesis vs Firehose

Kinesis Data Streams
→ Process stream

Data Firehose
→ Deliver stream


## Reserved vs Provisioned Concurrency

Reserved
→ Reserve / limit concurrency

Provisioned
→ Reduce cold starts


## Standard vs Express Step Functions

Standard
→ Long + durable

Express
→ Short + massive volume