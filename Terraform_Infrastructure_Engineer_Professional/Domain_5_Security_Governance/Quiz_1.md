# Domain 5: Terraform Security and Governance — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
An engineer marks a variable as `sensitive = true` in Terraform:
```hcl
variable "token" {
  type      = string
  sensitive = true
}
```
Does this prevent the token value from being written in plaintext inside the remote `.tfstate` file?

A. Yes, `sensitive = true` encrypts the value in the state file using AES-256 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Yes, Terraform deletes the value from state immediately after apply through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. No; `sensitive = true` only redacts the value in CLI output (`terraform plan` and `terraform apply`). The raw value is still written in plaintext inside the `.tfstate` JSON document unless the variable is declared as `ephemeral = true` or stored in an encrypted backend.
D. It hashes the token using SHA-256 in state while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `sensitive = true` is purely an output masking tool for CLI logs. It provides zero protection against state leakage; state encryption or `ephemeral = true` is required.
- **Why A is Incorrect:** `Yes, `sensitive = true` encrypts the value in the state file using AES-256 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Yes, Terraform deletes the value from state immediately after apply through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It hashes the token using SHA-256 in state while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
What is the function of the `ephemeral = true` variable attribute introduced in modern Terraform?

A. It guarantees that the value exists only in memory during graph evaluation and is strictly prohibited from being persisted to state files or saved plan files on disk.
B. The variable is automatically deleted after 24 hours during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. It allows the variable to be shared across all workspaces through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It turns the variable into a random string generator while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `ephemeral` ensures sensitive data vanishes after execution and is never saved to the state file.
- **Why B is Incorrect:** `The variable is automatically deleted after 24 hours during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It allows the variable to be shared across all workspaces through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It turns the variable into a random string generator while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
In HCP Terraform, what is the key operational difference between a **Soft Mandatory** Sentinel policy and a **Hard Mandatory** Sentinel policy?

A. Soft Mandatory policies only apply to staging; Hard Mandatory policies only apply to production during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Hard Mandatory policies cost more money while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. Soft Mandatory runs in advisory mode only through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Soft Mandatory policies halt the run on violation, but allow users with override permissions (e.g. SecOps) to approve and proceed; Hard Mandatory policies cannot be overridden under any circumstances and immediately terminate the run.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Soft Mandatory policies enforce a governance gate that can be bypassed by authorized personnel with explicit audit logging; Hard Mandatory policies strictly block.
- **Why A is Incorrect:** `Soft Mandatory policies only apply to staging; Hard Mandatory policies only apply to production during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Hard Mandatory policies cost more money while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Soft Mandatory runs in advisory mode only through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
An infrastructure team needs to generate short-lived, dynamic PostgreSQL database credentials for each deployment rather than hardcoding static admin passwords. Which tool natively integrates with Terraform to issue rotated, temporary database credentials on-demand?

A. Docker Compose by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. HashiCorp Vault
C. NGINX through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Git LFS while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** HashiCorp Vault dynamically generates database credentials with automatic Time-to-Live (TTL) expiration.
- **Why A is Incorrect:** `Docker Compose by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `NGINX through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Git LFS while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
When writing Open Policy Agent (OPA) policies in Rego to inspect planned infrastructure, what command converts a binary plan file (`tfplan`) into the JSON input format required by OPA?

A. `terraform show -json tfplan`
B. `terraform export -json` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `terraform plan -rego` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform convert tfplan.json` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform show -json <planfile>` outputs the complete machine-readable plan representation.
- **Why B is Incorrect:** ``terraform export -json` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform plan -rego` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform convert tfplan.json` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Which of the following configurations violates the CIS Infrastructure Security Benchmark for cloud networks?

