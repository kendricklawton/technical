# Domain 4: CI/CD & Automation Workflows — Hands-On Lab Exercises

## Lab 1: Multi-Platform Lockfile Generation & Verification (`terraform providers lock`)

### Scenario
Your engineering team uses macOS (`darwin_arm64`) laptops while CI/CD pipelines run on Ubuntu x86_64 (`linux_amd64`) and AWS Graviton (`linux_arm64`) containers. Ensure `.terraform.lock.hcl` contains verified `h1:` and `zh:` checksums for all three platforms.

### Step-by-Step Implementation
```bash
# 1. Generate cross-platform checksums in .terraform.lock.hcl:
terraform providers lock \
  -platform=darwin_arm64 \
  -platform=linux_amd64 \
  -platform=linux_arm64

# 2. Verify that .terraform.lock.hcl contains both h1: and zh: entries:
grep -E "h1:|zh:" .terraform.lock.hcl

# 3. Enforce strict lockfile adherence in CI (fail if lockfile would need changes):
terraform init -lockfile=readonly -input=false
```

---

## Lab 2: Two-Stage GitOps Saved Plan & JSON Policy Inspection Pipeline

### Scenario
Build a shell pipeline that creates a deterministic binary plan (`tfplan.binary`), exports it to JSON (`tfplan.json`) for automated policy inspection, verifies that zero resources are scheduled for destruction using `jq`, and applies the exact binary plan.

### Step-by-Step Implementation (`ci_pipeline.sh`)
```bash
#!/usr/bin/env bash
set -euo pipefail

export TF_IN_AUTOMATION=true

# 1. Initialize with readonly lockfile enforcement
terraform init -input=false -lockfile=readonly

# 2. Generate deterministic saved plan
terraform plan -input=false -no-color -out=tfplan.binary

# 3. Convert binary plan to machine-readable JSON
terraform show -json tfplan.binary > tfplan.json

# 4. Automated guardrail: Block apply if any resource has a "delete" action
DELETES=$(jq '[.resource_changes[]? | select(.change.actions[] == "delete")] | length' tfplan.json)
if [ "$DELETES" -gt 0 ]; then
  echo "BLOCKED: Plan contains $DELETES resource deletion(s). Manual override required!"
  exit 1
fi

# 5. Apply exact saved binary plan (no -auto-approve needed)
terraform apply -input=false -no-color tfplan.binary
```

---

## Lab 3: Configuring Air-Gapped Filesystem Provider Mirror

### Scenario
Prepare an offline provider mirror for an air-gapped Linux environment and configure Terraform CLI to install providers strictly from the local mirror without reaching out to `registry.terraform.io`.

### Step-by-Step Implementation
```bash
# 1. On connected staging host: download Linux AMD64 providers into mirror directory
mkdir -p /tmp/offline-tf-providers
terraform providers mirror -platform=linux_amd64 /tmp/offline-tf-providers

# 2. Create air-gapped CLI configuration file (/tmp/airgap.terraformrc):
cat << 'EOT' > /tmp/airgap.terraformrc
provider_installation {
  filesystem_mirror {
    path    = "/tmp/offline-tf-providers"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
EOT

# 3. Run terraform init using only the offline filesystem mirror:
export TF_CLI_CONFIG_FILE="/tmp/airgap.terraformrc"
terraform init -input=false -lockfile=readonly
```
