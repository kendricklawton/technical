# Domain 1: Scalable Configuration Design — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

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
- A. Only `"billing-queue"` is destroyed.
- B. `"notifications-queue"` at index `[2]` is destroyed and recreated at index `[1]`, causing downtime and message loss.
- C. Terraform updates the resource in-place with zero downtime.
- D. Terraform throws a runtime syntax error.

---

### Question 2
You need to ensure that an input variable `environment` accepts only `"dev"`, `"stage"`, or `"prod"`. How should the `validation` block be configured?
- A.
```hcl
validation {
  condition     = contains(["dev", "stage", "prod"], var.environment)
  error_message = "Environment must be dev, stage, or prod."
}
```
- B.
```hcl
validation {
  condition     = var.environment in ["dev", "stage", "prod"]
  error_message = "Environment must be dev, stage, or prod."
}
```
- C.
```hcl
validation {
  rule          = match(var.environment, "dev|stage|prod")
  error_message = "Environment must be dev, stage, or prod."
}
```
- D.
```hcl
validation {
  assert        = var.environment == "dev" || "stage" || "prod"
  error_message = "Environment must be dev, stage, or prod."
}
```

---

### Question 3
A child module `network` must accept an aliased AWS provider for another region (`aws.secondary`). How must this provider dependency be declared inside the child module's `terraform` block?
- A. `provider "aws" { alias = "secondary" }`
- B. `required_providers { aws = { source = "hashicorp/aws", configuration_aliases = [ aws.secondary ] } }`
- C. `child_providers = ["aws.secondary"]`
- D. `providers = { secondary = aws }`

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
- A. `for_each` only supports lists of strings.
- B. `data.aws_subnets.all.ids` is unknown until apply if dependent on newly created VPC attributes, and `for_each` keys must be known at plan time.
- C. Maps cannot have identical keys and values.
- D. `aws_subnets` data source cannot be iterated.

---

### Question 5
What is the difference between a `precondition` and a `postcondition` in a resource `lifecycle` block?
- A. `precondition` runs before `terraform init`, whereas `postcondition` runs after `terraform apply`.
- B. `precondition` validates inputs before evaluating the resource; `postcondition` validates the resulting resource attributes after it has been created or planned.
- C. `precondition` can only be used on data sources; `postcondition` can only be used on resources.
- D. There is no difference; they are aliases.

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
- A. `["80", "443", ""]`
- B. `["80", null, "443"]`
- C. `["80", "443"]`
- D. `[80, 443]`

---

### Question 7
In Terraform 1.15+, how does the Terraform Model Context Protocol (MCP) server assist AI coding agents?
- A. It automatically writes and executes `terraform apply` commands directly in production clouds.
- B. It exposes local workspace schemas, provider documentation, and validation engines via standard JSON-RPC without manual scraping.
- C. It replaces Terraform state files with an AI-generated vector database.
- D. It compiles HCL directly into C++ binaries.

---

### Question 8
Which Terraform function returns the first non-error expression from a list of arguments, making it ideal for retrieving nested optional object attributes?
- A. `coalesce()`
- B. `try()`
- C. `can()`
- D. `lookup()`

---

### Question 9
You have a map of objects containing nested lists. Which combination of expressions is required to produce a single flat list of all nested items with their parent keys?
- A. `flatten()` combined with a nested `for` expression.
- B. `concat()` combined with `distinct()`.
- C. `merge()` combined with `zipmap()`.
- D. `slice()` combined with `element()`.

---

### Question 10
What happens when `terraform plan` evaluates a variable validation rule whose `condition` evaluates to `false`?
- A. A warning is printed to stdout and planning continues.
- B. The variable is reset to its default value.
- C. Terraform immediately halts execution and prints the specified `error_message`.
- D. Terraform prompts the user interactively on CLI to provide a new value.

---

## Answer Key & Explanations

1. **B** — With `count`, Terraform identifies resources strictly by their array index (`[0]`, `[1]`, `[2]`). Removing `"billing"` shifts `"notifications"` from index `[2]` to `[1]`. Terraform interprets this as modifying index `[1]` to `"notifications"` and destroying index `[2]`.
2. **A** — `contains(list, value)` is the built-in function to verify set membership. Terraform HCL does not have an `in` keyword.
3. **B** — In child modules, all external provider aliases must be declared in `required_providers.<name>.configuration_aliases`.
4. **B** — `for_each` keys must be known during the plan phase so Terraform can construct the resource dependency graph.
5. **B** — `precondition` checks guarantees prior to execution; `postcondition` checks guarantees against the state or remote API response after execution.
6. **C** — `compact()` strips both `null` and empty string `""` values from a list of strings.
7. **B** — The Terraform MCP server provides AI agents with structured introspection of provider schemas, functions, and validation errors.
8. **B** — `try()` takes one or more expressions and evaluates to the first one that does not return an error.
9. **A** — `flatten([for parent_k, parent_v in var.map : [for child in parent_v.items : ...]])` flattens multi-dimensional structures.
10. **C** — Failed variable validations fail fast during the initial parsing/planning stage before any remote API calls are made.
