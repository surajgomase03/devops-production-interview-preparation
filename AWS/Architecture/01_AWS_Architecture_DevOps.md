# AWS Architecture & DevOps Interview Answers

---

# Q1. Design a globally accessible e-commerce application on AWS with scalability, high availability, reliability and fault tolerance.

## Interview-ready answer

“I would design this as a **multi-tier, multi-AZ architecture**, with a second AWS Region for disaster recovery or active-active service if the business requires global availability and low latency.

1. **Global entry and content delivery:** Use **Route 53** for DNS and health-based or latency-based routing. Use **CloudFront** to cache static content such as product images, CSS and JavaScript close to users.
2. **Edge security:** Place **AWS WAF** in front of the application to filter malicious HTTP requests and use **AWS Shield** for DDoS protection. Use **ACM** to manage TLS certificates.
3. **Network design:** Create a **VPC across at least three Availability Zones**. Put internet-facing load balancers in public subnets and application services and databases in private subnets. Use security groups with least-privilege rules.
4. **Traffic distribution:** Use an **Application Load Balancer (ALB)** to route HTTP/HTTPS requests to healthy application targets across AZs. For APIs, I can also use **API Gateway** for authentication, throttling, request validation and API lifecycle management.
5. **Application tier:** Run stateless microservices on **Amazon EKS** or **ECS Fargate** across multiple AZs. Use Kubernetes HPA/Cluster Autoscaler or ECS Service Auto Scaling to scale pods/tasks based on demand. I would choose EKS if the organization already needs Kubernetes; otherwise ECS Fargate can reduce cluster-management overhead.
6. **Data tier:** Use **Aurora/RDS Multi-AZ** for relational transactions such as orders and payments. Use **DynamoDB** for suitable high-scale key-value access patterns such as carts or sessions. Use **ElastiCache for Redis** for frequently accessed data.
7. **Asynchronous processing:** Use **SQS**, **SNS** and **EventBridge** to decouple services. For example, after an order is accepted, publish an event and process email, inventory updates and shipping workflows asynchronously. Configure retries and a dead-letter queue.
8. **Object storage:** Store product images, invoices and documents in **S3**. Enable versioning, lifecycle policies and cross-Region replication where required.
9. **Monitoring and security operations:** Use **CloudWatch** for metrics, logs, alarms and dashboards; **X-Ray/OpenTelemetry** for distributed tracing; and **CloudTrail** for AWS API auditing. Store secrets in **Secrets Manager** and encrypt data with **KMS**.
10. **Disaster recovery:** Deploy a recovery stack in a second Region. Use **Aurora Global Database**, **DynamoDB Global Tables** or S3 replication depending on the data type. Use **Route 53** or **Global Accelerator**, with **Route 53 Application Recovery Controller (ARC)** where appropriate, to shift traffic during a regional outage.

I would define the business **RTO and RPO** first, then select active-active or warm-standby recovery based on consistency, recovery targets and cost. I would also test failover and failback regularly.”

## Architecture flow

```text
                         GLOBAL CUSTOMERS
                                |
                    Route 53 / Global Accelerator
                                |
                  CloudFront + AWS WAF + Shield
                                |
                       Regional entry point
                         ALB / API Gateway
                                |
             +------------------+------------------+
             |                  |                  |
          AZ - 1             AZ - 2             AZ - 3
       Private subnet      Private subnet      Private subnet
             |                  |                  |
       EKS / ECS tasks     EKS / ECS tasks     EKS / ECS tasks
             +------------------+------------------+
                                |
             +------------------+------------------+
             |                  |                  |
         Aurora/RDS          DynamoDB          ElastiCache
          Multi-AZ        (key-value data)        Redis
             |
       SQS / SNS / EventBridge
             |
       Async workers / Lambda
             |
        S3 object storage

        CloudWatch + X-Ray/OpenTelemetry + CloudTrail

       SECOND AWS REGION
       - Duplicated VPC and application stack
       - Replicated data / backups
       - Route 53 or Global Accelerator failover
       - ARC / runbooks for controlled recovery
```

## Service selection and purpose

