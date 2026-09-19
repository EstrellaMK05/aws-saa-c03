# VPC / Networking

## Public Subnet

A subnet is public when its route table has a route to an Internet Gateway.

`0.0.0.0/0 → Internet Gateway`

For an EC2 instance to access the Internet using IPv4, it also needs:
- Public IPv4 OR Elastic IP

💡 Public subnet → Route to IGW

---

## Private Subnet

No direct route to an Internet Gateway.

For IPv4 outbound Internet access:

`Private Subnet → NAT Gateway → Internet Gateway`

- NAT Gateway is placed in a PUBLIC subnet
- Private subnet route table:

`0.0.0.0/0 → NAT Gateway`

💡 Private EC2 needs Internet → NAT Gateway

---

## Private IPv6 Outbound Only

Use an:

**Egress-Only Internet Gateway**

Allows outbound IPv6 connections while preventing Internet-initiated inbound connections.

💡 IPv6 outbound-only → Egress-Only IGW

---

# Security Group

- Stateful
- ALLOW rules only
- Attached to ENI/resources
- Has inbound and outbound rules

💡 Return traffic is automatically allowed.

---

# Network ACL

- Stateless
- ALLOW + DENY rules
- Subnet level
- Rules evaluated from lowest number first

💡 First matching rule wins.

⚠️ Return traffic must also be explicitly allowed.

---

# VPC Endpoints

Private access to AWS services without using the Internet/NAT.


## Gateway Endpoint

Supported services:
- S3
- DynamoDB

No endpoint hourly/data processing charges.

💡 Private VPC → S3/DynamoDB → usually Gateway Endpoint


## Interface Endpoint

Uses AWS PrivateLink and ENIs with private IPs.

Supported by many AWS services.

⚠️ S3 ALSO supports Interface Endpoints.

💡 Most AWS services → Interface Endpoint
💡 S3 from normal VPC workload → usually Gateway Endpoint
💡 S3 requiring PrivateLink/private IP connectivity → Interface Endpoint


---

# VPC Peering

Private connection between VPCs.

- CIDRs must NOT overlap
- NOT transitive

⚠️ A ↔ B and B ↔ C does NOT mean A ↔ C.

---

# Transit Gateway

Central hub for connecting many:
- VPCs
- VPNs
- Networks

💡 Many VPCs / hub-and-spoke → Transit Gateway

---

# Site-to-Site VPN

- On-premises ↔ AWS
- Uses public Internet
- Encrypted with IPsec
- Quick to establish

💡 Fast encrypted hybrid connection → Site-to-Site VPN

---

# Direct Connect

Dedicated private network connection between on-premises and AWS.

- More consistent network performance
- Does NOT provide encryption by default

💡 Dedicated hybrid connection → Direct Connect
💡 Dedicated + encryption → Direct Connect + VPN

---

# VPC Flow Logs

Capture network traffic metadata.

Useful for:
- Troubleshooting
- Monitoring
- Security analysis

💡 Need network traffic metadata → VPC Flow Logs

# Route 53 Routing

- Simple → one resource
- Weighted → distribute traffic by percentage
- Latency → lowest latency for user
- Failover → Primary / DR
- Geolocation → based on user's geographic location

💡 90% / 10% → Weighted
💡 Lowest latency → Latency
💡 Primary + DR → Failover