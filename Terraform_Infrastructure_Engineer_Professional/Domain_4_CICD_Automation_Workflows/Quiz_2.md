# Domain 4: CI/CD and Automation Workflows — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Automation & Pipeline Scenario Questions

---

---

### Question 1
Why should untrusted PRs from external or fork contributors NEVER trigger workflows with access to production cloud credentials or OIDC tokens?

A. It consumes too much CPU during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. GitHub limits PRs to 5 runs per day through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. A malicious contributor could modify `.tf` files or CI scripts to print environment variables, exfiltrate state, or run destructive resources during the plan step (`pull_request` vs `pull_request_target` vulnerability).
D. Terraform cannot parse code written by forks while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Malicious pull requests can execute arbitrary code inside runner environments if granted access to secrets.
- **Why A is Incorrect:** `It consumes too much CPU during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `GitHub limits PRs to 5 runs per day through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform cannot parse code written by forks while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
How do you prevent two concurrent pull request merges from triggering overlapping `terraform apply` executions that would contend for the state lock?

A. By adding a `concurrency: production-apply` block to the GitHub Actions workflow to serialize execution.
B. By disabling merges on GitHub by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. By sleeping 60 seconds at the start of each job through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By deleting the `.terraform` folder while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** GitHub Actions `concurrency` groups ensure that only one job with that group key runs at a time.
- **Why B is Incorrect:** `By disabling merges on GitHub by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `By sleeping 60 seconds at the start of each job through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By deleting the `.terraform` folder while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
When integrating Terraform with HashiCorp Vault in a pipeline:

A. Hardcode the Vault root token in `main.tf` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Vault does not support CI/CD while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. Store Vault secrets in a public S3 bucket through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The CI runner exchanges its OIDC token with Vault's JWT/OIDC auth method to receive a scoped, short-lived Vault token that generates dynamic cloud secrets on-the-fly.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Vault's JWT/OIDC auth engine allows runners to authenticate securely without static credentials.
- **Why A is Incorrect:** `Hardcode the Vault root token in `main.tf` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Vault does not support CI/CD while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Store Vault secrets in a public S3 bucket through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
Why should `-auto-approve` be strictly restricted to automated CI/CD runners on `main` branches and NEVER used by individual engineers locally against production?

A. Local terminals cannot parse `-auto-approve` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Local execution bypasses peer code review, branch protection rules, automated policy checks (Sentinel/OPA), and audit log trails.
C. Cloud APIs reject local auto-approve flags through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It doubles the execution time while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Governance, compliance, and peer review are bypassed when local engineers run uncontrolled `-auto-approve` applies.
- **Why A is Incorrect:** `Local terminals cannot parse `-auto-approve` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Cloud APIs reject local auto-approve flags through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It doubles the execution time while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
An enterprise deploys workloads spanning AWS and DigitalOcean. How should credentials be structured in the CI pipeline?

A. Use separate OIDC role assumption for AWS, and a securely injected short-lived DigitalOcean API token (`DIGITALOCEAN_TOKEN`) stored in an enterprise secret manager.
B. Store both in a single shared unencrypted file during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Combine both into a single IAM role through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Multi-cloud deployment cannot be automated in a single pipeline while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Separate, scoped credentials adhering to least privilege maintain security boundaries across multi-cloud estates.
- **Why B is Incorrect:** `Store both in a single shared unencrypted file during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Combine both into a single IAM role through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Multi-cloud deployment cannot be automated in a single pipeline while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Why is caching the `.terraform` plugin cache directory (`TF_PLUGIN_CACHE_DIR`) recommended in enterprise CI pipelines?

A. It prevents Terraform from checking code syntax during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It caches state files locally through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It avoids re-downloading hundreds of megabytes of provider binaries on every run, drastically reducing pipeline latency and avoiding registry rate limits.
D. It bypasses license checks while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `TF_PLUGIN_CACHE_DIR` speeds up pipeline execution and prevents hitting registry rate limits.
- **Why A is Incorrect:** `It prevents Terraform from checking code syntax during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It caches state files locally through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It bypasses license checks while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
What tool chain is typically configured in developer pre-commit hooks before committing Terraform code?

A. `docker run -d nginx` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `terraform apply -auto-approve` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `git push --force` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform fmt`, `terraform validate`, `tflint`, and `checkov`/`tfsec`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Linting, formatting, and static security scanning catch defects before code ever reaches remote repositories.
- **Why A is Incorrect:** ``docker run -d nginx` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform apply -auto-approve` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``git push --force` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What is the standard Terraform pattern for spinning up short-lived preview environments for PR testing?

A. Manually creating AWS accounts for each PR during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Creating a dynamic workspace named `pr-${{ github.event.number }}`, applying the plan, running automated integration tests, and executing `terraform destroy` when the PR is closed or merged.
C. Hardcoding PR numbers into `production.tfvars` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Sharing a single database across all PRs while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Ephemeral workspaces provide isolated staging sandboxes tied directly to pull request lifecycles.
- **Why A is Incorrect:** `Manually creating AWS accounts for each PR during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Hardcoding PR numbers into `production.tfvars` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Sharing a single database across all PRs while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
What is the primary architectural boundary between Terraform and Kubernetes GitOps tools like ArgoCD/Flux?

A. ArgoCD manages physical servers; Terraform manages containers during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. They perform identical roles and cannot be used together through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Terraform provisions the foundational cloud infrastructure (VPC, IAM, EKS/DOKS clusters, RDS databases), while ArgoCD manages the in-cluster Kubernetes manifests and applications deployed on top of that infrastructure.
D. Terraform has been deprecated in favor of ArgoCD while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Terraform manages substrate cloud infrastructure, while GitOps operators manage declarative workloads inside the cluster.
- **Why A is Incorrect:** `ArgoCD manages physical servers; Terraform manages containers during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `They perform identical roles and cannot be used together through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform has been deprecated in favor of ArgoCD while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
If a multi-step `terraform apply` fails halfway through due to a cloud API timeout, what state is the infrastructure left in?

A. The state file retains all resources successfully created up to the failure point; running `terraform apply` again resumes from the recorded state without recreating existing resources.
B. All resources are rolled back to the beginning automatically during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. The entire state file is erased through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The cloud provider deletes the account while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Terraform state is updated iteratively as resources are provisioned, enabling idempotent resumption upon re-run.
- **Why B is Incorrect:** `All resources are rolled back to the beginning automatically during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `The entire state file is erased through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The cloud provider deletes the account while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
