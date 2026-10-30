# Domain 3: Lifecycle Operations & Declarative Refactoring — Hands-On Lab Exercises

## Lab 1: Zero-Downtime Refactoring from `count` to `for_each` Using Declarative `moved` Blocks

### Scenario
A legacy module provisioned three AWS subnets using `count = 3`. You need to refactor the module to use `for_each` keyed by Availability Zone (`"us-east-1a"`, `"us-east-1b"`, `"us-east-1c"`) so that future subnet additions or removals never shift array indices, while guaranteeing **zero subnet destruction** during the migration.

### Step-by-Step Implementation (`refactoring.tf`)
```hcl
variable "subnets_by_az" {
  type = map(string)
  default = {
    "us-east-1a" = "10.20.1.0/24"
    "us-east-1b" = "10.20.2.0/24"
    "us-east-1c" = "10.20.3.0/24"
  }
}

resource "aws_subnet" "private" {
  for_each = var.subnets_by_az

  vpc_id            = aws_vpc.main.id
  availability_zone = each.key
  cidr_block        = each.value
}

# Declarative state migration from legacy count indices to new for_each keys:
moved {
  from = aws_subnet.private[0]
  to   = aws_subnet.private["us-east-1a"]
}

moved {
  from = aws_subnet.private[1]
  to   = aws_subnet.private["us-east-1b"]
}

moved {
  from = aws_subnet.private[2]
  to   = aws_subnet.private["us-east-1c"]
}
```

### Verification Commands
```bash
terraform plan
# Plan output confirms:
#   # aws_subnet.private[0] has moved to aws_subnet.private["us-east-1a"]
#   # aws_subnet.private[1] has moved to aws_subnet.private["us-east-1b"]
#   # aws_subnet.private[2] has moved to aws_subnet.private["us-east-1c"]
# Plan: 0 to add, 0 to change, 0 to destroy.
```

---

## Lab 2: Bulk Brownfield Resource Adoption with `import` (`for_each`) & `-generate-config-out`

### Scenario
Your cloud security team discovered four unmanaged IAM roles created manually in AWS. You must import all four roles declaratively in a single PR and auto-generate their HCL resource blocks.

### Step-by-Step Implementation (`imports.tf`)
```hcl
locals {
  unmanaged_iam_roles = {
    ci_deployer     = "CorpGitHubActionsDeployerRole"
    audit_reader    = "CorpSecurityAuditReadOnlyRole"
    billing_analyst = "CorpFinOpsBillingRole"
    eks_node        = "CorpEKSWorkerNodeRole"
  }
}

import {
  for_each = local.unmanaged_iam_roles
  to       = aws_iam_role.adopted[each.key]
  id       = each.value
}
```
```bash
# 1. Generate HCL configuration into a new file:
terraform plan -generate-config-out=generated_iam_roles.tf

# 2. Consolidate or verify generated resource block and run plan:
terraform plan -out=import.tfplan
# Expected: Plan: 4 to import, 0 to add, 0 to change, 0 to destroy.
```

---

## Lab 3: Decommissioning Resources Safely with `removed` Blocks & Continuous Health Validation (`check`)

### Scenario
Retire a legacy S3 bucket from Terraform state without deleting the underlying S3 bucket in AWS, and add a non-blocking `check` block that verifies an internal HTTPS service returns `200 OK`.

### Step-by-Step Implementation (`lifecycle_and_checks.tf`)
```hcl
# 1. Safely orphan legacy S3 bucket from Terraform state without destroying it:
removed {
  from = aws_s3_bucket.legacy_compliance_archive

  lifecycle {
    destroy = false
  }
}

# 2. Non-blocking post-apply continuous health check:
check "internal_portal_tls_and_status" {
  data "http" "portal" {
    url = "https://portal.internal.corp/api/v1/status"
  }

  assert {
    condition     = data.http.portal.status_code == 200
    error_message = "Internal portal returned HTTP ${data.http.portal.status_code}; expected 200."
  }
}
```
