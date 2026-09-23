---
aliases: [Amazon CloudFront]
tags: [aws, saa-c03, networking]
---
# Amazon CloudFront

## Mental Model

**CloudFront = an HTTP/HTTPS delivery layer near your viewers.** A cache hit serves content from the edge; a miss retrieves content from an origin. It can also accelerate dynamic requests without caching their responses.

## Core

### Origins and behaviors

Origins include S3 REST endpoints, HTTP servers, ALBs, and API Gateway. Cache behaviors match ordered path patterns to an origin and processing configuration: `/images/*` can go to S3 while `/api/*` goes to an ALB.

Supported **VPC origins** allow private ALB, NLB, or EC2 origins through service-managed connectivity. Do not assume every CloudFront origin must be publicly reachable. Check VPC-origin limitations for the required protocol and origin type before choosing it.

### Cache correctness before cache hit rate

A **cache policy** determines TTL settings and which request values form the cache key, including configured headers, cookies, and query strings. Requests with the same key can receive the same cached representation.

An **origin request policy** can forward additional request values without adding them to the cache key. Forwarding a user identifier while ignoring it in the key can be unsafe if personalized responses are cached together. Disable caching for sensitive personalized responses or design correct isolation and authorization.

Avoid including irrelevant tracking parameters in the key: they fragment the cache. TTL controls freshness versus origin load. A minimum TTL greater than zero can cause caching even when the origin sends `no-cache`, `no-store`, or `private` directives; select policies deliberately.

CloudFront caches GET/HEAD responses and optionally OPTIONS. Allowing other HTTP methods does not make POST/PUT responses cacheable.

Use versioned asset filenames for routine deployments. Invalidation removes selected cached paths before normal expiration, but propagation is not instantaneous and it does not clear viewers' independent browser caches.

### Two separate access boundaries

| Boundary | Mechanism | Purpose |
|---|---|---|
| CloudFront to private S3 origin | OAC and bucket policy | Restrict origin reads to the intended distribution |
| Viewer to restricted CloudFront content | Signed URL or signed cookies | Time-limited/policy-constrained viewer access |

OAC is preferred over legacy OAI. Use a regular S3 bucket REST origin, keep public access blocked, and authorize the distribution in the bucket policy. For SSE-KMS objects, configure the required key permissions too.

An **S3 website endpoint** is a custom HTTP origin: it does not support OAC/OAI or HTTPS from CloudFront to that website endpoint. If private S3 and HTTPS throughout are required, use the REST origin pattern instead.

Signed URLs suit an individual object or clients without cookie support. Signed cookies suit a collection of objects without changing every URL. The application authenticates the user and issues the signature; the signed artifact is a bearer credential until its conditions expire.

S3 presigned URLs authorize direct S3 requests. CloudFront signed URLs authorize requests through the distribution. They do not have identical access paths.

### HTTPS and security

Configure viewer HTTPS separately from origin HTTPS. For an ACM viewer certificate on a custom CloudFront domain, use **us-east-1**. An ALB origin certificate belongs in that ALB's Region. The domain names and certificates must match their respective connections.

AWS WAF filters web requests; Shield addresses DDoS protection. Neither is a substitute for origin access restrictions. A publicly accessible origin can otherwise be called directly.

### Availability and edge processing

An origin group defines a primary and secondary origin. Configured failures can trigger fallback for **GET, HEAD, and eligible cached OPTIONS** requests. POST/PUT requests do not get this origin failover. The secondary must already have suitable content/state; CloudFront does not replicate it.

Origin Shield adds a caching layer to consolidate origin requests. It is unrelated to AWS Shield DDoS protection. Price classes trade geographic edge coverage against delivery cost.

**CloudFront Functions** handles lightweight viewer request/response logic, such as URL normalization and redirects. **Lambda@Edge** supports more involved processing and origin events. Neither requires learning detailed runtime quotas for the main SAA architecture decisions.

## Comparisons

| Requirement | Evaluate |
|---|---|
| Repeated worldwide web downloads | CloudFront |
| Dynamic web application acceleration | CloudFront, even with caching disabled where appropriate |
| Global TCP/UDP flows and stable entry IPs | [[aws_global_accelerator]] |
| Long-distance object transfers to one S3 bucket | S3 Transfer Acceleration |
| DNS answer selection | [[aws_route53]] |

## Exam Traps

- OAC secures origin access; it does not verify a viewer's subscription.
- Signed cookies do not close a publicly readable S3 bucket.
- “Allow all HTTP methods” does not mean “cache all methods.”
- An origin request policy and cache policy solve different problems.
- Origin failover is not general failover for every API write request.
- A CDN can deliver dynamic traffic; “not cacheable” alone does not rule out CloudFront.
- Static global IP requirements are a strong Global Accelerator clue, but read the full service configuration in the question.

## Scenario Check

**Prompt:** Subscribers download many video segments globally. The bucket must not be publicly accessible, and URLs should remain unchanged.

**Answer:** Use CloudFront with an S3 REST origin, OAC and restrictive bucket policy, plus signed cookies for authorized viewers. Configure cache behavior and HTTPS. OAC alone would not restrict viewers to subscribers.

## 30-Second Review

> CloudFront delivers web content through edge locations. Cache keys determine which requests share responses; forwarding a value does not automatically isolate the cache. OAC protects private S3 origins, while signed URLs/cookies restrict viewers. Website endpoints cannot use OAC. Viewer ACM certificates belong in us-east-1. Origin failover covers selected read methods, not general API writes or data replication.

## Sources

- [Cache keys and TTL policies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/controlling-the-cache-key.html)
- [Origin request policies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/controlling-origin-requests.html)
- [Cache behavior and method settings](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DownloadDistValuesCacheBehavior.html)
- [S3 origin access](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [Signed URLs and cookies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-choosing-signed-urls-cookies.html)
- [VPC origins](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-vpc-origins.html)
- [HTTPS and custom domains](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-https-alternate-domain-names.html)
- [Origin failover](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html)
- [Edge function choices](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-choosing.html)

Reviewed: 2026-09-22. Back to [[networking_overview]].
