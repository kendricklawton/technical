# Domain 1: Scalable Configuration Design — Complete Study Guide

**Exam Weight:** 20% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** Advanced structural vs collection type constraints, `optional()` with deep defaults, `nullable = false`, `sensitive = true`, `ephemeral = true` variables & outputs (Terraform 1.10+ zero-state persistence), `for_each` vs `count` deep architecture, flattening nested hierarchies (`flatten()` + `setproduct()`), resolving unknown keys in `for_each`, nested `dynamic` blocks, comprehensive built-in function mastery, multi-region/multi-account provider aliasing (`configuration_aliases`), and the Terraform Model Context Protocol (MCP) server.

---

## 1. Architectural Principles of Scalable HCL

Writing scalable Terraform at enterprise scale requires designing configurations that are deterministic, strongly typed, modular, idempotent, and decoupled from environment-specific runtime state.

```text
+-----------------------------------------------------------------------------------+
|                 SCALABLE HCL CONFIGURATION EVALUATION PIPELINE                    |
+-----------------------------------------------------------------------------------+
  [1. Business Requirements (Multi-Region / Multi-Tier / Multi-Tenant)]
                                      |
                                      v
  [2. Input Variables (Structural object({}), optional(T, default), nullable=false)]
                                      |
                                      v
  [3. Custom Variable Validations (Cross-Variable Rules & can()/regex() Checks)]
                                      |
                                      v
  [4. Locals Normalization (flatten(), setproduct(), Deterministic Map Keying)]
                                      |
                                      v
  [5. Resource/Module for_each (Static Plan-Time Known String Keys)]
                                      |
                                      v
  [6. Dynamic Blocks (Nested Sub-Block Generation & Conditional Omission)]
                                      |
                                      v
  [7. Continuous Validation (Preconditions, Postconditions & Check Blocks)]
                                      |
                                      v
  [8. Module Contracts (Typed, Sensitive, or Child-Module Ephemeral Outputs)]
```

---

## 2. Advanced Type Constraints, Structural Types & Variable Modifiers

### 2.1 Collection Types vs Structural Types
Terraform distinguishes between **collection types** (which group multiple values of the *same* homogeneous type) and **structural types** (which group values of *different* heterogeneous types into a fixed schema):

- **Collection Type: `list(T)`**
  - **Ordering & Uniqueness:** Ordered sequence of elements of type `T`, indexed by zero-based integers (`0, 1, 2...`). Duplicates are permitted.
  - **Implicit Conversion:** Automatically converts from `tuple([...])` if all elements can convert to `T`.
  - **Architectural Use Case:** Ordered firewall rules, priority queues, AZ lists where index order matters (`cidrsubnet` calculations).
- **Collection Type: `set(T)`**
  - **Ordering & Uniqueness:** Unordered collection of unique values of type `T`. Duplicates are silently deduplicated; supports only whole-set iteration (`for_each`), not index lookup (`set[0]` is invalid!).
  - **Implicit Conversion:** Automatically converts from `list(T)` or `tuple([...])` (stripping ordering and duplicates). Note: `toset(["b", "a", "b"])` evaluates to `["a", "b"]` in lexicographical order.
  - **Architectural Use Case:** Direct input to resource `for_each` when provisioning homogeneous sets of strings (e.g., IAM role ARNs, subnet IDs, feature flags).
- **Collection Type: `map(T)`**
  - **Ordering & Uniqueness:** Unordered group of values of uniform type `T` identified by unique string keys (sorted lexicographically during iteration).
  - **Implicit Conversion:** Automatically converts from `object({...})` if every attribute value can convert to `T`.
  - **Architectural Use Case:** Resource `tags`, environment lookup tables, homogeneous key-value dictionaries passed to `for_each`.
- **Structural Type: `object({ attr = T, ... })`**
  - **Schema Enforcement:** Fixed set of named attributes where each attribute has its own explicit type constraint (`string`, `number`, `bool`, nested `object`, `list(object(...))`).
  - **Extra Attribute Handling:** If a caller passes an extra key not declared in the `object({...})` type definition, Terraform **silently discards** the undeclared attribute during type conversion! This prevents unknown caller keys from breaking internal module references, though it can surprise engineers who forget to declare a new field in `variables.tf`.
  - **Architectural Use Case:** Strongly typed configuration payloads for complex modules (e.g., EKS cluster configurations, VPC topologies, database cluster specifications).