A. Security group allowing inbound TCP 443 from `0.0.0.0/0` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Private subnets without Internet Gateways through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Security group allowing inbound TCP 22 (SSH) from `0.0.0.0/0`.
D. Storage buckets with server-side encryption enabled while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Exposing administrative SSH (port 22) or RDP (port 3389) to the entire internet (`0.0.0.0/0`) breaches standard security benchmarks.
- **Why A is Incorrect:** `Security group allowing inbound TCP 443 from `0.0.0.0/0` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Private subnets without Internet Gateways through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Storage buckets with server-side encryption enabled while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In Terraform 1.11+, how do **write-only arguments** enhance provider security?

A. They allow write access without read access on cloud IAM during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. They disable all `terraform destroy` commands while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. They prevent engineers from typing commands in the CLI through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. They enable providers to send sensitive values directly to remote APIs without Terraform storing those values in the state schema.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Write-only arguments prevent state persistence for provider-supported sensitive attributes.
- **Why A is Incorrect:** `They allow write access without read access on cloud IAM during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `They disable all `terraform destroy` commands while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `They prevent engineers from typing commands in the CLI through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What security risk arises if an engineer creates an S3 remote backend bucket without configuring `aws_s3_bucket_public_access_block`?

A. State files cannot be locked during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. An accidental misconfiguration or overly permissive IAM policy could expose sensitive credentials, API keys, and infrastructure topology contained in state to the public internet.
C. Terraform will refuse to run `init` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. DynamoDB tables will be deleted while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** State files are high-value targets containing complete architecture blueprints and embedded secrets.
- **Why A is Incorrect:** `State files cannot be locked during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform will refuse to run `init` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `DynamoDB tables will be deleted while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
How should IAM policies for CI/CD Terraform service roles be designed according to the principle of least privilege?

