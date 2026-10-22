# HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)

[![Exam Version](https://img.shields.io/badge/Terraform_Engine-v1.15+-844FBA.svg?logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Certification](https://img.shields.io/badge/Certification-Terraform_Engineer_Professional_002-623CE4.svg)](https://developer.hashicorp.com/certifications/infrastructure-automation)
[![Launch](https://img.shields.io/badge/Event-IBM_TechXchange_2026_Atlanta-052FAD.svg?logo=ibm&logoColor=white)](https://www.ibm.com/community/techxchange/)

Comprehensive, production-grade study curriculum and technical preparation guide for the **HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)** examination launching at **IBM TechXchange 2026 in Atlanta**.

This certification validates senior and staff-level expertise in architecting, authoring, testing, automating, and securing production-grade multi-environment infrastructure with Terraform 1.15+, HCP Terraform, OIDC dynamic provider credentials, custom testing frameworks, and Policy as Code.

---

## 1. Exam Blueprint & Domain Weightings

| Domain | Focus Area | Key Exam Competencies | Direct Links |
| :--- | :--- | :--- | :--- |
| **Domain 1: Scalable Configuration Design** | **20%** | Terraform 1.15 constructs, dynamic `for_each` graphs, functions, type constraints, providers, Terraform Model Context Protocol (MCP) server for AI coding agents | [Study Guide](Domain_1_Scalable_Configuration_Design/Study_Guide.md) \| [White Papers](Domain_1_Scalable_Configuration_Design/White_Papers.md) \| [Exercises](Domain_1_Scalable_Configuration_Design/Exercises.md) \| [Quiz 1](Domain_1_Scalable_Configuration_Design/Quiz_1.md) \| [Quiz 2](Domain_1_Scalable_Configuration_Design/Quiz_2.md) |
| **Domain 2: State & Backend Operations** | **18%** | Remote state architectures, state locking, migration, drift reconciliation, state inspection, DAG dependency chains, version upgrades | [Study Guide](Domain_2_State_Backend_Operations/Study_Guide.md) \| [White Papers](Domain_2_State_Backend_Operations/White_Papers.md) \| [Exercises](Domain_2_State_Backend_Operations/Exercises.md) \| [Quiz 1](Domain_2_State_Backend_Operations/Quiz_1.md) \| [Quiz 2](Domain_2_State_Backend_Operations/Quiz_2.md) |
| **Domain 3: Infrastructure Lifecycle & Operations** | **15%** | Multi-environment staging, declarative `import` blocks, `removed` blocks, `lifecycle` meta-arguments (`prevent_destroy`, `replace_triggered_by`), troubleshooting plans | [Study Guide](Domain_3_Lifecycle_Operations/Study_Guide.md) \| [White Papers](Domain_3_Lifecycle_Operations/White_Papers.md) \| [Exercises](Domain_3_Lifecycle_Operations/Exercises.md) \| [Quiz 1](Domain_3_Lifecycle_Operations/Quiz_1.md) \| [Quiz 2](Domain_3_Lifecycle_Operations/Quiz_2.md) |
| **Domain 4: CI/CD & Automation Workflows** | **15%** | Automated speculative plans, OIDC dynamic provider credentials (AWS, GCP, Vault), `.auto.tfvars` precedence, pipeline concurrency, VCS triggers | [Study Guide](Domain_4_CICD_Automation_Workflows/Study_Guide.md) \| [White Papers](Domain_4_CICD_Automation_Workflows/White_Papers.md) \| [Exercises](Domain_4_CICD_Automation_Workflows/Exercises.md) \| [Quiz 1](Domain_4_CICD_Automation_Workflows/Quiz_1.md) \| [Quiz 2](Domain_4_CICD_Automation_Workflows/Quiz_2.md) |
| **Domain 5: Security & Governance** | **12%** | Ephemeral values, write-only arguments (Terraform 1.11+), secret rotation with Vault, Sentinel & OPA/Rego policies, blast radius mitigation | [Study Guide](Domain_5_Security_Governance/Study_Guide.md) \| [White Papers](Domain_5_Security_Governance/White_Papers.md) \| [Exercises](Domain_5_Security_Governance/Exercises.md) \| [Quiz 1](Domain_5_Security_Governance/Quiz_1.md) \| [Quiz 2](Domain_5_Security_Governance/Quiz_2.md) |
| **Domain 6: Advanced Modules & Testing** | **10%** | Monolith refactoring, module semantic contracts, native `terraform test` framework, mock providers, assertions, private registry publishing | [Study Guide](Domain_6_Advanced_Modules_Testing/Study_Guide.md) \| [White Papers](Domain_6_Advanced_Modules_Testing/White_Papers.md) \| [Exercises](Domain_6_Advanced_Modules_Testing/Exercises.md) \| [Quiz 1](Domain_6_Advanced_Modules_Testing/Quiz_1.md) \| [Quiz 2](Domain_6_Advanced_Modules_Testing/Quiz_2.md) |
| **Domain 7: HCP Terraform / Enterprise Operations** | **10%** | Workspaces, Projects, Run Tasks, Cost Estimation, Team RBAC, Private Registry, Agent Pools, audit logging | [Study Guide](Domain_7_HCP_Terraform_Enterprise/Study_Guide.md) \| [White Papers](Domain_7_HCP_Terraform_Enterprise/White_Papers.md) \| [Exercises](Domain_7_HCP_Terraform_Enterprise/Exercises.md) \| [Quiz 1](Domain_7_HCP_Terraform_Enterprise/Quiz_1.md) \| [Quiz 2](Domain_7_HCP_Terraform_Enterprise/Quiz_2.md) |

---

## 2. Exam Format & Key Details

- **Exam Code:** 002 (Terraform Infrastructure Engineer Professional)
- **Target Audience:** Production Infrastructure Engineers, Platform Engineers, Staff Cloud Engineers, and Site Reliability Engineers.
- **Delivery Mode:** Proctored online or in-person testing centers (Debuting at IBM TechXchange Atlanta 2026).
- **Exam Duration:** 90 Minutes.
- **Question Types:** Multiple choice (single and multiple response), code scenario interpretation, dependency tree debugging.
- **Target Engine Version:** Terraform **v1.15+** (including modern features: `import` blocks, `removed` blocks, `terraform test` unit/integration tests, ephemeral variables, write-only arguments, and the Terraform MCP server for agentic AI).

---

## 3. High-Value Reference Materials

- **Master Cheat Sheet:** [Terraform_Master_Cheat_Sheet.md](Terraform_Master_Cheat_Sheet.md)
- **Practice Exams:** [Practice Exams Hub](Practice_Exams/README.md)
  - [Practice Exam 1](Practice_Exams/Practice_Exam_1.md)
  - [Practice Exam 2](Practice_Exams/Practice_Exam_2.md)
  - [Practice Exam 3](Practice_Exams/Practice_Exam_3.md)
  - [Practice Exam 4](Practice_Exams/Practice_Exam_4.md)
