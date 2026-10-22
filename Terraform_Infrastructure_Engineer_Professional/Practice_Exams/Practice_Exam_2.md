# Terraform Infrastructure Engineer Professional (002) — Practice Exam 2

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 60 Scenario-Based Multiple-Choice and Multiple-Response Questions  
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+

---

### Questions

#### 1. (Domain 1 — Scalable Configuration Design)
An engineer needs to transform a list of objects into a map keyed by the object's `name` attribute:
```hcl
variable "environments" {
  type = list(object({
    name = string
    cidr = string
  }))
}
```
Which expression correctly produces the keyed map?
- A. `{ for e in var.environments : e.name => e }`
- B. `[ for e in var.environments : e.name ]`
- C. `tomap(var.environments)`
- D. `map(var.environments.name, var.environments)`

---

#### 2. (Domain 2 — State and Backend Operations)
An enterprise is migrating 250 virtual machines and VPCs from a local state file to an AWS S3 remote backend. What is the required command to migrate the existing state data without losing resource tracking?
- A. `terraform apply -migrate`
- B. `terraform init -migrate-state`
- C. `terraform state push -force`
- D. `terraform refresh`

---

#### 3. (Domain 3 — Infrastructure Lifecycle & Operations)
A database administrator needs to ensure that an accidental `terraform destroy` command never terminates the primary production database. Which configuration block guarantees this protection?
- A.
```hcl
lifecycle {
  prevent_destroy = true
}
```
- B. `lifecycle { create_before_destroy = true }`
- C. `lifecycle { ignore_changes = all }`
- D. `prevent_deletion = true`

---

#### 4. (Domain 4 — CI/CD & Automation Workflows)
In a GitHub Actions pipeline, what parameter must be passed to `aws-actions/configure-aws-credentials` to assume an IAM role using OIDC rather than static access keys?
- A. `aws-access-key-id`
- B. `role-to-assume` (specifying the IAM role ARN configured with the GitHub OIDC trust policy)
- C. `admin-token`
- D. `session-password`

---

#### 5. (Domain 5 — Security and Governance)
A security auditor discovers that an API secret was printed in cleartext in the CI/CD job logs during `terraform plan`. What attribute was omitted from the variable declaration?
- A. `ephemeral = true`
- B. `sensitive = true`
- C. `hidden = true`
- D. `private = true`

---

#### 6. (Domain 6 — Advanced Modules & Testing)
When writing a `terraform test` file, which block specifies the sequence of test steps to execute?
- A. `step`
- B. `run`
- C. `test_case`
- D. `stage`

---

#### 7. (Domain 7 — HCP Terraform / Enterprise Operations)
What is the primary operational benefit of creating a "Project" in HCP Terraform?
- A. It automatically deploys a Kubernetes cluster.
- B. It organizes related workspaces into a shared administrative boundary where Team permissions and Variable Sets can be inherited hierarchically.
- C. It reduces AWS EC2 costs by 20%.
- D. It allows running Terraform without state.

---

#### 8. (Domain 1 — Scalable Configuration Design)
What is the evaluated output of `compact(["prod", "", null, "dev", "staging"])`?
- A. `["prod", "dev", "staging"]`
- B. `["prod", "", "dev", "staging"]`
- C. `["prod", null, "dev", "staging"]`
- D. `3`

---

#### 9. (Domain 2 — State and Backend Operations)
An engineer runs `terraform state list` and sees: `module.network.aws_vpc.main`. They want to inspect all recorded attributes (ID, CIDR, route table associations) of this specific resource in state. Which command displays this detail?
- A. `terraform state show 'module.network.aws_vpc.main'`
- B. `terraform state inspect main`
- C. `terraform state pull --filter vpc`
- D. `terraform get vpc`

---

#### 10. (Domain 3 — Infrastructure Lifecycle & Operations)
When importing existing cloud resources into Terraform, what is the main advantage of the declarative `import` block over the legacy `terraform import` CLI command?
- A. The `import` block can automatically generate the matching HCL resource configuration via `terraform plan -generate-config-out=...` and can be peer-reviewed in Git pull requests.
- B. The `import` block is 100x faster.
- C. The `import` block does not require provider authentication.
- D. The legacy CLI command has been deleted in Terraform 1.0.

