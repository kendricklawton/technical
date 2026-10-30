# Domain 5: Security, Governance & Policy-as-Code — Complete Study Guide

**Exam Weight:** 12% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** Zero-state secret persistence via Ephemeral Resources (`ephemeral` blocks in Terraform 1.10+) and Write-Only Attributes (`_wo` & `_wo_version` in Terraform 1.11+), `sensitive = true` propagation vs. state plaintext risks, HashiCorp Vault dynamic secret injection & lease lifecycle traps, Policy-as-Code with **HashiCorp Sentinel** (`tfplan/v2`, `tfconfig/v2`, `tfstate/v2`, `tfrun` imports & enforcement levels) and **Open Policy Agent (OPA Rego)**, and Shift-Left Static Analysis (`trivy`, `checkov`, `tflint`).

---

## 1. Secret Management Evolution: `sensitive` vs. `ephemeral` & Write-Only (`_wo`)

One of the highest-priority topics on the modern Professional (002) exam is understanding **what gets persisted into `terraform.tfstate` and saved binary plan files** versus what stays exclusively in runtime memory.

```text
+-----------------------------------------------------------------------------------+
|          TERRAFORM SECRET PERSISTENCE COMPARISON (v1.0 vs. v1.10/v1.11+)          |
+-----------------------------------------------------------------------------------+
  LEGACY PATTERN (data "aws_secretsmanager_secret_version" + sensitive = true):
    [Secrets Manager] ---> data.secret_string ---> resource.password = ...
                                |                         |
                                v                         v
                  [SAVED IN PLAINTEXT IN .tfstate & tfplan.binary!]
                  (sensitive = true ONLY redacts CLI stdout console logs!)

  MODERN ZERO-STATE PATTERN (ephemeral block + Write-Only `_wo` attribute):
    [Secrets Manager / Vault] ---> ephemeral.secret_string ---> resource.password_wo
                                              |                         |
                                     (In-Memory Only!)        (Sent to Cloud API,
                                              |                stored as null in
                                              v                .tfstate & tfplan!)
                                   [ZERO SECRET IN STATE!]
```

### 1.1 Why `sensitive = true` and `data` Sources Leak Secrets into State
* **`sensitive = true`:** Marking a variable or output `sensitive = true` (or wrapping an expression in `sensitive(...)`) **only redacts the value from terminal stdout (`(sensitive value)`) and HCP Terraform UI logs**. The raw cleartext string is still written directly into `terraform.tfstate` and `tfplan.binary`!
* **`data` Sources Persist to State:** If you fetch a password using `data "aws_secretsmanager_secret_version" "db"` or `data "vault_generic_secret" "db"`, Terraform caches the entire data source response—including `secret_string`—in plaintext inside `terraform.tfstate`.

### 1.2 Ephemeral Resources (`ephemeral` Block — Terraform 1.10+)
An `ephemeral` block opens a temporary resource or secret during each evaluation phase (`plan` and `apply`) and **never writes any of its attributes to `.tfstate` or plan files**:
```hcl
# Opens a short-lived Vault secret or KMS decryption session strictly in memory:
ephemeral "aws_secretsmanager_secret_version" "rds_master" {
  secret_id = "prod/core/rds-master-credentials"
}

# Use ephemeral credential inside a provider block (never stored in state!):
provider "postgresql" {
  host     = aws_db_instance.main.address
  username = jsondecode(ephemeral.aws_secretsmanager_secret_version.rds_master.secret_string)["username"]
  password = jsondecode(ephemeral.aws_secretsmanager_secret_version.rds_master.secret_string)["password"]
}
```
* **Lifecycle Operations of `ephemeral` Blocks:**
  1. **`Open`:** Invoked when Terraform evaluates the block during `plan` or `apply` (e.g., acquiring a dynamic database lease, opening an SSH tunnel, or fetching a secret version).
  2. **`Renew`:** Periodically invoked in the background if a long-running `apply` exceeds the ephemeral credential's initial lease TTL.
  3. **`Close`:** Invoked automatically at the end of the Terraform run to revoke the dynamic lease or close the network connection cleanly!

### 1.3 Write-Only Attributes (`_wo` and `_wo_version` — Terraform 1.11+)
Before Terraform 1.11, even if you fetched a secret via an `ephemeral` block, you could not pass it to `resource "aws_db_instance" { password = ... }` because regular resource attributes are persisted to `.tfstate`.
* **Write-Only Attributes (`<attr>_wo`):** Provider schemas now expose write-only counterparts (e.g., `password_wo`, `secret_string_wo`, `private_key_wo`) that accept `ephemeral` values, transmit the value to the cloud provider API during `Create`/`Update`, and **always record `null` in `terraform.tfstate` and the plan file**!
* **How Rotation Works Without State Diffs (`<attr>_wo_version`):**
  * Because `password_wo` is always `null` in state, Terraform cannot detect if the secret changed by comparing strings.
  * Instead, you pair `password_wo` with an integer attribute **`password_wo_version`** (which *is* stored in `.tfstate`). Incrementing `password_wo_version` from `1` to `2` triggers Terraform to re-evaluate `ephemeral` and send the updated `password_wo` to the cloud API!