- **Structural Type: `tuple([T1, T2, ...])`**
  - **Schema Enforcement:** Fixed-length sequence where each exact positional index has its own explicit type constraint (e.g., `tuple([string, number, bool])`).
  - **Architectural Use Case:** Rarely declared directly in `variables.tf`, but frequently returned by HCL expressions such as `[for x in ... : ...]` or `regexall()` before conversion to a `list`.
- **Dynamic Pseudo-Type: `any`**
  - **Behavior:** Placeholder indicating that Terraform should infer a single concrete type at runtime from the passed value.
  - **Exam Trap:** `any` is **NOT** a true dynamic/polymorphic type that permits mixed types inside a collection! For example, `list(any)` forces Terraform to find a single unified element type for all elements in the list (`["a", 1]` becomes `list(string)` `["a", "1"]`, whereas `["a", { k = "v" }]` fails type checking with an inconsistent conditional/element type error). Avoid `any` in production module contracts in favor of explicit `object({...})` schemas.

### 2.2 `optional(T, default)` Modifier for Nested Objects
Introduced in Terraform 1.3+, the `optional()` modifier allows specific attributes inside an `object({...})` type constraint to be omitted by the caller, substituting either `null` (when 1 argument is passed: `optional(T)`) or a concrete default value (when 2 arguments are passed: `optional(T, default)`):

```hcl
variable "eks_node_groups" {
  description = "Map of EKS managed node group specifications with nested optional defaults"
  type = map(object({
    instance_types = optional(list(string), ["m6i.large", "m6a.large"])
    capacity_type  = optional(string, "ON_DEMAND")
    disk_size_gb   = optional(number, 100)
    ami_type       = optional(string) # Defaults to null if omitted by caller
    scaling_config = optional(object({
      min_size     = optional(number, 2)
      max_size     = optional(number, 10)
      desired_size = optional(number, 3)
    }), {}) # Critical: Passing "{}" as the default for the parent object triggers inner defaults!
    labels = optional(map(string), {})
    taints = optional(list(object({
      key    = string
      value  = optional(string)
      effect = string
    })), [])
  }))

  # Prevent caller from explicitly passing `eks_node_groups = null`
  nullable = false

  validation {
    condition = alltrue([
      for ng_name, ng in var.eks_node_groups :
      contains(["ON_DEMAND", "SPOT"], ng.capacity_type)
    ])
    error_message = "Every node group capacity_type must be either ON_DEMAND or SPOT."
  }

  validation {
    condition = alltrue([
      for ng_name, ng in var.eks_node_groups :
      ng.scaling_config.min_size <= ng.scaling_config.desired_size &&
      ng.scaling_config.desired_size <= ng.scaling_config.max_size
    ])
    error_message = "Node group scaling_config must satisfy: min_size <= desired_size <= max_size."
  }
}
```

> [!IMPORTANT]
> **Nested `optional()` Default Evaluation Trap:** If an outer attribute is `scaling_config = optional(object({ min_size = optional(number, 2) }))` *without* the outer `, {}` default, omitting `scaling_config` sets `scaling_config = null`. Only when you specify `optional(object({...}), {})` will Terraform instantiate an empty object when `scaling_config` is omitted and recursively populate its nested `min_size = 2` default! Furthermore, if a caller explicitly passes `disk_size_gb = null`, Terraform converts that explicit `null` into the declared default `100`.

### 2.3 Variable & Output Modifiers: `nullable`, `sensitive`, and `ephemeral`
Understanding how Terraform handles nullability, UI redaction, and state persistence is heavily tested:

- **`nullable = false` (Default is `true`)**
  - **Behavior:** By default (`nullable = true`), passing `var_name = null` overrides any `default = ...` defined on the variable and sets the variable to `null` inside the module. Setting `nullable = false` guarantees the variable value can **never** be `null`; if a caller passes `null` to a variable with `nullable = false` and a `default` value is present, Terraform automatically falls back to the variable's `default` value instead of failing! If no `default` is defined and `nullable = false`, passing `null` raises an immediate validation error.
- **`sensitive = true` (Variables & Outputs)**
  - **Behavior:** Redacts the value from CLI console output (`(sensitive value)`) during `terraform plan` and `terraform apply`, and taint-propagates sensitivity to any downstream expression derived from it (e.g., `local.connection_string`).
  - **Security Limitation:** `sensitive = true` **DOES NOT prevent the value from being written in plaintext to `terraform.tfstate` and saved binary plan files (`tfplan.binary`)**!
  - **Functions:** Use `nonsensitive(expr)` to strip the sensitive mark when you have verified a derived value (like a public hostname extracted from a sensitive object) is safe to display, or `sensitive(expr)` to dynamically mark an expression as sensitive.
