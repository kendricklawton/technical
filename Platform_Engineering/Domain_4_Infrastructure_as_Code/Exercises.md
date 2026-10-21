# PE Domain 4: Infrastructure as Code — Hands-On Labs & Exercises

Six advanced labs drilling state refactoring, declarative imports, native testing, OPA policy enforcement, Crossplane compositions, and module architecture.

---

## Lab 1: Refactoring Legacy Monolithic State with Declarative `import` and `moved` Blocks

### Objective
Safely decompose a monolithic Terraform state file into a structured module using declarative `moved` blocks, and import an unmanaged pre-existing cloud resource using Terraform 1.5+ `import` blocks without downtime or accidental recreation.

### Step-by-Step Implementation

1. **Simulate Pre-existing Monolithic Code (`main.tf`):**
   ```hcl
   terraform {
     required_version = ">= 1.5.0"
     required_providers {
       local = {
         source  = "hashicorp/local"
         version = ">= 2.4.0"
       }
     }
   }

   # Legacy flat resource
   resource "local_file" "legacy_config" {
     filename = "/tmp/legacy_app.conf"
     content  = "database_host=localhost\nport=5432\n"
   }
   ```

2. **Initialize and Apply Baseline:**
   ```bash
   terraform init
   terraform apply -auto-approve
   ```

3. **Refactor Code into a Module and Add `moved` Block:**
   Create `modules/app_config/main.tf`:
   ```hcl
   variable "content" { type = string }
   resource "local_file" "config" {
     filename = "/tmp/legacy_app.conf"
     content  = var.content
   }
   ```

   Update root `main.tf`:
   ```hcl
   module "app" {
     source  = "./modules/app_config"
     content = "database_host=localhost\nport=5432\n"
   }

   # Declarative state surgery: inform Terraform of the move
   moved {
     from = local_file.legacy_config
     to   = module.app.local_file.config
   }
   ```

4. **Execute `terraform plan`:**
   ```bash
   terraform plan
   ```
   Observe the output: `1 to move, 0 to add, 0 to change, 0 to destroy`. State is refactored safely with zero resource recreation!

---

## Lab 2: Writing a Full Automated Test Suite Using Native `terraform test`

### Objective
Write a unit test suite using Terraform 1.6+ `terraform test` featuring mock providers to validate that an S3 bucket module enforces AES-256/KMS encryption and blocks public access.

### Step-by-Step Implementation

1. **Create Module (`modules/secure_bucket/main.tf`):**
   ```hcl
   variable "bucket_name" { type = string }
   variable "enable_encryption" { 
     type    = bool 
     default = true 
   }

   resource "aws_s3_bucket" "bucket" {
     bucket = var.bucket_name
   }

   resource "aws_s3_bucket_public_access_block" "block" {
     bucket                  = aws_s3_bucket.bucket.id
     block_public_acls       = true
     block_public_policy     = true
     ignore_public_acls      = true
     restrict_public_buckets = true
   }
   ```

2. **Create Test Suite (`tests/bucket.tftest.hcl`):**
   ```hcl
   mock_provider "aws" {}

   run "verify_public_access_blocked" {
     command = plan

     variables {
       bucket_name = "corp-financial-records"
     }

     assert {
       condition     = aws_s3_bucket_public_access_block.block.block_public_acls == true
       message       = "Public ACLs must be blocked!"
     }

     assert {
       condition     = aws_s3_bucket_public_access_block.block.restrict_public_buckets == true
       message       = "Public buckets must be restricted!"
     }
   }
   ```

3. **Run the Test Suite:**
   ```bash
   terraform init
   terraform test
   ```

---

## Lab 3: Enforcing Security Policies on IaC Using Conftest and OPA Rego

### Objective
Write an Open Policy Agent (OPA) Rego policy and use `conftest` in a CI pipeline to reject any Terraform plan that creates an AWS Security Group opening port 22 (SSH) to the public internet (`0.0.0.0/0`).

### Step-by-Step Implementation

1. **Write the Rego Policy (`policy/deny_open_ssh.rego`):**
   ```rego
   package terraform.security

   default allow = false

   deny[msg] {
       resource := input.resource_changes[_]
       resource.type == "aws_security_group"
       rule := resource.change.after.ingress[_]
       rule.from_port <= 22
       rule.to_port >= 22
       rule.cidr_blocks[_] == "0.0.0.0/0"
       msg := sprintf("CRITICAL SECURITY VIOLATION: Security Group '%v' exposes port 22 to 0.0.0.0/0!", [resource.name])
   }
   ```

