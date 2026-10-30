# Domain 1: Scalable Configuration Design — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
An engineer writes the following configuration to deploy three microservices:
```hcl
variable "services" {
  type = list(string)
  default = ["auth", "billing", "notifications"]
}

resource "aws_sqs_queue" "queues" {
  count = length(var.services)
  name  = "${var.services[count.index]}-queue"
}
```
If the engineer updates `var.services` to `["auth", "notifications"]` (removing `"billing"`), what does `terraform plan` indicate will happen to the `"notifications-queue"`?

A. Only `"billing-queue"` is destroyed during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform updates the resource in-place with zero downtime through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `"notifications-queue"` at index `[2]` is destroyed and recreated at index `[1]`, causing downtime and message loss.
D. Terraform throws a runtime syntax error while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** With `count`, Terraform identifies resources strictly by their array index (`[0]`, `[1]`, `[2]`). Removing `"billing"` shifts `"notifications"` from index `[2]` to `[1]`. Terraform interprets this as modifying index `[1]` to `"notifications"` and destroying index `[2]`.
- **Why A is Incorrect:** `Only `"billing-queue"` is destroyed during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terraform updates the resource in-place with zero downtime through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform throws a runtime syntax error while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
You need to ensure that an input variable `environment` accepts only `"dev"`, `"stage"`, or `"prod"`. How should the `validation` block be configured?

A. ```hcl
B. ```hcl by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. ```hcl through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. ```hcl while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `contains(list, value)` is the built-in function to verify set membership. Terraform HCL does not have an `in` keyword.
- **Why B is Incorrect:** ````hcl by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ````hcl through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ````hcl while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
A child module `network` must accept an aliased AWS provider for another region (`aws.secondary`). How must this provider dependency be declared inside the child module's `terraform` block?

A. `provider "aws" { alias = "secondary" }` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `providers = { secondary = aws }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `child_providers = ["aws.secondary"]` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `required_providers { aws = { source = "hashicorp/aws", configuration_aliases = [ aws.secondary ] } }`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** In child modules, all external provider aliases must be declared in `required_providers.<name>.configuration_aliases`.
- **Why A is Incorrect:** ``provider "aws" { alias = "secondary" }` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``providers = { secondary = aws }` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``child_providers = ["aws.secondary"]` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
An engineer attempts to use `for_each` over a map where the keys are generated dynamically from a remote API call:
```hcl
resource "aws_subnet" "public" {
  for_each   = { for s in data.aws_subnets.all.ids : s => s }
  cidr_block = ...
}
```
Why does `terraform plan` fail?

A. `for_each` only supports lists of strings during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `data.aws_subnets.all.ids` is unknown until apply if dependent on newly created VPC attributes, and `for_each` keys must be known at plan time.
C. Maps cannot have identical keys and values through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `aws_subnets` data source cannot be iterated while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `for_each` keys must be known during the plan phase so Terraform can construct the resource dependency graph.
- **Why A is Incorrect:** ``for_each` only supports lists of strings during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Maps cannot have identical keys and values through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``aws_subnets` data source cannot be iterated while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
What is the difference between a `precondition` and a `postcondition` in a resource `lifecycle` block?

A. `precondition` validates inputs before evaluating the resource; `postcondition` validates the resulting resource attributes after it has been created or planned.
B. `precondition` runs before `terraform init`, whereas `postcondition` runs after `terraform apply` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `precondition` can only be used on data sources; `postcondition` can only be used on resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. There is no difference; they are aliases while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `precondition` checks guarantees prior to execution; `postcondition` checks guarantees against the state or remote API response after execution.
- **Why B is Incorrect:** ``precondition` runs before `terraform init`, whereas `postcondition` runs after `terraform apply` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``precondition` can only be used on data sources; `postcondition` can only be used on resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `There is no difference; they are aliases while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Given the following local expression:
```hcl
locals {
  raw_ports = ["80", null, "443", ""]
  clean_ports = compact(local.raw_ports)
}
```
What is the exact value of `local.clean_ports`?

