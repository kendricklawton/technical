# AWS Certified Solutions Architect – Professional (SAP-C02) Study & Architecture Workspace

Welcome to the comprehensive study, architecture design, and exam preparation workspace for the **AWS Certified Solutions Architect – Professional (`SAP-C02`)** certification. This repository is structured directly around the official AWS `SAP-C02` Exam Guide and engineered for Principal/Senior Cloud Architects preparing for complex, multi-account, hybrid, and large-scale cloud transformation scenarios.

---

## 1. Exam Overview & Format

| Attribute | Details |
| :--- | :--- |
| **Exam Code** | `SAP-C02` |
| **Number of Questions** | **75 questions** (65 scored questions + 10 unscored pretest questions) |
| **Question Formats** | **Multiple Choice** (1 correct out of 4) and **Multiple Response** (2 or 3 correct out of 5 or 6) |
| **Allotted Time** | **180 minutes** (3 hours) — average **2.4 minutes (144 seconds) per question** |
| **Passing Score** | **750 / 1000** (scaled score; no penalty for guessing) |
| **Target Audience** | 2+ years of hands-on experience designing and deploying complex enterprise architectures on AWS |

---

## 2. Official SAP-C02 Exam Blueprint & Domain Weights

The `SAP-C02` exam is divided into four core architectural domains:

| Domain | Title | Exam Weight | Approx. Scored Questions | Core Focus Areas |
| :--- | :--- | :---: | :---: | :--- |
| **Domain 1** | **Design Solutions for Organizational Complexity** | **26%** | ~17 | Direct Connect (VIFs, DXGW, SiteLink, MACsec), BGP routing & traffic engineering, Transit Gateway (Inspection VPCs, Appliance Mode), Cloud WAN, PrivateLink vs VPC Lattice, Route 53 Hybrid DNS, AWS Organizations, SCPs, Control Tower, IAM Identity Center, ABAC, RAM, FinOps chargeback. |
| **Domain 2** | **Design for New Solutions** | **29%** | ~19 | CloudFront (OAC, Field-Level Encryption, Functions vs Lambda@Edge), Global Accelerator, WAF/Shield Advanced, KMS Multi-Region Keys, Multi-Region DR (Pilot Light, Warm Standby, Active-Active, Route 53 ARC), Aurora Global DB, DynamoDB Global Tables, S3 CRR/RTC/MRAP, AWS Backup Vault Lock, Kinesis/Firehose/Flink, Lake Formation TBAC, Redshift Data Sharing, EKS/ECS/Lambda/SQS FIFO/EventBridge/Step Functions. |
| **Domain 3** | **Continuous Improvement for Existing Solutions** | **25%** | ~16 | SSM (Session Manager, Patch Manager, Hybrid Activations), CloudFormation StackSets, Blue/Green & Canary deployments, CloudWatch Metric Streams, X-Ray, Config Auto-Remediation, IAM Roles Anywhere, Secrets Manager rotation, GuardDuty/Security Hub, Multi-tier caching (CloudFront, API GW, ElastiCache, DAX), Placement Groups & EFA, EBS `gp3`/`io2 Block Express`, EFS vs FSx, RDS Proxy, Savings Plans vs RIs vs Spot, S3 Intelligent-Tiering, NAT Gateway vs VPC Endpoint cost optimization. |
| **Domain 4** | **Accelerate Workload Migration and Modernization** | **20%** | ~13 | The 7 Rs framework, Migration Evaluator, Application Discovery Service (Agentless vs Agent), AWS MGN, VMware Cloud on AWS (HCX), License Manager, AWS DMS + SCT + Snowball Edge CDC, Babelfish for Aurora PostgreSQL, DataSync vs Snowball crossover math, AWS Transfer Family, Storage Gateway (S3 File, FSx File, Tape, Volume), Migration Hub Refactor Spaces (Strangler Fig), App2Container, AWS Mainframe Modernization (Blu Age vs Micro Focus). |

---

## 3. Workspace Navigation Directory

### Global Reference
* [**SAP-C02 Master Cheat Sheet (`SAP_C02_Master_Cheat_Sheet.md`)**](SAP_C02_Master_Cheat_Sheet.md) — High-density BGP/DX/TGW routing precedence tables, Multi-Account IAM evaluation logic, DR RTO/RPO matrices, Data Transfer crossover math, FinOps formulas, and Top 45 Exam Triggers & Traps.

