# 🌊 AWS Lake Formation

> [!summary] Mental Model
> **Lake Formation = Build + Secure + Govern a Data Lake**
>
> ```text
>             AWS Lake Formation
>                    ↓
>          Central Data Governance
>                    ↓
>         AWS Glue Data Catalog
>                    ↓
>                 Amazon S3
>                    ↓
>        Athena / Redshift / Glue
> ```

---

# 🎯 Core Purpose

AWS Lake Formation helps centrally manage and secure a data lake.

Think:

- Centralized data permissions
- Fine-grained access control
- AWS Glue Data Catalog integration
- S3 data lake governance
- Cross-account data sharing

> [!tip] Exam Pattern
> **S3 Data Lake**
> +
> **Centralized permissions**
> +
> **Fine-grained access**
>
> → ✅ AWS Lake Formation

---

# 🏗️ Core Architecture

```text
                 S3
             Data Lake
                 ↓
        AWS Glue Data Catalog
                 ↓
          Lake Formation
          Permissions
          /      |      \
         ↓       ↓       ↓
      Athena   Glue   Redshift
```

### Amazon S3

Stores the actual data.

### AWS Glue Data Catalog

Stores metadata:

- Databases
- Tables
- Columns
- S3 locations
- Schemas

### Lake Formation

Controls who can access the catalog resources and underlying data.

> [!summary]
> **S3 = DATA**
>
> **Glue Data Catalog = METADATA**
>
> **Lake Formation = GOVERNANCE / PERMISSIONS**

---

# 🔐 Fine-Grained Access Control

Lake Formation can grant permissions at granular levels.

```text
Database
   ↓
Table
   ↓
Columns
   ↓
Rows / Cells
```

Examples:

```text
Data Engineer
→ Full table

Analyst
→ SELECT table

Finance Analyst
→ Only finance-related columns/rows
```

Common Lake Formation permissions include:

- SELECT
- INSERT
- DELETE
- ALTER
- DROP
- DESCRIBE
- CREATE_TABLE

> [!tip] Exam Pattern
> **Athena users should access only specific tables / columns / rows**
>
> → Lake Formation Fine-Grained Access Control

---

# 🔑 IAM + Lake Formation

Lake Formation does NOT simply replace IAM.

Think of two permission layers:

```text
Principal
   ↓
IAM
"Can you call the AWS API?"
   ↓
Lake Formation
"Can you access this data?"
   ↓
Data
```

A request may require both:

```text
IAM Permission
      +
Lake Formation Permission
      ↓
    Access
```

> [!danger] Exam Trap
> Having IAM permissions does not necessarily mean the principal can
> access Lake Formation-managed data.

### Mental Model

```text
IAM
→ Service/API permissions

Lake Formation
→ Data permissions
```

---

# 📍 Registered S3 Locations

S3 locations can be registered with Lake Formation.

```text
S3 Bucket / Prefix
        ↓
Register Location
        ↓
Lake Formation
        ↓
Governed Data Lake
```

Lake Formation can then manage access to the data stored there.

---

# 📍 DATA_LOCATION_ACCESS

Important distinction:

`DATA_LOCATION_ACCESS` does NOT mean:

> "This user can SELECT/read all the data."

It allows a principal to create or alter Data Catalog resources that point to a registered S3 location.

```text
DATA_LOCATION_ACCESS
        ↓
Can reference this S3 location
from Data Catalog resources
```

> [!danger] Exam Trap
> `DATA_LOCATION_ACCESS`
> ≠
> `SELECT`

---

# 🏷️ LF-Tags

Lake Formation supports tag-based access control:

**LF-TBAC**

Instead of granting permissions individually to hundreds of resources:

```text
Table A → Grant
Table B → Grant
Table C → Grant
Table D → Grant
...
```

you can classify resources:

```text
LF-Tag:

Department = Finance
Sensitivity = Confidential
```

Then create permission policies based on those tags.

```text
Principal
   ↓
Permission:
Department = Finance
   ↓
All matching resources
```

> [!tip] Exam Pattern
> **Large data lake**
> +
> **Many databases/tables**
> +
> **Permissions based on classification**
>
> → ✅ LF-Tags / LF-TBAC

---

# 🆚 Named Resources vs LF-Tags

## Named Resource

Explicitly grant permissions:

```text
User
 ↓
SELECT
 ↓
Database A
 ↓
Table Orders
```

Think:

**Specific known resources**

---

## LF-TBAC

Grant based on attributes:

```text
User
 ↓
SELECT
 ↓
Department = Finance
```

Automatically applies to matching resources.

Think:

**Large-scale permission management**

> [!summary]
> Few specific resources
> → Named Resource
>
> Many dynamically changing resources
> → LF-Tags

---

# 🌐 Cross-Account Data Sharing

Very important for SAA.

```text
        PRODUCER ACCOUNT

             S3
              ↓
       Glue Data Catalog
              ↓
       Lake Formation
              ↓
             RAM
              ↓
      ─────────────────
              ↓
        CONSUMER ACCOUNT
              ↓
       Resource Link
              ↓
        Athena / etc.
```

Lake Formation supports sharing Data Catalog resources across AWS accounts.

AWS RAM is used behind the scenes for Lake Formation cross-account resource sharing.

> [!tip] Exam Pattern
> **Central data lake**
> +
> **Multiple AWS accounts**
> +
> **Fine-grained table/column access**
>
> → Lake Formation Cross-Account Sharing

---

# 🔗 Resource Links

A consumer account can create a **resource link** pointing to a shared Data Catalog resource.

```text
Producer Account

Database / Table
       ↓
   Shared
       ↓
─────────────────
       ↓
Consumer Account

Resource Link
       ↓
Athena
```

