# Domain 4: Infrastructure as Code — Essential Technical Documentation & Guide

> **Core Focus:** Terraform / OpenTofu Enterprise Patterns, State Management, Refactoring (`moved` blocks), Module Design, Policy as Code (OPA / Trivy), Ephemeral Environments, Crossplane, and Multi-Account Landing Zones.

---

## 1. Authoritative Documentation & Frameworks

### 1.0 [Terraform Infrastructure Engineer Professional (002) Master Workspace](../../Terraform_Infrastructure_Engineer_Professional/README.md)
* **Exam Scope & Blueprint:** 7 full domains covering Terraform v1.15+, ephemeral variables, write-only arguments, `terraform test` unit/integration suites, OIDC dynamic provider credentials, and HCP Terraform governance. Includes 2 full 60-question practice exams.

### 1.1 [Terraform / OpenTofu Module Design & Best Practices](https://opentofu.org/docs/)
* **Canonical Scope:** Writing composable, enterprise-ready infrastructure modules.
* **Design Principles:**
  * **Input Simplicity & Output Generosity:** Expose sane defaults while exporting all created resource IDs and ARNs for downstream consumption.
  * **Decoupling State from Orchestration:** Isolating state files by blast radius (Foundations -> Networking -> Workloads) to prevent monolithic state lockouts.
  * **Refactoring without Destruction:** Utilizing `moved` blocks to rename or relocate resources within modules without triggering destructive deletes and recreations.

### 1.2 [Crossplane: Composable Cloud Control Planes](https://docs.crossplane.io/)
* **Canonical Scope:** Extending the Kubernetes API to manage external cloud resources.
* **Core Architecture:**
  * `Managed Resource` (MR): Concrete external API representation (e.g. AWS S3 Bucket, RDS Instance).
  * `Composite Resource Definition` (XRD): Defining custom self-service infrastructure APIs tailored for internal developers.
  * `Composition`: Binding the XRD to actual underlying cloud resources across multiple cloud providers.

### 1.3 [Policy as Code with Open Policy Agent (OPA) & Rego](https://www.openpolicyagent.org/docs/)
* **Canonical Scope:** Automated guardrail enforcement during CI pull request pipelines.
* **Key Patterns:**
  * Evaluating `terraform plan -out=plan.json` against Rego policies (Conftest) to block public S3 buckets, unapproved instance types, or unencrypted storage before `terraform apply`.

---

## 2. Enterprise State & Landing Zone Patterns

### 2.1 Remote State Management & Locking
* **Architecture:** Amazon S3 + DynamoDB table (or native OpenTofu state backend with S3 conditional writes) providing atomic state locking and AES-256 encryption.
* **State Isolation:** Never manage Networking, Kubernetes clusters, and App deployments in a single state file.

### 2.2 Account Vending Machines (AVMs)
* Automating the creation of isolated AWS accounts / GCP projects on-demand for engineering teams with pre-configured network attachments, guardrails, and billing alarms.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"Terraform vs Crossplane in Modern IDPs":**
   * *Comparison:* Terraform is GitOps-driven, batch-executed, and well-understood, but struggles with continuous drift reconciliation and dynamic self-service developer portals. Crossplane runs a continuous reconciliation loop inside Kubernetes, enabling developers to request infrastructure via native `kubectl apply` manifests.
   * *Staff Recommendation:* Hybrid model—Terraform for base clusters and networking; Crossplane for developer self-service application dependencies (databases, queues).
2. **Handling Large-Scale Refactoring:**
   * Renaming resources inside production modules without causing downtime.
   * *Resolution:* Code declarative `moved { from = aws_s3_bucket.old to = aws_s3_bucket.new }` blocks in OpenTofu/Terraform; never run dangerous manual `state mv` commands on shared production state files.

---

## 4. Reading & Study Checklist

- [ ] Read *OpenTofu / Terraform Enterprise Module Design Guide*
- [ ] Read *Crossplane Core Concepts & Composition Architecture*
- [ ] Practice writing declarative `moved` blocks for zero-downtime refactors
- [ ] Write a Conftest / OPA Rego policy to audit Terraform JSON plans
- [ ] Review multi-account landing zone state isolation strategies