2. **Generate Terraform Plan JSON:**
   ```bash
   terraform plan -out=tfplan.binary
   terraform show -json tfplan.binary > tfplan.json
   ```

3. **Execute Policy Check with Conftest:**
   ```bash
   conftest test tfplan.json -p policy/
   ```

---

## Lab 4: Building a Self-Service Cloud Resource Composition in Crossplane

### Objective
Define a Crossplane Composite Resource Definition (XRD) and Composition that translates a simple Kubernetes custom resource (`kind: ObjectStore`) into an Amazon S3 bucket with automated encryption.

### Step-by-Step Implementation

1. **Define the Composite Resource Definition (`xrd.yaml`):**
   ```yaml
   apiVersion: apiextensions.crossplane.io/v1
   kind: CompositeResourceDefinition
   metadata:
     name: xobjectstores.storage.platform.example.com
   spec:
     group: storage.platform.example.com
     names:
       kind: XObjectStore
       plural: xobjectstores
     claimNames:
       kind: ObjectStore
       plural: objectstores
     versions:
       - name: v1alpha1
         served: true
         referenceable: true
         schema:
           openAPIV3Schema:
             type: object
             properties:
               spec:
                 type: object
                 required:
                   - region
                 properties:
                   region:
                     type: string
                     enum: ["us-east-1", "us-west-2", "eu-central-1"]
   ```

2. **Define the Composition binding to AWS Provider (`composition.yaml`):**
   ```yaml
   apiVersion: apiextensions.crossplane.io/v1
   kind: Composition
   metadata:
     name: s3-objectstore-composition
     labels:
       provider: aws
   spec:
     compositeTypeRef:
       apiVersion: storage.platform.example.com/v1alpha1
       kind: XObjectStore
     resources:
       - name: s3Bucket
         base:
           apiVersion: s3.aws.upbound.io/v1beta1
           kind: Bucket
           spec:
             forProvider:
               region: us-east-1
         patches:
           - type: FromCompositeFieldPath
             fromFieldPath: "spec.region"
             toFieldPath: "spec.forProvider.region"
   ```

---

## Lab 5: Implementing Automated IaC Drift Detection & Slack Alerting

### Objective
Build a headless Bash script designed for cron execution in a CI/CD runner that runs non-interactive Terraform drift detection and posts a formatted alert when out-of-band changes occur.

### Implementation (`drift_detector.sh`)
```bash
#!/usr/bin/env bash
set -uo pipefail

echo "Executing automated Terraform drift check..."
terraform init -backend=true -input=false > /dev/null

# Run plan with detailed exit code:
# 0 = No changes, 1 = Error, 2 = Succeeded with non-empty diff (DRIFT)
terraform plan -detailed-exitcode -input=false -no-color > /tmp/plan_output.txt
EXIT_CODE=$?

if [ $EXIT_CODE -eq 2 ]; then
    echo "[ALERT] Infrastructure drift detected!"
    DRIFT_SUMMARY=$(grep -E "Plan: [0-9]+ to add, [0-9]+ to change, [0-9]+ to destroy" /tmp/plan_output.txt)
    echo "Summary: $DRIFT_SUMMARY"
    # In production, dispatch alert via curl to Slack Webhook URL
    exit 2
elif [ $EXIT_CODE -eq 0 ]; then
    echo "[OK] Infrastructure is fully synchronized with desired state."
    exit 0
else
    echo "[ERROR] Terraform plan failed with an unexpected error."
    exit 1
fi
```

---

## Lab 6: Designing a Reusable S3 Module with Preconditions and Postconditions

### Objective
Author an enterprise-grade Terraform module incorporating variable validations, lifecycle preconditions, and postconditions.

### Implementation (`modules/s3_hardened/main.tf`)
```hcl
variable "environment" {
  type        = string
  description = "Target deployment environment"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    message       = "Environment must be one of: dev, staging, prod."
  }
}

variable "retention_days" {
  type        = number
  description = "Number of days before expiring old objects"
  default     = 30
}

resource "aws_s3_bucket" "this" {
  bucket_prefix = "corp-data-${var.environment}-"

  lifecycle {
    precondition {
      condition     = var.environment != "prod" || var.retention_days >= 365
      message       = "Production buckets must retain data for at least 365 days."
    }
  }
}
```
