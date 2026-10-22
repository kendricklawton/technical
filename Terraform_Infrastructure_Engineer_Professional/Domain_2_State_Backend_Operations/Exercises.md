# Domain 2: State and Backend Operations — Production Exercises & Labs

---

## Lab 1: Zero-Downtime State Backend Migration (Local to Remote S3)

### Objective
Migrate an existing local `terraform.tfstate` holding active production resources into a secure AWS S3 remote backend with DynamoDB state locking without modifying running infrastructure.

### Execution Steps
1. Create backend infrastructure configuration `backend_infra.tf`:
```hcl
resource "aws_s3_bucket" "tf_state" {
  bucket = "enterprise-tfstate-storage-2026"
  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "v" {
  bucket = aws_s3_bucket.tf_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_dynamodb_table" "locks" {
  name         = "tf-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

2. Add the `backend` block to `main.tf`:
```hcl
terraform {
  backend "s3" {
    bucket         = "enterprise-tfstate-storage-2026"
    key            = "prod/vpc.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-state-locks"
    encrypt        = true
  }
}
```

3. Initialize migration:
```bash
terraform init -migrate-state
# Answer 'yes' to copy existing state to new backend
```

---

## Lab 2: Investigating and Reconciling Out-of-Band Cloud Drift

### Objective
Simulate out-of-band drift where an operator manually increased a DigitalOcean Droplet tag or firewall rule directly in the console. Detect the drift and reconcile state cleanly.

```bash
# 1. Run refresh-only plan to detect differences between remote API and state:
terraform plan -refresh-only

# 2. Review the plan output to confirm only out-of-band attributes are flagged.
# 3. Apply the refresh to update .tfstate:
terraform apply -refresh-only
```
