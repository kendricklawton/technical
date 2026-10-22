# Domain 1: Scalable Configuration Design — Production Exercises & Labs

---

## Lab 1: Refactoring Fragile `count` to Robust `for_each` Without Downtime

### Objective
A legacy production module deploys 3 DigitalOcean Droplets using `count = length(var.droplet_names)`. If `var.droplet_names[1]` is deleted, `count` will destroy and recreate the 3rd Droplet. Refactor to `for_each` and migrate the Terraform state safely without destroying running instances.

### Scenario Code (Before Refactoring)
```hcl
variable "droplet_names" {
  type    = list(string)
  default = ["k8s-cp1", "k8s-worker1", "k8s-worker2"]
}

resource "digitalocean_droplet" "nodes" {
  count  = length(var.droplet_names)
  name   = var.droplet_names[count.index]
  region = "nyc3"
  size   = "s-2vcpu-4gb"
  image  = "ubuntu-24-04-x64"
}
```

### Refactored Code (Using `for_each`)
```hcl
variable "droplets" {
  type = map(object({
    size = string
  }))
  default = {
    "k8s-cp1"     = { size = "s-2vcpu-4gb" }
    "k8s-worker1" = { size = "s-2vcpu-4gb" }
    "k8s-worker2" = { size = "s-2vcpu-4gb" }
  }
}

resource "digitalocean_droplet" "nodes" {
  for_each = var.droplets
  name     = each.key
  region   = "nyc3"
  size     = each.value.size
  image    = "ubuntu-24-04-x64"
}
```

### State Migration Commands
Execute state surgery to map the array indexes to map keys:
```bash
terraform state mv 'digitalocean_droplet.nodes[0]' 'digitalocean_droplet.nodes["k8s-cp1"]'
terraform state mv 'digitalocean_droplet.nodes[1]' 'digitalocean_droplet.nodes["k8s-worker1"]'
terraform state mv 'digitalocean_droplet.nodes[2]' 'digitalocean_droplet.nodes["k8s-worker2"]'

# Verify plan shows 0 to add, 0 to change, 0 to destroy!
terraform plan
```

---

## Lab 2: Multi-Region Aliased Provider Configuration with Child Modules

### Objective
Configure a root module with two AWS provider instances (`us-east-1` and `eu-west-1`) and instantiate a child module requiring both providers using `configuration_aliases`.

```hcl
# main.tf
terraform {
  required_version = ">= 1.15.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "europe"
  region = "eu-west-1"
}

module "cross_region_vault" {
  source = "./modules/cross_region_vault"

  providers = {
    aws.primary = aws
    aws.dr      = aws.europe
  }
}
```
