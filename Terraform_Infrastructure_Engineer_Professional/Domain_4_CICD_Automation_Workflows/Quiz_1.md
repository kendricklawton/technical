# Domain 4: CI/CD and Automation Workflows — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
An engineer defines `variable "replica_count" { default = 2 }`. The following files and settings are present during pipeline execution:
1. `TF_VAR_replica_count = 5` in the shell environment.
2. `terraform.tfvars` containing `replica_count = 3`.
3. `prod.auto.tfvars` containing `replica_count = 8`.
4. The CLI command executed is: `terraform apply -var="replica_count=10"`

What is the final value of `var.replica_count` used by Terraform?

A. `2` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `5` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `10`
D. `8` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Explicit command-line arguments (`-var`) have the absolute highest precedence (Rank 7) and override all other sources.
- **Why A is Incorrect:** ``2` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``5` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``8` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
Continuing from Question 1, if the command was executed without the `-var` flag: `terraform apply`, which value wins?

A. `8` (`prod.auto.tfvars`)
B. `5` (`TF_VAR_`) by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `3` (`terraform.tfvars`) through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `2` (Default) during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `*.auto.tfvars` (Rank 5) overrides `terraform.tfvars` (Rank 3), environment variables (Rank 2), and defaults (Rank 1).
- **Why B is Incorrect:** ``5` (`TF_VAR_`) by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``3` (`terraform.tfvars`) through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``2` (Default) during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
Two files exist in the same directory: `a-tier.auto.tfvars` with `cluster_size = 5`, and `z-tier.auto.tfvars` with `cluster_size = 20`. How does Terraform determine which value takes precedence?

A. Terraform throws a duplicate variable error during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. The file created first wins while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. The smaller number wins through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Terraform evaluates `*.auto.tfvars` files in alphabetical order; because `z-tier.auto.tfvars` comes after `a-tier.auto.tfvars`, `cluster_size` evaluates to `20`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Multiple `auto.tfvars` files are loaded in alphabetical order; later files overwrite earlier files.
- **Why A is Incorrect:** `Terraform throws a duplicate variable error during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `The file created first wins while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `The smaller number wins through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
Why is OIDC authentication strictly superior to saving `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in GitHub repository secrets?