| Layer / requirement | AWS services | Purpose |
|---|---|---|
| Global DNS and routing | Route 53, Global Accelerator | DNS routing, health-based failover, low-latency global entry |
| CDN | CloudFront | Cache static content at edge locations |
| Web and DDoS protection | AWS WAF, AWS Shield | Filter web attacks and mitigate DDoS |
| TLS | AWS Certificate Manager (ACM) | Manage HTTPS certificates |
| Network | VPC, subnets, route tables, NAT Gateway, security groups | Isolate public, application and data tiers |
| Load balancing | ALB | Distribute HTTP/HTTPS traffic to healthy targets |
| API management | API Gateway (when needed) | API authentication, throttling, validation and routing |
| Compute | EKS or ECS Fargate | Run scalable application microservices |
| Compute scaling | HPA, Cluster Autoscaler/Karpenter, ECS Service Auto Scaling | Scale workloads and compute capacity |
| Relational database | Aurora/RDS Multi-AZ | Transactional and relational data |
| NoSQL database | DynamoDB | High-scale key-value/document access patterns |
| Global data recovery | Aurora Global Database, DynamoDB Global Tables | Replicate supported data across Regions |
| Cache | ElastiCache for Redis | Reduce database reads and application latency |
| Messaging | SQS, SNS, EventBridge | Decouple services and process events asynchronously |
| Serverless workers | Lambda | Run event-driven background tasks |
| Object storage | S3 | Product images, invoices, documents and backups |
| Secrets and encryption | Secrets Manager, KMS | Secure credentials and encryption keys |
| Observability | CloudWatch, X-Ray/OpenTelemetry, CloudTrail | Metrics, logs, traces and audit events |
| Disaster recovery | Route 53 ARC, Systems Manager Automation | Coordinate and automate regional recovery |

## How the four requirements are achieved

### 1. Scalability
- **Horizontal scaling:** Add more pods, ECS tasks or EC2 instances instead of depending only on larger servers.
- **Auto Scaling:** Scale application replicas based on CPU, memory, request count or custom business metrics.
- **Kubernetes scaling:** HPA scales pods; Cluster Autoscaler or Karpenter adds/removes worker nodes.
- **Caching and CDN:** CloudFront and Redis reduce repeated requests reaching the application and database.
- **Queue-based scaling:** SQS absorbs sudden traffic spikes; workers scale based on queue depth.
- **Database scaling:** Use read replicas/read endpoints for read-heavy workloads and DynamoDB capacity scaling where appropriate.

### 2. High availability
- Deploy application instances across **multiple Availability Zones**.
- Use ALB health checks to stop sending traffic to unhealthy targets.
- Keep multiple application replicas and use readiness/liveness probes where running on Kubernetes.
- Use Multi-AZ database deployment and managed failover.
- Avoid single points of failure in NAT, connectivity, secrets, queues and other dependencies.
- Use a second Region when the business requires protection from a full regional outage.

### 3. Reliability
- Set connection and request **timeouts**; use bounded retries with exponential backoff and jitter.
- Use **circuit breakers** to avoid repeatedly calling an unhealthy downstream service.
- Make order and payment APIs **idempotent** so retries do not create duplicate orders or charges.
- Use SQS dead-letter queues and replay procedures for messages that repeatedly fail.
- Apply graceful degradation: for example, product reviews may be temporarily unavailable without breaking product browsing or checkout.
- Use rolling, blue/green or canary deployments with health checks and rollback.

### 4. Fault tolerance
- Spread compute and data services across independent AZs.
- Replace failed pods/instances automatically and route traffic only to healthy targets.
- Replicate critical data and maintain tested backups.
- Use multi-Region recovery for regional failures.
- Run game days and disaster-recovery tests to prove that failover works.

## Production example: order and payment flow

1. A customer opens the website. Route 53 resolves the domain and CloudFront serves cached static content.
2. Dynamic checkout requests pass through WAF and reach the ALB/API Gateway.
3. The ALB routes the request to healthy checkout service replicas running in private subnets.
4. The checkout service validates the cart and inventory, then creates an order using a transactional data store.
5. The payment service calls the payment provider using a secure secret and an idempotency key.
6. After successful order creation, the application publishes an event to SQS, SNS or EventBridge.
7. Background workers process inventory updates, email/SMS notifications and shipping workflows.
8. CloudWatch metrics, logs and distributed traces help the team monitor the complete transaction.

