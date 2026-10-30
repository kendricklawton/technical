# Domain 6: Advanced Module Usage and Testing — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
What file extension is recognized by the `terraform test` command for test scenario suites?

A. `*.tfspec` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `*.test.tf` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `*.tftest.hcl` (or `*.tftest.json`)
D. `*.unit.tf` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** The native test runner parses `*.tftest.hcl` and `*.tftest.json` files located in the root or `tests/` directory.
- **Why A is Incorrect:** ``*.tfspec` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``*.test.tf` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``*.unit.tf` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
In a `*.tftest.hcl` file, what is the effect of setting `command = plan` inside a `run` block?

A. It runs in unit test mode, generating an in-memory execution plan and checking assertions without making any write calls to remote cloud APIs.
B. It provisions live cloud resources and then plans their deletion during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. It outputs a `.tfplan` file to the root directory through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It tests network connectivity to the cloud provider while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `command = plan` evaluates HCL logic and assertions against the planned state without mutating cloud resources.
- **Why B is Incorrect:** `It provisions live cloud resources and then plans their deletion during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It outputs a `.tfplan` file to the root directory through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It tests network connectivity to the cloud provider while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
How do you test that a module's variable `validation` block correctly rejects invalid inputs using `terraform test`?

A. By catching the exception in Python during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform test cannot test negative failures while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. By setting `ignore_errors = true` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By adding an `expect_failures` argument listing the variable that should fail validation within that `run` block.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `expect_failures` asserts that planned operations fail for the expected variables or preconditions.
- **Why A is Incorrect:** `By catching the exception in Python during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terraform test cannot test negative failures while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `By setting `ignore_errors = true` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
What is the purpose of the `mock_provider` block in `terraform test`?

A. It deploys mock resources into a local Docker container during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It mocks provider interactions in memory, allowing `command = plan` runs to execute completely offline without real cloud credentials or API access.
C. It generates fake billing invoices through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It replaces AWS with LocalStack while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `mock_provider` eliminates external cloud dependencies for fast, cost-free unit testing.
- **Why A is Incorrect:** `It deploys mock resources into a local Docker container during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It generates fake billing invoices through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It replaces AWS with LocalStack while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
When refactoring a monolithic Terraform configuration containing 50 resources into reusable modules, what is the primary architectural criteria for determining module boundaries?

A. Group resources that share a single cohesive lifecycle, common failure domain, or logical business capability (e.g. `networking`, `database`, `compute`).
B. Group all resources created on the same day into one module during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Place every individual resource into its own separate module through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Group resources strictly alphabetically while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Cohesion, shared lifecycle, and bounded context determine optimal module architecture.
- **Why B is Incorrect:** `Group all resources created on the same day into one module during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Place every individual resource into its own separate module through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Group resources strictly alphabetically while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Why should reusable child modules NEVER hardcode `provider` blocks?

A. Child modules do not support providers during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It causes syntax errors during `terraform fmt` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Hardcoding providers inside child modules prevents module consumers from overriding regions, credentials, or aliases, breaks module reusability, and can lead to provider initialization conflicts.
D. HashiCorp charges fees for child module providers while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Providers must be configured at the root level and passed into child modules to preserve flexibility and maintainability.
- **Why A is Incorrect:** `Child modules do not support providers during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It causes syntax errors during `terraform fmt` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `HashiCorp charges fees for child module providers while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
What is the required Git repository naming format for a module published to the public Terraform Registry?

A. `tf-module-<NAME>` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `<NAME>-terraform` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `hashicorp-<NAME>-module` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform-<PROVIDER>-<NAME>`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Public registry indexing requires the 3-part naming format: `terraform-<PROVIDER>-<NAME>`.
- **Why A is Incorrect:** ``tf-module-<NAME>` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``<NAME>-terraform` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``hashicorp-<NAME>-module` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
In a module repository, where should self-contained runnable examples demonstrating how to use the module be placed?