- **`ephemeral = true` (Terraform 1.10+ Variables & Outputs)**
  - **Behavior:** Declares that the value exists **strictly in memory during the current execution phase** and is **NEVER persisted to `terraform.tfstate` or saved plan files**.
  - **Strict Reference Rules:** Ephemeral variables (`variable "session_token" { ephemeral = true }`) can ONLY be referenced in contexts that do not persist values to state:
    1. Provider configuration blocks (`provider "aws" { token = var.session_token }`).
    2. Resource **Write-Only Attributes** (Terraform 1.11+ attributes such as `password_wo` on `aws_db_instance` or `secret_string_wo` on `aws_secretsmanager_secret_version`).
    3. `ephemeral` resource blocks (`ephemeral "aws_secretsmanager_secret_version" "db" { ... }`).
    4. `locals` (which themselves become implicitly ephemeral if derived from an ephemeral value).
    5. Child module `output` blocks marked with `ephemeral = true` (note: root module outputs *cannot* be marked `ephemeral = true` because root outputs are persisted in state; only child module outputs can use `ephemeral = true` to pass ephemeral data back to parent modules!).
    6. Provisioner and `connection` blocks.

```hcl
variable "Temporary_DB_Bootstrap_Token" {
  description = "Short-lived bootstrap credential passed only at runtime; never stored in state"
  type        = string
  sensitive   = true
  ephemeral   = true
}

# Child module output passing ephemeral value up to the parent module:
output "ephemeral_client_token" {
  description = "Ephemeral token passed between child module and root provider/write-only attribute"
  value       = var.Temporary_DB_Bootstrap_Token
  ephemeral   = true
}
```

---

## 3. `for_each` vs `count` Deep Architecture

### 3.1 The `count` Index-Shift Destruction Hazard
When `count` is used with a list of items (`count = length(var.subnet_cidrs)`), Terraform tracks each resource instance in state by its numerical array index: `aws_subnet.private[0]`, `aws_subnet.private[1]`, `aws_subnet.private[2]`.

```text
+-----------------------------------------------------------------------------------+
|                     THE count INDEX-SHIFT DESTRUCTION HAZARD                      |
+-----------------------------------------------------------------------------------+
  BEFORE (var.azs = ["us-east-1a", "us-east-1b", "us-east-1c"]):
    aws_subnet.app[0]  --->  us-east-1a
    aws_subnet.app[1]  --->  us-east-1b
    aws_subnet.app[2]  --->  us-east-1c

  AFTER REMOVING "us-east-1a" (var.azs = ["us-east-1b", "us-east-1c"]):
    aws_subnet.app[0]  --->  Mutated 1a -> 1b (FORCED REPLACEMENT: Destroy & Recreate!)
    aws_subnet.app[1]  --->  Mutated 1b -> 1c (FORCED REPLACEMENT: Destroy & Recreate!)
    aws_subnet.app[2]  --->  Index out of bounds (DESTROYED!)
+-----------------------------------------------------------------------------------+
  FIX: Use for_each = toset(var.azs) -> Addressed as aws_subnet.app["us-east-1a"].
       Removing "us-east-1a" ONLY destroys aws_subnet.app["us-east-1a"]!
+-----------------------------------------------------------------------------------+
```

- **When to use `count`:**
  1. Conditional boolean resource creation: `count = var.create_nat_gateway ? 1 : 0` (reference via `one(aws_nat_gateway.this[*].id)`).
  2. Truly identical, stateless, interchangeable resources where individual identity never changes independently.
- **When to use `for_each`:**
  1. Almost all multi-instance infrastructure (Subnets, Route Tables, Security Group Rules, IAM Roles, EKS Node Groups, S3 Buckets) where adding or removing an item in the middle of the collection must only affect that specific key (`aws_subnet.app["us-east-1a"]`).

### 3.2 Transforming Lists of Objects into Deterministic Keyed Maps
Because `for_each` accepts only a `map(any)` or `set(string)` (it rejects `list(object(...))` directly), you must transform lists of objects into a map keyed by a deterministic, unique string identifier using a `for` expression:

```hcl
variable "service_accounts" {
  type = list(object({
    env         = string
    team        = string
    role_suffix = string
    policies    = list(string)
  }))
}

locals {
  # Construct a composite deterministic key guaranteed to be unique across items:
  # Key format: "prod-payments-reader"
  sa_map = {
    for sa in var.service_accounts :
    "${sa.env}-${sa.team}-${sa.role_suffix}" => sa
  }
}

resource "aws_iam_role" "service" {
  for_each = local.sa_map

  name               = "svc-${each.key}"
  assume_role_policy = data.aws_iam_policy_document.oidc_trust.json

  tags = {
    Environment = each.value.env
    Team        = each.value.team
  }
}
```

