# Domain 3: Infrastructure Lifecycle and Operations — Production Exercises & Labs

---

## Lab 1: Declarative Infrastructure Import and Code Generation

### Objective
An unmanaged DigitalOcean VPC exists with ID `do-vpc-prod-01`. Import it into a clean Terraform configuration using a declarative `import` block, generate the HCL automatically, and verify a zero-diff plan.

### Steps
1. Create `import.tf`:
```hcl
import {
  to = digitalocean_vpc.imported_vpc
  id = "do-vpc-prod-01"
}
```

2. Run automated configuration generation:
```bash
terraform plan -generate-config-out=generated_vpc.tf
```

3. Review `generated_vpc.tf`, adjust formatting and variable interpolations.
4. Execute `terraform apply` to bind the cloud resource to the generated HCL.
5. Verify `terraform plan` reports: `No changes. Your infrastructure matches the configuration.`

---

## Lab 2: Zero-Downtime Rolling Certificate & Ingress Update

### Objective
Update a TLS certificate and reverse-proxy instance using `lifecycle { create_before_destroy = true }` to guarantee that the new instance is serving traffic before the legacy instance is terminated.

```hcl
resource "digitalocean_droplet" "ingress_gateway" {
  name   = "ingress-gateway-v2"
  region = "nyc3"
  size   = "s-2vcpu-4gb"
  image  = "ubuntu-24-04-x64"

  lifecycle {
    create_before_destroy = true
  }
}
```