A. Grant `AdministratorAccess` to ensure builds never fail due to permission errors during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Grant read-only access to all resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Scope permissions strictly to the specific resource actions (e.g. `ec2:RunInstances`, `s3:CreateBucket`) and resource ARNs required by that specific workspace.
D. Assign the `root` account credentials to the runner while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Scoped, least-privilege IAM policies contain the blast radius if CI runner credentials are leaked.
- **Why A is Incorrect:** `Grant `AdministratorAccess` to ensure builds never fail due to permission errors during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Grant read-only access to all resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Assign the `root` account credentials to the runner while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
What tool can be executed in local developer pre-commit hooks to automatically scan HCL code for security misconfigurations (such as unencrypted buckets or open security groups) before code is pushed to Git?

A. Trivy, tfsec, or Checkov
B. Webpack by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. ESLint through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Kubernetes Kustomize while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Static application security testing (SAST) tools like Trivy, tfsec, and Checkov analyze HCL for known security anti-patterns.
- **Why B is Incorrect:** `Webpack by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `ESLint through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Kubernetes Kustomize while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
Your security team mandates that all modules called across your organization's 200 workspaces must be sourced exclusively from your internal HCP Terraform Private Registry (`app.terraform.io/corp-platform/*`) and must pin an explicit `version` attribute. Which Sentinel import must you use to inspect the module `source` and `version` attributes declared in the HCL files?

A. `import "tfplan/v2" as tfplan`
B. `import "tfconfig/v2" as tfconfig`
C. `import "tfstate/v2" as tfstate`
D. `import "tfrun" as tfrun`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** The `tfconfig/v2` Sentinel import exposes the static HCL configuration AST—including `tfconfig.modules`, their raw `source` strings, and `version` constraints—which are not preserved as module call blocks in `tfstate/v2` or `tfplan/v2.resource_changes`.
- **Why A, C, and D are Incorrect:** `tfplan/v2` inspects planned resource diffs/values, `tfstate/v2` inspects current state values, and `tfrun` inspects run metadata and cost estimates.

</details>

---

### Question 12
Previously, your team used `data "vault_database_secret_backend_creds" "postgres"` to generate dynamic 1-hour database credentials and passed them to `aws_db_instance.main.password`. Why did this cause both security compliance violations and operational drift on subsequent runs, and how does Terraform 1.11 resolve it?

A. The `data` source cached the temporary credential in plaintext inside `terraform.tfstate`, and once Vault revoked the 1-hour lease, the credential in state was both leaked and expired; Terraform 1.11 resolves this by combining an `ephemeral` secret block with the write-only `password_wo` and `password_wo_version` attributes.
B. Vault `data` sources only execute during `terraform init`; Terraform 1.11 resolves this by moving Vault authentication into `.terraform.lock.hcl`.
C. The `data` source encrypted the entire `.tfstate` file with a temporary Vault transit key that expired after 1 hour; Terraform 1.11 disables state encryption for Vault resources.
D. Vault `data` sources cannot be used with RDS; Terraform 1.11 replaces Vault with native `provider::aws::generate_password()` functions.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Every `data` source persists its returned attributes into `terraform.tfstate` in plaintext and refreshes on every plan (triggering constant password updates on `aws_db_instance` whenever a dynamic lease rotates). Using an `ephemeral` block paired with `password_wo` and `password_wo_version` keeps the secret strictly in runtime memory and rotates only when `password_wo_version` is incremented.
- **Why B, C, and D are Incorrect:** `data` sources run during `plan`/`apply`, never encrypt `.tfstate`, and are fully supported alongside `ephemeral` blocks.

</details>

---

### Question 13
In HCP Terraform, a FinOps Sentinel policy checks that `tfrun.cost_estimate.delta_monthly_cost <= 1000.00`. Business leadership requires that if a deployment exceeds $1,000/month, the `apply` must be blocked by default, but an authorized Engineering VP or FinOps Manager must be able to review and approve the run in the UI with an audited justification comment without changing the Sentinel code. Which enforcement level is required?

A. `advisory`
B. `soft-mandatory`
C. `hard-mandatory`
D. `conditional-override`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `soft-mandatory` blocks the run when a policy fails, but allows users with explicit Policy Override permissions to approve the override with an audit log comment.
- **Why A is Incorrect:** `advisory` never blocks the run; it only logs a warning.
- **Why C is Incorrect:** `hard-mandatory` blocks the run unconditionally with zero override capability.
- **Why D is Incorrect:** `conditional-override` is not a valid Sentinel/OPA enforcement level.

</details>

---

### Question 14
You are authoring an Open Policy Agent (OPA) Rego policy for HCP Terraform that inspects the planned infrastructure changes. Under which built-in Rego input path does HCP Terraform expose the Terraform plan's `resource_changes` array during policy evaluation?

A. `input.plan.resource_changes`
B. `data.tfplan.v2.resource_changes`
C. `input.terraform.state.resources`
D. `tfconfig.module.resource_changes`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** In HCP Terraform's OPA integration, the JSON plan payload is mounted under `input.plan` (so planned resource changes are accessed at `input.plan.resource_changes`), while run metadata is mounted under `input.run`.
- **Why B, C, and D are Incorrect:** `tfplan/v2` and `tfconfig` are Sentinel import namespaces, not OPA Rego `input` paths.

</details>

---

### Question 15
An engineer marks an input variable `variable "api_token" { type = string, sensitive = true }` and passes it into `local.auth_header = "Bearer ${var.api_token}"`. When the engineer attempts to output `local.auth_header` in an `output "header" { value = local.auth_header }` block without adding `sensitive = true` to the `output` block, what happens?

A. Terraform automatically strips the sensitivity mark because `var.api_token` was concatenated into a new string in `locals`.
B. Terraform propagates the sensitivity mark to `local.auth_header` and fails during `validate`/`plan` with `Error: Output refers to sensitive values` until `sensitive = true` (or `nonsensitive()`) is added to the `output` block.
C. Terraform prints `Bearer (sensitive value)` in the CLI output and succeeds without error.
D. Terraform converts the output into an `ephemeral` output automatically.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Terraform tracks sensitivity dynamically across expressions (taint propagation). Any expression derived from a `sensitive = true` value becomes sensitive itself, and exposing a sensitive value in an `output` block without explicitly setting `sensitive = true` on that output fails validation.
- **Why A, C, and D are Incorrect:** String interpolation preserves sensitivity, and Terraform requires explicit opt-in (`sensitive = true` or `nonsensitive()`) on outputs.

</details>

---
