# Domain 4: CI/CD & Automation Workflows — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
Why must automated CI/CD pipelines run `terraform apply` using a saved plan file (`terraform apply tfplan`) rather than running `terraform apply -auto-approve` directly?
- A. `tfplan` makes the execution 5x faster.
- B. Passing a pre-generated plan file guarantees that the exact changes inspected, security-scanned, and approved in the review step are what gets applied, eliminating race conditions from concurrent changes or drift between plan and apply.
- C. `-auto-approve` is deprecated in Terraform v1.0+.
- D. Saved plan files encrypt cloud secrets.

---

### Question 2
In automated CI runners (such as GitHub Actions or GitLab CI), how should Terraform authenticate to cloud providers (AWS, Azure, GCP) without storing static long-lived credentials in pipeline secrets?
- A. By storing credentials in plaintext environment variables.
- B. By using OpenID Connect (OIDC) workload identity federation to exchange the runner's ephemeral JWT for short-lived cloud credentials.
- C. By writing passwords to local disk.
- D. By committing credentials into Git.

---

### Question 3
What CLI flag should be passed to `terraform fmt` in CI linting stages to verify formatting without modifying files on disk, failing the job if any file is unformatted?
- A. `terraform fmt -check`
- B. `terraform fmt -diff`
- C. `terraform fmt -dry-run`
- D. `terraform fmt -validate`

---

### Question 4
What does the CLI flag `-detailed-exitcode` on `terraform plan` return when changes are detected in infrastructure?
- A. Exit code 0
- B. Exit code 1
- C. Exit code 2
- D. Exit code 255

---

### Question 5
When running Terraform in non-interactive CI/CD automation environments, which environment variable disables interactive CLI prompts for missing variables?
- A. `TF_IN_AUTOMATION=true`
- B. `TF_LOG=TRACE`
- C. `TF_CLI_ARGS="-input=false"`
- D. `TF_WARN_ONLY=true`

---

### Question 6
You need to convert a binary plan file `tfplan` into machine-readable JSON format so an automated security scanner (such as Checkov or Trivy) can evaluate policy rules. Which command is correct?
- A. `terraform show -json tfplan > plan.json`
- B. `terraform convert tfplan -to json`
- C. `terraform plan -export json`
- D. `cat tfplan | jq .`

---

### Question 7
In an enterprise multi-environment pipeline (dev, staging, prod), how can workspace selection be automated safely via environment variables without hardcoding `terraform workspace select` commands in bash scripts?
- A. `TF_WORKSPACE=<env_name>`
- B. `TF_ENV=<env_name>`
- C. `TERRAFORM_WORKSPACE=<env_name>`
- D. `TF_TARGET_ENV=<env_name>`

---

### Question 8
Why is the dependency lock file `.terraform.lock.hcl` committed to version control in production repositories?
- A. It locks the state file against concurrent runs.
- B. It pins exact provider versions and cryptographic checksums, guaranteeing that every developer and CI runner downloads the exact same provider binaries and preventing supply-chain tampering.
- C. It stores API access keys.
- D. It prevents Git merge conflicts.

---

### Question 9
In an automated pipeline, what occurs if `terraform init` is run with the `-backend=false` flag?
- A. Terraform deletes the remote backend.
- B. Terraform downloads provider plugins and validates configuration without configuring or authenticating to the remote backend, ideal for fast linting and validation jobs.
- C. Terraform disables state locking.
- D. Terraform applies changes locally.

---

### Question 10
When using Atlantis for pull-request-driven Terraform automation, how does Atlantis prevent two pull requests from applying conflicting infrastructure changes simultaneously?
- A. By shutting down the Git server.
- B. By acquiring an Atlantis workspace/project lock on the target directory/workspace as soon as `atlantis plan` runs, blocking other PRs on that directory until the PR is merged or unlocked.
- C. By reverting commits automatically.
- D. By rejecting pull requests with more than 5 lines.

---

## Answer Key & Explanations

1. **B** — Applying a saved plan artifact ensures deterministic execution; running `apply` directly calculates a fresh plan that could differ from what was reviewed.
2. **B** — OIDC Workload Identity Federation replaces static, long-lived API keys with ephemeral, dynamically exchanged cloud IAM credentials.
3. **A** — `terraform fmt -check` returns exit code 0 if all files match formatting standards, and non-zero if any file requires formatting.
4. **C** — `-detailed-exitcode` returns `0` (success with no changes), `1` (error), and `2` (success with changes present).
5. **A** — `TF_IN_AUTOMATION=1` alters CLI output formatting and suppresses suggestions designed for interactive human sessions.
6. **A** — `terraform show -json <plan_file>` renders the binary plan representation into structured JSON for automated policy engines.
7. **A** — Setting the `TF_WORKSPACE` environment variable automatically selects the designated workspace during CLI operations.
8. **B** — `.terraform.lock.hcl` records SHA256 checksums (including cross-platform hashes via `terraform providers lock`) to ensure immutable provider dependencies.
9. **B** — `-backend=false` skips backend initialization, allowing syntax validation and formatting checks without requiring cloud backend credentials.
10. **B** — Atlantis enforces PR-level directory locking to prevent conflicting concurrent plans and applies across Git branches.
