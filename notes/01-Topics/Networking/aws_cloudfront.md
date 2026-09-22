# 🌍 Amazon CloudFront

> [!summary] Mental Model
> **CloudFront = CDN**
>
> Delivers content closer to users through AWS edge locations.
>
> ```text
> User
>   ↓
> Edge Location
>   ↓ cache miss
> CloudFront Origin
> ```

---

# 🎯 Core Purpose

Amazon CloudFront is AWS's Content Delivery Network (CDN).

Main benefits:

- Lower latency
- Global content delivery
- Caching at edge locations
- Reduced load on origins
- Integration with AWS WAF and AWS Shield

```text
Global Users
     ↓
CloudFront Edge Locations
     ↓
   Origin
```

---

# 🏠 Origins

CloudFront retrieves content from an **origin**.

Common origins:

- Amazon S3
- Application Load Balancer
- EC2 / HTTP server
- API Gateway
- Other HTTP origins

```text
             CloudFront
            /     |     \
           ↓      ↓      ↓
          S3     ALB    API
```

> [!tip] Exam Pattern
> **Serve content globally with low latency**
>
> → ✅ CloudFront

---

# ⚡ Edge Caching

```text
User
 ↓
Edge Location
 ↓
Cache HIT
 ↓
Return immediately ⚡
```

If the object is not cached:

```text
User
 ↓
Edge Location
 ↓
Cache MISS
 ↓
Origin
 ↓
Edge Cache
 ↓
User
```

Benefits:

- Lower latency
- Fewer requests to origin
- Reduced origin load

---

# ⏱️ TTL

TTL determines how long CloudFront keeps an object in cache.

```text
Origin Object
     ↓
CloudFront Cache
     ↓
     TTL
     ↓
Refresh from Origin
```

Think:

**Long TTL**
→ More caching  
→ Less origin traffic  
→ Potentially stale content

**Short TTL**
→ Fresher content  
→ More origin requests

---

# 🧹 Cache Invalidation

If content changes before TTL expires, CloudFront can invalidate cached objects.

```text
Old Object in Cache
        ↓
   Invalidation
        ↓
Remove from Edge Cache
        ↓
Fetch New Version
```

> [!tip] Exam Pattern
> **Content changed and must be updated immediately before TTL expires**
>
> → ✅ CloudFront Invalidation

Another common strategy:

```text
image-v1.jpg
      ↓
image-v2.jpg
```

→ **Versioned file names**

---

# 🪣 CloudFront + Private S3

Very important for SAA.

```text
Internet
   ↓
CloudFront
   ↓
OAC
   ↓
Private S3 Bucket
```

Use **Origin Access Control (OAC)** so users access the objects through CloudFront instead of directly through S3.

> [!tip] Exam Pattern
> **Private S3 content**
> +
> **Users must access through CloudFront**
>
> → ✅ CloudFront + OAC

> [!warning] OAI vs OAC
> **OAI = legacy**
>
> **OAC = recommended modern approach**

---

# 🔐 OAC vs Signed URL

These solve DIFFERENT problems.

```text
User
 ↓
Signed URL
 ↓
CloudFront
 ↓
OAC
 ↓
Private S3
```

**OAC**
→ Controls **CloudFront → S3**

**Signed URL / Cookie**
→ Controls **User → CloudFront**

> [!danger] Exam Trap
> OAC does NOT authenticate viewers.
>
> Signed URLs/Cookies do NOT replace OAC for restricting direct S3 access.

---

# 🔏 Signed URLs

Signed URLs provide temporary/restricted access to private CloudFront content.

```text
Authenticated User
       ↓
Application
       ↓
Signed URL
       ↓
CloudFront
       ↓
Private Content
```

Best when granting access to **individual files**.

Example:

```text
/private/video.mp4
?Expires=...
&Signature=...
```

> [!tip] Exam Pattern
> **Temporary access to one private CloudFront object**
>
> → ✅ Signed URL

---

# 🍪 Signed Cookies

Signed cookies are useful when a user needs access to **multiple restricted files**.

```text
Subscriber
    ↓
Signed Cookie
    ↓
CloudFront
    ↓
/premium/*
```

> [!tip] Exam Pattern
> **Access to many private files**
> +
> **Do not want to modify each URL**
>
> → ✅ Signed Cookies

### Signed URL vs Signed Cookie

| Requirement | Use |
|---|---|
| One/specific file | Signed URL |
| Multiple private files | Signed Cookies |
| Client does not support cookies | Signed URL |
| Keep existing URLs unchanged | Signed Cookies |

---

# 🆚 S3 Presigned URL vs CloudFront Signed URL

```text
S3 Presigned URL
      ↓
Direct S3 Access

CloudFront Signed URL
      ↓
CloudFront Edge
      ↓
Origin
```

> [!tip]
> **Temporary direct S3 access**
> → S3 Presigned URL
>
> **Private globally distributed cached content**
> → CloudFront Signed URL

---

# 🛣️ Cache Behaviors

Cache behaviors determine how CloudFront handles different URL patterns.

```text
CloudFront Distribution
│
├── /images/* → S3
├── /api/*    → ALB
└── Default   → S3
```

A behavior can control things such as:

- Origin
- Cache policy
- Allowed HTTP methods
- Viewer protocol policy

> [!tip] Exam Pattern
> **Different URL paths need different origins**
>
> → Cache Behaviors

---

# 🔀 Origin Failover

CloudFront can use an **origin group** containing:

