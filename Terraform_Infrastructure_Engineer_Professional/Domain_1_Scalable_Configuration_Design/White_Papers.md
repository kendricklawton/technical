# Domain 1: Scalable Configuration Design — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 20% of the 002 Professional Exam

---

## 1. Core Documentation References

- **[HashiCorp Terraform Language Specifications](https://developer.hashicorp.com/terraform/language):** Complete reference on expressions, types, and meta-arguments.
- **[Terraform Functions Directory](https://developer.hashicorp.com/terraform/language/functions):** Type conversion, string interpolation, and collection manipulation functions.
- **[Custom Conditions & Validations](https://developer.hashicorp.com/terraform/language/expressions/custom-conditions):** Variable validation, resource preconditions, and postconditions.
- **[Model Context Protocol (MCP) in Terraform](https://developer.hashicorp.com/terraform/docs/ai-mcp):** Connecting AI coding agents securely to local Terraform engine contexts.

---

## 2. Staff SRE Architecture Patterns

1. **Deterministic HCL Keys:** Never use numerical array indexes (`count.index`) for stateful resources like virtual machines, disks, or databases. If an element in the middle of a list is removed, `count` will force the destruction and recreation of every subsequent resource. Always use `for_each` keyed by stable, immutable identifiers.
2. **Strict Provider Pinning:** Always pin both provider source (`hashicorp/aws`, `digitalocean/digitalocean`) and semantic version constraints (`~> 5.0`).
3. **Fail-Fast Schema Constraints:** Use `validation` blocks with clear, actionable `error_message` strings so developers see immediate configuration feedback before code merges.
