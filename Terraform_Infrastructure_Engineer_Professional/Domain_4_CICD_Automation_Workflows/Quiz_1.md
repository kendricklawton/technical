# Domain 4: CI/CD and Automation Workflows — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An engineer defines `variable "replica_count" { default = 2 }`. The following files and settings are present during pipeline execution:
1. `TF_VAR_replica_count = 5` in the shell environment.
2. `terraform.tfvars` containing `replica_count = 3`.
3. `prod.auto.tfvars` containing `replica_count = 8`.
4. The CLI command executed is: `terraform apply -var="replica_count=10"`

What is the final value of `var.replica_count` used by Terraform?
- A. `2`
- B. `5`
- C. `8`
- D. `10`

---

### Question 2
Continuing from Question 1, if the command was executed without the `-var` flag: `terraform apply`, which value wins?
- A. `2` (Default)
- B. `5` (`TF_VAR_`)
- C. `3` (`terraform.tfvars`)
- D. `8` (`prod.auto.tfvars`)

---

### Question 3
Two files exist in the same directory: `a-tier.auto.tfvars` with `cluster_size = 5`, and `z-tier.auto.tfvars` with `cluster_size = 20`. How does Terraform determine which value takes precedence?
- A. Terraform throws a duplicate variable error.
- B. Terraform evaluates `*.auto.tfvars` files in alphabetical order; because `z-tier.auto.tfvars` comes after `a-tier.auto.tfvars`, `cluster_size` evaluates to `20`.
- C. The smaller number wins.
- D. The file created first wins.

---

### Question 4
Why is OIDC authentication strictly superior to saving `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in GitHub repository secrets?
- A. OIDC is 10x faster.
- B. OIDC eliminates static long-lived credentials, using cryptographically verified short-lived tokens that automatically expire and can be scoped strictly to specific branches, environments, and repos.
- C. OIDC encrypts Terraform state files.
- D. OIDC bypasses IAM policies.

---

### Question 5
In an enterprise CI/CD setup, what is the purpose of running `terraform fmt -check` before planning?
- A. It automatically fixes indentation in the remote cloud.
- B. It verifies that all HCL code follows canonical formatting conventions, exiting with a non-zero code if any file requires formatting, enforcing consistency across teams.
- C. It compiles code into bytecode.
- D. It validates API credentials.

---

### Question 6
You have a multi-branch Git workflow (`feature/*`, `staging`, `main`). When a developer opens a pull request from `feature/add-redis` to `staging`, what Terraform actions should run automatically?
- A. `terraform apply` to staging.
- B. `terraform init`, `validate`, `test`, and a speculative `terraform plan` (without saving a persistent plan or holding locks).
- C. `terraform destroy`.
- D. `terraform state rm`.

---

### Question 7
How does HCP Terraform / Terraform Cloud handle concurrent run requests on the same workspace?
- A. It executes them simultaneously and merges the state files.
- B. It enqueues runs sequentially, holding state locks to ensure only one plan or apply executes at a time.
- C. It drops the second run with a 404 error.
- D. It restarts the runner.

---

### Question 8
In an automated pipeline, an engineer needs to pass a secret API key without committing it to Git or exposing it on the CLI command line process table (`ps -ef`). What is the recommended method?
- A. Set an environment variable: `export TF_VAR_api_key="secret-value"`.
- B. Pass `-var="api_key=secret-value"`.
- C. Commit a `secret.tfvars` file to Git.
- D. Write the key to `/etc/hosts`.

---

### Question 9
What does the `terraform test` command do in a modern CI pipeline?
- A. It sends HTTP pings to google.com.
- B. It executes automated unit and integration tests defined in `*.tftest.hcl` files, asserting that resource attributes, validations, and preconditions match expected outputs.
- C. It scans for Docker container vulnerabilities.
- D. It benchmarks network bandwidth.

---

### Question 10
When using GitHub Actions with AWS OIDC, what IAM policy condition ensures that ONLY commits merged to the `main` branch of the `enterprise/infrastructure` repository can assume the deployment role?
- A. `StringEquals: { "token.actions.githubusercontent.com:aud": "sts.amazonaws.com" }` and `StringLike: { "token.actions.githubusercontent.com:sub": "repo:enterprise/infrastructure:ref:refs/heads/main" }`
- B. `StringEquals: { "aws:PrincipalType": "GitHub" }`
- C. `NumericLessThan: { "aws:MultiFactorAuthAge": 3600 }`
- D. `ArnEquals: { "aws:SourceArn": "arn:github:enterprise" }`

---

## Answer Key & Explanations

1. **D** — Explicit command-line arguments (`-var`) have the absolute highest precedence (Rank 7) and override all other sources.
2. **D** — `*.auto.tfvars` (Rank 5) overrides `terraform.tfvars` (Rank 3), environment variables (Rank 2), and defaults (Rank 1).
3. **B** — Multiple `auto.tfvars` files are loaded in alphabetical order; later files overwrite earlier files.
4. **B** — OIDC provides temporary, short-lived, least-privilege credentials bound to specific repository metadata.
5. **B** — `terraform fmt -check` returns exit code 0 if formatted correctly and non-zero if changes are needed, making it ideal for CI gates.
6. **B** — Speculative plans give reviewers insight into proposed changes without altering real infrastructure or blocking other developers with locks.
7. **B** — Workspaces queue runs sequentially to prevent race conditions and state corruption.
8. **A** — `TF_VAR_<name>` environment variables supply inputs securely without exposing secrets in process tables or CLI history.
9. **B** — `terraform test` is the native test harness for module and configuration validation.
10. **A** — Scoping the `sub` claim to `repo:<org>/<repo>:ref:refs/heads/main` guarantees that only code merged to `main` can execute production writes.