> [!TIP]
> **Handling Duplicate Keys with Grouping Mode (`...`):** In a map `for` expression `{ for item in var.list : item.key => item }`, if two items share the same `item.key`, Terraform throws a `Duplicate object key` error. Adding the ellipsis grouping operator `{ for item in var.list : item.key => item... }` groups values with identical keys into a `list` under that key!

### 3.3 Flattening Nested Hierarchies (`flatten` + `for_each`)
A classic exam scenario requires provisioning child resources across a 1-to-Many or Many-to-Many hierarchy (e.g., VPCs -> Subnets -> Route Table Associations, or IAM Roles -> Multiple Managed Policy Attachments). Because `for_each` only iterates over a single flat level, you must flatten the nested lists in `locals` first and project them into a flat map with composite keys:

```hcl
variable "vpcs" {
  type = map(object({
    cidr_block = string
    subnets = map(object({
      cidr_block = string
      az         = string
      public     = optional(bool, false)
    }))
  }))
}

locals {
  # Step 1: Flatten the 2-level hierarchy (VPC -> Subnets) into a flat list of objects
  flattened_subnets = flatten([
    for vpc_key, vpc in var.vpcs : [
      for subnet_key, subnet in vpc.subnets : {
        composite_key = "${vpc_key}.${subnet_key}"
        vpc_key       = vpc_key
        subnet_key    = subnet_key
        cidr_block    = subnet.cidr_block
        az            = subnet.az
        public        = subnet.public
      }
    ]
  ])

  # Step 2: Project the flat list into a map keyed by static composite_key
  subnets_by_key = {
    for item in local.flattened_subnets : item.composite_key => item
  }
}

resource "aws_vpc" "this" {
  for_each   = var.vpcs
  cidr_block = each.value.cidr_block
}

resource "aws_subnet" "this" {
  for_each = local.subnets_by_key

  # Reference parent VPC using static vpc_key lookup!
  vpc_id                  = aws_vpc.this[each.value.vpc_key].id
  cidr_block              = each.value.cidr_block
  availability_zone       = each.value.az
  map_public_ip_on_launch = each.value.public

  tags = {
    Name = each.key
  }
}
```

### 3.4 Resolving the Unknown Keys in `for_each` Error
> [!CAUTION]
> **The Cardinal Rule of `for_each` Keys:**  
> Every **key** in a `for_each` map (or every element in a `for_each` set of strings) MUST be statically known during the initial `terraform plan` graph construction! However, the **values** in a `for_each` map *are* allowed to contain unknown/computed attributes (`(known after apply)`).

#### Anti-Pattern (Triggers `Invalid for_each argument` Error):
```hcl
# BROKEN: aws_subnet.public[*].id is unknown until AWS creates the subnets during apply!
resource "aws_route_table_association" "broken" {
  for_each       = toset(aws_subnet.public[*].id)
  subnet_id      = each.value
  route_table_id = aws_route_table.public.id
}
```
Running `terraform plan` on the above configuration fails with:
`Error: Invalid for_each argument — The "for_each" map includes keys derived from resource attributes that cannot be determined until apply, and so Terraform cannot determine the full set of keys that will identify the instances of this resource.`

#### Production Fix (Separate Static Keys from Computed Values):
Always key the `for_each` map using the static input keys (or direct resource map reference), and reference the computed `.id` inside `each.value`:

```hcl
# FIXED PATTERN 1: Pass the upstream resource map directly to for_each!
# Because aws_subnet.this was created with static keys ("prod.web-a"), its keys are known at plan time!
resource "aws_route_table_association" "correct_chained" {
  for_each       = aws_subnet.this
  subnet_id      = each.value.id # Computed value is in each.value, NOT each.key!
  route_table_id = aws_route_table.public[each.value.vpc_key].id
}

# FIXED PATTERN 2: Map static keys to computed attributes
resource "aws_route_table_association" "correct_mapped" {
  for_each = {
    for k, subnet in aws_subnet.this : k => subnet.id
    if local.subnets_by_key[k].public == true
  }
  subnet_id      = each.value
  route_table_id = aws_route_table.public[local.subnets_by_key[each.key].vpc_key].id
}
```

