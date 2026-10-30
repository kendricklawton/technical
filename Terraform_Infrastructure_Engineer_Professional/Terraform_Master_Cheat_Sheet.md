# Terraform Infrastructure Engineer Professional (002) — Master Cheat Sheet

```text
========================================================================================
              TERRAFORM EXECUTION & STATE LIFECYCLE ARCHITECTURE (v1.11+)
========================================================================================

  [ HCL Code (.tf / .tfstack.hcl) ] + [ Input Vars (.tfvars / TF_VAR_*) ]
                  |
                  v
  [ 1. terraform init ] -----> Downloads providers, verifies .terraform.lock.hcl,
                  |            initializes backend (-migrate-state vs -reconfigure)
                  v
  [ 2. terraform validate / test ] -> Static type/schema check + .tftest.hcl mock runs
                  |
                  v
  [ 3. terraform plan -out=tfplan ]
        |-- Acquires Distributed Lock (S3 .tflock If-None-Match / DynamoDB / HCP)
        |-- Refreshes prior state against Cloud APIs (unless -refresh=false)
        |-- Evaluates Ephemeral blocks (in-memory only) & Provider Functions
        |-- Checks variable validations, preconditions, postconditions & check blocks
        +-- Writes encrypted/local binary plan file (ephemeral values excluded!)
                  |
                  v
  [ 4. Policy Gate (Sentinel / OPA Rego) ] -> Evaluates tfplan/v2, tfconfig/v2, tfstate/v2
                  |
                  v
  [ 5. terraform apply tfplan ]
        |-- Executes DAG graph in parallel (default -parallelism=10)
        |-- Sends Write-Only attributes (_wo) to Cloud APIs without recording in state
        |-- Commits updated state (increments serial, preserves lineage UUID)
        +-- Releases Distributed State Lock
```

---

## 1. Initialization, Backend Migration & Provider Locking

* **Standard & CI/CD Initialization:**
  ```bash
  # Standard interactive init
  terraform init

  # Non-interactive CI/CD init with partial backend configuration
  terraform init -input=false -backend-config="bucket=corp-prod-state" -backend-config="key=vpc/terraform.tfstate"

  # Copy existing state to a newly configured backend (preserves lineage & serial)
  terraform init -migrate-state

  # Re-initialize backend configuration WITHOUT copying existing state (clean slate / new workspace)
  terraform init -reconfigure

  # Upgrade providers to newest versions allowed by version constraints and update .terraform.lock.hcl
  terraform init -upgrade
  ```

* **Multi-Platform Dependency Lockfile (`.terraform.lock.hcl`):**
  ```bash
  # Populate SHA-256 hashes (`h1:` and `zh:`) for Mac developer laptops + Linux CI runners + ARM64 agents
  terraform providers lock \
    -platform=linux_amd64 \
    -platform=linux_arm64 \
    -platform=darwin_arm64 \
    -platform=windows_amd64
  ```

* **Air-Gapped Provider Network / Filesystem Mirror (`~/.terraformrc` or `TF_CLI_CONFIG_FILE`):**
  ```hcl
  provider_installation {
    filesystem_mirror {
      path    = "/opt/terraform/providers"
      include = ["registry.terraform.io/*/*"]
    }
    direct {
      exclude = ["registry.terraform.io/*/*"]
    }
  }
  ```
  ```bash
  # Populate local mirror directory for air-gapped transfer
  terraform providers mirror -platform=linux_amd64 /opt/terraform/providers
  ```

---

## 2. S3 Native State Locking (Terraform 1.10+) vs. Legacy DynamoDB

* **Modern S3 Native Locking (`use_lockfile = true`):**
  * Uses Amazon S3 conditional writes (`If-None-Match: "*"`) to create a `<key>.tflock` object directly inside the S3 bucket—eliminating the need to provision and maintain a separate DynamoDB lock table!
  ```hcl
  terraform {
    backend "s3" {
      bucket       = "corp-terraform-state-prod"
      key          = "network/vpc.tfstate"
      region       = "us-east-1"
      use_lockfile = true # Native S3 conditional-write locking (.tflock) in TF 1.10+
      encrypt      = true
      kms_key_id   = "arn:aws:kms:us-east-1:111122223333:key/mrk-state-key"
    }
  }
  ```
  * **Required IAM Permissions for S3 Native Locking:** `s3:GetObject`, `s3:PutObject`, and `s3:DeleteObject` on both `arn:aws:s3:::bucket/path/terraform.tfstate` AND `arn:aws:s3:::bucket/path/terraform.tfstate.tflock`.

---

## 3. Declarative State Refactoring vs. Imperative CLI Surgery

* **Declarative Refactoring Blocks (GitOps-Safe, Auditable in PRs):**
  ```hcl
  # 1. Rename resource, move into child module, or convert count -> for_each
  moved {
    from = aws_instance.web[0]
    to   = module.compute.aws_instance.app["primary"]
  }

  # 2. Decommission resource from Terraform management WITHOUT destroying cloud object (TF 1.7+)
  removed {
    from = aws_db_instance.legacy_analytics
    lifecycle {
      destroy = false
    }
  }

  # 3. Declarative bulk import with config generation (TF 1.5+ / for_each in TF 1.6+)
  import {
    for_each = var.existing_buckets
    to       = aws_s3_bucket.imported[each.key]
    id       = each.value
  }
  ```
  ```bash
  # Auto-generate HCL resource blocks from declarative import blocks
  terraform plan -generate-config-out=generated_resources.tf
  ```

