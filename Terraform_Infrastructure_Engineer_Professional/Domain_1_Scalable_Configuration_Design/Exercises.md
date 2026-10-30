# Domain 1: Scalable Configuration Design — Hands-On Lab Exercises

## Lab 1: Strongly Typed Multi-Tier EKS Module Contract with Nested `optional()` & Cross-Variable Validation

### Scenario
Your platform engineering team requires a reusable configuration contract for provisioning multi-tenant EKS node groups. Callers must be able to omit default scaling parameters and disk sizes while still enforcing strict invariants:
1. `min_size <= desired_size <= max_size`
2. `SPOT` node groups are prohibited when `environment == "prod"` unless `allow_spot_in_prod = true`.
3. Passing `null` to top-level variables must smoothly fall back to defaults (`nullable = false`).

### Step-by-Step Implementation (`variables.tf`)
```hcl
terraform {
  required_version = ">= 1.10.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

variable "environment" {
  type        = string
  default     = "prod"
  nullable    = false
  description = "Deployment environment tier (dev, staging, prod)"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment must be one of: dev, staging, prod."
  }
}

variable "allow_spot_in_prod" {
  type     = bool
  default  = false
  nullable = false
}

variable "node_groups" {
  type = map(object({
    capacity_type  = optional(string, "ON_DEMAND")
    instance_types = optional(list(string), ["m6i.large"])
    disk_size_gb   = optional(number, 100)
    scaling = optional(object({
      min_size     = optional(number, 2)
      desired_size = optional(number, 3)
      max_size     = optional(number, 10)
    }), {})
    labels = optional(map(string), {})
  }))
  nullable = false

  validation {
    condition = alltrue([
      for k, ng in var.node_groups :
      ng.scaling.min_size <= ng.scaling.desired_size &&
      ng.scaling.desired_size <= ng.scaling.max_size
    ])
    error_message = "Every node group must satisfy min_size <= desired_size <= max_size."
  }

  # Cross-variable validation (Terraform 1.9+):
  validation {
    condition = (
      var.environment != "prod" ||
      var.allow_spot_in_prod ||
      alltrue([for k, ng in var.node_groups : ng.capacity_type != "SPOT"])
    )
    error_message = "SPOT capacity_type is prohibited in prod unless allow_spot_in_prod is set to true."
  }
}
```

### Verification Commands
```bash
# 1. Test valid minimal caller payload relying on nested optional() defaults:
terraform plan -var='node_groups={"system":{}}'

# 2. Test cross-variable validation rejection in prod:
terraform plan -var='environment=prod' -var='node_groups={"workers":{"capacity_type":"SPOT"}}'
# Expected output: Error: Invalid value for variable (SPOT capacity_type is prohibited in prod...)
```

---

## Lab 2: Flattening Multi-Dimensional Hierarchies (`flatten` + Deterministic `for_each` Keys)

### Scenario
You are given a nested map of VPCs, where each VPC defines multiple subnets, and each subnet has a list of NACL ingress port numbers. You must flatten this 3-level hierarchy into a 1-dimensional map with deterministic string keys suitable for `aws_network_acl_rule` `for_each` iteration.

### Step-by-Step Implementation (`main.tf`)
```hcl
variable "vpc_topologies" {
  type = map(object({
    cidr = string
    subnets = map(object({
      cidr          = string
      ingress_ports = list(number)
    }))
  }))
  default = {
    core = {
      cidr = "10.10.0.0/16"
      subnets = {
        web = { cidr = "10.10.1.0/24", ingress_ports = [80, 443] }
        app = { cidr = "10.10.2.0/24", ingress_ports = [8080, 8443] }
      }
    }
  }
}

locals {
  flattened_nacl_rules = flatten([
    for vpc_key, vpc in var.vpc_topologies : [
      for sub_key, sub in vpc.subnets : [
        for idx, port in sub.ingress_ports : {
          composite_key = "${vpc_key}-${sub_key}-port-${port}"
          vpc_key       = vpc_key
          subnet_key    = sub_key
          subnet_cidr   = sub.cidr
          rule_number   = (idx + 1) * 100
          port          = port
        }
      ]
    ]
  ])

  nacl_rules_map = {
    for item in local.flattened_nacl_rules :
    item.composite_key => item
  }
}

output "deterministic_for_each_keys" {
  value = keys(local.nacl_rules_map)
  # Produces: ["core-app-port-8080", "core-app-port-8443", "core-web-port-443", "core-web-port-80"]
}
```

---

## Lab 3: Ephemeral Variables, Ephemeral Resources & Write-Only Attributes (Terraform 1.10 / 1.11+)

### Scenario
Provision an AWS RDS PostgreSQL instance whose password is generated dynamically at runtime and passed via a Write-Only attribute (`password_wo`) so that the password string is **completely absent** from `terraform.tfstate` and saved plan files.

### Step-by-Step Implementation (`ephemeral_rds.tf`)
```hcl
variable "password_rotation_version" {
  type        = number
  default     = 1
  description = "Increment this integer to trigger a write-only password rotation on next apply"
}

ephemeral "random_password" "db_master_pw" {
  length           = 24
  special          = true
  override_special = "!#$%&*()-_=+[]{}<>:?"
}

resource "aws_secretsmanager_secret" "rds_credentials" {
  name = "prod/payments/rds-master-pw"
}

resource "aws_secretsmanager_secret_version" "rds_credentials_ver" {
  secret_id                = aws_secretsmanager_secret.rds_credentials.id
  secret_string_wo         = ephemeral.random_password.db_master_pw.result
  secret_string_wo_version = var.password_rotation_version
}

resource "aws_db_instance" "payments" {
  identifier          = "payments-prod-db"
  engine              = "postgres"
  instance_class      = "db.t4g.micro"
  allocated_storage   = 20
  username            = "postgres_admin"
  skip_final_snapshot = true

  # Write-only password never touches terraform.tfstate!
  password_wo         = ephemeral.random_password.db_master_pw.result
  password_wo_version = var.password_rotation_version
}
```

### Verification Commands
```bash
# After applying, inspect raw state JSON to verify zero plaintext secret leakage:
terraform state pull | grep -E "password_wo|secret_string_wo"
# Result: password_wo and secret_string_wo are null in state; only password_wo_version = 1 is stored!
```