A. In the `/src` folder during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. In an `/examples` directory at the repository root.
C. Inside `/tests/fixtures` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. In the `.gitignore` file while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** The `/examples` directory is the canonical location for reference implementations.
- **Why A is Incorrect:** `In the `/src` folder during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Inside `/tests/fixtures` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `In the `.gitignore` file while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
How does a consumer reference a module hosted in a private HCP Terraform registry?

A. `source = "git::https://github.com/..."` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `source = "local::/modules/vpc"` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `source = "<HOSTNAME>/<ORGANIZATION-NAME>/<MODULE-NAME>/<PROVIDER>"` (e.g., `app.terraform.io/enterprise/vpc/aws`)
D. `source = "s3::https://..."` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Private registry source strings follow the 4-segment convention `<host>/<org>/<name>/<provider>`.
- **Why A is Incorrect:** ``source = "git::https://github.com/..."` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``source = "local::/modules/vpc"` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``source = "s3::https://..."` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
What happens at the end of a `terraform test` execution that used `command = apply`?

A. Terraform automatically destroys all temporary test resources created during the test run.
B. The created test infrastructure remains running indefinitely until manually destroyed.
C. The test resources are migrated to production through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The state file is saved to `/tmp` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform test` guarantees automated cleanup of resources provisioned during integration testing.
- **Why B is Incorrect:** `The created test infrastructure remains running indefinitely until manually destroyed.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `The test resources are migrated to production through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The state file is saved to `/tmp` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
You are authoring an integration test in `tests/rds_integration.tftest.hcl` that uses a helper module (`./testing/vpc_setup`) in the first `run "create_vpc"` block to provision a temporary VPC, and then tests the root RDS module in the second `run "create_rds"` block using `command = apply`. How does the second `run` block reference the `vpc_id` output from the first `run` block, and in what order does `terraform test` destroy the resources when the test finishes?

A. Reference `run.create_vpc.vpc_id`; at test completion, `terraform test` destroys resources in reverse run order (`create_rds` is destroyed first, followed by `create_vpc`).
B. Reference `module.create_vpc.vpc_id`; at test completion, `terraform test` destroys `create_vpc` first and `create_rds` second.
C. Reference `data.terraform_remote_state.create_vpc.outputs.vpc_id`; at test completion, the user must manually run `terraform test -destroy`.
D. Reference `var.create_vpc.vpc_id`; at test completion, both states are destroyed simultaneously in parallel without ordering.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** In `.tftest.hcl`, outputs from an earlier `run "<name>"` block are referenced via `run.<name>.<output_name>`. During automatic teardown, `terraform test` destroys states in **reverse execution order** so dependent downstream resources (`create_rds`) are cleanly removed before foundational prerequisite infrastructure (`create_vpc`).
- **Why B, C, and D are Incorrect:** `run.<name>.*` is the required syntax, and destroying the VPC before the RDS instance inside it would fail with a `DependencyViolation` error!

</details>

---

### Question 12
Your root module calls a child module `module "aurora_db"` that takes 25 minutes to provision in AWS. When writing fast unit tests (`tests/app_unit.tftest.hcl`) for the root module's IAM policies that depend on `module.aurora_db.cluster_arn`, how can you stub out the entire `module.aurora_db` call so `terraform test` does not execute or plan resources inside the child module?

A. Add an `override_module { target = module.aurora_db outputs = { cluster_arn = "arn:aws:rds:..." } }` block inside the `.tftest.hcl` file.
B. Pass `-exclude=module.aurora_db` on the `terraform test` CLI command line.
C. Set `count = 0` on `module "aurora_db"` inside `tests/app_unit.tftest.hcl`.
D. Use `mock_provider "module" { alias = "aurora_db" }` at the top of the test file.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Terraform 1.7+ provides `override_module` (alongside `override_resource` and `override_data`) inside `.tftest.hcl` to intercept a child module call and return synthetic `outputs = { ... }` directly without evaluating the child module's internal resources.
- **Why B, C, and D are Incorrect:** `-exclude`, overriding `count` on a module in `.tftest.hcl`, and `mock_provider "module"` do not exist.

