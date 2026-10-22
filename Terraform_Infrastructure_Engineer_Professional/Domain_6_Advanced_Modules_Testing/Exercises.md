# Domain 6: Advanced Module Usage and Testing — Production Exercises & Labs

---

## Lab 1: Writing Unit Tests with Mock Providers in `terraform test`

### Objective
Author a test file `tests/droplet_validation.tftest.hcl` for a DigitalOcean Droplet module that uses `mock_provider` to test variable validation and attribute mapping without requiring a DigitalOcean API token or spending cloud budget.

```hcl
# tests/droplet_validation.tftest.hcl

mock_provider "digitalocean" {}

variables {
  droplet_name = "production-app-node"
  region       = "nyc3"
  size         = "s-2vcpu-4gb"
}

run "unit_test_naming_convention" {
  command = plan

  assert {
    condition     = digitalocean_droplet.web.name == "production-app-node"
    error_message = "Droplet name does not match expected variable input."
  }

  assert {
    condition     = digitalocean_droplet.web.region == "nyc3"
    error_message = "Droplet region must be nyc3."
  }
}

run "negative_test_invalid_size" {
  command = plan

  variables {
    size = "invalid-size-xxx"
  }

  # Asserts that the validation rule on var.size correctly fails
  expect_failures = [
    var.size
  ]
}
```

### Execution Command
```bash
terraform test
```