> [!WARNING]
> Avoid using `-target=aws_subnet.public` as a workaround for `Invalid for_each argument`. While applying the upstream resource first populates its IDs in state, doing so breaks single-pass infrastructure provisioning (making `terraform apply` from scratch or `terraform test` fail without multi-stage scripts).

---

## 4. Dynamic Blocks & Collection Functions Mastery

### 4.1 Nested `dynamic` Blocks, Custom `iterator`, and Conditional Block Omission
Resource configuration blocks (such as `ingress`, `Origin`, `rule`, or `logging`) cannot be conditionally omitted by setting `logging = null` if they are schema blocks rather than attributes. Instead, use `dynamic` blocks:

1. **Conditional Block Omission Pattern:** Set `for_each = var.logging_config != null ? [var.logging_config] : []` (or `var.enable_encryption ? [1] : []`). When the list is empty `[]`, zero blocks are generated; when populated, exactly one block is rendered.
2. **Nested Dynamic Blocks & Custom `iterator`:** When nesting a `dynamic` block inside another `dynamic` block, always define explicit `iterator` labels to avoid shadowing the default block name variable:

```hcl
variable "waf_rule_groups" {
  type = list(object({
    name     = string
    priority = number
    action   = string
    statements = list(object({
      country_codes = list(string)
      forwarded_ip_config = optional(object({
        header_name       = string
        fallback_behavior = string
      }))
    }))
  }))
}

resource "aws_wafv2_web_acl" "enterprise" {
  name  = "enterprise-edge-waf"
  scope = "REGIONAL"

  default_action {
    allow {}
  }

  # Outer dynamic block iterating over rule groups
  dynamic "rule" {
    for_each = { for rg in var.waf_rule_groups : rg.name => rg }
    iterator = rule_group

    content {
      name     = rule_group.value.name
      priority = rule_group.value.priority

      action {
        dynamic "allow" {
          for_each = rule_group.value.action == "allow" ? [1] : []
          content {}
        }
        dynamic "block" {
          for_each = rule_group.value.action == "block" ? [1] : []
          content {}
        }
      }

      statement {
        or_statement {
          # Inner nested dynamic block iterating over statements inside each rule group
          dynamic "statement" {
            for_each = rule_group.value.statements
            iterator = stmt

            content {
              geo_match_statement {
                country_codes = stmt.value.country_codes

                # Conditional nested block omission when optional object is null:
                dynamic "forwarded_ip_config" {
                  for_each = stmt.value.forwarded_ip_config != null ? [stmt.value.forwarded_ip_config] : []
                  iterator = fwd_ip

                  content {
                    header_name       = fwd_ip.value.header_name
                    fallback_behavior = fwd_ip.value.fallback_behavior
                  }
                }
              }
            }
          }
        }
      }

      visibility_config {
        cloudwatch_metrics_enabled = true
        metric_name                = "waf-${rule_group.key}"
        sampled_requests_enabled   = true
      }
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "enterprise-edge-waf"
    sampled_requests_enabled   = true
  }
}
```

### 4.2 Deep Mastery of Built-In HCL Functions
On the Professional exam, you must be able to read and write complex functional transformations without documentation:

- **Collection & Map Manipulation Functions**
  - **`merge(map1, map2, ...)`:** Shallow-merges maps or objects from left to right. If duplicate keys exist, **right-most argument wins** (`merge(var.default_tags, var.env_tags, { Name = "web" })`). Note: `merge` is shallow; it does not recursively merge nested maps!
  - **`lookup(map, key, default)`:** Retrieves `map[key]`, returning `default` if `key` is not present. In modern HCL, prefer `try(map[key], default)` or typed `object` with `optional()`.
  - **`one(list_or_set)`:** Takes a list, set, or tuple with either **zero or one** elements. If empty `[]`, returns `null`. If 1 element `["vpc-123"]`, returns that element unwrapped (`"vpc-123"`). If 2+ elements exist, throws an error! Essential for safely referencing resources with `count = var.enabled ? 1 : 0` via `one(aws_kms_key.custom[*].arn)`.
  - **`setproduct(col1, col2, ...)`:** Computes the **Cartesian product** of two or more sets/lists. For example, `setproduct(["us-east-1", "us-west-2"], ["web", "db"])` produces 4 pairs: `[["us-east-1", "web"], ["us-east-1", "db"], ["us-west-2", "web"], ["us-west-2", "db"]]`. Cleaner than nested `for` loops when pairing homogeneous sets (e.g., peering every VPC in list A with every VPC in list B).
  - **`zipmap(keys_list, values_list)`:** Constructs a `map` from a list of string keys and a corresponding list of values of equal length: `zipmap(["a", "b"], [1, 2])` -> `{ a = 1, b = 2 }`.