A. `["80", "443", ""]` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `["80", null, "443"]` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `["80", "443"]`
D. `[80, 443]` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `compact()` strips both `null` and empty string `""` values from a list of strings.
- **Why A is Incorrect:** ``["80", "443", ""]` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``["80", null, "443"]` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``[80, 443]` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In Terraform 1.15+, how does the Terraform Model Context Protocol (MCP) server assist AI coding agents?

A. It automatically writes and executes `terraform apply` commands directly in production clouds.
B. It compiles HCL directly into C++ binaries while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. It replaces Terraform state files with an AI-generated vector database through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It exposes local workspace schemas, provider documentation, and validation engines via standard JSON-RPC without manual scraping.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** The Terraform MCP server provides AI agents with structured introspection of provider schemas, functions, and validation errors.
- **Why A is Incorrect:** `It automatically writes and executes `terraform apply` commands directly in production clouds.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It compiles HCL directly into C++ binaries while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `It replaces Terraform state files with an AI-generated vector database through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
Which Terraform function returns the first non-error expression from a list of arguments, making it ideal for retrieving nested optional object attributes?

A. `coalesce()` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `try()`
C. `can()` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `lookup()` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `try()` takes one or more expressions and evaluates to the first one that does not return an error.
- **Why A is Incorrect:** ``coalesce()` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``can()` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``lookup()` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
You have a map of objects containing nested lists. Which combination of expressions is required to produce a single flat list of all nested items with their parent keys?