### Domain 1: Design Solutions for Organizational Complexity (26%)
* [**Domain 1 Study Guide**](Domain_1_Organizational_Complexity/Study_Guide.md)
* [**Domain 1 White Papers & Architecture Guide**](Domain_1_Organizational_Complexity/White_Papers.md) — Deep dive into Hybrid Networking, Direct Connect, BGP, Transit Gateway, Hybrid DNS, AWS Organizations, SCPs, IAM Identity Center, and FinOps Governance.
* [**Domain 1 Architecture Exercises**](Domain_1_Organizational_Complexity/Exercises.md) — 6 hands-on enterprise case studies (Dual-Region DX/BGP failover, Centralized GWLB/Network Firewall Inspection VPC with Appliance Mode, Hybrid DNS + Centralized PrivateLink, VPC Lattice vs PrivateLink, Production SCPs, Federated ABAC + FinOps).
* [**Domain 1 Quiz 1: Hybrid Networking, Direct Connect, BGP & Transit Gateway**](Domain_1_Organizational_Complexity/Quiz_1.md) (15 Questions)
* [**Domain 1 Quiz 2: Hybrid DNS, PrivateLink, VPC Lattice & Overlapping CIDRs**](Domain_1_Organizational_Complexity/Quiz_2.md) (15 Questions)
* [**Domain 1 Quiz 3: Multi-Account Governance, SCPs, IAM Identity Center & RAM**](Domain_1_Organizational_Complexity/Quiz_3.md) (15 Questions)

### Domain 2: Design for New Solutions (29%)
* [**Domain 2 Study Guide**](Domain_2_Design_New_Solutions/Study_Guide.md)
* [**Domain 2 White Papers & Architecture Guide**](Domain_2_Design_New_Solutions/White_Papers.md) — Deep dive into Edge Security (CloudFront, Global Accelerator, WAF, Shield, KMS), Multi-Region DR & Business Continuity, Lakehouse/Streaming Data Architectures, and Serverless/Container Microservices.
* [**Domain 2 Architecture Exercises**](Domain_2_Design_New_Solutions/Exercises.md) — 6 hands-on enterprise case studies (Multi-Region Active-Active with ARC & Aurora Global DB, Zero-Trust Edge with Field-Level Encryption, Lake Formation Cross-Account Data Mesh, High-Throughput SQS FIFO + Step Functions, Immutable Ransomware Vault Lock, Multi-Tenant EKS + Karpenter).
* [**Domain 2 Quiz 1: Edge Architectures, Security, KMS & Identity**](Domain_2_Design_New_Solutions/Quiz_1.md) (15 Questions)
* [**Domain 2 Quiz 2: Business Continuity, Multi-Region DR & Resilient Databases**](Domain_2_Design_New_Solutions/Quiz_2.md) (15 Questions)
* [**Domain 2 Quiz 3: Data Lakehouse, Streaming, Containers & Event-Driven Compute**](Domain_2_Design_New_Solutions/Quiz_3.md) (15 Questions)

### Domain 3: Continuous Improvement for Existing Solutions (25%)
* [**Domain 3 Study Guide**](Domain_3_Continuous_Improvement/Study_Guide.md)
* [**Domain 3 White Papers & Architecture Guide**](Domain_3_Continuous_Improvement/White_Papers.md) — Deep dive into Operational Excellence (SSM, StackSets, Blue/Green, Observability), Security Remediation (Roles Anywhere, Secrets Rotation, GuardDuty), Performance Optimization (Caching, EFA, EBS/FSx), Reliability (RDS Proxy, Hot Partitions), and Cost Optimization.
* [**Domain 3 Architecture Exercises**](Domain_3_Continuous_Improvement/Exercises.md) — 6 hands-on enterprise case studies (Zero-Trust Fleet Patching & Roles Anywhere, Event-Driven Compromised EC2 Forensic Isolation, RDS Proxy + Multi-Tier Caching, HPC EFA + FSx for Lustre, Cloud FinOps Refactoring, RDS & ECS Blue/Green Pipelines).
* [**Domain 3 Quiz 1: Operational Excellence, Automation & Security Remediation**](Domain_3_Continuous_Improvement/Quiz_1.md) (15 Questions)
* [**Domain 3 Quiz 2: Performance Optimization & Reliability Engineering**](Domain_3_Continuous_Improvement/Quiz_2.md) (15 Questions)
* [**Domain 3 Quiz 3: FinOps, Purchasing Models, Storage Tiers & Network Cost Traps**](Domain_3_Continuous_Improvement/Quiz_3.md) (15 Questions)

