---
aliases: [VPC Endpoints, AWS PrivateLink]
tags: [aws, saa-c03, networking]
---
# VPC Endpoints and PrivateLink

## Mental Model

**An endpoint gives a private path to a service.** It does not move the entire service into your VPC, connect every network behind that service, or replace authorization.

## Core

### Gateway endpoints

S3 and DynamoDB support gateway endpoints. Associate the endpoint with the route tables used by the workloads. Service prefix-list routes direct matching service traffic to the endpoint instead of a general NAT route.

No endpoint ENI, attached SG, hourly charge, or endpoint data-processing charge is required. IAM, service resource policies where applicable, and endpoint policies still govern access.

A gateway endpoint serves resources in its VPC. It cannot be consumed through VPN, Direct Connect, peering, or Transit Gateway by remote networks. Create endpoints in the consuming VPCs or choose a suitable interface endpoint design.

### Interface endpoints

PrivateLink interface endpoints provide ENIs with private addresses in selected subnets. Use multiple AZs when availability requires it. Permit client connections to the endpoint service port, often TCP 443, in the endpoint SG.

With private DNS enabled and the required VPC DNS settings, normal service hostnames can resolve to endpoint addresses. Merely creating an ENI does not ensure that applications are using it; check DNS, network reachability, and the requested API endpoint.

Remote networks can reach supported interface endpoints through suitable private connectivity and DNS. On-premises DNS commonly forwards relevant names through a Route 53 Resolver inbound endpoint. An interface endpoint itself is not a hybrid connection.

Interface endpoints incur charges. Both **S3 and DynamoDB now support interface endpoints**, in addition to their gateway options; do not memorize “DynamoDB has no interface endpoint.” Confirm service/Region support for a real deployment.

### PrivateLink endpoint services

A provider can publish a service behind a Network Load Balancer. Consumers create interface endpoints, and the provider controls allowed principals and any required acceptance. Consumers reach the published service rather than obtaining broad routing into the provider VPC.

This service-oriented model can work with overlapping consumer/provider CIDRs because it does not rely on directly routing between the original address spaces. It is not general bidirectional VPC peering.

### Policies and security

Endpoint policies restrict actions/resources through an endpoint where supported. They do not grant permissions missing from IAM or resource policies. Explicit denies still apply.

For an S3 bucket restricted to a particular endpoint, consider an appropriate bucket policy using `aws:SourceVpce`. Check administrative and AWS service access paths before applying a deny: requests through the public console path may no longer qualify.

## Comparisons

| Requirement | Gateway endpoint | Interface endpoint |
|---|---|---|
| Same-VPC S3/DynamoDB access at low endpoint cost | Usually preferred | Possible, paid |
| Private ENI addresses for supported APIs | No | Yes |
| Direct consumption from on-premises over private connectivity | No | Supported designs |
| Security-group control on endpoint | No | Yes |
| Route-table service prefix list | Yes | No gateway-endpoint-style route required |
| Service support | S3 and DynamoDB | Many AWS/partner/custom services |

**Gateway Load Balancer endpoints** are a separate concept: they steer traffic to virtual appliances for inspection. They are not S3/DynamoDB gateway endpoints. PrivateLink also has newer resource/networking capabilities; the interface endpoint and NLB service pattern is the core SAA decision to master first.

## Exam Traps

- An endpoint does not authorize an S3 read by itself.
- Gateway endpoints cannot be shared transitively through a hub VPC.
- Adding an interface endpoint without private DNS can leave applications on the public/NAT path.
- One endpoint does not cover all dependencies. Private container image pulls may need ECR API, ECR registry, S3, and logging access as applicable.
- “PrivateLink” is not a replacement for Transit Gateway when full network routing is required.
- Private connectivity and TLS are separate controls; retain encryption in transit.

## Scenario Check

**Prompt:** A data center must access S3 using private endpoint addresses through an existing VPN. Can it use the VPC's S3 gateway endpoint?

**Answer:** No. Evaluate an S3 interface endpoint, configure DNS resolution from on-premises, and allow the traffic. The VPN supplies connectivity; the endpoint supplies private service access; policies supply authorization.

## 30-Second Review

> Gateway endpoints use route tables for S3 and DynamoDB in the originating VPC, without endpoint charges. Interface endpoints use private ENIs, security groups, DNS, and paid PrivateLink connectivity. Remote networks need their own routed path and DNS design. Endpoint policies restrict access but do not grant missing IAM permissions. Publishing one private service differs from connecting entire networks.

## Sources

- [DynamoDB gateway and interface choices](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-ddb.html)
- [Interface endpoint configuration](https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html)
- [AWS PrivateLink concepts](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html)
- [S3 gateway endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)

Reviewed: 2026-09-22. Back to [[networking_overview]].