Resource links allow shared resources to appear in the consumer's Data Catalog.

> [!tip]
> **Cross-account Lake Formation + Athena**
>
> → Think **Resource Link**

---

# 🧩 Lake Formation + AWS RAM

Do not confuse their responsibilities.

```text
Lake Formation
→ Defines DATA permissions

AWS RAM
→ Facilitates cross-account resource sharing
```

Lake Formation automatically uses AWS RAM for supported cross-account Data Catalog sharing.

> [!danger]
> You normally don't choose RAM instead of Lake Formation when the
> requirement is fine-grained data governance.
>
> They can work together.

---

# 🔎 Lake Formation + Athena

Very common architecture:

```text
Analyst
   ↓
Athena
   ↓
Glue Data Catalog
   ↓
Lake Formation Permission Check
   ↓
S3 Data
```

Lake Formation can restrict what the analyst is allowed to query.

Example:

```text
Table: customers

id
name
email
credit_card
salary
```

Analyst could receive:

```text
SELECT:
id
name
email
```

without access to sensitive columns.

> [!tip] Exam Pattern
> **Athena + restrict access to specific columns**
>
> → Lake Formation

---

# 🆚 Lake Formation vs Glue Data Catalog

```text
Glue Data Catalog
→ METADATA

Lake Formation
→ GOVERNANCE
```

| Requirement | Think |
|---|---|
| Store table metadata | Glue Data Catalog |
| Discover schemas | Glue Crawler |
| Central data permissions | Lake Formation |
| Column/row-level permissions | Lake Formation |
| ETL | AWS Glue |
| Query S3 with SQL | Athena |

---

# 🆚 Lake Formation vs IAM

```text
IAM
→ AWS resource/API authorization

Lake Formation
→ Fine-grained data authorization
```

Example:

```text
IAM
→ User can call Athena APIs

Lake Formation
→ User can SELECT sales.orders
→ But NOT hr.employees
```

---

# 🆚 Lake Formation vs Macie

```text
Macie
→ DISCOVER sensitive data in S3

Lake Formation
→ CONTROL ACCESS to data
```

Example:

```text
"Find buckets containing PII"
→ Macie

"Only HR can query salary column"
→ Lake Formation
```

> [!summary]
> Macie = FIND sensitive data
>
> Lake Formation = PROTECT / GOVERN access

---

# 🆚 Lake Formation vs AWS RAM

```text
RAM
→ Share AWS resources across accounts

Lake Formation
→ Govern and share DATA with fine-grained permissions
```

Examples:

```text
Share Transit Gateway
→ RAM

Share VPC Subnet
→ RAM

Share Data Catalog table with SELECT permission
→ Lake Formation
```

---

# 🆚 Lake Formation vs AWS Organizations

```text
Organizations
→ Manage AWS accounts

RAM
→ Share AWS resources

Lake Formation
→ Govern data
```

Memory:

```text
ACCOUNTS
→ Organizations

RESOURCES
→ RAM

DATA
→ Lake Formation
```

---

# 👤 Data Lake Administrator

A Data Lake Administrator manages Lake Formation permissions and governance.

Think:

```text
Data Lake Administrator
        ↓
Register Locations
Manage Catalog Permissions
Grant / Revoke Access
```

> [!danger]
> Being a Data Lake Administrator does not automatically mean
> unrestricted SELECT access to every existing dataset.

Administration and data access are separate concepts.

---

# ⚠️ IAMAllowedPrincipals

Lake Formation can operate with legacy Glue/IAM-style access.

You may encounter:

```text
IAMAllowedPrincipals
```

This exists for backward compatibility with IAM-based Glue Data Catalog permissions.

For fine-grained Lake Formation governance, access is instead explicitly managed through Lake Formation permissions.

> [!tip] Exam Mental Model
> Legacy / IAM-only style
> → IAMAllowedPrincipals
>
> Fine-grained governance
> → Lake Formation permissions

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> **S3 data lake + centralized fine-grained permissions**
>
> → Lake Formation

> [!danger] Trap 2
> **Discover schema / metadata**
>
> → Glue Crawler + Glue Data Catalog
>
> Not Lake Formation.

> [!danger] Trap 3
> **Query S3 using SQL**
>
> → Athena
>
> Lake Formation controls permissions.

> [!danger] Trap 4
> **Find PII in S3**
>
> → Macie
>
> Not Lake Formation.

> [!danger] Trap 5
> **Share Transit Gateway / Subnet**
>
> → RAM
>
> Not Lake Formation.

> [!danger] Trap 6
> **Cross-account governed data sharing**
>
> → Lake Formation
>
> RAM can be used behind the scenes.

> [!danger] Trap 7
> **Large number of tables requiring scalable permissions**
>
> → LF-TBAC

> [!danger] Trap 8
> **DATA_LOCATION_ACCESS**
>
> → Permission to reference a registered S3 location
>
> NOT permission to SELECT the data.

---

# 🧠 Lake Formation in 30 Seconds

```text
DATA LAKE GOVERNANCE
→ Lake Formation

DATA
→ S3

METADATA
→ Glue Data Catalog

SCHEMA DISCOVERY
→ Glue Crawler

SQL QUERY
→ Athena

FINE-GRAINED DATA ACCESS
→ Lake Formation

MANY RESOURCES
→ LF-Tags

CROSS-ACCOUNT DATA
→ Lake Formation + RAM

CONSUMER CATALOG
→ Resource Link

FIND PII
→ Macie
```

> [!summary] SAA Memory
> **S3 = DATA**
>
> **GLUE CATALOG = METADATA**
>
> **ATHENA = QUERY**
>
> **LAKE FORMATION = GOVERN**
>
> **MACIE = DISCOVER SENSITIVE DATA**