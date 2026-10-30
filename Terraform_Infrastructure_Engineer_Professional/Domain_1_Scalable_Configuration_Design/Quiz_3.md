# Domain 1: Scalable Configuration Design — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
You have an optional input variable `backup_retention` that should default to 30 days if not supplied or if explicitly set to `null`. Which modern HCL expression cleanly achieves this fallback?

A. `try(var.backup_retention, 30)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. `default(var.backup_retention, 30)` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `coalesce(var.backup_retention, 30)`
D. `if var.backup_retention == null then 30` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `coalesce()` returns the first non-null, non-empty string or value from its arguments, making `coalesce(var.backup_retention, 30)` the standard pattern for defaulting nulls.
- **Why A is Incorrect:** ``try(var.backup_retention, 30)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``default(var.backup_retention, 30)` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``if var.backup_retention == null then 30` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
When designing a reusable module with optional nested object attributes, how can you set default values for individual optional fields without requiring the caller to supply the entire object?

A. Using the `type = optional(object(...))` syntax with defaults: `optional(string, "default-value")` introduced in Terraform 1.3+.
B. By splitting each field into 10 separate top-level variables by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. By writing a custom Python preprocessor script through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Terraform cannot set defaults for nested object attributes while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Terraform 1.3+ introduced optional attributes with defaults in type constraints: `optional(type, default_value)`.
- **Why B is Incorrect:** `By splitting each field into 10 separate top-level variables by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `By writing a custom Python preprocessor script through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform cannot set defaults for nested object attributes while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
An enterprise platform module receives a list of CIDR blocks: `["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]`. You need to produce a map where the key is the subnet name (`"subnet-0"`, `"subnet-1"`, etc.) and the value is the CIDR. Which `for` expression accomplishes this?

A. `zipmap(var.cidrs)` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. `[ for cidr in var.cidrs : { "subnet" = cidr } ]` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `map(var.cidrs)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `{ for idx, cidr in var.cidrs : "subnet-${idx}" => cidr }`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `{ for idx, cidr in var.cidrs : "subnet-${idx}" => cidr }` iterates over index and value to construct a map with custom keys.
- **Why A is Incorrect:** ``zipmap(var.cidrs)` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``[ for cidr in var.cidrs : { "subnet" = cidr } ]` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``map(var.cidrs)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
You have two maps of configuration tags: `local.default_tags` and `var.custom_tags`. If the same tag key exists in both maps, `var.custom_tags` must override `local.default_tags`. Which HCL function produces the combined map?

A. `concat(local.default_tags, var.custom_tags)` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `merge(local.default_tags, var.custom_tags)`
C. `zipmap(local.default_tags, var.custom_tags)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `union(local.default_tags, var.custom_tags)` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `merge()` takes one or more maps and combines them; if identical keys exist, the rightmost map argument takes precedence.
- **Why A is Incorrect:** ``concat(local.default_tags, var.custom_tags)` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``zipmap(local.default_tags, var.custom_tags)` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``union(local.default_tags, var.custom_tags)` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
Why should platform engineers prefer using `for_each` over `count` when managing stateful infrastructure resources like relational databases, virtual disks, and DNS records?

A. `for_each` identifies resources by unique map keys or set strings rather than integer array indices, preventing accidental resource destruction and recreation when an item is removed from the middle of the list.
B. `for_each` executes parallel API calls while `count` is strictly sequential during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `count` is deprecated in Terraform v1.5+ through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `for_each` uses less memory in the Terraform CLI while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `count` indexes resources by integers (`[0]`, `[1]`, `[2]`). Removing `[1]` causes Terraform to shift index `[2]` into `[1]`, destroying and recreating the downstream resource. `for_each` keys remain stable.
- **Why B is Incorrect:** ``for_each` executes parallel API calls while `count` is strictly sequential during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``count` is deprecated in Terraform v1.5+ through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``for_each` uses less memory in the Terraform CLI while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
You are writing a reusable module where a resource block needs to be created only if a specific feature flag `var.enable_flow_logs` is `true`. How should the resource be conditionally declared using `count`?

A. `count = var.enable_flow_logs` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. `count = if var.enable_flow_logs then 1 else 0` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `count = var.enable_flow_logs ? 1 : 0`
D. `enabled = var.enable_flow_logs` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** The canonical ternary expression `count = var.enable_flow_logs ? 1 : 0` evaluates to 1 when true and 0 when false.
- **Why A is Incorrect:** ``count = var.enable_flow_logs` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``count = if var.enable_flow_logs then 1 else 0` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``enabled = var.enable_flow_logs` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In Terraform v1.9+, what capability does `cross-variable validation` provide in `variable` blocks?

A. It allows validation rules to compare input variables against external HTTP endpoints during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It decrypts secrets before plan evaluation while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. It compiles variables to WebAssembly through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It allows a variable's `condition` expression to reference other input variables within the same module, enforcing cross-field constraints (e.g. `var.min_size <= var.max_size`).

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Cross-variable validation (Terraform 1.9+) allows variable `validation` blocks to reference other variables to enforce relational constraints.
- **Why A is Incorrect:** `It allows validation rules to compare input variables against external HTTP endpoints during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It decrypts secrets before plan evaluation while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `It compiles variables to WebAssembly through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
You need to convert a complex nested list of lists `[["a", "b"], ["c", "d"]]` into a single flat list `["a", "b", "c", "d"]`. Which built-in function does this?

A. `compact()` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. `flatten()`
C. `distinct()` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `slice()` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `flatten()` eliminates nested list hierarchies into a single flat 1D list.
- **Why A is Incorrect:** ``compact()` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``distinct()` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``slice()` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
What does the `sensitive = true` argument on an `output` block accomplish?

A. It encrypts the value in `terraform.tfstate` with AES-256 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It deletes the value from memory immediately through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It masks the output value from appearing in plaintext console output (`terraform apply` and CLI logs), preventing casual shoulder-surfing and log leakage.
D. It prevents downstream modules from reading the value while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Setting `sensitive = true` prevents the CLI from printing the value in terminal output, though the value remains stored in plaintext in the state file.
- **Why A is Incorrect:** `It encrypts the value in `terraform.tfstate` with AES-256 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It deletes the value from memory immediately through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It prevents downstream modules from reading the value while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
When using `dynamic` blocks inside a resource, what keyword represents the current item of the collection within the `content` block if no label is specified?

A. `<block_name>.value`
B. `each.value` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `item` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `self` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Inside a `dynamic "ingress"` block, the iterator defaults to the name of the block, accessed as `ingress.value`.
- **Why B is Incorrect:** ``each.value` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``item` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``self` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