</details>

---

### Question 13
A platform module specifies `version = "~> 4.2.0"` on a Private Registry module dependency, while another workspace specifies `version = "~> 4.2"`. If versions `4.2.0`, `4.2.5`, `4.3.1`, and `5.0.0` are published in the Private Registry, which highest version will each workspace select during `terraform init -upgrade`?

A. Both workspaces select `4.3.1` because trailing zeroes in pessimistic version constraints are ignored by SemVer.
B. The workspace with `~> 4.2.0` selects `4.2.5` (`>= 4.2.0, < 4.3.0`), whereas the workspace with `~> 4.2` selects `4.3.1` (`>= 4.2.0, < 5.0.0`).
C. The workspace with `~> 4.2.0` selects `4.2.5`, whereas the workspace with `~> 4.2` selects `5.0.0`.
D. Both workspaces select `4.2.5` unless `allow_minor_upgrades = true` is set in `required_providers`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** The pessimistic constraint operator `~>` allows only the rightmost specified version component to increment. In `~> 4.2.0`, the rightmost component is the patch digit (`< 4.3.0` -> `4.2.5`). In `~> 4.2`, the rightmost component is the minor digit (`< 5.0.0` -> `4.3.1`).
- **Why A, C, and D are Incorrect:** Trailing zeroes are structurally significant in `~>`, and `~> 4.2` never permits major version `5.0.0`.

</details>

---

### Question 14
Your Platform Engineering team is building a self-service 'Tenant Factory' where application teams open Pull Requests adding `tenants/<team-name>.yaml` files into a directory, and Terraform automatically provisions a standardized namespace module for each YAML file. Which HCL pattern cleanly discovers and decodes all YAML files at plan time?

A. Use `fileset("${path.module}/tenants", "*.yaml")` inside a `locals` `for` comprehension with `yamldecode(file(...))`, keyed by `trimsuffix(f, ".yaml")`, and pass that map to `module.tenant`'s `for_each`.
B. Use a `data "external"` bash script running `ls tenants/*.yaml` and pipe the output into `templatefile()`.
C. Use `import` blocks with wildcard globbing `id = "tenants/*.yaml"`.
D. Use `count = length(file("tenants/*.yaml"))` on the child module block.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `fileset(path, pattern)` enumerates matching files deterministically at plan time, and combining it with `yamldecode(file(...))` in a map keyed by tenant name provides a native, zero-dependency Golden Path module factory.
- **Why B, C, and D are Incorrect:** `data "external"` is unnecessary and non-portable; `import` is for cloud resources; and `file()` cannot read wildcard globs.

</details>

---

### Question 15
You want to publish a newly created internal module to your organization's HCP Terraform Private Registry using VCS tag-based publishing. You connect the GitHub repository `corp-org/aws-eks-golden-module` and push tag `v1.0.0`, but HCP Terraform rejects the repository during registry ingestion. Why?

A. Private Registry tag-based modules must follow the strict repository naming convention `terraform-<PROVIDER>-<NAME>` (e.g., `terraform-aws-eks-golden-module`).
B. Git tags in the Private Registry must omit the leading `v` prefix and be named `release-1.0.0`.
C. Every Private Registry module must contain a compiled Go binary in its `bin/` directory.
D. Modules using `for_each` cannot be published to the Private Registry.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Tag-based modules in the public and private Terraform Registry strictly require the repository name to follow the 3-part format `terraform-<PROVIDER>-<NAME>` so the registry can parse the provider namespace and module name.
- **Why B, C, and D are Incorrect:** `v1.0.0` is valid SemVer, modules are pure HCL (not Go binaries), and all HCL features are supported.

</details>

---
