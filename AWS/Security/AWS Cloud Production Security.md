# AWS Cloud Production Security – Comprehensive Architecture Guide

* **Target Focus:** Cloud Security Architecture, DevSecOps, Identity Management, Network Isolation, and Compliance
* **Core Domains:** Defense-in-Depth Strategy, Zero Trust Architecture, Infrastructure Hardening, and Automated Threat Detection

---

## The Core Paradigm: Defense in Depth

> 💡 **In Simple Terms:** Security in AWS means never relying on a single lock to protect your house. Instead, you design a system where an attacker must bypass an exterior gate, security guards, biometric doors, alarms, and a heavy fireproof vault before reaching the data. Even if one security mechanism fails, the remaining layers completely contain the threat.

---

## 1. The 7-Layer AWS Infrastructure Security Matrix

This matrix outlines the specific, independent architectural controls required across the modern AWS cloud application stack.

| Layer | Primary Objective | Key Strategic Controls | AWS Native Services |
| :--- | :--- | :--- | :--- |
| **1. Edge** | Stop distributed attacks, malicious traffic, and scraping before it touches your network. | Global DDoS mitigation, IP rate limiting, geo-blocking, malicious bot filtering. | AWS Shield Advanced, AWS WAF, Amazon CloudFront |
| **2. Identity** | Authenticate and authorize every single request, user, and automated service loop. | Multi-Factor Authentication (MFA), Single Sign-On (SSO), Identity Federation, Ephemeral tokens. | AWS IAM, AWS IAM Identity Center (SSO), AWS STS |
| **3. Network** | Isolate and restrict all internal packet movement to trusted traffic paths only. | Micro-segmentation, private routing, drop-by-default virtual firewalls, egress proxies. | Amazon VPC Subnets, Security Groups, Network ACLs, VPC Endpoints |
| **4. Compute** | Protect the underlying compute host, EC2 server instance, or serverless execution engine. | OS Hardening, automated patching, minimal base images, continuous vulnerability scanning. | Amazon Inspector, EC2 Image Builder, AWS Systems Manager (SSM) |
| **5. Application / API** | Prevent external exploitation of business logic vulnerabilities and software flaws. | Input sanitization, API token authentication (OAuth/JWT), payload sizing, rate limiting. | Amazon API Gateway, AWS AppSync |
| **6. Data** | Protect information at rest and in transit from extraction, exposure, or corruption. | Envelope encryption, customer-managed keys, automated credential rotation, strict access control. | AWS KMS, AWS Secrets Manager, AWS ACM (TLS) |
| **7. Audit / Detection** | Gain complete timeline visibility, track user actions, and catch intrusions immediately. | Automated behavioral analytics, continuous compliance checking, centralized log storage. | AWS CloudTrail, Amazon GuardDuty, AWS Security Hub, AWS Config |

---

## 2. Deep Dive: Layer-by-Layer Production Configuration

### Layer 1: Edge Security
* **WAF Layer Protections:** Deploy AWS WAF directly on CloudFront distributions. Enforce **AWS Managed Rule Groups** covering the OWASP Top 10 web vulnerabilities (SQL injection, Cross-Site Scripting).
* **Rate Limiting:** Implement a strict rate-limiting rule (e.g., maximum 100 requests per rolling 5-minute window per IP) on authentication endpoints like `/login` or `/checkout` to completely block brute-force attacks.
* **DDoS Infrastructure Mitigation:** Keep all public web assets strictly behind CloudFront. Use AWS Shield Advanced to automatically absorb layer 3 and layer 4 network attacks without passing traffic load down to the backend compute layers.

### Layer 2: Identity and Access Management (IAM)
* **Workload Identity (Zero Hardcoded Keys):** Never place long-lived AWS IAM Access Keys inside app code, Git repositories, or environment files. Use **IAM Instance Profiles** to attach IAM roles directly to EC2 instances, allowing AWS to dynamically issue short-lived, temporary tokens.
* **Principle of Least Privilege:** Restrict IAM statements to target exact AWS resource ARNs instead of using wildcards (`*`). Include strict conditional clauses (`StringEquals: {"aws:PrincipalOrgID": "o-xxxxxx"}`) to prevent data extraction out of corporate boundaries.
* **Human Access Management:** Force all human access through AWS IAM Identity Center integrated with your company's Identity Provider (IdP like Okta or Azure AD). Enforce context-aware **Phishing-Resistant MFA (FIDO2/WebAuthn)** for every single login session.

