# 🛡️ AWS WAF

> [!summary] Mental Model
> **AWS WAF = Layer 7 protection for web applications and APIs**
>
> Inspects **HTTP/HTTPS requests** and filters malicious or unwanted web traffic.
>
> **WAF → Web request protection**  
> **Inspector → Vulnerability scanning**  
> **GuardDuty → Threat detection**  
> **Shield → DDoS protection**

---

## 📌 Core

AWS WAF protects web applications and APIs by inspecting incoming **HTTP/HTTPS requests**.

Can inspect request characteristics such as:

- IP address
- Country / geographic location
- URI path
- Query strings
- HTTP headers
- Request body
- SQL Injection patterns
- Cross-Site Scripting (XSS)

> [!tip] 🎯 Exam Clue
> **Malicious HTTP/HTTPS requests**
> → Think **AWS WAF**

---

# 📜 Web ACL

A **Web ACL** contains the rules that AWS WAF evaluates.

```text
HTTP Request
     ↓
   Web ACL
     ↓
┌───────────────┐
│ Rule 1        │
│ Rule 2        │
│ Rule 3        │
└───────────────┘
     ↓
Allow / Block / Count
```

Common actions:

- **ALLOW** → Allow the request
- **BLOCK** → Block the request
- **COUNT** → Count matching requests without blocking

A Web ACL also has a **default action** for requests that don't match any rule.

---

# ⚙️ WAF Rules

## 📏 Regular Rule

Matches requests based on **specific characteristics or attack patterns**.

Can be used to:

- Block specific IP addresses
- Block requests from specific countries
- Filter URI paths
- Inspect HTTP headers
- Inspect query strings
- Inspect request bodies
- Detect SQL Injection
- Detect XSS

### Example

```text
Request contains SQL Injection
            ↓
      Regular Rule
            ↓
         BLOCK 🚫
```

> [!tip] 🎯 Exam Clue
> **Known characteristic or attack pattern**
> → Regular Rule

Examples:

`Specific IP → Regular Rule`

`Specific country → Regular Rule`

`SQL Injection → Regular Rule`

`XSS → Regular Rule`

---

## 🚦 Rate-Based Rule

Tracks the **rate of incoming requests** and applies an action when traffic exceeds the configured threshold.

Useful for:

- Excessive HTTP requests
- HTTP floods
- Abusive clients
- Bots
- Brute-force-like traffic

### Example

```text
Client A → normal rate      → ALLOW ✅
Client B → normal rate      → ALLOW ✅
Client C → excessive rate   → BLOCK 🚫
```

AWS WAF tracks request rates and can temporarily apply the configured action to sources generating excessive traffic.

> [!tip] 🎯 Exam Clue
> **"High number of requests"**
>
> **"Excessive requests"**
>
> **"Limit request rate"**
>
> **"Minimize impact on legitimate users"**
>
> → **Rate-Based Rule**

---

## 🧠 Regular vs Rate-Based

| Requirement | Rule |
|---|---|
| Block specific IP | **Regular** |
| Block specific country | **Regular** |
| Filter URI/header/body | **Regular** |
| SQL Injection | **Regular** |
| XSS | **Regular** |
| Excessive request rate | **Rate-Based** |
| HTTP flood | **Rate-Based** |
| Rate-limit abusive clients | **Rate-Based** |

> [!warning] ⚠️ Exam Trap
> **Regular Rule → WHAT does the request look like?**
>
> **Rate-Based Rule → HOW MANY requests are being sent?**

---

# 🧰 AWS Managed Rules

AWS provides **preconfigured rule groups** for common web threats.

Useful for protection against:

- SQL Injection
- XSS
- Known bad inputs
- Common web vulnerabilities

Reduces the need to manually create and maintain every security rule.

> [!tip] 🎯 Exam Clue
> **Common web exploits + minimal operational overhead**
> → **AWS Managed Rules**

---

# 🌐 Integrations

AWS WAF can protect services such as:

