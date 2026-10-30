# Terraform Infrastructure Engineer Professional (002) — Practice Exam 2

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 25 Deep Scenario-Based Questions with Complete Distractor Analysis
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+

---

### Questions

---

### Question 1 (Domain 1 — Scalable Configuration Design)
An engineer needs to transform a list of objects into a map keyed by the object's `name` attribute:
```hcl
variable "environments" {
  type = list(object({
    name = string
    cidr = string
  }))
}
```
Which expression correctly produces the keyed map?

A. `tomap(var.environments)` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `[ for e in var.environments : e.name ]`
C. `{ for e in var.environments : e.name => e }`
D. `map(var.environments.name, var.environments)`

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
An enterprise is migrating 250 virtual machines and VPCs from a local state file to an AWS S3 remote backend. What is the required command to migrate the existing state data without losing resource tracking?

A. `terraform init -migrate-state`
B. `terraform apply -migrate`
C. `terraform state push -force`
D. `terraform refresh`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 3 (Domain 3 — Infrastructure Lifecycle & Operations)
A database administrator needs to ensure that an accidental `terraform destroy` command never terminates the primary production database. Which configuration block guarantees this protection?

A. `prevent_deletion = true`
B. `lifecycle { create_before_destroy = true }`
C. `lifecycle { ignore_changes = all }`
D.
```hcl
lifecycle {
  prevent_destroy = true
}
```

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 4 (Domain 4 — CI/CD & Automation Workflows)
In a GitHub Actions pipeline, what parameter must be passed to `aws-actions/configure-aws-credentials` to assume an IAM role using OIDC rather than static access keys?

A. `aws-access-key-id` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `role-to-assume` (specifying the IAM role ARN configured with the GitHub OIDC trust policy)
C. `admin-token` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. `session-password` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 4 — CI/CD & Automation Workflows without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD & Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 5 (Domain 5 — Security and Governance)
A security auditor discovers that an API secret was printed in cleartext in the CI/CD job logs during `terraform plan`. What attribute was omitted from the variable declaration?

A. `ephemeral = true`
B. `private = true`
C. `hidden = true`
D. `sensitive = true`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 6 (Domain 6 — Advanced Modules & Testing)
When writing a `terraform test` file, which block specifies the sequence of test steps to execute?

A. `step`
B. `test_case`
C. `run`
D. `stage`

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
What is the primary operational benefit of creating a "Project" in HCP Terraform?

A. It organizes related workspaces into a shared administrative boundary where Team permissions and Variable Sets can be inherited hierarchically.
B. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

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
What is the evaluated output of `compact(["prod", "", null, "dev", "staging"])`?

A. `["prod", "", "dev", "staging"]`
B. `["prod", "dev", "staging"]`
C. `["prod", null, "dev", "staging"]`
D. `3` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

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
An engineer runs `terraform state list` and sees: `module.network.aws_vpc.main`. They want to inspect all recorded attributes (ID, CIDR, route table associations) of this specific resource in state. Which command displays this detail?

A. `terraform state pull --filter vpc`
B. `terraform state inspect main` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `terraform state show 'module.network.aws_vpc.main'`
D. `terraform get vpc` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

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
When importing existing cloud resources into Terraform, what is the main advantage of the declarative `import` block over the legacy `terraform import` CLI command?

A. The legacy CLI command has been deleted in Terraform 1.0 when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. The `import` block does not require provider authentication when evaluating the resource dependency graph during the plan and apply lifecycle phases.
D. The `import` block can automatically generate the matching HCL resource configuration via `terraform plan -generate-config-out=...` and can be peer-reviewed in Git pull requests.

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
If an input variable is declared in `terraform.tfvars` as `env = "staging"`, but the pipeline executes:
`export TF_VAR_env="production"`
What value does Terraform use?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. `"production"` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `"staging"` (because `terraform.tfvars` has higher precedence than `TF_VAR_` environment variables)
D. `"default"` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

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
What is the effect of setting a Sentinel policy to **Hard Mandatory** in an HCP Terraform organization?

A. If the policy fails evaluation during a run, the apply is completely blocked and cannot be overridden by anyone, regardless of their role.
B. The policy failure can be overridden by workspace administrators when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.
D. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 13 (Domain 6 — Advanced Modules & Testing)
An integration test in `tests/integration.tftest.hcl` has `command = apply`. What happens to the provisioned cloud resources after the test suite completes?

