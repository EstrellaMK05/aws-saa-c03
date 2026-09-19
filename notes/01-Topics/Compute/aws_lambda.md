# ⚡ AWS Lambda

> [!summary] Mental Model
> **AWS Lambda = Run code without managing servers**
>
> Event → Lambda → Action
>
> ```text
> Event
>   ↓
> Lambda
>   ↓
> Execute Code
>   ↓
> AWS Service / Database / API
> ```

---

# 📌 Core

AWS Lambda is a **serverless compute service**.

You provide the code and AWS manages the underlying infrastructure.

Key characteristics:

- No servers to provision/manage
- Event-driven
- Automatically scales
- Pay for requests and execution
- Short-lived/stateless compute
- Maximum execution time: **15 minutes**

> [!tip] 🎯 Exam Clue
> **Run code in response to events without managing servers**
> → AWS Lambda

> [!warning] ⚠️ Exam Trap
> **Execution longer than 15 minutes**
> → Lambda is probably NOT the right choice.

---

# ⚙️ How Lambda Works

```text
Event Source
     ↓
   Lambda
     ↓
Execution Environment
     ↓
Your Code
     ↓
Downstream Service
```

Common event sources:

- API Gateway
- S3
- EventBridge
- SQS
- SNS
- DynamoDB Streams
- Kinesis

Common downstream services:

- S3
- DynamoDB
- RDS
- SQS
- SNS
- Step Functions
- APIs

---

# 🔐 Execution Role

A Lambda function uses an **IAM Execution Role** to access AWS services.

Example:

```text
Lambda
   │
   │ Execution Role
   ↓
S3 GetObject
DynamoDB PutItem
CloudWatch Logs
```

The role defines **what Lambda is allowed to do**.

> [!tip] 🎯 Exam Clue
> **Lambda needs permission to access S3/DynamoDB/etc.**
> → Lambda Execution Role

---

# 🔄 Invocation Types

One of the most important Lambda topics.

## 🔵 Synchronous Invocation

The caller **waits for Lambda to finish** and receives the response.

```text
Client
   ↓
Invoke Lambda
   ↓
Lambda executes
   ↓
Response
   ↓
Client
```

Common example:

```text
Client
   ↓
API Gateway
   ↓
Lambda
   ↓
Response
```

Examples include:

- API Gateway
- Application Load Balancer
- Cognito
- Lambda@Edge

> [!tip] 🎯 Exam Clue
> **Caller needs immediate response**
> → Synchronous Invocation

---

## 🟣 Asynchronous Invocation

The caller sends the event and **doesn't wait for the function to finish**.

Lambda queues the event internally.

```text
Event
   ↓
Lambda Internal Queue
   ↓
Lambda
   ↓
Process
```

The caller receives:

```text
202 Accepted
```

This means:

> Event accepted/queued

NOT:

> Function completed successfully

Common examples:

- S3 events
- EventBridge
- CloudWatch Logs

> [!tip] 🎯 Exam Clue
> **Background event processing**
> → Asynchronous Invocation

---

# 🧠 Sync vs Async

| | Synchronous | Asynchronous |
|---|---|---|
| Caller waits | ✅ | ❌ |
| Immediate result | ✅ | ❌ |
| Internal Lambda queue | ❌ | ✅ |
| Example | API Gateway | S3 / EventBridge |
| Typical use | Request/response | Background events |

Mental model:

```text
SYNC
→ "Give me the result"

ASYNC
→ "Do this when you can"
```

---

# 📥 Event Source Mapping

For queues and streams, Lambda can use an **Event Source Mapping**.

Lambda polls the source and invokes your function.

Common sources:

- Amazon SQS
- DynamoDB Streams
- Kinesis Data Streams
- Amazon MSK
- Amazon MQ
- Apache Kafka

```text
SQS / Stream
     ↓
Event Source Mapping
     ↓
   Lambda
```

It can collect records into **batches** before invoking the function.

> [!tip] 🎯 Exam Clue
> **Lambda processing SQS / Kinesis / DynamoDB Streams**
> → Event Source Mapping

---

# 🚨 Important: Push vs Poll

Don't assume every service invokes Lambda the same way.

### Push-style event

```text
S3
 ↓
Lambda
```

### Poll-based

```text
SQS
 ↓
Lambda Event Source Mapping
 ↓
Lambda
```

