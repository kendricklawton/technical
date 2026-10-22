# Domain 6: Advanced Module Usage and Testing — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 10% of the 002 Professional Exam

---

## 1. Authoritative References

- **[Terraform Test Framework Documentation](https://developer.hashicorp.com/terraform/language/tests):** `run` blocks, assertions, mock providers, and `expect_failures`.
- **[Standard Module Structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure):** Recommended file layouts, examples, and submodules.
- **[Private Module Registry](https://developer.hashicorp.com/terraform/cloud-docs/registry):** Publishing, versioning, and consuming internal enterprise modules.

---

## 2. Staff SRE Module Principles

1. **Test-Driven Infrastructure Development:** Every public or shared private module must ship with a suite of `*.tftest.hcl` files covering both positive validation (valid plans) and negative validation (`expect_failures` on malformed inputs).
2. **Immutable Semantic Releases:** Never point production root configurations to `source = "git::https://...#main"`. Always pin to immutable semver tags or registry versions (`version = "3.2.0"`).