- **Defensive Fallback & Error Handling Functions**
  - **`coalesce(arg1, arg2, ...)`:** Returns the first argument that is **not `null` and not an empty string `""`**. All arguments must convert to the same primitive type.
  - **`coalescelist(list1, list2, ...)`:** Returns the first list/tuple argument that is **not `null` and not empty `[]`**.
  - **`try(expr1, expr2, ...)`:** Evaluates expressions sequentially and returns the first one that does not produce a dynamic error. Unlike `coalesce`, `try(local.obj.missing_key, "fallback")` catches attribute-lookup errors on unstructured data (`yamldecode` / `jsondecode` outputs). Note: If `local.obj.existing_key` exists and its value is `null`, `try(local.obj.existing_key, "fallback")` returns `null` (because evaluating `null` is not an error)! Combine `coalesce(try(local.obj.key, null), "fallback")` to guard against both missing keys and `null` values.
  - **`can(expr)`:** Returns a boolean (`true` or `false`) indicating whether `expr` evaluates without runtime error. Widely used inside `validation`, `precondition`, and `postcondition` blocks (e.g., `can(cidrnetmask(var.vpc_cidr))` or `can(regex("^arn:aws:", var.role_arn))`).

- **IP Networking & CIDR Math Functions**
  - **`cidrsubnet(prefix, newbits, netnum)`:** Calculates a subnet CIDR within a parent CIDR block:
    - `prefix`: Parent CIDR string (e.g., `"10.10.0.0/16"`).
    - `newbits`: Additional bits added to the prefix length (`16 + 4 = /20`, or `16 + 8 = /24`).
    - `netnum`: Zero-based integer index of the subnet slice.
    - *Example:* `cidrsubnet("10.10.0.0/16", 8, 3)` adds 8 bits (`/24`) and selects slice `3` -> `"10.10.3.0/24"`.
    - *Example:* `cidrsubnet("10.10.0.0/16", 4, 2)` adds 4 bits (`/20`, which increments the 3rd octet by $2^{8-4} = 16$) and selects slice `2` ($2 \times 16 = 32$) -> `"10.10.32.0/20"`.
  - **`cidrsubnets(prefix, newbits1, newbits2, ...)`:** Allocates a sequence of contiguous non-overlapping subnets of **varying sizes** from a parent prefix (e.g., `cidrsubnets("10.0.0.0/16", 4, 4, 8, 8)` allocates two `/20` subnets followed by two `/24` subnets without wasting address space).
  - **`cidrhost(prefix, hostnum)`:** Calculates the full IP address for a given host number within a subnet CIDR (`cidrhost("10.10.1.0/24", 5)` -> `"10.10.1.5"`; negative numbers count backward from the broadcast address: `cidrhost("10.10.1.0/24", -2)` -> `"10.10.1.254"`).

- **Templating & Serialization Functions**
  - **`templatefile(path, vars_map)`:** Reads an external file at `path` during plan evaluation and renders its embedded Terraform template syntax (`${var}`, `%{ if ... }`, `%{ for ... }`) using `vars_map`. Cannot be used with dynamically generated strings created during the same run.
  - **`templatestring(string_expr, vars_map)` (Terraform 1.9+):** Renders a template stored in a dynamic string (such as a template fetched from a data source, SSM parameter, or input variable) rather than requiring a static local disk file path!
  - **`yamldecode(str)` / `jsondecode(str)`:** Parses YAML or JSON configuration files (`yamldecode(file("${path.module}/tenants.yaml"))`) into native Terraform HCL objects/maps/lists for data-driven infrastructure factories.
  - **`jsonencode(val)` / `yamlencode(val)`:** Serializes native HCL maps/objects into valid JSON/YAML strings (preferred over heredoc interpolation for IAM policies and container definitions because it prevents syntax injection and handles escaping automatically).

### 4.3 Provider-Defined Functions (Terraform 1.8+)
Starting in Terraform 1.8, providers can export custom, domain-specific pure functions invoked via the namespace syntax **`provider::<provider_name>::<function_name>(args...)`** (or `provider::<provider_name>::<alias>::<function_name>()` when provider configuration is required):

