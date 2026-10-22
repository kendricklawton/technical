# Domain 4: Analyzing and Optimizing Technical and Business Processes — Study Guide

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Weight:** ~19% of the certification curriculum

---

## 1. Cloud FinOps & Cost Optimization
- **Committed Use Discounts (CUDs):**
  - **Resource-Based CUDs:** 1-year or 3-year commitment for predictable compute resources (vCPUs, RAM, GPUs, Local SSDs) in a specific region and machine family (e.g. N2 in `us-central1`). Up to 57% savings.
  - **Spend-Based CUDs:** 1-year or 3-year dollar-per-hour spend commitment across multiple services and regions (Compute Engine, Cloud SQL, Cloud Run, GKE, Spanner). Up to 52% savings. Highly flexible for evolving architectures.
- **Spot VMs:**
  - Excess Compute Engine capacity offered at 60% to 91% discount off standard on-demand pricing.
  - Compute Engine can preempt instances with a 30-second ACPI shutdown notice.
  - Optimal for fault-tolerant, stateless, batch, HPC, and CI/CD workloads.
- **Storage Lifecycle Optimization:**
  - Automated transition between Cloud Storage classes (Standard -> Nearline -> Coldline -> Archive) based on object age (`age: 30`, `age: 90`, `age: 365`).
  - Delete non-current object versions after $N$ days to manage versioning storage costs.
- **BigQuery Pricing & Optimization:**
  - **On-Demand:** $6.25 per TB of data scanned. Cost managed via partition filters, clustering, and `maximum_bytes_billed` limits.
  - **BigQuery Editions (Standard, Enterprise, Enterprise Plus):** Slot-based compute reservations (baseline + autoscaling slots) for predictable enterprise budgets.
- **Active Assist Recommenders:**
  - Cost Recommender, VM Sizing Recommender (downsizes over-provisioned VMs), IAM Recommender (revokes unused permissions), and Unattended Project Recommender.

---

## 2. Enterprise Cloud Migration Framework (The 6 Rs)
- **Rehost (Lift and Shift):** Migrate workloads with minimal changes using **Migrate to Virtual Machines**.
- **Replatform (Lift and Optimize):** Move to managed platforms without rewriting application core (e.g. self-managed MySQL to Cloud SQL; bare metal containers to GKE).
- **Refactor / Re-architect:** Redesign applications into cloud-native microservices (Cloud Run, Cloud Spanner, Pub/Sub).
- **Repurchase (Drop and Shop):** Replace custom systems with SaaS solutions.
- **Retain:** Keep legacy systems on-premises due to compliance or technical constraints.
- **Retire:** Decommission redundant applications.

---

## 3. Data Migration Services & Tooling
- **Storage Transfer Service (STS):**
  - High-performance, scalable online data transfer from Amazon S3, Azure Blob, HTTP sources, or on-premises POSIX file systems into Cloud Storage.
  - Supports automated scheduling, periodic synchronization, and bandwidth throttling.
- **Transfer Appliance:**
  - Ruggedized physical high-capacity hardware appliance (40 TB or 300 TB usable) shipped to customer data center for offline data migration when network bandwidth is insufficient (>100 TB with <1 Gbps uplink).
- **Database Migration Service (DMS):**
  - Serverless, high-fidelity migration for MySQL, PostgreSQL, Oracle, and SQL Server to Cloud SQL and AlloyDB with minimal downtime via continuous Change Data Capture (CDC).
- **Migrate to Virtual Machines:**
  - Continuous data replication from on-premises VMware / AWS EC2 into Compute Engine with built-in pre-migration validation, non-disruptive test clones, and cutover orchestration.

---

## 4. Software Development Lifecycle (SDLC) & Release Velocity
- **Continuous Integration / Continuous Delivery (CI/CD):**
  - Cloud Build, Artifact Registry, Google Cloud Deploy.
  - Deployment strategies: Canary (traffic splitting with automated verification), Blue-Green (instant zero-downtime cutover), Shadow / Traffic Mirroring (testing live traffic with zero user impact).
- **Drift Detection & GitOps:**
  - Infrastructure declared in Git; Config Sync / Terraform CI automatically detects and remediates out-of-band manual drift.