* **Imperative CLI State Surgery (Immediate State Mutation):**
  ```bash
  terraform state list                                    # List all tracked resource addresses
  terraform state show 'aws_subnet.private["us-east-1a"]' # Inspect single resource state attributes
  terraform state mv <source> <dest>                      # Imperatively rename/move address in state
  terraform state rm <addr>                               # Orphan resource from state (does not destroy)
  terraform state pull > backup.tfstate                   # Download raw state JSON to stdout
  terraform state push backup.tfstate                     # Upload state JSON (checks lineage & serial)
  terraform state replace-provider hashicorp/aws registry.internal.corp/aws/aws
  terraform force-unlock <LOCK_ID>                        # Break orphaned state lock after crash
  ```

---

## 4. Drift Reconciliation, Replacement & Exit Codes

* **Safe Drift Detection (`-refresh-only`):**
  ```bash
  # Preview out-of-band cloud changes without mutating state or infrastructure
  terraform plan -refresh-only

  # Accept out-of-band changes into .tfstate after human review
  terraform apply -refresh-only
  ```

* **Targeted Resource Replacement (`-replace` replaces deprecated `terraform taint`):**
  ```bash
  # Schedule a single damaged resource for destroy-and-recreate inside the plan
  terraform plan -replace='aws_instance.worker["az1"]' -out=tfplan
  ```

* **CI/CD Automated Drift Detection (`-detailed-exitcode`):**
  ```bash
  terraform plan -detailed-exitcode -input=false -no-color
  # Exit Code 0 = Succeeded, zero diff (No drift)
  # Exit Code 1 = Error (Syntax, authentication, or API failure)
  # Exit Code 2 = Succeeded, non-empty diff (Drift or pending changes detected!)
  ```

---

## 5. Ephemeral Resources & Write-Only Attributes (Zero-State Secrets)

```hcl
# 1. Fetch short-lived secret in memory only (NEVER saved to .tfstate or tfplan!)
ephemeral "aws_secretsmanager_secret_version" "db_master" {
  secret_id = "prod/rds/master-password"
}

# 2. Pass ephemeral secret to a Write-Only attribute (_wo) + version trigger (_wo_version)
resource "aws_db_instance" "primary" {
  identifier          = "prod-postgres"
  engine              = "postgres"
  instance_class      = "db.r6g.large"
  username            = "dbadmin"
  password_wo         = ephemeral.aws_secretsmanager_secret_version.db_master.secret_string
  password_wo_version = var.secret_rotation_version # Increment integer to trigger password rotation!
}
```

---

## 6. Continuous Validation Matrix (`validation` vs. `precondition` vs. `postcondition` vs. `check`)

* **Variable `validation` Block:**
  * **Phase:** Early input evaluation (before API calls).
  * **Scope:** Can reference `var.<self>` and (in TF 1.9+) other variables/locals/data sources in the same module.
  * **Failure Behavior:** Halts `plan` and `apply` immediately (hard blocker).
* **Lifecycle `precondition` Block:**
  * **Phase:** Before evaluating/provisioning the target resource, data source, or output.
  * **Scope:** Can reference any variable, local, data source, or upstream dependency in the module.
  * **Failure Behavior:** Halts `plan` or `apply` before modifying the resource (hard blocker).
* **Lifecycle `postcondition` Block:**
  * **Phase:** After reading a data source or after applying/refreshing a resource.
  * **Scope:** Uses **`self.<attr>`** to inspect actual live attributes returned by the cloud provider API.
  * **Failure Behavior:** Halts downstream dependents and fails the run (hard blocker).
* **Top-Level `check` Block (Terraform 1.5+):**
  * **Phase:** End of `plan` and `apply` execution.
  * **Scope:** Can include its own scoped `data` source (e.g., `data "http" "healthz"`) and `assert` blocks.
  * **Failure Behavior:** Emits a **non-blocking Warning** (`Warning: Check block assertion failed`) without halting the apply or rolling back infrastructure!

---

## 7. Native Testing Framework (`terraform test` / `.tftest.hcl`)

```hcl
# tests/vpc_unit.tftest.hcl
mock_provider "aws" {
  mock_resource "aws_vpc" {
    defaults = {
      id  = "vpc-mock123456"
      arn = "arn:aws:ec2:us-east-1:123456789012:vpc/vpc-mock123456"
    }
  }
}

run "verify_cidr_and_negative_validation" {
  command = plan

  variables {
    vpc_cidr    = "10.99.0.0/16"
    environment = "invalid_env"
  }

  # Assert that invalid input triggers the expected validation failure:
  expect_failures = [
    var.environment
  ]
}
```

---

## 8. Precedence & Hierarchy Quick Rules

* **Variable Precedence (Lowest $\rightarrow$ Highest Winner):**
  1. `default` attribute inside `variable` block
  2. Environment variables (`TF_VAR_<name>`)
  3. `terraform.tfvars` file (auto-loaded)
  4. `terraform.tfvars.json` file (auto-loaded)
  5. `*.auto.tfvars` / `*.auto.tfvars.json` (auto-loaded in lexicographical filename order)
  6. `-var` and `-var-file` CLI flags (evaluated in order passed; **last flag on CLI wins!**)
* **HCP Terraform Variable Precedence (Lowest $\rightarrow$ Highest Winner):**
  1. Organization Global Variable Set
  2. Project-Scoped Variable Set
  3. Workspace-Scoped Variable Set
  4. Workspace-Specific Variables (defined directly on the workspace)
  5. CLI `-var` / `-var-file` (when applicable in CLI-driven runs)