```hcl
locals {
  # Parse an AWS ARN into its structural components (partition, service, region, account_id, resource)
  # without brittle regex() or split(":", arn) expressions!
  parsed_role_arn = provider::aws::arn_parse("arn:aws:iam::123456789012:role/platform/EKSAdminRole")
  # parsed_role_arn.account_id -> "123456789012"
  # parsed_role_arn.service    -> "iam"
  # parsed_role_arn.resource   -> "role/platform/EKSAdminRole"

  # Decode a multi-document Kubernetes YAML manifest string via the Kubernetes provider:
  k8s_manifests = provider::kubernetes::manifest_decode_multi(file("${path.module}/crds.yaml"))

  # Trim IAM policy JSON whitespace for canonical comparison:
  canonical_policy = provider::aws::iam_policy_document_normalize(local.raw_policy_json)
}
```
* **Key Exam Rules for Provider-Defined Functions:**
  * The provider **must be declared** in `terraform.required_providers` before calling `provider::<name>::<fn>()`.
  * Provider-defined functions execute offline inside the local provider plugin RPC process during `plan`/`validate` without making external cloud API calls (unlike `data` sources).

---

## 5. Provider Aliasing & Multi-Region / Multi-Account Topologies

### 5.1 How Provider Inheritance & Aliasing Works
By default, a root module defines a default (unaliased) `provider "aws" {}` block, and every resource or child module automatically inherits that default provider unless overridden. When deploying multi-region (e.g., Primary + DR) or multi-account (e.g., Shared Services Transit Gateway + Workload VPCs) architectures within a single workspace, you define additional `provider` blocks with the `alias` meta-argument.

```text
+-----------------------------------------------------------------------------------+
|           MULTI-REGION / CROSS-ACCOUNT PROVIDER ALIASING ARCHITECTURE             |
+-----------------------------------------------------------------------------------+
  ROOT MODULE (providers.tf & main.tf)
  +-------------------------------------------------------------------------------+
  |  provider "aws" {}                         (Default: us-east-1)               |
  |  provider "aws" { alias = "usw2" }         (DR Region: us-west-2)             |
  |  provider "aws" { alias = "sec_acct" }     (AssumeRole: Security Account)     |
  +-------------------+---------------------------------------+-------------------+
                      |                                       |
      providers = {   |                                       |   providers = {
        aws.hub = aws |                                       |     aws.spoke = aws.usw2
      }               v                                       v   }
  +-------------------------------------------------------------------------------+
  |  CHILD MODULE (modules/multi_region_tgw)                                      |
  |  terraform {                                                                  |
  |    required_providers {                                                       |
  |      aws = {                                                                  |
  |        source                = "hashicorp/aws"                                |
  |        configuration_aliases = [aws.hub, aws.spoke]                           |
  |      }                                                                        |
  |    }                                                                          |
  |  }                                                                            |
  |  +-----------------------------------+  +----------------------------------+  |
  |  | aws_ec2_transit_gateway_peering   |  | aws_ec2_tgw_peering_accepter     |  |
  |  | provider = aws.hub                |  | provider = aws.spoke             |  |
  |  +-----------------------------------+  +----------------------------------+  |
  +-------------------------------------------------------------------------------+
```

### 5.2 Declaring `configuration_aliases` in Child Modules
A child module that needs to provision resources across multiple provider configurations must declare `configuration_aliases` inside its `terraform.required_providers` block. This informs Terraform's dependency graph that the parent module must pass explicit provider mappings:

```hcl
# modules/cross_account_vpc_peering/versions.tf
terraform {
  required_version = ">= 1.10.0"

  required_providers {
    aws = {
      source                = "hashicorp/aws"
      version               = "~> 5.0"
      configuration_aliases = [aws.requester, aws.accepter]
    }
  }
}

# modules/cross_account_vpc_peering/main.tf
resource "aws_vpc_peering_connection" "peer" {
  provider    = aws.requester
  vpc_id      = var.requester_vpc_id
  peer_vpc_id = var.accepter_vpc_id
  peer_region = data.aws_region.accepter.name
  peer_owner_id = data.aws_caller_identity.accepter.account_id
}

resource "aws_vpc_peering_connection_accepter" "accept" {
  provider                  = aws.accepter
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
  auto_accept               = true
}
```

In the **Root Module**, pass the mapped providers via the `providers` meta-argument:

```hcl
# Root module: providers.tf
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "dr_west"
  region = "us-west-2"
  assume_role {
    role_arn = "arn:aws:iam::999988887777:role/TerraformDRRole"
  }
}

# Root module: main.tf
module "vpc_peering_dr" {
  source = "./modules/cross_account_vpc_peering"

  providers = {
    aws.requester = aws
    aws.accepter  = aws.dr_west
  }

  requester_vpc_id = module.vpc_primary.vpc_id
  accepter_vpc_id  = module.vpc_dr.vpc_id
}
```

