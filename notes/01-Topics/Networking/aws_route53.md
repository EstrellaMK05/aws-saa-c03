---
aliases: [Amazon Route 53, Route 53 Resolver]
tags: [aws, saa-c03, networking]
---
# Amazon Route 53

## Mental Model

**Route 53 chooses a DNS answer.** The client then connects to that destination. Route 53 is not a proxy carrying every application request, so DNS caching affects how quickly changes reach clients.

## Core

### Records and hosted zones

- **A:** IPv4 address. **AAAA:** IPv6 address. **CNAME:** another DNS name.
- A CNAME cannot be used at the zone apex, such as `example.com`.
- A Route 53 **alias** can point the apex to supported targets, such as an ALB or CloudFront distribution. It is not a general alias for every arbitrary hostname.
- Use the load balancer's DNS target rather than hardcoding its currently resolved IP addresses.
- A **public hosted zone** publishes records through public DNS. A **private hosted zone** answers through associated VPCs and configured hybrid resolution paths.
- DNS TTL determines resolver caching lifetime. Lower TTL can help changes propagate sooner, but does not eliminate caching, health-detection time, or established sessions.

### Routing policies

| Policy | Decision rule |
|---|---|
| Simple | Basic answer for a name; may contain multiple values |
| Weighted | Relative share of DNS answers; useful for gradual rollout |
| Latency | AWS Region with the best measured network latency |
| Failover | Primary/secondary based on configured health |
| Geolocation | User's inferred geographic location; configure a default |
| Geoproximity | Resource geography with optional bias to shift coverage |
| Multivalue answer | Up to eight healthy answers; not a full load balancer |
| IP-based | Match configured client CIDR collections |

Weighted 90/10 is not a guarantee that exactly 10% of HTTP requests reach the new deployment. Recursive resolvers and clients cache answers. Latency is not the same criterion as geographic proximity.

### Health and failover

Route 53 can use endpoint checks, calculated health checks, and supported CloudWatch alarm health checks. For supported aliases, **Evaluate Target Health** can use the target's health rather than requiring a separate endpoint check.

Public health checkers cannot directly probe a private-only IP. For private workloads, evaluate supported target health or a metric/alarm-based design that represents application health. DNS failover does not start a database, restore a backup, or replicate state.

Do not assume that unhealthy records can never be returned: all-unhealthy behavior depends on the routing configuration. A health check must actually be associated with the relevant record or target evaluation.

### Hybrid DNS direction

| Question originates in... | Name belongs to... | Needed component |
|---|---|---|
| On-premises | AWS private DNS | Resolver inbound endpoint and on-premises forwarding |
| VPC | On-premises DNS | Resolver outbound endpoint and forwarding rule |

Direction is relative to AWS Resolver. Deploy endpoint IPs across AZs and allow the required DNS traffic, normally UDP and TCP 53. VPN/DX supplies the underlying network path; Resolver endpoints do not create that connection.

Private hosted zones require the appropriate VPC associations and DNS settings (`enableDnsSupport` and `enableDnsHostnames`). Split-view DNS can return different internal and public answers for the same name.

If a matching private zone exists but lacks a requested record, the resolver does not simply fall back to the public zone; NXDOMAIN can result. A matching forwarding rule can also take precedence over a private hosted zone.

## Comparisons

| Service | Main decision |
|---|---|
| Route 53 | Which address/name should DNS return? |
| ALB | Which target handles this HTTP request? |
| CloudFront | Can the edge serve/cache this web response? |
| Global Accelerator | Which healthy regional endpoint receives this network flow? |

DNSSEC authenticates DNS data; it does not encrypt the application's traffic. Resolver DNS Firewall filters domain queries; WAF filters web requests.

## Exam Traps

- An alias at the apex solves a DNS record constraint, not TLS certificate configuration.
- Simple routing does not provide record-associated health checks like failover/weighted designs.
- Multivalue answer routing does not replace an ALB's request routing and connection handling.
- Geolocation selects DNS answers; it is not sufficient by itself to enforce data residency or prevent access from other countries.
- Changing a DNS record does not migrate existing connections immediately.
- Inbound Resolver is for queries entering AWS, not for responses returning from on-premises.

## Scenario Check

**Prompt:** Data center clients can reach an internal AWS address through VPN, but cannot resolve `db.internal.example.com` from an AWS private hosted zone.

**Answer:** Configure a Resolver inbound endpoint and on-premises conditional forwarding, verify the private zone's VPC association, and allow DNS traffic. Another internet gateway does not solve private name resolution.

## 30-Second Review

> Route 53 returns DNS answers; caches delay changes. Alias supports selected AWS targets at the apex. Weighted means proportions of answers; latency means network performance; failover means primary/secondary. Private zones need associations and DNS settings. Hybrid inbound resolves AWS names from on-premises; outbound forwards AWS queries to on-premises. Health-based DNS does not replicate application data.

## Sources

- [Routing policy choices](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [Alias versus CNAME](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html)
- [Health checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)
- [Private hosted zone behavior](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zone-private-considerations.html)
- [Route 53 VPC Resolver](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)

Reviewed: 2026-09-22. Back to [[networking_overview]].
