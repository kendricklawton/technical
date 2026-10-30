# Domain 4: CI/CD & Automation Workflows — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
Why must automated CI/CD pipelines run `terraform apply` using a saved plan file (`terraform apply tfplan`) rather than running `terraform apply -auto-approve` directly?

A. `tfplan` makes the execution 5x faster during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `-auto-approve` is deprecated in Terraform v1.0+ through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Passing a pre-generated plan file guarantees that the exact changes inspected, security-scanned, and approved in the review step are what gets applied, eliminating race conditions from concurrent changes or drift between plan and apply.
D. Saved plan files encrypt cloud secrets while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Applying a saved plan artifact ensures deterministic execution; running `apply` directly calculates a fresh plan that could differ from what was reviewed.
- **Why A is Incorrect:** ``tfplan` makes the execution 5x faster during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``-auto-approve` is deprecated in Terraform v1.0+ through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Saved plan files encrypt cloud secrets while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
In automated CI runners (such as GitHub Actions or GitLab CI), how should Terraform authenticate to cloud providers (AWS, Azure, GCP) without storing static long-lived credentials in pipeline secrets?

A. By using OpenID Connect (OIDC) workload identity federation to exchange the runner's ephemeral JWT for short-lived cloud credentials.
B. By storing credentials in plaintext environment variables during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. By writing passwords to local disk through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By committing credentials into Git while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** OIDC Workload Identity Federation replaces static, long-lived API keys with ephemeral, dynamically exchanged cloud IAM credentials.
- **Why B is Incorrect:** `By storing credentials in plaintext environment variables during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `By writing passwords to local disk through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By committing credentials into Git while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
What CLI flag should be passed to `terraform fmt` in CI linting stages to verify formatting without modifying files on disk, failing the job if any file is unformatted?

A. `terraform fmt -validate` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `terraform fmt -diff` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `terraform fmt -dry-run` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform fmt -check`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `terraform fmt -check` returns exit code 0 if all files match formatting standards, and non-zero if any file requires formatting.
- **Why A is Incorrect:** ``terraform fmt -validate` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform fmt -diff` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``terraform fmt -dry-run` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
What does the CLI flag `-detailed-exitcode` on `terraform plan` return when changes are detected in infrastructure?

A. Exit code 0 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Exit code 2
C. Exit code 1 by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
D. Exit code 255 while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `-detailed-exitcode` returns `0` (success with no changes), `1` (error), and `2` (success with changes present).
- **Why A is Incorrect:** `Exit code 0 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Exit code 1 by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Exit code 255 while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
When running Terraform in non-interactive CI/CD automation environments, which environment variable disables interactive CLI prompts for missing variables?

A. `TF_IN_AUTOMATION=true`
B. `TF_LOG=TRACE` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `TF_CLI_ARGS="-input=false"` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `TF_WARN_ONLY=true` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `TF_IN_AUTOMATION=1` alters CLI output formatting and suppresses suggestions designed for interactive human sessions.
- **Why B is Incorrect:** ``TF_LOG=TRACE` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``TF_CLI_ARGS="-input=false"` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``TF_WARN_ONLY=true` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
You need to convert a binary plan file `tfplan` into machine-readable JSON format so an automated security scanner (such as Checkov or Trivy) can evaluate policy rules. Which command is correct?

A. `terraform plan -export json` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. `terraform convert tfplan -to json` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `terraform show -json tfplan > plan.json`
D. `cat tfplan | jq .` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform show -json <plan_file>` renders the binary plan representation into structured JSON for automated policy engines.
- **Why A is Incorrect:** ``terraform plan -export json` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform convert tfplan -to json` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``cat tfplan | jq .` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In an enterprise multi-environment pipeline (dev, staging, prod), how can workspace selection be automated safely via environment variables without hardcoding `terraform workspace select` commands in bash scripts?

A. `TF_TARGET_ENV=<env_name>` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `TF_ENV=<env_name>` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `TERRAFORM_WORKSPACE=<env_name>` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `TF_WORKSPACE=<env_name>`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Setting the `TF_WORKSPACE` environment variable automatically selects the designated workspace during CLI operations.
- **Why A is Incorrect:** ``TF_TARGET_ENV=<env_name>` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``TF_ENV=<env_name>` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``TERRAFORM_WORKSPACE=<env_name>` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
Why is the dependency lock file `.terraform.lock.hcl` committed to version control in production repositories?

A. It locks the state file against concurrent runs during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It pins exact provider versions and cryptographic checksums, guaranteeing that every developer and CI runner downloads the exact same provider binaries and preventing supply-chain tampering.
C. It stores API access keys through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It prevents Git merge conflicts while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `.terraform.lock.hcl` records SHA256 checksums (including cross-platform hashes via `terraform providers lock`) to ensure immutable provider dependencies.
- **Why A is Incorrect:** `It locks the state file against concurrent runs during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It stores API access keys through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It prevents Git merge conflicts while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
In an automated pipeline, what occurs if `terraform init` is run with the `-backend=false` flag?

A. Terraform deletes the remote backend during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform disables state locking through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Terraform downloads provider plugins and validates configuration without configuring or authenticating to the remote backend, ideal for fast linting and validation jobs.
D. Terraform applies changes locally while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `-backend=false` skips backend initialization, allowing syntax validation and formatting checks without requiring cloud backend credentials.
- **Why A is Incorrect:** `Terraform deletes the remote backend during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terraform disables state locking through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform applies changes locally while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
When using Atlantis for pull-request-driven Terraform automation, how does Atlantis prevent two pull requests from applying conflicting infrastructure changes simultaneously?

A. By acquiring an Atlantis workspace/project lock on the target directory/workspace as soon as `atlantis plan` runs, blocking other PRs on that directory until the PR is merged or unlocked.
B. By shutting down the Git server during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. By reverting commits automatically through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By rejecting pull requests with more than 5 lines while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Atlantis enforces PR-level directory locking to prevent conflicting concurrent plans and applies across Git branches.
- **Why B is Incorrect:** `By shutting down the Git server during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `By reverting commits automatically through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By rejecting pull requests with more than 5 lines while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
