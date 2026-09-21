# 🚪 Amazon API Gateway

> [!summary] Mental Model
> **API Gateway = Managed Front Door for APIs**
>
> ```text
> Client
>   ↓
> API Gateway
>   ↓
> Backend
> ├── Lambda
> ├── HTTP Endpoint
> └── AWS Service
> ```

---

# 🎯 Core Purpose

Amazon API Gateway is a managed service for creating, publishing, securing, monitoring, and managing APIs.

Common architecture:

```text
Web / Mobile Client
        ↓
   API Gateway
        ↓
      Lambda
        ↓
   DynamoDB / RDS
```

> [!tip] Exam Pattern
> **Serverless API**
> +
> **No servers to manage**
>
> → ✅ API Gateway + Lambda

---

# 🔌 API Types

## REST API

Think:

- Full API Gateway feature set
- RESTful applications
- API keys / usage plans
- Caching
- Request/response transformations

```text
GET /users
POST /orders
DELETE /items/123
```

---

## HTTP API

Think:

- Simpler APIs
- Lower cost
- Lower latency
- Commonly Lambda / HTTP backends

> [!tip]
> Simple, cost-effective HTTP API
>
> → Think **HTTP API**

---

## WebSocket API

Used for persistent, two-way communication.

```text
Client ←────────→ API Gateway
                   WebSocket
                      ↓
                    Lambda
```

Common use cases:

- Chat applications
- Real-time dashboards
- Live notifications

> [!tip] Exam Pattern
> **Real-time + two-way communication**
>
> → ✅ WebSocket API

---

# ⚡ API Gateway + Lambda

One of the most important SAA architectures:

```text
Users
  ↓
API Gateway
  ↓
Lambda
  ↓
Database
```

Benefits:

- Serverless
- Automatic scaling
- No EC2 management
- Good for unpredictable traffic

> [!tip] Exam Pattern
> **API + sudden traffic bursts**
>
> → API Gateway + Lambda

---

# 🔐 Authentication & Authorization

API Gateway can protect APIs using mechanisms such as:

- IAM authorization
- Amazon Cognito
- Lambda Authorizers

### IAM

Think:

**AWS identities**

```text
IAM Principal
     ↓
SigV4 Request
     ↓
API Gateway
```

### Cognito

Think:

**Application users**

```text
Mobile / Web User
       ↓
Amazon Cognito
       ↓
JWT / Token
       ↓
API Gateway
```

### Lambda Authorizer

Think:

**Custom authorization logic**

```text
Client Token
     ↓
API Gateway
     ↓
Lambda Authorizer
     ↓
Allow / Deny
```

> [!summary] Memory
> AWS identity → IAM
>
> App users → Cognito
>
> Custom authorization → Lambda Authorizer

---

# 🚦 Throttling

API Gateway can throttle incoming requests.

```text
Too Many Requests
       ↓
API Gateway
       ↓
Throttle
       ↓
429 Too Many Requests
```

Two concepts:

**Rate**
→ Sustained requests per second

**Burst**
→ Temporary spike in requests

> [!tip] Exam Pattern
> **Protect backend from too many API requests**
>
> → ✅ API Gateway Throttling

---

# 🎟️ API Keys & Usage Plans

Usage plans can control how clients consume a REST API.

Can define:

- API keys
- Throttling
- Quotas

```text
Client
  ↓
API Key
  ↓
Usage Plan
  ↓
API Gateway
```

> [!danger]
> **API Keys are NOT an authentication/authorization mechanism.**
>
> They are mainly used for:
>
> - Identifying clients
> - Metering
> - Usage plans
> - Quotas
> - Throttling

Do not use an API key as the main security mechanism for sensitive APIs.

---

# ⚡ API Gateway Caching

API Gateway REST APIs can cache backend responses.

```text
Client
  ↓
API Gateway
  ↓
Cache HIT ⚡
  ↓
Return Response

Cache MISS
  ↓
Backend
```

Benefits:

- Lower backend load
- Lower latency
- Fewer Lambda/backend calls

> [!tip] Exam Pattern
> **Same API GET requests repeatedly**
> +
> **Reduce backend calls**
>
> → ✅ API Gateway Cache

Default cache TTL:

**300 seconds**

Maximum:

**3600 seconds**

---

# 🏗️ Stages

Stages represent different API deployment environments.

Examples:

```text
/dev
/test
/prod
```

A stage points to an API deployment.

Stage-specific configuration can include:

- Logging
- Throttling
- Caching
- Stage variables

> [!tip]
> **Different API environments**
>
> → API Gateway Stages