**Important:** Payment processing needs strong consistency and careful duplicate-request handling. I would not assume that every database or cache is suitable for payment records. The payment provider and application must also meet applicable payment-data security requirements.

## Multi-Region disaster recovery choices

| Strategy | How it works | Relative cost | Recovery |
|---|---|---|---|
| Backup and restore | Restore infrastructure and data after a disaster | Lowest | Slowest |
| Pilot light | Keep core data and minimum services ready; scale up during recovery | Low–medium | Faster than backup/restore |
| Warm standby | Run a smaller but functional application stack in the DR Region | Medium–high | Faster recovery |
| Active-active | Both Regions serve traffic; requires careful data and conflict handling | Highest | Potentially fastest, but most complex |

- **RTO:** Maximum acceptable time to restore the service.
- **RPO:** Maximum acceptable data-loss window.
- A multi-AZ deployment protects against many AZ-level failures, but **does not by itself protect against a complete Region outage**.
- Cross-Region replication is service-specific. Replication lag, write ownership, conflict resolution and data residency must be considered.
- DNS changes are not necessarily instantaneous because of DNS caching and client behavior.

## Security best practices

- Keep application and database resources in private subnets; expose only required entry points.
- Use IAM roles and workload identity instead of long-lived access keys.
- Enforce least privilege for users, CI/CD pipelines and application workloads.
- Use TLS in transit and KMS encryption at rest.
- Store database passwords and API credentials in Secrets Manager; rotate them.
- Apply WAF managed rules, rate limits and Shield protections.
- Restrict security-group rules to required sources and ports.
- Scan container images, dependencies and infrastructure-as-code before deployment.
- Enable CloudTrail and appropriate VPC, application and database audit logs.
- Protect backups and test restoration regularly.

## Production troubleshooting scenarios

**Scenario 1: Sudden traffic spike**
- Check ALB request count, target response time, 4xx/5xx and application metrics.
- Confirm HPA/ECS scaling and cluster capacity.
- Check queue depth and database connections.
- Scale workers and application capacity; use caching or rate limiting where appropriate.

**Scenario 2: One Availability Zone fails**
- ALB health checks stop routing to unhealthy targets.
- Remaining AZs continue serving traffic.
- Auto Scaling replaces failed capacity, subject to available regional capacity.
- Confirm the database has completed failover and monitor remaining AZ capacity.

**Scenario 3: Database becomes slow**
- Check CPU, connections, locks, slow queries, storage and read/write latency.
- Optimize queries and indexes; use caching and read replicas for suitable read traffic.
- Apply connection pooling and protect the database from retry storms.
- Do not scale application replicas endlessly if the database is already saturated.

**Scenario 4: Entire Region becomes unavailable**
- Declare the regional incident using the agreed runbook.
- Validate DR Region health, data replication state and database promotion.
- Use ARC/automation and Route 53 or Global Accelerator to shift traffic.
- Validate critical user journeys, then communicate status and monitor recovery.
- Follow a controlled failback plan after the primary Region is healthy.

## Possible follow-up questions

1. **Why use CloudFront if Route 53 is already available?**  
   Route 53 provides DNS resolution and routing; CloudFront is a CDN that caches and delivers content from edge locations.

2. **ALB vs API Gateway?**  
   ALB distributes application traffic to targets. API Gateway adds API-focused capabilities such as authorization, throttling, validation and API management. They can be used together when needed.

3. **EKS vs ECS Fargate?**  
   EKS is suitable when Kubernetes capabilities and ecosystem compatibility are required. ECS Fargate offers a simpler managed container runtime with less infrastructure management.

4. **Multi-AZ vs Multi-Region?**  
   Multi-AZ protects against an AZ failure within a Region. Multi-Region supports regional disaster recovery and can improve global latency, but adds data and operational complexity.

5. **How do you prevent duplicate payments?**  
   Use idempotency keys, transaction status checks and safe retry logic. Do not blindly retry a payment request after an unknown outcome.

6. **Why use SQS?**  
   It buffers work, decouples services and allows consumers to process messages independently. Configure visibility timeout, retries and a DLQ.