> [!warning] ⚠️ Exam Trap
> With services such as **SQS, Kinesis and DynamoDB Streams**, Lambda uses an Event Source Mapping to read/poll records.

---

# 📈 Concurrency

**Concurrency = number of Lambda executions running at the same time.**

Example:

```text
1 request
→ 1 concurrent execution

100 simultaneous requests
→ potentially many concurrent executions
```

Lambda automatically scales execution environments as demand increases.

---

# 🔒 Reserved Concurrency

Reserved Concurrency reserves part of the concurrency capacity for a specific function.

It also acts as the function's **maximum concurrency**.

```text
Account Concurrency
        │
        ├── Lambda A
        │      ↓
        │ Reserved: 100
        │
        └── Other Lambdas
```

Useful for:

- Guaranteeing concurrency for an important function
- Preventing one function from consuming excessive concurrency
- Limiting downstream load

> [!tip] 🎯 Exam Clue
> **Guarantee capacity AND limit maximum concurrency**
> → Reserved Concurrency

---

# 🚀 Provisioned Concurrency

Provisioned Concurrency keeps execution environments **pre-initialized**.

```text
Request
   ↓
Already initialized Lambda
   ↓
Execute immediately
```

Used to reduce:

# **Cold Start latency**

Useful for:

- Latency-sensitive applications
- APIs requiring predictable response times

> [!tip] 🎯 Exam Clue
> **Reduce cold starts**
>
> **Predictable low latency**
>
> → Provisioned Concurrency

---

# ⚔️ Reserved vs Provisioned Concurrency

| | Reserved | Provisioned |
|---|---|---|
| Reserve concurrency capacity | ✅ | — |
| Maximum concurrency limit | ✅ | ❌ |
| Pre-initialize environments | ❌ | ✅ |
| Reduce cold starts | ❌ | ✅ |
| Main purpose | Capacity/control | Low latency |

> [!warning] ⚠️ Exam Trap
> **Reserved → Capacity**
>
> **Provisioned → Performance / Cold Starts**

---

# 🥶 Cold Starts

When Lambda needs a new execution environment, initialization can add latency.

```text
Request
   ↓
Create Environment
   ↓
Initialize Runtime
   ↓
Load Code
   ↓
Execute
```

This is a **cold start**.

Subsequent invocations may reuse an existing environment:

```text
Request
   ↓
Existing Environment
   ↓
Execute
```

→ **Warm invocation**

> [!tip] 🎯 Exam Clue
> **Lambda latency caused by initialization**
> → Cold Start
>
> Need predictable low latency?
> → Provisioned Concurrency

---

# 📦 Lambda Layers

A **Layer** contains reusable code or dependencies.

Examples:

- Libraries
- SDKs
- Shared code
- Custom runtimes

```text
Lambda Function
      +
Lambda Layer
      ↓
Execution
```

Benefits:

- Smaller deployment package
- Reuse dependencies
- Separate application code from libraries

> [!tip] 🎯 Exam Clue
> **Share libraries between Lambda functions**
>
> **Keep deployment package modular/smaller**
>
> → Lambda Layer

---

# 📦 Deployment Packages

Lambda code can be deployed using:

### ZIP

```text
Code + Dependencies
       ↓
      ZIP
       ↓
     Lambda
```

### Container Image

```text
Container Image
      ↓
Amazon ECR
      ↓
Lambda
```

> [!tip] 🎯 Exam Clue
> Lambda can run code packaged as:
>
> **ZIP or Container Image**

---

# 🏷️ Versions

A Lambda **Version** is an immutable snapshot of the function.

Example:

```text
$LATEST

Publish
   ↓
Version 1

Change code

Publish
   ↓
Version 2
```

Published versions cannot be modified.

---

# 🔖 Aliases

An **Alias** is a pointer to a Lambda version.

```text
PROD
 ↓
Version 5
```

Example:

```text
DEV  → Version 8
TEST → Version 7
PROD → Version 5
```

This lets applications reference:

```text
PROD
```

instead of a specific version number.

---

## 🚦 Traffic Shifting

Aliases can help route traffic between Lambda versions.

Example:

```text
PROD
 │
 ├── 90% → Version 5
 │
 └── 10% → Version 6
```

Useful for gradual deployments.

