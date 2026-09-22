# 🌊 Amazon Kinesis

> [!summary] Mental Model
> **Kinesis = Real-Time Streaming Data**
>
> ```text
> Producers
>    ↓
> Continuous Stream
>    ↓
> Kinesis
>    ↓
> Consumers / Analytics / Storage
> ```

---

# 🎯 Core Purpose

Amazon Kinesis is used to collect and process **streaming data in real time**.

Common examples:

- Application logs
- Clickstreams
- IoT telemetry
- Financial transactions
- Market data
- Real-time metrics
- Social media feeds

> [!tip] Exam Pattern
> **Continuous data**
> +
> **Real-time processing**
>
> → Think **Kinesis**

---

# 🧠 Main Services

For SAA, focus primarily on:

```text
Amazon Kinesis
│
├── Kinesis Data Streams (KDS)
│   └── Collect + process streams
│
├── Amazon Data Firehose
│   └── Deliver streams to destinations
│
└── Kinesis Video Streams
    └── Stream video/audio
```

---

# 🌊 Kinesis Data Streams

Kinesis Data Streams (KDS) collects and stores real-time streaming data for consumers to process.

```text
Producers
   ↓
Kinesis Data Streams
   ↓
Consumers
├── Lambda
├── Applications
├── Firehose
└── Analytics
```

Think:

**"I need to PROCESS the stream."**

Examples:

- Clickstream processing
- Financial market feeds
- Real-time fraud detection
- Logs
- IoT events

---

# 🧩 Producers and Consumers

## Producers

Write records into the stream.

```text
Applications
IoT Devices
Web Servers
     ↓
Kinesis Data Streams
```

## Consumers

Read and process records.

```text
Kinesis Data Streams
       ↓
Consumers
├── Lambda
├── KCL Application
├── Firehose
└── Other processing apps
```

> [!tip]
> Multiple consumers can independently process the same stream.

---

# 🧱 Shards

A Kinesis Data Stream is divided into **shards**.

```text
Kinesis Stream
│
├── Shard 1
├── Shard 2
├── Shard 3
└── Shard 4
```

Shards provide stream capacity and parallelism.

More shards:

```text
More Shards
    ↓
More Capacity
    +
More Parallel Processing
```

> [!tip] Exam Pattern
> **Kinesis Data Streams is throttling because capacity is insufficient**
>
> → Increase stream capacity / shards

---

# 🔑 Partition Key

When a producer sends a record, it includes a **partition key**.

```text
Record
├── Data
└── Partition Key
        ↓
       Hash
        ↓
      Shard
```

Records with the same partition key are routed consistently and preserve ordering within that shard.

> [!danger] Exam Trap
> Poor partition-key distribution can create a **hot shard**.

Think:

```text
Same Partition Key
        ↓
Same Shard
        ↓
Too Much Traffic
        ↓
HOT SHARD 🔥
```

Use high-cardinality / well-distributed partition keys when possible.

---

# 🔢 Ordering

Kinesis provides ordering **within a shard**.

```text
Shard

Record A
   ↓
Record B
   ↓
Record C
```

> [!important]
> Ordering is not a simple global ordering guarantee across all shards.

---

# ⏱️ Real-Time Processing

Kinesis Data Streams is designed for very low-latency streaming.

AWS states that the typical delay between putting a record and being able to retrieve it is **less than one second**. :chatgpt-content-reference{index="2"}

> [!tip] Exam Pattern
> **Process streaming events within seconds / near real time**
>
> → Kinesis Data Streams

---

# 💾 Retention

Unlike a simple message delivery mechanism, Kinesis Data Streams retains records for a configured period.

This allows consumers to:

- Read records
- Reprocess records
- Replay historical stream data

```text
Producer
   ↓
Kinesis Stream
   ↓
Stored Records
   ↓
Consumer can replay
```

> [!tip] Exam Pattern
> **Need to replay/reprocess streaming events**
>
> → Kinesis Data Streams