A. `merge()` combined with `zipmap()` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. `concat()` combined with `distinct()` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `flatten()` combined with a nested `for` expression.
D. `slice()` combined with `element()` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `flatten([for parent_k, parent_v in var.map : [for child in parent_v.items : ...]])` flattens multi-dimensional structures.
- **Why A is Incorrect:** ``merge()` combined with `zipmap()` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``concat()` combined with `distinct()` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``slice()` combined with `element()` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
What happens when `terraform plan` evaluates a variable validation rule whose `condition` evaluates to `false`?

A. Terraform immediately halts execution and prints the specified `error_message`.
B. The variable is reset to its default value by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. A warning is printed to stdout and planning continues during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
D. Terraform prompts the user interactively on CLI to provide a new value.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Failed variable validations fail fast during the initial parsing/planning stage before any remote API calls are made.
- **Why B is Incorrect:** `The variable is reset to its default value by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `A warning is printed to stdout and planning continues during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform prompts the user interactively on CLI to provide a new value.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
Your Platform Engineering team maintains a Golden Path VPC module. You want to parse an incoming AWS IAM Role ARN (`var.admin_role_arn`) into its structural `account_id` and `resource` components at plan time without making external AWS API calls (`data "aws_arn"`) or maintaining complex `regex()` strings. Assuming Terraform 1.8+ and `hashicorp/aws` v5.40+, which native expression accomplishes this?

A. Invoke the provider-defined function `provider::aws::arn_parse(var.admin_role_arn)` inside a `locals` block after declaring `aws` in `terraform.required_providers`.
B. Invoke the built-in core function `core::arn_decode(var.admin_role_arn)` inside the `variable` validation block before provider initialization.
C. Declare an `ephemeral "aws_arn" "parsed"` block so the ARN components are evaluated in memory without calling the AWS STS endpoint.
D. Use `templatestring(var.admin_role_arn, { delimiter = ":" })` to destructure the ARN segments into a typed `object`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Terraform 1.8+ supports Provider-Defined Functions using the `provider::<provider_name>::<function_name>()` syntax. `provider::aws::arn_parse()` executes locally inside the AWS provider plugin RPC process during `plan`/`validate` with zero network calls.
- **Why B is Incorrect:** Terraform Core does not have a `core::arn_decode()` namespace or cloud-specific ARN parser.
- **Why C is Incorrect:** `aws_arn` is a `data` source or provider function, not an `ephemeral` resource type.
- **Why D is Incorrect:** `templatestring()` renders `${...}` template directives inside a dynamic string; it does not parse or split colon-delimited ARNs.

</details>

---

### Question 12
A developer passes a YAML configuration file into your platform module containing an extra attribute `cost_center_owner = "alice@corp.com"` that is not declared inside your variable's `type = object({ env = string, instance_type = string })` constraint. How does Terraform handle this undeclared attribute during `terraform plan`?

A. Terraform aborts during input evaluation with `Error: Unsupported attribute "cost_center_owner" in object value`.
B. Terraform preserves `cost_center_owner` inside `var.config` as an untyped dynamic field accessible via `try(var.config.cost_center_owner, null)`.
C. Terraform silently discards `cost_center_owner` during structural `object({...})` type conversion so only declared schema attributes exist on `var.config`.
D. Terraform converts the entire `object({...})` into a `map(string)` so all caller keys can be retained.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** When converting an arbitrary map or object into a declared structural `object({ ... })` type constraint, Terraform silently strips any extra attributes not explicitly listed in the `object` schema.
- **Why A is Incorrect:** Extra attributes in a caller's input map/object do not trigger an error during `object` conversion.
- **Why B is Incorrect:** Once stripped by the type converter, `var.config.cost_center_owner` does not exist inside the module.
- **Why D is Incorrect:** Explicit `object({...})` constraints never downgrade themselves to `map(string)`.

</details>

---

### Question 13
You are designing a reusable child module that receives a short-lived bootstrap token via `variable "bootstrap_token" { type = string, ephemeral = true }`. You also need to pass a derived ephemeral connection header back up to the parent root module. How must the child module's `output` block and the root module's consumption be configured?

A. Mark both the child module `output` and the root module `output` with `ephemeral = true` so the value is redacted from CLI logs.
B. Mark the child module `output` with `ephemeral = true`, and consume that output in the root module strictly inside an ephemeral-compatible context (such as a `provider` block, another `ephemeral` block, or a resource write-only `_wo` attribute).
C. Wrap the child module `output` value in `nonsensitive()` so it can be passed across module boundaries without triggering state persistence checks.
D. Store the child module's ephemeral variable inside a `terraform_data` resource's `input` attribute and export `.output` to the root module.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Child module outputs can be marked `ephemeral = true` to pass ephemeral values back to a parent module, whereas root module outputs **cannot** be marked `ephemeral = true` (because root outputs are written to `.tfstate`). The parent module can only pass that child output into non-state-persisted sinks (`provider`, `ephemeral`, or `_wo` attributes).
- **Why A is Incorrect:** Root module outputs cannot use `ephemeral = true`.
- **Why C is Incorrect:** `nonsensitive()` only strips the UI sensitivity flag; it cannot strip the `ephemeral` state-exclusion guarantee.
- **Why D is Incorrect:** `terraform_data` persists its `input` and `output` to `.tfstate`, so referencing an ephemeral value in `terraform_data` is rejected at validation time.

</details>

---

### Question 14
Your platform module provisions an `aws_instance` map (`for_each = var.instances`) and attaches an `aws_eip` to each instance. A junior engineer writes `for_each = toset([for inst in aws_instance.app : inst.id])` on the `aws_eip` resource. Why does `terraform plan` fail on initial deployment, and what is the correct fix?

A. It fails because `toset()` cannot accept a list comprehension; fix it by wrapping the expression in `flatten()`.
B. It fails because `inst.id` is a computed attribute unknown until `apply`, and `for_each` keys must be known at `plan` time; fix it by setting `for_each = aws_instance.app` on `aws_eip` so both resources share the static keys of `var.instances`.
C. It fails because `aws_eip` does not support `for_each`; fix it by switching both resources to `count = length(var.instances)`.
D. It fails because `for_each` requires `-target=aws_instance.app` on the first run; fix it by adding `depends_on = [aws_instance.app]`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Every `for_each` map/set key determines a node address in the DAG (`aws_eip.this["<key>"]`) and must be statically known during `plan`. Chaining `for_each = aws_instance.app` uses the static keys from `var.instances` while exposing `each.value.id` inside the resource body.
- **Why A & C are Incorrect:** `toset()` accepts lists, and all resources support `for_each`.
- **Why D is Incorrect:** `depends_on` does not make computed cloud IDs known during `plan`; only `-target` (an anti-pattern) or static key chaining resolves unknown keys.

</details>

---

### Question 15
You declare a variable `variable "backup_retention" { type = number, default = 14, nullable = false }`. If a parent module calls your module and explicitly sets `backup_retention = null` (for example, via `try(local.tenant.retention, null)` when the tenant omitted the field), what value does `var.backup_retention` take inside your module?

A. Terraform halts immediately with `Error: var.backup_retention may not be null`.
B. It evaluates to `0` because `nullable = false` coerces `null` numbers to their zero value.
C. It automatically substitutes the declared `default = 14` because `nullable = false` instructs Terraform to treat an explicit `null` caller input the same as an omitted argument when a default is present.
D. It evaluates to `null` during `plan` and only applies the default `14` during `apply`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Setting `nullable = false` on a variable that has a `default` value causes Terraform to replace any incoming `null` value with the variable's `default` (`14`). Only if no `default` is defined will passing `null` to `nullable = false` raise an error.
- **Why A, B, and D are Incorrect:** Terraform does not throw an error when a `default` exists alongside `nullable = false`, nor does it coerce to `0` or defer evaluation to `apply`.

</details>

---