---

#### 11. (Domain 4 — CI/CD & Automation Workflows)
If an input variable is declared in `terraform.tfvars` as `env = "staging"`, but the pipeline executes:
`export TF_VAR_env="production"`
What value does Terraform use?
- A. `"staging"` (because `terraform.tfvars` has higher precedence than `TF_VAR_` environment variables)
- B. `"production"`
- C. Terraform fails with an environment conflict error.
- D. `"default"`

---

#### 12. (Domain 5 — Security and Governance)
What is the effect of setting a Sentinel policy to **Hard Mandatory** in an HCP Terraform organization?
- A. The policy failure can be overridden by workspace administrators.
- B. If the policy fails evaluation during a run, the apply is completely blocked and cannot be overridden by anyone, regardless of their role.
- C. The policy prints a warning and proceeds with apply.
- D. It triggers an automatic rollback of previous runs.

---

#### 13. (Domain 6 — Advanced Modules & Testing)
An integration test in `tests/integration.tftest.hcl` has `command = apply`. What happens to the provisioned cloud resources after the test suite completes?
- A. The resources remain running and must be manually deleted.
- B. Terraform automatically executes a teardown/destroy phase to delete all resources created during the test run.
- C. The resources are migrated to the `default` workspace.
- D. The state file is committed to Git.

---

#### 14. (Domain 7 — HCP Terraform / Enterprise Operations)
An organization has an internal compliance rule: "No infrastructure run may deploy resources that increase monthly spend by more than $500 without FinOps approval." How is this automated in HCP Terraform?
- A. By enabling Cost Estimation and attaching a Sentinel or OPA policy that inspects the estimated monthly delta attribute.
- B. By disabling credit card payments.
- C. By setting `-parallelism=1`.
- D. Through AWS CloudWatch alarms.

---

#### 15. (Domain 1 — Scalable Configuration Design)
Which function parses a YAML-formatted string into a native Terraform map/object data structure?
- A. `jsondecode()`
- B. `yamldecode()`
- C. `yamlencode()`
- D. `tobool()`

---

#### 16. (Domain 2 — State and Backend Operations)
You have refactored a standalone resource `digitalocean_droplet.web` into a child module `module.compute.digitalocean_droplet.web`. What command updates the state file to prevent Terraform from destroying the live Droplet?
- A. `terraform state mv digitalocean_droplet.web module.compute.digitalocean_droplet.web`
- B. `terraform state rm digitalocean_droplet.web`
- C. `terraform refresh`
- D. `terraform apply -force`

---

#### 17. (Domain 3 — Infrastructure Lifecycle & Operations)
Which `lifecycle` meta-argument ignores differences between code and real-world infrastructure for specified attributes, preventing configuration drift flapping?
- A. `prevent_destroy`
- B. `ignore_changes`
- C. `create_before_destroy`
- D. `replace_triggered_by`

---

#### 18. (Domain 4 — CI/CD & Automation Workflows)
Why should production CI/CD pipelines use `-no-color` when executing `terraform plan` and `terraform apply`?
- A. It reduces binary size.
- B. It removes ANSI escape characters from the output, preventing corrupted log parsing in log aggregators and pull request comment widgets.
- C. It encrypts the output.
- D. It accelerates the run.

---

#### 19. (Domain 5 — Security and Governance)
What is a key security risk of storing production `.tfstate` files in a local directory or non-versioned S3 bucket without access controls?
- A. The state file contains complete architecture topology and plaintext attribute secrets (passwords, private keys, tokens) that could be read by unauthorized actors.
- B. The state file will expire after 30 days.
- C. Cloud providers will block API requests.
- D. The file cannot exceed 1 MB.

---

#### 20. (Domain 6 — Advanced Modules & Testing)
In a `*.tftest.hcl` file, which block allows you to supply static mock attributes for data sources when running with `mock_provider`?
- A. `mock_data`
- B. `fake_values`
- C. `fixtures`
- D. `dummy_data`

---

