# Terraform Infrastructure Engineer Professional (002) — Practice Exam 1

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 60 Scenario-Based Multiple-Choice and Multiple-Response Questions  
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+

---

### Questions

#### 1. (Domain 1 — Scalable Configuration Design)
An engineer writes the following variable definition:
```hcl
variable "node_pools" {
  type = map(object({
    size       = string
    count      = number
    autoscale  = optional(bool, false)
  }))
}
```
If a caller supplies:
```hcl
node_pools = {
  "compute" = { size = "s-4vcpu-8gb", count = 5 }
}
```
What is the evaluated value of `var.node_pools["compute"].autoscale`?
- A. `null`
- B. `false`
- C. `true`
- D. Terraform throws a validation error.

---

#### 2. (Domain 2 — State and Backend Operations)
A CI pipeline running `terraform apply` crashes unexpectedly. When a subsequent run starts, it fails with:
`Error: Error acquiring the state lock: ConditionalCheckFailedException: Lock Info: ID: a819b42a-9938-4e12-b102-398418293847`.
After verifying that no other process is actively running, what is the correct command to clear the lock?
- A. `terraform unlock --force`
- B. `terraform force-unlock a819b42a-9938-4e12-b102-398418293847`
- C. `terraform state clear-lock`
- D. `terraform init -reconfigure`

---

#### 3. (Domain 5 — Security and Governance)
In Terraform 1.10+, an engineer wants to ensure that a database master password retrieved dynamically from Vault is never written to disk or recorded in the `.tfstate` JSON file. How should the variable or data source be configured?
- A. By adding `sensitive = true` only.
- B. By adding `ephemeral = true` (or using an `ephemeral` block), which ensures the value resides only in memory during execution and is omitted from the state file.
- C. By encrypting the `.tf` file with PGP.
- D. By passing the password as a command-line flag `-var`.

---

#### 4. (Domain 3 — Infrastructure Lifecycle & Operations)
You need to import an existing DigitalOcean Droplet with ID `394819284` into your Terraform configuration using modern declarative syntax. Which HCL block achieves this?
- A.
```hcl
import {
  to = digitalocean_droplet.web
  id = "394819284"
}
```
- B. `terraform import digitalocean_droplet.web 394819284` in `main.tf`
- C. `data "digitalocean_droplet" "web" { id = "394819284" }`
- D. `resource "import" "droplet" { id = "394819284" }`

---

#### 5. (Domain 4 — CI/CD & Automation Workflows)
Which of the following sources has the absolute highest precedence when evaluating Terraform input variables?
- A. `terraform.tfvars`
- B. `*.auto.tfvars`
- C. `TF_VAR_<name>` environment variable
- D. `-var="name=value"` CLI argument

---

#### 6. (Domain 6 — Advanced Modules & Testing)
In a `*.tftest.hcl` test suite, what does `mock_provider "aws" {}` accomplish?
- A. It creates temporary free-tier AWS resources.
- B. It mocks provider operations in-memory, allowing `command = plan` test runs to validate HCL logic and assertions completely offline without real AWS credentials.
- C. It runs LocalStack in Docker.
- D. It generates mock data in `.tfstate`.

---

#### 7. (Domain 7 — HCP Terraform / Enterprise Operations)
An enterprise needs to execute Terraform plans and applies against private databases inside a dark datacenter with zero inbound internet connectivity. Which HCP Terraform execution mode must be selected?
- A. Remote Execution
- B. Agent Execution (using `tfc-agent`)
- C. Local Execution only
- D. Webhook Execution

---

#### 8. (Domain 1 — Scalable Configuration Design)
An engineer evaluates `cidrsubnet("10.0.0.0/16", 8, 2)`. What is the resulting CIDR block?
- A. `10.0.2.0/24`
- B. `10.0.0.2/24`
- C. `10.0.8.2/24`
- D. `10.0.16.0/24`

---

#### 9. (Domain 2 — State and Backend Operations)
An engineer changes the name of a resource block from `digitalocean_firewall.old_rules` to `digitalocean_firewall.cluster_rules`. If they apply this change without state surgery, what will happen?
- A. Terraform updates the firewall in place.
- B. Terraform destroys the old firewall and provisions a new one.
- C. Terraform generates a cycle error.
- D. The cloud provider blocks the request.

---

#### 10. (Domain 3 — Infrastructure Lifecycle & Operations)
What is the effect of configuring `lifecycle { prevent_destroy = true }` on a database resource?
- A. The database can never be stopped.
- B. Any `terraform destroy` or plan that would result in the resource's destruction is rejected during the planning phase, halting execution with an error.
- C. It enables automated backups.
- D. It creates a read-replica.

---

#### 11. (Domain 4 — CI/CD & Automation Workflows)
How does OpenID Connect (OIDC) enhance security for GitHub Actions pipelines running Terraform against cloud providers?
- A. It automatically writes HCL code.
- B. It eliminates long-lived static API access keys in repository secrets by exchanging signed, short-lived OIDC tokens for temporary cloud credentials.
- C. It bypasses IAM role policies.
- D. It accelerates DNS resolution.

