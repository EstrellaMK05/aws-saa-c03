# 🌱 AWS Elastic Beanstalk

> [!summary] Mental Model
> **Elastic Beanstalk = Deploy my application without manually managing the infrastructure**
>
> ```text
> Application Code
>       ↓
> Elastic Beanstalk
>       ↓
> AWS Infrastructure
> ├── EC2
> ├── Auto Scaling
> ├── Load Balancer
> └── CloudWatch
> ```

---

# 🎯 Core Purpose

AWS Elastic Beanstalk is a managed application deployment service.

You provide the application code and Elastic Beanstalk handles much of the infrastructure provisioning, deployment, monitoring, and scaling.

```text
Developer
   ↓
Application Code
   ↓
Elastic Beanstalk
   ↓
Deploy + Scale + Monitor
```

> [!tip] Exam Pattern
> **Deploy a traditional web application**
> +
> **Minimal infrastructure management**
> +
> **Still need access/control over underlying AWS resources**
>
> → ✅ Elastic Beanstalk

---

# 🏗️ Core Concepts

```text
Application
   ↓
Application Versions
   ↓
Environments
```

### Application

Logical container for:

- Environments
- Application versions
- Configurations

### Application Version

A specific version of deployable application code.

### Environment

AWS resources running one application version.

Examples:

```text
Application: MyApp

├── dev environment
├── test environment
└── prod environment
```

---

# 🌐 Web Server Environment

Used for applications that serve HTTP requests.

Classic architecture:

```text
             Internet
                ↓
        Elastic Load Balancer
          ↙            ↘
       EC2              EC2
          \            /
           Auto Scaling
```

A load-balanced scalable environment commonly uses:

- Elastic Load Balancing
- EC2
- Auto Scaling
- CloudWatch

> [!tip] Exam Pattern
> **Highly available scalable web application**
>
> → Elastic Beanstalk load-balanced environment

---

# 👷 Worker Environment

Used for background/asynchronous processing.

```text
Web Application
      ↓
     SQS
      ↓
Elastic Beanstalk
Worker Environment
      ↓
Background Processing
```

Elastic Beanstalk worker environments use an **SQS queue** and worker instances process messages from it.

> [!tip] Exam Pattern
> **Background tasks**
> +
> **Asynchronous processing**
>
> → Elastic Beanstalk Worker Environment

---

# 🆚 Web vs Worker

| Requirement | Environment |
|---|---|
| HTTP web application | Web Server |
| User-facing requests | Web Server |
| Background processing | Worker |
| SQS-based jobs | Worker |

```text
HTTP Request
→ Web Environment

SQS Message
→ Worker Environment
```

---

# 📈 Auto Scaling

Elastic Beanstalk can manage EC2 Auto Scaling for Standard environments.

```text
Traffic ↑
   ↓
Auto Scaling
   ↓
More EC2

Traffic ↓
   ↓
Auto Scaling
   ↓
Fewer EC2
```

> [!important]
> Elastic Beanstalk does not replace Auto Scaling.
>
> It can **configure and manage Auto Scaling resources for you**.

---

# 🚀 Deployment Policies

## All at Once

Deploy to all instances simultaneously.

```text
v1 v1 v1
   ↓
v2 v2 v2
```

Fastest deployment.

❌ Causes temporary downtime.

---

## Rolling

Deploy in batches.

```text
v1 v1 v1 v1

↓ batch 1

v2 v2 v1 v1

↓ batch 2

v2 v2 v2 v2
```

Capacity is reduced during deployment.

---

## Rolling with Additional Batch

Launch an additional batch first.

```text
Existing Capacity
      +
New Batch
      ↓
Deploy gradually
```

Maintains full capacity during deployment.

---

## Immutable

Launch a fresh set of instances with the new version.

```text
Old Instances
   v1 v1
      +
New Instances
   v2 v2

Validation
   ↓
Replace old
```

Safer rollback.

Uses additional resources during deployment.

---

## Traffic Splitting

Temporarily send a percentage of traffic to the new version.

```text
Users
  ↓
  ├── 90% → v1
  └── 10% → v2
```

Useful for canary-style validation.

---

# 🧠 Deployment Comparison

| Deployment | Downtime | Extra Capacity | Mental Model |
|---|---|---|---|
| All at Once | Yes | No | Fast |
| Rolling | No* | No | Batches |
| Rolling + Batch | No | Yes | Keep capacity |
| Immutable | No | Yes | Fresh instances |
| Traffic Splitting | No | Yes | Canary traffic |

> [!tip] Exam Pattern
> **Fastest deployment and downtime acceptable**
> → All at Once
>
> **Deploy gradually without extra capacity**
> → Rolling
>
> **Maintain full capacity**
> → Rolling with Additional Batch
>
> **Safest deployment / easy rollback**
> → Immutable
>
> **Test new version with percentage of real traffic**
> → Traffic Splitting

---

# 🔵 Blue/Green Deployment

Another important Elastic Beanstalk pattern:

```text
BLUE Environment
Production v1

GREEN Environment
New v2
       ↓
Test Green
       ↓
Swap Environment URLs
       ↓
GREEN becomes Production
```