7. **How do you scale EKS?**  
   HPA scales pod replicas; Cluster Autoscaler or Karpenter adjusts worker-node capacity. Set resource requests/limits and monitor pending pods.

8. **How do you decide the DR strategy?**  
   Start with business RTO/RPO, data consistency, regulatory requirements, dependencies and cost; then test that the chosen design meets those targets.

## What the interviewer expects

- Explain the complete request flow from a global customer to the application and data layers.
- Map scalability, availability, reliability and fault tolerance to specific design choices.
- Understand the difference between AZ-level high availability and Region-level disaster recovery.
- Justify EKS/ECS, ALB/API Gateway, Aurora/DynamoDB and caching choices.
- Explain asynchronous processing, idempotency, retries, circuit breakers and graceful degradation.
- Discuss security, observability, RTO/RPO and operational testing.

## What to avoid

- Do not say that a load balancer alone makes an application highly available.
- Do not claim that Multi-AZ protects against a full Region outage.
- Do not say CloudFront should cache sensitive or dynamic checkout/payment responses indiscriminately.
- Do not treat an Aurora standby as an ordinary read replica in every configuration.
- Do not assume cross-Region replication is instantaneous or conflict-free.
- Do not promise zero downtime or zero data loss without a proven design and tested RTO/RPO.
- Do not add every AWS service without explaining why it is needed.

## Quick revision keywords

`Route 53` · `Global Accelerator` · `CloudFront` · `WAF` · `Shield` · `ACM` · `VPC` · `Multi-AZ` · `Public/Private Subnets` · `ALB` · `API Gateway` · `EKS/ECS` · `HPA` · `Cluster Autoscaler/Karpenter` · `Aurora Multi-AZ` · `DynamoDB Global Tables` · `ElastiCache` · `SQS` · `SNS` · `EventBridge` · `S3 CRR` · `CloudWatch` · `X-Ray` · `CloudTrail` · `Secrets Manager` · `KMS` · `Idempotency` · `Circuit Breaker` · `RTO` · `RPO` · `Route 53 ARC`

---

# Q2. Monolithic Architecture vs Microservices Architecture. When would you choose each?

## Interview-ready answer

“A monolithic application is built and deployed as one main unit. Its modules may be logically separated, but they share one deployment lifecycle and often one runtime.

Microservices split the application into smaller, independently deployable services around business capabilities. Each service owns a clear responsibility and communicates through APIs or messaging.

- **Monolith advantages:** simpler development and deployment at the beginning, easier local testing, and fewer distributed-system concerns.
- **Monolith limitations:** the application can become tightly coupled; scaling or releasing one component may require releasing the whole application.
- **Microservices advantages:** independent deployment and scaling, team ownership, and better isolation of some failures.
- **Microservices limitations:** network latency, service discovery, distributed tracing, data consistency, security and operational complexity.

I would choose based on domain boundaries, scale, release independence and team maturity—not simply because microservices are popular. A modular monolith is often a good starting point.”

## Comparison

| Area | Monolith | Microservices |
|---|---|---|
| Deployment | Usually one application unit | Independent service deployments |
| Scaling | Often scale the whole application | Scale individual services |
| Communication | In-process calls are common | APIs or asynchronous messaging |
| Data | Often shared database/schema | Prefer service-owned data |
| Failure | A process failure can affect the whole app | Failures can be isolated, if designed correctly |
| Complexity | Simpler distributed operations | More networking and operational complexity |
| Testing | Easier end-to-end local setup initially | Contract, integration and distributed testing needed |

## Production example

In an e-commerce platform, catalogue, cart, order, payment and notification can be separate services when they have distinct business ownership, scaling and release needs. For a small application with a small team and limited traffic, a modular monolith may be cheaper and easier to operate.

## What to avoid

- Do not say microservices are always better.
- Do not say microservices automatically make an application highly available.
- Do not assume every microservice must have a separate physical database server.
- Do not choose architecture based only on team size or technology trend.

## Quick revision keywords

`Tightly Coupled` · `Modular Monolith` · `Independent Deployment` · `Bounded Context` · `Service Ownership` · `API Gateway` · `Event-Driven` · `Service Discovery` · `Distributed Tracing` · `Circuit Breaker` · `Saga` · `Data Consistency`

