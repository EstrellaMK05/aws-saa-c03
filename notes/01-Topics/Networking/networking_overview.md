---
aliases: [Networking Decision Guide]
tags: [aws, saa-c03, networking]
---
# Networking Decision Guide

## Mental Model

Networking questions usually ask you to solve one of five different problems: **reach an address, allow a connection, resolve a name, connect networks, or deliver traffic globally**. Identify that problem before choosing a service.

## Core

### Study order

1. [[aws_vpc]] — subnets, routing, internet access, security, troubleshooting.
2. [[aws_vpc_endpoints]] — private service access, policies, and cost.
3. [[aws_network_connectivity]] — peering, Transit Gateway, VPN, Direct Connect.
4. [[aws_route53]] — DNS records, routing policies, hybrid resolution.
5. [[aws_cloudfront]] — caching, private origins, viewer access.
6. [[aws_global_accelerator]] — static entry addresses and TCP/UDP acceleration.

### Three-tier reference design

Internet clients resolve the application name through Route 53. An internet-facing ALB accepts HTTPS in public subnets across AZs. Application instances run in private subnets; their security group accepts application traffic from the ALB security group. The database accepts its database port from the application security group.

Use endpoints for supported AWS service traffic. Provide NAT for required IPv4 internet egress. Databases do not need public addresses just because the frontend is public. Put redundant application and database components in different AZs; multiple subnets in one AZ do not provide AZ failure protection.

Related compute decisions: [[aws_elb]]. Storage decisions: [[aws_s3]].

## Comparisons

| Requirement | First choice to evaluate |
|---|---|
| Reach internet directly from public IPv4 EC2 | Public address + subnet route to IGW + security rules |
| Private IPv4 workload downloads updates | Public NAT gateway |
| Native IPv6 outbound-only internet | Egress-only internet gateway |
| S3/DynamoDB from the same VPC, minimize endpoint cost | Gateway endpoint |
| Supported service through private ENI addresses | Interface endpoint |
| Connect a few entire VPC networks | VPC peering |
| Route among many VPCs and hybrid networks | Transit Gateway |
| Expose one application privately to consumers | PrivateLink endpoint service |
| Quickly connect a data center securely | Site-to-Site VPN |
| Predictable dedicated hybrid connectivity | Direct Connect, with separate encryption/resiliency decisions |
| Choose a DNS answer by health or location | Route 53 |
| Cache web content near viewers | CloudFront |
| Stable global IPs for TCP/UDP applications | Global Accelerator |

### Cost and resilience checkpoints

Compare the whole traffic path: NAT processing, endpoint hours, endpoint processing, cross-AZ traffic, Transit Gateway, and data transfer. A paid interface endpoint per service per AZ is not automatically cheaper for every workload. Gateway endpoints are especially valuable for S3/DynamoDB traffic originating in the VPC.

For traditional **zonal** NAT, use a NAT per active AZ with local routes when AZ independence is required. A single shared zonal NAT creates a dependency. Current **regional** NAT can manage AZ coverage automatically; one regional resource does not imply a single-AZ bill.

## Exam Traps

- Routing permits a path; it does not grant application or IAM authorization.
- A DNS answer does not prove the destination is reachable.
- A private connection does not automatically mean encrypted traffic.
- CloudFront, Route 53, and Global Accelerator can work together; they solve different layers.
- New features do not invalidate a question that explicitly specifies a traditional deployment. Read “zonal,” “regional,” “gateway,” and “interface.”
- Older summaries elsewhere in the vault may omit these distinctions. Use the detailed linked notes for the qualifying conditions.

## Scenario Check

**Prompt:** Private application instances upload large objects to S3 through NAT. Reduce network cost without adding public IPs.

**Answer:** Evaluate an S3 gateway endpoint, associate the application subnet route tables, and verify bucket/IAM/endpoint policies. This removes NAT from that S3 path; unrelated internet requests still require egress connectivity.

## 30-Second Review

> Start with the traffic path. Routes choose destinations; security rules allow connections; IAM authorizes service actions. Endpoints provide private service access. Peering connects pairs; Transit Gateway joins networks at scale. VPN encrypts hybrid traffic; Direct Connect provides dedicated connectivity. Route 53 answers DNS, CloudFront caches web content, and Global Accelerator supplies stable global TCP/UDP entry points.

## Sources

- [VPC route priority](https://docs.aws.amazon.com/vpc/latest/userguide/route-tables-priority.html)
- [VPC pricing](https://aws.amazon.com/vpc/pricing/)
- [Regional NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html)

Reviewed: 2026-09-22.
