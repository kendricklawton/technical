# Domain 2: Terraform Fundamentals — Study Guide (004)

> **Official 004 Blueprint Focus:** Objective 2 — Terraform Fundamentals (`2a`, `2b`, `2c`, `2d`).  
> **Tested Version:** Terraform 1.12+

---

## 1. Install and Version Terraform Providers (`2a`) & How Terraform Uses Providers (`2b`)

### 1.1 Terraform Core vs. Provider Plugins Architecture
Terraform is engineered as a two-tier architecture:
1. **Terraform Core (`terraform` CLI binary):**
   - Reads and interpolates `.tf` and `.tf.json` configuration files.
   - Manages the state file (`terraform.tfstate`) and builds the **Directed Acyclic Graph (DAG)**.
   - Executes plans and orchestrates concurrency (default `-parallelism=10`).
   - Launches provider plugin binaries as subprocesses and communicates with them over local **gRPC (Inter-Process Communication)**.
2. **Provider Plugins (`terraform-provider-*` binaries):**
   - Standalone executables written in Go (using the Terraform Plugin Framework / SDK).
   - Handle vendor-specific authentication (e.g., AWS IAM/OIDC, Azure Service Principal, GCP Service Account).
   - Define the schema and validation rules for each `resource`, `data` source, and `ephemeral` resource.
   - Translate Terraform CRUD operations into upstream REST/gRPC API calls.

### 1.2 Declaring Providers & Version Constraints (`required_providers`)
```hcl
terraform {
  required_version = ">= 1.12.0, < 2.0.0"

  required_providers {
    aws = {
      source  = "registry.terraform.io/hashicorp/aws" # Default registry hostname can be omitted: "hashicorp/aws"
      version = "~> 5.80.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.30"
    }
  }
}
```

* **Provider Source Address Format:** `[<HOSTNAME>/]<NAMESPACE>/<TYPE>`
  * If `<HOSTNAME>` is omitted, Terraform defaults to the Public Terraform Registry: `registry.terraform.io`.
  * **Tiers on the Public Registry:**
    * **Official:** Owned and maintained by **HashiCorp** (e.g., `hashicorp/aws`, `hashicorp/azurerm`, `hashicorp/google`, `hashicorp/vault`).
    * **Partner:** Owned and maintained by a verified third-party technology company for their own API (e.g., `datadog/datadog`, `cloudflare/cloudflare`, `MongoDB/mongodbatlas`).
    * **Community:** Published and maintained by individual open-source contributors.
* **Version Constraint Operators:**
  * `= 5.80.0` (or `"5.80.0"`): Exact version match only.
  * `!= 5.80.0`: Excludes a problematic version.
  * `>= 5.0.0, < 6.0.0`: Range comparison.
  * `~> 5.80.0` (**Pessimistic Patch Constraint**): Allows only the **rightmost** version component to increment (`>= 5.80.0` and `< 5.81.0`).
  * `~> 5.80` (**Pessimistic Minor Constraint**): Allows the minor version to increment (`>= 5.80.0` and `< 6.0.0`).

### 1.3 Where Providers Are Installed & The Dependency Lock File (`.terraform.lock.hcl`)
When you run `terraform init`:
1. Terraform downloads the matching provider binaries into:
   `.terraform/providers/<HOSTNAME>/<NAMESPACE>/<TYPE>/<VERSION>/<OS>_<ARCH>/`
2. If `TF_PLUGIN_CACHE_DIR` is set (or `plugin_cache_dir` in `.terraformrc` / `terraform.rc`), Terraform caches and symlinks provider binaries across projects to save bandwidth and disk space.
3. Terraform generates or updates **`.terraform.lock.hcl`** in the root working directory:
   - Records the **exact provider version** selected and cryptographic SHA-256 checksums (`h1:` package hash and `zh:` zip archive hash).
   - **Exam Rule:** Always commit **`.terraform.lock.hcl`** to Version Control (Git) so every teammate and CI runner installs the exact same provider binary. Never commit the `.terraform/` directory itself!
   - To upgrade a locked provider to the newest version allowed by your `version` constraint, run `terraform init -upgrade`.

### 1.4 Provider Inspection & Multi-Platform Lock Management (`2a`)
- **`terraform providers`:** Prints a tree of all provider requirements across the root module and any child modules, making it easy to identify which module is requesting a specific or outdated provider version.
- **`terraform providers lock -platform=...`:** Pre-fetches checksums for multiple target operating systems and architectures without needing to run `terraform init` on each platform. For example, a developer on macOS Apple Silicon can lock packages for Linux CI runners and Windows workstations:
  ```bash
  terraform providers lock \
    -platform=darwin_arm64 \
    -platform=linux_amd64 \
    -platform=windows_amd64
  ```
  This adds the appropriate `zh:` zip hashes to `.terraform.lock.hcl` so CI pipelines running in Linux containers do not fail hash verification.
- **`terraform providers mirror <TARGET-DIR>`:** Downloads and copies provider binaries into a local directory filesystem to support air-gapped, offline, or restricted enterprise environments.

---

## 2. Write Terraform Configuration Using Multiple Providers (`2c`)

A single configuration can combine **different provider types** (e.g., `aws` + `cloudflare`) or **multiple configurations of the same provider** using the `alias` meta-argument (e.g., multi-region or multi-account AWS deployments):

```hcl
# 1. Default (unaliased) AWS provider — used automatically by all aws_* resources unless overridden
provider "aws" {
  region = "us-east-1"
}

# 2. Aliased AWS provider for Disaster Recovery region
provider "aws" {
  alias  = "dr_west"
  region = "us-west-2"
}

# Uses default provider (us-east-1)
resource "aws_kms_key" "primary" {
  description = "Primary KMS Key in us-east-1"
}

# Uses aliased provider (us-west-2) via the 'provider' meta-argument
resource "aws_kms_replica_key" "replica" {
  provider        = aws.dr_west # Reference syntax: <PROVIDER_NAME>.<ALIAS> (unquoted!)
  primary_key_arn = aws_kms_key.primary.arn
}
```

> [!TIP]
> **Passing Aliased Providers to Child Modules:** To pass an alternate provider into a child module, use the `providers` map inside the `module` block:
> ```hcl
> module "dr_vpc" {
>   source = "./modules/vpc"
>   providers = {
>     aws = aws.dr_west
>   }
> }
> ```

---

## 3. Explain How Terraform Uses and Manages State (`2d`)

Terraform is a **stateful** Infrastructure as Code engine. Whenever `terraform apply` provisions or updates infrastructure, it serializes the current state of managed resources into `terraform.tfstate`:
1. **Mapping Configuration to the Real World:** Maps logical HCL resource addresses (`aws_vpc.main`) to real-world provider IDs (`vpc-0a1b2c3d4e5f6g7h8`).
2. **Metadata & Dependency Tracking:** Stores internal dependency relationships (essential for determining exact deletion order when a resource block is removed from `.tf` code).
3. **Performance & API Caching:** Caches resource attributes locally/remotely so `terraform plan` can quickly diff large infrastructures.
4. **Collaboration & Concurrency Locking:** When stored in a remote backend (HCP Terraform, S3, GCS, AzureRM), state locking ensures only one user or pipeline can mutate infrastructure at a time.
