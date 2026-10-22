# Terraform Infrastructure Engineer Professional (002) — Practice Exam 3

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 25 Comprehensive Scenario-Based Questions Covering All 7 Domains with In-Depth Technical Explanations  
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+ / HCP Terraform

---

### Questions

#### 1. (Domain 1 — Scalable Configuration Design)
An enterprise module receives a variable `subnets` containing a map of objects with CIDRs and AZs. You need to construct a filtered list of only the subnet IDs located in availability zone `"us-east-1a"`. Which HCL `for` expression is syntactically correct?
- A. `[ for id, subnet in var.subnets : id if subnet.az == "us-east-1a" ]`
- B. `{ for id, subnet in var.subnets : id if subnet.az == "us-east-1a" }`
- C. `filter(var.subnets, "az == us-east-1a")`
- D. `[ select id from var.subnets where az == "us-east-1a" ]`

---

#### 2. (Domain 2 — State & Backend Operations)
During a team restructuring, an engineer splits a monolithic configuration into two independent workspaces: `networking` and `compute`. To migrate the VPC resources from the monolithic state file to the new `networking` state file without destroying cloud infrastructure, what sequence of state commands should be used?
- A. `terraform destroy` in monolith, followed by `terraform apply` in networking.
- B. Run `terraform state mv -state-out=../networking/terraform.tfstate module.vpc module.vpc` or extract and import the state.
- C. Edit `terraform.tfstate` directly using `vim`.
- D. `terraform refresh -force`

---

#### 3. (Domain 3 — Lifecycle Operations)
An engineer modifies an `aws_iam_role` resource name from `"api-role-v1"` to `"api-role-v2"`. IAM roles have globally unique names within an AWS account. The resource block includes `lifecycle { create_before_destroy = true }`. What will happen when `terraform apply` executes?
- A. Terraform creates `"api-role-v2"` successfully and then destroys `"api-role-v1"`.
- B. The apply fails because `"api-role-v2"` has a different name, so name collision does not occur.
- C. Terraform fails because AWS IAM does not permit two roles with the same name, but since the names differ, creation succeeds before the old role is destroyed.
- D. Terraform deletes the entire AWS account.

---

#### 4. (Domain 4 — CI/CD Automation Workflows)
In an automated GitHub Actions pipeline, `terraform plan -out=tfplan` runs on pull requests, and `terraform apply tfplan` runs on merge to `main`. A developer merges a pull request 48 hours after opening it. In the meantime, another developer merged a PR that provisioned an S3 bucket with the identical name. What happens when the apply job executes?
- A. Terraform automatically renames the new bucket.
- B. The apply job fails because the cloud provider API returns a 409 Conflict / BucketAlreadyExists error, preserving state integrity without overwriting the existing bucket.
- C. The existing bucket is silently overwritten and deleted.
- D. The GitHub runner reboots.

---

#### 5. (Domain 5 — Security & Governance)
You need to prevent developers from provisioning security groups that open port `22` (SSH) to `0.0.0.0/0`. In HashiCorp Sentinel, which rule definition correctly evaluates this condition across all planned security group rules?
- A.
```sentinel
import "tfplan/v2" as tfplan

violating_rules = filter tfplan.resource_changes as _, rc {
  rc.type is "aws_security_group_rule" and
  rc.change.after.from_port <= 22 and
  rc.change.after.to_port >= 22 and
  "0.0.0.0/0" in rc.change.after.cidr_blocks
}
main = rule { length(violating_rules) is 0 }
```
- B.
```sentinel
main = rule { aws_security_group.port == 22 }
```
- C.
```sentinel
deny { port: 22 }
```
- D.
```sentinel
rule { block_all_ssh() }
```

---

#### 6. (Domain 6 — Advanced Modules & Testing)
In a `.tftest.hcl` file, you want to test whether a module properly creates 3 subnets when given a list of 3 CIDRs during `terraform test`. Which assertion condition verifies this?
- A. `condition = length(output.subnet_ids) == 3`
- B. `condition = count(output.subnet_ids) == 3`
- C. `condition = size(output.subnet_ids) == 3`
- D. `expect = 3`

---

#### 7. (Domain 7 — HCP Terraform & Enterprise)
What is the purpose of an HCP Terraform `OIDC Run Token`?
- A. It authenticates users to the web console.
- B. It is a short-lived JSON Web Token (JWT) injected into the execution container that allows the Terraform CLI to assume cloud provider IAM roles dynamically without long-lived static API secrets.
- C. It encrypts the state file with PGP.
- D. It acts as an SSH key for git checkout.

---

#### 8. (Domain 1 — Scalable Configuration Design)
You have a variable `cluster_nodes` with default `null`. You want to assign `local.node_count` to `var.cluster_nodes` if provided, or fallback to `3` if `null`. Which HCL function accomplishes this cleanly?
- A. `coalesce(var.cluster_nodes, 3)`
- B. `default(var.cluster_nodes, 3)`
- C. `fallback(var.cluster_nodes, 3)`
- D. `ternary(var.cluster_nodes, 3)`