A. The resources remain running and must be manually deleted when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Terraform automatically executes a teardown/destroy phase to delete all resources created during the test run.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 6 — Advanced Modules & Testing without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 14 (Domain 7 — HCP Terraform / Enterprise Operations)
An organization has an internal compliance rule: "No infrastructure run may deploy resources that increase monthly spend by more than $500 without FinOps approval." How is this automated in HCP Terraform?

A. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
B. By enabling Cost Estimation and attaching a Sentinel or OPA policy that inspects the estimated monthly delta attribute.
C. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 7 — HCP Terraform / Enterprise Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 7 — HCP Terraform / Enterprise Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 15 (Domain 1 — Scalable Configuration Design)
Which function parses a YAML-formatted string into a native Terraform map/object data structure?

A. `jsondecode()`
B. `tobool()`
C. `yamlencode()`
D. `yamldecode()`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 1 — Scalable Configuration Design without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 1 — Scalable Configuration Design constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 16 (Domain 2 — State and Backend Operations)
You have refactored a standalone resource `digitalocean_droplet.web` into a child module `module.compute.digitalocean_droplet.web`. What command updates the state file to prevent Terraform from destroying the live Droplet?

A. `terraform refresh` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
B. `terraform state rm digitalocean_droplet.web` when evaluating the resource dependency graph during the plan and apply lifecycle phases.
C. `terraform state mv digitalocean_droplet.web module.compute.digitalocean_droplet.web`
D. `terraform apply -force` when evaluating the resource dependency graph during the plan and apply lifecycle phases.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 2 — State and Backend Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 2 — State and Backend Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 17 (Domain 3 — Infrastructure Lifecycle & Operations)
Which `lifecycle` meta-argument ignores differences between code and real-world infrastructure for specified attributes, preventing configuration drift flapping?

A. `ignore_changes`
B. `prevent_destroy`
C. `create_before_destroy`
D. `replace_triggered_by`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Option A accurately implements the required Terraform architecture and lifecycle behavior for Domain 3 — Infrastructure Lifecycle & Operations without introducing state drift, lock contention, or security vulnerabilities.
- **Why B is Incorrect:** Fails to satisfy the Domain 3 — Infrastructure Lifecycle & Operations constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 18 (Domain 4 — CI/CD & Automation Workflows)
Why should production CI/CD pipelines use `-no-color` when executing `terraform plan` and `terraform apply`?

A. Configure `terraform init -reconfigure -upgrade` inside the root module to bypass state lineage verification and force immediate schema reconciliation across all child modules.
B. It removes ANSI escape characters from the output, preventing corrupted log parsing in log aggregators and pull request comment widgets.
C. Declare a `data "terraform_remote_state"` lookup pointing to the local `.terraform/terraform.tfstate` cache and wrap the expression inside `nonsensitive(try(...))` during plan evaluation.
D. Pass the `-refresh=false` and `-compact-warnings` flags during `terraform apply` so the provider plugin skips read-after-write verification and commits the cached graph state directly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Option B accurately implements the required Terraform architecture and lifecycle behavior for Domain 4 — CI/CD & Automation Workflows without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 4 — CI/CD & Automation Workflows constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why C is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 19 (Domain 5 — Security and Governance)
What is a key security risk of storing production `.tfstate` files in a local directory or non-versioned S3 bucket without access controls?

A. Add `lifecycle { ignore_changes = all }` alongside `prevent_destroy = false` on the target resource block so out-of-band API mutations are merged automatically during `terraform validate`.
B. Define an inline `provider` configuration block with `alias = "override"` inside the reusable child module and trigger a state refresh via `terraform refresh -target=module.core`.
C. The state file contains complete architecture topology and plaintext attribute secrets (passwords, private keys, tokens) that could be read by unauthorized actors.
D. Set `TF_PLUGIN_CACHE_MAY_BREAK_DEPENDENCY_LOCK_FILE=true` and run `terraform state push -force` to synchronize computed attributes across remote workspaces without acquiring a state lock.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Option C accurately implements the required Terraform architecture and lifecycle behavior for Domain 5 — Security and Governance without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 5 — Security and Governance constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why D is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 20 (Domain 6 — Advanced Modules & Testing)
In a `*.tftest.hcl` file, which block allows you to supply static mock attributes for data sources when running with `mock_provider`?

A. `dummy_data`

---

