# Kubernetes Production Setup – Point-wise Notes

* **Technology:** Amazon EKS / Kubernetes
* **Target Audience:** DevOps Engineers, SREs, Cloud Architects

---

## 1. Scalability

> 💡 **In Simple Terms:** The ability to **handle more work when traffic grows** without crashing or slowing down. 
> * **Analogy:** Like a restaurant during rush hour. You either hire more chefs (**Horizontal Scaling**) or give your current chef a faster, bigger stove (**Vertical Scaling**) so customers aren't left waiting.

* **HPA (Horizontal Pod Autoscaler):** Automatically increases or decreases Pod replicas based on CPU, memory, or custom metrics.
* **VPA (Vertical Pod Autoscaler):** Adjusts or recommends CPU and memory requests for containers.
* **Karpenter:** Automatically provisions suitable worker nodes when additional capacity is required.
* **Cluster Autoscaler:** Automatically increases or decreases the size of configured node groups.
* **KEDA:** Scales workloads based on event sources, such as SQS queue length.
* **Resource Requests:** Define minimum CPU and memory required by each container.
* **Resource Limits:** Set maximum CPU and memory a container can consume.
* **Redis Cache:** Reduces repeated database calls and application response time.
* **CloudFront CDN:** Caches static content closer to global users.
* **SQS:** Handles sudden traffic spikes through asynchronous processing.

---

## 2. High Availability (HA)

> 💡 **In Simple Terms:** Making sure your application is **always accessible**, even if hardware breaks.
> * **Analogy:** Like a global bank keeping its data on multiple synchronized servers across different cities. If a blackout hits one city, the other locations instantly pick up the slack, and customers never notice a disruption.

* **Multi-AZ Cluster:** Distribute worker nodes across multiple Availability Zones.
* **Multiple Replicas:** Run multiple instances of critical application Pods.
* **Pod Anti-Affinity:** Prevent replicas from running on the same node or failure domain.
* **Topology Spread Constraints:** Distribute Pods across different AZs.
* **Load Balancer:** Use ALB to distribute traffic across healthy application endpoints.
* **Readiness Probe:** Send traffic only to healthy and ready Pods.
* **Pod Disruption Budget (PDB):** Maintain a minimum number of available Pods during voluntary disruptions.
* **Rolling Updates:** Deploy new application versions without taking down all replicas.
* **Multi-AZ Database:** Use RDS/Aurora Multi-AZ for database availability.
* **Spare Capacity:** Maintain enough worker-node capacity to handle an AZ failure.

---

## 3. Reliability

> 💡 **In Simple Terms:** Making sure the system **works correctly and predictably** under normal and heavy conditions.
> * **Analogy:** Like a modern car equipped with anti-lock brakes (ABS) and structural sensors. It monitors its own internal health, safely flags issues on the dashboard, and prevents catastrophic failures before they happen.

* **Liveness Probe:** Restart containers that become unhealthy or stuck.
* **Readiness Probe:** Prevent traffic from reaching Pods that are not ready.
* **Startup Probe:** Allow slow-starting applications sufficient initialization time.
* **Resource Requests and Limits:** Prevent uncontrolled resource consumption.
* **Rolling Deployment:** Update applications gradually.
* **Rollback:** Revert to a previous stable application version.
* **Graceful Shutdown:** Allow applications to finish ongoing requests before termination.
* **Timeouts:** Prevent requests from waiting indefinitely.
* **Retry Mechanism:** Retry temporary failures with exponential backoff.
* **Circuit Breaker:** Stop repeated requests to an unhealthy downstream service.
* **Idempotency:** Prevent duplicate operations during retries.
* **Resource Quotas:** Control resource consumption at namespace level.

---

## 4. Fault Tolerance

> 💡 **In Simple Terms:** The ability to **survive sudden, unexpected disasters** and keep running normally.
> * **Analogy:** Like a commercial airplane equipped with two engines. If one engine suddenly fails mid-flight, the plane doesn't crash; the secondary engine automatically carries the weight to safely reach the destination.

* **Container Failure:** Kubelet restarts failed containers according to the restart policy.
* **Pod Failure:** Deployment creates replacement Pods to maintain the desired replica count.
* **Node Failure:** Kubernetes reschedules workloads onto healthy nodes when possible.
* **AZ Failure:** Remaining AZs continue serving traffic if capacity and dependencies are available.
* **Pod Replicas:** Ensure multiple application instances are available.
* **Auto Healing:** Automatically replace unhealthy application Pods.
* **Load Balancer Health Checks:** Stop routing traffic to unhealthy targets.
* **Persistent Storage:** Use suitable persistent volumes for stateful workloads.
* **Message Queue:** Use SQS to prevent temporary downstream failures from losing background work.
* **Dead Letter Queue (DLQ):** Store messages that repeatedly fail processing.
* **Database Replication:** Maintain database replicas using database-supported replication.
* **Multi-Region DR:** Maintain a recovery environment in another AWS Region.

