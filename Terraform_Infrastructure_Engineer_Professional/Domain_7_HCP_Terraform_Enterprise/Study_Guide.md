# Domain 7: HCP Terraform & Terraform Enterprise Administration — Complete Study Guide

**Exam Weight:** 10% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** HCP Terraform Hierarchy (Organizations, Projects, Workspaces, Teams & RBAC inheritance), Variable Sets & Variable Precedence, Workspace Execution Modes (Remote, Local, Self-Hosted Agent Pools), Cross-Workspace Orchestration (Run Triggers vs. `tfe_outputs`), **Terraform Stacks (`*.tfstack.hcl` & `*.tfdeploy.hcl`, `component`, `deployment`, `orchestrate` rules)**, Continuous Drift & Health Assessments, Private Registry & No-Code Provisioning, and **Terraform Enterprise (TFE) Flexible Deployment Options (FDO)** with External Services (PostgreSQL, Redis, S3/Blob Object Storage) & Active-Active High Availability.

---

## 1. HCP Terraform Resource Hierarchy, RBAC & Variable Precedence

```text
+-----------------------------------------------------------------------------------+
|            HCP TERRAFORM ORGANIZATIONAL HIERARCHY & VARIABLE PRECEDENCE           |
+-----------------------------------------------------------------------------------+
  [Organization (e.g. "corp-cloud-platform")]
    |-- Global Variable Sets (Lowest Precedence)
    |-- Policy Sets (Sentinel / OPA Rego) & Private Registry
    |-- Agent Pools (Self-Hosted VPC Runners)
    |
    +---> [Project: "Payments-Prod"] (Team RBAC: Read / Plan / Write / Admin)
    |       |-- Project-Scoped Variable Sets (Overrides Global Variable Sets!)
    |       |
    |       +---> [Workspace: "payments-networking-use1"]
    |       |       |-- Workspace-Scoped Variable Sets (Overrides Project Sets!)
    |       |       +-- Workspace Variables (Highest Precedence inside HCP TF!)
    |       |
    |       +---> [Stack: "payments-multi-region-stack"] (*.tfstack.hcl)
```

### 1.1 Variable Set Precedence Order (Lowest $\rightarrow$ Highest Winner)
When a variable with the same key (e.g., `aws_region` or `TF_VAR_log_level`) is defined at multiple levels in HCP Terraform, the **more specific scope always wins**:
1. **Global Variable Set** (applied to all workspaces in the Organization) — *Lowest Precedence*
2. **Project-Scoped Variable Set** (assigned to specific Projects)
3. **Workspace-Scoped Variable Set** (assigned to specific Workspaces)
4. **Workspace Variable** (defined directly on the individual Workspace) — *Highest Precedence in HCP UI*
*(Note: Setting the **"Priority"** flag on a Variable Set reverses this behavior, forcing that Priority Variable Set to override non-priority Project/Workspace variables—used by Security teams to lock mandatory OIDC or compliance variables!)*

### 1.2 Workspace Execution Modes
Each HCP Terraform workspace operates in one of three execution modes:
* **1. Remote Execution Mode (Default):**
  * Plans and applies execute on HashiCorp-managed multi-tenant cloud VMs.
  * Supports VCS-driven workflows, Policy-as-Code (Sentinel/OPA), Cost Estimation, and Health Assessments.
* **2. Agent Execution Mode (Self-Hosted `tfc-agent` Pools):**
  * Plans and applies execute on **self-hosted `tfc-agent` containers/VMs** deployed inside your private AWS VPC, Kubernetes cluster, or on-premises data center.
  * **Network Security Advantage:** `tfc-agent` initiates **outbound-only HTTPS (port 443)** polling to HCP Terraform (`app.terraform.io`). You **never** need to open inbound firewall ports into your private network to manage internal RDS databases, private EKS API servers, or internal vSphere clusters!
* **3. Local Execution Mode:**
  * Plans and applies execute entirely on the engineer's local workstation or external CI runner; HCP Terraform is used **strictly as a remote state storage and locking backend** (no remote runs, no Sentinel/OPA policy checks, no cost estimation).