---

## 2. Policy-as-Code: HashiCorp Sentinel vs. Open Policy Agent (OPA)

In HCP Terraform and Terraform Enterprise, Policy-as-Code executes **between the `plan` and `apply` phases** to enforce security, compliance, and FinOps guardrails before any cloud API is mutated.

```text
+-----------------------------------------------------------------------------------+
|            POLICY-AS-CODE EVALUATION IN THE TERRAFORM RUN PIPELINE                |
+-----------------------------------------------------------------------------------+
  [1. Plan Phase Completes] ---> Exports JSON representations:
                                   |-- tfconfig/v2 (Static HCL AST & module calls)
                                   |-- tfplan/v2   (Planned resource diffs & after values)
                                   |-- tfstate/v2  (Prior state values)
                                   +-- tfrun       (Workspace metadata, cost estimate)
                                         |
                                         v
  [2. Policy Check Phase (Sentinel or OPA Rego)]
        |-- Advisory:       Logs warning on failure; run proceeds automatically.
        |-- Soft-Mandatory: Blocks apply; authorized admin CAN override with reason.
        +-- Hard-Mandatory: Blocks apply; ZERO overrides permitted (must fix code!).
```

### 2.1 HashiCorp Sentinel Imports & Rules
Sentinel uses four specialized Terraform imports (always use **`/v2`** imports on the exam!):
* **`import "tfplan/v2" as tfplan`:** Inspects the calculated change set (`tfplan.resource_changes`). Used to verify that planned resource attribute values (`rc.change.after`) comply with security standards (e.g., `rc.change.after.encrypted is true`).
* **`import "tfconfig/v2" as tfconfig`:** Inspects the static HCL source code (`tfconfig.resources`, `tfconfig.modules`, `tfconfig.providers`). Used to enforce coding standards (e.g., requiring all module calls to come from the Private Registry and pin an exact version).
* **`import "tfstate/v2" as tfstate`:** Inspects the current state prior to the plan (`tfstate.resources`).
* **`import "tfrun" as tfrun`:** Inspects workspace metadata, VCS branch, organization name, and **Cost Estimation** (`tfrun.cost_estimate.delta_monthly_cost`).

```python
# Sentinel Policy Example: Enforce KMS encryption & Cost Delta Limit
import "tfplan/v2" as tfplan
import "tfrun" as tfrun
import "decimal"

# Filter all AWS EBS volumes being created or updated:
ebs_volumes = filter tfplan.resource_changes as _, rc {
  rc.type is "aws_ebs_volume" and
  (rc.change.actions contains "create" or rc.change.actions contains "update")
}

# Rule 1: Every EBS volume must have encrypted == true and a non-null kms_key_id
encryption_compliant = rule {
  all ebs_volumes as _, vol {
    vol.change.after.encrypted is true and
    vol.change.after.kms_key_id is not null
  }
}

# Rule 2: Monthly cost increase must not exceed $500.00
cost_compliant = rule {
  decimal.less_than_or_equals(tfrun.cost_estimate.delta_monthly_cost, 500.00)
}

main = rule {
  encryption_compliant and cost_compliant
}
```

### 2.2 Open Policy Agent (OPA Rego) in HCP Terraform
HCP Terraform also natively supports **OPA policies written in Rego**, evaluating the standard JSON plan (`input.plan.resource_changes`) and run context (`input.run`):
```rego
package terraform.policies.s3_security

import rego.v1

# Deny any S3 bucket public access block where block_public_acls is not true
deny contains msg if {
  some rc in input.plan.resource_changes
  rc.type == "aws_s3_bucket_public_access_block"
  "create" in rc.change.actions
  rc.change.after.block_public_acls != true
  msg := sprintf("Resource %s must set block_public_acls = true", [rc.address])
}
```

---

## 3. Shift-Left Static Analysis (`tflint`, `trivy`, `checkov`)

While Sentinel and OPA run after `terraform plan` (requiring cloud credentials), shift-left tools run locally or in pre-plan CI steps without cloud credentials:
* **`terraform fmt -check` & `terraform validate`:** Built-in syntax and internal type/attribute consistency checks. (`validate` runs offline using cached providers with `-backend=false`).
* **`tflint`:** Provider-aware linter that catches invalid instance types (e.g., `t3.superlarge`), deprecated syntax, and naming convention violations before calling AWS APIs.
* **`trivy config` / `checkov`:** Security static analysis scanners that inspect `.tf` files and rendered `tfplan.json` for CIS Benchmark and NIST compliance violations.

