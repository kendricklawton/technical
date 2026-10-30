# Domain 2: State and Backend Operations — Complete Study Guide

**Exam Weight:** 18% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** Internal `.tfstate` JSON schema (`version`, `serial`, `lineage`, `check_results`), Remote Backend architectures, S3 Native State Locking (`use_lockfile = true` via S3 conditional writes `.tflock` in Terraform 1.10+) vs. legacy DynamoDB locking, Partial Backend Configurations (`-backend-config`), Backend Migration (`-migrate-state` vs. `-reconfigure`), Blast-Radius Segmentation, Cross-Stack Data Sharing (`terraform_remote_state` vs. `tfe_outputs` vs. Cloud Data Source/SSM Decoupling), Safe Drift Reconciliation (`-refresh-only` & `-detailed-exitcode`), and State Disaster Recovery (`force-unlock`, `state pull/push`, `state replace-provider`).

---

## 1. Internal Anatomy of `terraform.tfstate` & Concurrency Invariants

Terraform state is the single source of truth mapping declarative HCL resource addresses (`aws_vpc.main`) to real-world cloud provider physical IDs (`vpc-0a1b2c3d4e5f6g7h8`), tracking metadata, provider dependency graphs, and cached attribute values.

```text
+-----------------------------------------------------------------------------------+
|            DISTRIBUTED STATE LOCKING & CONCURRENCY LIFECYCLE (v1.10+)             |
+-----------------------------------------------------------------------------------+
  [Engineer / CI Runner 1]                        [Engineer / CI Runner 2]
             |                                               |
  1. PUT <key>.tflock                             2. PUT <key>.tflock
     (Header: If-None-Match: "*")                    (Header: If-None-Match: "*")
             |                                               |
             v                                               v
  [S3 Lock Acquired: 200 OK]                      [HTTP 412 Precondition Failed!]
             |                                    (Error acquiring state lock:
  3. GET <key>.tfstate                             Lock Info: ID, Who, Operation)
  4. Refresh Cloud APIs & Compute DAG Plan
  5. Execute Apply & Increment `serial` (+1)
  6. PUT <key>.tfstate (Verify `lineage` matches)
  7. DELETE <key>.tflock (Release Lock)
```

### 1.1 Core Top-Level Fields in `.tfstate` JSON
When you inspect raw state via `terraform state pull`, the JSON document contains five critical top-level control fields:
* **`version` (Integer, currently `4`):** Defines the state file schema format version. Terraform refuses to read a state file whose format version is newer than supported by the running binary.
* **`terraform_version` (String):** Records the exact Terraform CLI version that last wrote the state file. By default, Terraform blocks operations if the state was written by a newer minor/major release unless state format compatibility allows it.
* **`serial` (Monotonically Increasing Integer):** Increments by `+1` on every state write (`apply`, `state mv`, `state rm`, `state push`). Before overwriting remote state, Terraform checks that the remote `serial` has not advanced since the run started, preventing lost updates.
* **`lineage` (UUID String):** Generated once when a state file is first initialized (`terraform init`). It uniquely identifies the infrastructure lifecycle history.
  * **Safety Invariant:** `terraform state push` will **refuse** to overwrite a remote state file if either:
    1. The incoming state has a **different `lineage` UUID** than the existing remote state, OR
    2. The incoming state has a **lower or equal `serial` number** than the existing remote state.
  * Overriding either protection requires explicitly passing `terraform state push -force <file>`, which should only be used during catastrophic state recovery.
* **`check_results` (Array):** Tracks the latest pass/fail status of continuous validation variables, preconditions, postconditions, and `check` blocks.

---

## 2. Remote Backends & S3 Native State Locking (Terraform 1.10+)

### 2.1 S3 Native Lockfiles (`use_lockfile = true`) vs. Legacy DynamoDB Locking
Historically, the `s3` backend required provisioning a separate Amazon DynamoDB table (`dynamodb_table` with a string partition key named `LockID`) to support distributed locking.
* **Terraform 1.10+ Native S3 State Locking (`use_lockfile = true`):**
  * Leverages Amazon S3's native support for **conditional writes** (`If-None-Match: "*"` HTTP header).
  * When a run starts, Terraform writes a lock metadata object adjacent to the state file at **`<state-key>.tflock`**.
  * If another runner attempts to acquire the lock simultaneously, S3 atomically rejects the second `PutObject` call with `412 Precondition Failed` because `<state-key>.tflock` already exists!
  * **Deprecation Note:** `dynamodb_table` is now deprecated in modern Terraform 1.10/1.11+ in favor of `use_lockfile = true`. During transitional migrations, if both `use_lockfile = true` and `dynamodb_table = "..."` are configured, Terraform acquires locks in **both** S3 and DynamoDB so legacy and upgraded runners cannot collide.