---

# Q3. For a team of 10–12 developers and DevOps engineers, would you choose monolith or microservices?

## Interview-ready answer

“I would not decide based only on the number of people. I would first understand the business domains, application complexity, scaling requirements, deployment frequency and the team's operational maturity.

If the product is relatively small, the domains are tightly related, or the team has limited platform and observability maturity, I would start with a **modular monolith**. It gives simpler testing, deployment and operations.

If the application has clear bounded contexts, different modules need independent scaling or releases, and teams can own services end to end, I would consider microservices.

For a team of 10–12, I would also define service ownership, API contracts, CI/CD standards, security controls, monitoring and on-call responsibility. The decision should reduce business and operational risk, not add unnecessary complexity.”

## Decision checklist

- Are business domains clearly separated?
- Do components have different scaling requirements?
- Do they need independent release schedules?
- Can teams own services from development to production?
- Is there CI/CD, container orchestration and observability maturity?
- Can the organization handle distributed data, network failures and on-call operations?

## Production example

A team may begin with a modular monolith and extract the payment or notification capability later if it needs independent scaling, release cadence or security boundaries. This is a gradual decomposition rather than a full rewrite.

## Follow-up questions

1. How would you split a monolith into microservices?
2. What is the strangler pattern?
3. How would you avoid creating a distributed monolith?
4. How would you assign service ownership?
5. What platform capabilities are needed before adopting microservices?

## What the interviewer expects

- A reasoned decision with trade-offs.
- Recognition that team size alone is not enough.
- Understanding of DevOps maturity, ownership and operational overhead.
- A practical migration path rather than an all-at-once rewrite.

## What to avoid

- Do not say “10 developers means microservices.”
- Do not split services by technical layer alone (for example, one service for all database work).
- Do not ignore testing, monitoring, security and on-call costs.

## Quick revision keywords

`Team Topology` · `Bounded Context` · `Modular Monolith` · `Independent Scaling` · `Service Ownership` · `Platform Maturity` · `Strangler Pattern` · `Distributed Monolith`

---

# Q6. If an entire AWS Availability Zone or Region goes down, how do you keep the application running?

## Interview-ready answer

“First, I would distinguish an Availability Zone failure from a full Region failure.

1. **AZ failure:** Deploy the application across multiple AZs behind a load balancer. Use health checks and Auto Scaling so unhealthy targets are removed and replacement capacity is launched. Use an appropriately configured Multi-AZ database.
2. **Region failure:** Multi-AZ in one Region is not enough I would deploy a recovery environment in another Region.
3. **Traffic failover:** Use Route 53 health checks and routing policies or AWS Global Accelerator to direct traffic to the healthy Region.
4. **Data recovery:** Replicate data using an appropriate service-specific approach, such as Aurora Global Database, DynamoDB Global Tables or S3 Cross-Region Replication.
5. **Recovery process:** Automate infrastructure deployment with Terraform or CloudFormation, maintain tested runbooks and regularly run DR exercises.

I would choose active-active or warm-standby based on the required RTO, RPO, consistency needs and budget. I would never promise zero interruption or zero data loss without validating the design.”

## AZ vs Region failure

| Failure | Typical approach |
|---|---|
| One instance | Health checks and Auto Scaling replace it |
| One AZ | Multi-AZ application capacity and database failover |
| Full Region | Cross-Region recovery, traffic failover and data replication |

## DR strategies

- **Backup and restore:** Lowest standby cost; recovery takes longer.
- **Pilot light:** Core data and essential services are replicated; scale up during recovery.
- **Warm standby:** A smaller but working environment runs in the recovery Region.
- **Multi-site / active-active:** Both Regions serve traffic; highest complexity and cost.

## RTO and RPO

- **RTO (Recovery Time Objective):** Maximum acceptable time to restore the service.
- **RPO (Recovery Point Objective):** Maximum acceptable data-loss window measured in time.

Example: If RTO is 30 minutes and RPO is 5 minutes, the recovery design must restore service within 30 minutes and limit data loss to a maximum of 5 minutes. Actual results must be proven through tests.

