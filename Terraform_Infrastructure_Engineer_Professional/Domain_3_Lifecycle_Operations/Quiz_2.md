# Domain 3: Infrastructure Lifecycle and Operations — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Lifecycle & Deployment Scenario Questions

---

### Question 1: Terraform Version Pinning
What is the effect of specifying `required_version = "~> 1.15.0"` in your `terraform` block?
- A. Allows Terraform 1.15.0, 1.15.1, up to (but not including) 1.16.0.
- B. Allows any version greater than or equal to 1.15.0.
- C. Requires exactly 1.15.0 only.
- D. Automatically upgrades Terraform on the host.

---

### Question 2: In-Place Update vs Destruction
In `terraform plan` output, what symbol denotes that a resource will be destroyed and re-created (replacement)?
- A. `~` (tilde)
- B. `-/+` (minus followed by plus in red/green)
- C. `+` (plus)
- D. `<=` (arrow)

---

### Question 3: The Danger of `-target` in Production Pipelines
Why do Staff Infrastructure Engineers strictly forbid using `terraform apply -target=...` in CI/CD release pipelines?
- A. It bypasses IAM role authentication.
- B. It applies only a partial dependency graph, which can leave dependent resources in an inconsistent state, skips state refreshes on untargeted resources, and masks configuration drift.
- C. It doubles cloud costs.
- D. It deletes unselected resources.

---

### Question 4: Dry-Run Speculative Plans in PR Workflows
What is a "speculative plan" in modern automated GitOps / HCP Terraform workflows?
- A. A plan that predicts cloud costs for the next 5 years.
- B. A temporary, read-only plan generated on pull requests that displays prospective changes without saving a plan file, holding state locks, or altering infrastructure.
- C. A machine-learning model that writes HCL code.
- D. A test that deploys resources to a local Docker container.

---

### Question 5: Preventing Accidental Database Deletion on Branch Merges
A developer accidentally changes a database identifier attribute in HCL, which cloud providers treat as a "Force New Resource" (destroy and recreate). How does `prevent_destroy = true` safeguard the cluster?
- A. Terraform ignores the identifier change and silently keeps the existing database.
- B. Terraform generates an error during the planning phase stating the resource cannot be destroyed, blocking the apply.
- C. Terraform moves the database to a secondary account.
- D. Terraform takes a manual snapshot and then destroys the database.

---

### Question 6: Managing Multi-Account Infrastructure
When organizing Terraform to provision across AWS Management, Security, Staging, and Production accounts:
- A. Run a single Terraform apply using the root account credentials.
- B. Use separate workspaces/directories where each workspace assumes an IAM role (`role_arn`) in the target account via provider configuration.
- C. Store all account credentials in a shared `.env` file.
- D. AWS does not permit cross-account Terraform configurations.

---

### Question 7: Rebuilding Immutable AMI Instances
You deploy virtual machines using custom AMIs built with Packer. When a new AMI is published, you update `var.ami_id`. How do you ensure the replacement VM is healthy before the existing VM is decommissioned?
- A. Use `lifecycle { create_before_destroy = true }` along with health check integration.
- B. Run `terraform destroy` first.
- C. Use `lifecycle { prevent_destroy = true }`.
- D. Set `parallelism = 1`.

---

### Question 8: Updating Providers Without Upgrading State
You run `terraform init -upgrade`. What does this command do?
- A. It upgrades your local Terraform CLI binary.
- B. It queries provider registries to download the newest acceptable provider versions matching the constraints in `required_providers` and updates `.terraform.lock.hcl`.
- C. It rewrites `.tf` code into Go.
- D. It clears all cloud resources.

---

### Question 9: `terraform plan` Exit Codes for CI Automation
When running `terraform plan -detailed-exitcode` in an automated bash script, what does exit code `2` signify?
- A. An unhandled exception or syntax error occurred.
- B. The plan succeeded with zero changes (infrastructure is up to date).
- C. The plan succeeded and changes are present (diff detected).
- D. Remote state lock failed.

---

### Question 10: Ignoring Tag Updates from Cloud Custodian
An automated enterprise compliance daemon (Cloud Custodian / AWS Config) automatically attaches audit tags to newly created cloud resources. Terraform plan keeps trying to remove these tags. What is the most maintainable resolution?
- A. Add `lifecycle { ignore_changes = [tags, tags_all] }` on resources, or configure `ignore_tags` in the AWS provider block.
- B. Disable the compliance daemon.
- C. Run `terraform apply` continuously in an infinite loop.
- D. Hardcode the audit tags into all `.tf` files.

---

## Answer Key & Explanations

1. **A** — Pessimistic constraint `~> 1.15.0` allows patch version increases within minor version 1.15.
2. **B** — `-/+` indicates destruction followed by recreation.
3. **B** — Targeting breaks the holistic evaluation of the infrastructure graph and leads to untracked drift.
4. **B** — Speculative plans give pull request reviewers visibility into proposed changes safely.
5. **B** — `prevent_destroy` rejects execution before any destructive cloud API calls are made.
6. **B** — Using STS AssumeRole across distinct workspaces enforces strict multi-account security boundaries.
7. **A** — `create_before_destroy = true` creates the replacement first to avoid service downtime.
8. **B** — `terraform init -upgrade` checks for new versions of providers and modules within specified version constraints.
9. **C** — With `-detailed-exitcode`: 0 = No changes, 1 = Error, 2 = Plan contains changes.
10. **A** — The AWS provider's `ignore_tags` block or resource-level `ignore_changes` cleanly prevents tag flapping.