```hcl
terraform {
  required_version = ">= 1.10.0"

  backend "s3" {
    bucket       = "corp-prod-terraform-state-123456789012"
    key          = "networking/core-vpc/terraform.tfstate"
    region       = "us-east-1"
    use_lockfile = true # Native S3 .tflock conditional write locking (TF 1.10+)
    encrypt      = true # Enforces SSE-S3 or SSE-KMS encryption at rest
    kms_key_id   = "arn:aws:kms:us-east-1:123456789012:key/mrk-998877665544"
  }
}
```

### 2.2 Least-Privilege IAM Policy for the `s3` Backend with `use_lockfile = true`
* **Bucket-Level Permissions:**
  * `s3:ListBucket` on `arn:aws:s3:::corp-prod-terraform-state-123456789012`
* **Object-Level Permissions (State File + `.tflock` File):**
  * `s3:GetObject` and `s3:PutObject` on `arn:aws:s3:::corp-prod-terraform-state-123456789012/networking/core-vpc/terraform.tfstate`
  * `s3:GetObject`, `s3:PutObject`, and **`s3:DeleteObject`** on `arn:aws:s3:::corp-prod-terraform-state-123456789012/networking/core-vpc/terraform.tfstate.tflock` *(Note: `s3:DeleteObject` is mandatory on the `.tflock` path so Terraform can release the lock upon completion!)*
* **KMS Permissions (when `kms_key_id` is used):**
  * `kms:Decrypt`, `kms:GenerateDataKey`, and `kms:DescribeKey` on the KMS key ARN.

### 2.3 Partial Backend Configurations for Multi-Environment CI/CD
Because `backend` and `cloud` blocks are evaluated during `terraform init` before variables or locals are loaded, **you cannot use `var.*`, `local.*`, or `data.*` expressions inside a `backend` block!**
To reuse the same root configuration across `dev`, `staging`, and `prod` state paths, omit environment-specific attributes in `backend.tf` and inject them at `init` time:

```hcl
# backend.tf (Partial Configuration — reusable across environments)
terraform {
  backend "s3" {
    region       = "us-east-1"
    use_lockfile = true
    encrypt      = true
  }
}
```
```bash
# Inject environment-specific bucket and key during CI/CD initialization:
terraform init \
  -input=false \
  -backend-config="bucket=corp-${ENV}-terraform-state" \
  -backend-config="key=services/payments/${ENV}.tfstate"
# Alternatively, pass a file: terraform init -backend-config="backends/${ENV}.s3.tfbackend"
```

### 2.4 Backend Migration Mechanics: `-migrate-state` vs. `-reconfigure`
When modifying the `backend` block or switching from `backend "s3"` to the HCP Terraform `cloud` block:
* **`terraform init -migrate-state`:**
  * Instructs Terraform to connect to both the **previous backend** (recorded in `.terraform/terraform.tfstate`) and the **newly configured backend**, and copy the existing state payload into the new backend while preserving `lineage` and `serial`.
  * In non-interactive CI/CD pipelines, combine with `-force-copy` (`terraform init -migrate-state -force-copy -input=false`) to suppress the interactive yes/no confirmation prompt.
* **`terraform init -reconfigure`:**
  * Discards the cached local backend pointer in `.terraform/terraform.tfstate` and initializes the new backend configuration **without copying or migrating any existing state data**.
  * **When to use `-reconfigure`:** When switching your local working directory to point at an already-populated remote state file, or after manually migrating state out-of-band.
  * **Exam Trap:** Running `terraform init -reconfigure` when migrating from a populated local state file to an empty new S3 bucket leaves the new S3 bucket completely empty—causing the next `terraform plan` to attempt to recreate every resource from scratch!

---

## 3. Blast-Radius Segmentation & Cross-Stack Data Sharing

Monolithic state files containing thousands of resources across networking, databases, and application compute introduce three severe production failure modes:
1. **Unacceptable Blast Radius:** A syntax error, bad provider upgrade, or credential issue in an application microservice blocks or corrupts core VPC/database deployments.
2. **State Lock Contention:** 30 developers queueing against a single `.tfstate` lock serializes all CI/CD pipelines.
3. **API Rate Limiting & Slow Graph Refresh:** Refreshing 3,000 resources on every `terraform plan` triggers cloud provider API throttling (`RequestLimitExceeded` / `ThrottlingException`).

