# Domain 5: Security, Governance & Policy-as-Code — Hands-On Lab Exercises

## Lab 1: End-to-End Zero-State Secret Rotation with `ephemeral` & Write-Only (`_wo`) Attributes

### Scenario
Configure a Vault KV-v2 ephemeral secret lookup and pass it into an AWS Secrets Manager secret version using `secret_string_wo` and `secret_string_wo_version` so that neither the Vault token nor the secret payload ever touches `terraform.tfstate`.

### Step-by-Step Implementation (`zero_state_secrets.tf`)
```hcl
variable "vault_ephemeral_token" {
  type      = string
  sensitive = true
  ephemeral = true
}

variable "secret_version_counter" {
  type    = number
  default = 1
}

provider "vault" {
  address = "https://vault.internal.corp:8200"
  token   = var.vault_ephemeral_token
}

ephemeral "vault_kv_secret_v2" "api_signing_key" {
  mount = "secret"
  name  = "prod/payments/stripe-webhook-key"
}

resource "aws_secretsmanager_secret" "webhook" {
  name = "prod/payments/webhook-signing-key"
}

resource "aws_secretsmanager_secret_version" "webhook_ver" {
  secret_id                = aws_secretsmanager_secret.webhook.id
  secret_string_wo         = jsonencode(ephemeral.vault_kv_secret_v2.api_signing_key.data)
  secret_string_wo_version = var.secret_version_counter
}
```

---

## Lab 2: Authoring & Testing an OPA Rego Policy Against `tfplan.json`

### Scenario
Write an OPA Rego policy that rejects any Terraform plan attempting to create an AWS Security Group ingress rule allowing CIDR `0.0.0.0/0` on port `22` (SSH), and evaluate it locally using `opa exec` / `opa eval`.

### Step-by-Step Implementation (`policy/no_public_ssh.rego`)
```rego
package terraform.security

import rego.v1

deny contains reason if {
  some rc in input.resource_changes
  rc.type == "aws_vpc_security_group_ingress_rule"
  "create" in rc.change.actions
  rc.change.after.cidr_ipv4 == "0.0.0.0/0"
  rc.change.after.from_port <= 22
  rc.change.after.to_port >= 22
  reason := sprintf("CRITICAL: %s opens SSH (port 22) to 0.0.0.0/0!", [rc.address])
}
```
```bash
# Evaluate Rego policy against exported Terraform JSON plan:
terraform show -json tfplan.binary > tfplan.json
opa eval --input tfplan.json --data policy/no_public_ssh.rego "data.terraform.security.deny" --fail-defined
```