## Production example

For an e-commerce application, keep application capacity in two AZs in the primary Region. Maintain a warm-standby environment in a second Region, replicate supported data, replicate required container images/configuration, and use health-based DNS or Global Accelerator failover. During a regional event, validate database promotion and application dependencies before shifting all traffic.

## Security and reliability best practices

- Encrypt replicated data and backups.
- Restrict cross-Region replication roles and network access.
- Keep secrets and configuration available securely in the recovery Region.
- Protect backups from accidental deletion and ransomware.
- Replicate container images and deployment artifacts where required.
- Test DNS, certificates, quotas, IAM, dependencies and database promotion.
- Define failback procedures; failback can be as complex as failover.

## Follow-up questions

1. What is the difference between Multi-AZ and Multi-Region?
2. What are RTO and RPO?
3. What is the difference between active-active and active-passive?
4. How does Aurora Global Database help?
5. What is the difference between read replica and Multi-AZ standby?
6. How do you test disaster recovery?
7. How do you avoid split-brain or conflicting writes?

## What the interviewer expects

- Clear distinction between AZ resilience and regional disaster recovery.
- Understanding of data replication and consistency.
- A DR strategy tied to business RTO/RPO and cost.
- Awareness that failover requires testing, not just configuration.

## What to avoid

- Do not claim Multi-AZ protects against a full Region outage.
- Do not assume every AWS service replicates automatically across Regions.
- Do not assume DNS failover is instantaneous; DNS caching and client behavior matter.
- Do not promise zero data loss without synchronous replication and a validated design.
- Do not forget dependencies such as secrets, certificates, quotas, images and external integrations.

## Quick revision keywords

`Multi-AZ` · `Multi-Region` · `Health Checks` · `Failover` · `Active-Active` · `Warm Standby` · `Aurora Global Database` · `DynamoDB Global Tables` · `S3 CRR` · `RTO` · `RPO` · `DR Drill` · `Failback`

---

# Q7. How would you secure an e-commerce application and AWS infrastructure?

## Interview-ready answer

“I would apply security in layers using a defence-in-depth approach.

1. **DDoS and edge:** AWS Shield for DDoS protection, AWS WAF for web filtering, rate-based rules and managed rules.
2. **Identity:** IAM roles, least privilege, MFA for privileged users, federation/SSO and separate permissions for developers, CI/CD and workloads.
3. **Network:** Private subnets for application and databases, restrictive security groups, controlled egress and VPC endpoints where suitable.
4. **Application:** Secure coding, input validation, authentication, authorization, dependency scanning, container scanning and API rate limiting.
5. **Data:** TLS in transit, KMS encryption at rest, Secrets Manager for credentials, rotation and database access restrictions.
6. **Detection and audit:** CloudTrail, relevant service logs, security findings, alerts and incident-response runbooks.

I would also patch systems, scan infrastructure as code, back up critical data and regularly test security controls.”

## Security layers

| Layer | Controls |
|---|---|
| Edge | Shield, WAF, CloudFront, rate limiting |
| Identity | IAM, roles, least privilege, MFA, federation |
| Network | VPC, private subnets, security groups, NACLs, endpoints |
| Compute/container | Hardened images, patching, non-root containers, image scanning |
| Application/API | Authentication, authorization, input validation, API throttling |
| Data | TLS, KMS, Secrets Manager, backups, access controls |
| Audit/detection | CloudTrail, GuardDuty, Security Hub, Config, logs and alerts |

## Production example

A customer accesses the e-commerce site through CloudFront and WAF. The ALB is the only public application entry point. Application tasks or pods run in private subnets and access the database through restricted security-group rules. Workloads use IAM roles rather than embedded AWS keys. Database credentials are stored in Secrets Manager and encrypted with KMS. CloudTrail and security monitoring provide audit and detection.

## Practical best practices

- Apply least privilege to human users, CI/CD roles and application roles.
- Use MFA and short-lived federated credentials for human access.
- Never commit access keys, passwords or tokens to Git.
- Use Secrets Manager/Parameter Store and rotate secrets.
- Encrypt data in transit and at rest.
- Restrict inbound and outbound network paths.
- Use WAF managed rules and rate limits; tune to avoid blocking valid customers.
- Scan dependencies, IaC, container images and machine images.
- Patch OS, runtimes and Kubernetes components.
- Enable CloudTrail and protect audit logs.
- Use backups with access controls and test restores.
- Define incident response, key rotation and breach containment procedures.