> [!tip] 🎯 Exam Clue
> **Stable name pointing to Lambda version**
> → Alias
>
> **Immutable snapshot**
> → Version

---

# 🌐 Lambda + VPC

By default, a Lambda function is **not connected to your private VPC resources**.

To access resources such as:

- Private RDS
- Private EC2
- Internal services

configure:

- VPC
- Subnets
- Security Groups

```text
Lambda
   ↓
VPC Configuration
   ↓
Private Subnet
   ↓
RDS
```

Lambda creates/manages network interfaces to access resources in your VPC.

> [!tip] 🎯 Exam Clue
> **Lambda needs access to private RDS**
> → Configure Lambda for VPC access

---

# ⚠️ Lambda VPC + Internet Access

A very important networking trap:

Putting Lambda in a **public subnet does NOT automatically give it Internet access**.

For outbound IPv4 Internet access from a VPC-connected Lambda, a common architecture is:

```text
Lambda
   ↓
Private Subnet
   ↓
NAT Gateway
   ↓
Internet Gateway
   ↓
Internet
```

For supported AWS services, consider:

```text
Lambda
   ↓
VPC Endpoint
   ↓
AWS Service
```

> [!warning] ⚠️ Exam Trap
> **Lambda attached to VPC ≠ automatically has Internet access**

---

# 🔗 Lambda Function URL

Lambda can expose a dedicated **HTTPS endpoint** without API Gateway.

```text
Internet
   ↓
Lambda Function URL
   ↓
Lambda
```

Authentication options include:

- `AWS_IAM`
- `NONE`

Useful for simple HTTP endpoints.

> [!tip] 🎯 Exam Clue
> **Simple HTTPS endpoint directly for one Lambda**
> → Lambda Function URL

---

# ⚔️ Function URL vs API Gateway

## Lambda Function URL

→ Simple direct HTTPS endpoint  
→ Less infrastructure  
→ Basic HTTP use cases

## API Gateway

→ Full API management

Features can include:

- Routing
- Authorization
- Throttling
- API stages
- Request/response transformations
- API management features

> [!tip] 🧠 Mental Model
> **Simple Lambda HTTP endpoint**
> → Function URL
>
> **Full-featured API**
> → API Gateway

---

# 🌍 Lambda@Edge

Lambda@Edge runs Lambda functions in association with **CloudFront** events.

```text
User
  ↓
CloudFront Edge
  ↓
Lambda@Edge
  ↓
Origin
```

Can modify requests/responses at points such as:

- Viewer Request
- Origin Request
- Origin Response
- Viewer Response

Useful for:

- Request/response manipulation
- Authentication logic
- URL rewrites
- Header manipulation

> [!tip] 🎯 Exam Clue
> **Execute code close to CloudFront users**
> → Lambda@Edge

---

# 💾 Lambda + EFS

Lambda can mount **Amazon EFS**.

```text
Lambda
   ↓
EFS
   ↓
Shared Files
```

Useful when Lambda functions need:

- Shared persistent files
- Large shared dependencies/data
- File-system access

> [!warning]
> Lambda local execution storage is not the same as durable shared storage.
>
> For shared persistent file storage:
> → EFS

---

# 🔐 Environment Variables

Environment variables store configuration as key-value pairs.

Examples:

```text
DB_HOST
ENVIRONMENT
API_ENDPOINT
```

Environment variables are encrypted at rest.

For sensitive secrets such as database passwords, prefer a dedicated secret-management service when the scenario requires secure secret storage/rotation.

```text
Lambda
   ↓
Secrets Manager
   ↓
Secret
```

> [!tip] 🎯 Exam Clue
> **Configuration**
> → Environment Variables
>
> **Password/API secret + rotation**
> → Secrets Manager

---

# 📊 Monitoring

Lambda integrates with **Amazon CloudWatch**.

## CloudWatch Metrics

Examples:

- Invocations
- Errors
- Duration
- Throttles
- Concurrent executions

## CloudWatch Logs

```text
Lambda
   ↓
CloudWatch Logs
```

Used for function logs and troubleshooting.

---

# 🔎 AWS X-Ray

AWS X-Ray provides **distributed tracing**.

Useful for tracing a request across:

```text
API Gateway
     ↓
Lambda
     ↓
DynamoDB
```

Helps identify:

