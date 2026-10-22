# Terraform Infrastructure Engineer Professional (002) — Practice Exams

This directory contains four comprehensive, scenario-based practice examinations engineered for the upcoming **HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)** examination launching at IBM TechXchange 2026.

---

## 1. Practice Exam Directory & Links

| Exam | Focus & Scope | Questions | Time Limit | Target Score | Link |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Practice Exam 1** | Configuration Design, Backend Migration, State Locking, Lifecycle Hooks, Sentinel Policies | 30 Questions | 90 mins | $\ge 80\%$ | [**Open `Practice_Exam_1.md`**](Practice_Exam_1.md) |
| **Practice Exam 2** | Dynamic Provider Credentials, Cross-Variable Validation, Terraform Test Mocking, Atlantis CI | 25 Questions | 90 mins | $\ge 80\%$ | [**Open `Practice_Exam_2.md`**](Practice_Exam_2.md) |
| **Practice Exam 3** | S3 Backend KMS/Object Lock, Declarative Refactoring, Ephemeral Values, HCP Terraform Agents | 25 Questions | 90 mins | $\ge 80\%$ | [**Open `Practice_Exam_3.md`**](Practice_Exam_3.md) |
| **Practice Exam 4** | OPA Rego Policies, Speculative Plans, Plugin Cache Tuning, TFLint AST Analysis, State Replace | 25 Questions | 90 mins | $\ge 80\%$ | [**Open `Practice_Exam_4.md`**](Practice_Exam_4.md) |

---

## 2. Domain Distribution Across All Exams

1. **Domain 1: Scalable Configuration Design (20%)** — Dynamic blocks, complex HCL collections (`for`, `flatten`, `coalesce`, `merge`), optional object attributes, cross-variable validation.
2. **Domain 2: State & Backend Operations (20%)** — State migration, remote locking backends, state inspection (`state rm`, `state mv`, `state replace-provider`), declarative `import {}`.
3. **Domain 3: Lifecycle Operations & Day-2 Management (15%)** — `create_before_destroy`, `prevent_destroy`, `replace_triggered_by`, `terraform_data`, `precondition`/`postcondition`.
4. **Domain 4: CI/CD & Automation Workflows (15%)** — Deterministic plan execution (`terraform apply tfplan`), OIDC workload identity, lock file pinning (`.terraform.lock.hcl`), Atlantis PR workflows.
5. **Domain 5: Security, Compliance & Governance (15%)** — State encryption at rest (KMS), HashiCorp Sentinel (`advisory`, `soft-mandatory`, `hard-mandatory`), OPA Rego policies, Checkov/tfsec, ephemeral values.
6. **Domain 6: Advanced Modules & Testing Frameworks (10%)** — Native `terraform test` (`.tftest.hcl`), mock providers (`mock_provider`), assertions, Terratest, `terraform-docs`.
7. **Domain 7: HCP Terraform & Terraform Enterprise Operations (5%)** — Workspaces, Variable Sets, Run Triggers, HCP Terraform Agents, Cost Estimation, Continuous Validation.