```text
+-----------------------------------------------------------------------------------+
|               ENTERPRISE LAYERED STATE ARCHITECTURE & DATA FLOW                   |
+-----------------------------------------------------------------------------------+
  [Layer 1: Foundation Networking (networking/prod.tfstate)]
    |-- Provisions: AWS VPC, Subnets, Transit Gateway, Route Tables
    +-- Publishes:  SSM Parameters (/infra/prod/vpc_id) & Output Contract
                          |
          +---------------+---------------+
          |                               |
          v                               v
  [Layer 2: Stateful Data Tier]   [Layer 3: Kubernetes Compute Platform]
  (data/prod-rds.tfstate)         (compute/prod-eks.tfstate)
    |-- Reads VPC/Subnet IDs        |-- Reads VPC/Subnet IDs & DB Endpoints
    +-- Isolated RBAC & Locks       +-- Independent CI/CD Release Cadence
```

### 3.1 Comparing the 3 Cross-Stack Reference Patterns

* **Pattern A: `data "terraform_remote_state"` (Tightly Coupled — Legacy/Restricted)**
  ```hcl
  data "terraform_remote_state" "network" {
    backend = "s3"
    config = {
      bucket = "corp-prod-terraform-state"
      key    = "networking/core-vpc/terraform.tfstate"
      region = "us-east-1"
    }
  }
  ```
  * **Critical Security Flaw:** To read a single output (e.g., `vpc_id`), the consuming workspace's IAM role must be granted `s3:GetObject` on the producer's **entire raw `.tfstate` file**, exposing every plaintext secret or sensitive attribute stored inside the producer's state!
  * **Operational Flaw:** Exposes only **root-level outputs** (`outputs.*`). If the producer removes or renames an output, the consumer fails on its next plan.

* **Pattern B: `data "tfe_outputs"` (HCP Terraform / Terraform Enterprise Least-Privilege Sharing)**
  ```hcl
  data "tfe_outputs" "network" {
    organization = "corp-platform"
    workspace    = "prod-networking-use1"
  }

  resource "aws_security_group" "app" {
    vpc_id = data.tfe_outputs.network.values.vpc_id
  }
  ```
  * **Security Advantage:** When configured with HCP Terraform workspace **"Share outputs with specific workspaces"** settings, `tfe_outputs` fetches *only* the exported output values via the HCP Terraform API **without granting the consumer access to the producer's underlying raw state file**!

* **Pattern C: Cloud-Native Parameter Store / Data Source Lookups (Fully Decoupled — Gold Standard)**
  ```hcl
  # Producer writes explicit integration contract to SSM Parameter Store:
  resource "aws_ssm_parameter" "vpc_id" {
    name  = "/platform/${var.env}/network/vpc_id"
    type  = "String"
    value = aws_vpc.main.id
  }

  # Consumer reads ONLY the allowed SSM path or queries tags via data "aws_vpc":
  data "aws_ssm_parameter" "vpc_id" {
    name = "/platform/${var.env}/network/vpc_id"
  }
  ```
  * **Architectural Advantage:** Complete decoupling from Terraform backend mechanics; fine-grained IAM path access (`arn:aws:ssm:*:*:parameter/platform/prod/network/*`); works seamlessly even if the producer stack is managed by CloudFormation, Pulumi, or another tool.

---

## 4. Drift Detection, Reconciliation & CI/CD Exit Codes

### 4.1 Why Legacy `terraform refresh` Is Dangerous
Running the standalone `terraform refresh` command immediately overwrites the remote `.tfstate` file with whatever returned from the cloud provider APIs **without generating a reviewable execution plan or prompting for confirmation**.
* **Catastrophic Scenario:** If your AWS IAM credentials temporarily expired or lacked read permissions for a region, or if a cloud API outage returned empty responses, `terraform refresh` could silently wipe resources from `.tfstate`!

### 4.2 The Safe `-refresh-only` Workflow
Always use `-refresh-only` inside `plan` and `apply`:
```bash
# 1. Generate a reviewable diff showing ONLY how .tfstate would change to match cloud reality:
terraform plan -refresh-only -out=refresh.tfplan

# 2. Apply the reviewed state update (zero cloud resources are modified):
terraform apply refresh.tfplan
```
* **When to accept drift (`apply -refresh-only`):** An on-call engineer made an emergency hotfix in the AWS Console during an outage, and you want to absorb that change into `.tfstate` first while updating your `.tf` code to match.
* **When to revert drift (standard `terraform apply`):** Someone manually opened port `0.0.0.0/0` on a production Security Group out-of-band, and you want Terraform to overwrite the manual change and restore the secure state declared in Git.

