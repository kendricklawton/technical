# Domain 6: Advanced Modules & Native Testing Frameworks — Complete Study Guide

**Exam Weight:** 10% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** Composable Module Architecture & Dependency Inversion, Semantic Versioning constraint operators (`~> 2.1.0` vs. `~> 2.1`), Private Registry publishing requirements (`terraform-<PROVIDER>-<NAME>`), Module Refactoring with `moved` blocks, and deep mastery of the **Native Terraform Test Framework (`terraform test` / `.tftest.hcl`)** including sequential vs. isolated `run` blocks, `command = plan | apply`, negative testing with `expect_failures`, and **Mock Providers (`mock_provider`, `mock_resource`, `override_resource`, `override_data`, `override_module`)**.

---

## 1. Enterprise Module Architecture, Composition & Dependency Inversion

Monolithic "god modules" that attempt to provision VPCs, IAM roles, KMS keys, RDS databases, and EKS clusters inside a single deeply nested module tree create brittle interfaces and impossible upgrade paths.

```text
+-----------------------------------------------------------------------------------+
|          FLAT MODULE COMPOSITION & DEPENDENCY INVERSION (RECOMMENDED)             |
+-----------------------------------------------------------------------------------+
  [Root Workspace Configuration (main.tf)]
        |
        +---> [module "network" (source = "app.terraform.io/corp/vpc/aws")]
        |          +-- Outputs: vpc_id, database_subnet_ids, app_subnet_ids
        |                  |
        |                  v (Passed sideways via Root Module — Dependency Inversion!)
        +---> [module "rds"     (source = "app.terraform.io/corp/rds/aws")]
        |          +-- Inputs:  vpc_id = module.network.vpc_id
        |                       subnet_ids = module.network.database_subnet_ids
        |          +-- Outputs: db_endpoint, db_security_group_id
        |                  |
        |                  v (Passed sideways via Root Module)
        +---> [module "eks"     (source = "app.terraform.io/corp/eks/aws")]
                   +-- Inputs:  vpc_id = module.network.vpc_id
                                subnet_ids = module.network.app_subnet_ids
```

### 1.1 Dependency Inversion & Flat Module Hierarchies
* **Flat Composition Rule:** Keep module call trees shallow (1 to 2 levels deep). Instead of having `module.eks` internally instantiate `module.vpc`, instantiate both `module.network` and `module.eks` at the root level and pass `module.network.vpc_id` into `module.eks` as an input variable (**Dependency Inversion**).
* **Conditional Data Source Lookups vs. Direct Input Injection:** Never force a child module to execute `data "aws_vpc"` if the caller already created the VPC in the same run (because the VPC might not exist yet during `plan`). Accept `variable "vpc_id" { type = string }` directly!

### 1.2 Semantic Versioning Constraint Operators
Understanding the pessimistic constraint operator (`~>`) is tested on almost every exam sitting:
* **`= 2.1.0` (or `2.1.0`):** Exact version match only.
* **`!= 2.1.3`:** Excludes a specific broken patch release (often combined: `">= 2.1.0, != 2.1.3, < 3.0.0"`).
* **`~> 2.1.0` (3-segment pessimistic constraint):** Allows **only the rightmost component (PATCH)** to increment: matches **`>= 2.1.0, < 2.2.0`** (e.g., `2.1.1`, `2.1.9` match; `2.2.0` is rejected!).
* **`~> 2.1` (2-segment pessimistic constraint):** Allows the **rightmost component (MINOR)** and patch to increment: matches **`>= 2.1.0, < 3.0.0`** (e.g., `2.2.0`, `2.9.5` match; `3.0.0` is rejected!).

### 1.3 Publishing Modules to the Private Registry (HCP Terraform / TFE)
For a VCS-backed module repository to be ingested into the HCP Terraform or Terraform Enterprise Private Registry, it **must** satisfy three strict requirements:
1. **Repository Naming Convention:** The Git repository **must** be named **`terraform-<PROVIDER>-<NAME>`** (e.g., `terraform-aws-vpc-enterprise` or `terraform-google-gke-cluster`).*(Note: Branch-based modules in HCP Terraform can bypass the Git tag requirement, but tag-based registry modules require this exact naming convention).*
2. **Semantic Version Git Tags:** Releases must be tagged using valid SemVer (`v1.0.0` or `1.0.0`). Pushing a new Git tag via webhook automatically publishes the new version to the Private Registry.
3. **Standard Module Structure:** Root directory containing `main.tf`, `variables.tf`, `outputs.tf`, and `README.md` (with optional `examples/` and `modules/` subdirectories).

---

## 2. Native Terraform Test Framework (`terraform test` & `.tftest.hcl`)

Introduced in Terraform 1.6 and enhanced with **Mock Providers** in Terraform 1.7+, `terraform test` allows you to execute fast, deterministic unit and integration tests written directly in native HCL without writing Go code (`Terratest`)!

