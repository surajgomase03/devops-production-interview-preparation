# AWS Compute Optimization & Cost Architecture Guide

* **Target Focus:** Cloud Financial Operations (FinOps), Systems Engineering, Compute Infrastructure, and Cost Management
* **Core Domains:** Compute Commitment Models, Spot Instance Orchestration, and Automated Resource Right-Sizing

---

## 1. Savings Plans vs. Reserved Instances (RIs)

> 💡 **In Simple Terms:** Both options provide deep infrastructure discounts (up to 72%) in exchange for committing to a 1-year or 3-year usage term with AWS. The defining difference is architectural freedom: RIs lock you into specific deployment choices, while Savings Plans automatically adapt as your infrastructure evolves.
> * **Analogy:** **Reserved Instances** are like purchasing a non-refundable train ticket for a hyper-specific route, date, and seat—if your plans change and you need to take a different line, the ticket loses its entire value. **Savings Plans** are like loading a flexible transit card with a set cash budget—you can use it on trains, buses, or subways anywhere in the network, as long as you meet your hourly spend promise.

| Evaluation Metric | Savings Plans (Compute Option) | Reserved Instances (RIs) |
| :--- | :--- | :--- |
| **Commitment Mechanic** | **Monetary spend per hour** (e.g., committing to spend \$10/hour across the account). | **Number of specific instances** (e.g., committing to run 5 `m5.large` instances). |
| **Architectural Flexibility** | **Maximum.** Automatically applies to EC2 instances, AWS Lambda functions, and AWS Fargate containers regardless of OS, Region, or instance family. | **Limited.** Standard RIs lock you into specific instance families, geographic regions, and operating systems. |
| **Management Overhead** | **Very Low.** Automatically calculates and shifts discount coverage as your developers spin new infrastructure up or down. | **High.** Requires operational overhead to manually modify, exchange, or sell unused instances on the AWS Marketplace. |
| **Primary Use Case** | Highly dynamic cloud environments running containerized apps, utilizing serverless, or shifting instance sizes frequently. | Static, monolithic workloads running on fixed, predictable EC2 virtual servers within a single geographic region. |

---

## 2. When to Use Spot Instances

> 💡 **In Simple Terms:** Buying AWS's excess, unused data center capacity at a massive discount (up to 90%). The catch is that AWS can reclaim the server instantly with a strict **2-minute warning** if a full-paying customer requires the compute capacity.
> * **Analogy:** Buying last-minute standby airline tickets. You get the exact same seat incredibly cheap, but if the flight fills up with full-price passengers, you get bumped off the plane before takeoff.

### Best Architectural Fit For:
* **Stateless Web Tiers:** Disposable application servers resting behind an Application Load Balancer where individual instances can drop out without severing active user sessions.
* **Batch Processing Jobs:** Heavy data transformation pipelines, image rendering, or video encoding where if a server terminates, the work manager can safely retry the block on a new instance.
* **CI/CD Build Runners:** Automated software compilation and unit testing workers that can fail and restart cleanly without impacting production availability.
* **Distributed Big Data Workers:** Analytics nodes (e.g., Amazon EMR, Spark worker pools) where a persistent master node coordinates tasks and handles individual worker node drops gracefully.

### ❌ NEVER Deploy Spot Instances For:
* **Production Relational Databases:** (e.g., Amazon RDS, self-hosted PostgreSQL/MySQL clusters) where sudden server termination can cause catastrophic write failure and data corruption.
* **Stateful Legacy Applications:** Monolithic software platforms that store live user session state or ongoing file streams in local server memory.
* **Core Microservice APIs:** Crucial microservices that demand continuous, low-latency availability and cannot tolerate temporary traffic drops during server swaps.

---

## 3. How to Right-Size EC2 Instances

> 💡 **In Simple Terms:** Auditing your cloud infrastructure to eliminate waste by shrinking oversized servers down to match their actual processing demands.
> * **Analogy:** Renting a massive commercial moving box truck to transport a single school backpack. Right-sizing means swapping that expensive, empty truck for a compact sedan because it perfectly fits the size of your cargo.

### Step 1: Aggregate Performance Telemetry
Utilize **AWS Compute Optimizer** or **Amazon CloudWatch** to review historical infrastructure utilization metrics over a minimum 14-day observation window:
* **Identify Waste:** An instance is heavily oversized if peak **CPU utilization remains < 20%** and **Memory utilization stays < 40%**.
* **Filter for Steady State:** Focus metrics analysis on sustained average workloads rather than sharp, short-lived spikes to find the true compute resource requirement.

### Step 2: Execute Target Optimization Actions
* **Downsize Within Family:** If an `m5.2xlarge` server averages low single-digit resource usage, modify the instance configuration down to an `m5.large`.
* **Cross-Elastic Family Swaps:** If a workload shows sustained high memory usage but near-zero CPU usage, modify the instance family away from General Purpose (`m5`) and move it to Memory-Optimized (`r5`).
* **Modernize Infrastructure Generations:** Upgrade legacy architecture versions to current equivalents (e.g., shifting from `t3` to `t4g`). Migrating to **AWS Graviton processors** (`g` instance variants) provides up to **40% better price-performance** over legacy x86 hardware.

### Step 3: Architect Elastic Safeguards
* **Scale Horizontally, Not Vertically:** Rather than paying for one massive instance that sits mostly idle to handle peak daily traffic spikes, deploy multiple smaller instances inside an **Auto Scaling Group**. This setup allows the cluster to multiply dynamically when busy and automatically shrink down when traffic drops.