---

#### 12. (Domain 5 — Security and Governance)
Which Sentinel enforcement level halts a run on policy failure, but allows an authorized user (e.g. SecOps Lead) to approve and override the failure?
- A. Advisory
- B. Soft Mandatory
- C. Hard Mandatory
- D. Optional

---

#### 13. (Domain 1 — Scalable Configuration Design)
When configuring a child module that requires an aliased provider from the parent module, where and how must the alias requirement be declared inside the child module?
- A. Inside the child module's `provider` block using `alias = "..."`.
- B. Inside `terraform.required_providers.<provider>.configuration_aliases`.
- C. Inside `locals`.
- D. Child modules cannot use aliased providers.

---

#### 14. (Domain 2 — State and Backend Operations)
An operator manually modified an EC2 instance type from `t3.medium` to `t3.large` in the AWS console. The team agrees to keep this change in state without modifying other resources. What is the modern, safe workflow to update state?
- A. `terraform refresh`
- B. `terraform plan -refresh-only` followed by `terraform apply -refresh-only`
- C. `terraform state rm`
- D. `terraform import`

---

#### 15. (Domain 6 — Advanced Modules & Testing)
What happens if an assertion inside a `terraform test` file fails?
- A. The test continues and prints a notice.
- B. The test command fails with a non-zero exit code, printing the custom `error_message` and assertion details, which fails the CI pipeline.
- C. Terraform retries the test 5 times.
- D. Terraform rolls back Git.

---

#### 16. (Domain 3 — Infrastructure Lifecycle & Operations)
Which `lifecycle` meta-argument ensures that a replacement virtual machine is fully provisioned and healthy before the existing virtual machine is terminated?
- A. `replace_triggered_by`
- B. `create_before_destroy = true`
- C. `prevent_destroy = true`
- D. `ignore_changes = all`

---

#### 17. (Domain 5 — Security and Governance)
A Rego policy for Open Policy Agent (OPA) evaluates which data format generated by Terraform?
- A. Binary `.tfstate` files.
- B. JSON plan representation generated via `terraform show -json <planfile>`.
- C. Raw `.tf` text files.
- D. System syslog files.

---

#### 18. (Domain 7 — HCP Terraform / Enterprise Operations)
Where are shared variables applied when scoped to an HCP Terraform "Project"?
- A. To all workspaces across all organizations in the account.
- B. Automatically to all current and future workspaces residing inside that specific Project.
- C. Only to workspaces created by administrators.
- D. Only to local state files.

---

#### 19. (Domain 1 — Scalable Configuration Design)
What is the role of the Terraform Model Context Protocol (MCP) server introduced in Terraform 1.15+?
- A. It deploys Kubernetes pods.
- B. It provides standardized JSON-RPC schemas, documentation, and validation hooks to local AI coding agents.
- C. It replaces cloud providers with AI simulators.
- D. It compiles HCL to WebAssembly.

---

#### 20. (Domain 2 — State and Backend Operations)
Which command removes a resource from the Terraform state file without issuing an API call to delete the real cloud resource?
- A. `terraform destroy -target`
- B. `terraform state rm`
- C. `terraform state drop`
- D. `terraform delete`

---

#### 21. (Domain 4 — CI/CD & Automation Workflows)
Two files exist in the same directory: `01-dev.auto.tfvars` setting `port = 8080`, and `99-prod.auto.tfvars` setting `port = 443`. What is the evaluated value of `var.port`?
- A. `8080`
- B. `443` (because files matching `*.auto.tfvars` are loaded in alphabetical order, later files override earlier files)
- C. Terraform fails with a duplicate variable error.
- D. `null`

---

#### 22. (Domain 3 — Infrastructure Lifecycle & Operations)
What does the declarative `removed` block do when configured with `destroy = false`?
- A. It deletes the resource from the cloud.
- B. It removes the resource from `.tfstate` while leaving the real cloud resource running.
- C. It locks the resource against future edits.
- D. It restarts the virtual machine.

---

#### 23. (Domain 6 — Advanced Modules & Testing)
Under Semantic Versioning, what version bump is required when you remove an input variable or rename an output in a shared Terraform module?
- A. Patch (`v1.0.1`)
- B. Minor (`v1.1.0`)
- C. Major (`v2.0.0`)
- D. Build metadata (`v1.0.0+build1`)

---

#### 24. (Domain 7 — HCP Terraform / Enterprise Operations)
What is the function of a "Run Task" in HCP Terraform?
- A. It runs a shell script on developer laptops.
- B. It integrates external third-party security, compliance, or cost tools (e.g. Snyk, Infracost) directly into the plan/apply lifecycle.
- C. It reboots the HCP server.
- D. It converts Terraform code into CloudFormation.

---

#### 25. (Domain 1 — Scalable Configuration Design)
Which function returns `true` if an expression can be evaluated without errors, and `false` otherwise?
- A. `try()`
- B. `can()`
- C. `coalesce()`
- D. `validate()`

