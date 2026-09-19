# 🌐 Amazon VPC

> [!summary] Mental Model
> **Amazon VPC = Your private network inside AWS**
>
> You control:
> - IP ranges
> - Subnets
> - Routing
> - Internet connectivity
> - Private connectivity
> - Network security
>
> ```text
> AWS Region
> └── VPC
>     ├── Public Subnet
>     │   └── Internet-facing resources
>     │
>     └── Private Subnet
>         └── Private resources
> ```

---

# 📌 Core

Amazon Virtual Private Cloud (**VPC**) provides a logically isolated virtual network in AWS.

A VPC:

- Belongs to **one Region**
- Spans multiple **Availability Zones**
- Has one or more CIDR blocks
- Contains subnets
- Uses route tables to control traffic

> [!warning] ⚠️ Exam Trap
> **VPC → Regional**
>
> **Subnet → One Availability Zone**

---

# 🔢 CIDR

When creating a VPC, you define an IP address range using CIDR notation.

Example:

```text
VPC
10.0.0.0/16
```

Then divide it into smaller subnets:

```text
VPC: 10.0.0.0/16

├── 10.0.1.0/24
├── 10.0.2.0/24
├── 10.0.3.0/24
└── 10.0.4.0/24
```

VPC IPv4 CIDR blocks generally range from:

`/16 → /28`

CIDR ranges inside the VPC **cannot overlap**.

> [!tip] 🎯 Exam Clue
> When connecting networks:
>
> **Check for overlapping CIDR ranges.**

---

# 🏠 Subnets

A subnet is a range of IP addresses inside a VPC.

Each subnet exists entirely inside **one Availability Zone**.

```text
VPC
│
├── AZ-A
│   ├── Public Subnet
│   └── Private Subnet
│
└── AZ-B
    ├── Public Subnet
    └── Private Subnet
```

Using multiple AZs improves **high availability**.

---

## 🌍 Public Subnet

A subnet is considered public when its route table has a route to an **Internet Gateway**.

```text
0.0.0.0/0
     ↓
Internet Gateway
```

Example:

```text
Internet
   ↕
Internet Gateway
   ↕
Public Subnet
   ↓
EC2
```

For IPv4 internet connectivity, the resource also needs a **public IPv4 / Elastic IP** where applicable.

> [!warning] ⚠️ Exam Trap
> Having a public IP alone does NOT make a subnet public.
>
> The subnet needs a route to an **Internet Gateway**.

---

## 🔒 Private Subnet

A private subnet does **not** have a direct route to an Internet Gateway.

Typical resources:

- Databases
- Internal application servers
- Backend services

If private resources need **outbound IPv4 Internet access**, commonly:

```text
Private EC2
     ↓
Route Table
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

---

# 🚪 Internet Gateway — IGW

An **Internet Gateway** enables communication between a VPC and the Internet.

```text
Internet
   ↕
  IGW
   ↕
 VPC
```

For a public subnet:

```text
Destination      Target

10.0.0.0/16      local
0.0.0.0/0        igw-xxxxx
```

> [!tip] 🎯 Exam Clue
> **Public subnet needs Internet access**
> → Internet Gateway

---

# 🗺️ Route Tables

Route tables determine **where network traffic goes**.

Example:

```text
Destination      Target

10.0.0.0/16      local
0.0.0.0/0        igw-xxxxx
```

The `local` route allows resources inside the VPC to communicate.

Each subnet is associated with a route table.

---

## 🧠 Longest Prefix Match

When multiple routes match a destination, AWS chooses the **most specific route**.

Example:

```text
10.0.0.0/8     → Target A
10.1.0.0/16    → Target B
```

Traffic to:

```text
10.1.5.10
```

uses:

```text
10.1.0.0/16 → Target B
```

because `/16` is more specific than `/8`.

> [!tip] 🎯 Exam Clue
> Routing decision between multiple matching routes
> → **Longest Prefix Match**

---

# 🚪 NAT Gateway

A NAT Gateway allows resources in private subnets to initiate **outbound IPv4 connections** without allowing unsolicited inbound Internet connections.

```text
Private EC2
    ↓
Private Route Table
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

