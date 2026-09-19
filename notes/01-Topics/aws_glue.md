# 🧪 AWS Glue

> [!summary] Mental Model
> **AWS Glue = Serverless Data Integration / ETL**
>
> Discover → Catalog → Transform → Load
>
> ```text
> Data Sources
>     ↓
> Glue Crawler
>     ↓
> Glue Data Catalog
>     ↓
> Glue ETL Job
>     ↓
> Data Target
> ```

---

## 📌 Core

AWS Glue is a **fully managed/serverless data integration service** commonly used for ETL.

**ETL**

- **Extract** → Read data
- **Transform** → Clean / convert / enrich
- **Load** → Write to target

Common data stores:

- Amazon S3
- Amazon RDS
- Amazon Redshift
- Amazon DynamoDB
- JDBC-compatible databases

> [!tip] 🎯 Exam Clue
> **Serverless ETL / transform data / prepare data for analytics**
> → AWS Glue

---

# 📚 Glue Data Catalog

The **Glue Data Catalog** is a centralized metadata repository.

It stores metadata such as:

- Databases
- Tables
- Columns
- Schemas
- Partitions
- Data locations

> [!warning]
> The Data Catalog stores **metadata**, not the actual data.

Example:

```text
S3
└── sales/
    └── orders.parquet
          ↑
          │ describes
          │
Glue Data Catalog
└── Database: sales_db
    └── Table: orders
        ├── order_id
        ├── customer_id
        └── amount
```

---

## 🔎 Glue + Athena

Athena can use the **Glue Data Catalog** as its metadata catalog.

```text
S3
 ↓
Glue Crawler
 ↓
Glue Data Catalog
 ↓
Athena
 ↓
SQL Queries
```

> [!tip] 🎯 Exam Clue
> **Query structured/semi-structured data in S3 using SQL**
> → Athena
>
> **Automatically discover schema / create metadata**
> → Glue Crawler + Data Catalog

---

# 🕷️ Glue Crawler

A **Crawler** scans data sources and automatically discovers:

- Data format
- Schema
- Columns
- Partitions
- Table metadata

Then writes the metadata to the **Glue Data Catalog**.

```text
S3 / JDBC / DynamoDB
        ↓
   Glue Crawler
        ↓
 Schema Discovery
        ↓
 Glue Data Catalog
```

Crawlers can run:

- On demand
- On a schedule

For stable incremental datasets, incremental crawling can process newly added folders rather than crawling everything again.

> [!tip] 🎯 Exam Clue
> **Automatically detect schema of files in S3**
> → Glue Crawler

---

# 🏷️ Classifiers

A **Classifier** helps a Crawler determine the structure/schema of data.

Glue provides built-in classifiers.

Custom classifiers can be created for formats such as:

- CSV
- JSON
- XML
- Grok

Mental model:

```text
Crawler
   ↓
Classifier
   ↓
"What format/schema is this?"
   ↓
Data Catalog
```

---

# ⚙️ Glue ETL Jobs

A Glue Job performs the actual data processing.

```text
Source
  ↓
Glue ETL Job
  ↓
Transform
  ↓
Target
```

Common transformations:

- Clean
- Filter
- Join
- Map
- Convert formats
- Flatten
- Enrich

Common job types include:

- Apache Spark
- Streaming ETL
- Python shell

> [!tip] 🎯 Exam Clue
> **Transform large datasets without managing servers**
> → Glue ETL Job

---

# 🔖 Job Bookmarks

**Job Bookmarks maintain processing state.**

They help Glue remember which data has already been processed.

```text
First Run
Files 1 → 100
       ↓
    Processed ✅

Second Run
Files 1 → 100 → Skip
Files 101 → 120 → Process ✅
```

Useful for:

- Incremental ETL
- Preventing unnecessary reprocessing
- Processing only new data

> [!tip] 🎯 Exam Clue
> **Process only new data**
>
> **Avoid reprocessing old data**
>
> → Glue Job Bookmarks

> [!warning] ⚠️ Exam Trap
> **Crawler**
> → discovers/catalogs data
>
> **Job Bookmark**
> → remembers processing state

---

# 🧩 DynamicFrame

A **DynamicFrame** is an AWS Glue data abstraction designed for ETL.

Useful for:

- Semi-structured data
- Nested data
- Schema inconsistencies
- Data cleaning/transformation

Each record can contain its own schema information.

```text
Messy / Semi-structured Data
            ↓
       DynamicFrame
            ↓
     Glue Transforms
            ↓
        Clean Data
```

> [!tip] 🧠 Mental Model
> **Spark DataFrame**
> → structured Spark processing
>
> **Glue DynamicFrame**
> → ETL with more schema flexibility

---

# 🔌 Glue Connections

Glue Connections store information required to connect Glue to external/data-store resources.

Commonly used for:

- Amazon RDS
- Amazon Redshift
- JDBC databases
- Network connectivity

```text
Glue Job
   ↓
Glue Connection
   ↓
VPC / Database
```

For private resources, Glue may need VPC networking configured to reach the target.

> [!tip] 🎯 Exam Clue
> **Glue needs to connect to private RDS/JDBC database**
> → Glue Connection + VPC connectivity

---

# 📦 Common Architecture

## S3 → Glue → Redshift

```text
Raw Data
   ↓
  S3
   ↓
Crawler
   ↓
Data Catalog
   ↓
Glue ETL Job
   ↓
Amazon Redshift
```

Use when data needs transformation before loading into a data warehouse.

---

## S3 → Glue → S3 → Athena

