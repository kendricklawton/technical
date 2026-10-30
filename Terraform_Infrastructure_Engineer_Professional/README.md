# HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)

> [!IMPORTANT]
> **Exam Format & Target Profile:** The **Terraform Infrastructure Engineer Professional (002)** (formerly Authoring & Operations Professional) certification validates Staff/Principal-level mastery of enterprise Infrastructure-as-Code architecture, state lifecycle engineering, CI/CD automation, Policy-as-Code (Sentinel & OPA), native `.tftest.hcl` mock testing, Ephemeral/Write-Only secret workflows (Terraform 1.10/1.11+), Terraform Stacks (`*.tfstack.hcl`), and HCP Terraform / Terraform Enterprise (FDO) administration.

---

## Exam Blueprint & Domain Weighting

* **Domain 1: Scalable Configuration Design (20%)**
  * **Core Focus:** Structural vs. collection types, nested `optional(T, default)`, `nullable = false`, `ephemeral = true` variables/outputs, `for_each` vs. `count` index-shift prevention, `flatten()` + `setproduct()`, `dynamic` blocks, Provider-Defined Functions (`provider::aws::arn_parse`), and `configuration_aliases`.
  * **Study Materials:**
    * [Domain 1 Study Guide](./Domain_1_Scalable_Configuration_Design/Study_Guide.md)
    * [Domain 1 Hands-On Exercises](./Domain_1_Scalable_Configuration_Design/Exercises.md)
    * [Domain 1 Reference White Papers](./Domain_1_Scalable_Configuration_Design/White_Papers.md)
    * [Quiz 1](./Domain_1_Scalable_Configuration_Design/Quiz_1.md) | [Quiz 2](./Domain_1_Scalable_Configuration_Design/Quiz_2.md) | [Quiz 3](./Domain_1_Scalable_Configuration_Design/Quiz_3.md)

* **Domain 2: State & Backend Operations (18%)**
  * **Core Focus:** Remote backend architectures, S3 native state locking (`use_lockfile = true` via `.tflock` conditional writes) vs. DynamoDB locking, `-migrate-state` vs. `-reconfigure`, partial backend configs, blast-radius segmentation, `terraform_remote_state` vs. `tfe_outputs` vs. data source decoupling, `plan -refresh-only`, and state disaster recovery (`force-unlock`, `state pull/push`, lineage & serial invariants).
  * **Study Materials:**
    * [Domain 2 Study Guide](./Domain_2_State_Backend_Operations/Study_Guide.md)
    * [Domain 2 Hands-On Exercises](./Domain_2_State_Backend_Operations/Exercises.md)
    * [Domain 2 Reference White Papers](./Domain_2_State_Backend_Operations/White_Papers.md)
    * [Quiz 1](./Domain_2_State_Backend_Operations/Quiz_1.md) | [Quiz 2](./Domain_2_State_Backend_Operations/Quiz_2.md) | [Quiz 3](./Domain_2_State_Backend_Operations/Quiz_3.md)