### Domain 4: Accelerate Workload Migration and Modernization (20%)
* [**Domain 4 Study Guide**](Domain_4_Migration_Modernization/Study_Guide.md)
* [**Domain 4 White Papers & Architecture Guide**](Domain_4_Migration_Modernization/White_Papers.md) — Deep dive into the 7 Rs, Discovery & Wave Planning, AWS MGN & VMware HCX, DMS/SCT/Snowball Database Migrations, Large-Scale Storage Transfers (DataSync, Transfer Family, Storage Gateway, Snow Family), and Application Modernization (Refactor Spaces, App2Container, Mainframe Blu Age/Micro Focus).
* [**Domain 4 Architecture Exercises**](Domain_4_Migration_Modernization/Exercises.md) — 6 hands-on enterprise case studies (Discovery Dependency Mapping & MGN Wave Planning, 60 TB Oracle to Aurora via Snowball + DMS CDC, 500 TB NAS Migration via Snowball + DataSync + FSx, Strangler Fig with Refactor Spaces, Transfer Family Custom IdP + Mainframe Blu Age, App2Container + Babelfish).
* [**Domain 4 Quiz 1: 7 Rs, Discovery, Server Migration (MGN/HCX) & Licensing**](Domain_4_Migration_Modernization/Quiz_1.md) (15 Questions)
* [**Domain 4 Quiz 2: Database Migration (DMS/SCT) & Large-Scale Data Transfer**](Domain_4_Migration_Modernization/Quiz_2.md) (15 Questions)
* [**Domain 4 Quiz 3: Application Modernization, Refactor Spaces & Mainframes**](Domain_4_Migration_Modernization/Quiz_3.md) (15 Questions)

### Full-Length Practice Exams
* [**Practice Exams Guide (`Practice_Exams/README.md`)**](Practice_Exams/README.md) — Exam simulation guidelines, pacing strategies, and domain scoring rubric.
* [**Practice Exam 1 (`Practice_Exams/Practice_Exam_1.md`)**](Practice_Exams/Practice_Exam_1.md) — 25 full-length SAP-C02 scenario questions across all 4 domains.
* [**Practice Exam 2 (`Practice_Exams/Practice_Exam_2.md`)**](Practice_Exams/Practice_Exam_2.md) — 25 full-length SAP-C02 scenario questions across all 4 domains.
* [**Practice Exam 3 (`Practice_Exams/Practice_Exam_3.md`)**](Practice_Exams/Practice_Exam_3.md) — 25 full-length SAP-C02 scenario questions across all 4 domains.

---

## 4. SAP-C02 Exam Strategy & Time-Management Heuristics

1. **Read the Final Question Prompt First**: `SAP-C02` questions often contain 3–4 paragraphs of dense enterprise context. Before reading the narrative, jump to the final sentence to identify the **optimization constraint**:
   * *"Which solution meets these requirements with the LEAST operational overhead?"* > Eliminate custom scripts, EC2 cron jobs, self-managed proxies, and manual Lambda glue when a native managed service feature exists (e.g., S3 RTC, Firehose Dynamic Partitioning, EventBridge Pipes, DataSync, RDS Proxy).
   * *"Which solution is MOST cost-effective?"* > Watch for NAT Gateway data processing traps (use Gateway VPC Endpoints for S3/DynamoDB), S3 storage tiering (Intelligent-Tiering vs Glacier Deep Archive), or Spot/Savings Plans combinations.
   * *"Which solution achieves an RPO of seconds and RTO of under 1 minute?"* > Eliminate Backup & Restore or Pilot Light; look for Aurora Global Database, DynamoDB Global Tables, Route 53 ARC, or Multi-Region Active-Active.
2. **Enforce the 2-Minute Triage Rule**: You have 144 seconds per question. If a question has a massive wall of text and you are torn between two options after 90 seconds, select your best choice, **Flag for Review**, and keep moving. Never spend 6 minutes on a single question that might be one of the 10 unscored pretest items.
3. **Spot Subtle Technical Disqualifiers in Distractors**: Often 3 of the 4 options are 90% valid, except for a single architectural falsehood buried in the middle of the paragraph:
   * Using a **Direct Connect Public VIF** to directly attach to a VPC or Transit Gateway (requires Private VIF or Transit VIF; Public VIF only connects to public AWS endpoints, though you can establish a Site-to-Site VPN over a Public VIF).
   * Using **AWS Schema Conversion Tool (SCT)** for a homogeneous migration (e.g., Oracle to RDS Oracle) where native tools or DMS alone suffice.
   * Attaching **AWS WAF** directly to a Network Load Balancer (NLB) or S3 bucket (WAF only attaches to CloudFront, ALB, API Gateway, AppSync, Cognito User Pools, App Runner, and Verified Access).
   * Forgetting **Appliance Mode** (`ApplianceModeSupport: enable`) on a Transit Gateway VPC attachment when routing traffic through stateful firewalls (AWS Network Firewall / GWLB) across multiple AZs.
   * Assuming **Service Control Policies (SCPs)** grant permissions or affect the **Management Account** or **service-linked roles** (SCPs never grant permissions, never affect identities in the Management Account, and do not restrict service-linked roles).

---

## 5. 4-Week Study Plan & Progress Tracker

Use the interactive checklist below to track your completion across all study guides, architecture exercises, domain quizzes, and practice exams:

