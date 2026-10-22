# Domain 5: Terraform Security and Governance — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 12% of the 002 Professional Exam

---

## 1. Authoritative References

- **[Ephemeral Values in Terraform](https://developer.hashicorp.com/terraform/language/values/variables#ephemeral):** Eliminating credentials from persistent state.
- **[Sentinel Policy as Code](https://developer.hashicorp.com/sentinel/docs):** Language specification and framework integration with HCP Terraform.
- **[Open Policy Agent (OPA) for Terraform](https://www.openpolicyagent.org/docs/latest/terraform/):** Evaluating JSON plan files with Rego rules.
- **[HashiCorp Vault Terraform Provider](https://registry.terraform.io/providers/hashicorp/vault/latest/docs):** Dynamic secrets and token lifecycle management.

---

## 2. Staff SRE Security Architecture Checklist

- [ ] Ensure all input variables containing credentials declare `sensitive = true`.
- [ ] Upgrade configurations to leverage `ephemeral = true` for zero-state leakage of temporary credentials.
- [ ] Enforce automated Policy as Code checks (Hard Mandatory in production, Soft Mandatory with SecOps override in staging).
- [ ] Verify that S3/GCS state storage buckets have default encryption enabled, versioning enabled, and public access completely blocked.
- [ ] Audit all firewall and security group rules to prohibit unrestricted ingress (`0.0.0.0/0`) on administrative ports (22, 3389, 5432, 3306, 6443).
