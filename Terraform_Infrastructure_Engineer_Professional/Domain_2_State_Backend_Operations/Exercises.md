# Domain 2: State and Backend Operations — Hands-On Lab Exercises

## Lab 1: Configuring S3 Native State Locking (`use_lockfile = true`) & Partial Backend Initialization

### Scenario
Configure a production S3 backend utilizing Terraform 1.10+ native S3 lockfiles (`use_lockfile = true`) without DynamoDB, and initialize environment-specific state keys dynamically via partial configuration files.

### Step-by-Step Implementation
1. **Create Partial Backend Block (`backend.tf`):**
   ```hcl
   terraform {
     required_version = ">= 1.10.0"

     backend "s3" {
       use_lockfile = true
       encrypt      = true
     }
   }
   ```
2. **Create Environment Backend Config (`config/prod.s3.tfbackend`):**
   ```hcl
   bucket     = "corp-terraform-state-prod-123456789012"
   key        = "platform/core-network/terraform.tfstate"
   region     = "us-east-1"
   kms_key_id = "arn:aws:kms:us-east-1:123456789012:key/mrk-state-encryption"
   ```
3. **Initialize and Migrate Existing Local State:**
   ```bash
   terraform init -migrate-state -backend-config="config/prod.s3.tfbackend"
   ```

---

## Lab 2: Safe Out-of-Band Drift Reconciliation & Automated CI Drift Detection

### Scenario
An on-call engineer manually added an emergency tag `IncidentTicket = "INC-9042"` to a production VPC in the AWS Console. You need to detect this drift in an automated pipeline using `-detailed-exitcode` and safely absorb the tag into `.tfstate` without modifying any cloud infrastructure.

### Step-by-Step Implementation
```bash
#!/usr/bin/env bash
set -u

# 1. Run non-interactive drift check
terraform plan -refresh-only -detailed-exitcode -out=drift.tfplan -no-color
EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ]; then
  echo "OK: Zero infrastructure drift detected."
elif [ $EXIT_CODE -eq 1 ]; then
  echo "ERROR: Terraform plan failed due to error."
  exit 1
elif [ $EXIT_CODE -eq 2 ]; then
  echo "WARNING: Out-of-band drift detected! Displaying state refresh diff:"
  terraform show -no-color drift.tfplan
  # Safe reconciliation after approval:
  # terraform apply -input=false drift.tfplan
fi
```

---

## Lab 3: Emergency State Disaster Recovery (`state pull`, Serial Increment & `state push`)

### Scenario
A broken script accidentally ran `terraform state rm aws_rds_cluster.prod` against your remote state. Because S3 Bucket Versioning is enabled, you need to restore the prior `.tfstate` version safely while respecting Terraform's `lineage` and `serial` protections.

### Step-by-Step Implementation
```bash
# 1. Inspect current (broken) remote state serial and lineage:
terraform state pull | jq '{serial: .serial, lineage: .lineage}'
# Output: { "serial": 42, "lineage": "a1b2c3d4-..." }

# 2. Download the previous intact version from S3 object versioning:
VERSION_ID=$(aws s3api list-object-versions \
  --bucket corp-terraform-state-prod-123456789012 \
  --prefix platform/core-network/terraform.tfstate \
  --query 'Versions[1].VersionId' --output text)

aws s3api get-object \
  --bucket corp-terraform-state-prod-123456789012 \
  --key platform/core-network/terraform.tfstate \
  --version-id "$VERSION_ID" \
  recovered_raw.tfstate

# 3. Increment serial in recovered state from 41 -> 43 (> 42) so standard state push succeeds without -force:
jq '.serial = 43' recovered_raw.tfstate > recovered_ready.tfstate

# 4. Push restored state and verify zero diff:
terraform state push recovered_ready.tfstate
terraform plan -detailed-exitcode
```
