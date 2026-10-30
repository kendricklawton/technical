# Terraform Infrastructure Engineer Professional (002) — Practice Exams Suite

This directory contains **4 full-length, Staff-level scenario practice exams** engineered to mirror the complexity, multi-constraint trade-offs, and HCL code analysis required to pass the **HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)** exam.

## Practice Exam Index

* **[Practice Exam 1: Enterprise Multi-Account Architecture & State Refactoring](./Practice_Exam_1.md)**
  * **Primary Focus:** Complex structural types (`optional()`), `for_each` unknown-key resolution, S3 native state locking (`use_lockfile = true`), cross-stack decoupling, and declarative refactoring (`moved`, `removed`, `import`).
  * **Passing Target:** 80%+

* **[Practice Exam 2: CI/CD Automation, Ephemeral Secrets & Policy Governance](./Practice_Exam_2.md)**
  * **Primary Focus:** Saved binary plan pipelines, `-detailed-exitcode` drift workflows, OIDC dynamic provider credentials, `.terraform.lock.hcl` multi-OS hashes, Ephemeral resources (`ephemeral` blocks) & Write-Only attributes (`_wo`), Sentinel (`tfplan/v2`), and OPA Rego.
  * **Passing Target:** 80%+

* **[Practice Exam 3: Native Testing (`.tftest.hcl`), Advanced Modules & Lifecycle Mastery](./Practice_Exam_3.md)**
  * **Primary Focus:** `terraform test` with `mock_provider`, `override_resource`, and `expect_failures`; multi-region provider aliasing (`configuration_aliases`); `create_before_destroy` DAG propagation; `terraform_data` + `replace_triggered_by`; and continuous validation (`check` blocks).
  * **Passing Target:** 80%+

* **[Practice Exam 4: HCP Terraform Stacks, TFE Architecture & Production Incident Recovery](./Practice_Exam_4.md)**
  * **Primary Focus:** Terraform Stacks (`*.tfstack.hcl` & `*.tfdeploy.hcl`), HCP Terraform Variable Set precedence, Private Registry No-Code provisioning, TFE Flexible Deployment Options (FDO) Active-Active architecture, and corrupted state/lock recovery.
  * **Passing Target:** 80%+

## Exam-Day Strategy
1. **Eliminate State-Mutating Anti-Patterns First:** On any question involving refactoring or production drift, immediately eliminate answers that use deprecated `terraform taint`, raw `terraform refresh`, or destructive resource renaming without `moved` blocks.
2. **Distinguish `sensitive`, `ephemeral`, and Write-Only (`_wo`):** Remember that `sensitive = true` only redacts CLI output (values remain in plaintext in `.tfstate`). Only `ephemeral = true` variables/resources paired with Write-Only attributes (`password_wo`) guarantee zero state persistence.
3. **Check `plan` vs. `apply` Phase Constraints:** Keys in `for_each` and `import.id` expressions must be known during the `plan` phase (`Error: Invalid for_each argument` occurs when keys depend on computed attributes like `.id` or `.arn` of unapplied resources).