- **Amazon CloudFront**
- **Application Load Balancer (ALB)**
- **Amazon API Gateway**
- **AWS AppSync**

---

## ☁️ CloudFront + WAF

```text
Internet
   ↓
CloudFront + WAF 🛡️
   ↓
Origin
```

Malicious requests can be blocked before reaching the origin.

> [!tip] 🎯 Exam Clue
> **Global web application + CloudFront + malicious requests**
> → WAF + CloudFront

---

## ⚖️ ALB + WAF

```text
Internet
   ↓
ALB + WAF 🛡️
   ↓
EC2 / ECS
```

Useful for protecting regional web applications behind an ALB.

> [!tip] 🎯 Exam Clue
> **ALB + malicious HTTP requests**
> → AWS WAF

---

# 🔍 Monitoring

## Amazon CloudWatch

WAF provides metrics such as:

- Allowed requests
- Blocked requests
- Counted requests
- Rule activity

## WAF Logging

Can provide information about requests such as:

- Source IP
- URI
- User-Agent
- Geographic information
- Request details

Useful for:

→ Security analysis  
→ Troubleshooting  
→ Forensics

---

# ⚔️ WAF vs Shield

| | AWS WAF | AWS Shield |
|---|---|---|
| Main purpose | Web request filtering | DDoS protection |
| HTTP inspection | ✅ | ❌ |
| SQL Injection | ✅ | ❌ |
| XSS | ✅ | ❌ |
| Rate-based rules | ✅ | ❌ |
| DDoS protection | Limited via rules | ✅ |

### WAF

Think:

- SQL Injection
- XSS
- HTTP filtering
- IP filtering
- URI/header/body inspection
- Rate limiting

### Shield

Think:

- DDoS
- Volumetric attacks
- Network/transport attacks

> [!tip] 🧠 Mental Model
> **WAF → Is the web request malicious?**
>
> **Shield → Is someone trying to overwhelm the application?**

WAF and Shield can be used **together**.

---

# 🔎 WAF vs Inspector

This is another important distinction.

| | AWS WAF | Amazon Inspector |
|---|---|---|
| Focus | Web traffic | Workload vulnerabilities |
| HTTP filtering | ✅ | ❌ |
| SQLi / XSS blocking | ✅ | ❌ |
| Rate limiting | ✅ | ❌ |
| CVE detection | ❌ | ✅ |
| Vulnerable packages | ❌ | ✅ |
| EC2 vulnerability scanning | ❌ | ✅ |
| ECR image scanning | ❌ | ✅ |
| Lambda vulnerability scanning | ❌ | ✅ |

### Example

```text
Attacker
   │
   │ SQLi / XSS / malicious HTTP
   ↓
🛡️ WAF
   ↓
Application
   │
   ├── EC2 vulnerable package
   ├── ECR vulnerable image
   └── Lambda vulnerable dependency
                ↑
          🔍 Inspector
```

> [!tip] 🎯 Exam Clue
> **Block malicious web requests**
> → WAF
>
> **Find CVEs / vulnerable software**
> → Inspector

---

# 🕵️ WAF vs GuardDuty

### AWS WAF

→ **BLOCK/FILTER malicious web requests**

### Amazon GuardDuty

→ **DETECT suspicious or malicious activity**

Examples:

**WAF**
- SQL Injection
- XSS
- Excessive HTTP requests

**GuardDuty**
- Suspicious API activity
- Compromised credentials
- Unusual network behavior
- Potentially compromised workloads

> [!tip] 🧠 Mental Model
> **WAF → Block web attacks**
>
> **Inspector → Find vulnerabilities**
>
> **GuardDuty → Detect threats**

---

# 🧱 WAF vs NACL

### AWS WAF

Works at the **web/application layer**.

Understands things like:

```text
GET /login
POST /checkout
Headers
URI
SQL Injection
XSS
```

### Network ACL

Works at the **subnet/network layer**.

Controls:

- IP addresses
- Protocols
- Ports
- Allow/Deny rules