```text
+-----------------------------------------------------------------------------------+
|               NATIVE TERRAFORM TEST FRAMEWORK EXECUTION MODES                     |
+-----------------------------------------------------------------------------------+
  [terraform test] ---> Scans directory and `tests/` for `*.tftest.hcl`
                              |
        +---------------------+---------------------+
        |                                           |
        v                                           v
  [Unit Test Mode: command = plan]            [Integration Mode: command = apply]
  + mock_provider "aws" {} (TF 1.7+)          + Provisions ephemeral real or mocked
  + Zero cloud credentials required!            resources into temporary test state
  + Validates locals, dynamic blocks,         + Evaluates post-apply outputs/data
    variable validations & expect_failures    + Automatically destroys all created
    in milliseconds!                            resources in reverse order at end!
```

### 2.1 Anatomy of a `.tftest.hcl` File (`variables`, `run`, `assert`, `expect_failures`)
* **File Discovery:** By default, `terraform test` discovers all files ending in **`.tftest.hcl`** (or `.tftest.json`) in the root directory or the **`tests/`** subdirectory.
* **`run "<name>"` Blocks:** Execute sequentially from top to bottom.
  * **`command = apply` (Default!):** If `command` is omitted, `terraform test` defaults to `command = apply`! Set **`command = plan`** explicitly for fast unit tests.
  * **Variable Scoping:** File-level `variables { ... }` apply to all `run` blocks in the test file; `variables { ... }` inside a specific `run` block override the file-level variables for that single `run` block.
  * **Referencing Prior `run` Outputs (`run.<prev_run_name>.<output>`):** Subsequent `run` blocks can reference outputs produced by earlier `run` blocks (e.g., `vpc_id = run.setup_network.vpc_id`) to stage multi-step integration tests.
  * **`expect_failures = [...]` (Negative Testing):** Asserts that a specific validation rule, precondition, postcondition, or check block **must fail** for the test to pass!

```hcl
# tests/s3_module_unit.tftest.hcl
variables {
  environment = "prod"
  bucket_name = "corp-financial-ledger"
}

# Run 1: Verify positive plan behavior
run "validate_prod_bucket_naming_and_encryption" {
  command = plan

  assert {
    condition     = aws_s3_bucket.this.bucket == "corp-financial-ledger-prod"
    error_message = "Bucket name did not append the environment suffix correctly."
  }

  assert {
    condition     = aws_s3_bucket_server_side_encryption_configuration.this.rule[0].apply_server_side_encryption_by_default[0].sse_algorithm == "aws:kms"
    error_message = "Production buckets must default to aws:kms encryption."
  }
}

# Run 2: Verify negative validation rejection when invalid environment is passed
run "reject_invalid_environment_name" {
  command = plan

  variables {
    environment = "sandbox_unapproved"
  }

  # Test PASSES only if var.environment's validation block throws an error!
  expect_failures = [
    var.environment,
  ]
}
```

### 2.2 Mock Providers & Resource Overrides (Terraform 1.7+)
During unit tests (`command = plan` or even `command = apply`), resources often depend on computed attributes (`id`, `arn`, `endpoint`) that are normally unknown until a real cloud API creates the object. **Mock Providers (`mock_provider`)** simulate the provider plugin entirely in memory without requiring cloud credentials or network access!

```hcl
# tests/mocked_eks.tftest.hcl
mock_provider "aws" {
  # Provide deterministic default values for specific resource types across all runs:
  mock_resource "aws_vpc" {
    defaults = {
      id         = "vpc-0123456789abcdef0"
      arn        = "arn:aws:ec2:us-east-1:123456789012:vpc/vpc-0123456789abcdef0"
      cidr_block = "10.50.0.0/16"
    }
  }

  mock_data "aws_caller_identity" {
    defaults = {
      account_id = "123456789012"
      arn        = "arn:aws:iam::123456789012:root"
    }
  }
}

# You can also override a SINGLE specific resource, data source, or child module inside a run block:
run "test_iam_role_trust_policy_with_overrides" {
  command = apply # Executes against mock_provider in memory in <100ms!

  override_module {
    target = module.kms_vault
    outputs = {
      key_arn = "arn:aws:kms:us-east-1:123456789012:key/mock-vault-key-999"
    }
  }

  override_data {
    target = data.aws_Partition.current
    values = {
      partition = "aws-us-gov"
    }
  }

  assert {
    condition     = strcontains(aws_iam_role.worker.assume_role_policy, "aws-us-gov")
    error_message = "IAM trust policy failed to render GovCloud partition dynamically."
  }
}
```