The NAT Gateway is normally placed in a **public subnet**.

Private subnet route:

```text
0.0.0.0/0 → NAT Gateway
```

Public subnet containing the NAT:

```text
0.0.0.0/0 → Internet Gateway
```

> [!tip] 🎯 Exam Clue
> **Private EC2 needs Internet access for updates/downloads**
> → NAT Gateway

> [!warning] ⚠️ Exam Trap
> NAT Gateway does NOT allow the Internet to initiate connections to private instances.

---

# 🆚 NAT Gateway vs Internet Gateway

| | Internet Gateway | NAT Gateway |
|---|---|---|
| Used by | Public resources | Private resources |
| Outbound Internet | ✅ | ✅ |
| Direct inbound Internet | ✅* | ❌ |
| Attached/placed | VPC | Public subnet |
| Route target | IGW | NAT GW |

`*` Subject to public IP, routing and security rules.

Mental model:

```text
Public subnet
→ IGW

Private subnet needing Internet
→ NAT Gateway → IGW
```

---

# 🔐 Security Groups

Security Groups act as virtual firewalls for resources such as EC2 network interfaces.

Characteristics:

- **Stateful**
- Supports **ALLOW rules only**
- Controls inbound and outbound traffic

Example:

```text
Inbound:

HTTPS 443
Source: 0.0.0.0/0
```

Because SGs are stateful:

```text
Request allowed
     ↓
Response automatically allowed
```

You don't need a separate rule for return traffic.

> [!tip] 🧠 Remember
> **Security Group = STATEFUL**

---

# 🚧 Network ACL — NACL

Network ACLs operate at the **subnet level**.

Characteristics:

- **Stateless**
- Supports **ALLOW and DENY**
- Controls inbound and outbound traffic
- Rules are evaluated using rule numbers

Because NACLs are stateless:

```text
Inbound traffic
     ↓
must be allowed

AND

Outbound return traffic
     ↓
must also be allowed
```

---

# ⚔️ Security Group vs NACL

| | Security Group | NACL |
|---|---|---|
| Level | Resource / ENI | Subnet |
| Stateful | ✅ | ❌ |
| Allow | ✅ | ✅ |
| Deny | ❌ | ✅ |
| Return traffic automatic | ✅ | ❌ |

> [!tip] 🎯 Exam Clue
> **Block a specific IP**
> → NACL can explicitly DENY
>
> **Control access to EC2**
> → Security Group

> [!warning] ⚠️ Exam Trap
> **SG → Stateful + Allow only**
>
> **NACL → Stateless + Allow/Deny**

---

# 🔌 VPC Endpoints

VPC Endpoints allow resources inside a VPC to privately access supported services without requiring:

- Internet Gateway
- NAT Gateway
- Public IP

```text
Private EC2
     ↓
VPC Endpoint
     ↓
AWS Service
```

Traffic remains on the AWS network.

There are two especially important endpoint types for SAA:

1. **Gateway Endpoint**
2. **Interface Endpoint**

---

# 🚪 Gateway Endpoint

Gateway Endpoints support:

- **Amazon S3**
- **Amazon DynamoDB**

They are configured through **route tables**.

```text
Private EC2
     ↓
Route Table
     ↓
Gateway Endpoint
     ↓
S3 / DynamoDB
```

No NAT Gateway is required.

> [!tip] 🎯 Exam Clue
> **Private EC2 → S3/DynamoDB + lowest cost**
> → Gateway Endpoint

---

# 🔗 Interface Endpoint

Interface Endpoints use **AWS PrivateLink**.

They create **Elastic Network Interfaces (ENIs)** with private IP addresses inside your subnets.

```text
Private EC2
     ↓
Private IP
     ↓
Interface Endpoint ENI
     ↓
AWS Service
```

Interface Endpoints:

- Use PrivateLink
- Use ENIs
- Can have Security Groups
- Support many AWS services
- Have hourly/data processing costs

> [!tip] 🎯 Exam Clue
> **Privately access an AWS service other than S3/DynamoDB**
> → Usually Interface Endpoint

---

# 🆚 Gateway vs Interface Endpoint

| | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| Technology | Route table | PrivateLink |
| S3 | ✅ | Supported separately |
| DynamoDB | ✅ | — |
| ENI | ❌ | ✅ |
| Security Group | ❌ | ✅ |
| Cost | No endpoint hourly charge | Paid |
| Services | S3 / DynamoDB | Many AWS services |

> [!tip] 🧠 SAA Shortcut
> **S3 / DynamoDB**
> → Gateway Endpoint
>
> **Most other AWS services**
> → Interface Endpoint / PrivateLink

---

# 🔗 VPC Peering

VPC Peering privately connects **two VPCs**.

```text
VPC A
  ↕
Peering
  ↕
VPC B
```

Supports:

- Same account
- Different accounts
- Same Region
- Different Regions

Traffic uses **private IP addresses**.

CIDR blocks **cannot overlap**.

---

## ⚠️ Peering Is NOT Transitive

Suppose:

```text
VPC A ↔ VPC B ↔ VPC C
```

A cannot automatically communicate with C.

You would need:

```text
VPC A ↔ VPC C
```

as another peering connection.

> [!warning] ⚠️ Exam Trap
> **VPC Peering is NOT transitive.**

---

# 🚉 AWS Transit Gateway

Transit Gateway acts as a **central networking hub**.

Instead of:

```text
A ↔ B
A ↔ C
A ↔ D
B ↔ C
B ↔ D
...
```

Use:

```text
       VPC A
         │
VPC B ─ TGW ─ VPC C
         │
       VPC D
         │
    On-Premises
```

Useful for:

- Many VPCs
- Multiple AWS accounts
- Hybrid connectivity
- Centralized networking

> [!tip] 🎯 Exam Clue
> **Connect many VPCs with simplified management**
> → Transit Gateway

---

# ⚔️ VPC Peering vs Transit Gateway

| | VPC Peering | Transit Gateway |
|---|---|---|
| Architecture | Point-to-point | Hub-and-spoke |
| Few VPCs | ✅ | Possible |
| Many VPCs | Complex | ✅ |
| Transitive routing | ❌ | ✅ |
| Centralized management | ❌ | ✅ |

Mental model:

```text
2 VPCs
→ Peering

Many VPCs
→ Transit Gateway
```

---

# 🏢 Site-to-Site VPN

Connects an on-premises network to AWS using encrypted **IPsec tunnels over the Internet**.

Components:

```text
On-Premises
     │
Customer Gateway
     │
 Internet
     │
IPsec VPN
     │
Virtual Private Gateway
     │
    VPC
```

### Customer Gateway — CGW

Represents the **customer/on-premises side**.

### Virtual Private Gateway — VGW

Represents the **AWS/VPC side**.

A Site-to-Site VPN provides **two tunnels** for redundancy.

> [!tip] 🎯 Exam Clue
> **Quick encrypted connection from on-premises to AWS**
> → Site-to-Site VPN

---

# 🔌 AWS Direct Connect

Direct Connect provides a **dedicated network connection** between on-premises infrastructure and AWS.

```text
Data Center
     │
Dedicated Connection
     │
Direct Connect
     │
    AWS
```

Advantages:

- More consistent network performance
- Private dedicated connectivity
- Useful for large or predictable traffic

Unlike Site-to-Site VPN, Direct Connect does not simply use an IPsec VPN over the public Internet.

> [!tip] 🎯 Exam Clue
> **Dedicated connection + consistent network performance**
> → Direct Connect

---

# ⚔️ VPN vs Direct Connect

| | Site-to-Site VPN | Direct Connect |
|---|---|---|
| Connection | Internet | Dedicated |
| Encryption | IPsec | Not inherently IPsec |
| Setup | Faster | Longer |
| Performance consistency | Lower | Higher |
| Typical cost | Lower | Higher |

> [!tip] 🎯 Exam Clue
> **Need connection quickly**
> → VPN
>
> **Need dedicated/predictable connectivity**
> → Direct Connect

They can also be combined:

```text
Direct Connect
     +
VPN
```

for private connectivity with encryption requirements.

---

# 🌐 IPv6

VPCs can support:

- IPv4
- IPv6
- Dual-stack

IPv6 addresses are globally routable.

For outbound-only IPv6 Internet access, use an:

