---
aliases: [Elastic Load Balancing, ELB, ALB, NLB, GWLB]
tags: [aws/saa, compute]
---

# Elastic Load Balancing

## Mental Model

**One entry point distributes traffic to application targets. Choose the balancer by the protocol and routing requirement.**

ELB handles traffic; [[aws_auto_scaling|Auto Scaling]] supplies and replaces EC2 capacity.

## Core

| Type | Layer / traffic | Choose for |
|---|---|---|
| Application Load Balancer (ALB) | Layer 7 HTTP/HTTPS | Host/path routing, web applications, AWS WAF integration |
| Network Load Balancer (NLB) | Layer 4, including TCP/TLS/UDP | Transport-level traffic, static per-AZ IP addresses, high-performance connections |
| Gateway Load Balancer (GWLB) | Layer 3 forwarding with GENEVE encapsulation | Transparently inserting fleets of firewalls or inspection appliances |
| Classic Load Balancer | Previous-generation option | Recognize existing deployments; prefer modern types when choosing new designs |

### ALB building blocks

- **Listener:** protocol and port accepting client connections.
- **Rule:** conditions such as hostname or path and an action, such as forwarding to a target group.
- **Target group:** registered destinations and their health-check configuration; supported ALB targets include instances, IP addresses and Lambda functions.
- **Health check:** validates target readiness independently of EC2 host status.

Example: `/images/*` goes to an image service; `/api/*` goes to an API service. One ALB can route to separate groups with independent scaling.

### Availability and access

An ordinary Regional ALB deployment uses subnets in at least two AZs. Place healthy application capacity across those AZs. An internet-facing ALB can forward to targets with private addresses; the application instances do not need public IPs.

For a common ALB design, permit client traffic to the ALB security group, then allow application and health-check traffic from that security group to the targets. Also verify routing and NACLs.

NLBs **support security groups**. Associate them at creation if needed; an NLB created without security groups cannot have them added later. Feature compatibility can depend on listener configuration.

### Connections and encryption

- An HTTPS ALB listener can terminate TLS using a certificate managed in ACM. HTTPS to the target group adds encryption on the backend connection.
- An NLB TLS listener can terminate TLS; a TCP listener can pass encrypted traffic through for termination at the target.
- An internet-facing NLB can use an Elastic IP for each enabled AZ. ALB addresses can change; use its DNS name rather than pinning observed addresses.
- **Stickiness** keeps a client's requests associated with a target under supported configurations. It is not durable storage for sessions.
- **Deregistration delay** helps drain in-flight work when a target is removed; application shutdown must cooperate.

## Comparisons

| Requirement | Better starting point |
|---|---|
| Route `/orders` and `/catalog` to different services | ALB rules and target groups |
| Static IP allowlisting for TCP service | NLB |
| Inspect traffic using third-party virtual appliances | GWLB |
| Globally cache HTTP content | [[aws_cloudfront|CloudFront]], possibly in front of an ALB |
| Add more EC2 instances when demand increases | ASG, integrated with the load balancer |

**Cross-zone load balancing:** ALB enables it at the load-balancer level; a target group can disable it. NLB and GWLB default to disabled. Check target distribution and relevant transfer costs when changing the setting.

## Exam Traps

- **NLB is not an HTTP path router.** Use ALB for content-based web routing.
- **“NLB has no security groups” is outdated.**
- **Healthy EC2 is not necessarily a healthy application.** Use an appropriate target health check and ASG health integration.
- **ALB can fail open if all registered targets are unhealthy.** Do not treat health checks as an absolute security boundary.
- **WAF is a Layer 7 protection choice.** Do not attach an ALB-style WAF solution directly to an NLB.
- **TLS termination at the balancer does not imply backend encryption.** Check both connections.
- **Sticky sessions do not preserve data when the target fails.** Externalize important session state.

## Scenario Check

**A web application needs hostname routing and protection against malicious HTTP requests.** Choose ALB with AWS WAF. An NLB's static IP capability does not satisfy the Layer 7 routing/protection requirement by itself.

## 30-Second Review

ALB routes HTTP by application content; NLB handles transport traffic and static per-AZ IPs; GWLB inserts network appliances. Listeners receive, rules select, target groups organize, and health checks assess targets. ASG changes EC2 capacity. Plan AZ distribution, security groups, TLS on both connections and graceful draining. Stickiness is not durable state, and NLB security groups are supported.

## Sources

- [How ELB works](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html)
- [Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html)
- [ALB target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
- [ALB health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)
- [Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html)
- [NLB security groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-security-groups.html)
- [Gateway Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
