---
aliases: [AWS Global Accelerator]
tags: [aws, saa-c03, networking]
---
# AWS Global Accelerator

## Mental Model

**Global Accelerator = stable global entry addresses for network flows.** Traffic enters the AWS global network near the client and is directed to regional application endpoints. It does not cache application objects.

## Core

### Standard accelerator

A standard accelerator provides static anycast addresses. IPv4 configurations normally receive two static IPv4 addresses; dual-stack adds two IPv6 addresses. These remain stable while endpoint configurations change, as long as the accelerator resource is retained.

A listener accepts TCP or UDP on configured ports. Regional endpoint groups contain supported application endpoints. Standard endpoints include ALBs, NLBs, EC2 instances, and Elastic IP addresses.

Traffic distribution considers endpoint health, geography, and configured controls. Health-based routing can direct new traffic away from an unavailable Region without waiting for clients to resolve a different DNS address. Existing connections and application sessions can still be interrupted; clients need retry/reconnect behavior.

### Distribution controls

- **Traffic dial:** adjusts the portion of traffic directed to a regional endpoint group relative to what would otherwise go there. It is not a simple percentage of all worldwide requests.
- **Endpoint weights:** distribute eligible traffic among endpoints inside an endpoint group.
- **Client affinity:** can help consistently direct a client's connections. It does not store session state or replicate it between Regions.

Keep regional services, data replication, capacity, and failover readiness aligned with the routing design. Sending traffic to an unprepared Region does not make the application healthy.

### When to choose it

Strong clues are global TCP/UDP applications, stable IP addresses for client firewall allowlists, and rapid health-based changes among regional endpoints. Examples include gaming, voice, and applications using protocols beyond HTTP.

Global Accelerator can also front web applications. CloudFront remains relevant when HTTP delivery features, caching, and private content controls match the requirements.

**Custom routing accelerators** map traffic deterministically to particular EC2 destinations in VPC subnets. They are a separate model; do not apply standard accelerator automatic health-routing assumptions to them. For SAA, prioritize the standard accelerator use case.

## Comparisons

| Capability | Route 53 | CloudFront | Global Accelerator |
|---|---|---|---|
| Main function | DNS resolution | HTTP delivery/cache | Global TCP/UDP routing |
| Caches application objects | No | Yes, when configured | No |
| Health change mechanism | Change DNS answers | Origin behavior/failover | Route flows to endpoints |
| Client DNS caching affects switch | Yes | Same distribution name | Static entry IPs retained |
| Typical exam clue | Weighted/latency/failover DNS | CDN and private cached content | Static anycast IPs; TCP/UDP |

An NLB can supply static addresses per AZ within its Region. Global Accelerator supplies a global entry layer across supported endpoints. The scope of the static-address requirement matters.

## Exam Traps

- Global users do not automatically imply a CDN.
- Global Accelerator is not a private data-center connection or replacement for Direct Connect.
- It does not cache files, perform SQL replication, or restore a failed database.
- S3 buckets are not direct standard accelerator endpoints.
- A traffic dial of zero is not a substitute for understanding failover behavior; do not treat traffic controls as security boundaries.
- Static entry IPs do not mean application connections survive every endpoint failure.
- WAF protection belongs on a supported application resource, such as an ALB; it is not a WAF attachment to the accelerator itself.

## Scenario Check

**Prompt:** A global UDP application operates in two Regions. Customers must allowlist fixed IPs, and new traffic should avoid an unhealthy regional deployment.

**Answer:** Evaluate a standard Global Accelerator with UDP listeners and healthy regional endpoints. CloudFront's web delivery model does not satisfy this UDP requirement. Prepare application state and client reconnection separately.

## 30-Second Review

> Global Accelerator provides stable anycast entry addresses and carries TCP/UDP flows over the AWS network to regional endpoints. It does not cache content. Standard endpoints include ALB, NLB, EC2, and Elastic IPs. Health routing helps new connections avoid failures, but applications still need retries and replicated state. Distinguish global entry addresses from regional NLB addresses.

## Sources

- [How Global Accelerator works](https://docs.aws.amazon.com/global-accelerator/latest/dg/introduction-how-it-works.html)
- [Use cases and static addresses](https://docs.aws.amazon.com/global-accelerator/latest/dg/introduction-benefits-of-migrating.html)
- [Traffic dials](https://docs.aws.amazon.com/global-accelerator/latest/dg/about-endpoint-groups-traffic-dial.html)
- [Endpoint weights](https://docs.aws.amazon.com/global-accelerator/latest/dg/about-endpoints-endpoint-weights.html)

Reviewed: 2026-09-22. Back to [[networking_overview]].