## Follow-up questions

1. What is the difference between Shield and WAF?
2. What is the difference between security groups and NACLs?
3. How does IAM role assumption work?
4. How do you store secrets in AWS?
5. What is KMS and envelope encryption?
6. How would you secure an EKS cluster?
7. How do you prevent public access to an S3 bucket?
8. What is the shared responsibility model?

## What the interviewer expects

- Defence in depth across identity, network, application and data.
- Practical least-privilege IAM and secret handling.
- Understanding of DDoS versus application-layer filtering.
- Logging, detection, patching and response—not only preventive controls.

## What to avoid

- Do not say WAF alone prevents every DDoS attack.
- Do not use one administrator role for all workloads.
- Do not hardcode secrets in source code, Docker images or Terraform files.
- Do not expose databases directly to the internet.
- Do not assume encryption alone replaces access control.
- Do not claim AWS secures customer configurations automatically; understand shared responsibility.

## Quick revision keywords

`Defence in Depth` · `Shield` · `WAF` · `IAM` · `Least Privilege` · `MFA` · `Federation` · `Private Subnets` · `Security Groups` · `TLS` · `KMS` · `Secrets Manager` · `CloudTrail` · `GuardDuty` · `Security Hub` · `Vulnerability Scanning`

---

# Q8. Give five cost optimization techniques without compromising performance and availability.

## Interview-ready answer

“I would optimize based on usage data and business criticality, not simply reduce resources.

1. **Right-sizing:** Analyze CloudWatch CPU, memory, network and utilization; resize overprovisioned EC2, RDS and container workloads.
2. **Purchase models:** Use Savings Plans or Reserved Instances for predictable baseline usage; use Spot for fault-tolerant, interruptible workloads.
3. **Auto Scaling and scheduling:** Scale capacity with demand and stop non-production resources outside working hours where safe.
4. **Storage and data transfer:** Apply S3 lifecycle policies, delete unused volumes/snapshots carefully, compress data and review cross-AZ/cross-Region transfer.
5. **Observability and architecture:** Set log retention, sampling and metric-cardinality controls; use caching and managed/serverless services when they reduce total cost.

I would track cost by tags and account, review Cost Explorer and budgets, and validate every change against latency, availability and recovery requirements.”

## Five practical areas

1. **Compute:** Rightsizing, Auto Scaling, Savings Plans/Reserved Instances, Spot for suitable workloads.
2. **Scheduling:** Stop/start development and test resources outside working hours; confirm dependencies and startup behavior.
3. **Storage:** S3 lifecycle to suitable storage classes, EBS volume cleanup, snapshot retention and database storage review.
4. **Network:** Review NAT Gateway processing, data transfer, cross-AZ traffic and CDN caching opportunities.
5. **Monitoring/platform:** Log retention, metric cardinality, trace sampling, unused resources and overprovisioned observability tiers.

## Production example

Use CloudWatch to identify lower-environment EC2 instances that consistently run at low utilization. Validate memory, CPU, disk and performance requirements before downsizing. Schedule non-production instances to stop overnight only after confirming no batch jobs, integrations or test windows depend on them. Keep production capacity and failover headroom.

## Security and reliability guardrails

- Do not downsize below tested peak demand and failover capacity.
- Do not use Spot for workloads that cannot tolerate interruption.
- Preserve backup retention and security logs required by policy.
- Test scheduled shutdown/startup and service recovery.
- Use budgets and anomaly detection; cost optimization should not remove security controls.
- Review total cost, including data transfer, support and operational effort.

## Follow-up questions

1. Savings Plans vs Reserved Instances?
2. When would you use Spot Instances?
3. How do you right-size EC2?
4. How do you reduce S3 cost?
5. How do you reduce CloudWatch log cost?
6. What is the difference between scale-up and scale-out?
7. How do you ensure cost reduction does not affect availability?

## What the interviewer expects

