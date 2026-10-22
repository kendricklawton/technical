# Domain 1: Scalable Configuration Design — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Performance & Code Scenario Questions

---

### Question 1: Complex Type Constraints with Optional Attributes
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
- A. Both evaluate to `null`.
- B. `dns_servers` is `["1.1.1.1", "8.8.8.8"]` and `enable_nat` is `false`.
- C. Terraform returns an error stating required arguments are missing.
- D. `dns_servers` is `[]` and `enable_nat` is `null`.

---

### Question 2: `cidrsubnet()` Calculation
You evaluate the expression `cidrsubnet("10.50.0.0/16", 4, 3)`. What is the resulting CIDR block?
- A. `10.50.3.0/20`
- B. `10.50.48.0/20`
- C. `10.50.32.0/20`
- D. `10.50.16.0/20`

---

### Question 3: Dynamic Nested Block Iteration
Inside a resource `aws_security_group`, you need to generate multiple `ingress` rules dynamically from a variable `var.ingress_rules`. Which syntax correctly accesses the current loop item's port?
- A. `ingress.value.port`
- B. `each.value.port`
- C. `self.ingress.port`
- D. `var.ingress_rules.port`

---

### Question 4: Sensitive Variables & Outputs
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
- A. The plan succeeds and displays the plaintext derived token.
- B. Terraform throws an error because an output deriving from a sensitive input must also specify `sensitive = true`.
- C. Terraform automatically strips the `sensitive` flag.
- D. The plan succeeds, but the token value is redacted with `(sensitive value)`.

---

### Question 5: Safe Map Lookup with Fallback
You want to read `var.env_tags[var.environment]`, but if `var.environment` is not a key in the map, you want to fall back to `var.default_tags`. Which expression accomplishes this safely without crashing?
- A. `try(var.env_tags[var.environment], var.default_tags)`
- B. `coalesce(var.env_tags[var.environment], var.default_tags)`
- C. `lookup(var.env_tags, var.environment, var.default_tags)`
- D. Both A and C work correctly.

---

### Question 6: Overriding Default Provider in Child Module
A child module internally defines a default provider block:
```hcl
# Inside child module
provider "aws" {
  region = "eu-central-1"
}
```
Why is this an anti-pattern in scalable module design?
- A. Child modules are prohibited from containing any provider configurations; providers should always be passed down from the root module.
- B. It prevents `terraform init` from running.
- C. Child module providers cannot use remote backends.
- D. It prevents the module from being published to Terraform Cloud.

---

### Question 7: Filtering with `for` Expressions
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
- A. `[for k, v in local.instances : k if v.public]`
- B. `[for k, v in local.instances : v if k.public]`
- C. `keys(local.instances.public)`
- D. `filter(local.instances, "public == true")`

---

### Question 8: Validating IP Address with Regex
Which condition ensures an input variable `ip_address` contains a valid IPv4 address pattern?
- A. `condition = can(regex("^(\\d{1,3}\\.){3}\\d{1,3}$", var.ip_address))`
- B. `condition = contains(var.ip_address, ".")`
- C. `condition = cidrhost(var.ip_address, 0) != ""`
- D. `condition = check_ip(var.ip_address)`

---

### Question 9: Module Monorepo Organization
When designing a large-scale enterprise module repository, what is the best practice for separating production modules from local development configurations?
- A. Place all environments (`dev`, `stage`, `prod`) in the root directory.
- B. Store reusable modules in `/modules/<module-name>` with their own `README.md`, `variables.tf`, `outputs.tf`, and `tests/`, keeping root environments isolated.
- C. Duplicate the `.tfstate` file into each folder.
- D. Never use child modules.

---

### Question 10: `terraform validate` vs `terraform plan`
What does `terraform validate` verify that `terraform plan` does not?
- A. `terraform validate` connects to remote cloud APIs to verify IAM permissions.
- B. `terraform validate` checks syntax, type consistency, and module interfaces locally without needing remote state locks or provider API access.
- C. `terraform validate` applies changes to mock resources.
- D. `terraform validate` scans for CVEs.

---

## Answer Key & Explanations

1. **B** — Using `optional(type, default)` supplies the specified default value if the caller omits that attribute.
2. **B** — `cidrsubnet("10.50.0.0/16", 4, 3)`: Adding 4 bits gives a `/20` (subnets of 4096 IPs, increments of 16 in the 3rd octet). Index 0 = `10.50.0.0/20`, Index 1 = `10.50.16.0/20`, Index 2 = `10.50.32.0/20`, Index 3 = `10.50.48.0/20`.
3. **A** — Inside a `dynamic "ingress"` block, the iterator defaults to the block name (`ingress`), and its current object is accessed via `ingress.value`.
4. **B** — In Terraform, if an output depends on sensitive data, it must explicitly be marked `sensitive = true`, or Terraform fails during validation/planning.
5. **D** — Both `try()` and `lookup()` provide safe access with fallback defaults.
6. **A** — Best practice and HashiCorp guidelines dictate that reusable child modules must never declare provider configurations; they must only declare `required_providers`.
7. **A** — `[for k, v in local.instances : k if v.public]` filters items by the `if` clause and outputs the key `k`.
8. **A** — `can(regex(...))` safely tests regex patterns against strings without throwing unhandled exceptions.
9. **B** — Modular repository structure decouples shared infrastructure logic from environment-specific instantiations.
10. **B** — `terraform validate` performs static analysis entirely offline without checking remote state or authenticating to cloud providers.