---

## 2. Terraform Stacks (`*.tfstack.hcl` & `*.tfdeploy.hcl`)

Managing multi-region or multi-tier infrastructure across dozens of separate HCP Terraform workspaces linked by Run Triggers requires manual coordination and struggles with circular dependencies. **Terraform Stacks** natively orchestrate multiple coupled infrastructure **components** across multiple **deployments** (environments/regions) as a single cohesive unit!

```text
+-----------------------------------------------------------------------------------+
|               TERRAFORM STACKS ARCHITECTURE (*.tfstack.hcl & *.tfdeploy.hcl)      |
+-----------------------------------------------------------------------------------+
  [Stack Definition (components.tfstack.hcl + providers.tfstack.hcl)]
    |-- component "vpc" { source = "./modules/vpc" ... }
    |-- component "eks" { source = "./modules/eks", inputs = { vpc_id = component.vpc.id } }
    +-- component "app" { source = "./modules/k8s_app", for_each = var.regions ... }
                                      |
                                      v
  [Deployment Configurations (deployments.tfdeploy.hcl)]
    |-- deployment "staging_us_east_1"    (inputs: regions = ["us-east-1"])
    |-- deployment "prod_multi_region"    (inputs: regions = ["us-east-1", "us-west-2"])
    +-- orchestrate "auto_approve" "safe_staging" {
          check { condition = context.plan.deployment == deployment.staging_us_east_1 }
          check { condition = context.plan.changes.remove == 0 }
        }
```

### 2.1 Core Building Blocks of Terraform Stacks
* **Component (`*.tfstack.hcl`):** Declares a reusable Terraform module (`component "<name>" { source = "..." inputs = { ... } providers = { ... } }`) within the stack. Unlike child modules in a standard root configuration, each component in a Stack has its **own isolated lifecycle and state segment**, and **providers can be iterated with `for_each`** at the stack level (`provider "aws" "by_region" { for_each = var.regions ... }`)!
  * **Exam Breakthrough:** In standard Terraform `.tf` files, you *cannot* put `for_each` on a `provider` block. In **Terraform Stacks (`*.tfstack.hcl`)**, `provider` blocks **natively support `for_each`** so you can dynamically instantiate providers across an arbitrary list of regions or accounts!
* **Deployment (`*.tfdeploy.hcl`):** Instantiates the stack for a specific environment or account (`deployment "production" { inputs = { ... } }`), bindings OIDC identity tokens (`identity_token "aws" { audience = ["aws.workload.identity"] }`) per deployment.
* **Orchestration Rules (`orchestrate` block in `*.tfdeploy.hcl`):** Defines automated governance rules (`orchestrate "auto_approve" "<name>"` or `orchestrate "replan"`) using `context.plan.*` assertions (e.g., auto-approving plans in `dev` if `context.plan.changes.remove == 0`).

---

## 3. Terraform Enterprise (TFE) Flexible Deployment Options (FDO) & High Availability

For enterprises requiring self-hosted control planes, **Terraform Enterprise (TFE)** runs via **Flexible Deployment Options (FDO)** on **Kubernetes (EKS/GKE/AKS/OpenShift)**, **Docker**, **Podman**, or **Nomad**.

```text
+-----------------------------------------------------------------------------------+
|         TERRAFORM ENTERPRISE (TFE) ACTIVE-ACTIVE PRODUCTION ARCHITECTURE          |
+-----------------------------------------------------------------------------------+
                     [External L4/L7 HTTPS Load Balancer]
                                      |
              +-----------------------+-----------------------+
              |                                               |
              v                                               v
  [TFE Node / Pod 1 (Stateless)]                  [TFE Node / Pod 2 (Stateless)]
              |                                               |
              +-----------------------+-----------------------+
                                      |
        +-----------------------------+-----------------------------+
        |                             |                             |
        v                             v                             v
  [External PostgreSQL]       [External Redis Cluster]    [External Object Storage]
  (RDS Aurora / Cloud SQL)    (ElastiCache / Memorystore) (AWS S3 / GCS / Azure Blob)
  * Stores relational metadata * Coordinates Active-Active * Stores encrypted .tfstate,
    workspaces, RBAC, runs      job queues & pub/sub cache  plan files & provider cache
```