### 2.3 State Isolation vs. Shared State Across `run` Blocks
* **Default Behavior (Shared State per Test File):** By default, all `run` blocks inside a single `.tftest.hcl` file share and mutate the same internal test state sequentially, unless a `run` block specifies a helper `module { source = "./testing/setup" }` (which gets its own isolated state).
* **`state_key` Attribute:** You can explicitly control which `run` blocks share state or run in isolated state buckets within a `.tftest.hcl` file using `state_key = "isolated_state_1"`.
* **Automatic Teardown Order:** When `terraform test` finishes executing a `.tftest.hcl` file that used `command = apply`, it destroys created infrastructure in **reverse `run` block execution order** (so downstream resources created in Run 2 are destroyed before foundational VPC resources created in Run 1).

---

## 3. Domain 6 Production Scenario & Exam Trap Matrix

* **Scenario 1: Testing that an invalid CIDR input causes `variable "cidr"` validation to fail**
  * **Trap:** Writing an `assert { condition = !can(...) }` block inside `run` when testing `var.cidr` validation.
  * **Reality:** If `var.cidr` fails validation, the `run` block aborts *before* evaluating `assert` blocks, failing the test! You must use **`expect_failures = [var.cidr]`** inside the `run` block so Terraform knows the validation failure is the expected passing outcome.
* **Scenario 2: Difference between `~> 3.2.0` and `~> 3.2`**
  * **Trap:** Assuming `~> 3.2.0` permits upgrading to `3.3.0`.
  * **Reality:** `~> 3.2.0` only allows patch increments (`>= 3.2.0, < 3.3.0`). Only `~> 3.2` allows minor increments (`>= 3.2.0, < 4.0.0`).

---

## 4. Platform Engineering Deep Dive: "Golden Path" YAML/JSON Data-Driven Module Factories

A signature pattern of modern Platform Engineering teams (using Backstage, Port, or GitOps PR generators) is allowing product developers to self-service infrastructure by submitting a simple **YAML or JSON specification file** (`tenants/payments.yaml`) without writing raw HCL. Terraform ingests these files using `fileset()`, `yamldecode()`, and `for_each`:

```hcl
# main.tf — Platform Engineering Self-Service Tenant Factory
locals {
  # 1. Discover all YAML files under the tenants/ directory:
  tenant_files = fileset("${path.module}/tenants", "*.yaml")

  # 2. Decode each YAML file into a strongly keyed map by tenant filename:
  tenants = {
    for f in local.tenant_files :
    trimsuffix(f, ".yaml") => yamldecode(file("${path.module}/tenants/${f}"))
  }
}

# 3. Instantiate the vetted Private Registry Golden Path module per tenant:
module "tenant_namespace" {
  source  = "app.terraform.io/corp-platform/eks-tenant-namespace/kubernetes"
  version = "~> 2.4.0"

  for_each = local.tenants

  tenant_name  = each.key
  team_email   = each.value.owner_email
  cpu_quota    = try(each.value.quotas.cpu, "4")
  memory_quota = try(each.value.quotas.memory, "8Gi")
  enable_istio = try(each.value.mesh_enabled, true)
}
```
* **Why `fileset()` + `yamldecode()` Excels at Scale:**
  * Adding a new tenant (`tenants/analytics.yaml`) adds a single deterministic key (`module.tenant_namespace["analytics"]`) to `for_each` without touching any existing tenant's state address.
  * Schema validation is enforced by feeding the decoded YAML object into a typed submodule input variable with `object({...})` and `optional()` defaults!

---

## 5. Advanced `.tftest.hcl` Patterns: Helper Setup Modules & Parallel Test Execution

### 5.1 Multi-Stage Integration Testing with Helper Modules (`module` block in `run`)
When testing a module that requires prerequisite infrastructure (such as an existing VPC or KMS key) during a real `command = apply` integration test, a `run` block can point to a dedicated **helper setup module** using an inner `module { source = "./testing/setup" }` block:

```hcl
# tests/integration_rds.tftest.hcl

# Stage 1: Provision ephemeral prerequisite VPC using helper module
run "setup_test_vpc" {
  command = apply

  module {
    source = "./testing/setup_vpc"
  }
}

# Stage 2: Test the root RDS module using outputs from Stage 1 (`run.setup_test_vpc.*`)
run "provision_rds_in_test_vpc" {
  command = apply

  variables {
    vpc_id     = run.setup_test_vpc.vpc_id
    subnet_ids = run.setup_test_vpc.database_subnet_ids
  }

  assert {
    condition     = aws_db_instance.this.MultiAZ == true
    error_message = "Production RDS module must enable MultiAZ by default."
  }
}
# Teardown Order: Terraform automatically destroys `provision_rds_in_test_vpc` FIRST,
# and only then destroys `setup_test_vpc`!
```

### 5.2 Parallel Test Execution (`parallel = true` — Terraform 1.9+)
By default, `run` blocks in `.tftest.hcl` execute sequentially because they share the same default state key. In Terraform 1.9+, you can accelerate large test suites by marking independent `run` blocks with **`parallel = true`** (provided they either use `command = plan` or specify distinct `state_key` values so they don't mutate the same state concurrently!).

