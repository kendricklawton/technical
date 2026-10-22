# Domain 5: Terraform Security and Governance — Production Exercises & Labs

---

## Lab 1: Zero-State Leakage with Ephemeral Variables

### Objective
Configure a database instance where the administrator password is provided via an `ephemeral` variable. Apply the configuration and inspect the resulting `.tfstate` JSON file to mathematically prove that the password was never written to disk or state.

```hcl
variable "db_password" {
  type      = string
  sensitive = true
  ephemeral = true # Prevents persistence to .tfstate
}

resource "digitalocean_database_cluster" "app_db" {
  name       = "secure-db-cluster"
  engine     = "pg"
  version    = "16"
  size       = "db-s-1vcpu-1gb"
  region     = "nyc3"
  node_count = 1
}

# In older versions, sensitive variables leak to state.
# In Terraform 1.10+, ephemeral variables vanish from state entirely!
```

### Verification Command
```bash
# Pull the state and grep for the password string:
terraform state pull | grep -i "${TF_VAR_db_password}"
# Output must be empty!
```

---

## Lab 2: Authoring and Enforcing an OPA/Rego Policy Against Public SSH

### Objective
Write an Open Policy Agent (OPA) rule that inspects `terraform show -json` output and fails the pipeline if any security group allows inbound TCP port 22 from `0.0.0.0/0`.

```rego
# policy/no_public_ssh.rego
package terraform.security

default allow = false

# Find all rules opening port 22 to the world
violations[msg] {
    some resource in input.resource_changes
    resource.type == "digitalocean_firewall"
    some rule in resource.change.after.inbound_rule
    rule.protocol == "tcp"
    rule.port_range == "22"
    some cidr in rule.source_addresses
    cidr == "0.0.0.0/0"
    msg := sprintf("Firewall '%v' allows public SSH (0.0.0.0/0)!", [resource.name])
}

allow {
    count(violations) == 0
}
```

### Execution Steps
```bash
# 1. Output JSON plan:
terraform plan -out=tfplan
terraform show -json tfplan > plan.json

# 2. Evaluate with OPA CLI:
opa eval --data policy/no_public_ssh.rego --input plan.json "data.terraform.security.allow"
```
