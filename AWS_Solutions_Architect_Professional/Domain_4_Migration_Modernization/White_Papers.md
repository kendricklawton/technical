# Domain 4: Accelerate Workload Migration and Modernization — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 20% of AWS Certified Solutions Architect - Professional (SAP-C02)  
> **Core Objective:** Plan, execute, and validate large-scale data center migrations and application modernizations to AWS using the 7 Rs framework, native migration tooling (AWS MGN, DMS, SCT, DataSync), and automated containerization.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [AWS Cloud Adoption Framework (CAF) & The 7 Rs](https://docs.aws.amazon.com/whitepapers/latest/overview-aws-cloud-adoption-framework/welcome.html)
* **Canonical Scope:** Structured enterprise framework for cloud transformation across Business, People, Governance, Platform, Security, and Operations perspectives.
* **The 7 Rs Migration Strategies:**
  | Strategy | Definition | Typical Effort | Value Realization |
  | :--- | :--- | :--- | :--- |
  | **Rehost ("Lift and Shift")** | Move application as-is without architectural change (AWS MGN). | Lowest | Fast exit from data center |
  | **Replatform ("Lift, Tinker, and Shift")** | Move to managed services without core code rewrite (e.g., EC2 to RDS or ECS).| Low to Medium | Reduces operational overhead |
  | **Refactor / Re-architect** | Re-imagine architecture using cloud-native serverless and microservices. | Highest | Maximum agility, scalability, cost |
  | **Repurchase ("Drop and Shop")** | Replace existing system with commercial SaaS product (e.g., Salesforce).| Variable | Instant modern capability |
  | **Relocate** | Move virtual machines directly without new hardware (VMware Cloud on AWS). | Very Low | Seamless hypervisor migration |
  | **Retain** | Keep applications on-premises due to compliance or deprecation timelines. | Zero | Avoids unnecessary migration cost |
  | **Retire** | Decommission applications no longer delivering business value. | Zero | Immediate cost elimination |

### 1.2 [Database Migration — How to Migrate from Commercial DBs to Open Source / Aurora](https://docs.aws.amazon.com/whitepapers/latest/database-migration-open-source-aurora/welcome.html)
* **Canonical Scope:** Architectural methodology for migrating proprietary databases (Oracle, Microsoft SQL Server) to open-source engines (PostgreSQL, MySQL, Aurora).
* **Two-Phase Migration Pattern:**
  1. **AWS Schema Conversion Tool (SCT):** Automated conversion of schemas, stored procedures, triggers, and views. Flags unconvertible code for manual refactoring.
  2. **AWS Database Migration Service (DMS):** Full initial load combined with Continuous Replication (Change Data Capture - CDC) to keep source and target databases in sync until cutover.

### 1.3 [Decomposing Monoliths to Microservices with AWS Migration Hub Refactor Spaces](https://docs.aws.amazon.com/migrationhub-refactor-spaces/latest/userguide/what-is-refactor-spaces.html)
* **Canonical Scope:** Implementing the Strangler Fig pattern safely in multi-account environments.
* **Key Architecture:**
  * Refactor Spaces automates cross-account networking (Transit Gateway), API Gateway routing, and Network Load Balancers to transparently route microservice traffic away from the monolith to new containerized/serverless endpoints without breaking client contracts.

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [AWS Application Migration Service (AWS MGN)](https://docs.aws.amazon.com/mgn/latest/ug/)
* **Deep Dive:**
  * **Replication Architecture:** AWS Replication Agent installed on source servers (physical, virtual, or other clouds); continuously replicates block-level data over TLS to lightweight staging area in AWS.
  * **Non-Disruptive Testing:** Launching test EC2 instances into isolated subnets without interrupting ongoing continuous replication.
  * **Cutover Sequence:** Final data sync -> Shut down source server -> Launch production cutover instances -> Validate application.

### 2.2 [AWS Database Migration Service (DMS) & SCT](https://docs.aws.amazon.com/dms/latest/userguide/)
* **Deep Dive:**
  * Homogeneous migrations (Oracle to Oracle, Postgres to Postgres) vs Heterogeneous migrations (Oracle to Aurora PostgreSQL).
  * SCT with Snowball Edge: Offloading initial multiterabyte database dumps to a physical Snowball appliance, extracting data with SCT, shipping to AWS, and using DMS CDC over VPN/DX for delta synchronization.

### 2.3 [Large-Scale Data Transfer: AWS DataSync & Snow Family](https://docs.aws.amazon.com/datasync/latest/userguide/)
* **Deep Dive:**
  * **AWS DataSync:** Purpose-built agent for accelerating online file transfers (NFS, SMB, HDFS, S3) up to 10x faster than open-source tools with automated integrity verification and bandwidth throttling.
  * **Snowball Edge & Snowmobile:** Offline data transfer appliances (80 TB Snowball Edge Storage Optimized; petabyte-scale Snowmobile) for offline bulk migration when network bandwidth is constrained.

### 2.4 [AWS Mainframe Modernization & App2Container](https://docs.aws.amazon.com/m2/latest/ug/)
* **Deep Dive:**
  * **AWS Mainframe Modernization:** Automated refactoring via AWS Blu Age (converting COBOL to Java) or replatforming via Micro Focus.
  * **AWS App2Container:** CLI tool that analyzes Java and .NET web applications running on VMs and standardizes them into Docker container images ready for Amazon ECS or EKS.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **Large Database Migration with Limited Bandwidth (60 TB Oracle):**
   * Migrating over a saturated 100 Mbps connection would take months and choke business traffic.
   * *Resolution:* Export initial database backup using **AWS SCT + Snowball Edge**; import into Amazon Aurora in AWS; simultaneously start **AWS DMS with CDC** to capture transaction logs over the network until cutover.
2. **NAS Migration with Millions of Small Files (500 TB):**
   * Standard `rsync` or S3 CLI experiences extreme performance degradation on millions of small metadata transactions.
   * *Resolution:* Deploy **AWS DataSync** agents on-premises to parallelize transfers, verify file hashes, and transfer directly to Amazon FSx or S3.
3. **Application Modernization without Client Disruption:**
   * Progressively migrating features from an on-premises monolith to AWS microservices without client API changes.
   * *Resolution:* Use **AWS Migration Hub Refactor Spaces** to manage the Strangler Fig pattern with central API Gateway and Transit Gateway routing.

---

## 4. Reading & Study Checklist

- [ ] Read *AWS Cloud Adoption Framework (CAF)*
- [ ] Read *Database Migration — How to Migrate from Commercial DBs to Open Source / Aurora*
- [ ] Review AWS Application Migration Service (MGN) lifecycle and staging architecture
- [ ] Master AWS DMS full load + CDC synchronization mechanics
- [ ] Study AWS DataSync vs Snowball Edge decision boundaries for bulk data transfer
- [ ] Review AWS Migration Hub Refactor Spaces multi-account topology