---

## 5. Security

> 💡 **In Simple Terms:** **Protecting your system and data** from unauthorized access, attacks, or mistakes.
> * **Analogy:** Like a high-security military base. It features a perimeter fence (**Network Security**), strictly vetted ID badges (**RBAC**), locked vaults for classified documents (**Secrets Security**), and background checks on all supplies entering the gates (**Container Security**).

### Cluster Security
* **RBAC:** Control who can access Kubernetes resources.
* **IAM Roles:** Assign AWS permissions to workloads using IRSA or EKS Pod Identity.
* **Least Privilege:** Grant only the permissions required.
* **Private API Endpoint:** Restrict Kubernetes API access where appropriate.
* **Audit Logging:** Track Kubernetes API activities.
* **Namespace Isolation:** Separate applications and environments.

### Network Security
* **Private Subnets:** Keep worker nodes in private subnets where appropriate.
* **Security Groups:** Restrict inbound and outbound traffic.
* **Network Policies:** Control communication between Pods.
* **TLS Encryption:** Encrypt communication between clients and services.
* **AWS WAF:** Protect internet-facing applications from common web attacks.
* **AWS Shield:** Protect against DDoS attacks.

### Container Security
* **Image Scanning:** Scan Docker images for vulnerabilities.
* **Trusted Images:** Use approved and minimal base images.
* **Non-root User:** Run containers without root privileges.
* **Read-only Filesystem:** Prevent unnecessary filesystem modifications.
* **Drop Capabilities:** Remove unnecessary Linux capabilities.
* **Image Tagging:** Use versioned image tags or immutable digests.
* **Pod Security Standards:** Enforce restricted workload security settings.

### Secrets Security
* **AWS Secrets Manager:** Store database passwords and API credentials.
* **Kubernetes Secrets:** Store sensitive configuration with appropriate protection.
* **KMS Encryption:** Encrypt secrets and persistent data.
* **Secret Rotation:** Rotate credentials periodically.
* **No Hardcoded Credentials:** Never store passwords or access keys in Git or Dockerfiles.

---

## 6. Monitoring and Observability

> 💡 **In Simple Terms:** **Seeing exactly what is happening** inside your system at any given second.
> * **Analogy:** Like the cockpit dashboard of a commercial jetliner. Pilots don't guess their speed or altitude; they rely on gauges, real-time warning alarms, and black-box flight logs to see the health of the entire aircraft.

* **CloudWatch:** Monitor EKS cluster and application metrics.
* **Prometheus:** Collect Kubernetes and application metrics.
* **Grafana:** Create monitoring dashboards.
* **CloudWatch Logs:** Centralize application and container logs.
* **Alertmanager:** Send alerts based on Prometheus rules.
* **AWS X-Ray:** Trace requests across application services.
* **Container Insights:** Monitor nodes, Pods and cluster performance.
* **Kubernetes Events:** Identify scheduling and workload failures.
* **Application Metrics:** Monitor request rate, latency, and error percentage.
* **SLO/SLI:** Measure service reliability and availability.

---

## 7. Backup and Disaster Recovery

> 💡 **In Simple Terms:** Having a plan and a **spare copy of your system** to recover if everything gets wiped out.
> * **Analogy:** Like keeping a fireproof safe containing duplicates of your birth certificates, deeds, and family photos, alongside a comprehensive insurance plan to completely rebuild your house if a disaster strikes.

* **Velero:** Back up and restore Kubernetes resources and supported persistent volumes.
* **EBS Snapshots:** Back up persistent volumes where applicable.
* **Database Backups:** Configure automated database backups.
* **S3:** Store backup data securely.
* **Cross-Region Replication:** Replicate required data to another AWS Region.
* **Terraform:** Recreate infrastructure using Infrastructure as Code.
* **Helm:** Maintain application deployment configurations.
* **Git:** Store Kubernetes manifests and deployment files.
* **RTO (Recovery Time Objective):** Define the maximum acceptable time to get the system back online.
* **RPO (Recovery Point Objective):** Define the maximum acceptable data loss measured in time.
* **DR Testing:** Regularly test backup restoration and regional recovery.

---

## 8. Important Kubernetes Commands

| Purpose | Command |
| :--- | :--- |
| Check nodes | `kubectl get nodes -o wide` |
| Check Pods | `kubectl get pods -A -o wide` |
| Check deployments | `kubectl get deployments -A` |
| Check services | `kubectl get svc -A` |
| Check HPA | `kubectl get hpa -A` |
| Check PDB | `kubectl get pdb -A` |
| Check resource usage | `kubectl top nodes` |