### Week 1: Organizational Complexity (Domain 1 - 26%)
- [ ] Read [Domain 1 Study Guide](Domain_1_Organizational_Complexity/Study_Guide.md)
- [ ] Review [Domain 1 White Papers & Documentation](Domain_1_Organizational_Complexity/White_Papers.md)
- [ ] Complete [Domain 1 Architecture Exercises (Exercises 1–6)](Domain_1_Organizational_Complexity/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 1 Quiz 1 (Hybrid Networking, DX, BGP & TGW)](Domain_1_Organizational_Complexity/Quiz_1.md)
- [ ] Score $\ge 80\%$ on [Domain 1 Quiz 2 (Hybrid DNS, PrivateLink & VPC Lattice)](Domain_1_Organizational_Complexity/Quiz_2.md)
- [ ] Score $\ge 80\%$ on [Domain 1 Quiz 3 (Multi-Account Governance, SCPs & IAM)](Domain_1_Organizational_Complexity/Quiz_3.md)

### Week 2: Design for New Solutions (Domain 2 - 29%)
- [ ] Read [Domain 2 Study Guide](Domain_2_Design_New_Solutions/Study_Guide.md)
- [ ] Review [Domain 2 White Papers & Documentation](Domain_2_Design_New_Solutions/White_Papers.md)
- [ ] Complete [Domain 2 Architecture Exercises (Exercises 1–6)](Domain_2_Design_New_Solutions/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 2 Quiz 1 (Edge Security, CloudFront, Global Accelerator & KMS)](Domain_2_Design_New_Solutions/Quiz_1.md)
- [ ] Score $\ge 80\%$ on [Domain 2 Quiz 2 (Business Continuity, DR, Aurora Global & ARC)](Domain_2_Design_New_Solutions/Quiz_2.md)
- [ ] Score $\ge 80\%$ on [Domain 2 Quiz 3 (Data Lakehouse, Streaming & Microservices)](Domain_2_Design_New_Solutions/Quiz_3.md)

### Week 3: Continuous Improvement for Existing Solutions (Domain 3 - 25%)
- [ ] Read [Domain 3 Study Guide](Domain_3_Continuous_Improvement/Study_Guide.md)
- [ ] Review [Domain 3 White Papers & Documentation](Domain_3_Continuous_Improvement/White_Papers.md)
- [ ] Complete [Domain 3 Architecture Exercises (Exercises 1–6)](Domain_3_Continuous_Improvement/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 3 Quiz 1 (Operational Excellence, SSM & Security Remediation)](Domain_3_Continuous_Improvement/Quiz_1.md)
- [ ] Score $\ge 80\%$ on [Domain 3 Quiz 2 (Performance Optimization & Reliability)](Domain_3_Continuous_Improvement/Quiz_2.md)
- [ ] Score $\ge 80\%$ on [Domain 3 Quiz 3 (FinOps, Savings Plans, S3 Tiers & Network Costs)](Domain_3_Continuous_Improvement/Quiz_3.md)

### Week 4: Migration, Modernization & Full Practice Exams (Domain 4 - 20% + Exams)
- [ ] Read [Domain 4 Study Guide](Domain_4_Migration_Modernization/Study_Guide.md)
- [ ] Review [Domain 4 White Papers & Documentation](Domain_4_Migration_Modernization/White_Papers.md)
- [ ] Complete [Domain 4 Architecture Exercises (Exercises 1–6)](Domain_4_Migration_Modernization/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 4 Quiz 1 (7 Rs, Discovery, MGN, VMware HCX & Licensing)](Domain_4_Migration_Modernization/Quiz_1.md)
- [ ] Score $\ge 80\%$ on [Domain 4 Quiz 2 (DMS, SCT, Snowball, DataSync & Storage Gateway)](Domain_4_Migration_Modernization/Quiz_2.md)
- [ ] Score $\ge 80\%$ on [Domain 4 Quiz 3 (Refactor Spaces, App2Container & Mainframe Modernization)](Domain_4_Migration_Modernization/Quiz_3.md)
- [ ] Review [SAP-C02 Master Cheat Sheet](SAP_C02_Master_Cheat_Sheet.md)
- [ ] Complete [Practice Exam 1 (Timed: 60 mins)](Practice_Exams/Practice_Exam_1.md) — Target: $\ge 20 / 25$ ($80\%$)
- [ ] Complete [Practice Exam 2 (Timed: 60 mins)](Practice_Exams/Practice_Exam_2.md) — Target: $\ge 20 / 25$ ($80\%$)
- [ ] Complete [Practice Exam 3 (Timed: 60 mins)](Practice_Exams/Practice_Exam_3.md) — Target: $\ge 20 / 25$ ($80\%$)
