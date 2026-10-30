# Domain 4: CI/CD & Automation Workflows — Complete Study Guide

**Exam Weight:** 14% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** GitOps PR-driven execution pipelines (Speculative vs. Saved Binary Plans `-out=tfplan`), non-interactive CLI automation flags (`-input=false`, `-no-color`, `-compact-warnings`, `-parallelism=N`), OIDC Dynamic Provider Credentials (HCP Terraform & GitHub Actions/GitLab Workload Identity Federation), Dependency Lockfile internals (`.terraform.lock.hcl` `h1:` vs. `zh:` hashes & multi-platform `providers lock`), Plugin Caching (`TF_PLUGIN_CACHE_DIR` & `plugin_cache_may_break_dependency_lock_file`), and Air-Gapped Provider Installation Mirrors (`filesystem_mirror` & `network_mirror`).

---

## 1. Enterprise GitOps Pipeline Architecture: Speculative vs. Saved Plans

Running `terraform apply -auto-approve` without passing a saved binary plan file in a production CI/CD pipeline introduces a critical **Time-of-Check to Time-of-Use (TOCTOU)** vulnerability: if a second commit lands or a cloud resource changes between the PR review (`plan`) and merge (`apply`), `apply -auto-approve` silently recalculates a brand-new plan and executes unreviewed changes!

```text
+-----------------------------------------------------------------------------------+
|            ENTERPRISE GITOPS TWO-PHASE SAVED PLAN PIPELINE ARCHITECTURE           |
+-----------------------------------------------------------------------------------+
  PHASE 1: PULL REQUEST OPENED (Speculative Validation & Review)
  +-------------------------------------------------------------------------------+
  |  1. terraform fmt -check -recursive                                           |
  |  2. terraform init -input=false                                               |
  |  3. terraform validate && terraform test                                      |
  |  4. terraform plan -input=false -no-color (Speculative Plan posted to PR)     |
  +---------------------------------------+---------------------------------------+
                                          |
                                          v (PR Approved & Merged to `main`)
  PHASE 2: MERGE TO MAIN (Deterministic Locked Plan -> Policy -> Apply)
  +-------------------------------------------------------------------------------+
  |  1. Acquire State Lock & Generate Deterministic Binary Plan:                  |
  |     terraform plan -input=false -out=tfplan.binary                            |
  |  2. Export JSON Plan for Policy-as-Code Gate (OPA / Sentinel / Checkov):      |
  |     terraform show -json tfplan.binary > tfplan.json                          |
  |  3. Execute Exact Approved Binary Plan (Zero Re-Planning Allowed!):           |
  |     terraform apply -input=false tfplan.binary                                |
  +-------------------------------------------------------------------------------+
```

### 1.1 Essential Non-Interactive CI/CD Flags & Environment Variables
* **`TF_IN_AUTOMATION=true`:** Suppresses human-targeted "next step" hints in CLI output so CI logs remain clean and parseable.
* **`-input=false`:** Instructs Terraform to **immediately fail** rather than hang indefinitely waiting for stdin if a variable is missing or a backend prompt occurs.
* **`-out=tfplan.binary`:** Serializes the exact resource diff, provider versions, and prior state serial into a binary plan artifact.
  * **Important Exam Rule:** When passing a saved plan file to `terraform apply tfplan.binary`, you **do NOT need `-auto-approve`** (passing a saved plan file implicitly approves those exact actions), and you **cannot pass `-var` or `-var-file` flags** to `terraform apply tfplan.binary` because all variables are already baked into the saved plan!
* **`-parallelism=N` (Default: `10`):** Controls the maximum number of concurrent DAG resource operations. Lowering `-parallelism=4` resolves cloud provider API rate-limiting (`HTTP 429 TooManyRequests`) in massive workspaces.
* **`TF_LOG` & `TF_LOG_PATH`:**
  * Set `TF_LOG=TRACE` (or `DEBUG`, `INFO`, `WARN`, `ERROR`), `TF_LOG_CORE=TRACE` (core graph engine only), or `TF_LOG_PROVIDER=TRACE` (provider RPC plugin calls only), paired with `TF_LOG_PATH=/tmp/terraform-crash.log` to capture diagnostic logs in CI artifacts.

---

## 2. Workload Identity Federation (OIDC Dynamic Provider Credentials)

Storing static, long-lived cloud credentials (`AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`) in CI/CD secrets or HCP Terraform variables creates severe credential-leakage and rotation risks. Enterprise workflows use **OpenID Connect (OIDC) Workload Identity Federation** to issue ephemeral, per-run credentials.

```text
+-----------------------------------------------------------------------------------+
|            OIDC WORKLOAD IDENTITY FEDERATION (ZERO STATIC SECRETS)                |
+-----------------------------------------------------------------------------------+
  [HCP Terraform / GitHub Actions]
         |
         | 1. Generates signed OIDC JWT (`sub`: org:corp:project:prod:workspace:net:run_phase:plan)
         v
  [Cloud IAM STS / Workload Identity Pool (AWS STS / GCP STS / Azure AD / Vault)]
         |
         | 2. Verifies JWT signature via JWKS + validates `aud` and `sub` claims
         | 3. Issues short-lived (15-60 min) STS Session Credentials
         v
  [Terraform Provider Plugin (aws / google / azurerm / vault)]
         +-- Executes API calls with Least-Privilege PlanRole or ApplyRole
```

