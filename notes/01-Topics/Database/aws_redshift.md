---
aliases: [Amazon Redshift, Redshift, Data Warehouse]
tags: [aws/saa, database]
---

# Amazon Redshift

## Mental Model

**Analyze many rows and columns in parallel. Run business analytics without making the transactional application database do every large aggregation.**

Redshift is a warehouse service; the main distinction is analytical access, not whether SQL is used.

## Core

Redshift uses columnar storage and massively parallel processing for analytical queries. Typical workloads include BI dashboards, historical reporting and large aggregations across datasets.

### Compute choices

| Option | Model | Decision |
|---|---|---|
| Provisioned | Choose/manage cluster capacity within supported options | Predictable analytics with explicit capacity planning |
| Serverless | Managed analytics capacity using workgroups/namespaces | Reduce cluster administration for variable or intermittent analytics |

Serverless still requires IAM, network/security configuration and cost controls. It is not a promise of no charges outside query execution: stored data and other resources can remain billable.

### Data movement and query paths

- **COPY:** parallel bulk loading from supported sources such as S3. Appropriate file sizes/formats and parallelism matter.
- **UNLOAD:** write query results to S3 for downstream use.
- **Spectrum / external tables:** query supported S3 data without first loading all of it into warehouse tables; can combine external and internal data.
- **Data sharing:** expose supported live warehouse data to authorized consumers without maintaining a separate copied dataset for each consumer. Consumer compute and permissions still matter.

For S3 queries, use suitable columnar formats, compression, partitions and predicates to reduce scanned data. “No loading required” does not mean no scan cost or equal speed for every format/layout.

### Performance, security and resilience

Distribution and sort choices affect data movement and scan efficiency; automatic optimization can reduce manual work. At SAA depth, identify skew, large scans and concurrency pressure rather than memorizing SQL tuning commands.

Workload management and supported concurrency-scaling features help isolate/prioritize workloads. Additional capacity does not fix incorrect joins or unlimited scans automatically.

Use encryption, appropriate database permissions and IAM roles for S3 access. A network route to S3 does not itself authorize COPY or Spectrum to read data. Choose supported availability/recovery options deliberately, and retain snapshots according to recovery requirements.

## Comparisons

| Requirement | Starting point |
|---|---|
| Transactional orders/payments | [[aws_rds|RDS]] / [[aws_aurora|Aurora]] |
| Large warehouse analytics and BI | Redshift |
| Ad hoc SQL directly on S3 without operating a warehouse | Athena |
| Existing Redshift users join S3 data with warehouse tables | Spectrum / external tables |
| ETL/catalog preparation | Glue; complementary to the query engine |
| Key-based operational records at scale | [[aws_dynamodb|DynamoDB]] |

## Exam Traps

- **SQL does not imply RDS.** Both transactional and analytical services can expose SQL.
- **An RDS read replica is not automatically the best warehouse for years of analytics.**
- **Spectrum avoids loading data, not permissions or scan economics.**
- **Serverless does not remove limits, workload design or cost management.**
- **Backup snapshots are not the same as continuous query availability.**

## Scenario Check

**A BI team already uses Redshift and must join warehouse sales with archived S3 datasets without loading all files.** Use supported external tables/Spectrum with suitable catalog, IAM and data layout. A larger transactional RDS instance does not provide this warehouse integration by itself.

## 30-Second Review

Redshift is a parallel columnar warehouse for analytics. Provisioned controls cluster capacity; Serverless reduces capacity administration. COPY loads, UNLOAD exports, and Spectrum queries S3 without loading all data. Formats, partitions and permissions still matter. Choose RDS/Aurora for operational transactions and Athena for suitable ad hoc S3 SQL. Separate performance scaling, availability and snapshot recovery decisions.

## Sources

- [Redshift architecture](https://docs.aws.amazon.com/redshift/latest/dg/c_redshift_system_overview.html)
- [Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html)
- [Redshift Serverless](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-whatis.html)
- [COPY](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html)
- [Data sharing](https://docs.aws.amazon.com/redshift/latest/dg/datashare-overview.html)

Reviewed: 2026-09-22. Back to [[database_overview|Databases]].
