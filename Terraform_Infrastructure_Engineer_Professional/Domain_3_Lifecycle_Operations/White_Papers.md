# Domain 3: Infrastructure Lifecycle and Operations — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 15% of the 002 Professional Exam

---

## 1. Authoritative References

- **[Resource Lifecycle Meta-Arguments](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle):** `create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`.
- **[Importing Existing Infrastructure](https://developer.hashicorp.com/terraform/language/import):** Declarative `import` blocks and code generation.
- **[Removing Resources from State](https://developer.hashicorp.com/terraform/language/resources/syntax#removed):** Declarative `removed` block syntax.
- **[Dependency Lock File (`.terraform.lock.hcl`)](https://developer.hashicorp.com/terraform/language/files/dependency-lock):** Provider checksum caching and multi-platform cross-verification (`terraform providers lock`).

---

## 2. Staff SRE Multi-Environment Architecture

### Workspaces vs Directories (The Staff SRE Decision Framework)
- **Use CLI Workspaces ONLY when:** Environments share 100% identical configuration and state backend settings, differing only by small variable inputs (e.g., ephemeral preview PR environments).
- **Use Isolated Directory Trees (or HCP Terraform Projects) when:** Environments have divergent lifecycles, different cloud accounts/credentials, different IAM boundaries, or compliance separation (Production vs Development).