### 5.3 Why Inline `provider` Blocks Inside Reusable Modules Are an Anti-Pattern
> [!CAUTION]
> **Never define a `provider "..." { ... }` configuration block inside a reusable child module!**
> Defining an inline `provider` block inside a child module creates critical architectural failures:
> 1. **Blocks `for_each`, `count`, and `depends_on` on the Module Call:** Terraform cannot dynamically instantiate or destroy provider plugin processes mid-graph. If a child module contains its own `provider` configuration block, calling that module with `for_each` or `count` fails immediately during `terraform validate` / `plan` with: `Error: Module is incompatible with count, for_each, and depends_on (legacy module with provider configuration)`.
> 2. **Orphaned State Destruction Deadlock:** If you later remove the `module "x"` block from your root configuration so its resources can be destroyed, the inline `provider` block inside the removed module also disappears! Terraform then lacks the provider configuration required to authenticate and delete the remaining resources recorded in `.tfstate`, resulting in an unresolvable `Provider configuration not present` error.

---

## 6. Terraform Model Context Protocol (MCP) Server

In modern Terraform 1.15+ workflows, the **Terraform MCP Server** (`hashicorp/terraform-mcp-server`) provides a standardized JSON-RPC interface connecting AI engineering agents to live provider schemas and registry metadata:

- **Capabilities:** Queries exact resource/data-source schemas from the Terraform Registry and Private Registry, inspects required vs optional vs computed attributes, retrieves module input/output contracts, and validates HCL expressions without hallucinating deprecated provider arguments.
- **Security Invariant:** Operates as a read-only context and validation server; it never mutates remote state or executes cloud write operations autonomously without passing through standard `terraform plan` / policy gates.

---

## 7. Domain 1 Production Scenario & Exam Trap Matrix

- **Scenario 1: Removing an element from the middle of a list used with `count`**
  - **Trap:** Assuming Terraform matches resources by `tags["Name"]` or subnet CIDR.
  - **Reality:** Terraform matches `count` instances purely by numeric index `[0], [1], [2]`. Removing index `0` shifts every subsequent element down by 1, triggering cascading updates or forced replacements across the entire array and destroying the last index `[N-1]`.
  - **Fix:** Convert to `for_each = { for item in var.list : item.unique_id => item }` and migrate existing state addresses via `moved` blocks.
- **Scenario 2: Passing a caller object with an undeclared field into `type = object({...})`**
  - **Trap:** Expecting Terraform to fail with an "unknown attribute" error when a caller passes an extra key in a `.tfvars` file.
  - **Reality:** Structural `object({...})` type conversion silently strips any undeclared attributes not listed in the type definition.
- **Scenario 3: Omitting a nested object attribute that has inner `optional()` defaults**
  - **Trap:** Writing `logging = optional(object({ enabled = optional(bool, true) }))` and expecting `var.config.logging.enabled` to be `true` when `logging` is omitted.
  - **Reality:** If `logging` is omitted and has no outer default, `var.config.logging` evaluates to `null`, causing `var.config.logging.enabled` to fail with `Attempt to get attribute from null value`.
  - **Fix:** Provide an empty object default on the parent: `logging = optional(object({ enabled = optional(bool, true) }), {})`.
- **Scenario 4: Storing an `ephemeral = true` variable in a normal resource attribute or root output**
  - **Trap:** Passing `var.ephemeral_password` to a regular `password` attribute (instead of write-only `password_wo`) or exposing it in a root module `output` with `ephemeral = true`.
  - **Reality:** Regular resource attributes and root module outputs are persisted in `terraform.tfstate`. Terraform blocks this at `validate`/`plan` time because ephemeral values cannot touch state. Use write-only attributes (`password_wo` + `password_wo_version`) and restrict `ephemeral = true` outputs to child modules only.
- **Scenario 5: Dynamic `for_each` keys derived from a resource `.id` or `.arn`**
  - **Trap:** Using `for_each = toset(aws_instance.web[*].id)` on an `aws_eip` resource.
  - **Reality:** Instance IDs are unknown until `apply`. Keys in `for_each` must be known at `plan` time.
  - **Fix:** Key `aws_instance.web` with static names (`for_each = var.instance_names`), and chain `for_each = aws_instance.web` on `aws_eip`, accessing `each.value.id` inside the resource body.
