# Domain 6: Advanced Module Usage and Testing — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Module Refactoring & Testing Architecture Questions

---

---

### Question 1
Can a `run` block in `tests/main.tftest.hcl` execute a test against a nested sub-module located in `./modules/subnet`?

A. No, tests can only evaluate root modules during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Only if the submodule is hosted on GitHub through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Yes, by specifying `module { source = "./modules/subnet" }` inside the `run` block.
D. Only if using Terratest in Go while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** A `run` block can target nested child modules by setting `module { source = "..." }`.
- **Why A is Incorrect:** `No, tests can only evaluate root modules during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Only if the submodule is hosted on GitHub through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Only if using Terratest in Go while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
When using `mock_provider` in `terraform test`, what values are returned for data sources by default?

A. `null` or empty strings unless overridden with `mock_data` blocks.
B. Real data from the internet by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Terraform crashes if a data source is queried under a mock provider.
D. Random UUIDs while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Mock providers return zero-values unless explicitly given fixtures via `mock_data`.
- **Why B is Incorrect:** `Real data from the internet by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform crashes if a data source is queried under a mock provider.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Random UUIDs while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
How can you provide static mock return values for a data source (e.g. `data.aws_ami.ubuntu`) in a `mock_provider` block?

A. By creating a `.json` fixture in `/tmp` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. By compiling a custom Go plugin while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. By intercepting DNS requests through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By using the `mock_data` nested block inside `mock_provider` matching the data source name and defining returned attributes.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** The `mock_data` block allows deterministic mocking of data source outputs for comprehensive unit testing.
- **Why A is Incorrect:** `By creating a `.json` fixture in `/tmp` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `By compiling a custom Go plugin while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `By intercepting DNS requests through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
An engineer needs to remove an input variable `db_password` and replace it with an ephemeral Vault integration in a shared module. Under Semantic Versioning rules, what version increment is required?

A. Patch version increment (`v1.2.0` -> `v1.2.1`) during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Major version increment (`v1.2.0` -> `v2.0.0`) because removing a required input variable is a breaking change for existing module callers.
C. Minor version increment (`v1.2.0` -> `v1.3.0`) by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
D. No version change is needed while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** SemVer rules require major version increments (`X.y.z`) whenever an API's backward compatibility is broken.
- **Why A is Incorrect:** `Patch version increment (`v1.2.0` -> `v1.2.1`) during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Minor version increment (`v1.2.0` -> `v1.3.0`) by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `No version change is needed while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
Why should production root modules pin module versions with exact (`= 2.4.1`) or pessimistic (`~> 2.4.0`) constraints rather than pointing to `main`?

A. Pointing to `main` exposes production to untested breaking changes or regressions pushed to the module repository without code review.
B. Git branches do not support submodules during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Terraform cannot download code from `main` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Registries prohibit git branches while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Pinning prevents breaking changes in upstream modules from unexpectedly cascading into production deployments.
- **Why B is Incorrect:** `Git branches do not support submodules during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform cannot download code from `main` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Registries prohibit git branches while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Why is documenting the `description` and `type` of every output in `outputs.tf` critical for scalable module consumption?

A. Descriptions encrypt outputs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. The Terraform compiler requires descriptions or it refuses to initialize by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Automated documentation generators (like `terraform-docs`) inspect these attributes to render module reference tables and downstream developers rely on them for schema compliance.
D. Descriptions reduce state file size while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `description` and `type` provide essential developer ergonomics and enable automated documentation pipelines.
- **Why A is Incorrect:** `Descriptions encrypt outputs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `The Terraform compiler requires descriptions or it refuses to initialize by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Descriptions reduce state file size while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
When should a module be authored as a nested submodule (`./modules/cache`) rather than a standalone repository?

A. When the module contains secrets while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. Always; standalone module repos are deprecated by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Only when using Docker through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. When the submodule is tightly coupled to the parent module's domain and is rarely consumed independently.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Submodules represent specialized, tightly coupled sub-capabilities packaged within a larger module ecosystem.
- **Why A is Incorrect:** `When the module contains secrets while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Always; standalone module repos are deprecated by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Only when using Docker through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
In a `terraform test` assertion:
```hcl
assert {
  condition     = length(output.subnet_ids) == 3
  error_message = "Module must produce exactly 3 subnet IDs."
}
```
If the module produces only 2 subnet IDs, what does `terraform test` output?

A. A warning in stdout during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. The test run fails with exit code 1, printing the specified `error_message` and the failed expression details.
C. It re-runs the test 3 times through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It adds a 3rd subnet automatically while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Failed assertions cause `terraform test` to exit non-zero, immediately failing CI pipelines.
- **Why A is Incorrect:** `A warning in stdout during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It re-runs the test 3 times through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It adds a 3rd subnet automatically while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
What command formats all HCL files recursively in a module directory tree?

A. `terraform tidy` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. `terraform clean` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `terraform fmt -recursive`
D. `terraform lint --fix` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform fmt -recursive` scans and formats subdirectories and files in-place.
- **Why A is Incorrect:** ``terraform tidy` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform clean` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform lint --fix` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
How can a platform engineering team verify that upgrading the AWS or DigitalOcean provider from v4 to v5 will not cause unplanned resource destruction across their module library?

A. Run `terraform test` across all module test suites using the new provider version constraint; inspect assertion outcomes and plan diffs for unexpected replacements.
B. Deploy to production immediately during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Delete all state files through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Provider upgrades never cause destruction while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Comprehensive automated unit testing harnesses validate that provider schema changes do not trigger unintended resource replacements.
- **Why B is Incorrect:** `Deploy to production immediately during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Delete all state files through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Provider upgrades never cause destruction while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
