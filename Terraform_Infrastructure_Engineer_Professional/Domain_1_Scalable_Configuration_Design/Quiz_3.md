# Domain 1: Scalable Configuration Design — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
You have an optional input variable `backup_retention` that should default to 30 days if not supplied or if explicitly set to `null`. Which modern HCL expression cleanly achieves this fallback?
- A. `coalesce(var.backup_retention, 30)`
- B. `default(var.backup_retention, 30)`
- C. `try(var.backup_retention, 30)`
- D. `if var.backup_retention == null then 30`

---

### Question 2
When designing a reusable module with optional nested object attributes, how can you set default values for individual optional fields without requiring the caller to supply the entire object?
- A. Using the `type = optional(object(...))` syntax with defaults: `optional(string, "default-value")` introduced in Terraform 1.3+.
- B. By splitting each field into 10 separate top-level variables.
- C. By writing a custom Python preprocessor script.
- D. Terraform cannot set defaults for nested object attributes.

---

### Question 3
An enterprise platform module receives a list of CIDR blocks: `["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]`. You need to produce a map where the key is the subnet name (`"subnet-0"`, `"subnet-1"`, etc.) and the value is the CIDR. Which `for` expression accomplishes this?
- A. `{ for idx, cidr in var.cidrs : "subnet-${idx}" => cidr }`
- B. `[ for cidr in var.cidrs : { "subnet" = cidr } ]`
- C. `map(var.cidrs)`
- D. `zipmap(var.cidrs)`

---

### Question 4
You have two maps of configuration tags: `local.default_tags` and `var.custom_tags`. If the same tag key exists in both maps, `var.custom_tags` must override `local.default_tags`. Which HCL function produces the combined map?
- A. `concat(local.default_tags, var.custom_tags)`
- B. `merge(local.default_tags, var.custom_tags)`
- C. `zipmap(local.default_tags, var.custom_tags)`
- D. `union(local.default_tags, var.custom_tags)`

---

### Question 5
Why should platform engineers prefer using `for_each` over `count` when managing stateful infrastructure resources like relational databases, virtual disks, and DNS records?
- A. `for_each` executes parallel API calls while `count` is strictly sequential.
- B. `for_each` identifies resources by unique map keys or set strings rather than integer array indices, preventing accidental resource destruction and recreation when an item is removed from the middle of the list.
- C. `count` is deprecated in Terraform v1.5+.
- D. `for_each` uses less memory in the Terraform CLI.

---

### Question 6
You are writing a reusable module where a resource block needs to be created only if a specific feature flag `var.enable_flow_logs` is `true`. How should the resource be conditionally declared using `count`?
- A. `count = var.enable_flow_logs ? 1 : 0`
- B. `count = if var.enable_flow_logs then 1 else 0`
- C. `count = var.enable_flow_logs`
- D. `enabled = var.enable_flow_logs`

---

### Question 7
In Terraform v1.9+, what capability does `cross-variable validation` provide in `variable` blocks?
- A. It allows validation rules to compare input variables against external HTTP endpoints.
- B. It allows a variable's `condition` expression to reference other input variables within the same module, enforcing cross-field constraints (e.g. `var.min_size <= var.max_size`).
- C. It compiles variables to WebAssembly.
- D. It decrypts secrets before plan evaluation.

---

### Question 8
You need to convert a complex nested list of lists `[["a", "b"], ["c", "d"]]` into a single flat list `["a", "b", "c", "d"]`. Which built-in function does this?
- A. `flatten()`
- B. `compact()`
- C. `distinct()`
- D. `slice()`

---

### Question 9
What does the `sensitive = true` argument on an `output` block accomplish?
- A. It encrypts the value in `terraform.tfstate` with AES-256.
- B. It masks the output value from appearing in plaintext console output (`terraform apply` and CLI logs), preventing casual shoulder-surfing and log leakage.
- C. It deletes the value from memory immediately.
- D. It prevents downstream modules from reading the value.

---

### Question 10
When using `dynamic` blocks inside a resource, what keyword represents the current item of the collection within the `content` block if no label is specified?
- A. `each.value`
- B. `<block_name>.value`
- C. `item`
- D. `self`

---

## Answer Key & Explanations

1. **A** — `coalesce()` returns the first non-null, non-empty string or value from its arguments, making `coalesce(var.backup_retention, 30)` the standard pattern for defaulting nulls.
2. **A** — Terraform 1.3+ introduced optional attributes with defaults in type constraints: `optional(type, default_value)`.
3. **A** — `{ for idx, cidr in var.cidrs : "subnet-${idx}" => cidr }` iterates over index and value to construct a map with custom keys.
4. **B** — `merge()` takes one or more maps and combines them; if identical keys exist, the rightmost map argument takes precedence.
5. **B** — `count` indexes resources by integers (`[0]`, `[1]`, `[2]`). Removing `[1]` causes Terraform to shift index `[2]` into `[1]`, destroying and recreating the downstream resource. `for_each` keys remain stable.
6. **A** — The canonical ternary expression `count = var.enable_flow_logs ? 1 : 0` evaluates to 1 when true and 0 when false.
7. **B** — Cross-variable validation (Terraform 1.9+) allows variable `validation` blocks to reference other variables to enforce relational constraints.
8. **A** — `flatten()` eliminates nested list hierarchies into a single flat 1D list.
9. **B** — Setting `sensitive = true` prevents the CLI from printing the value in terminal output, though the value remains stored in plaintext in the state file.
10. **B** — Inside a `dynamic "ingress"` block, the iterator defaults to the name of the block, accessed as `ingress.value`.
