# Terraform Infrastructure Engineer Professional (002) — Practice Exam 1

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 35 Deep Scenario-Based Questions with Complete Distractor Analysis
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+

---

### Questions

---

### Question 1 (Domain 1 — Scalable Configuration Design)
An engineer writes the following variable definition:
```hcl
variable "node_pools" {
  type = map(object({
    size       = string
    count      = number
    autoscale  = optional(bool, false)
  }))
}
```
If a caller supplies:
```hcl
node_pools = {
  "compute" = { size = "s-4vcpu-8gb", count = 5 }
}
```
What is the evaluated value of `var.node_pools["compute"].autoscale`?

A. `null`
B. `true`
C. `false`
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 1 — Scalable Configuration Design without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 2 (Domain 2 — State and Backend Operations)
A CI pipeline running `terraform apply` crashes unexpectedly. When a subsequent run starts, it fails with:
`Error: Error acquiring the state lock: ConditionalCheckFailedException: Lock Info: ID: a819b42a-9938-4e12-b102-398418293847`.
After verifying that no other process is actively running, what is the correct command to clear the lock?

A. `terraform force-unlock a819b42a-9938-4e12-b102-398418293847`
B. `terraform unlock --force` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `terraform state clear-lock` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. `terraform init -reconfigure` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 3 (Domain 5 — Security and Governance)
In Terraform 1.10+, an engineer wants to ensure that a database master password retrieved dynamically from Vault is never written to disk or recorded in the `.tfstate` JSON file. How should the variable or data source be configured?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
D. By adding `ephemeral = true` (or using an `ephemeral` block), which ensures the value resides only in memory during execution and is omitted from the state file.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 4 (Domain 3 — Infrastructure Lifecycle & Operations)
You need to import an existing DigitalOcean Droplet with ID `394819284` into your Terraform configuration using modern declarative syntax. Which HCL block achieves this?

A. `terraform import digitalocean_droplet.web 394819284` in `main.tf`
B.
```hcl
import {
  to = digitalocean_droplet.web
  id = "394819284"
}
```
C. `data "digitalocean_droplet" "web" { id = "394819284" }`
D. `resource "import" "droplet" { id = "394819284" }`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 5 (Domain 4 — CI/CD & Automation Workflows)
Which of the following sources has the absolute highest precedence when evaluating Terraform input variables?

A. `terraform.tfvars` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `*.auto.tfvars` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `TF_VAR_<name>` environment variable
D. `-var="name=value"` CLI argument

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 4 — CI/CD & Automation Workflows without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD & Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 6 (Domain 6 — Advanced Modules & Testing)
In a `*.tftest.hcl` test suite, what does `mock_provider "aws" {}` accomplish?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
C. It mocks provider operations in-memory, allowing `command = plan` test runs to validate HCL logic and assertions completely offline without real AWS credentials.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 6 — Advanced Modules & Testing without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 7 (Domain 7 — HCP Terraform / Enterprise Operations)
An enterprise needs to execute Terraform plans and applies against private databases inside a dark datacenter with zero inbound internet connectivity. Which HCP Terraform execution mode must be selected?

A. Agent Execution (using `tfc-agent`)
B. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
C. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
D. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 7 — HCP Terraform / Enterprise Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform / Enterprise Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 8 (Domain 1 — Scalable Configuration Design)
An engineer evaluates `cidrsubnet("10.0.0.0/16", 8, 2)`. What is the resulting CIDR block?

A. `10.0.0.2/24`
B. `10.0.2.0/24`
C. `10.0.8.2/24`
D. `10.0.16.0/24`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 1 — Scalable Configuration Design without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 9 (Domain 2 — State and Backend Operations)
An engineer changes the name of a resource block from `digitalocean_firewall.old_rules` to `digitalocean_firewall.cluster_rules`. If they apply this change without state surgery, what will happen?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
C. Terraform destroys the old firewall and provisions a new one.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 10 (Domain 3 — Infrastructure Lifecycle & Operations)
What is the effect of configuring `lifecycle { prevent_destroy = true }` on a database resource?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
D. Any `terraform destroy` or plan that would result in the resource's destruction is rejected during the planning phase, halting execution with an error.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 11 (Domain 4 — CI/CD & Automation Workflows)
How does OpenID Connect (OIDC) enhance security for GitHub Actions pipelines running Terraform against cloud providers?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
C. It eliminates long-lived static API access keys in repository secrets by exchanging signed, short-lived OIDC tokens for temporary cloud credentials.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 4 — CI/CD & Automation Workflows without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD & Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 12 (Domain 5 — Security and Governance)
Which Sentinel enforcement level halts a run on policy failure, but allows an authorized user (e.g. SecOps Lead) to approve and override the failure?

A. Soft Mandatory
B. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
C. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
D. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 13 (Domain 1 — Scalable Configuration Design)
When configuring a child module that requires an aliased provider from the parent module, where and how must the alias requirement be declared inside the child module?

A. Inside the child module's `provider` block using `alias = "..."`.
B. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Inside `terraform.required_providers.<provider>.configuration_aliases`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 1 — Scalable Configuration Design without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 14 (Domain 2 — State and Backend Operations)
An operator manually modified an EC2 instance type from `t3.medium` to `t3.large` in the AWS console. The team agrees to keep this change in state without modifying other resources. What is the modern, safe workflow to update state?

