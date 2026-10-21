# Domain 2: Design for New Solutions — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 29% of AWS Certified Solutions Architect - Professional (SAP-C02)  
> **Core Objective:** Design enterprise-scale, greenfield cloud architectures that satisfy multi-dimensional functional and non-functional requirements: high reliability, security, multi-region disaster recovery, edge acceleration, and modern data streaming.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [AWS Well-Architected Framework (Six Pillars)](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
* **Canonical Scope:** The foundational architectural doctrine for all AWS solutions.
* **Six Pillars:**
  1. **Operational Excellence:** Running and monitoring systems to deliver business value continually.
  2. **Security:** Protecting information, systems, and assets while delivering business value via risk assessments and mitigation strategies.
  3. **Reliability:** The ability of a system to recover from infrastructure or service disruptions and dynamically acquire resources.
  4. **Performance Efficiency:** Using computing resources efficiently to meet requirements and maintain efficiency as demand changes.
  5. **Cost Optimization:** Avoiding unneeded costs and running workloads at the lowest price point.
  6. **Sustainability:** Minimizing the environmental impacts of running cloud workloads.

### 1.2 [Disaster Recovery of Workloads on AWS: Recovery in the Cloud](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/welcome.html)
* **Canonical Scope:** Architectural blueprints for RPO/RTO compliance.
* **Multi-Region Patterns:**
  * **Active-Passive (Pilot Light / Warm Standby):** Data replicated continuously; application tier scaled up upon Route 53 ARC DNS switch.
  * **Active-Active (Multi-Region):** Global DynamoDB / Aurora Global Database with Route 53 Geolocation / Latency routing and Application Recovery Controller (ARC).

### 1.3 [Designing Data Lakes on AWS with AWS Lake Formation](https://docs.aws.amazon.com/whitepapers/latest/designing-data-lakes-on-aws/welcome.html)
* **Canonical Scope:** Enterprise data governance, centralized access control, and cross-account data sharing.
* **Key Architecture:**
  * Decoupling storage (S3) from compute (Athena, EMR, Redshift, QuickSight).
  * Granular column-level, row-level, and cell-level permissions enforced via Lake Formation without managing complex IAM policies or separate S3 bucket copies.

### 1.4 [Best Practices for Securing Sensitive Data at the Edge](https://docs.aws.amazon.com/whitepapers/latest/secure-content-delivery-amazon-cloudfront/welcome.html)
* **Canonical Scope:** Perimeter protection architectures using CloudFront, WAF, Shield Advanced, and Global Accelerator.
* **Edge Compute Decision Matrix:**
  | Feature | CloudFront Functions | Lambda@Edge |
  | :--- | :--- | :--- |
  | **Runtime** | Lightweight JavaScript (ES 5.1) | Node.js, Python |
  | **Execution Location** | 600+ CloudFront PoPs (Edge) | 13 Regional Edge Caches |
  | **Execution Time** | Sub-millisecond ($<1$ ms) | Up to 5s (viewer), 30s (origin) |
  | **Memory Limit** | 2 MB | 128 MB to 10 GB |
  | **Network / Body Access** | No network calls; No request body | Full VPC/Internet access; Body read/write |
  | **Use Cases** | URL rewrite, header normalization, JWT validation | Dynamic HTML rendering, complex auth, payload mutation |

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [AWS Global Accelerator & Amazon CloudFront](https://docs.aws.amazon.com/global-accelerator/latest/dg/)
* **Deep Dive:**
  * **Global Accelerator:** Anycast static IP addresses routing TCP/UDP traffic over the AWS global private fiber backbone directly to ALBs, NLBs, or EC2 instances in single or multiple regions. Ideal for non-HTTP protocols, gaming, VoIP, and instant failover.
  * **CloudFront Field-Level Encryption:** Encrypting sensitive credit card or SSN fields at the edge using public/private key pairs before the payload traverses origins.

### 2.2 [Amazon EKS Multi-Tenancy & Karpenter](https://docs.aws.amazon.com/eks/latest/userguide/)
* **Deep Dive:**
  * Multi-tenant cluster isolation using Kubernetes Namespaces, NetworkPolicies, and IAM Roles for Service Accounts (IRSA) / EKS Pod Identity.
  * **Karpenter:** Fast, intelligent Kubernetes node autoscaler that provisions rightsized compute directly from EC2 capacity pools without Auto Scaling Group constraints.

### 2.3 [Amazon Event-Driven & Streaming Architectures](https://docs.aws.amazon.com/kinesis/latest/dev/)
* **Deep Dive:**
  * **Amazon Kinesis Data Streams:** Real-time ingestion with dedicated throughput per shard (1 MB/s in, 2 MB/s out) and Enhanced Fan-Out (dedicated 2 MB/s per consumer via HTTP/2).
  * **Amazon MSK (Managed Streaming for Apache Kafka):** Native Kafka compatibility, cross-cluster replication (MirrorMaker 2), and multi-VPC private connectivity.
  * **Amazon SQS FIFO + Step Functions:** Exactly-once processing and ordered message handling at high scale.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **RPO of Seconds, RTO $< 1$ Minute Multi-Region Database:**
   * Standard RDS cross-region read replicas fail the RTO requirement due to manual promotion and DNS propagation.
   * *Resolution:* **Amazon Aurora Global Database** provides storage-level replication $<1$ second latency and managed failover taking $<1$ minute without data loss.
2. **High-Throughput Global Ingestion with Uncached Payloads:**
   * Caching is ineffective for real-time dynamic IoT or transaction ingestion.
   * *Resolution:* Use **AWS Global Accelerator** to terminate TCP at the edge and route over the private AWS global backbone directly to regional NLBs.
3. **Cross-Account Data Mesh with Lake Formation:**
   * Sharing petabyte datasets across dozens of accounts without data duplication.
   * *Resolution:* AWS Lake Formation cross-account resource links and tag-based access control (TBAC) shared via AWS RAM.

---

## 4. Reading & Study Checklist

- [ ] Read *AWS Well-Architected Framework Whitepaper*
- [ ] Read *Disaster Recovery of Workloads on AWS: Recovery in the Cloud*
- [ ] Read *Designing Data Lakes on AWS with AWS Lake Formation*
- [ ] Master CloudFront Functions vs Lambda@Edge architectural boundaries
- [ ] Review AWS Global Accelerator vs Amazon CloudFront use-case distinctions
- [ ] Study Aurora Global Database and DynamoDB Global Tables multi-region patterns
