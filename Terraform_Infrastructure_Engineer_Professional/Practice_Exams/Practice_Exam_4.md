# Terraform Infrastructure Engineer Professional (002) — Practice Exam 4

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 30 Deep Scenario-Based Questions with Complete Distractor Analysis
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+ / HCP Terraform

---

### Questions

---

### Question 1 (Domain 1 — Scalable Configuration Design)
An enterprise module requires an input variable `cidr_block` to be a valid IPv4 CIDR string and within the private `10.0.0.0/8` range. Which variable validation expression is correct?

A.
```hcl
check = is_cidr(var.cidr_block)
```
B.
```hcl
validation = var.cidr_block in "10.0.0.0/8"
```
C.
```hcl
variable "cidr_block" {
  type = string
  validation {
    condition     = can(cidrnetmask(var.cidr_block)) && startswith(var.cidr_block, "10.")
    error_message = "Must be a valid IPv4 CIDR within 10.0.0.0/8"
  }
}
```
D.
```hcl
assert { ip == "10.*" }
```

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `can(cidrnetmask(var.cidr_block))` verifies valid IPv4 CIDR syntax, and `startswith()` verifies the prefix.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 2 (Domain 2 — State & Backend Operations)
During a team state inspection, an engineer runs `terraform state show aws_security_group.web` and discovers that a database password was written to state in plaintext. What is the most immediate security remediation step?

A. Rotate the database credential immediately upstream in the database and secrets manager, and ensure the Terraform configuration is updated to use dynamic or ephemeral credentials.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Plaintext secrets in state must be immediately invalidated/rotated upstream in the secrets manager, followed by adopting ephemeral/dynamic values.
- **Why B is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 3 (Domain 3 — Lifecycle Operations)
You have a resource with `lifecycle { ignore_changes = [tags["CostCenter"]] }`. A developer updates the resource tag `"Environment"` from `"dev"` to `"staging"` in the Terraform manifest, while an external AWS billing script updates the tag `"CostCenter"` from `"100"` to `"200"` in the AWS console. What will `terraform plan` propose?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. It throws a syntax error because map key indexing in `ignore_changes` is not allowed.
D. It proposes updating `"Environment"` to `"staging"` and does NOT propose changing `"CostCenter"`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Terraform's diff engine ignores attributes explicitly declared in `ignore_changes`, while processing updates to other unignored attributes.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 4 (Domain 4 — CI/CD Automation Workflows)
In a secure automated CI/CD pipeline, how should provider binaries be cached on the runner to reduce network bandwidth and prevent rate limiting from HashiCorp Registry?

A. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
B. By configuring a provider plugin cache directory using the environment variable `TF_PLUGIN_CACHE_DIR=/opt/terraform/plugin-cache`.
C. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
D. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `TF_PLUGIN_CACHE_DIR` tells Terraform to store and reuse downloaded provider binaries locally, eliminating redundant downloads.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 5 (Domain 5 — Security & Governance)
You need to enforce that all AWS S3 buckets created across your organization have AWS KMS encryption enabled (`aws:kms`). In Open Policy Agent (OPA) with Rego, how is this rule expressed against the `tfplan.json` output?

A.
```rego
package main
deny = true
```
B.
```rego
allow = true
```
C.
```rego
rule = "s3_kms_only"
```
D.
```rego
package terraform.security

deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_s3_bucket_server_side_encryption_configuration"
    rule := resource.change.after.rule[_]
    rule.apply_server_side_encryption_by_default.sse_algorithm != "aws:kms"
    msg := sprintf("S3 bucket %v must use aws:kms encryption", [resource.address])
}
```

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** The Rego rule inspects S3 server-side encryption configurations and verifies `sse_algorithm == "aws:kms"`.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 6 (Domain 6 — Advanced Modules & Testing)
In a `.tftest.hcl` test file, what occurs if a `run` block does NOT specify `command`?

A. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. It defaults to `command = apply`.
D. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** In `terraform test`, a `run` block defaults to `command = apply` if not explicitly specified.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 7 (Domain 7 — HCP Terraform & Enterprise)
What is the effect of an HCP Terraform workspace setting `Auto-apply: true`?

A. Once a VCS-driven plan completes successfully and all Sentinel / OPA policy checks pass, the changes are applied immediately without requiring manual human confirmation.
B. The workspace automatically fixes Terraform syntax errors when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `Auto-apply: true` executes the apply immediately upon a successful plan and passing policy checks.
- **Why B is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 8 (Domain 1 — Scalable Configuration Design)
You have a list of numbers: `[10, 20, 30]`. You need to calculate the sum of all elements. Which HCL expression accomplishes this?

