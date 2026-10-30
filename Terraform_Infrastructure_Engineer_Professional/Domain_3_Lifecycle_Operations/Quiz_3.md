# Domain 3: Lifecycle Operations & Day-2 Management — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
In an autoscaling group or database cluster, you need to ensure that a newly updated instance is fully provisioned and healthy BEFORE the old instance is terminated. Which lifecycle configuration must be used?

A. `lifecycle { ignore_changes = [all] }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. `lifecycle { prevent_destroy = true }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `lifecycle { create_before_destroy = true }`
D. `lifecycle { replace_triggered_by = [] }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `create_before_destroy = true` reverses Terraform's default destroy-then-create ordering, spinning up the replacement resource before terminating the original.
- **Why A is Incorrect:** ``lifecycle { ignore_changes = [all] }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``lifecycle { prevent_destroy = true }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``lifecycle { replace_triggered_by = [] }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
You want to force a resource (e.g. `aws_instance.bastion`) to be recreated whenever a specific cryptographic hash changes on a bootstrap user-data script. In modern Terraform (v1.2+), which lifecycle argument replaces legacy `taint` commands?

A. `replace_triggered_by = [terraform_data.bootstrap.output]`
B. `recreate_on_change = true` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `auto_taint = true` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `taint_when = true` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `replace_triggered_by` accepts references to other resources or attributes, triggering a recreation whenever those references change.
- **Why B is Incorrect:** ``recreate_on_change = true` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``auto_taint = true` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``taint_when = true` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
A developer accidentally updates a production database configuration and runs `terraform apply`. Terraform attempts to delete the production database. Which lifecycle rule acts as a safety guardrail by failing the plan if a destruction is attempted?

A. `lifecycle { lock = true }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `lifecycle { create_before_destroy = true }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `lifecycle { immutable = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `lifecycle { prevent_destroy = true }`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `prevent_destroy = true` causes Terraform to throw an immediate error during plan if the execution plan would destroy the resource.
- **Why A is Incorrect:** ``lifecycle { lock = true }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``lifecycle { create_before_destroy = true }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``lifecycle { immutable = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
Why is `terraform apply -target=...` strongly discouraged for day-to-day routine operations in production environments?

A. It runs 10x slower than full apply during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Targeting isolates a sub-graph, ignoring dependency relationships and leaving other resources un-refreshed, which leads to hidden configuration drift and inconsistent state.
C. It deletes all untargeted resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It invalidates AWS IAM credentials while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Targeted applies bypass dependency graph updates and can introduce subtle state divergence and orphaned infrastructure.
- **Why A is Incorrect:** `It runs 10x slower than full apply during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It deletes all untargeted resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It invalidates AWS IAM credentials while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
You have an AWS EC2 instance whose tags are modified by external automated compliance bots. Terraform plan continuously proposes reverting these external tags. How can you configure Terraform to ignore changes to the `tags` attribute?

A. `lifecycle { ignore_changes = [tags] }`
B. `lifecycle { ignore_changes = all }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `lifecycle { static_tags = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Remove the `tags` argument from the HCL manifest while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `ignore_changes = [tags]` instructs Terraform's diff engine to disregard any drift in the specified attributes.
- **Why B is Incorrect:** ``lifecycle { ignore_changes = all }` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``lifecycle { static_tags = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Remove the `tags` argument from the HCL manifest while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
What is the function of the built-in `terraform_data` resource introduced in Terraform v1.4 (replacing `null_resource`)?

A. It provisions local Docker containers during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It connects to MySQL databases through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It stores arbitrary values in state without interacting with an external provider, frequently used to trigger lifecycle `replace_triggered_by` events.
D. It generates TLS private keys while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform_data` is a core-managed primitive that holds data and triggers lifecycle replacements without requiring the third-party `hashicorp/null` provider.
- **Why A is Incorrect:** `It provisions local Docker containers during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It connects to MySQL databases through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It generates TLS private keys while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
What is the consequence of configuring `create_before_destroy = true` on a resource with a strictly unique name (such as an S3 bucket or an IAM role) without using a dynamic prefix?

A. Terraform automatically appends a random GUID to the name during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. AWS allows duplicate S3 bucket names while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. The old resource is deleted first through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The apply will fail because the new resource cannot be created with a duplicate name while the old resource still exists.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Cloud resources with globally or regionally unique names will collide if created before the predecessor is destroyed; use `name_prefix` instead.
- **Why A is Incorrect:** `Terraform automatically appends a random GUID to the name during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `AWS allows duplicate S3 bucket names while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `The old resource is deleted first through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
When refactoring configurations by changing module source paths, what happens if an engineer simply edits the `source` attribute and runs `terraform apply` without running `terraform init`?

A. Terraform automatically downloads the new module in memory during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform aborts with an error stating that the module is not installed, requiring `terraform init`.
C. Terraform deletes all resources managed by that module through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The plan executes using the old cached module code while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Any alteration to module sources, versions, or providers requires `terraform init` to download and link packages into `.terraform/modules`.
- **Why A is Incorrect:** `Terraform automatically downloads the new module in memory during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform deletes all resources managed by that module through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The plan executes using the old cached module code while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
How does Terraform handle circular dependencies between resources in its directed acyclic graph (DAG)?

A. It breaks the cycle arbitrarily at runtime during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It converts circular resources into outputs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It fails during graph evaluation with a `Cycle error` before executing any cloud API calls.
D. It executes the first resource twice while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Terraform builds a Directed Acyclic Graph (DAG); cycles cannot be resolved topologically, triggering a `Cycle:` graph error.
- **Why A is Incorrect:** `It breaks the cycle arbitrarily at runtime during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It converts circular resources into outputs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It executes the first resource twice while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
In Terraform v1.5+, what happens when an `import` block targets a resource that does not exist in the remote cloud provider?

A. Terraform plan reports an error stating that the resource could not be found with the specified ID.
B. Terraform creates the resource from scratch during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Terraform hangs indefinitely through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The state file is deleted while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** If the cloud provider API returns a 404/not found for the imported ID, Terraform plan fails with an import resolution error.
- **Why B is Incorrect:** `Terraform creates the resource from scratch during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform hangs indefinitely through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The state file is deleted while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