---

# 🐤 Canary Deployments

REST APIs can use canary releases.

```text
Users
  ↓
API Gateway
  ├── 90% → Current Version
  └── 10% → New Version
```

Useful for gradually testing a new API deployment.

---

# 🌐 Endpoint Types

For REST APIs, know these concepts:

### Edge-Optimized

Think:

**Geographically distributed clients**

Uses the AWS edge network to improve access for global clients.

### Regional

Think:

**Clients primarily in the same Region**

### Private

Think:

**API accessible privately from a VPC**

```text
VPC
 ↓
Interface VPC Endpoint
 ↓
Private API Gateway
```

> [!tip] Exam Pattern
> Internal/private API that must not be publicly accessible
>
> → **Private API**

---

# 🛡️ API Gateway + WAF

AWS WAF can protect API Gateway REST APIs against malicious web requests.

```text
Internet
   ↓
AWS WAF
   ↓
API Gateway
   ↓
Lambda
```

Think:

- SQL injection
- XSS
- HTTP request filtering
- Rate-based rules

> [!danger] Don't Confuse
> **API Gateway Throttling**
> → Control API request rate
>
> **WAF**
> → Inspect/filter malicious HTTP requests
>
> **Shield**
> → DDoS protection

---

# 🌍 CORS

CORS controls whether browser applications from another origin can call the API.

```text
frontend.com
     ↓
API
api.example.com
```

> [!tip] Exam Pattern
> **Browser frontend hosted on one domain cannot call API on another domain**
>
> → Check **CORS**

---

# 🔄 Request / Response Transformation

REST APIs can transform requests and responses before passing them between clients and integrations.

```text
Client Request
      ↓
API Gateway
Transform
      ↓
Backend
```

Useful when the client and backend expect different formats.

---

# 🆚 API Gateway vs ALB

Both can route HTTP traffic, but think differently.

| Requirement | Think |
|---|---|
| Managed API | API Gateway |
| Serverless API + Lambda | API Gateway |
| API keys / usage plans | API Gateway |
| API caching | API Gateway |
| WebSocket API | API Gateway |
| Route traffic to EC2/ECS | ALB |
| Host-based/path-based load balancing | ALB |

> [!tip]
> **"Create/manage an API"**
> → API Gateway
>
> **"Load balance EC2/ECS"**
> → ALB

---

# 🆚 API Gateway vs CloudFront

```text
CloudFront
→ CDN
→ Cache/distribute content globally

API Gateway
→ API management
→ Authentication
→ Throttling
→ API routing
```

They can also be used together.

---

# 🆚 API Gateway vs Lambda Function URL

```text
Lambda Function URL
→ Simple HTTP endpoint directly to Lambda

API Gateway
→ Full API management
→ Routes
→ Authorization
→ Throttling
→ API lifecycle features
```

> [!tip]
> Simple Lambda HTTP endpoint
> → Function URL
>
> Full managed API
> → API Gateway

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> Sudden API traffic bursts + serverless
>
> → API Gateway + Lambda

> [!danger] Trap 2
> Protect backend from excessive requests
>
> → API Gateway throttling

> [!danger] Trap 3
> Repeated GET requests are overloading backend
>
> → API Gateway caching

> [!danger] Trap 4
> Real-time bidirectional application
>
> → WebSocket API

> [!danger] Trap 5
> Authenticate application users
>
> → Cognito
>
> Not API Keys.

> [!danger] Trap 6
> Custom authentication logic
>
> → Lambda Authorizer

> [!danger] Trap 7
> API accessible only privately from VPC
>
> → Private API + VPC Endpoint

> [!danger] Trap 8
> SQL injection / XSS against API
>
> → AWS WAF

---

# 🧠 API Gateway in 20 Seconds

```text
Managed API
→ API Gateway

Serverless API
→ API Gateway + Lambda

App Users
→ Cognito

Custom Auth
→ Lambda Authorizer

Control Request Rate
→ Throttling

Repeated GET Requests
→ Cache

Client Usage Limits
→ Usage Plans

Real-Time Two-Way
→ WebSocket

Private VPC API
→ Private API

Malicious HTTP Requests
→ WAF

Browser Cross-Origin Error
→ CORS
```

> [!summary] SAA Memory
> **API → API Gateway**
>
> **API + SERVERLESS → Lambda**
>
> **TOO MANY REQUESTS → Throttling**
>
> **REPEATED READS → Cache**
>
> **REAL-TIME TWO-WAY → WebSocket**
>
> **APP AUTH → Cognito**