---

#### 9. (Domain 2 — State & Backend Operations)
An engineer runs `terraform state list` and notices that an obsolete resource `aws_instance.old_bastion` is still tracked in state, but the physical VM was terminated manually in the cloud console weeks ago. How can state be synchronized without executing a full apply?
- A. `terraform apply -auto-approve`
- B. `terraform apply -refresh-only`
- C. `terraform state delete aws_instance.old_bastion`
- D. `terraform destroy`

---

#### 10. (Domain 3 — Lifecycle Operations)
In Terraform v1.5+, what is the difference between a `precondition` and a `postcondition` in a resource lifecycle block?
- A. `precondition` runs before `terraform init`; `postcondition` runs after `terraform apply`.
- B. `precondition` evaluates assumptions about inputs and configuration before creating/updating the resource; `postcondition` evaluates guarantees about the resource's resulting state after creation (such as verifying a public IP was assigned).
- C. `postcondition` can only be used on data sources.
- D. `precondition` cannot reference other resources.

---

#### 11. (Domain 4 — CI/CD Automation Workflows)
Why should `.terraform.lock.hcl` be committed to version control in production repositories?
- A. It locks remote state in S3.
- B. It records exact provider versions and cryptographic checksums, guaranteeing deterministic provider binaries across developer workstations and CI runners.
- C. It stores cloud provider passwords.
- D. It prevents Git merge conflicts.

---

#### 12. (Domain 5 — Security & Governance)
What is the primary operational advantage of using Checkov or tfsec over Sentinel in open-source CI/CD workflows?
- A. Checkov runs 100x faster than Sentinel.
- B. Checkov and tfsec are open-source and run locally in any standard CI pipeline without requiring an enterprise HCP Terraform license.
- C. Sentinel cannot inspect plan files.
- D. Checkov automatically edits HCL files.

---

#### 13. (Domain 6 — Advanced Modules & Testing)
When creating a test file `main.tftest.hcl`, you want to mock the AWS provider so tests can execute in CI without AWS credentials. How is this defined in Terraform v1.7+?
- A.
```hcl
mock_provider "aws" {}
```
- B.
```hcl
provider "aws" { mock = true }
```
- C.
```hcl
override_provider "aws" {}
```
- D.
```hcl
fake_provider "aws" {}
```

---

#### 14. (Domain 7 — HCP Terraform & Enterprise)
What is the primary function of HCP Terraform `Variable Sets`?
- A. To enforce naming conventions on variables.
- B. To define a centralized bundle of Terraform and environment variables once and apply them globally or to selected projects and workspaces.
- C. To encrypt code in Git.
- D. To purchase cloud discounts.

---

#### 15. (Domain 1 — Scalable Configuration Design)
What happens if you define a local value:
```hcl
locals {
  service_names = toset(["auth", "billing", "auth"])
}
```
What is `length(local.service_names)`?
- A. 3
- B. 2 (sets eliminate duplicates)
- C. 1
- D. Terraform throws a duplicate key error.

---

#### 16. (Domain 2 — State & Backend Operations)
An organization has an S3 state backend. To ensure maximum durability and disaster recovery capability, which three S3 features must be enabled on the state bucket?
- A. S3 Versioning, Server-Side Encryption (SSE-KMS), and Object Lock (or MFA Delete).
- B. Public read access, Website hosting, and CORS.
- C. S3 Glacier Direct Ingestion, BitTorrent, and Transfer Acceleration.
- D. Static IP hosting, Requester Pays, and CloudFront.

---

#### 17. (Domain 3 — Lifecycle Operations)
You need to ensure that an EC2 instance is replaced whenever a user-data bash script file on disk changes. Which HCL configuration achieves this in modern Terraform?
- A.
```hcl
lifecycle {
  replace_triggered_by = [
    terraform_data.bootstrap.output
  ]
}
```
- B.
```hcl
lifecycle {
  taint_on_change = true
}
```
- C.
```hcl
lifecycle {
  recreate = file("script.sh")
}
```
- D.
```hcl
lifecycle {
  destroy_on_update = true
}
```

---

#### 18. (Domain 4 — CI/CD Automation Workflows)
In Atlantis, what command is posted as a pull request comment to execute the approved plan against infrastructure?
- A. `atlantis apply`
- B. `atlantis run`
- C. `atlantis execute`
- D. `atlantis merge`

---

#### 19. (Domain 5 — Security & Governance)
What is the function of the `ephemeral` keyword in Terraform v1.10+?
- A. It creates temporary spot VMs.
- B. It declares ephemeral values and resources that exist purely in memory during execution and are never persisted to the state file on disk or remote backends.
- C. It deletes resources after 24 hours.
- D. It encrypts Git commits.