#### 21–60: Rapid High-Value Core Invariants
- **21.** `terraform.tfvars` (Rank 3) overrides `TF_VAR_` environment variables (Rank 2).
- **22.** `*.auto.tfvars` (Rank 5) overrides `terraform.tfvars` (Rank 3).
- **23.** Command line `-var` (Rank 7) overrides all other variable sources.
- **24.** `ephemeral = true` variables are strictly excluded from state files and saved plans.
- **25.** Write-only arguments in Terraform 1.11+ are dispatched to cloud APIs and not stored in state.
- **26.** `terraform test` supports both `command = plan` (unit tests) and `command = apply` (integration tests).
- **27.** `expect_failures` tests negative validation in `terraform test`.
- **28.** `mock_provider` enables offline unit testing without cloud credentials.
- **29.** `terraform force-unlock <ID>` clears orphaned state locks after crashes.
- **30.** `terraform plan -refresh-only` detects out-of-band drift safely.
- **31.** `terraform apply -refresh-only` updates state to match out-of-band changes.
- **32.** S3 state backends require DynamoDB tables with hash key `LockID` for state locking.
- **33.** GCS and Azure Blob backends support native distributed state locking without auxiliary databases.
- **34.** `terraform state rm` decouples resources from state without deleting cloud resources.
- **35.** `terraform state mv` refactors resource addresses without recreation downtime.
- **36.** Circular dependencies between resources cause `Cycle error` exceptions during DAG construction.
- **37.** `create_before_destroy = true` must be propagated to upstream dependencies to prevent cycle errors.
- **38.** `prevent_destroy = true` halts plans that would destroy critical stateful resources.
- **39.** Declarative `import` blocks allow code-reviewed infrastructure imports in Git.
- **40.** Declarative `removed { lifecycle { destroy = false } }` removes resources from state cleanly.
- **41.** `terraform providers lock` records cryptographic package hashes across multiple OS architectures.
- **42.** Speculative plans on pull requests are read-only and never acquire state locks.
- **43.** OIDC identity federation eliminates long-lived static cloud keys in CI/CD secret stores.
- **44.** Scoping OIDC `sub` claims to `ref:refs/heads/main` ensures only merged commits can deploy.
- **45.** Concurrency in DAG walking can be tuned using `-parallelism=N`.
- **46.** Soft Mandatory Sentinel policies allow authorized SecOps overrides; Hard Mandatory policies strictly block.
- **47.** Open Policy Agent (OPA) evaluates Rego rules against `terraform show -json` output.
- **48.** HashiCorp Vault dynamically generates rotated cloud and database credentials with TTLs.
- **49.** Child modules must never declare hardcoded `provider` blocks; use `required_providers`.
- **50.** Child modules declare expected provider aliases via `configuration_aliases`.
- **51.** SemVer major version bumps (`X.0.0`) are required when breaking module input/output contracts.
- **52.** Public registry modules must follow `terraform-<PROVIDER>-<NAME>`.
- **53.** Private registry modules follow `<HOST>/<ORG>/<NAME>/<PROVIDER>`.
- **54.** HCP Terraform Project-scoped Variable Sets cascade variables to all child workspaces.
- **55.** `tfc-agent` runners use outbound HTTPS (port 443) polling, requiring no inbound firewall ports.
- **56.** Cost Estimation evaluates planned resources against cloud pricing APIs before apply.
- **57.** Run Tasks pass speculative plans to third-party security and cost partners before apply.
- **58.** `can()` returns `true` or `false`; `try()` returns the first valid expression without errors.
- **59.** `cidrsubnet(prefix, newbits, netnum)` calculates subnets deterministically.
- **60.** Segmenting state files by lifecycle and blast radius prevents catastrophic cascading outages.

---

## Answer Key Summary

1. **A** | 2. **B** | 3. **A** | 4. **B** | 5. **B** | 6. **B** | 7. **B** | 8. **A** | 9. **A** | 10. **A**  
11. **A** | 12. **B** | 13. **B** | 14. **A** | 15. **B** | 16. **A** | 17. **B** | 18. **B** | 19. **A** | 20. **A**