---

# ⚡ Kinesis + Lambda

Very common serverless architecture:

```text
Producers
    ↓
Kinesis Data Streams
    ↓
Lambda
    ↓
Processing
```

Lambda polls the stream and processes records in batches.

> [!tip] Exam Pattern
> **Real-time stream + serverless processing**
>
> → Kinesis Data Streams + Lambda

---

# 🚒 Amazon Data Firehose

> [!warning] Naming Update
> Previously:
> **Amazon Kinesis Data Firehose**
>
> Current name:
> **Amazon Data Firehose**

Firehose is used primarily to **deliver streaming data to destinations**.

```text
Streaming Data
      ↓
Data Firehose
      ↓
Destination
```

Common destinations include:

- Amazon S3
- Amazon Redshift
- Amazon OpenSearch Service
- Splunk
- HTTP endpoints

AWS can manage the delivery infrastructure automatically. :chatgpt-content-reference{index="3"}

> [!tip] Exam Pattern
> **Continuously deliver streaming data to S3**
>
> → ✅ Amazon Data Firehose

---

# 🔥 KDS vs Firehose

This is the MOST important Kinesis comparison.

## Kinesis Data Streams

```text
Data
 ↓
KDS
 ↓
Custom Consumers
 ↓
PROCESS
```

Think:

- Custom processing
- Multiple consumers
- Replay
- Consumer applications
- Real-time processing

## Data Firehose

```text
Data
 ↓
Firehose
 ↓
S3 / Redshift / OpenSearch
```

Think:

- Managed delivery
- Minimal administration
- Destination-oriented

| Requirement | Think |
|---|---|
| Custom real-time processing | **Kinesis Data Streams** |
| Multiple independent consumers | **Kinesis Data Streams** |
| Replay stream data | **Kinesis Data Streams** |
| Deliver stream to S3 | **Data Firehose** |
| Deliver stream to Redshift | **Data Firehose** |
| Deliver stream to OpenSearch | **Data Firehose** |
| Least operational overhead for delivery | **Data Firehose** |

> [!summary] Memory
> **KDS = PROCESS**
>
> **FIREHOSE = DELIVER**

---

# 🔗 KDS + Firehose

They can also work together.

```text
Producers
    ↓
Kinesis Data Streams
    ├──→ Lambda / Consumer A
    ├──→ Consumer B
    │
    └──→ Data Firehose
             ↓
             S3
```

Firehose can use an existing Kinesis Data Stream as its source. :chatgpt-content-reference{index="4"}

This allows:

- Real-time processing by consumers
- Simultaneous delivery to persistent storage

---

# 📦 Firehose Buffering

Firehose generally **buffers records before delivery**.

```text
Records
 ↓
Firehose
 ↓
Buffer
(size / time)
 ↓
Destination
```

> [!danger]
> Firehose is not necessarily an instant record-by-record delivery mechanism.

AWS uses buffer size and buffer interval when delivering data. :chatgpt-content-reference{index="5"}

---

# 🔄 Firehose Transformations

Firehose can transform records before delivery.

```text
Stream
 ↓
Firehose
 ↓
Transformation
 ↓
Destination
```

It can also perform format conversion for supported configurations.

> [!tip] Exam Pattern
> **Streaming data → transform → automatically deliver to S3**
>
> → Data Firehose

---

# 🟥 Firehose + Redshift

Important architecture detail:

```text
Firehose
   ↓
Amazon S3
   ↓
COPY
   ↓
Amazon Redshift
```

For Redshift destinations, Firehose first delivers the data to S3 and then uses a Redshift `COPY` operation. :chatgpt-content-reference{index="6"}

---

# 📹 Kinesis Video Streams

Used for streaming:

- Video
- Audio
- Time-encoded media

```text
Camera
  ↓
Kinesis Video Streams
  ↓
Processing / Analytics
```