- Measured optimization using utilization and cost data.
- Awareness of compute, storage, network, database and observability costs.
- Workload-aware purchasing decisions.
- Safety guardrails and post-change validation.

## What to avoid

- Do not say “use Spot everywhere.”
- Do not stop production resources without a resilience and business-impact assessment.
- Do not delete backups or security logs just to reduce cost.
- Do not right-size based on CPU alone if memory, I/O or peak traffic matters.

## Quick revision keywords

`Right-Sizing` · `Cost Explorer` · `Budgets` · `Savings Plans` · `Reserved Instances` · `Spot` · `Auto Scaling` · `Scheduling` · `S3 Lifecycle` · `EBS Cleanup` · `Data Transfer` · `Log Retention` · `FinOps`

---

# Q9. How do you establish secure connectivity between on-premises infrastructure and AWS?

## Interview-ready answer

“I would choose the connectivity based on bandwidth, latency, availability, security and cost.

1. **Site-to-Site VPN:** An encrypted IPsec tunnel over the internet. It is faster to establish and suitable for lower or variable bandwidth needs, and can also provide backup connectivity.
2. **AWS Direct Connect:** A dedicated private network connection for more consistent bandwidth and latency. It is not encrypted by default, so sensitive traffic may need additional encryption such as a VPN over Direct Connect.
3. **Transit Gateway:** A central hub to connect multiple VPCs and on-premises networks at scale.
4. **Routing:** Use BGP where appropriate, plan non-overlapping CIDR ranges, and control route propagation and segmentation.
5. **Security and operations:** Restrict security groups and network paths, monitor tunnel/link health, log network activity and test failover.

For critical enterprise connectivity, I would commonly consider redundant Direct Connect paths with separate locations and a VPN backup, depending on the availability requirement.”

## Connectivity options

| Option | Characteristics | Typical use |
|---|---|---|
| Site-to-Site VPN | IPsec encrypted tunnels over internet | Quick setup, backup or moderate traffic |
| Direct Connect | Dedicated private connectivity; not encrypted by default | Predictable bandwidth and latency |
| Transit Gateway | Hub-and-spoke routing across VPCs and on-premises | Many VPCs/accounts/sites |
| VPN over Direct Connect | Adds encryption over the private connection | When encryption is required over DX |

## Production example

An enterprise has a data centre and several AWS VPCs. Establish redundant Direct Connect connections through separate locations, attach them to a Transit Gateway through the appropriate Direct Connect gateway design, and use BGP for route exchange. Configure Site-to-Site VPN as a backup path. Apply route segmentation and firewall/security-group controls.

## Security and reliability best practices

- Use IPsec VPN where encryption over the public internet is required.
- Remember Direct Connect is private connectivity, not encryption by itself.
- Use redundant links and separate physical locations for resilience.
- Plan non-overlapping CIDR blocks and route tables.
- Apply least-privilege network access and firewall rules.
- Monitor BGP sessions, tunnel status, packet loss, latency and bandwidth.
- Document routing, failover and ownership; test failover regularly.

## Follow-up questions

1. Direct Connect vs Site-to-Site VPN?
2. Is Direct Connect encrypted by default?
3. What is BGP?
4. What is Transit Gateway?
5. What is a Direct Connect gateway?
6. How do you design redundant connectivity?
7. What happens if the primary link fails?
8. Why avoid overlapping CIDR ranges?

## What the interviewer expects

- You can select the right connectivity option based on requirements.
- You understand encryption, routing, redundancy and failover.
- You know Transit Gateway's hub-and-spoke role.
- You understand Direct Connect is not automatically encrypted.

## What to avoid

- Do not say Direct Connect is an encrypted VPN.
- Do not assume one VPN tunnel or one Direct Connect circuit provides complete high availability.
- Do not ignore BGP, route propagation, CIDR overlap and network segmentation.
- Do not promise fixed latency or bandwidth without checking the selected service and provider design.

## Quick revision keywords

`Site-to-Site VPN` · `IPsec` · `Direct Connect` · `BGP` · `Transit Gateway` · `Direct Connect Gateway` · `Redundant Links` · `Route Propagation` · `CIDR Planning` · `Network Segmentation`
