# Terraform Infrastructure Engineer Professional (002) — Practice Exam 3

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 30 Deep Scenario-Based Questions with Complete Distractor Analysis
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+ / HCP Terraform

---

### Questions

---

### Question 1 (Domain 1 — Scalable Configuration Design)
An enterprise module receives a variable `subnets` containing a map of objects with CIDRs and AZs. You need to construct a filtered list of only the subnet IDs located in availability zone `"us-east-1a"`. Which HCL `for` expression is syntactically correct?

A. `filter(var.subnets, "az == us-east-1a")` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `{ for id, subnet in var.subnets : id if subnet.az == "us-east-1a" }`
C. `[ for id, subnet in var.subnets : id if subnet.az == "us-east-1a" ]`
D. `[ select id from var.subnets where az == "us-east-1a" ]`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `[ for id, subnet in var.subnets : id if subnet.az == "us-east-1a" ]` filters map elements based on a condition and returns a list of matching keys.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 2 (Domain 2 — State & Backend Operations)
During a team restructuring, an engineer splits a monolithic configuration into two independent workspaces: `networking` and `compute`. To migrate the VPC resources from the monolithic state file to the new `networking` state file without destroying cloud infrastructure, what sequence of state commands should be used?

A. Run `terraform state mv -state-out=../networking/terraform.tfstate module.vpc module.vpc` or extract and import the state.
B. `terraform destroy` in monolith, followed by `terraform apply` in networking.
C. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
D. `terraform refresh -force` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform state mv` allows moving resource state between distinct state files without touching physical infrastructure.
- **Why B is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 3 (Domain 3 — Lifecycle Operations)
An engineer modifies an `aws_iam_role` resource name from `"api-role-v1"` to `"api-role-v2"`. IAM roles have globally unique names within an AWS account. The resource block includes `lifecycle { create_before_destroy = true }`. What will happen when `terraform apply` executes?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. The apply fails because `"api-role-v2"` has a different name, so name collision does not occur.
C. Terraform fails because AWS IAM does not permit two roles with the same name, but since the names differ, creation succeeds before the old role is destroyed.
D. Terraform creates `"api-role-v2"` successfully and then destroys `"api-role-v1"`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** With `create_before_destroy = true` and different resource names, Terraform safely provisions the replacement before terminating the original.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 4 (Domain 4 — CI/CD Automation Workflows)
In an automated GitHub Actions pipeline, `terraform plan -out=tfplan` runs on pull requests, and `terraform apply tfplan` runs on merge to `main`. A developer merges a pull request 48 hours after opening it. In the meantime, another developer merged a PR that provisioned an S3 bucket with the identical name. What happens when the apply job executes?

A. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
B. The apply job fails because the cloud provider API returns a 409 Conflict / BucketAlreadyExists error, preserving state integrity without overwriting the existing bucket.
C. The existing bucket is silently overwritten and deleted when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Plan files capture intent, but execution fails safely if remote resources collide, preventing state corruption.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 5 (Domain 5 — Security & Governance)
You need to prevent developers from provisioning security groups that open port `22` (SSH) to `0.0.0.0/0`. In HashiCorp Sentinel, which rule definition correctly evaluates this condition across all planned security group rules?

A.
```sentinel
rule { block_all_ssh() }
```
B.
```sentinel
main = rule { aws_security_group.port == 22 }
```
C.
```sentinel
deny { port: 22 }
```
D.
```sentinel
import "tfplan/v2" as tfplan