A. `add([10, 20, 30])`
B. `sum([10, 20, 30])`
C. `total([10, 20, 30])`
D. `math.sum([10, 20, 30])`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `sum()` sums a list or set of numbers.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 9 (Domain 2 — State & Backend Operations)
In Terraform v1.5+, what command automatically generates HCL configuration files from resources imported using `import` blocks?

A. `terraform generate` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `terraform import -export=hcl`
C. `terraform plan -generate-config-out=generated.tf`
D. `terraform hcl-convert` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform plan -generate-config-out=generated.tf` generates HCL manifests from imported resources.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 10 (Domain 3 — Lifecycle Operations)
What happens if you declare two resources with identical identifiers in the same configuration, e.g.:
```hcl
resource "aws_s3_bucket" "data" {}
resource "aws_s3_bucket" "data" {}
```

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
D. Terraform fails during initial compilation with a duplicate resource definition error before generating a plan.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Resource blocks must have unique combinations of type and name within the same module scope.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 11 (Domain 4 — CI/CD Automation Workflows)
In an enterprise multi-tier architecture, what is the recommended way to share outputs (such as VPC ID and Subnet IDs) from the `networking` workspace with the `app-cluster` workspace?

A. Storing outputs in a shared text file on a network share when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
C. Using HCP Terraform / Terraform Enterprise `Run Triggers` and `terraform_remote_state` data sources or HCP Terraform cross-workspace state sharing.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** HCP Terraform workspace sharing and `terraform_remote_state` allow controlled, auditable consumption of outputs across boundaries.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 12 (Domain 5 — Security & Governance)
What is the function of the `tflint` tool in a platform engineering workflow?

A. It is a Pluggable Terraform Linter that catches provider-specific errors (e.g. invalid EC2 instance types, deprecated attributes, naming conventions) that `terraform validate` misses.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `tflint` catches provider-specific validation errors and style violations via AST analysis.
- **Why B is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 13 (Domain 6 — Advanced Modules & Testing)
When authoring integration tests with Terratest, what Go command executes the test suite?

A. `go build` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `terraform test`
C. `terratest run`
D. `go test -v -timeout 30m`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `go test` with a sufficient timeout (e.g., 30m) executes Terratest suites against real cloud APIs.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 14 (Domain 7 — HCP Terraform & Enterprise)
What is the purpose of `Workspace Tagging` in HCP Terraform?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. To categorize, filter, and organize workspaces, and to scope Variable Sets and notification webhooks to matching workspace tags.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Workspace tags categorize workspaces and facilitate targeted assignment of variable sets and notification policies.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 15 (Domain 1 — Scalable Configuration Design)
Which function converts an encoded JSON string into an HCL object/map?

A. `from_json()`
B. `jsonencode()`
C. `parse_json()`
D. `jsondecode()`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `jsondecode()` parses JSON into HCL native data structures.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 16 (Domain 2 — State & Backend Operations)
Under what scenario would an engineer need to run `terraform state replace-provider`?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. When switching from an obsolete or deprecated provider namespace to a new provider namespace (e.g. migrating `hashicorp/template` to `template_file` or moving custom providers to Upbound/community namespaces) in state.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `state replace-provider` updates the provider namespace recorded in existing state entries without recreation.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 17 (Domain 3 — Lifecycle Operations)
Why is `lifecycle { ignore_changes = [all] }` considered an operational anti-pattern for long-term managed infrastructure?

A. It permanently blinds Terraform to any and all drift occurring on the resource in the cloud, defeating the core value proposition of declarative Infrastructure-as-Code.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Ignoring all changes stops Terraform from detecting configuration drift, undermining IaC state fidelity.
- **Why B is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 18 (Domain 4 — CI/CD Automation Workflows)
In a GitHub Actions workflow, what environment variable must be set to authenticate to HCP Terraform when using the remote backend?

A. `HCP_PASSWORD` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `TF_TOKEN_app_terraform_io`
C. `TERRAFORM_SECRET_KEY`
D. `ATLAS_TOKEN` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `TF_TOKEN_<hostname>` (e.g. `TF_TOKEN_app_terraform_io`) supplies the authentication token for remote operations.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 19 (Domain 5 — Security & Governance)
When configuring HashiCorp Sentinel, what is the role of the `sentinel.hcl` file?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. It defines the policy set, policy sources, and enforcement levels (`advisory`, `soft-mandatory`, `hard-mandatory`).
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `sentinel.hcl` configures the policy suite and enforcement modes for HCP Terraform Sentinel checks.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 20 (Domain 6 — Advanced Modules & Testing)
In a `.tftest.hcl` file, how can a test author execute tests against an existing deployed staging environment without creating new resources?

A. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. By specifying `command = plan` and providing staging variable definitions.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `command = plan` against existing state runs assertions without provisioning new infrastructure.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 21 (Domain 7 — HCP Terraform & Enterprise)
In Terraform Enterprise, what is the recommended backup strategy for an Active/Active Flexible Deployment Option (FDO) installation?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Backing up the external PostgreSQL database and the external object storage (S3/Azure Blob/GCS bucket).
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** TFE Active/Active stores state and metadata in PostgreSQL and object storage; backing up those two components ensures complete recovery.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 22 (Domain 1 — Scalable Configuration Design)
What does `try()` function in HCL do?

A. It evaluates a list of expressions in sequence and returns the result of the first one that does not produce any errors.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `try()` returns the first expression that evaluates without an error.
- **Why B is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 23 (Domain 2 — State & Backend Operations)
What information does the `serial` integer represent in `terraform.tfstate`?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. A monotonic revision counter incremented each time the state file is successfully modified and saved by Terraform.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `serial` is incremented on every state modification to prevent out-of-order writes.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 24 (Domain 3 — Lifecycle Operations)
What does `terraform taint` do in legacy configurations (and how is it handled in modern Terraform)?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. It marks a resource for recreation on the next apply; in modern Terraform (v1.2+), this is accomplished via `terraform apply -replace=...` or `replace_triggered_by`.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Tainting flags a resource for deletion and recreation; modern Terraform uses `-replace`.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 25 (Domain 7 — HCP Terraform & Enterprise)
What is the function of `Speculative Plans` in HCP Terraform?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Ephemeral plans generated automatically on pull requests to predict changes and policy compliance without locking the workspace or allowing changes to be applied.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Speculative plans are read-only, non-locking plans generated on pull requests for preview and policy evaluation.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 26 (Domain 2 — State and Backend Operations)
Your platform security team is migrating 40 workspaces on Terraform 1.11 from legacy DynamoDB state locking to native Amazon S3 state locking (`use_lockfile = true`). To prevent accidental state deletion, a security engineer attaches an IAM Deny policy blocking `s3:DeleteObject` on `arn:aws:s3:::corp-prod-state/*`. What happens when the next CI/CD pipeline runs `terraform apply`?

A. The run succeeds completely because `use_lockfile = true` stores lock metadata inside S3 object tags rather than creating separate objects.
B. The run fails to acquire a lock during initialization because S3 conditional writes require `s3:DeleteBucket` permissions on the parent bucket.
C. The run acquires the lock and applies changes, but fails at the end of the run when attempting to release the `<key>.tflock` lock object via `s3:DeleteObject`, leaving a permanent orphaned lock.
D. Terraform automatically falls back to in-memory local locking and emits a non-blocking warning in the CLI output.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Native S3 state locking (`use_lockfile = true`) creates a `<key>.tflock` object using `s3:PutObject` with `If-None-Match: "*"` and removes that lock at the end of the run using `s3:DeleteObject`. Blocking `s3:DeleteObject` on `*.tflock` prevents Terraform from releasing the lock! The IAM Deny for `s3:DeleteObject` must be scoped strictly to `*.tfstate` while allowing `s3:DeleteObject` on `*.tflock`.
- **Why A, B, and D are Incorrect:** Lockfiles are distinct S3 objects (`<key>.tflock`), not object tags; `s3:DeleteBucket` is never used; and Terraform never silently downgrades distributed locking to local memory.

</details>

---

### Question 27 (Domain 5 — Security and Governance)
You are provisioning an `aws_db_instance` in Terraform 1.11 using an ephemeral password generated via `ephemeral "random_password" "db" {}` passed into the write-only attribute `password_wo`. Six months later, your security compliance policy requires rotating the database master password during the next `terraform apply`. Because `password_wo` is stored as `null` in `terraform.tfstate`, how do you instruct Terraform to rotate and transmit a new write-only password to RDS?

A. Run `terraform apply -refresh-only` so the provider compares the ephemeral memory value against the live RDS instance hash.
B. Increment the integer value assigned to the companion `password_wo_version` attribute on the `aws_db_instance` resource and run `terraform apply`.
C. Run `terraform plan -replace=ephemeral.random_password.db` to force the ephemeral resource to regenerate its state entry.
D. Temporarily set `ephemeral = false` on the `random_password` block for one run so the new secret hash is committed to `.tfstate`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Write-only attributes (`password_wo`) never persist values in `.tfstate`, so Terraform tracks rotation intent via the companion integer attribute `password_wo_version` (which *is* stored in `.tfstate`). Incrementing `password_wo_version` (e.g., from `1` to `2`) causes a state diff that triggers the provider to evaluate the `ephemeral` secret and send the new `password_wo` value to the RDS API.
- **Why A, C, and D are Incorrect:** `-refresh-only` cannot read write-only attributes; `ephemeral` resources do not exist in `.tfstate` and cannot be targeted with `-replace`; and `ephemeral` blocks do not have an `ephemeral = false` toggle.

</details>

---

### Question 28 (Domain 1 — Scalable Configuration Design)
In a module targeting Terraform 1.9+, you declare a variable with nested optional objects:
```hcl
variable "service_config" {
  type = object({
    name = string
    telemetry = optional(object({
      enabled       = optional(bool, true)
      retention_days = optional(number, 30)
    }))
  })
}
```
If a caller passes `service_config = { name = "checkout" }` and your module references `var.service_config.telemetry.enabled`, what is the result?

A. It evaluates to `true` because nested `optional()` defaults are automatically synthesized regardless of parent object presence.
B. It evaluates to `null` without raising an error because HCL automatically short-circuits attribute traversal on omitted optional objects.
C. Terraform fails during evaluation with `Attempt to get attribute from null value` because `telemetry` itself defaulted to `null` when omitted without an outer `, {}` default.
D. Terraform fails during `terraform init` with an invalid nested type constraint specification error.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Because `telemetry = optional(object({...}))` was declared with only one argument (no `, {}` default for the parent object), omitting `telemetry` sets `var.service_config.telemetry = null`. Referencing `.enabled` on `null` throws `Attempt to get attribute from null value`. To trigger inner defaults when `telemetry` is omitted, you must write `telemetry = optional(object({...}), {})`.
- **Why A, B, and D are Incorrect:** Inner defaults only evaluate when the enclosing object is non-null; HCL does not auto-short-circuit null attribute access unless wrapped in `try()`; and nested `optional()` is valid syntax.

</details>

---

### Question 29 (Domain 6 — Advanced Modules & Testing)
You are authoring unit tests in `tests/network.tftest.hcl` using Terraform 1.8+. One of your `run` blocks tests that passing an invalid CIDR string `"192.168.999.0/24"` to `var.vpc_cidr` is properly rejected by the variable's `validation` block. How must the `run` block be structured so `terraform test` reports `PASS` when the validation fails?

A. Configure `command = plan` and include `expect_failures = [var.vpc_cidr]` inside the `run` block without asserting on downstream resources.
B. Configure `command = apply` and add an `assert { condition = !can(var.vpc_cidr) }` block inside the `run` block.
C. Configure `mock_provider "aws" { enforce_validations = false }` and inspect `check_results` inside the test output.
D. Wrap the `variables` block inside `try()` so the validation exception is converted into a boolean test assertion.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** In `.tftest.hcl`, negative testing of custom validation rules, preconditions, postconditions, and check blocks requires declaring `expect_failures = [var.vpc_cidr]` inside the `run` block. The test passes if and only if the specified checkable object raises a validation error during the run.
- **Why B, C, and D are Incorrect:** If `var.vpc_cidr` fails validation without `expect_failures`, the `run` aborts before evaluating `assert` blocks and marks the test as `FAIL`.

</details>

---

### Question 30 (Domain 7 — HCP Terraform & Enterprise)
Your enterprise platform team wants to deploy an identical multi-component architecture (VPC, EKS, and RDS) across a dynamic set of AWS regions (`var.regions = ["us-east-1", "us-west-2", "eu-central-1"]`) where the `provider "aws"` configuration itself iterates using `for_each = var.regions`. Which Terraform capability supports iterating `provider` blocks with `for_each`?

A. Standard root module `providers.tf` files when `required_version = ">= 1.10.0"` and `configuration_aliases` are enabled.
B. Terraform Stacks (`*.tfstack.hcl` and `*.tfdeploy.hcl`) in HCP Terraform, which natively allow `for_each` on `provider` and `component` blocks.
C. Child modules called via `module "regional" { for_each = var.regions }` that contain an inline `provider "aws" { region = each.key }` block.
D. Dynamic provider blocks (`dynamic "provider" { for_each = var.regions ... }`) declared inside `terraform.required_providers`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Standard Terraform `.tf` files strictly prohibit `for_each` on `provider` blocks and forbid `for_each` on child modules that contain inline `provider` blocks. **Terraform Stacks (`*.tfstack.hcl`)** solve this at the orchestration layer by natively supporting `provider "aws" "regional" { for_each = var.regions ... }` and binding each regional provider instance to `component` blocks.
- **Why A, C, and D are Incorrect:** Standard `.tf` files never allow `for_each` on `provider` blocks; inline providers in child modules break `for_each` on the module call; and `dynamic "provider"` does not exist in HCL.

</details>

---
