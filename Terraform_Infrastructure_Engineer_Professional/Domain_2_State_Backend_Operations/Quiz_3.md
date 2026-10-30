# Domain 2: State & Backend Operations — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
You are running a terraform operation against an S3/DynamoDB remote backend, but the terminal session crashes midway through execution. Subsequent runs fail with `Error acquiring the state lock: ConditionalCheckFailedException`. What is the correct and safe recovery command?

A. `rm -rf .terraform` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `terraform state push -force` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `terraform force-unlock <LOCK-ID>` after verifying that no other engineer or pipeline is actively running.
D. Delete the DynamoDB table from AWS console while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform force-unlock <LOCK-ID>` releases a stale lock once it is confirmed no execution process is holding it.
- **Why A is Incorrect:** ``rm -rf .terraform` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform state push -force` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Delete the DynamoDB table from AWS console while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
You need to remove a legacy resource from Terraform management without destroying the actual physical cloud infrastructure. Which command achieves this?

A. `terraform state rm aws_s3_bucket.legacy`
B. `terraform destroy -target=aws_s3_bucket.legacy` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `terraform state delete aws_s3_bucket.legacy` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform taint aws_s3_bucket.legacy` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform state rm` deletes the resource from state tracking without invoking the provider's `Delete` API.
- **Why B is Incorrect:** ``terraform destroy -target=aws_s3_bucket.legacy` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform state delete aws_s3_bucket.legacy` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform taint aws_s3_bucket.legacy` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
In Terraform v1.1+, which block allows engineers to declare that a resource was moved or renamed inside the configuration without modifying the state file manually?

A. `import {}` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `replace {}` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `alias {}` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `moved {}`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** The declarative `moved { from = ... to = ... }` block records refactoring in code, avoiding manual CLI state interventions.
- **Why A is Incorrect:** ``import {}` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``replace {}` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``alias {}` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
What happens if you run `terraform state pull > state.json`, edit `state.json` to change a resource attribute, and run `terraform state push state.json` without incrementing `serial`?

A. Terraform automatically increments the serial during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform rejects the push with a state lineage or serial mismatch error unless `-force` is supplied.
C. The physical cloud infrastructure updates instantly through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. DynamoDB deletes the lock table while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Remote backends compare `lineage` and `serial` numbers to reject stale or concurrently modified state uploads unless `-force` is specified.
- **Why A is Incorrect:** `Terraform automatically increments the serial during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `The physical cloud infrastructure updates instantly through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `DynamoDB deletes the lock table while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
You are migrating an existing remote state from an S3 backend to Terraform Cloud / HCP Terraform. Which command initiates the migration process?

A. `terraform init -migrate-state`
B. `terraform refresh` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `terraform apply -migrate` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform backend push` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Changing the backend block and running `terraform init -migrate-state` prompts Terraform to copy existing state to the new backend.
- **Why B is Incorrect:** ``terraform refresh` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform apply -migrate` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform backend push` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Why does standard Terraform state locking NOT prevent state corruption if two engineers run `terraform apply` using a backend that does not support locking (such as standard HTTP without lock headers or basic local file storage)?

A. Terraform automatically switches to Git for locking during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. The cloud provider rejects the API calls through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. The second apply will overwrite the state file written by the first apply, causing a race condition and losing tracked resource metadata.
D. The Terraform binary exits with an error code while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Without locking, concurrent writes cause last-write-wins collisions, corrupting or wiping state entries.
- **Why A is Incorrect:** `Terraform automatically switches to Git for locking during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `The cloud provider rejects the API calls through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The Terraform binary exits with an error code while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In Terraform v1.5+, what block syntax allows importing existing cloud infrastructure into Terraform state declaratively during `terraform plan`?

A. `resource_import {}` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `fetch {}` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `moved {}` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `import { to = aws_s3_bucket.example, id = "my-bucket-name" }`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Declarative `import` blocks (`import { to = ... id = ... }`) generate configuration and import state during standard plan/apply workflows.
- **Why A is Incorrect:** ``resource_import {}` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``fetch {}` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``moved {}` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
When using `terraform_remote_state` data source to read outputs from a networking state file, what is a key architectural risk?

A. It causes AWS bill charges for every plan by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. It creates an explicit tight coupling between workspaces and requires the consumer workspace to have read access to the entire remote state file, exposing all sensitive data contained in that state.
C. It only works with Terraform 0.11 through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It deletes the producer workspace while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `terraform_remote_state` exposes the entire target state file (including all sensitive outputs and resource attributes) to the consumer, violating least privilege.
- **Why A is Incorrect:** `It causes AWS bill charges for every plan by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It only works with Terraform 0.11 through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It deletes the producer workspace while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
What is the effect of running `terraform refresh` (or running `terraform plan -refresh-only`)?

A. It destroys all tainted resources during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It deletes local lock files through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It queries the cloud provider APIs to update the state file with the current real-world status of managed infrastructure without proposing changes to configuration.
D. It downloads new provider plugins while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Refresh queries external APIs to reconcile drift between state metadata and actual cloud infrastructure.
- **Why A is Incorrect:** `It destroys all tainted resources during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It deletes local lock files through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It downloads new provider plugins while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
Under what condition will `terraform plan` fail with `Lineage mismatch`?

A. When you try to push a state file that originated from a completely different state initialization (different `lineage` UUID).
B. When you have unused variables in `variables.tf` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. When you use AWS provider v5.0 through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. When the state file contains more than 100 resources while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `lineage` is a unique UUID generated at backend initialization; mismatched lineages indicate distinct, unrelated state histories.
- **Why B is Incorrect:** `When you have unused variables in `variables.tf` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `When you use AWS provider v5.0 through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `When the state file contains more than 100 resources while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
