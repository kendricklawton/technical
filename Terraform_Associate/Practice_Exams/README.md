# HashiCorp Certified: Terraform Associate (004) — Practice Exams Suite

Welcome to the full-length scenario practice exam suite for the **HashiCorp Certified: Terraform Associate (004)** certification (**Terraform 1.12+**).

---

## Official 004 Exam Blueprint & Objectives Matrix

- **Tested Version:** Terraform 1.12+
- **Official Exam Format:** ~57 questions (Multiple-choice, Multiple-answer, True/False) in **60 minutes**.
- **Passing Target:** Aim for **$\ge 80\%$** on Domain Quizzes and Practice Exams 1–3, and **$\ge 75\%$ (43 / 57)** on the Full-Length Simulation (Practice Exam 4).

| Objective | Official 004 Exam Domain | Key Sub-Objectives |
| :--- | :--- | :--- |
| **Objective 1** | **Infrastructure as Code (IaC) with Terraform** | `1a` What IaC is · `1b` Advantages of IaC · `1c` Multi-cloud, hybrid & service-agnostic workflows |
| **Objective 2** | **Terraform Fundamentals** | `2a` Install/version providers (`.terraform.lock.hcl`, `lock -platform`) · `2b` Plugin architecture (gRPC) · `2c` Multiple providers (`alias`) · `2d` Purpose of state |
| **Objective 3** | **Core Terraform Workflow** | `3a` Write-Plan-Apply · `3b` `init` vs `get` · `3c` `validate` · `3d` `plan` · `3e` `apply` (`-replace`) · `3f` `destroy` · `3g` `fmt` · `console` & `graph` |
| **Objective 4** | **Terraform Configuration** *(Heavy 004 Focus!)* | `4a` `resource` vs `data` · `4b` Cross-references · `4c` Variables & outputs · `4d` Complex types · `4e` Built-in functions, `for_each`/`count`, `dynamic` · **`4f` `depends_on` & `create_before_destroy`** · **`4g` Custom conditions (`validation`, `precondition`, `postcondition`, `check`)** · **`4h` Sensitive data, Vault, `ephemeral` values & write-only (`_wo`) arguments** · Provisioners & `terraform_data` |
| **Objective 5** | **Terraform Modules** | `5a` Module sources · `5b` Variable scope/encapsulation · `5c` Calling modules · `5d` Module versioning & registry conventions |
| **Objective 6** | **Terraform State Management** *(Heavy 004 Focus!)* | `6a` Local backend & CLI workspaces (`list`, `select`, `new`, `delete`) · `6b` State locking (`use_lockfile`, `force-unlock`) · `6c` Remote `backend` & `terraform_remote_state` · **`6d` Drift (`-refresh-only`) & declarative state refactoring (`moved`, `removed` blocks)** |
| **Objective 7** | **Maintain Infrastructure with Terraform** | **`7a` Importing infrastructure (`import` block + `-generate-config-out` & `terraform import`)** · `7b` `terraform state` CLI (`list`, `show`, `mv`, `rm`, `pull`) · `7c` Verbose logging (`TF_LOG`, `TF_LOG_CORE`, `TF_LOG_PROVIDER`, `TF_LOG_PATH`) |
| **Objective 8** | **HCP Terraform** *(Heavy 004 Focus!)* | `8a` Creating infrastructure (`cloud` block) · `8b` Governance (Sentinel/OPA, Dynamic Credentials, Cost Estimation, Private Registry, Health Assessments) · **`8c` Organizing Workspaces & Projects, Variable Sets, Run Triggers** · `8d` VCS/CLI/API workflows & Agents · HCP Terraform vs. Terraform Enterprise |

---

## Practice Exam Catalog

1. **[Practice Exam 1: Core Workflow, HCL Configuration, Custom Conditions & Ephemeral Secrets](./Practice_Exam_1.md)**
   - 25 scenario questions focusing heavily on Objectives 1–4, including `precondition`/`postcondition` vs. `check` blocks, `ephemeral` resources, `password_wo` / `password_wo_version`, variable precedence, and `create_before_destroy`.
2. **[Practice Exam 2: State Management, `moved`/`removed`/`import` Blocks, Modules & CLI Operations](./Practice_Exam_2.md)**
   - 25 scenario questions focusing heavily on Objectives 5–7, including declarative `moved`, `removed`, and `import` blocks (`-generate-config-out`), S3 native locking (`use_lockfile = true`), `-refresh-only`, module encapsulation, and `TF_LOG_PROVIDER`.
3. **[Practice Exam 3: HCP Terraform Projects, Variable Sets, Sentinel/OPA, Drift & Comprehensive 004 Review](./Practice_Exam_3.md)**
   - 25 scenario questions focusing on Objective 8 and cross-domain 004 synthesis, including HCP Terraform Projects, Variable Set precedence, Dynamic Provider Credentials (OIDC), Health Assessments, Sentinel/OPA enforcement levels, and complex HCL expressions.
4. **[Practice Exam 4: Full-Length Final Simulation (57 Questions — 60-Minute Timed Mock)](./Practice_Exam_4_Full_Length_Simulation.md)**
   - **57 full-length scenario questions** mirroring the exact question volume, 60-minute duration, and domain weight distribution of the real HashiCorp Certified: Terraform Associate (004) exam, complete with comprehensive answer keys and distractor analyses.