### Layer 3: Network Isolation
* **Private Egress Topology:** Lock down all application compute layers, cache tiers, and database instances in strictly **Private Subnets**. Eliminate all direct internet routing paths. Force all external internet communication out through a restricted pool of highly monitored **NAT Gateways**.
* **VPC Endpoints (PrivateLink):** Ensure traffic sent from your backend applications to AWS internal systems (like S3, DynamoDB, or Secrets Manager) never travels across the public internet. Deploy Interface VPC Endpoints inside your network to route this traffic through completely private AWS backbone cables.
* **Stateful Firewalls:** Implement a **Default Drop (Deny All)** firewall stance for Security Groups. Explicitly allow ingress traffic only from specific source Security Groups on dedicated application ports (e.g., permit the Database Security Group to accept traffic *only* on port `5432` originating exclusively from the Application Load Balancer Security Group).

### Layer 4: Compute Hardening
* **Systems Manager (SSM) Session Manager:** Completely eliminate the need for open SSH (Port 22) or RDP ports on EC2 instances. Force all administrative access through AWS Systems Manager Session Manager, which secures all sessions using TLS encryption and logs every keystroke natively into CloudTrail.
* **Automated Patch Management:** Utilize AWS Systems Manager Patch Manager to automate OS patching cycles across your instance fleets based on predefined maintenance windows and severity criticalities.
* **Vulnerability Scanning Pipelines:** Utilize Amazon Inspector to continually scan EC2 instances and Amazon ECR container images for software vulnerabilities, alerting SRE teams automatically upon detection.

### Layer 5: Data Protection & Secrets Management
* **Cryptographic Envelope Encryption:** Enforce encryption on every AWS S3 bucket, EBS volume, and RDS instance using **AWS KMS Customer Managed Keys (CMKs)**. This ensures that even if raw underlying physical hard drives are compromised, data remains completely unreadable without appropriate KMS decryption authorization tokens.
* **Automated Secrets Rotation:** Store all database connection URLs, encryption keys, and third-party API keys within AWS Secrets Manager. Configure an automated AWS Lambda routine to automatically scramble and update these operational credentials every 30 days without causing service downtime.

### Layer 6: Audit & Continuous Threat Detection
* **Tamper-Proof Logging Auditing:** Centralize all **AWS CloudTrail** and VPC Flow Logs into an isolated security AWS account. Lock down this storage bucket using **S3 Object Lock** configured in *Compliance Mode* to prevent internal employees or compromised credentials from deleting or altering audit trails to hide illicit activity.
* **Automated Threat Hunting:** Turn on **Amazon GuardDuty** across all operational accounts. GuardDuty continuously analyzes DNS queries, CloudTrail data, and VPC network logs using machine learning to detect unusual activity like crypto-mining, strange cross-country API calls, or database exfiltration attempts.

---

## 3. High-Priority Diagnostic Security Commands

| Purpose | AWS CLI Command |
| :--- | :--- |
| **Verify AWS Caller Identity** | `aws sts get-caller-identity` |
| **Audit Active IAM Roles** | `aws iam list-roles --query "Roles[?contains(RoleName, 'prod')].{Name:RoleName, Arn:Arn}"` |
| **Check Active GuardDuty Detectors** | `aws guardduty list-detectors` |
| **List Attached IAM Policies on Role** | `aws iam list-attached-role-policies --role-name <role-name>` |
| **Scan Security Group Rules** | `aws ec2 describe-security-groups --group-ids <sg-id> --query "SecurityGroups[*].IpPermissions"` |

---

## 4. Key AWS Security Architecture Differences

| Structural Concept | Architectural Layer | Operational Mechanics | Primary Defensive Focus |
| :--- | :--- | :--- | :--- |
| **AWS IAM Policy** | Control Plane | Evaluates API operations to approve or deny resource modification actions. | Preventing accidental infrastructure deletion or access leaks. |
| **Security Group** | Network (Instance) | Stateful firewall that tracks return communication implicitly. | Protecting compute workloads from open port scanning. |
| **Network ACL** | Network (Subnet) | Stateless firewall requiring explicit inbound and outbound permit records. | Dropping traffic from blocked malicious network blocks at the subnet border. |
| **AWS KMS Key Policy** | Data Plane | Direct access control layer attached to encryption keys, independent of IAM. | Preventing unauthorized root users from decrypting sensitive data volumes. |