---

#### 20. (Domain 6 — Advanced Modules & Testing)
In a module `README.md`, which tool automatically generates the inputs, outputs, requirements, and provider tables from HCL source code?
- A. `terraform-docs`
- B. `tflint`
- C. `terragrunt`
- D. `terratest`

---

#### 21. (Domain 7 — HCP Terraform & Enterprise)
What is the primary architectural purpose of deploying an `HCP Terraform Agent` in a corporate network?
- A. To replace the local Docker daemon.
- B. To execute Terraform runs against private cloud networks and on-premises data centers without opening inbound firewall ports from the Internet.
- C. To back up local state to USB drives.
- D. To monitor employee web traffic.

---

#### 22. (Domain 1 — Scalable Configuration Design)
Which function removes `null` or empty string elements from a list?
- A. `compact()`
- B. `distinct()`
- C. `flatten()`
- D. `slice()`

---

#### 23. (Domain 2 — State & Backend Operations)
When migrating state from a local backend to an S3 backend using `terraform init -migrate-state`, what happens to the existing local `terraform.tfstate` file?
- A. It is permanently deleted immediately.
- B. Terraform copies the state to S3 and preserves or creates a backup copy locally, leaving the local state empty or backed up as `terraform.tfstate.backup`.
- C. It is committed to Git.
- D. The cloud provider rejects the migration.

---

#### 24. (Domain 3 — Lifecycle Operations)
What does `prevent_destroy = true` do when placed inside a resource's `lifecycle` block?
- A. It prevents anyone from deleting the Git branch.
- B. It causes Terraform to reject any plan that would destroy the physical infrastructure resource, acting as a critical safety guardrail against accidental deletion.
- C. It makes the resource read-only in the cloud console.
- D. It disables cloud billing for that resource.

---

#### 25. (Domain 5 — Security & Governance)
Why is hardcoding secrets inside `terraform.tfvars` considered a severe security risk even if the repository is private?
- A. `terraform.tfvars` files are easily leaked, committed accidentally, and stored in unencrypted Git history, exposing credentials to all repository viewers.
- B. Terraform CLI throws a compilation error.
- C. Cloud providers disable API keys if found in `.tfvars`.
- D. The Terraform binary crashes.

---

## Answer Key & Rationales

1. **A** — `[ for id, subnet in var.subnets : id if subnet.az == "us-east-1a" ]` filters map elements based on a condition and returns a list of matching keys.
2. **B** — `terraform state mv` allows moving resource state between distinct state files without touching physical infrastructure.
3. **A** — With `create_before_destroy = true` and different resource names, Terraform safely provisions the replacement before terminating the original.
4. **B** — Plan files capture intent, but execution fails safely if remote resources collide, preventing state corruption.
5. **A** — Sentinel filters `resource_changes` in `tfplan/v2` and verifies no rule exposes port 22 to `0.0.0.0/0`.
6. **A** — `condition = length(output.subnet_ids) == 3` asserts that the output list contains exactly 3 elements.
7. **B** — HCP Terraform OIDC tokens allow ephemeral authentication to AWS STS, Azure AD, and GCP Workload Identity.
8. **A** — `coalesce()` evaluates arguments sequentially and returns the first non-null/non-empty value.
9. **B** — `terraform apply -refresh-only` reconciles state with external manual deletions without applying configuration diffs.
10. **B** — Preconditions validate inputs before resource evaluation; postconditions validate attributes after creation.
11. **B** — `.terraform.lock.hcl` pins provider versions and platform hashes for supply chain immutability.
12. **B** — Checkov and tfsec are open-source and run locally in any CI/CD runner.
13. **A** — `mock_provider "aws" {}` enables unit testing without real credentials.
14. **B** — Variable Sets enable centralized reuse of variable groups across workspaces and projects.
15. **B** — Sets store unique elements; duplicate `"auth"` is deduplicated, leaving 2 items.
16. **A** — S3 versioning, SSE-KMS, and Object Lock provide durability and immutability for state files.
17. **A** — `replace_triggered_by` combined with `terraform_data` replaces resources when inputs or files change.
18. **A** — In Atlantis, commenting `atlantis apply` executes the reviewed plan.
19. **B** — Ephemeral values in Terraform 1.10+ prevent sensitive secrets from being stored in the state file.
20. **A** — `terraform-docs` generates standard Markdown interface tables from HCL ASTs.
21. **B** — HCP Terraform Agents poll outbound over HTTPS, executing workloads in private networks without inbound holes.
22. **A** — `compact()` removes empty strings from a list of strings.
23. **B** — `init -migrate-state` transfers state to the remote store while backing up local state.
24. **B** — `prevent_destroy = true` aborts any plan proposing resource destruction.
25. **A** — Storing plaintext secrets in files risks accidental commit and exposes credentials in version history.
