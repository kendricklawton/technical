# Domain 1: Scalable Configuration Design — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Performance & Code Scenario Questions

---

---

### Question 1
Given the following variable declaration:
```hcl
variable "network_config" {
  type = object({
    cidr_block = string
    dns_servers = optional(list(string), ["1.1.1.1", "8.8.8.8"])
    enable_nat  = optional(bool, false)
  })
}
```
If a user supplies `network_config = { cidr_block = "10.0.0.0/16" }`, what are the effective values of `dns_servers` and `enable_nat`?

A. Both evaluate to `null` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform returns an error stating required arguments are missing.
C. `dns_servers` is `["1.1.1.1", "8.8.8.8"]` and `enable_nat` is `false`.
D. `dns_servers` is `[]` and `enable_nat` is `null` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Using `optional(type, default)` supplies the specified default value if the caller omits that attribute.
- **Why A is Incorrect:** `Both evaluate to `null` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terraform returns an error stating required arguments are missing.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``dns_servers` is `[]` and `enable_nat` is `null` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
You evaluate the expression `cidrsubnet("10.50.0.0/16", 4, 3)`. What is the resulting CIDR block?

A. `10.50.48.0/20`
B. `10.50.3.0/20` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `10.50.32.0/20` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `10.50.16.0/20` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `cidrsubnet("10.50.0.0/16", 4, 3)`: Adding 4 bits gives a `/20` (subnets of 4096 IPs, increments of 16 in the 3rd octet). Index 0 = `10.50.0.0/20`, Index 1 = `10.50.16.0/20`, Index 2 = `10.50.32.0/20`, Index 3 = `10.50.48.0/20`.
- **Why B is Incorrect:** ``10.50.3.0/20` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``10.50.32.0/20` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``10.50.16.0/20` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
Inside a resource `aws_security_group`, you need to generate multiple `ingress` rules dynamically from a variable `var.ingress_rules`. Which syntax correctly accesses the current loop item's port?

A. `var.ingress_rules.port` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `each.value.port` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `self.ingress.port` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `ingress.value.port`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Inside a `dynamic "ingress"` block, the iterator defaults to the block name (`ingress`), and its current object is accessed via `ingress.value`.
- **Why A is Incorrect:** ``var.ingress_rules.port` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``each.value.port` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``self.ingress.port` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
A root module defines:
```hcl
variable "api_key" {
  type      = string
  sensitive = true
}

output "derived_token" {
  value = "token-${var.api_key}"
}
```
What happens when you run `terraform plan`?

A. The plan succeeds and displays the plaintext derived token during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform throws an error because an output deriving from a sensitive input must also specify `sensitive = true`.
C. Terraform automatically strips the `sensitive` flag through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The plan succeeds, but the token value is redacted with `(sensitive value)`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** In Terraform, if an output depends on sensitive data, it must explicitly be marked `sensitive = true`, or Terraform fails during validation/planning.
- **Why A is Incorrect:** `The plan succeeds and displays the plaintext derived token during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform automatically strips the `sensitive` flag through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The plan succeeds, but the token value is redacted with `(sensitive value)`.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
You want to read `var.env_tags[var.environment]`, but if `var.environment` is not a key in the map, you want to fall back to `var.default_tags`. Which expression accomplishes this safely without crashing?

A. Both A and C work correctly.
B. `coalesce(var.env_tags[var.environment], var.default_tags)` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `lookup(var.env_tags, var.environment, var.default_tags)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `try(var.env_tags[var.environment], var.default_tags)` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Both `try()` and `lookup()` provide safe access with fallback defaults.
- **Why B is Incorrect:** ``coalesce(var.env_tags[var.environment], var.default_tags)` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``lookup(var.env_tags, var.environment, var.default_tags)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``try(var.env_tags[var.environment], var.default_tags)` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
A child module internally defines a default provider block:
```hcl
# Inside child module
provider "aws" {
  region = "eu-central-1"
}
```
Why is this an anti-pattern in scalable module design?

A. Child module providers cannot use remote backends through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. It prevents `terraform init` from running by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Child modules are prohibited from containing any provider configurations; providers should always be passed down from the root module.
D. It prevents the module from being published to Terraform Cloud while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Best practice and HashiCorp guidelines dictate that reusable child modules must never declare provider configurations; they must only declare `required_providers`.
- **Why A is Incorrect:** `Child module providers cannot use remote backends through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It prevents `terraform init` from running by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It prevents the module from being published to Terraform Cloud while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
Given the map:
```hcl
locals {
  instances = {
    "web-1" = { public = true,  size = "small" }
    "db-1"  = { public = false, size = "large" }
    "web-2" = { public = true,  size = "medium" }
  }
}
```
Which expression returns only the keys of public instances (`["web-1", "web-2"]`)?

A. `filter(local.instances, "public == true")` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `[for k, v in local.instances : v if k.public]` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `keys(local.instances.public)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `[for k, v in local.instances : k if v.public]`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `[for k, v in local.instances : k if v.public]` filters items by the `if` clause and outputs the key `k`.
- **Why A is Incorrect:** ``filter(local.instances, "public == true")` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``[for k, v in local.instances : v if k.public]` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``keys(local.instances.public)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
Which condition ensures an input variable `ip_address` contains a valid IPv4 address pattern?

A. `condition = contains(var.ip_address, ".")` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. `condition = can(regex("^(\\d{1,3}\\.){3}\\d{1,3}$", var.ip_address))`
C. `condition = cidrhost(var.ip_address, 0) != ""` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `condition = check_ip(var.ip_address)` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `can(regex(...))` safely tests regex patterns against strings without throwing unhandled exceptions.
- **Why A is Incorrect:** ``condition = contains(var.ip_address, ".")` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``condition = cidrhost(var.ip_address, 0) != ""` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``condition = check_ip(var.ip_address)` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
When designing a large-scale enterprise module repository, what is the best practice for separating production modules from local development configurations?

A. Place all environments (`dev`, `stage`, `prod`) in the root directory during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Duplicate the `.tfstate` file into each folder through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Store reusable modules in `/modules/<module-name>` with their own `README.md`, `variables.tf`, `outputs.tf`, and `tests/`, keeping root environments isolated.
D. Never use child modules while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Modular repository structure decouples shared infrastructure logic from environment-specific instantiations.
- **Why A is Incorrect:** `Place all environments (`dev`, `stage`, `prod`) in the root directory during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Duplicate the `.tfstate` file into each folder through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Never use child modules while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
What does `terraform validate` verify that `terraform plan` does not?

A. `terraform validate` checks syntax, type consistency, and module interfaces locally without needing remote state locks or provider API access.
B. `terraform validate` connects to remote cloud APIs to verify IAM permissions during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `terraform validate` applies changes to mock resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform validate` scans for CVEs while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform validate` performs static analysis entirely offline without checking remote state or authenticating to cloud providers.
- **Why B is Incorrect:** ``terraform validate` connects to remote cloud APIs to verify IAM permissions during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform validate` applies changes to mock resources through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform validate` scans for CVEs while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
