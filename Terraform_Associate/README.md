# HashiCorp Certified: Terraform Associate (004) Master Curriculum

Welcome to the comprehensive, exam-grade curriculum for the **HashiCorp Certified: Terraform Associate (004)** certification.

This curriculum is mapped 1:1 to HashiCorp's official **Terraform Associate (004)** exam blueprint (**Terraform 1.12+**), incorporating all newly introduced 004 topics: **custom condition validations** (`precondition`, `postcondition`, `check` blocks), **ephemeral values & write-only arguments** (`ephemeral = true`, `_wo` attributes), **declarative state refactoring** (`moved`, `removed`, `import` blocks with `-generate-config-out`), **S3 native state locking** (`use_lockfile = true`), and **HCP Terraform Projects, Variable Sets, Dynamic Provider Credentials, and Run Triggers**.

---

## Official 004 Exam Blueprint & Domain Matrix

| Domain | Official 004 Exam Objective | Sub-Objectives | Core 004 Competencies & Deliverables |
| :--- | :--- | :--- | :--- |
| **[Domain 1](./Domain_1_IaC_with_Terraform/Study_Guide.md)** | **1. Infrastructure as Code (IaC) with Terraform** | `1a` – `1c` | What IaC is, Declarative vs. Imperative, Idempotence, Multi-cloud / Hybrid cloud / Service-agnostic workflows, Day 1 vs. Day 2 operations |
| **[Domain 2](./Domain_2_Terraform_Fundamentals/Study_Guide.md)** | **2. Terraform Fundamentals** | `2a` – `2d` | Provider plugin architecture (gRPC), `required_providers`, Version constraints (`~>`, `>=`), Dependency Lock File (`.terraform.lock.hcl`), Multi-platform checksums (`terraform providers lock`), Multiple provider instances (`alias`), State fundamentals |
| **[Domain 3](./Domain_3_Core_Terraform_Workflow/Study_Guide.md)** | **3. Core Terraform Workflow** | `3a` – `3g` | Core workflow (`Write` $\rightarrow$ `Plan` $\rightarrow$ `Apply`), `terraform init` vs. `terraform get`, `terraform validate`, `terraform plan` (`-out`, `-refresh-only`), `terraform apply` (`-auto-approve`, `-replace`), `terraform destroy`, `terraform fmt` (`-check`, `-recursive`), `terraform console`, `terraform graph`, `terraform show` |
| **[Domain 4](./Domain_4_Terraform_Configuration/Study_Guide.md)** | **4. Terraform Configuration** | `4a` – `4h` | `resource` vs. `data` blocks, Cross-resource references, Variables & Precedence, Outputs, Complex types (`list`, `map`, `set`, `object`, `tuple`), High-yield Built-in Functions (`lookup`, `element`, `merge`, `flatten`, `try`, `can`, `templatefile`), `count` vs. `for_each`, `dynamic` blocks, `depends_on` & `lifecycle` (`create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`), **Custom Conditions (`validation`, `precondition`, `postcondition`, `check`)**, **Ephemeral Values (`ephemeral` block, `ephemeral = true`) & Write-Only Arguments (`_wo`)**, Provisioners (`local-exec`, `remote-exec`) & `terraform_data` |
| **[Domain 5](./Domain_5_Terraform_Modules/Study_Guide.md)** | **5. Terraform Modules** | `5a` – `5d` | Root vs. Child modules, Module sources (Local, Public/Private Registry, Git `?ref=`, S3), Variable scope & encapsulation, Calling child modules & accessing outputs, Module version constraints & registry naming standards |
| **[Domain 6](./Domain_6_Terraform_State_Management/Study_Guide.md)** | **6. Terraform State Management** | `6a` – `6d` | Local backend (`terraform.tfstate`), State locking (`use_lockfile = true`, `force-unlock`), Remote backends (`s3` vs DynamoDB, `azurerm`, `gcs`, `cloud` block), `terraform_remote_state` data source, CLI Workspaces (`list`, `show`, `select`, `new`, `delete`), **Managing Drift & Declarative State Refactoring (`-refresh-only`, `moved` blocks, `removed` blocks)** |
| **[Domain 7](./Domain_7_Maintain_Infrastructure/Study_Guide.md)** | **7. Maintain Infrastructure with Terraform** | `7a` – `7c` | **Importing existing infrastructure (`import` block with `-generate-config-out` & `terraform import` CLI)**, Inspecting & manipulating state (`terraform state list`, `show`, `mv`, `rm`, `pull`), Verbose logging (`TF_LOG`, `TF_LOG_CORE`, `TF_LOG_PROVIDER`, `TF_LOG_PATH`) |
| **[Domain 8](./Domain_8_HCP_Terraform/Study_Guide.md)** | **8. HCP Terraform** | `8a` – `8d` | Creating infrastructure with HCP Terraform (`cloud` block, `terraform login`), Collaboration & Governance (**Teams, Sentinel & OPA Policy-as-Code, Cost Estimation, Private Registry, Drift Detection, Health Assessments, Dynamic Provider Credentials**), **Organizing Workspaces & Projects (Projects, Variable Sets, Run Triggers)**, Workflow integrations (VCS-driven, CLI-driven, API-driven, Run Tasks, Agents), HCP Terraform vs. Terraform Enterprise |
| **[Practice Exams](./Practice_Exams/README.md)** | **Comprehensive Practice Exams Suite** | **Suite** | **4 Full-Scale Mock Exams (132 Practice Questions Total)** testing Terraform 1.12+ features, scenario code snippets, and complete distractor analysis |

---

## Quick Navigation & Core References

- **[Terraform Associate (004) Master Cheat Sheet](./Terraform_Associate_004_Master_Cheat_Sheet.md):** Complete 004 CLI Reference, Variable Precedence, Custom Conditions (`precondition`/`postcondition`/`check`), Ephemeral & Write-Only (`_wo`) Secrets, Declarative `moved`/`removed`/`import` blocks, Built-in Functions, Provisioners, and HCP Terraform Projects/Workspaces.
- **[Practice Exams Suite](./Practice_Exams/README.md):**
  - [Practice Exam 1: Core Workflow, HCL Configuration, Custom Conditions & Ephemeral Secrets (25 Questions)](./Practice_Exams/Practice_Exam_1.md)
  - [Practice Exam 2: State Management, `moved`/`removed`/`import` Blocks, Modules & CLI Operations (25 Questions)](./Practice_Exams/Practice_Exam_2.md)
  - [Practice Exam 3: HCP Terraform Projects, Variable Sets, Sentinel/OPA, Drift & Edge Cases (25 Questions)](./Practice_Exams/Practice_Exam_3.md)
  - [Practice Exam 4: Full-Length Final Simulation (57 Questions — 60-Minute Timed Mock)](./Practice_Exams/Practice_Exam_4_Full_Length_Simulation.md)
