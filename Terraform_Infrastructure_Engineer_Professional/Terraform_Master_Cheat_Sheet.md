# Terraform Infrastructure Engineer Professional (002) — Master Cheat Sheet

> **Target Version:** HashiCorp Terraform v1.15+  
> **Exam Level:** Professional (002) — Advanced Production Engineering

---

## 1. Variable Definition Precedence Hierarchy (Strict Order: 1 = Lowest, 7 = Highest)

Understanding exact variable evaluation order is critical for production CI/CD debugging and exam scenario questions:

```
[1] Default values defined inside variable "x" { default = ... }
       │
[2] Environment variables: TF_VAR_<variable_name>
       │
[3] terraform.tfvars file (automatically loaded)
       │
[4] terraform.tfvars.json file (automatically loaded)
       │
[5] *.auto.tfvars or *.auto.tfvars.json files (alphabetical sorting order)
       │
[6] Command line flags: -var-file="custom.tfvars" (in order of invocation)
       │
[7] Command line flags: -var="key=value" (WINNER / HIGHEST PRECEDENCE)
```

> [!IMPORTANT]
> If multiple `*.auto.tfvars` files define the same variable, Terraform parses them in **lexicographical (alphabetical) order**. For example, `z-prod.auto.tfvars` overrides `a-prod.auto.tfvars`.

---

## 2. Terraform 1.10–1.15 Modern Core Features

### 2.1 Ephemeral Values & Write-Only Arguments (Zero State Leakage)
Terraform 1.10+ introduced **ephemeral variables** and **ephemeral resources**, preventing sensitive credentials (like dynamic IAM keys or short-lived Vault tokens) from being stored in cleartext inside `.tfstate` files:

```hcl
variable "db_admin_password" {
  type      = string
  ephemeral = true  # Cannot be persisted to state!
  sensitive = true
}

ephemeral "vault_kv_secret_v2" "db_creds" {
  mount = "secret"
  name  = "database/credentials"
}

resource "aws_db_instance" "primary" {
  allocated_storage = 50
  engine            = "postgres"
  username          = "postgres"
  password          = ephemeral.vault_kv_secret_v2.db_creds.data["password"]
  # In Terraform 1.11+, write-only arguments are transmitted directly to the provider API
  # and never written to the plan file or state file.
}
```

### 2.2 Declarative `import` Blocks
Replaces imperative `terraform import` CLI commands with code-reviewed, declarative syntax:

```hcl
import {
  to = digitalocean_droplet.web_server
  id = "384918293" # Provider-specific resource ID
}

# Generate configuration automatically if migrating legacy infrastructure:
# terraform plan -generate-config-out=generated_resources.tf
```

### 2.3 Declarative `removed` Blocks (State Decoupling Without Destroying Infrastructure)
Introduced in Terraform 1.7+, this tells Terraform to remove a resource from state without running a cloud API delete call:

```hcl
removed {
  from = aws_s3_bucket.legacy_data_lake

  lifecycle {
    destroy = false # Retains the S3 bucket in AWS, removes it from .tfstate!
  }
}
```

---

## 3. Native Testing Framework (`terraform test`)

Since Terraform 1.6+, testing is built directly into core HCL. Tests live in `*.tftest.hcl` files inside `/tests` or next to `.tf` files.

```hcl
# tests/vpc_validation.tftest.hcl

# Mock provider allows testing module logic without cloud API credentials:
mock_provider "digitalocean" {}

variables {
  region     = "nyc3"
  vpc_name   = "production-vpc"
  ip_range   = "10.116.0.0/20"
}

run "validate_cidr_calculation" {
  command = plan # Options: plan (read-only validation) or apply (creates test resources)

  assert {
    condition     = digitalocean_vpc.main.ip_range == "10.116.0.0/20"
    error_message = "VPC CIDR block must match required private range."
  }

  assert {
    condition     = can(regex("^[a-z0-9-]+$", var.vpc_name))
    error_message = "VPC name must be alphanumeric and lowercase."
  }
}
```

---

## 4. Terraform Model Context Protocol (MCP) Server for AI Coding Agents

In Terraform 1.15+, HashiCorp introduced the official **Terraform MCP Server**, enabling IDE AI coding agents (Anthropic Claude Desktop, Cursor, Gemini Code Assist) to interact with Terraform workspaces with full context:

- Exposes schemas of installed providers to the AI agent.
- Enables autonomous module discovery from internal and public registries.
- Safely validates HCL syntax and evaluates `terraform validate` directly in the agent loop.

---

## 5. HCL Expression & Function Reference

| Function / Expression | Example Syntax | Production Output / Behavior |
| :--- | :--- | :--- |
| **`can()`** | `can(regex("^[0-9]+$", var.id))` | Returns `true` if expression succeeds, `false` if it evaluates to an error. |
| **`try()`** | `try(local.config.port, 8080)` | Returns first expression that does not produce an error; ideal for optional object attributes. |
| **`flatten()`** | `flatten([for v in var.subnets : v.cidrs])` | Recursively collapses nested lists into a single flat list. |
| **`merge()`** | `merge(local.default_tags, var.custom_tags)` | Combines multiple maps; right-hand keys overwrite left-hand keys. |
| **`lookup()`** | `lookup(var.env_map, "prod", "default_val")` | Retrieves map value with fallback default if key does not exist. |
| **`cidrsubnet()`** | `cidrsubnet("10.0.0.0/16", 4, 2)` | Splits `10.0.0.0/16` adding 4 prefix bits: returns `10.0.32.0/20`. |
| **`for_each` on Maps** | `for_each = { for s in var.subnets : s.name => s }` | Turns a list of objects into a distinct map keyed by a unique identifier. |

---

## 6. Advanced State Surgery Commands

```bash
# Move resource address (e.g. refactoring into a module without destroying resources):
terraform state mv 'aws_security_group.web' 'module.network.aws_security_group.web'

# Move individual element within a for_each map:
terraform state mv 'digitalocean_droplet.workers["node-1"]' 'digitalocean_droplet.workers["node-primary"]'

# Remove resource from state (keeps cloud resource alive):
terraform state rm 'aws_iam_role.ci_cd_role'

# Replace legacy provider namespace (e.g. during upgrades):
terraform state replace-provider 'registry.terraform.io/-/digitalocean' 'digitalocean/digitalocean'

# Emergency state lock release (when CI pipeline crashes):
terraform force-unlock <LOCK-ID>
```

---

## 7. Dynamic Provider Credentials (OIDC Workflows)

Never store static AWS access keys, GCP service account JSON keys, or Vault tokens in CI/CD runners:

```
[CI Runner: GitHub Actions / GitLab / HCP Terraform]
        │
        ▼ 1. Request JWT ID Token with aud="https://github.com/..."
[OIDC Identity Provider]
        │
        ▼ 2. Present JWT token via AssumeRoleWithWebIdentity
[Cloud Provider: AWS STS / GCP Workload Identity / Vault]
        │
        ▼ 3. Issues short-lived (15 min - 1 hr) temporary session credentials
[Terraform Engine Execution]
```