### 3.1 TFE Operational Modes: Mounted Disk vs. External Services vs. Active-Active
* **Mounted Disk Mode (PoC / Dev / Testing Only):**
  * Runs PostgreSQL, Redis, and blob storage inside containers on a single host writing to a local directory volume. Cannot scale horizontally to multiple nodes.
* **External Services Mode (Single-Node Production):**
  * Connects a single TFE instance to managed external **PostgreSQL**, **External Object Storage** (S3 / Azure Blob / GCS), and (optionally) external Redis.
* **Active-Active Mode (Enterprise High Availability — Recommended for Production):**
  * Runs **2 or more stateless TFE nodes/pods** behind an L7 HTTPS Load Balancer.
  * **Mandatory External Dependencies for Active-Active:**
    1. **External PostgreSQL Database** (e.g., AWS RDS Multi-AZ)
    2. **External Object Storage** (e.g., AWS S3, Azure Blob, GCS, or MinIO)
    3. **External Redis** (mandatory in Active-Active to synchronize run queues, locks, and cache across nodes!)
    4. **Consistent Encryption Password (`TFE_ENCRYPTION_PASSWORD`):** Must be identical across all TFE nodes to derive the root Vault key that encrypts data in PostgreSQL and Object Storage.

### 3.2 TFE Backup & Disaster Recovery (`tfe-admin` / External Snapshots)
In Production External Services / Active-Active deployments, disaster recovery relies on backing up the **three stateful persistence tiers** plus the encryption secret:
1. **`TFE_ENCRYPTION_PASSWORD`** (Without this secret, backed-up database and S3 blobs are cryptographically unrecoverable!).
2. **PostgreSQL Database Snapshot / `pg_dump`**
3. **Object Storage Bucket Replication / Versioning**

---

## 4. Continuous Drift Detection, Health Assessments & No-Code Provisioning

* **HCP Terraform Health Assessments:**
  * Periodically runs background **`terraform plan -refresh-only`** against workspaces to detect **Infrastructure Drift** and evaluates **Continuous Validation (`check` blocks, preconditions, postconditions)** on a schedule without requiring a Git commit!
* **No-Code Provisioning (Private Registry):**
  * Platform teams mark vetted Private Registry modules as **"No-Code Ready"** and define variable option lists. Application developers can then self-service provision a dedicated workspace and infrastructure directly from the HCP Terraform UI or API **without writing any HCL code or setting up a Git repository**!

---

## 5. Domain 7 Production Scenario & Exam Trap Matrix

* **Scenario 1: Managing resources inside a private VPC with zero inbound internet access from HCP Terraform**
  * **Trap:** Whitelisting HCP Terraform's public IP CIDR ranges on your private VPC firewall and exposing the internal Kubernetes/database API publicly.
  * **Reality:** Deploy a self-hosted **HCP Terraform Agent (`tfc-agent`)** inside the private VPC and switch the workspace Execution Mode to **Agent**. The agent initiates outbound-only HTTPS (443) connections to `app.terraform.io`.
* **Scenario 2: Iterating a `provider "aws"` block across a dynamic list of regions**
  * **Trap:** Attempting to write `provider "aws" { for_each = var.regions ... }` inside a standard `.tf` root module.
  * **Reality:** Standard `.tf` files forbid `for_each` on `provider` blocks. Use **Terraform Stacks (`*.tfstack.hcl`)**, which natively support `for_each` on `provider` blocks and `component` blocks across multi-region deployments!
* **Scenario 3: Scaling Terraform Enterprise from 1 node to 2 Active-Active nodes**
  * **Trap:** Attempting to run Active-Active TFE using Mounted Disk storage or without External Redis.
  * **Reality:** Active-Active TFE strictly requires **External PostgreSQL + External Object Storage + External Redis** plus a shared `TFE_ENCRYPTION_PASSWORD` behind an L7 load balancer.

