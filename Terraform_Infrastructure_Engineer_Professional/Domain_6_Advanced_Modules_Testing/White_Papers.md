# Domain 6: Advanced Modules & Native Testing Frameworks — Reference White Papers & Architectural Deep Dives

## 1. Shift-Left Module Verification with `terraform test` & Provider Mocking
* **Primary Reference:** *HashiCorp Engineering Guide — Contract Testing & Provider Mocking in Terraform 1.7+*
* **Core Architectural Takeaways:**
  * Traditional integration testing (`Terratest`) requires provisioning real cloud infrastructure (taking 15–45 minutes for EKS/RDS and incurring cloud spend).
  * Combining `mock_provider` (which automatically synthesizes valid types for all computed schema attributes) with `override_resource`, `override_data`, and `override_module` enables sub-second contract testing of complex HCL graphs in pre-merge CI checks.

---

## 2. Composable Module Design & Refactoring Preservation
* **Primary Reference:** *Terraform Module Authoring Best Practices — Dependency Inversion & Non-Breaking Upgrades*
* **Core Architectural Takeaways:**
  * Never embed `provider` configuration blocks inside child modules.
  * When refactoring internal resource addresses inside a shared module, ship `moved` blocks inside the module release so downstream consumers inherit the state migration automatically upon bumping the module version constraint.