- Latency
- Bottlenecks
- Service dependencies
- Errors across distributed applications

> [!tip] 🎯 Exam Clue
> **Trace requests across distributed/serverless application**
> → AWS X-Ray

---

# ⚔️ Lambda vs EC2

## Lambda

→ Serverless  
→ Event-driven  
→ Automatic scaling  
→ Short-lived execution  
→ Maximum 15 minutes  
→ Pay for usage

## EC2

→ Full server control  
→ Long-running workloads  
→ OS access  
→ Persistent compute

> [!tip] 🎯 Exam Clue
> **Short event-driven task**
> → Lambda
>
> **Long-running process / OS control**
> → EC2

---

# ⚔️ Lambda vs ECS/Fargate

## Lambda

Best for:

- Event-driven functions
- Short executions
- Rapid automatic scaling

## Fargate

Best for:

- Containers
- Longer-running workloads
- More runtime/container control

```text
Short Event-Driven Code
→ Lambda

Longer Container Workload
→ Fargate
```

---

# 🔄 Lambda + SQS

Very common architecture:

```text
Producer
   ↓
  SQS
   ↓
Lambda Event Source Mapping
   ↓
Lambda
```

Benefits:

- Decoupling
- Buffering
- Handling traffic spikes
- Asynchronous processing

> [!tip] 🎯 Exam Clue
> **Traffic spikes overwhelm Lambda/downstream system**
> → Put SQS between producer and consumer

---

# 🔄 Lambda + API Gateway

Classic serverless API:

```text
Client
   ↓
API Gateway
   ↓
Lambda
   ↓
DynamoDB
```

Useful for:

- REST APIs
- HTTP APIs
- Serverless backends

---

# 🔄 Lambda + EventBridge

Event-driven architecture:

```text
Application
    ↓
EventBridge
    ↓
Lambda
```

Useful for reacting to events without tightly coupling services.

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| Serverless event-driven compute | **Lambda** |
| Execution > 15 minutes | **Not Lambda** |
| Caller waits for result | **Synchronous** |
| Background event | **Asynchronous** |
| SQS → Lambda | **Event Source Mapping** |
| DynamoDB Streams → Lambda | **Event Source Mapping** |
| Kinesis → Lambda | **Event Source Mapping** |
| Limit/guarantee concurrency | **Reserved Concurrency** |
| Reduce cold starts | **Provisioned Concurrency** |
| Shared dependencies | **Lambda Layers** |
| Immutable function snapshot | **Version** |
| Friendly pointer to version | **Alias** |
| Private RDS access | **Lambda + VPC** |
| VPC Lambda needs Internet | **NAT Gateway** |
| Private AWS service access | **VPC Endpoint** |
| Simple direct HTTPS endpoint | **Function URL** |
| Full API management | **API Gateway** |
| Code at CloudFront edge | **Lambda@Edge** |
| Shared persistent filesystem | **EFS** |
| Distributed tracing | **X-Ray** |
| Buffer traffic spikes | **SQS** |

---

# 🚨 Most Important Exam Distinctions

```text
Synchronous
→ Caller WAITS

Asynchronous
→ Event QUEUED
```

```text
Reserved Concurrency
→ CAPACITY / LIMIT

Provisioned Concurrency
→ COLD START / LATENCY
```

```text
Version
→ Immutable SNAPSHOT

Alias
→ POINTER to version
```

```text
Lambda
→ Short event-driven code

Fargate / EC2
→ Longer-running workloads
```

---

> [!abstract] 🧠 AWS Lambda in 30 Seconds
> **Purpose:** Serverless event-driven compute
>
> **Maximum runtime:** 15 minutes
>
> **Sync:** Caller waits
>
> **Async:** Event queued
>
> **Queues/Streams:** Event Source Mapping
>
> **Reserved Concurrency:** Reserve + limit capacity
>
> **Provisioned Concurrency:** Reduce cold starts
>
> **Layer:** Shared dependencies
>
> **Version:** Immutable snapshot
>
> **Alias:** Pointer to version
>
> **Private resources:** VPC configuration
>
> **Simple HTTPS:** Function URL
>
> **Full API:** API Gateway
>
> **Edge execution:** Lambda@Edge
>
> **Shared files:** EFS
>
> **Monitoring:** CloudWatch
>
> **Tracing:** X-Ray