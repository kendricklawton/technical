# Domain 6: Advanced Modules & Native Testing Frameworks — Hands-On Lab Exercises

## Lab 1: Authoring Unit & Negative Validation Tests with `mock_provider` (`.tftest.hcl`)

### Scenario
Write a comprehensive `.tftest.hcl` test suite for an AWS S3 module that:
1. Uses `mock_provider "aws"` so zero AWS credentials are needed.
2. Uses `command = apply` against the mock provider to verify computed ARN outputs.
3. Uses `expect_failures` to verify that non-HTTPS access or unapproved environment names are rejected.

### Step-by-Step Implementation (`tests/bucket.tftest.hcl`)
```hcl
mock_provider "aws" {
  mock_resource "aws_s3_bucket" {
    defaults = {
      id  = "corp-mock-bucket-prod"
      arn = "arn:aws:s3:::corp-mock-bucket-prod"
    }
  }
}

variables {
  bucket_prefix = "corp-audit"
  environment   = "prod"
}

run "verify_computed_arn_and_tags" {
  command = apply

  assert {
    condition     = output.bucket_arn == "arn:aws:s3:::corp-mock-bucket-prod"
    error_message = "Output bucket_arn did not match expected mock ARN."
  }

  assert {
    condition     = aws_s3_bucket.this.tags["Environment"] == "prod"
    error_message = "Environment tag was not applied to S3 bucket."
  }
}

run "reject_short_bucket_prefix" {
  command = plan

  variables {
    bucket_prefix = "ab" # Violates min length >= 4 validation rule
  }

  expect_failures = [
    var.bucket_prefix
  ]
}
```
```bash
# Execute the test suite offline:
terraform test -verbose
```

---

## Lab 2: Isolating Child Module Dependencies with `override_module` and `override_data`

### Scenario
Your application module calls a child module `module.database` and queries `data "aws_ssm_parameter" "db_kms_key"`. Test the application module in isolation using `override_module` and `override_data`.

### Step-by-Step Implementation (`tests/isolated_app.tftest.hcl`)
```hcl
mock_provider "aws" {}

run "verify_ecs_task_env_injection" {
  command = plan

  override_module {
    target = module.database
    outputs = {
      writer_endpoint = "mock-postgres.cluster-xyz.us-east-1.rds.amazonaws.com:5432"
    }
  }

  override_data {
    target = data.aws_ssm_parameter.db_kms_key
    values = {
      value = "arn:aws:kms:us-east-1:123456789012:key/mock-kms-key"
    }
  }

  assert {
    condition     = aws_ecs_task_definition.app.Family == "payments-worker"
    error_message = "ECS task family mismatch."
  }
}
```