### 4.3 Automated Drift Monitoring with `-detailed-exitcode`
When running scheduled drift-detection pipelines (e.g., GitHub Actions cron every 4 hours):
```bash
terraform plan -refresh-only -detailed-exitcode -input=false -no-color
```
* **Exit Code `0`:** Succeeded; zero differences detected between `.tfstate` and live cloud APIs.
* **Exit Code `1`:** Error occurred (provider authentication failure, network timeout, syntax error).
* **Exit Code `2`:** Succeeded; **non-empty diff detected** (out-of-band infrastructure drift exists!). Your CI wrapper script traps exit code `2` and posts the formatted plan diff to Slack/PagerDuty or opens a GitHub Issue.

---

## 5. State Surgery & Disaster Recovery Runbooks

### 5.1 Resolving Orphaned State Locks (`force-unlock`)
If a CI/CD worker node crashes or loses network connectivity mid-run, the state lock remains held in S3 (`.tflock`) or DynamoDB.
1. **Verify Zero Active Runs:** First check your CI/CD job queue and cloud provider audit logs (CloudTrail) to confirm the previous `terraform apply` process is genuinely dead (forcing an unlock while another process is actively mutating state causes catastrophic state corruption!).
2. **Execute `force-unlock`:**
   ```bash
   terraform force-unlock 4b825dc6-42cb-6eb9-a060-e54bf8d69288
   ```
3. **Verify State Integrity:** Run `terraform plan -refresh-only` to check whether any resources were partially created or modified right before the crash before running a normal `plan`.

### 5.2 Recovering from Corrupted or Accidentally Overwritten Remote State
1. **Enable Backend Object Versioning:** Always enable **S3 Bucket Versioning** (or GCS Object Versioning) on remote state buckets.
2. **Pull Historical Version & Inspect:** Download the previous intact object version from S3 (`aws s3api get-object --bucket ... --version-id <VER_ID> recovered.tfstate`).
3. **Increment Serial & Push:**
   * If you open `recovered.tfstate` and increment its `"serial"` field so that it is higher than the corrupted remote state's `"serial"` (with matching `"lineage"`), a standard `terraform state push recovered.tfstate` will succeed cleanly!
   * Otherwise, use `terraform state push -force recovered.tfstate` to overwrite the corrupted remote state.

### 5.3 Migrating Provider Namespaces (`state replace-provider`)
When moving from a public registry provider to an internal mirror namespace or when a provider changes ownership:
```bash
terraform state replace-provider \
  registry.terraform.io/hashicorp/aws \
  registry.internal.corp/platform/aws
```

---

## 6. Domain 2 Production Scenario & Exam Trap Matrix

* **Scenario 1: Migrating from DynamoDB locking to S3 native locking (`use_lockfile = true`)**
  * **Trap:** Removing `s3:DeleteObject` from the Terraform IAM role to "prevent accidental state deletion."
  * **Reality:** While `s3:DeleteObject` is not needed for `.tfstate`, S3 native locking (`use_lockfile = true`) creates and deletes `<key>.tflock` on every run. Blocking `s3:DeleteObject` on `*.tflock` causes every run to leave a permanent orphaned lock! Scope `Deny s3:DeleteObject` strictly to `*.tfstate` while allowing `s3:DeleteObject` on `*.tflock`.
* **Scenario 2: Switching from local state to S3 backend using `terraform init -reconfigure`**
  * **Trap:** Using `-reconfigure` instead of `-migrate-state` when moving an active local workspace to remote storage.
  * **Reality:** `-reconfigure` ignores existing local state and initializes an empty remote state, causing Terraform to lose track of all existing cloud resources.
* **Scenario 3: Using variables inside the `backend "s3"` block**
  * **Trap:** Writing `bucket = "state-${var.env}"` inside `terraform { backend "s3" { ... } }`.
  * **Reality:** Backend initialization runs before variable evaluation. Use partial backend configuration and pass `-backend-config="bucket=state-prod"` to `terraform init`.

---

## 7. Platform Engineering Deep Dive: Zero-Downtime Monolithic State Splitting Runbook

In a Platform Engineering role (and on the Professional 002 exam), one of the most critical operational tasks is decomposing a monolithic state file (`prod-monolith.tfstate` containing Networking + RDS + EKS) into isolated domain states (`networking.tfstate`, `rds.tfstate`, `eks.tfstate`) **with zero cloud resource destruction**.