Instead of updating the existing environment:

1. Create a second environment.
2. Deploy new version.
3. Test it.
4. Swap environment CNAMEs.

> [!tip] Exam Pattern
> **Deploy new environment**
> +
> **Test before production**
> +
> **Quick switch / rollback**
>
> → ✅ Blue/Green Deployment

---

# 🔐 IAM Roles

Elastic Beanstalk Standard commonly involves two IAM concepts:

### Service Role

```text
Elastic Beanstalk
      ↓
Service Role
      ↓
Manage AWS Resources
```

Allows Elastic Beanstalk to perform operations on your behalf.

### EC2 Instance Profile

```text
EC2
 ↓
Instance Profile
 ↓
S3 / CloudWatch / other AWS services
```

Provides permissions to EC2 instances running the application.

> [!danger] Don't Confuse
> **Service Role**
> → Elastic Beanstalk permissions
>
> **Instance Profile**
> → EC2 application permissions

---

# ⚙️ Configuration

Elastic Beanstalk allows configuration through:

- Console
- EB CLI
- AWS CLI
- Configuration files

Application-specific environment configuration can be managed alongside deployments.

---

# 💰 Pricing

There is **no additional charge for Elastic Beanstalk itself**.

You pay for the AWS resources used by the environment, such as:

- EC2
- Load Balancer
- RDS
- S3
- CloudWatch

> [!tip]
> **Elastic Beanstalk itself → no additional service charge**
>
> **Underlying resources → you pay**

---

# 🆚 Elastic Beanstalk vs CloudFormation

```text
Elastic Beanstalk
→ Deploy APPLICATIONS

CloudFormation
→ Deploy INFRASTRUCTURE
```

| Requirement | Think |
|---|---|
| Deploy web application easily | Beanstalk |
| Define AWS infrastructure as code | CloudFormation |
| Full infrastructure control | CloudFormation |
| Managed application platform | Beanstalk |

> [!danger]
> Elastic Beanstalk is **not an IaC replacement for CloudFormation**.

---

# 🆚 Elastic Beanstalk vs Lambda

```text
Traditional Application
→ Elastic Beanstalk

Event-Driven Serverless Code
→ Lambda
```

| Requirement | Think |
|---|---|
| Traditional web app | Beanstalk |
| Long-running application | Beanstalk |
| Event-driven function | Lambda |
| Sudden burst within seconds | Lambda |
| No server management | Lambda |

> [!danger] Exam Trap
> **Sudden unpredictable burst that must scale within seconds**
>
> → Lambda
>
> Don't choose Elastic Beanstalk merely because it supports Auto Scaling.

---

# 🆚 Elastic Beanstalk vs ECS

```text
Application Deployment Platform
→ Elastic Beanstalk

Container Orchestration
→ ECS
```

If the question specifically emphasizes:

- Containers
- Tasks
- Services
- Container orchestration

→ Think ECS.

---

# 🆚 Elastic Beanstalk vs OpsWorks

```text
Application Deployment
→ Elastic Beanstalk

Configuration Management
Chef / Puppet
→ OpsWorks
```

For SAA, Beanstalk is much more important.

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> **Elastic Beanstalk manages infrastructure, but you still own/pay for the underlying resources.**

---

> [!danger] Trap 2
> **Background processing + SQS**
>
> → Worker Environment

---

> [!danger] Trap 3
> **Fastest deployment + downtime acceptable**
>
> → All at Once

---

> [!danger] Trap 4
> **Deployment in batches**
>
> → Rolling

---

> [!danger] Trap 5
> **Maintain full capacity during rolling deployment**
>
> → Rolling with Additional Batch

---

> [!danger] Trap 6
> **Fresh instances + safer rollback**
>
> → Immutable

---

> [!danger] Trap 7
> **Percentage of production traffic to new version**
>
> → Traffic Splitting

---

> [!danger] Trap 8
> **Completely separate environment + switch URLs**
>
> → Blue/Green

---

# 🆕 Current AWS Note

Elastic Beanstalk currently has:

```text
Elastic Beanstalk
│
├── Standard
│   └── EC2 + Auto Scaling
│
└── Cluster
    └── Containers + EKS
```

For **SAA exam questions**, prioritize the classic **Beanstalk Standard / EC2 model** unless the question explicitly mentions the newer cluster/container model.

---

# 🧠 Elastic Beanstalk in 20 Seconds

```text
Deploy Web Application
→ Elastic Beanstalk

HTTP
→ Web Environment

SQS / Background Jobs
→ Worker Environment

Scaling
→ Auto Scaling

Fast + Downtime OK
→ All at Once

Batches
→ Rolling

Batches + Full Capacity
→ Rolling + Additional Batch

Fresh Instances / Safe Rollback
→ Immutable

Small % Real Traffic
→ Traffic Splitting

Separate Environment
→ Blue/Green
```

> [!summary] SAA Memory
> **BEANSTALK = APPLICATION DEPLOYMENT**
>
> **WEB = HTTP**
>
> **WORKER = SQS**
>
> **IMMUTABLE = NEW INSTANCES**
>
> **BLUE/GREEN = NEW ENVIRONMENT + SWAP**