### 2.1 Separating Plan vs. Apply IAM Roles in HCP Terraform Dynamic Credentials
HCP Terraform injects a signed OIDC JWT into every run where the `sub` claim explicitly includes `run_phase:plan` or `run_phase:apply`:
* **Subject Claim Format:** `organization:<ORG>:project:<PROJ>:workspace:<WS>:run_phase:<plan|apply>`
* **Security Best Practice:** Configure two separate IAM roles in AWS:
  1. **`TerraformPlanReadOnlyRole`:** Trusts only `...:run_phase:plan` so untrusted Pull Requests triggering speculative plans **cannot assume the write role** even if an attacker injects malicious `external` data source scripts or local-exec commands!
  2. **`TerraformApplyAdminRole`:** Trusts only `...:run_phase:apply`.
* **Required HCP Terraform Environment Variables for AWS Dynamic Credentials:**
  * `TFC_AWS_PROVIDER_AUTH = "true"`
  * `TFC_AWS_PLAN_ROLE_ARN = "arn:aws:iam::123456789012:role/TerraformPlanReadOnlyRole"`
  * `TFC_AWS_APPLY_ROLE_ARN = "arn:aws:iam::123456789012:role/TerraformApplyAdminRole"`

---

## 3. Dependency Lockfile Internals (`.terraform.lock.hcl`)

Committed directly to version control, `.terraform.lock.hcl` guarantees that every developer laptop and CI/CD runner installs the **exact same provider version and cryptographic binary checksum**.