```text
Raw S3
  ↓
Glue ETL
  ↓
Processed S3
  ↓
Glue Data Catalog
  ↓
Athena
```

Example:

```text
CSV
 ↓
Glue
 ↓
Parquet
 ↓
Athena
```

> [!tip] 🎯 Exam Clue
> Convert large amounts of raw data into an analytics-friendly format
> → Glue

---

# 📡 Event-Driven ETL

Glue jobs can be triggered when new data arrives.

Example architecture:

```text
New Object
    ↓
   S3
    ↓
 Event
    ↓
 Lambda
    ↓
 Glue Job
```

Useful for automated ETL pipelines.

---

# 🖥️ AWS Glue Studio

Glue Studio provides a **visual interface** for building and managing ETL jobs.

Can be used to:

- Visually create ETL pipelines
- Configure sources and targets
- Edit transformations
- Run jobs
- Monitor jobs

> [!tip] 🧠 Mental Model
> **Glue**
> → ETL engine
>
> **Glue Studio**
> → Visual interface for building Glue ETL jobs

---

# 🧹 AWS Glue DataBrew

DataBrew provides **visual data preparation**.

Useful for:

- Cleaning data
- Normalizing data
- Profiling datasets
- Preparing data without writing much code

> [!tip] 🎯 Exam Clue
> **Business analyst needs to clean/prepare data visually without coding**
> → Glue DataBrew

---

# 📐 Glue Schema Registry

Provides centralized schema management for **streaming applications**.

Integrates with services such as:

- Amazon Kinesis Data Streams
- Amazon MSK / Apache Kafka
- AWS Lambda

Helps:

- Discover schemas
- Control schemas
- Evolve schemas
- Enforce schema compatibility

> [!tip] 🎯 Exam Clue
> **Manage/evolve schemas for streaming data**
> → Glue Schema Registry

---

# 🔒 Security

Glue uses IAM roles and policies to access AWS resources.

Example:

```text
Glue Job
   │
   │ IAM Role
   ↓
S3 / Redshift / RDS
```

Glue security configurations can protect:

- S3 data
- CloudWatch Logs
- Job bookmarks

Encryption can use:

- SSE-S3
- SSE-KMS

Data in transit can use SSL.

---

## 🔑 KMS

KMS can protect:

- Job bookmarks
- ETL-related data
- Logs
- Data Catalog metadata / connection information

> [!tip] 🎯 Exam Clue
> **Glue data encrypted with customer-controlled keys**
> → AWS KMS

---

# 📊 Monitoring

## CloudWatch Logs

→ Glue job/crawler logs

## CloudWatch Metrics

→ Job metrics and operational monitoring

## CloudTrail

→ Who performed Glue API actions

> [!tip] 🧠 Remember
> **CloudWatch → What is happening?**
>
> **CloudTrail → Who did what?**

---

# ⚔️ Glue vs Athena

| | AWS Glue | Amazon Athena |
|---|---|---|
| Main purpose | ETL / Data integration | SQL queries |
| Transform data | ✅ | Limited/not primary purpose |
| Query S3 using SQL | ❌ | ✅ |
| Data Catalog | ✅ | Uses it |
| Schema discovery | Crawler | ❌ |
| Serverless | ✅ | ✅ |

Mental model:

```text
Glue
→ PREPARE the data

Athena
→ QUERY the data
```

---

# ⚔️ Glue vs EMR

## AWS Glue

→ Serverless / managed ETL  
→ Minimal infrastructure management  
→ ETL-focused

## Amazon EMR

→ Managed big-data platform  
→ More control over frameworks/configuration  
→ Spark, Hadoop and other big-data workloads

> [!tip] 🎯 Exam Clue
> **Serverless ETL + minimal operations**
> → Glue
>
> **Need greater control over big-data cluster/framework**
> → EMR

---

# ⚔️ Glue vs DMS

This one is important.

## AWS Glue

→ **Transform data**

```text
Extract → Transform → Load
```

## AWS DMS

→ **Migrate/replicate databases**

```text
Database A
    ↓
   DMS
    ↓
Database B
```

> [!tip] 🎯 Exam Clue
> **Migrate Oracle → RDS with minimal downtime**
> → DMS
>
> **Clean/transform data before analytics**
> → Glue

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| Serverless ETL | **Glue** |
| Discover S3 schema | **Glue Crawler** |
| Central metadata repository | **Glue Data Catalog** |
| Query S3 with SQL | **Athena** |
| Process only new data | **Job Bookmark** |
| Semi-structured ETL | **DynamicFrame** |
| Visual ETL development | **Glue Studio** |
| Visual data preparation | **DataBrew** |
| Streaming schema management | **Schema Registry** |
| Private JDBC database | **Glue Connection** |
| Database migration | **DMS** |
| Heterogeneous DB schema conversion | **SCT** |
| Big-data platform / more control | **EMR** |
| ETL logs | **CloudWatch Logs** |
| Who changed Glue resource | **CloudTrail** |

---

> [!abstract] 🧠 AWS Glue in 20 Seconds
> **Purpose:** Serverless data integration / ETL
>
> **Crawler:** Discover schema
>
> **Data Catalog:** Store metadata
>
> **Job:** Transform data
>
> **Job Bookmark:** Avoid reprocessing old data
>
> **DynamicFrame:** Schema-flexible ETL
>
> **Connection:** Connect to data stores/VPC resources
>
> **Studio:** Visual ETL
>
> **DataBrew:** Visual data preparation
>
> **Schema Registry:** Streaming schemas
>
> **Athena:** Query data
>
> **DMS:** Migrate databases
>
> **EMR:** Big-data processing with more control