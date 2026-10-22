# Domain 4: CI/CD and Automation Workflows — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 15% of the 002 Professional Exam

---

## 1. Authoritative References

- **[Dynamic Provider Credentials (OIDC)](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/dynamic-provider-credentials):** Architecture of OIDC federation between CI/CD platforms and AWS, Azure, GCP, and Vault.
- **[Variable Precedence & Overrides](https://developer.hashicorp.com/terraform/language/values/variables#variable-definition-precedence):** Complete evaluation order rules.
- **[Automating Terraform in GitHub Actions](https://developer.hashicorp.com/terraform/tutorials/automation/github-actions):** Workflow patterns for automated plan reviews and applies.

---

## 2. Staff SRE Principles for Infrastructure Pipelines

1. **Strict Plan-Apply Decoupling:** Never run `terraform apply` directly in a single step without saving and locking the plan. Always generate a plan artifact (`-out=tfplan`), archive it securely, and apply that exact artifact after human approval or policy validation passes.
2. **Zero Long-Lived Credentials:** All CI systems must authenticate via OpenID Connect (OIDC) identity federation with fine-grained claim restrictions (e.g. scoping AWS role assumption strictly to `repo:org/infra:ref:refs/heads/main`).