## Egress-Only Internet Gateway

```text
Private IPv6 Resource
        ↓
Egress-Only IGW
        ↓
Internet
```

Allows outbound IPv6 connections while preventing Internet-initiated inbound connections.

> [!tip] 🎯 Exam Clue
> **IPv4 private outbound Internet**
> → NAT Gateway
>
> **IPv6 outbound-only Internet**
> → Egress-Only Internet Gateway

---

# 📋 VPC Flow Logs

VPC Flow Logs capture metadata about IP traffic.

Can help identify:

- Accepted traffic
- Rejected traffic
- Source/destination IP
- Source/destination ports
- Troubleshooting connectivity

Flow Logs can be created for:

- VPC
- Subnet
- Network Interface

Destinations include services such as CloudWatch Logs and S3.

> [!warning] ⚠️ Exam Trap
> Flow Logs contain **network traffic metadata**.
>
> They are NOT packet payload/content capture.

---

# 🪞 VPC Traffic Mirroring

Traffic Mirroring copies network traffic from supported network interfaces to monitoring/security appliances.

Useful for:

- Deep packet inspection
- Threat monitoring
- Troubleshooting
- Security appliances

Mental model:

```text
Flow Logs
→ Traffic metadata

Traffic Mirroring
→ Copy actual network traffic for inspection
```

---

# 🔍 Reachability Analyzer

Reachability Analyzer helps determine whether a network path between two resources is reachable.

It performs **configuration analysis**.

It does NOT need to send real packets.

Useful for finding issues involving:

- Route tables
- Security Groups
- NACLs
- Gateways
- Network paths

> [!tip] 🎯 Exam Clue
> **Why can't resource A reach resource B?**
> → Reachability Analyzer

---

# 📍 VPC IPAM

**IP Address Manager (IPAM)** helps centrally manage IP address space.

Useful for:

- Planning CIDR ranges
- Tracking IP usage
- Detecting overlapping ranges
- Managing IPs across accounts/VPCs

> [!tip] 🎯 Exam Clue
> **Large organization needs centralized IP address management**
> → VPC IPAM

---

# ⚠️ Quick Exam Traps

| If you see... | Think... |
|---|---|
| Public subnet Internet | **Internet Gateway** |
| Private subnet outbound Internet | **NAT Gateway** |
| IPv6 outbound-only Internet | **Egress-Only IGW** |
| Private S3/DynamoDB access | **Gateway Endpoint** |
| Private AWS service access | **Interface Endpoint / PrivateLink** |
| Stateful firewall | **Security Group** |
| Stateless firewall | **NACL** |
| Explicit network DENY | **NACL** |
| Two VPCs privately connected | **VPC Peering** |
| Many VPCs / central hub | **Transit Gateway** |
| Peering transitive routing | ❌ **Not supported** |
| Overlapping VPC CIDRs + Peering | ❌ **Not supported** |
| On-prem → AWS encrypted over Internet | **Site-to-Site VPN** |
| Dedicated on-prem connection | **Direct Connect** |
| Network traffic metadata | **VPC Flow Logs** |
| Copy traffic for deep inspection | **Traffic Mirroring** |
| Troubleshoot network path | **Reachability Analyzer** |
| Centralized IP management | **VPC IPAM** |

---

> [!abstract] 🧠 Amazon VPC in 30 Seconds
> **VPC:** Regional virtual network
>
> **Subnet:** One AZ
>
> **Public Subnet:** Route → IGW
>
> **Private Internet:** NAT Gateway → IGW
>
> **IPv6 outbound:** Egress-Only IGW
>
> **Route selection:** Longest Prefix Match
>
> **SG:** Stateful + Allow only
>
> **NACL:** Stateless + Allow/Deny
>
> **S3/DynamoDB private access:** Gateway Endpoint
>
> **Other AWS services privately:** Interface Endpoint
>
> **2 VPCs:** Peering
>
> **Many VPCs:** Transit Gateway
>
> **VPN:** Encrypted over Internet
>
> **Direct Connect:** Dedicated connection
>
> **Flow Logs:** Traffic metadata
>
> **Reachability Analyzer:** Troubleshoot connectivity