violating_rules = filter tfplan.resource_changes as _, rc {
  rc.type is "aws_security_group_rule" and
  rc.change.after.from_port <= 22 and
  rc.change.after.to_port >= 22 and
  "0.0.0.0/0" in rc.change.after.cidr_blocks
}
main = rule { length(violating_rules) is 0 }
```

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Sentinel filters `resource_changes` in `tfplan/v2` and verifies no rule exposes port 22 to `0.0.0.0/0`.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 6 (Domain 6 — Advanced Modules & Testing)
In a `.tftest.hcl` file, you want to test whether a module properly creates 3 subnets when given a list of 3 CIDRs during `terraform test`. Which assertion condition verifies this?

A. `condition = size(output.subnet_ids) == 3`
B. `condition = count(output.subnet_ids) == 3`
C. `condition = length(output.subnet_ids) == 3`
D. `expect = 3` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `condition = length(output.subnet_ids) == 3` asserts that the output list contains exactly 3 elements.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 7 (Domain 7 — HCP Terraform & Enterprise)
What is the purpose of an HCP Terraform `OIDC Run Token`?

A. It is a short-lived JSON Web Token (JWT) injected into the execution container that allows the Terraform CLI to assume cloud provider IAM roles dynamically without long-lived static API secrets.
B. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
C. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** HCP Terraform OIDC tokens allow ephemeral authentication to AWS STS, Azure AD, and GCP Workload Identity.
- **Why B is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 8 (Domain 1 — Scalable Configuration Design)
You have a variable `cluster_nodes` with default `null`. You want to assign `local.node_count` to `var.cluster_nodes` if provided, or fallback to `3` if `null`. Which HCL function accomplishes this cleanly?

A. `default(var.cluster_nodes, 3)`
B. `coalesce(var.cluster_nodes, 3)`
C. `fallback(var.cluster_nodes, 3)`
D. `ternary(var.cluster_nodes, 3)`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `coalesce()` evaluates arguments sequentially and returns the first non-null/non-empty value.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 9 (Domain 2 — State & Backend Operations)
An engineer runs `terraform state list` and notices that an obsolete resource `aws_instance.old_bastion` is still tracked in state, but the physical VM was terminated manually in the cloud console weeks ago. How can state be synchronized without executing a full apply?

A. `terraform apply -auto-approve`
B. `terraform state delete aws_instance.old_bastion`
C. `terraform apply -refresh-only`
D. `terraform destroy`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform apply -refresh-only` reconciles state with external manual deletions without applying configuration diffs.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 10 (Domain 3 — Lifecycle Operations)
In Terraform v1.5+, what is the difference between a `precondition` and a `postcondition` in a resource lifecycle block?

A. `precondition` runs before `terraform init`; `postcondition` runs after `terraform apply` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `precondition` cannot reference other resources when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `postcondition` can only be used on data sources when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. `precondition` evaluates assumptions about inputs and configuration before creating/updating the resource; `postcondition` evaluates guarantees about the resource's resulting state after creation (such as verifying a public IP was assigned).

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Preconditions validate inputs before resource evaluation; postconditions validate attributes after creation.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 11 (Domain 4 — CI/CD Automation Workflows)
Why should `.terraform.lock.hcl` be committed to version control in production repositories?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
C. It records exact provider versions and cryptographic checksums, guaranteeing deterministic provider binaries across developer workstations and CI runners.
D. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `.terraform.lock.hcl` pins provider versions and platform hashes for supply chain immutability.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 12 (Domain 5 — Security & Governance)
What is the primary operational advantage of using Checkov or tfsec over Sentinel in open-source CI/CD workflows?

A. Checkov and tfsec are open-source and run locally in any standard CI pipeline without requiring an enterprise HCP Terraform license.
B. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
C. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Checkov and tfsec are open-source and run locally in any CI/CD runner.
- **Why B is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 13 (Domain 6 — Advanced Modules & Testing)
When creating a test file `main.tftest.hcl`, you want to mock the AWS provider so tests can execute in CI without AWS credentials. How is this defined in Terraform v1.7+?

A.
```hcl
fake_provider "aws" {}
```
B.
```hcl
provider "aws" { mock = true }
```
C.
```hcl
override_provider "aws" {}
```
D.
```hcl
mock_provider "aws" {}
```

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `mock_provider "aws" {}` enables unit testing without real credentials.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 14 (Domain 7 — HCP Terraform & Enterprise)
What is the primary function of HCP Terraform `Variable Sets`?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. To define a centralized bundle of Terraform and environment variables once and apply them globally or to selected projects and workspaces.
C. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
D. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Variable Sets enable centralized reuse of variable groups across workspaces and projects.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 15 (Domain 1 — Scalable Configuration Design)
What happens if you define a local value:
```hcl
locals {
  service_names = toset(["auth", "billing", "auth"])
}
```
What is `length(local.service_names)`?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
D. 2 (sets eliminate duplicates)

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Sets store unique elements; duplicate `"auth"` is deduplicated, leaving 2 items.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 16 (Domain 2 — State & Backend Operations)
An organization has an S3 state backend. To ensure maximum durability and disaster recovery capability, which three S3 features must be enabled on the state bucket?