```text
+-----------------------------------------------------------------------------------+
|       ZERO-DOWNTIME STATE SPLITTING: DECLARATIVE removed + import WORKFLOW        |
+-----------------------------------------------------------------------------------+
  [Source Workspace: prod-monolith]               [Target Workspace: prod-rds]
                 |                                              |
  1. Replace `resource "aws_db_instance" "main"`  2. Add `resource "aws_db_instance" "main"`
     with `removed { destroy = false }`              AND `import { to = ..., id = ... }`
                 |                                              |
                 v                                              v
  3. Run `terraform plan && apply`                4. Run `terraform plan && apply`
     --> Removes RDS from monolith state             --> Adopts RDS into new state
     --> Physical RDS stays running!                 --> Verifies 0 changes, 0 destroy!
```

### 7.1 Method A: Modern Declarative State Split (`removed` + `import` Blocks — Recommended for GitOps)
Unlike legacy CLI commands that require local terminal access to production state buckets, the declarative approach executes entirely through standard Pull Requests:
1. **In the Source Workspace (`prod-monolith`):** Remove the `resource "aws_db_instance" "main"` block and add a `removed` block so Terraform drops the resource from the monolith state without issuing an API delete call:
   ```hcl
   removed {
     from = aws_db_instance.main
     lifecycle {
       destroy = false
     }
   }
   ```
2. **In the Destination Workspace (`prod-rds`):** Copy the `resource "aws_db_instance" "main"` HCL block into the new workspace and add a declarative `import` block:
   ```hcl
   import {
     to = aws_db_instance.main
     id = "prod-payments-postgres-01"
   }
   ```
3. **Verification Gate:** Run `terraform plan` in both workspaces before applying. The source workspace must show **`0 to destroy (1 removed from state)`** and the destination workspace must show **`1 to import, 0 to add, 0 to change, 0 to destroy`**.

### 7.2 Method B: Imperative CLI State Split (`state pull` / `state mv -state-out` / `state push`)
When working with legacy Terraform versions (`< 1.7`) or performing emergency surgery:
```bash
# 1. Pull both remote state files locally:
cd /work/prod-monolith && terraform state pull > /tmp/monolith.tfstate
cd /work/prod-rds      && terraform state pull > /tmp/rds.tfstate

# 2. Move the RDS resource from monolith.tfstate into rds.tfstate locally:
terraform state mv \
  -state=/tmp/monolith.tfstate \
  -state-out=/tmp/rds.tfstate \
  aws_db_instance.main \
  aws_db_instance.main

# 3. Push both updated state files back to their respective remote backends:
cd /work/prod-monolith && terraform state push /tmp/monolith.tfstate
cd /work/prod-rds      && terraform state push /tmp/rds.tfstate

# 4. Run `terraform plan -detailed-exitcode` in both workspaces to confirm Exit Code 0!
```

---

## 8. Cross-Account S3 Backend Architecture (`assume_role` in `backend "s3"`)

In multi-account AWS Organizations (Landing Zone / Control Tower), platform engineers centralize all `.tfstate` files inside a dedicated **Central State / Shared Services AWS Account (`111122223333`)**, while workloads are deployed into spoke accounts (`444455556666`).
* **Decoupling Backend Credentials from Provider Credentials:**
  * The `backend "s3"` block supports its own dedicated `assume_role` block (Terraform 1.6+) so Terraform uses one IAM role to read/write state in the Central State Account and a completely different IAM role in `provider "aws"` to provision resources in the Spoke Workload Account:

```hcl
terraform {
  backend "s3" {
    bucket       = "central-org-terraform-state-111122223333"
    key          = "accounts/payments-prod/network.tfstate"
    region       = "us-east-1"
    use_lockfile = true
    encrypt      = true
    kms_key_id   = "arn:aws:kms:us-east-1:111122223333:key/central-state-kms"

    # Dedicated IAM role for reading/locking state in the Central State Account:
    assume_role = {
      role_arn     = "arn:aws:iam::111122223333:role/TerraformStateBackendAccessRole"
      session_name = "tf-backend-payments-prod"
      external_id  = "org-state-guard-9912"
    }
  }
}

provider "aws" {
  region = "us-east-1"

  # Separate IAM role for provisioning infrastructure in the Spoke Workload Account:
  assume_role {
    role_arn     = "arn:aws:iam::444455556666:role/TerraformWorkloadProvisionerRole"
    session_name = "tf-workload-payments-prod"
  }
}
```