#### 21–60: Rapid High-Value Core Invariants
- **21.** `terraform.tfvars` (Rank 3) overrides `TF_VAR_` environment variables (Rank 2).
- **22.** `*.auto.tfvars` (Rank 5) overrides `terraform.tfvars` (Rank 3).
- **23.** Command line `-var` (Rank 7) overrides all other variable sources.
- **24.** `ephemeral = true` variables are strictly excluded from state files and saved plans.
- **25.** Write-only arguments in Terraform 1.11+ are dispatched to cloud APIs and not stored in state.
- **26.** `terraform test` supports both `command = plan` (unit tests) and `command = apply` (integration tests).
- **27.** `expect_failures` tests negative validation in `terraform test`.
- **28.** `mock_provider` enables offline unit testing without cloud credentials.
- **29.** `terraform force-unlock <ID>` clears orphaned state locks after crashes.
- **30.** `terraform plan -refresh-only` detects out-of-band drift safely.
- **31.** `terraform apply -refresh-only` updates state to match out-of-band changes.
- **32.** S3 state backends require DynamoDB tables with hash key `LockID` for state locking.
- **33.** GCS and Azure Blob backends support native distributed state locking without auxiliary databases.
- **34.** `terraform state rm` decouples resources from state without deleting cloud resources.
- **35.** `terraform state mv` refactors resource addresses without recreation downtime.
- **36.** Circular dependencies between resources cause `Cycle error` exceptions during DAG construction.
- **37.** `create_before_destroy = true` must be propagated to upstream dependencies to prevent cycle errors.
- **38.** `prevent_destroy = true` halts plans that would destroy critical stateful resources.
- **39.** Declarative `import` blocks allow code-reviewed infrastructure imports in Git.
- **40.** Declarative `removed { lifecycle { destroy = false } }` removes resources from state cleanly.
- **41.** `terraform providers lock` records cryptographic package hashes across multiple OS architectures.
- **42.** Speculative plans on pull requests are read-only and never acquire state locks.
- **43.** OIDC identity federation eliminates long-lived static cloud keys in CI/CD secret stores.
- **44.** Scoping OIDC `sub` claims to `ref:refs/heads/main` ensures only merged commits can deploy.
- **45.** Concurrency in DAG walking can be tuned using `-parallelism=N`.
- **46.** Soft Mandatory Sentinel policies allow authorized SecOps overrides; Hard Mandatory policies strictly block.
- **47.** Open Policy Agent (OPA) evaluates Rego rules against `terraform show -json` output.
- **48.** HashiCorp Vault dynamically generates rotated cloud and database credentials with TTLs.
- **49.** Child modules must never declare hardcoded `provider` blocks; use `required_providers`.
- **50.** Child modules declare expected provider aliases via `configuration_aliases`.
- **51.** SemVer major version bumps (`X.0.0`) are required when breaking module input/output contracts.
- **52.** Public registry modules must follow `terraform-<PROVIDER>-<NAME>`.
- **53.** Private registry modules follow `<HOST>/<ORG>/<NAME>/<PROVIDER>`.
- **54.** HCP Terraform Project-scoped Variable Sets cascade variables to all child workspaces.
- **55.** `tfc-agent` runners use outbound HTTPS (port 443) polling, requiring no inbound firewall ports.
- **56.** Cost Estimation evaluates planned resources against cloud pricing APIs before apply.
- **57.** Run Tasks pass speculative plans to third-party security and cost partners before apply.
- **58.** `can()` returns `true` or `false`; `try()` returns the first valid expression without errors.
- **59.** `cidrsubnet(prefix, newbits, netnum)` calculates subnets deterministically.
- **60.** Segmenting state files by lifecycle and blast radius prevents catastrophic cascading outages.
B. `fake_values`
C. `fixtures`
D. `mock_data`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Option D accurately implements the required Terraform architecture and lifecycle behavior for Domain 6 — Advanced Modules & Testing without introducing state drift, lock contention, or security vulnerabilities.
- **Why A is Incorrect:** Fails to satisfy the Domain 6 — Advanced Modules & Testing constraint or relies on invalid HCL/CLI evaluation mechanics.
- **Why B is Incorrect:** Introduces state desynchronization, unnecessary resource replacement, or violates least-privilege governance.
- **Why C is Incorrect:** Misapplies Terraform phase ordering (`init` vs. `plan` vs. `apply`) or uses an unsupported configuration pattern.

</details>

---

### Question 21 (Domain 2 — State and Backend Operations)
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

### Question 22 (Domain 5 — Security and Governance)
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

### Question 23 (Domain 1 — Scalable Configuration Design)
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

### Question 24 (Domain 6 — Advanced Modules & Testing)
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

### Question 25 (Domain 7 — HCP Terraform & Enterprise)
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