> [!tip] Exam Pattern
> **Thousands of cameras continuously stream video to AWS**
>
> → Kinesis Video Streams

---

# 🆚 Kinesis vs SQS

Another important SAA distinction.

## SQS

```text
Producer
 ↓
Queue
 ↓
Consumer
```

Think:

- Decoupling
- Work queue
- Asynchronous processing

## Kinesis

```text
Producer
 ↓
Stream
 ↓
Consumer A
Consumer B
Consumer C
```

Think:

- Streaming
- Real-time analytics
- Multiple independent consumers
- Replay

| Requirement | Think |
|---|---|
| Decouple application components | **SQS** |
| Background jobs | **SQS** |
| Real-time data stream | **Kinesis** |
| Clickstream analytics | **Kinesis** |
| Market data feed | **Kinesis** |
| Replay streaming data | **Kinesis** |

> [!summary]
> **SQS = MESSAGE QUEUE**
>
> **KINESIS = DATA STREAM**

---

# 🆚 Kinesis vs SNS

```text
SNS
→ Push notifications / fan-out

Kinesis
→ Continuous ordered data stream
→ Processing / analytics
```

Example:

```text
Send notification to many subscribers
→ SNS

Process millions of clickstream events
→ Kinesis
```

---

# 🆚 Kinesis vs EventBridge

```text
EventBridge
→ Application/service EVENTS
→ Routing based on rules

Kinesis
→ High-volume STREAMING DATA
→ Continuous processing
```

Think:

```text
"EC2 instance changed state"
→ EventBridge

"Millions of website clicks"
→ Kinesis
```

---

# 🆚 Kinesis vs MSK

```text
Kinesis
→ AWS-native managed streaming

Amazon MSK
→ Managed Apache Kafka
```

> [!tip]
> **Existing Kafka application**
>
> → Amazon MSK
>
> **AWS-native streaming requirement**
>
> → Kinesis

---

# 🆕 Streaming Analytics

Older study material may mention:

**Kinesis Data Analytics for SQL**

Do NOT prioritize it as a current service.

AWS discontinued Kinesis Data Analytics for SQL applications in 2026.

For managed stream processing, know:

**Amazon Managed Service for Apache Flink**

```text
Kinesis / Kafka
      ↓
Managed Service for Apache Flink
      ↓
Real-Time Stream Processing
```

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> **Real-time clickstream / market feed**
>
> → Kinesis Data Streams

> [!danger] Trap 2
> **Deliver streaming data automatically to S3**
>
> → Data Firehose

> [!danger] Trap 3
> **Need replay/reprocessing**
>
> → Kinesis Data Streams

> [!danger] Trap 4
> **Need custom consumers**
>
> → Kinesis Data Streams

> [!danger] Trap 5
> **Need a work queue / decouple applications**
>
> → SQS

> [!danger] Trap 6
> **Need event routing based on AWS application events**
>
> → EventBridge

> [!danger] Trap 7
> **Video streaming**
>
> → Kinesis Video Streams

> [!danger] Trap 8
> **Hot shard**
>
> → Check partition-key distribution

---

# 🧠 Kinesis in 30 Seconds

```text
REAL-TIME DATA
→ Kinesis

PROCESS / REPLAY
→ Kinesis Data Streams

DELIVER
→ Data Firehose

VIDEO
→ Kinesis Video Streams

STREAM + SERVERLESS PROCESSING
→ KDS + Lambda

STREAM → S3
→ Firehose

QUEUE / DECOUPLE
→ SQS

EVENT ROUTING
→ EventBridge

KAFKA
→ MSK
```

> [!summary] SAA Memory
> **KDS = PROCESS**
>
> **FIREHOSE = DELIVER**
>
> **SQS = QUEUE**
>
> **SNS = NOTIFY**
>
> **EVENTBRIDGE = ROUTE EVENTS**
>
> **MSK = KAFKA**