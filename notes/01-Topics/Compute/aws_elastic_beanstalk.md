---
aliases: [AWS Elastic Beanstalk, Elastic Beanstalk]
tags: [aws/saa, compute]
---

# AWS Elastic Beanstalk

## Mental Model

**Provide an application; Beanstalk provisions and operates its environment using AWS resources.**

It reduces deployment and infrastructure work. You still configure the environment, own application behavior and pay for underlying resources.

## Core

### Application, version and environment

- **Application:** logical collection of versions, environments and configuration.
- **Application version:** a particular deployable code bundle.
- **Environment:** resources running a deployed application version; create separate environments for development, test and production.

This note's deployment and worker comparisons focus on **Beanstalk Standard**, the familiar EC2-based model. **Beanstalk Cluster** runs containers using EKS and has a different configuration surface. Do not apply Standard-specific instance profiles or deployment details universally to Cluster environments.

### Standard environment tiers

| Tier / configuration | Purpose | Important detail |
|---|---|---|
| Web server, load-balanced | Serve HTTP requests with load balancing and scalable compute | Configure multiple AZs and adequate capacity for HA |
| Web server, single-instance | Simple low-cost environment | Does not provide redundant web compute |
| Worker | Process background work from SQS | Worker daemon delivers messages to the local application; design for retries and duplicates |

An environment being managed does not prove it is highly available. [[aws_auto_scaling|Auto Scaling]] and [[aws_elb|load balancing]] still need suitable settings.

### Data and permissions

- Keep uploads and important state outside local instance storage, because instances can be replaced.
- For production, consider an **external RDS database with an independent lifecycle**. Coupling a database to an environment complicates termination and blue/green changes; check retention/snapshot settings.
- The **service role** authorizes Beanstalk's service operations. The **EC2 instance profile** supplies application-instance permissions in Standard environments.
- Manage configuration reproducibly through supported configuration files and deployment tools. Avoid relying on manual changes to one replaceable instance.

## Comparisons

### Deployment policies for Standard environments

| Policy | How it deploys | Capacity / rollback trade-off |
|---|---|---|
| All at once | Updates all instances together | Brief outage is expected; fastest/simple option when downtime is acceptable |
| Rolling | Updates batches of existing instances | Reduced serving capacity during each batch; old/new versions coexist |
| Rolling with additional batch | Adds a batch before rotating existing instances | Maintains intended serving capacity with temporary extra resources |
| Immutable | Builds a fresh instance fleet and validates it | Extra capacity; failed deployment can discard new instances without updating the original fleet |
| Traffic splitting | Sends a configured share of traffic to the new fleet for evaluation | Canary-style validation; requires supported load-balancer/environment configuration |
| Blue/green pattern | Deploys a separate environment, tests, then swaps environment CNAMEs | Separate environment and lifecycle; DNS propagation and database compatibility still matter |

“No downtime” assumes sufficient capacity, passing health checks and compatible application/data changes. A deployment policy cannot guarantee this for an incompatible schema migration.

### Nearby services

| Requirement | Starting point |
|---|---|
| Managed deployment of a conventional web application | Beanstalk |
| Define general infrastructure as code | CloudFormation |
| Short event-driven function | [[aws_lambda|Lambda]] |
| Direct container orchestration control | [[aws_containers|ECS or EKS]] |

Beanstalk and CloudFormation can coexist. Beanstalk is application-environment focused; CloudFormation models broader infrastructure. Beanstalk also supports containerized applications, so “containers always means ECS” is too absolute.

## Exam Traps

- **Managed application deployment is not the same as Lambda-style function execution.** Runtime and lifecycle matter.
- **A sudden traffic spike is not enough to pick Lambda over Beanstalk.** Check workload compatibility, warm capacity, scaling delay and concurrency constraints.
- **Rolling can reduce available capacity.** Choose an additional batch when preserving capacity during that rollout is required.
- **Immutable and blue/green are different.** Fresh instances within a deployment are not the same as a separate environment and CNAME swap.
- **Swapping URLs does not migrate database data or undo writes.** Plan shared/external data and backward-compatible changes.
- **Do not terminate the old environment before traffic migration and validation are complete.** DNS/client caching can delay the transition.
- **The EC2 instance profile is not the Beanstalk service role.** Check which actor needs permission.

## Scenario Check

**A production app needs a fully separate environment that can be tested before receiving traffic, with a quick route back to the old environment.** Use a blue/green pattern and controlled CNAME swap. An immutable update builds a new fleet but does not by itself create the same independent environment boundary. Neither approach automatically rolls back database changes.

## 30-Second Review

Beanstalk manages application environments. Standard web tiers serve HTTP; worker tiers process SQS jobs. Rolling deploys in batches; an additional batch preserves capacity; immutable creates a fresh fleet; traffic splitting tests a traffic share; blue/green uses separate environments. Keep persistent data independent, distinguish service and instance roles, and validate capacity and schema compatibility. Managed does not automatically mean highly available.

## Sources

- [Beanstalk concepts](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.html)
- [Deployment policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html)
- [Blue/green deployments](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.CNAMESwap.html)
- [Worker environments](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features-managing-env-tiers.html)
- [RDS and Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.RDS.html)
- [Beanstalk Cluster architecture](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/beanstalk-cluster-concepts.html)

Reviewed: 2026-09-21. Back to [[compute_overview|Compute]].