A. OIDC is 10x faster during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. OIDC eliminates static long-lived credentials, using cryptographically verified short-lived tokens that automatically expire and can be scoped strictly to specific branches, environments, and repos.
C. OIDC encrypts Terraform state files through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. OIDC bypasses IAM policies while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** OIDC provides temporary, short-lived, least-privilege credentials bound to specific repository metadata.
- **Why A is Incorrect:** `OIDC is 10x faster during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `OIDC encrypts Terraform state files through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `OIDC bypasses IAM policies while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
In an enterprise CI/CD setup, what is the purpose of running `terraform fmt -check` before planning?

A. It verifies that all HCL code follows canonical formatting conventions, exiting with a non-zero code if any file requires formatting, enforcing consistency across teams.
B. It automatically fixes indentation in the remote cloud during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. It compiles code into bytecode through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It validates API credentials while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform fmt -check` returns exit code 0 if formatted correctly and non-zero if changes are needed, making it ideal for CI gates.
- **Why B is Incorrect:** `It automatically fixes indentation in the remote cloud during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It compiles code into bytecode through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It validates API credentials while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
You have a multi-branch Git workflow (`feature/*`, `staging`, `main`). When a developer opens a pull request from `feature/add-redis` to `staging`, what Terraform actions should run automatically?

A. `terraform apply` to staging during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `terraform destroy` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `terraform init`, `validate`, `test`, and a speculative `terraform plan` (without saving a persistent plan or holding locks).
D. `terraform state rm` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Speculative plans give reviewers insight into proposed changes without altering real infrastructure or blocking other developers with locks.
- **Why A is Incorrect:** ``terraform apply` to staging during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform destroy` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform state rm` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
How does HCP Terraform / Terraform Cloud handle concurrent run requests on the same workspace?

A. It executes them simultaneously and merges the state files during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It restarts the runner while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. It drops the second run with a 404 error through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It enqueues runs sequentially, holding state locks to ensure only one plan or apply executes at a time.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Workspaces queue runs sequentially to prevent race conditions and state corruption.
- **Why A is Incorrect:** `It executes them simultaneously and merges the state files during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It restarts the runner while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `It drops the second run with a 404 error through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
In an automated pipeline, an engineer needs to pass a secret API key without committing it to Git or exposing it on the CLI command line process table (`ps -ef`). What is the recommended method?

A. Pass `-var="api_key=secret-value"` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. Set an environment variable: `export TF_VAR_api_key="secret-value"`.
C. Commit a `secret.tfvars` file to Git through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Write the key to `/etc/hosts` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `TF_VAR_<name>` environment variables supply inputs securely without exposing secrets in process tables or CLI history.
- **Why A is Incorrect:** `Pass `-var="api_key=secret-value"` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Commit a `secret.tfvars` file to Git through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Write the key to `/etc/hosts` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
What does the `terraform test` command do in a modern CI pipeline?

A. It sends HTTP pings to google.com during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It scans for Docker container vulnerabilities through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It executes automated unit and integration tests defined in `*.tftest.hcl` files, asserting that resource attributes, validations, and preconditions match expected outputs.
D. It benchmarks network bandwidth while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform test` is the native test harness for module and configuration validation.
- **Why A is Incorrect:** `It sends HTTP pings to google.com during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It scans for Docker container vulnerabilities through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It benchmarks network bandwidth while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
When using GitHub Actions with AWS OIDC, what IAM policy condition ensures that ONLY commits merged to the `main` branch of the `enterprise/infrastructure` repository can assume the deployment role?

A. `StringEquals: { "token.actions.githubusercontent.com:aud": "sts.amazonaws.com" }` and `StringLike: { "token.actions.githubusercontent.com:sub": "repo:enterprise/infrastructure:ref:refs/heads/main" }`
B. `StringEquals: { "aws:PrincipalType": "GitHub" }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `NumericLessThan: { "aws:MultiFactorAuthAge": 3600 }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `ArnEquals: { "aws:SourceArn": "arn:github:enterprise" }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Scoping the `sub` claim to `repo:<org>/<repo>:ref:refs/heads/main` guarantees that only code merged to `main` can execute production writes.
- **Why B is Incorrect:** ``StringEquals: { "aws:PrincipalType": "GitHub" }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``NumericLessThan: { "aws:MultiFactorAuthAge": 3600 }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``ArnEquals: { "aws:SourceArn": "arn:github:enterprise" }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
Your Platform Engineering team inspects `terraform show -json tfplan.binary` inside a GitHub Actions gate before allowing production deployments. For a critical `aws_lb.public` resource that is configured with `lifecycle { create_before_destroy = true }`, an engineer modifies a ForceNew attribute (`internal = false` -> `true`). What exact value appears in the `change.actions` JSON array for `aws_lb.public`?

A. ["replace"]
B. ["delete", "create"]
C. ["create", "delete"]
D. ["update", "create_before_destroy"]

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** In the Terraform JSON plan format (`terraform show -json`), replacements are represented as a two-element array in execution order: `["delete", "create"]` for standard destroy-then-create replacement, and **`["create", "delete"]`** when `create_before_destroy = true` is active!
- **Why A & D are Incorrect:** `"replace"` and `"create_before_destroy"` are not valid action strings in `change.actions`.
- **Why B is Incorrect:** `["delete", "create"]` represents default replacement where destruction occurs before creation.

</details>

---

### Question 12
In a busy monorepo, Developer A's PR generates a saved plan `tfplan.binary` when the remote state is at `serial = 112`. Two minutes later, an automated hotfix pipeline applies a change to the same workspace, advancing the remote state to `serial = 113`. When Developer A's pipeline executes `terraform apply -input=false tfplan.binary`, what happens?

A. Terraform applies only the non-conflicting resources from `tfplan.binary` and increments the state serial to `114`.
B. Terraform automatically re-runs `terraform refresh` against `serial = 113`, updates `tfplan.binary` in place, and applies the changes.
C. Terraform aborts immediately without modifying any infrastructure and throws `Error: Saved plan is stale` because the remote state serial advanced after `tfplan.binary` was created.
D. Terraform overwrites `serial = 113` with the prior state snapshot embedded inside `tfplan.binary`, rolling back the hotfix.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** A saved binary plan records the exact state `serial` and `lineage` from when the plan was computed. If the remote state changes before `apply`, Terraform rejects the stale plan (`Error: Saved plan is stale`) to prevent overwriting concurrent changes.
- **Why A, B, and D are Incorrect:** Terraform never partially applies, silently re-plans, or rolls back newer state serials when executing a saved plan.

</details>

---

### Question 13
To speed up CI builds on a self-hosted Kubernetes runner pod that executes 4 parallel `terraform init` jobs simultaneously across different directories on the same shared volume, an engineer sets `TF_PLUGIN_CACHE_DIR="/mnt/shared/plugin-cache"`. Soon after, jobs begin failing intermittently with corrupted provider binary signature errors. Why?

A. `TF_PLUGIN_CACHE_DIR` only supports caching a single provider version per directory and deletes older versions when a second job runs.
B. Terraform's local plugin cache directory is not concurrency-safe for simultaneous writes; parallel `terraform init` processes downloading the same provider into a shared writable `TF_PLUGIN_CACHE_DIR` race and corrupt the unpacked binary.
C. `TF_PLUGIN_CACHE_DIR` automatically disables `.terraform.lock.hcl` verification across all workspaces.
D. Provider plugins larger than 100 MB cannot be stored in `TF_PLUGIN_CACHE_DIR` without enabling `TF_ENABLE_LARGE_PLUGINS=1`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Official Terraform documentation explicitly warns that `plugin_cache_dir` / `TF_PLUGIN_CACHE_DIR` has no internal file locking and must never be written to concurrently by multiple `terraform init` processes. Pre-populate a read-only filesystem mirror (`terraform providers mirror`) in the container image instead.
- **Why A, C, and D are Incorrect:** The cache stores multiple versions side by side, does not disable `.terraform.lock.hcl`, and has no 100 MB size cap.

</details>

---

### Question 14
You are configuring HCP Terraform Dynamic Provider Credentials via AWS OIDC Workload Identity Federation. To prevent untrusted Pull Requests from executing destructive operations via malicious `data "external"` scripts during speculative plans, how should you configure the AWS IAM roles and HCP Terraform workspace environment variables?

A. Use a single IAM role with `AdministratorAccess` and add a Sentinel `hard-mandatory` policy, because Sentinel policies evaluate before `terraform plan` starts.
B. Configure two separate IAM roles whose OIDC trust policies match `...:run_phase:plan` (read-only permissions) and `...:run_phase:apply` (write permissions), and set `TFC_AWS_PLAN_ROLE_ARN` and `TFC_AWS_APPLY_ROLE_ARN` on the workspace.
C. Set `TF_IN_AUTOMATION=true` and `-lockfile=readonly`, which blocks `data "external"` from executing during `terraform plan`.
D. Disable OIDC and store read-only static IAM user keys in a Global Variable Set.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** HCP Terraform embeds `run_phase:plan` or `run_phase:apply` in the OIDC JWT `sub` claim. Configuring `TFC_AWS_PLAN_ROLE_ARN` (trusted only for `run_phase:plan` with read-only IAM policies) and `TFC_AWS_APPLY_ROLE_ARN` (trusted only for `run_phase:apply`) enforces cryptographic least privilege during PR plans.
- **Why A is Incorrect:** Sentinel and OPA run *after* `terraform plan` completes—meaning any malicious `data "external"` script inside the PR has already executed during the plan phase!
- **Why C & D are Incorrect:** Neither flag blocks `data "external"`, and static IAM keys are a security anti-pattern.

</details>

---

### Question 15
In a strict CI/CD pipeline, you want `terraform init` to verify that every required provider binary matches the exact cryptographic hashes in `.terraform.lock.hcl` and immediately fail if any provider upgrade or missing platform checksum would cause `.terraform.lock.hcl` to be modified. Which flag should you pass?

A. `terraform init -lockfile=readonly`
B. `terraform init -lock=false`
C. `terraform init -upgrade=false -reconfigure`
D. `terraform init -verify-checksums=strict`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform init -lockfile=readonly` instructs Terraform to read `.terraform.lock.hcl` strictly without modifying it, failing immediately if any provider version or platform hash does not match the committed lockfile.
- **Why B is Incorrect:** `-lock=false` disables state locking, not dependency lockfile modification.
- **Why C & D are Incorrect:** Without `-lockfile=readonly`, `terraform init` can still append new platform hashes to `.terraform.lock.hcl`; `-verify-checksums=strict` is not a valid flag.

</details>

---