* **Domain 3: Lifecycle Operations & Declarative Refactoring (16%)**
  * **Core Focus:** Resource `lifecycle` meta-arguments (`create_before_destroy` DAG propagation, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`), `terraform_data` replacement triggers, declarative state refactoring (`moved`, `removed`, `import` blocks with `for_each` and `-generate-config-out`), `-replace` vs. deprecated `taint`, and continuous validation (`check` blocks vs. `precondition`/`postcondition`).
  * **Study Materials:**
    * [Domain 3 Study Guide](./Domain_3_Lifecycle_Operations/Study_Guide.md)
    * [Domain 3 Hands-On Exercises](./Domain_3_Lifecycle_Operations/Exercises.md)
    * [Domain 3 Reference White Papers](./Domain_3_Lifecycle_Operations/White_Papers.md)
    * [Quiz 1](./Domain_3_Lifecycle_Operations/Quiz_1.md) | [Quiz 2](./Domain_3_Lifecycle_Operations/Quiz_2.md) | [Quiz 3](./Domain_3_Lifecycle_Operations/Quiz_3.md)

* **Domain 4: CI/CD & Automation Workflows (14%)**
  * **Core Focus:** Speculative vs. Saved Binary Plans (`-out=tfplan`), `-detailed-exitcode` (`0`/`1`/`2`) drift pipelines, OIDC workload identity federation (AWS/GCP/Azure/Vault dynamic credentials), `.terraform.lock.hcl` multi-platform checksums (`providers lock`), `TF_PLUGIN_CACHE_DIR` concurrency & lockfile interactions, and air-gapped network/filesystem mirrors (`provider_installation`).
  * **Study Materials:**
    * [Domain 4 Study Guide](./Domain_4_CICD_Automation_Workflows/Study_Guide.md)
    * [Domain 4 Hands-On Exercises](./Domain_4_CICD_Automation_Workflows/Exercises.md)
    * [Domain 4 Reference White Papers](./Domain_4_CICD_Automation_Workflows/White_Papers.md)
    * [Quiz 1](./Domain_4_CICD_Automation_Workflows/Quiz_1.md) | [Quiz 2](./Domain_4_CICD_Automation_Workflows/Quiz_2.md) | [Quiz 3](./Domain_4_CICD_Automation_Workflows/Quiz_3.md)

* **Domain 5: Security, Governance & Policy-as-Code (12%)**
  * **Core Focus:** Ephemeral resources (`ephemeral` block in TF 1.10+) and Write-Only attributes (`_wo` and `_wo_version` in TF 1.11+) for zero-state secret persistence, HashiCorp Vault dynamic secret lease management, Sentinel Policy-as-Code (`tfplan/v2`, `tfconfig/v2`, `tfstate/v2`, `tfrun` imports & enforcement levels), Open Policy Agent (OPA Rego), and shift-left static analysis (`trivy`, `checkov`, `tflint`).
  * **Study Materials:**
    * [Domain 5 Study Guide](./Domain_5_Security_Governance/Study_Guide.md)
    * [Domain 5 Hands-On Exercises](./Domain_5_Security_Governance/Exercises.md)
    * [Domain 5 Reference White Papers](./Domain_5_Security_Governance/White_Papers.md)
    * [Quiz 1](./Domain_5_Security_Governance/Quiz_1.md) | [Quiz 2](./Domain_5_Security_Governance/Quiz_2.md) | [Quiz 3](./Domain_5_Security_Governance/Quiz_3.md)

* **Domain 6: Advanced Modules & Native Testing Frameworks (10%)**
  * **Core Focus:** Composable module hierarchy, Dependency Inversion, semantic version constraints (`~> 2.1.0` vs. `~> 2.1`), Private Registry publishing, and the native `terraform test` framework (`.tftest.hcl`, sequential/isolated `run` blocks, `command = plan | apply`, `expect_failures`, `mock_provider`, and `override_resource`/`override_data`/`override_module`).
  * **Study Materials:**
    * [Domain 6 Study Guide](./Domain_6_Advanced_Modules_Testing/Study_Guide.md)
    * [Domain 6 Hands-On Exercises](./Domain_6_Advanced_Modules_Testing/Exercises.md)
    * [Domain 6 Reference White Papers](./Domain_6_Advanced_Modules_Testing/White_Papers.md)
    * [Quiz 1](./Domain_6_Advanced_Modules_Testing/Quiz_1.md) | [Quiz 2](./Domain_6_Advanced_Modules_Testing/Quiz_2.md) | [Quiz 3](./Domain_6_Advanced_Modules_Testing/Quiz_3.md)

* **Domain 7: HCP Terraform & Terraform Enterprise Administration (10%)**
  * **Core Focus:** Organizations, Projects, Workspaces, Variable Sets & precedence hierarchy, Execution Modes (Remote, Local, Agent Pools), Run Triggers vs. `tfe_outputs`, Terraform Stacks (`*.tfstack.hcl` & `*.tfdeploy.hcl`), Continuous Drift & Health Assessments, and Terraform Enterprise Flexible Deployment Options (FDO) with External Services & Active-Active HA.
  * **Study Materials:**
    * [Domain 7 Study Guide](./Domain_7_HCP_Terraform_Enterprise/Study_Guide.md)
    * [Domain 7 Hands-On Exercises](./Domain_7_HCP_Terraform_Enterprise/Exercises.md)
    * [Domain 7 Reference White Papers](./Domain_7_HCP_Terraform_Enterprise/White_Papers.md)
    * [Quiz 1](./Domain_7_HCP_Terraform_Enterprise/Quiz_1.md) | [Quiz 2](./Domain_7_HCP_Terraform_Enterprise/Quiz_2.md) | [Quiz 3](./Domain_7_HCP_Terraform_Enterprise/Quiz_3.md)

---

## Rapid Reference & Full-Length Mock Exams

* **[Terraform Master Cheat Sheet](./Terraform_Master_Cheat_Sheet.md)** — Complete CLI, HCL function, state surgery, `.tftest.hcl`, Sentinel/OPA, and architecture decision matrix.
* **[Full-Length Practice Exams Suite](./Practice_Exams/README.md)**
  * [Practice Exam 1: Enterprise Multi-Account Architecture & State Refactoring](./Practice_Exams/Practice_Exam_1.md)
  * [Practice Exam 2: CI/CD Pipelines, Ephemeral Secrets & Policy-as-Code](./Practice_Exams/Practice_Exam_2.md)
  * [Practice Exam 3: Native Testing (`mock_provider`), Modules & Lifecycle Operations](./Practice_Exams/Practice_Exam_3.md)
  * [Practice Exam 4: HCP Terraform Stacks, TFE Active-Active & Disaster Recovery](./Practice_Exams/Practice_Exam_4.md)