```text
CloudFront
    ↓
Origin Group
├── Primary Origin
└── Secondary Origin
```

If the primary origin fails under configured conditions:

```text
Primary ❌
   ↓
CloudFront
   ↓
Secondary ✅
```

> [!tip] Exam Pattern
> **CloudFront + origin high availability**
>
> → Origin Group / Origin Failover

---

# 🛡️ CloudFront + WAF + Shield

```text
Internet
   ↓
Shield
   ↓
AWS WAF
   ↓
CloudFront
   ↓
Origin
```

Think:

**Shield**
→ DDoS

**WAF**
→ HTTP/HTTPS filtering  
→ SQL injection  
→ XSS  
→ Rate-based rules

**CloudFront**
→ CDN / edge caching

---

# 🌐 HTTPS

CloudFront supports HTTPS between:

```text
Viewer
 ↓ HTTPS
CloudFront
```

and CloudFront can use HTTPS to communicate with compatible origins.

Custom domain certificates for CloudFront use AWS Certificate Manager.

> [!tip]
> **CloudFront ACM certificate**
>
> → Remember **us-east-1 (N. Virginia)** for the viewer certificate.

---

# 💰 Price Classes

CloudFront Price Classes can limit which edge locations are used.

Mental model:

```text
More Edge Locations
→ Better global coverage
→ Potentially higher cost

Restricted Price Class
→ Lower cost
→ Potential latency tradeoff
```

> [!tip] Exam Pattern
> **Reduce CloudFront cost and geographic coverage can be limited**
>
> → Price Class

---

# 🛡️ Origin Shield

Origin Shield adds another caching layer between CloudFront edge locations and the origin.

```text
Users
  ↓
Edge Locations
  ↓
Origin Shield
  ↓
Origin
```

Benefits:

- Reduce requests reaching origin
- Improve cache hit ratio
- Protect origin from request spikes

---

# 🆚 CloudFront vs Global Accelerator

Very important for SAA.

## CloudFront

```text
HTTP / HTTPS Content
        ↓
Edge Cache
        ↓
Origin
```

Think:

- CDN
- Content caching
- Static/dynamic web content
- HTTP/HTTPS

## Global Accelerator

```text
User
 ↓
AWS Global Network
 ↓
Regional Endpoint
```

Think:

- TCP/UDP
- Static Anycast IP addresses
- Global network routing
- Applications that should not rely on caching

| Requirement | Think |
|---|---|
| CDN / caching | **CloudFront** |
| Static website assets | **CloudFront** |
| Video/content distribution | **CloudFront** |
| TCP/UDP application | **Global Accelerator** |
| Static Anycast IPs | **Global Accelerator** |
| Global routing without caching | **Global Accelerator** |

> [!danger] Exam Trap
> **Global users** alone does NOT automatically mean CloudFront.
>
> Ask:
>
> **Do I need caching/content delivery?**
> → CloudFront
>
> **Do I need TCP/UDP acceleration/static Anycast IPs?**
> → Global Accelerator

---

# 🆚 CloudFront vs S3 Transfer Acceleration

```text
CloudFront
→ DOWNLOAD / deliver content to users
→ Cache at edge

S3 Transfer Acceleration
→ UPLOAD/DOWNLOAD objects to/from S3 faster over long distances
→ Uses edge network
→ No CDN cache
```

> [!tip] Exam Pattern
> Users worldwide repeatedly download the same content
> → CloudFront
>
> Users worldwide upload large files to one S3 bucket
> → S3 Transfer Acceleration

---

# 🆚 CloudFront vs Route 53

```text
Route 53
→ DNS

CloudFront
→ CDN / Content Delivery
```

Route 53 decides **where DNS points**.

CloudFront delivers/caches **content**.

---

# ⚠️ High-Value Exam Traps

> [!danger] Trap 1
> **Private S3 + only CloudFront should access origin**
>
> → OAC

> [!danger] Trap 2
> **Temporary access to ONE private CloudFront object**
>
> → Signed URL

> [!danger] Trap 3
> **Access to MULTIPLE private CloudFront objects**
>
> → Signed Cookies

> [!danger] Trap 4
> **Immediately remove stale cached content**
>
> → Invalidation

> [!danger] Trap 5
> **CloudFront origin high availability**
>
> → Origin Group / Origin Failover

> [!danger] Trap 6
> **Global TCP/UDP application**
>
> → Global Accelerator, not CloudFront

> [!danger] Trap 7
> **Worldwide large uploads to S3**
>
> → S3 Transfer Acceleration

> [!danger] Trap 8
> **SQL injection / XSS at the edge**
>
> → WAF + CloudFront

---

# 🧠 CloudFront in 30 Seconds

```text
Global Content
→ CloudFront

Cache
→ Edge Locations

Private S3
→ OAC

One Private File
→ Signed URL

Many Private Files
→ Signed Cookies

Stale Cached Content
→ Invalidation

Origin HA
→ Origin Group

Extra Origin Cache Layer
→ Origin Shield

Web Attacks
→ WAF

DDoS
→ Shield

TCP/UDP + Static IP
→ Global Accelerator

Fast Global S3 Upload
→ Transfer Acceleration
```

> [!summary] SAA Memory
> **CDN = CLOUDFRONT**
>
> **PRIVATE S3 = OAC**
>
> **PRIVATE VIEWER ACCESS = SIGNED URL / COOKIE**
>
> **ORIGIN HA = ORIGIN GROUP**
>
> **TCP/UDP = GLOBAL ACCELERATOR**