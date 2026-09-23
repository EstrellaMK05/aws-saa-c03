---
aliases: [Amazon VPC]
tags: [aws, saa-c03, networking]
---
# Amazon VPC

## Mental Model

**VPC = your regional address space and routing boundary.** Subnets place resources in an AZ. Route tables determine the next hop; security groups and NACLs determine whether applicable traffic is allowed.

## Core

### Addressing and routing

- A VPC belongs to one Region and can span AZs. A subnet belongs to one AZ.
- Standard IPv4 VPC/subnet sizes range from `/16` to `/28`; subnet ranges must fit their VPC and not overlap.
- AWS reserves five IPv4 addresses in a standard subnet: `/24` gives 251 usable addresses, `/28` gives 11. Leave room for load balancers and service ENIs.
- Each subnet uses one route table, explicitly associated or inherited from the main table. One table can serve multiple subnets.
- The local route covers internal VPC connectivity, subject to security rules. The most specific matching destination wins: `/24` beats `/16`, which beats `/0`.
- A route to an internet gateway makes a subnet public. For direct IPv4 internet communication, an EC2 instance also needs a public IPv4/EIP and permissive security rules.
- `0.0.0.0/0` covers IPv4; `::/0` covers IPv6. They are separate routes and security-rule families.

### Internet egress

| Resource | Typical route/path | Important condition |
|---|---|---|
| Public IPv4 EC2 | Default route to IGW | Public address required |
| Private IPv4 EC2 | Default route to public NAT, then IGW | NAT allows initiated connections and replies |
| Native IPv6 outbound-only workload | `::/0` to egress-only IGW | Blocks unsolicited internet-initiated connections |
| IPv6-only workload calling IPv4 destination | DNS64 + NAT64 on NAT gateway | Different from native IPv6 egress |

For **zonal public NAT**, place the NAT in a public subnet with an EIP and IGW route. Private subnets route to it. Deploy one per workload AZ for AZ independence and route each subnet to its local NAT.

**Regional public NAT** is a current alternative: it does not need a hosting public subnet and automatic mode expands AZ coverage. Configure workload routes and an attached IGW. Expansion is not instantaneous; manual mode requires managing AZ coverage. Private NAT remains a zonal option.

**Private NAT** translates addresses for private network connectivity through supported paths such as TGW/VGW. It does not provide internet access through an IGW.

NAT gateways are managed; a NAT instance requires EC2 operations, scaling, failover design, and disabling source/destination checking. NAT is not an inbound port-forwarding service.

### Security Groups vs NACLs

| Property | Security group | Network ACL |
|---|---|---|
| Scope | Associated resources/ENIs | Subnet boundary |
| State | Stateful | Stateless |
| Rules | Allow only; combined permissions | Allow/deny; lowest rule number first |
| Replies | Automatically allowed for tracked connections | Explicit return-path rules required |
| Multiple associations | Resource can have multiple SGs | Subnet has one NACL |

Example: allow the application port **from the ALB SG**, then the database port **from the application SG**. Referencing a group identifies allowed sources; it does not copy that group's rules or create routing.

For a client opening HTTPS, destination port is 443; return traffic targets the client's ephemeral port. Stateless NACLs must allow that return path, using the applicable client/OS port range. Do not assume allowing 443 both ways is sufficient.

A new custom SG starts without inbound permissions; a new custom NACL starts denying traffic. Default resources have different defaults. Neither SGs nor NACLs block AmazonProvidedDNS traffic; evaluate Resolver DNS Firewall for domain filtering.

### Diagnose the correct layer

| Tool | Question it answers |
|---|---|
| VPC Flow Logs | What IP traffic metadata was accepted/rejected? |
| Reachability Analyzer | Which supported configuration blocks a path? |
| Traffic Mirroring | What packets should an inspection appliance examine? |
| IPAM | How is address space allocated and used across networks? |

Flow Logs do not contain payloads or every traffic category. Reachability Analyzer analyzes configuration without sending packets; it does not prove that an application is running. Mirroring encrypted packets does not automatically decrypt application data.

## Comparisons

**Route problem:** missing next hop, wrong association, overlap, or missing return route. **Security problem:** SG/NACL denies needed traffic. **DNS problem:** name resolves incorrectly or not at all. **Application problem:** process not listening, failed authentication, or unavailable backend. Check these independently.

For service connectivity see [[aws_vpc_endpoints]]; for network-to-network routing see [[aws_network_connectivity]].

## Exam Traps

- A public subnet does not automatically give an instance a public IP.
- A public subnet does not give VPC-attached Lambda public IPv4 internet access.
- One zonal NAT is redundant inside its AZ, but does not survive that AZ failing for other dependent AZs.
- A NAT gateway has no attached SG. Control clients with SGs and applicable subnet traffic with NACLs.
- An egress-only IGW is for IPv6; it does not solve private IPv4 egress.
- NACL first match wins; a later deny cannot override an earlier matching allow.
- Adding more permissive SGs cannot override a NACL deny.

## Scenario Check

**Prompt:** Private instances in two AZs share a zonal NAT in AZ A. Both lose outbound internet when A fails.

**Answer:** Use a zonal NAT in each AZ with corresponding private routes, or evaluate regional NAT if offered by the scenario. Adding another application instance does not remove the shared network dependency.

## 30-Second Review

> VPC is regional; a subnet occupies one AZ. Public IPv4 access needs an IGW route, public address, and security permission. Private IPv4 egress uses NAT; native IPv6 outbound-only uses an egress-only IGW. Distinguish zonal from regional NAT. SGs are stateful allow lists; NACLs are ordered, stateless allow/deny rules. Diagnose routing, filtering, DNS, and applications separately.

## Sources

- [Subnet addressing](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html)
- [Route priority](https://docs.aws.amazon.com/vpc/latest/userguide/route-tables-priority.html)
- [Zonal NAT characteristics and NAT64](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-basics.html)
- [Public and private NAT connectivity](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Regional NAT](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html)
- [Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

Reviewed: 2026-09-22. Back to [[networking_overview]].