Does NOT inspect application-level HTTP content.

> [!warning] ⚠️ Exam Trap
> **SQLi / XSS / HTTP rate limiting**
> → WAF
>
> **Subnet-level IP/port filtering**
> → NACL

---

# 🔐 WAF vs Security Groups

| | WAF | Security Group |
|---|---|---|
| Focus | Web requests | Network access |
| HTTP inspection | ✅ | ❌ |
| SQLi/XSS | ✅ | ❌ |
| Rate-based rules | ✅ | ❌ |
| Port filtering | ❌ | ✅ |
| Stateful | — | ✅ |
| Rules | Allow/Block/etc. | Allow only |

> [!tip] 🎯 Exam Clue
> **Allow HTTPS to EC2**
> → Security Group
>
> **Block malicious HTTPS requests**
> → WAF

---

# 🌐 WAF vs Network Firewall

### AWS WAF

→ Web applications  
→ HTTP/HTTPS  
→ Application-level web filtering

### AWS Network Firewall

→ VPC network traffic  
→ Network traffic inspection/filtering

```text
Web Request Security
        ↓
       WAF

VPC Network Security
        ↓
 AWS Network Firewall
```

---

# 🔗 WAF vs PrivateLink

### AWS PrivateLink

Provides **private connectivity** between services/VPCs.

It does NOT:

- Inspect HTTP attacks
- Detect SQL Injection
- Detect XSS
- Rate-limit malicious clients

> [!warning] ⚠️ Exam Trap
> **PrivateLink → Private connectivity**
>
> **WAF → Web request protection**

---

# 📝 Practice Scenario

### Architecture

```text
Internet
   ↓
AWS WAF
   ↓
Application Load Balancer
   ↓
EC2 Auto Scaling Group
```

### Problem

- High number of illegitimate requests
- Multiple external systems
- Frequently changing traffic sources
- Must minimize impact on legitimate users

### Solution

**WAF Rate-Based Rule + Web ACL associated with ALB**

### Why?

The rate-based rule identifies sources generating excessive requests and applies the configured action while normal traffic continues.

### Why NOT Regular Rule?

A regular rule matches defined request characteristics.

The main requirement here is:

**Excessive request rate**

→ **Rate-Based Rule**

### Why NOT NACL?

NACL does not provide application-level HTTP rate-based filtering.

### Why NOT PrivateLink?

PrivateLink provides private connectivity, not web request filtering.

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| SQL Injection | **WAF** |
| XSS | **WAF** |
| Malicious HTTP requests | **WAF** |
| Excessive HTTP requests | **WAF Rate-Based Rule** |
| HTTP flood | **WAF Rate-Based Rule** |
| Filter URI/header/body | **WAF Regular Rule** |
| Common web exploits + low ops | **AWS Managed Rules** |
| Protect ALB web application | **WAF + ALB** |
| Protect CloudFront web traffic | **WAF + CloudFront** |
| DDoS | **Shield** |
| Find CVEs | **Inspector** |
| Vulnerable ECR image | **Inspector** |
| Suspicious activity | **GuardDuty** |
| Subnet IP/port filtering | **NACL** |
| Stateful network access | **Security Group** |
| VPC network inspection | **Network Firewall** |
| Private connectivity | **PrivateLink** |

---

> [!abstract] 🧠 AWS WAF in 20 Seconds
> **Purpose:** Protect web applications and APIs
>
> **Traffic:** HTTP/HTTPS
>
> **Web ACL:** Collection of WAF rules
>
> **Regular Rule:** WHAT does the request look like?
>
> **Rate-Based Rule:** HOW MANY requests?
>
> **Managed Rules:** Preconfigured protection
>
> **SQLi / XSS:** WAF
>
> **HTTP flood:** WAF Rate-Based Rule
>
> **CVEs:** Inspector
>
> **Threat detection:** GuardDuty
>
> **DDoS:** Shield
>
> **Subnet filtering:** NACL