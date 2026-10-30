# Domain 6: Advanced Modules & Testing Frameworks — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
In the native `terraform test` framework (Terraform v1.6+), what is the difference between `command = plan` and `command = apply` within a `run` block?

A. `command = plan` executes static syntax checks only; `command = apply` deploys code to production during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `command = apply` cannot use assertions through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `command = plan` generates an execution plan and evaluates assertions without creating real cloud infrastructure; `command = apply` provisions ephemeral test resources, asserts conditions, and destroys them upon completion.
D. There is no difference while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** In `terraform test`, `command = plan` runs in-memory evaluation; `command = apply` creates physical resources and tears them down at the end of the test run.
- **Why A is Incorrect:** ``command = plan` executes static syntax checks only; `command = apply` deploys code to production during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``command = apply` cannot use assertions through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `There is no difference while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
You are authoring a `.tftest.hcl` file and want to verify that an output `bucket_arn` matches a specific regex pattern. Which block and syntax is used?

A. ```hcl
B. ```hcl by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. ```hcl through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. ```hcl while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `assert { condition = ... error_message = ... }` inside a `run` block tests planned outputs against logical conditions.
- **Why B is Incorrect:** ````hcl by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ````hcl through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ````hcl while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
How does Terratest differ from the native `terraform test` framework?

A. Terratest is a static analysis linter while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. Terratest is built into the Terraform binary by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Terratest does not support AWS through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Terratest is written in Go and provisions real infrastructure while executing real-world end-to-end integration tests (such as sending HTTP requests or SSHing into VMs) using cloud SDKs.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Terratest is a Go testing library that leverages Go's native testing framework and cloud client libraries to validate functional end-to-end behavior.
- **Why A is Incorrect:** `Terratest is a static analysis linter while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terratest is built into the Terraform binary by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Terratest does not support AWS through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
In Terraform v1.5+, what is the purpose of the top-level `check` block in configuration files?

A. It blocks `terraform apply` if any syntax error exists during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It declares continuous assertions and health checks about infrastructure (e.g. verifying an HTTPS endpoint returns status 200) that emit warnings during plan/apply without failing the deployment.
C. It checks Git commit signatures through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It verifies cloud billing limits while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `check` blocks run validation assertions outside the critical path; failures produce warnings rather than halting execution.
- **Why A is Incorrect:** `It blocks `terraform apply` if any syntax error exists during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It checks Git commit signatures through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It verifies cloud billing limits while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
When structuring an enterprise Terraform module repository, where should example configurations be located according to HashiCorp best practices?

A. In an `examples/` subfolder (e.g. `examples/complete/`, `examples/basic/`) containing functional, deployable root modules demonstrating usage patterns.
B. In the `tests/` directory only during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. In the root directory mixed with module code through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. In the `.terraform` folder while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** The standard HashiCorp module layout specifies an `examples/` directory containing complete, runnable example roots.
- **Why B is Incorrect:** `In the `tests/` directory only during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `In the root directory mixed with module code through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `In the `.terraform` folder while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
You are writing a unit test with `terraform test` and want to test a module that depends on an external provider without making real AWS API calls or requiring cloud credentials. What feature introduced in Terraform v1.7+ enables this?

A. Dryrun CLI flag through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. Local state emulation by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Mock providers (`mock_provider "aws" {}`)
D. Dummy backend while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Terraform 1.7 introduced `mock_provider`, allowing test authors to synthesize provider responses without making external network calls.
- **Why A is Incorrect:** `Dryrun CLI flag through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Local state emulation by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Dummy backend while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In a reusable root module, how can you publish a module to a private HCP Terraform / Terraform Enterprise registry?

A. By uploading a zip file via FTP during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. By compiling the code with Go while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. By emailing the code to HashiCorp support through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By tagging a Git repository following the naming convention `terraform-<PROVIDER>-<NAME>` (e.g., `terraform-aws-vpc`) with semantic version tags (e.g., `v1.0.0`).

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** HCP Terraform and Terraform Enterprise registry modules must follow the three-part repository naming convention `terraform-<PROVIDER>-<NAME>`.
- **Why A is Incorrect:** `By uploading a zip file via FTP during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `By compiling the code with Go while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `By emailing the code to HashiCorp support through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What is the function of the `terraform-docs` utility in modern module engineering workflows?

A. It compiles Markdown files into PDF manuals during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It automatically parses module inputs, outputs, providers, and requirements, generating standardized Markdown documentation tables in `README.md`.
C. It audits security CVEs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It executes integration tests while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `terraform-docs` inspects HCL ASTs and maintains synchronized documentation for module interfaces.
- **Why A is Incorrect:** `It compiles Markdown files into PDF manuals during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It audits security CVEs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It executes integration tests while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
In `terraform test`, how can variables be passed to a specific `run` block without affecting subsequent `run` blocks?

A. By setting environment variables in bash during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. By modifying `terraform.tfvars` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. By declaring a nested `variables {}` block inside the specific `run` block.
D. By using `-var` on the command line while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Variables defined inside a `run` block scope apply exclusively to that execution step.
- **Why A is Incorrect:** `By setting environment variables in bash during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `By modifying `terraform.tfvars` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By using `-var` on the command line while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
When using `postcondition` blocks inside a resource lifecycle, what keyword represents the resource's newly created or planned attributes within the condition expression?

A. `self`
B. `each.value` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `this` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `resource` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** In `lifecycle { postcondition {} }`, `self` refers to the attributes of the resource being created or updated.
- **Why B is Incorrect:** ``each.value` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``this` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``resource` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