---

## 6. Platform Engineering Deep Dive: "Terraform-for-Terraform" (`tfe` Provider Factory)

Platform Engineering teams never click manually in the HCP Terraform / Terraform Enterprise UI to create hundreds of workspaces, variable sets, and RBAC bindings. Instead, they manage HCP Terraform itself as code using the official **`hashicorp/tfe` provider** inside an administrative **"Workspace Factory" (Bootstrap) Workspace**:

```hcl
# Admin Workspace: Provisions Projects, Workspaces, Variable Sets & OIDC bindings
terraform {
  required_providers {
    tfe = {
      source  = "hashicorp/tfe"
      version = "~> 0.60.0"
    }
  }
}

resource "tfe_project" "platform_services" {
  organization = "corp-cloud-platform"
  name         = "Platform-Core-Services"
}

resource "tfe_workspace" "eks_clusters" {
  for_each = toset(["dev", "staging", "prod"])

  organization      = "corp-cloud-platform"
  project_id        = tfe_project.platform_services.id
  name              = "eks-cluster-${each.key}"
  terraform_version = "~> 1.11.0"
  execution_mode    = "agent"
  agent_pool_id     = data.tfe_agent_pool.private_vpc.id
  assessments_enabled = true # Enables continuous drift detection & health checks!

  vcs_repo {
    identifier     = "corp-org/infra-eks-clusters"
    branch         = each.key == "prod" ? "main" : each.key
    oauth_token_id = var.vcs_oauth_token_id
  }
}

# Cross-Workspace Run Trigger: Automatically queue a plan in `eks-cluster-prod`
# whenever `networking-prod` successfully completes an apply!
resource "tfe_run_trigger" "network_to_eks_prod" {
  workspace_id  = tfe_workspace.eks_clusters["prod"].id
  sourceable_id = data.tfe_workspace.networking_prod.id
}
```

---

## 7. Linked Terraform Stacks (`publish_output` & `upstream_input`) & API-Driven Runs

### 7.1 Cross-Stack Wiring (`publish_output` and `upstream_input`)
When an enterprise decomposes infrastructure across multiple **Terraform Stacks** (e.g., a central `network-stack` and downstream `eks-platform-stack`), HCP Terraform links them natively without `terraform_remote_state`:
1. **In the Producer Stack (`deployments.tfdeploy.hcl`):** Use **`publish_output`** to export a strongly typed value from a specific deployment:
   ```hcl
   publish_output "prod_vpc_ids" {
     description = "Map of regional VPC IDs exported from the production network deployment"
     value       = deployment.production.vpc_ids
   }
   ```
2. **In the Consumer Stack (`deployments.tfdeploy.hcl`):** Use **`upstream_input`** to subscribe to the producer stack's published output (which also automatically triggers downstream stack replans when the upstream output changes!):
   ```hcl
   upstream_input "network" {
     type   = "stack"
     source = "app.terraform.io/corp-cloud-platform/core-services/network-stack"
   }

   deployment "production_eks" {
     inputs = {
       regional_vpc_ids = upstream_input.network.prod_vpc_ids
     }
   }
   ```

### 7.2 API-Driven Run Workflow (Custom Internal Developer Portals)
When integrating HCP Terraform with a custom Internal Developer Portal (IDP) or custom CI orchestrator that does not use direct VCS webhooks or the Terraform CLI, use the **API-Driven Run Workflow**:
1. **`POST /api/v2/workspaces/:workspace_id/configuration-versions`:** Creates a new `configuration-version` object (`auto-queue-runs: true` or `false`) and returns a signed `upload-url`.
2. **`PUT <upload-url>`:** Uploads the packaged `.tar.gz` archive containing the `.tf` files directly to HCP Terraform object storage.
3. **`POST /api/v2/runs`:** Triggers the remote plan/apply execution against the uploaded configuration version and streams logs/policy results back to the IDP via webhooks or polling.