---

## 4. Domain 5 Production Scenario & Exam Trap Matrix

* **Scenario 1: Preventing an RDS password from ever appearing in `terraform.tfstate`**
  * **Trap:** Choosing `variable "db_password" { sensitive = true }` or `data "vault_generic_secret"` passed to `password = ...`.
  * **Reality:** Both approaches write the plaintext password into `.tfstate`! The only correct solution is using an `ephemeral` resource or `ephemeral = true` variable passed to the Write-Only attribute **`password_wo`** (paired with `password_wo_version`).
* **Scenario 2: Enforcing that every module call in `main.tf` uses a specific Private Registry hostname**
  * **Trap:** Using `tfplan/v2` in Sentinel to inspect module source strings.
  * **Reality:** `tfplan/v2` focuses on evaluated resource instances and planned attribute values; to inspect static HCL module `source` strings and `version` constraints as written in `.tf` files, you must use **`tfconfig/v2`**!

---

## 5. Platform Engineering Deep Dive: Solving the HashiCorp Vault Lease-in-State Paradox

When integrating Terraform with **HashiCorp Vault** to generate short-lived dynamic credentials (such as dynamic AWS IAM STS tokens, GCP OAuth tokens, or dynamic PostgreSQL database credentials), platform engineers must understand the difference between legacy `data` sources and modern `ephemeral` blocks:

```text
+-----------------------------------------------------------------------------------+
|         VAULT DYNAMIC CREDENTIAL LIFECYCLE: LEGACY DATA vs. EPHEMERAL             |
+-----------------------------------------------------------------------------------+
  LEGACY ANTI-PATTERN (`data "vault_database_secret_backend_creds" "db"`):
    1. Terraform reads dynamic DB username/password from Vault (Lease TTL = 1 hour).
    2. Terraform writes the short-lived password into `terraform.tfstate`!
    3. 1 hour later, Vault automatically REVOKES the lease in PostgreSQL!
    4. `terraform.tfstate` now contains an expired, dead credential & plaintext leak!

  MODERN PLATFORM PATTERN (`ephemeral` + Write-Only `_wo` OR OIDC Workload Identity):
    1. For Provider Auth: Use HCP Terraform Dynamic Vault Credentials (`TFC_VAULT_PROVIDER_AUTH`).
    2. For Resource Secrets: Use `ephemeral "vault_kv_secret_v2"` -> `password_wo`.
       * Opens lease at start of run, renews during apply, closes at end of run,
         and NEVER writes credentials to `terraform.tfstate`!
```

---

## 6. Authoring & Unit-Testing Sentinel and OPA Policy Sets Locally

In a Platform Engineering team, Policy-as-Code is treated just like application code: policies live in a dedicated Git repository connected to HCP Terraform **Policy Sets** and are unit-tested in CI before merge.

### 6.1 Directory Structure & `sentinel.hcl` Configuration
```text
policies/
├── sentinel.hcl
├── enforce-mandatory-tags.sentinel
├── restrict-db-public-access.sentinel
└── test/
    └── enforce-mandatory-tags/
        ├── pass-all-tags-present.hcl
        └── fail-missing-cost-center.hcl
```
Inside **`sentinel.hcl`**, each policy declares its source path and **enforcement level** (`advisory`, `soft-mandatory`, or `hard-mandatory`):
```hcl
policy "enforce-mandatory-tags" {
  source            = "./enforce-mandatory-tags.sentinel"
  enforcement_level = "soft-mandatory"
}

policy "restrict-db-public-access" {
  source            = "./restrict-db-public-access.sentinel"
  enforcement_level = "hard-mandatory"
}
```
* **Testing Locally with `sentinel test`:**
  * Run `sentinel test` in the repository root. Sentinel executes every test case under `test/<policy-name>/*.hcl` using mocked `tfplan/v2` data exported from HCP Terraform (`mock-tfplan-v2.sentinel`) and verifies both passing (`true`) and failing (`false`) assertions!

### 6.2 Unit-Testing OPA Rego Policies (`opa test -v`)
```rego
# policy/rds_security_test.rego
package terraform.policies.rds_test

import rego.v1
import data.terraform.policies.rds

test_public_rds_denied if {
  mock_input := {
    "plan": {
      "resource_changes": [{
        "address": "aws_db_instance.bad",
        "type": "aws_db_instance",
        "change": {
          "actions": ["create"],
          "after": {"publicly_accessible": true}
        }
      }]
    }
  }
  count(rds.deny) == 1 with input as mock_input
}
```

