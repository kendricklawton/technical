# Domain 4: CI/CD and Automation Workflows — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Automation & Pipeline Scenario Questions

---

### Question 1: Handling Secrets in CI Pull Requests
Why should untrusted PRs from external or fork contributors NEVER trigger workflows with access to production cloud credentials or OIDC tokens?
- A. It consumes too much CPU.
- B. A malicious contributor could modify `.tf` files or CI scripts to print environment variables, exfiltrate state, or run destructive resources during the plan step (`pull_request` vs `pull_request_target` vulnerability).
- C. GitHub limits PRs to 5 runs per day.
- D. Terraform cannot parse code written by forks.

---

### Question 2: Pipeline Concurrency Control in GitHub Actions
How do you prevent two concurrent pull request merges from triggering overlapping `terraform apply` executions that would contend for the state lock?
- A. By adding a `concurrency: production-apply` block to the GitHub Actions workflow to serialize execution.
- B. By disabling merges on GitHub.
- C. By sleeping 60 seconds at the start of each job.
- D. By deleting the `.terraform` folder.

---

### Question 3: Dynamic Vault Credentials in CI/CD
When integrating Terraform with HashiCorp Vault in a pipeline:
- A. Hardcode the Vault root token in `main.tf`.
- B. The CI runner exchanges its OIDC token with Vault's JWT/OIDC auth method to receive a scoped, short-lived Vault token that generates dynamic cloud secrets on-the-fly.
- C. Store Vault secrets in a public S3 bucket.
- D. Vault does not support CI/CD.

---

### Question 4: Auto-Approve Risks
Why should `-auto-approve` be strictly restricted to automated CI/CD runners on `main` branches and NEVER used by individual engineers locally against production?
- A. Local terminals cannot parse `-auto-approve`.
- B. Local execution bypasses peer code review, branch protection rules, automated policy checks (Sentinel/OPA), and audit log trails.
- C. Cloud APIs reject local auto-approve flags.
- D. It doubles the execution time.

---

### Question 5: Managing Multi-Cloud Credentials
An enterprise deploys workloads spanning AWS and DigitalOcean. How should credentials be structured in the CI pipeline?
- A. Store both in a single shared unencrypted file.
- B. Use separate OIDC role assumption for AWS, and a securely injected short-lived DigitalOcean API token (`DIGITALOCEAN_TOKEN`) stored in an enterprise secret manager.
- C. Combine both into a single IAM role.
- D. Multi-cloud deployment cannot be automated in a single pipeline.

---

### Question 6: Caching `.terraform` Directory in CI
Why is caching the `.terraform` plugin cache directory (`TF_PLUGIN_CACHE_DIR`) recommended in enterprise CI pipelines?
- A. It prevents Terraform from checking code syntax.
- B. It avoids re-downloading hundreds of megabytes of provider binaries on every run, drastically reducing pipeline latency and avoiding registry rate limits.
- C. It caches state files locally.
- D. It bypasses license checks.

---

### Question 7: Pre-Commit Hooks for Infrastructure Code
What tool chain is typically configured in developer pre-commit hooks before committing Terraform code?
- A. `terraform fmt`, `terraform validate`, `tflint`, and `checkov`/`tfsec`.
- B. `terraform apply -auto-approve`.
- C. `git push --force`.
- D. `docker run -d nginx`.

---

### Question 8: Ephemeral Environments (PR Preview Environments)
What is the standard Terraform pattern for spinning up short-lived preview environments for PR testing?
- A. Manually creating AWS accounts for each PR.
- B. Creating a dynamic workspace named `pr-${{ github.event.number }}`, applying the plan, running automated integration tests, and executing `terraform destroy` when the PR is closed or merged.
- C. Hardcoding PR numbers into `production.tfvars`.
- D. Sharing a single database across all PRs.

---

### Question 9: GitOps Engine Comparison (Terraform vs ArgoCD)
What is the primary architectural boundary between Terraform and Kubernetes GitOps tools like ArgoCD/Flux?
- A. ArgoCD manages physical servers; Terraform manages containers.
- B. Terraform provisions the foundational cloud infrastructure (VPC, IAM, EKS/DOKS clusters, RDS databases), while ArgoCD manages the in-cluster Kubernetes manifests and applications deployed on top of that infrastructure.
- C. They perform identical roles and cannot be used together.
- D. Terraform has been deprecated in favor of ArgoCD.

---

### Question 10: Handling Terraform Apply Failures in CI
If a multi-step `terraform apply` fails halfway through due to a cloud API timeout, what state is the infrastructure left in?
- A. All resources are rolled back to the beginning automatically.
- B. The state file retains all resources successfully created up to the failure point; running `terraform apply` again resumes from the recorded state without recreating existing resources.
- C. The entire state file is erased.
- D. The cloud provider deletes the account.

---

## Answer Key & Explanations

1. **B** — Malicious pull requests can execute arbitrary code inside runner environments if granted access to secrets.
2. **A** — GitHub Actions `concurrency` groups ensure that only one job with that group key runs at a time.
3. **B** — Vault's JWT/OIDC auth engine allows runners to authenticate securely without static credentials.
4. **B** — Governance, compliance, and peer review are bypassed when local engineers run uncontrolled `-auto-approve` applies.
5. **B** — Separate, scoped credentials adhering to least privilege maintain security boundaries across multi-cloud estates.
6. **B** — `TF_PLUGIN_CACHE_DIR` speeds up pipeline execution and prevents hitting registry rate limits.
7. **A** — Linting, formatting, and static security scanning catch defects before code ever reaches remote repositories.
8. **B** — Ephemeral workspaces provide isolated staging sandboxes tied directly to pull request lifecycles.
9. **B** — Terraform manages substrate cloud infrastructure, while GitOps operators manage declarative workloads inside the cluster.
10. **B** — Terraform state is updated iteratively as resources are provisioned, enabling idempotent resumption upon re-run.