A. S3 Glacier Direct Ingestion, BitTorrent, and Transfer Acceleration.
B. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
C. S3 Versioning, Server-Side Encryption (SSE-KMS), and Object Lock (or MFA Delete).
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** S3 versioning, SSE-KMS, and Object Lock provide durability and immutability for state files.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 17 (Domain 3 — Lifecycle Operations)
You need to ensure that an EC2 instance is replaced whenever a user-data bash script file on disk changes. Which HCL configuration achieves this in modern Terraform?

A.
```hcl
lifecycle {
  replace_triggered_by = [
    terraform_data.bootstrap.output
  ]
}
```
B.
```hcl
lifecycle {
  taint_on_change = true
}
```
C.
```hcl
lifecycle {
  recreate = file("script.sh")
}
```
D.
```hcl
lifecycle {
  destroy_on_update = true
}
```

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `replace_triggered_by` combined with `terraform_data` replaces resources when inputs or files change.
- **Why B is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 18 (Domain 4 — CI/CD Automation Workflows)
In Atlantis, what command is posted as a pull request comment to execute the approved plan against infrastructure?

A. `atlantis run`
B. `atlantis apply`
C. `atlantis execute`
D. `atlantis merge`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** In Atlantis, commenting `atlantis apply` executes the reviewed plan.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 19 (Domain 5 — Security & Governance)
What is the function of the `ephemeral` keyword in Terraform v1.10+?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
C. It declares ephemeral values and resources that exist purely in memory during execution and are never persisted to the state file on disk or remote backends.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Ephemeral values in Terraform 1.10+ prevent sensitive secrets from being stored in the state file.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 20 (Domain 6 — Advanced Modules & Testing)
In a module `README.md`, which tool automatically generates the inputs, outputs, requirements, and provider tables from HCL source code?

A. `terratest`
B. `tflint` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `terragrunt`
D. `terraform-docs`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `terraform-docs` generates standard Markdown interface tables from HCL ASTs.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 21 (Domain 7 — HCP Terraform & Enterprise)
What is the primary architectural purpose of deploying an `HCP Terraform Agent` in a corporate network?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
C. To execute Terraform runs against private cloud networks and on-premises data centers without opening inbound firewall ports from the Internet.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** HCP Terraform Agents poll outbound over HTTPS, executing workloads in private networks without inbound holes.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform & Enterprise constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 22 (Domain 1 — Scalable Configuration Design)
Which function removes `null` or empty string elements from a list?

A. `compact()`
B. `distinct()`
C. `flatten()`
D. `slice()`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `compact()` removes empty strings from a list of strings.
- **Why B is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 23 (Domain 2 — State & Backend Operations)
When migrating state from a local backend to an S3 backend using `terraform init -migrate-state`, what happens to the existing local `terraform.tfstate` file?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Terraform copies the state to S3 and preserves or creates a backup copy locally, leaving the local state empty or backed up as `terraform.tfstate.backup`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `init -migrate-state` transfers state to the remote store while backing up local state.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State & Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 24 (Domain 3 — Lifecycle Operations)
What does `prevent_destroy = true` do when placed inside a resource's `lifecycle` block?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. It causes Terraform to reject any plan that would destroy the physical infrastructure resource, acting as a critical safety guardrail against accidental deletion.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `prevent_destroy = true` aborts any plan proposing resource destruction.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Lifecycle Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 25 (Domain 5 — Security & Governance)
Why is hardcoding secrets inside `terraform.tfvars` considered a severe security risk even if the repository is private?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Cloud providers disable API keys if found in `.tfvars` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. `terraform.tfvars` files are easily leaked, committed accidentally, and stored in unencrypted Git history, exposing credentials to all repository viewers.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Storing plaintext secrets in files risks accidental commit and exposes credentials in version history.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security & Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
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
