# Domain 7: HCP Terraform & Terraform Enterprise Administration — Reference White Papers & Architectural Deep Dives

## 1. Terraform Stacks: Multi-Component & Multi-Deployment Orchestration
* **Primary Reference:** *HashiCorp Architecture Guide — Managing Multi-Region & Multi-Environment Infrastructure with Terraform Stacks*
* **Core Architectural Takeaways:**
  * Traditional Terraform workspaces couple 1 root configuration + 1 state file + static provider blocks, forcing platform engineers to duplicate directory trees (`envs/dev/us-east-1`, `envs/prod/us-east-1`, `envs/prod/us-west-2`) or use external wrappers like Terragrunt.
  * Terraform Stacks (`*.tfstack.hcl` & `*.tfdeploy.hcl`) elevate multi-region provider iteration (`for_each` on `provider` blocks) and inter-component dependency graphs into first-class native language constructs evaluated by HCP Terraform.

---

## 2. Terraform Enterprise (TFE) Flexible Deployment Options (FDO) Reference Architecture
* **Primary Reference:** *Terraform Enterprise Production Reference Architecture — Active-Active High Availability & Disaster Recovery*
* **Core Architectural Takeaways:**
  * Active-Active TFE separates stateless compute (2+ TFE containers behind an L7 load balancer) from stateful persistence (External PostgreSQL, External Redis, and External S3/GCS/Azure Blob Storage).
  * Recovery Point Objective (RPO) and Recovery Time Objective (RTO) depend on continuous PostgreSQL point-in-time recovery (PITR), cross-region object storage replication, and secure backup of the `TFE_ENCRYPTION_PASSWORD`.