### 3.1 `h1:` vs. `zh:` Cryptographic Checksum Schemes
Inside `.terraform.lock.hcl`, each `provider` block records two distinct types of hashes:
* **`zh:<hex>` (Zip Hash):** SHA-256 checksum of the **official `.zip` archive** distributed by the Terraform Registry (derived from the registry's signed `SHA256SUMS` file). A single `terraform init` populates `zh:` hashes for *all* platforms published in the registry release.
* **`h1:<base64>` (Package Directory Hash v1):** SHA-256 hash computed over the **unpacked contents** of the provider plugin binary for a specific OS/architecture.

### 3.2 Solving Cross-Platform Checksum Failures in CI/CD (`providers lock`)
* **Classic Failure Scenario:** An engineer on an Apple Silicon MacBook (`darwin_arm64`) adds a provider via a local filesystem cache or upgrades a provider without `zh:` coverage and commits `.terraform.lock.hcl` containing only the `darwin_arm64` `h1:` hash. When the Linux CI/CD runner (`linux_amd64`) runs `terraform init`, it fails with:
  `Error: Failed to install provider: the current package for registry.terraform.io/hashicorp/aws doesn't match any of the checksums previously recorded in the dependency lock file.`
* **Resolution:** Pre-populate `.terraform.lock.hcl` with `h1:` and `zh:` checksums across all developer and CI architectures:
  ```bash
  terraform providers lock \
    -platform=linux_amd64 \
    -platform=linux_arm64 \
    -platform=darwin_arm64 \
    -platform=windows_amd64
  ```

---

## 4. Provider Plugin Caching & Air-Gapped Mirrors

### 4.1 Local Plugin Cache (`TF_PLUGIN_CACHE_DIR`)
Downloading 500 MB of provider binaries (`aws`, `google`, `kubernetes`) on every CI job wastes bandwidth and time. Setting `TF_PLUGIN_CACHE_DIR="/var/cache/terraform/plugins"` (or `plugin_cache_dir` in `.terraformrc`) caches downloaded provider binaries across runs.
* **Critical Exam Caveats for `TF_PLUGIN_CACHE_DIR`:**
  1. **No Concurrent Write Safety:** The Terraform plugin cache directory is **NOT safe for concurrent writes**! If two parallel `terraform init` jobs on the same runner write the same provider binary into `TF_PLUGIN_CACHE_DIR` simultaneously, the binary can be corrupted. Pre-warm the cache during golden runner image builds (`packer`/`Docker`) rather than sharing a writable cache across parallel jobs.
  2. **Lockfile Interaction (`plugin_cache_may_break_dependency_lock_file`):** When installing a provider from `TF_PLUGIN_CACHE_DIR`, Terraform only has the unpacked binary (`h1:`), not the original registry `.zip` (`zh:`). If a new provider isn't already in `.terraform.lock.hcl`, Terraform refuses to use the cache unless `plugin_cache_may_break_dependency_lock_file = true` is set, because it would only record the single platform's `h1:` hash!

### 4.2 Air-Gapped Provider Installation Mirrors (`provider_installation`)
In regulated government/DOE or banking environments with zero outbound internet access to `registry.terraform.io`, configure `.terraformrc` (or point `TF_CLI_CONFIG_FILE` to a custom config file) with an explicit `provider_installation` block:

```hcl
provider_installation {
  # 1. Pull providers from an internal HTTPS network mirror (or Artifactory/Nexus)
  network_mirror {
    url     = "https://terraform-mirror.internal.corp/providers/"
    include = ["registry.terraform.io/*/*"]
  }

  # 2. Explicitly disable direct internet access to registry.terraform.io
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```
To build an offline filesystem bundle for transfer across an air gap:
```bash
terraform providers mirror -platform=linux_amd64 /mnt/usb-transfer/tf-mirror
```

---

## 5. Domain 4 Production Scenario & Exam Trap Matrix

* **Scenario 1: Passing `-var` to `terraform apply tfplan.binary`**
  * **Trap:** Running `terraform apply -var="env=prod" tfplan.binary`.
  * **Reality:** Terraform rejects `-var`, `-var-file`, and `-target` flags when applying a saved binary plan file (`Error: Can't set variables when applying a saved plan`). All variables must be passed during `terraform plan -out=tfplan.binary`.
* **Scenario 2: Untrusted Fork / Pull Request Executing Malicious Code During Speculative `terraform plan`**
  * **Trap:** Assuming `terraform plan` is strictly read-only and safe to run with production `AdministratorAccess` credentials on untrusted PR branches.
  * **Reality:** During `terraform plan`, Terraform executes `data "external"` scripts, local provider binaries, and reads sensitive secrets into memory! Never run speculative PR plans using production write credentials; isolate PR plans to a dedicated read-only OIDC role (`run_phase:plan`) and require maintainer review before triggering plans on external forks.

---

## 6. Platform Engineering Deep Dive: Internal Structure of `terraform show -json`

Platform engineers building custom CI/CD policy gates, automated PR commenters, FinOps cost calculators, or blast-radius analyzers inspect the JSON output of **`terraform show -json tfplan.binary`**. The Professional exam tests the exact top-level keys of this JSON representation:

```text
+-----------------------------------------------------------------------------------+
|            ANATOMY OF `terraform show -json tfplan.binary` OUTPUT                 |
+-----------------------------------------------------------------------------------+
  {
    "format_version": "1.2",
    "terraform_version": "1.11.0",
    "variables": { "env": { "value": "prod" } },
    "planned_values": {
      "Outputs": { ... },
      "root_module": { "resources": [...], "child_modules": [...] }
    },
    "resource_changes": [
      {
        "address": "module.network.aws_subnet.private[\"us-east-1a\"]",
        "type": "aws_subnet",
        "change": {
          "actions": ["delete", "create"],  // <-- Replacement!
          "before": { "cidr_block": "10.0.1.0/24" },
          "after":  { "cidr_block": "10.0.10.0/24" },
          "after_unknown": { "id": true, "arn": true },
          "before_sensitive": {},
          "after_sensitive": {}
        }
      }
    ],
    "resource_drift": [ ... ],   // <-- Out-of-band cloud changes detected during refresh!
    "prior_state": { ... },      // <-- Snapshot of .tfstate after refresh phase
    "configuration": { ... }     // <-- Static AST of HCL expressions, references & providers
  }
```

### 6.1 Decoding `change.actions` Arrays in `resource_changes`
* **`["no-op"]`:** Resource is unchanged.
* **`["create"]`:** New resource will be created.
* **`["read"]`:** Data source will be read during `apply` (because one of its inputs is unknown until `apply`).
* **`["update"]`:** Resource will be updated in-place without downtime.
* **`["delete"]`:** Resource will be destroyed.
* **`["delete", "create"]`:** Standard **Destroy-then-Create** replacement (`-/+`).
* **`["create", "delete"]`:** Zero-downtime **`create_before_destroy = true`** replacement (`+/-`)! *(Exam favorite: notice the array element order reverses from `["delete", "create"]` to `["create", "delete"]`!)*

---

## 7. Resolving `Error: Saved plan is stale` in Multi-Developer Monorepos

In high-velocity Platform Engineering teams where dozens of engineers open Pull Requests against a shared infrastructure repository:
* **Why `Saved plan is stale` Occurs:**
  * If Pipeline Run A generates `tfplan.binary` when the remote state is at `serial = 50`, and before Run A executes `terraform apply tfplan.binary`, Pipeline Run B completes an apply (incrementing the remote state to `serial = 51`), Run A's `terraform apply tfplan.binary` will immediately abort with:
    `Error: Saved plan is stale. The given plan file can no longer be applied because the state was changed by another operation after the plan was created.`
* **Platform Engineering Solutions:**
  1. **PR Merge Queue / Lock-on-PR (Atlantis / HCP Terraform Run Queue):** Serialize plan-and-apply execution per workspace so a new plan is always generated at the head of the queue after the prior run finishes.
  2. **Blast-Radius State Decomposition:** Split the monorepo into independent micro-stacks (`services/payments/`, `services/checkout/`) backed by separate `.tfstate` keys, using `git diff --name-only` path filters in GitHub Actions/GitLab CI so unrelated commits never contend for the same state serial.