---

#### 26. (Domain 2 — State and Backend Operations)
What happens if two engineers simultaneously run `terraform apply` against a backend that supports distributed locking (e.g. S3 with DynamoDB)?
- A. Both applies merge their changes.
- B. The first process acquires the lock; the second process fails immediately with a lock error or waits until the lock is released.
- C. The state file is deleted.
- D. The cloud provider pauses both accounts.

---

#### 27. (Domain 5 — Security and Governance)
Why is running Terraform with AWS `AdministratorAccess` considered a major security risk?
- A. It increases AWS billing rates.
- B. If CI runner credentials or OIDC claims are compromised, attackers gain unrestricted root-equivalent access to the entire cloud organization.
- C. Terraform takes longer to initialize.
- D. AWS restricts AdministratorAccess to 1 hour.

---

#### 28. (Domain 3 — Infrastructure Lifecycle & Operations)
When executing `terraform plan -out=tfplan`, what is contained inside the binary `tfplan` file?
- A. Only the Git commit hash.
- B. The exact planned actions, resource attributes, configuration snapshot, and state dependencies computed at plan time.
- C. An encrypted backup of the cloud database.
- D. Plaintext passwords.

---

#### 29. (Domain 4 — CI/CD & Automation Workflows)
Why should `terraform apply` in an automated production pipeline specify the pre-generated plan file (`terraform apply tfplan`)?
- A. It prevents Terraform from re-evaluating external variables or picking up concurrent out-of-band drift between the review step and apply step.
- B. It reduces network bandwidth.
- C. It runs without an internet connection.
- D. It is required by GitHub.

---

#### 30. (Domain 6 — Advanced Modules & Testing)
How do you assert that a specific variable validation rule fails as expected in a `terraform test` file?
- A. By adding an `expect_failures = [var.<variable_name>]` block inside the `run` block.
- B. By wrapping the test in a `try()` block.
- C. By writing a Python script.
- D. By checking `dmesg`.

---

#### 31–60: Rapid High-Value Core Invariants
- **31.** `try(a, b)` returns `a` if valid, otherwise `b`.
- **32.** S3 state buckets require bucket versioning and default encryption for disaster recovery.
- **33.** `terraform state mv` updates HCL addresses in state without cloud resource recreation.
- **34.** `ignore_changes = [desired_capacity]` stops autoscaler drift flapping.
- **35.** HCP Terraform Variable Sets can be Global, Project-scoped, or Workspace-scoped.
- **36.** `ephemeral` variables vanish from state entirely in Terraform 1.10+.
- **37.** Reusable modules must NEVER declare hardcoded `provider` blocks.
- **38.** `terraform fmt -check` exits non-zero if files require canonical formatting.
- **39.** Cyclic dependencies trigger a `Cycle error` during graph construction.
- **40.** `terraform providers lock` downloads hashes for multiple target OS platforms.
- **41.** `replace_triggered_by` forces resource replacement when another resource changes.
- **42.** Outbound HTTPS (port 443) is the only network requirement for `tfc-agent`.
- **43.** Cost Estimation calculates estimated spend changes between plan and apply.
- **44.** `can(regex(...))` validates string formats safely without unhandled exceptions.
- **45.** `terraform state pull` outputs the current remote state JSON to stdout.
- **46.** Speculative plans on pull requests are read-only and never hold locks.
- **47.** Concurrency in graph execution can be tuned using `-parallelism=N`.
- **48.** Hard Mandatory Sentinel policies cannot be overridden by anyone.
- **49.** A module's public contract is defined by its variables and outputs.
- **50.** `compact()` removes `null` and empty strings `""` from string lists.
- **51.** `flatten()` turns multi-dimensional lists into a single flat list.
- **52.** Dynamic blocks iterate using `for_each` and expose attributes via `<name>.value`.
- **53.** Preconditions validate inputs prior to execution; postconditions validate API outputs.
- **54.** `-generate-config-out=PATH` synthesizes HCL for declarative `import` blocks.
- **55.** OIDC JWT claims should be scoped strictly to `refs/heads/main`.
- **56.** Public modules must be named `terraform-<PROVIDER>-<NAME>`.
- **57.** Audit Log Streaming exports HCP Terraform audit events to SIEMs.
- **58.** Write-only arguments in Terraform 1.11+ bypass state persistence entirely.
- **59.** `terraform test` automatically tears down resources created with `command = apply`.
- **60.** Decoupling state files by failure domain minimizes operational blast radius.

---

## Answer Key Summary

1. **B** | 2. **B** | 3. **B** | 4. **A** | 5. **D** | 6. **B** | 7. **B** | 8. **A** | 9. **B** | 10. **B**  
11. **B** | 12. **B** | 13. **B** | 14. **B** | 15. **B** | 16. **B** | 17. **B** | 18. **B** | 19. **B** | 20. **B**  
21. **B** | 22. **B** | 23. **C** | 24. **B** | 25. **B** | 26. **B** | 27. **B** | 28. **B** | 29. **A** | 30. **A**
