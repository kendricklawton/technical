# Domain 1: Scalable Configuration Design — Reference White Papers & Architectural Deep Dives

## 1. HashiCorp Type System & Structural Object Conversion Specification
* **Primary Reference:** *Terraform Language Documentation — Type Constraints & Structural Conversion Mechanics*
* **Core Architectural Takeaways:**
  * **Homogeneous Collections (`list`, `map`, `set`) vs. Heterogeneous Structures (`object`, `tuple`):** Collection types enforce a single element type across arbitrary lengths, whereas structural types enforce a fixed schema of named attributes or positional elements with independent types.
  * **Silent Attribute Stripping on `object({...})`:** During input variable evaluation, when a caller supplies a map or object containing keys not declared in the target `object({...})` type constraint, Terraform's type converter silently drops the extra keys to preserve strict module encapsulation.
  * **Recursive `optional(T, default)` Evaluation:** Nested `optional()` attributes inside an outer `optional(object({...}))` only populate their inner defaults if the outer object itself is non-null (either passed by the caller or initialized via an outer `, {}` default).
  * **The `any` Pseudo-Type Pitfall:** `any` is a type-inference placeholder, not a union or dynamic variant type. In `list(any)` or `map(any)`, Terraform still requires all elements to unify to a single concrete type.

---

## 2. Deterministic Graph Addressing: `for_each` vs. `count` & Unknown Key Resolution
* **Primary Reference:** *HashiCorp Architecture Guide — Resource Iteration, Graph Addressing & Plan-Time Determinism*
* **Core Architectural Takeaways:**
  * **DAG Node Identity:** Terraform builds its Directed Acyclic Graph (DAG) during the `plan` phase before invoking any resource create/update RPCs. Consequently, every node address (`resource_type.name["key"]`) must be known during `plan`.
  * **Decoupling Keys from Computed Attributes:** Passing a computed attribute (such as an uncreated EC2 instance ID or VPC ARN) as a `for_each` key fails with `The "for_each" map includes keys derived from resource attributes that cannot be determined until apply`. Always key the upstream resource map by static plan-time strings (e.g., AZ name, logical service name) and chain `for_each = aws_instance.upstream` on downstream resources.

---

## 3. Zero-State Secret Architecture: Ephemeral Values & Write-Only Arguments (Terraform 1.10 / 1.11+)
* **Primary Reference:** *HashiCorp Security Whitepaper — Eliminating Plaintext Secrets from Terraform State with Ephemeral Blocks and Write-Only Attributes*
* **Core Architectural Takeaways:**
  * **Lifecycle of an `ephemeral` Block:** Unlike `data` sources (which execute `ReadDataSource` during plan/refresh and store all returned attributes in `.tfstate`), `ephemeral` resources execute `OpenEphemeralResource` at runtime, hold credentials exclusively in process memory, and invoke `CloseEphemeralResource` (optional lease revocation/cleanup) when the Terraform run completes.
  * **Write-Only (`_wo`) Attribute Versioning:** Because Terraform intentionally stores `null` for write-only attributes in `.tfstate`, it cannot diff the live cloud password against state. Instead, providers pair each `<attr>_wo` argument with a non-sensitive integer `<attr>_wo_version` stored in state; incrementing `<attr>_wo_version` signals the provider to re-send the write-only payload during `apply`.
