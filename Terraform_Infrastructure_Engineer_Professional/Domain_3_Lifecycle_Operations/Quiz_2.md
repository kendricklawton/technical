# Domain 3: Infrastructure Lifecycle and Operations — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Lifecycle & Deployment Scenario Questions

---

---

### Question 1
What is the effect of specifying `required_version = "~> 1.15.0"` in your `terraform` block?

A. Requires exactly 1.15.0 only through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. Allows any version greater than or equal to 1.15.0 by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Allows Terraform 1.15.0, 1.15.1, up to (but not including) 1.16.0.
D. Automatically upgrades Terraform on the host while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Pessimistic constraint `~> 1.15.0` allows patch version increases within minor version 1.15.
- **Why A is Incorrect:** `Requires exactly 1.15.0 only through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Allows any version greater than or equal to 1.15.0 by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Automatically upgrades Terraform on the host while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
In `terraform plan` output, what symbol denotes that a resource will be destroyed and re-created (replacement)?

A. `-/+` (minus followed by plus in red/green)
B. `~` (tilde) during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `+` (plus) through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `<=` (arrow) while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `-/+` indicates destruction followed by recreation.
- **Why B is Incorrect:** ``~` (tilde) during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``+` (plus) through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``<=` (arrow) while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
Why do Staff Infrastructure Engineers strictly forbid using `terraform apply -target=...` in CI/CD release pipelines?

A. It bypasses IAM role authentication during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It deletes unselected resources while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. It doubles cloud costs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It applies only a partial dependency graph, which can leave dependent resources in an inconsistent state, skips state refreshes on untargeted resources, and masks configuration drift.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Targeting breaks the holistic evaluation of the infrastructure graph and leads to untracked drift.
- **Why A is Incorrect:** `It bypasses IAM role authentication during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It deletes unselected resources while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `It doubles cloud costs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
What is a "speculative plan" in modern automated GitOps / HCP Terraform workflows?

A. A plan that predicts cloud costs for the next 5 years during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. A temporary, read-only plan generated on pull requests that displays prospective changes without saving a plan file, holding state locks, or altering infrastructure.
C. A machine-learning model that writes HCL code through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. A test that deploys resources to a local Docker container while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Speculative plans give pull request reviewers visibility into proposed changes safely.
- **Why A is Incorrect:** `A plan that predicts cloud costs for the next 5 years during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `A machine-learning model that writes HCL code through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `A test that deploys resources to a local Docker container while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
A developer accidentally changes a database identifier attribute in HCL, which cloud providers treat as a "Force New Resource" (destroy and recreate). How does `prevent_destroy = true` safeguard the cluster?

A. Terraform generates an error during the planning phase stating the resource cannot be destroyed, blocking the apply.
B. Terraform ignores the identifier change and silently keeps the existing database.
C. Terraform moves the database to a secondary account through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Terraform takes a manual snapshot and then destroys the database while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `prevent_destroy` rejects execution before any destructive cloud API calls are made.
- **Why B is Incorrect:** `Terraform ignores the identifier change and silently keeps the existing database.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform moves the database to a secondary account through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform takes a manual snapshot and then destroys the database while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
When organizing Terraform to provision across AWS Management, Security, Staging, and Production accounts:

A. Run a single Terraform apply using the root account credentials during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Store all account credentials in a shared `.env` file through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Use separate workspaces/directories where each workspace assumes an IAM role (`role_arn`) in the target account via provider configuration.
D. AWS does not permit cross-account Terraform configurations while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Using STS AssumeRole across distinct workspaces enforces strict multi-account security boundaries.
- **Why A is Incorrect:** `Run a single Terraform apply using the root account credentials during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Store all account credentials in a shared `.env` file through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `AWS does not permit cross-account Terraform configurations while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
You deploy virtual machines using custom AMIs built with Packer. When a new AMI is published, you update `var.ami_id`. How do you ensure the replacement VM is healthy before the existing VM is decommissioned?

A. Set `parallelism = 1` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. Run `terraform destroy` first by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Use `lifecycle { prevent_destroy = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Use `lifecycle { create_before_destroy = true }` along with health check integration.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `create_before_destroy = true` creates the replacement first to avoid service downtime.
- **Why A is Incorrect:** `Set `parallelism = 1` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Run `terraform destroy` first by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Use `lifecycle { prevent_destroy = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
You run `terraform init -upgrade`. What does this command do?

A. It upgrades your local Terraform CLI binary during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It queries provider registries to download the newest acceptable provider versions matching the constraints in `required_providers` and updates `.terraform.lock.hcl`.
C. It rewrites `.tf` code into Go through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It clears all cloud resources while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `terraform init -upgrade` checks for new versions of providers and modules within specified version constraints.
- **Why A is Incorrect:** `It upgrades your local Terraform CLI binary during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It rewrites `.tf` code into Go through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It clears all cloud resources while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
When running `terraform plan -detailed-exitcode` in an automated bash script, what does exit code `2` signify?

A. An unhandled exception or syntax error occurred during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. The plan succeeded with zero changes (infrastructure is up to date).
C. The plan succeeded and changes are present (diff detected).
D. Remote state lock failed while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** With `-detailed-exitcode`: 0 = No changes, 1 = Error, 2 = Plan contains changes.
- **Why A is Incorrect:** `An unhandled exception or syntax error occurred during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `The plan succeeded with zero changes (infrastructure is up to date).` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Remote state lock failed while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
An automated enterprise compliance daemon (Cloud Custodian / AWS Config) automatically attaches audit tags to newly created cloud resources. Terraform plan keeps trying to remove these tags. What is the most maintainable resolution?

A. Add `lifecycle { ignore_changes = [tags, tags_all] }` on resources, or configure `ignore_tags` in the AWS provider block.
B. Disable the compliance daemon by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Run `terraform apply` continuously in an infinite loop through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Hardcode the audit tags into all `.tf` files while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** The AWS provider's `ignore_tags` block or resource-level `ignore_changes` cleanly prevents tag flapping.
- **Why B is Incorrect:** `Disable the compliance daemon by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Run `terraform apply` continuously in an infinite loop through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Hardcode the audit tags into all `.tf` files while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
