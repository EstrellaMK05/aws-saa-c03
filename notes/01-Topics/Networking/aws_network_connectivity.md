---
aliases: [Hybrid Networking, VPC Peering, Transit Gateway, Direct Connect, Site-to-Site VPN]
tags: [aws, saa-c03, networking]
---
# Network and Hybrid Connectivity

## Mental Model

**Decide whether you are connecting whole networks or exposing one service.** Then decide whether you need encryption, predictable performance, transitive routing, and independent failure paths.

## Core

### VPC peering

Peering connects two VPCs, including across accounts or Regions. Configure routes on both sides and appropriate security rules. Address ranges must not overlap.

Peering is not transitive: A–B and B–C do not give A–C connectivity. A peer also cannot borrow another VPC's IGW, NAT, VPN, Direct Connect connection, or gateway endpoint as a transit path. DNS resolution options are separate from basic IP routing.

### Transit Gateway

A regional Transit Gateway (TGW) connects VPC and hybrid attachments through a central routing hub. It can be shared across accounts with AWS RAM and peered with other TGWs.

VPC subnet route tables must send remote destinations to TGW. A TGW attachment associates with a TGW route table; propagation adds learned routes to selected TGW tables. Association and propagation serve different purposes. Use separate route tables for segmentation, such as isolating production from development.

TGW makes transitive designs possible, but attachments alone do not guarantee communication. Verify routes in both directions and avoid overlapping CIDRs. Centralized inspection additionally needs correctly routed, symmetric traffic paths; a hub does not inspect packets by default.

### Site-to-Site VPN

VPN normally establishes encrypted IPsec tunnels over the internet between the customer gateway device and an AWS VGW or TGW. The AWS customer gateway resource describes the customer side; it is not the physical appliance.

Each connection includes two tunnels. Configure both. A single on-premises device or ISP can still be a shared failure point. Use additional independent customer devices/connections when required. Dynamic routing uses BGP; configure routing and tunnel monitoring rather than assuming failover will work automatically.

**Client VPN** serves individual remote users. **Site-to-Site VPN** connects networks. They are not interchangeable deployment models.

### Direct Connect

Direct Connect provides dedicated connectivity to AWS, with more consistent performance than an internet-only path. Provisioning takes planning; it is not the usual “connection needed today” answer.

| Virtual interface | Primary use |
|---|---|
| Private VIF | VPC private addressing through VGW/Direct Connect gateway designs |
| Public VIF | AWS public service addresses over Direct Connect |
| Transit VIF | TGWs through a Direct Connect gateway |

A Direct Connect gateway helps connect the supported VPC/TGW architecture across Regions. It does not turn VPCs into a freely communicating peering mesh.

Dedicated does not mean encrypted. Use application TLS or a suitable IPsec VPN over Direct Connect when required. MACsec offers link encryption on supported connections; it is not a universal end-to-end replacement for TLS/IPsec.

Build resilient connectivity with redundant devices and connections in distinct Direct Connect locations. Multiple links in a LAG at one location do not protect against losing that entire location. An internet VPN can be a lower-cost backup if its capacity and recovery behavior satisfy requirements; test route convergence and failover.

## Comparisons

| Need | Evaluate | Main limitation |
|---|---|---|
| Few VPCs, straightforward network access | Peering | No transit; mesh management grows |
| Many networks with central route control | TGW | Additional cost and routing design |
| Only a provider application | PrivateLink | Service access, not broad network transit |
| Rapid encrypted hybrid deployment | VPN | Internet variability in standard design |
| Predictable dedicated hybrid path | Direct Connect | Lead time; encryption and HA are separate |
| Employee laptop access | Client VPN | Requires user authentication and authorization |

See [[aws_vpc_endpoints]] for PrivateLink and [[aws_route53]] for hybrid DNS.

## Exam Traps

- TGW does not automatically solve overlapping network addresses.
- Two VPN tunnels are not two independent customer sites or devices.
- A public VIF means access to AWS public addresses, not general internet transit.
- A DX gateway is not the same resource as a TGW or VGW.
- Redundant circuits in one location do not satisfy location-level resilience.
- VPN over Direct Connect is possible; “VPN always uses the internet” is too broad.
- Private DNS resolution still needs configuration after network connectivity works.

## Scenario Check

**Prompt:** Forty VPCs need controlled access to a data center and selected shared services. Maintaining a peering mesh is becoming difficult.

**Answer:** Evaluate TGW, attach the VPCs and hybrid path, and segment routes by environment. Use Direct Connect if predictable dedicated performance is required; add encryption and redundant paths according to the stated requirements.

## 30-Second Review

> Peering connects pairs without transit. Transit Gateway centralizes routed network connectivity and segmentation. PrivateLink exposes services. Site-to-Site VPN supplies encrypted hybrid tunnels; Client VPN serves remote users. Direct Connect supplies dedicated connectivity, with encryption and redundancy designed separately. Check CIDR overlap, both routing directions, DNS, and independent failure paths before declaring a hybrid architecture highly available.

## Sources

- [VPC peering limitations](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html)
- [Transit Gateway routing](https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html)
- [Site-to-Site VPN](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
- [Direct Connect VIF types](https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html)
- [Direct Connect resilience](https://docs.aws.amazon.com/directconnect/latest/UserGuide/max-resiliency-set-up.html)
- [MACsec](https://docs.aws.amazon.com/directconnect/latest/UserGuide/MACsec.html)

Reviewed: 2026-09-22. Back to [[networking_overview]].