A. `terraform refresh` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `terraform plan -refresh-only` followed by `terraform apply -refresh-only`
C. `terraform state rm` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. `terraform import` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 15 (Domain 6 — Advanced Modules & Testing)
What happens if an assertion inside a `terraform test` file fails?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. The test command fails with a non-zero exit code, printing the custom `error_message` and assertion details, which fails the CI pipeline.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 6 — Advanced Modules & Testing without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 16 (Domain 3 — Infrastructure Lifecycle & Operations)
Which `lifecycle` meta-argument ensures that a replacement virtual machine is fully provisioned and healthy before the existing virtual machine is terminated?

A. `replace_triggered_by`
B. `prevent_destroy = true`
C. `create_before_destroy = true`
D. `ignore_changes = all`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 17 (Domain 5 — Security and Governance)
A Rego policy for Open Policy Agent (OPA) evaluates which data format generated by Terraform?

A. JSON plan representation generated via `terraform show -json <planfile>`.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 18 (Domain 7 — HCP Terraform / Enterprise Operations)
Where are shared variables applied when scoped to an HCP Terraform "Project"?

A. To all workspaces across all organizations in the account.
B. Automatically to all current and future workspaces residing inside that specific Project.
C. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
D. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 7 — HCP Terraform / Enterprise Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform / Enterprise Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 19 (Domain 1 — Scalable Configuration Design)
What is the role of the Terraform Model Context Protocol (MCP) server introduced in Terraform 1.15+?

A. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. It provides standardized JSON-RPC schemas, documentation, and validation hooks to local AI coding agents.
D. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 1 — Scalable Configuration Design without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 20 (Domain 2 — State and Backend Operations)
Which command removes a resource from the Terraform state file without issuing an API call to delete the real cloud resource?

A. `terraform destroy -target`
B. `terraform delete`
C. `terraform state drop`
D. `terraform state rm`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 21 (Domain 4 — CI/CD & Automation Workflows)
Two files exist in the same directory: `01-dev.auto.tfvars` setting `port = 8080`, and `99-prod.auto.tfvars` setting `port = 443`. What is the evaluated value of `var.port`?

A. `8080` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
C. `443` (because files matching `*.auto.tfvars` are loaded in alphabetical order, later files override earlier files)
D. `null` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 4 — CI/CD & Automation Workflows without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD & Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 22 (Domain 3 — Infrastructure Lifecycle & Operations)
What does the declarative `removed` block do when configured with `destroy = false`?

A. It removes the resource from `.tfstate` while leaving the real cloud resource running.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 23 (Domain 6 — Advanced Modules & Testing)
Under Semantic Versioning, what version bump is required when you remove an input variable or rename an output in a shared Terraform module?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
C. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
D. Major (`v2.0.0`)

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 6 — Advanced Modules & Testing without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 24 (Domain 7 — HCP Terraform / Enterprise Operations)
What is the function of a "Run Task" in HCP Terraform?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. It integrates external third-party security, compliance, or cost tools (e.g. Snyk, Infracost) directly into the plan/apply lifecycle.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 7 — HCP Terraform / Enterprise Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform / Enterprise Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 25 (Domain 1 — Scalable Configuration Design)
Which function returns `true` if an expression can be evaluated without errors, and `false` otherwise?

A. `try()`
B. `validate()`
C. `coalesce()`
D. `can()`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 1 — Scalable Configuration Design without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 26 (Domain 2 — State and Backend Operations)
What happens if two engineers simultaneously run `terraform apply` against a backend that supports distributed locking (e.g. S3 with DynamoDB)?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
C. The first process acquires the lock; the second process fails immediately with a lock error or waits until the lock is released.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 27 (Domain 5 — Security and Governance)
Why is running Terraform with AWS `AdministratorAccess` considered a major security risk?

A. If CI runner credentials or OIDC claims are compromised, attackers gain unrestricted root-equivalent access to the entire cloud organization.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 28 (Domain 3 — Infrastructure Lifecycle & Operations)
When executing `terraform plan -out=tfplan`, what is contained inside the binary `tfplan` file?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. The exact planned actions, resource attributes, configuration snapshot, and state dependencies computed at plan time.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 29 (Domain 4 — CI/CD & Automation Workflows)
Why should `terraform apply` in an automated production pipeline specify the pre-generated plan file (`terraform apply tfplan`)?

A. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. It prevents Terraform from re-evaluating external variables or picking up concurrent out-of-band drift between the review step and apply step.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 4 — CI/CD & Automation Workflows without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD & Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 30 (Domain 6 — Advanced Modules & Testing)
How do you assert that a specific variable validation rule fails as expected in a `terraform test` file?

A. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. By adding an `expect_failures = [var.<variable_name>]` block inside the `run` block.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 6 — Advanced Modules & Testing without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 31 (Domain 2 — State and Backend Operations)
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

### Question 32 (Domain 5 — Security and Governance)
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

### Question 33 (Domain 1 — Scalable Configuration Design)
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

### Question 34 (Domain 6 — Advanced Modules & Testing)
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

### Question 35 (Domain 7 — HCP Terraform & Enterprise)
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
