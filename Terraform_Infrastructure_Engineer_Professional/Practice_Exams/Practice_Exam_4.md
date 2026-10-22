# Terraform Infrastructure Engineer Professional (002) — Practice Exam 4

**Exam Code:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Time Limit:** 90 Minutes  
**Format:** 25 Comprehensive Scenario-Based Questions Covering All 7 Domains with In-Depth Technical Explanations  
**Passing Score:** ~70% (Target: $\ge 80\%$)  
**Target Engine:** Terraform v1.15+ / HCP Terraform

---

### Questions

#### 1. (Domain 1 — Scalable Configuration Design)
An enterprise module requires an input variable `cidr_block` to be a valid IPv4 CIDR string and within the private `10.0.0.0/8` range. Which variable validation expression is correct?
- A.
```hcl
variable "cidr_block" {
  type = string
  validation {
    condition     = can(cidrnetmask(var.cidr_block)) && startswith(var.cidr_block, "10.")
    error_message = "Must be a valid IPv4 CIDR within 10.0.0.0/8"
  }
}
```
- B.
```hcl
validation = var.cidr_block in "10.0.0.0/8"
```
- C.
```hcl
check = is_cidr(var.cidr_block)
```
- D.
```hcl
assert { ip == "10.*" }
```

---

#### 2. (Domain 2 — State & Backend Operations)
During a team state inspection, an engineer runs `terraform state show aws_security_group.web` and discovers that a database password was written to state in plaintext. What is the most immediate security remediation step?
- A. Delete the entire cloud VPC.
- B. Rotate the database credential immediately upstream in the database and secrets manager, and ensure the Terraform configuration is updated to use dynamic or ephemeral credentials.
- C. Run `terraform destroy`.
- D. Git commit the state file.

---

#### 3. (Domain 3 — Lifecycle Operations)
You have a resource with `lifecycle { ignore_changes = [tags["CostCenter"]] }`. A developer updates the resource tag `"Environment"` from `"dev"` to `"staging"` in the Terraform manifest, while an external AWS billing script updates the tag `"CostCenter"` from `"100"` to `"200"` in the AWS console. What will `terraform plan` propose?
- A. It proposes updating `"Environment"` to `"staging"` and does NOT propose changing `"CostCenter"`.
- B. It proposes reverting both tags.
- C. It throws a syntax error because map key indexing in `ignore_changes` is not allowed.
- D. It destroys the resource.

---

#### 4. (Domain 4 — CI/CD Automation Workflows)
In a secure automated CI/CD pipeline, how should provider binaries be cached on the runner to reduce network bandwidth and prevent rate limiting from HashiCorp Registry?
- A. By copying provider binaries into Git.
- B. By configuring a provider plugin cache directory using the environment variable `TF_PLUGIN_CACHE_DIR=/opt/terraform/plugin-cache`.
- C. By running `terraform init` twice.
- D. By disabling SSL verification.

---

#### 5. (Domain 5 — Security & Governance)
You need to enforce that all AWS S3 buckets created across your organization have AWS KMS encryption enabled (`aws:kms`). In Open Policy Agent (OPA) with Rego, how is this rule expressed against the `tfplan.json` output?
- A.
```rego
package terraform.security

deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_s3_bucket_server_side_encryption_configuration"
    rule := resource.change.after.rule[_]
    rule.apply_server_side_encryption_by_default.sse_algorithm != "aws:kms"
    msg := sprintf("S3 bucket %v must use aws:kms encryption", [resource.address])
}
```
- B.
```rego
allow = true
```
- C.
```rego
rule = "s3_kms_only"
```
- D.
```rego
package main
deny = true
```

---

#### 6. (Domain 6 — Advanced Modules & Testing)
In a `.tftest.hcl` test file, what occurs if a `run` block does NOT specify `command`?
- A. The test runner aborts with an error.
- B. It defaults to `command = apply`.
- C. It defaults to `command = plan`.
- D. It skips the run block.

---

#### 7. (Domain 7 — HCP Terraform & Enterprise)
What is the effect of an HCP Terraform workspace setting `Auto-apply: true`?
- A. The workspace automatically fixes Terraform syntax errors.
- B. Once a VCS-driven plan completes successfully and all Sentinel / OPA policy checks pass, the changes are applied immediately without requiring manual human confirmation.
- C. The workspace applies changes even if policies fail.
- D. The workspace pushes code to GitHub.

---

#### 8. (Domain 1 — Scalable Configuration Design)
You have a list of numbers: `[10, 20, 30]`. You need to calculate the sum of all elements. Which HCL expression accomplishes this?
- A. `sum([10, 20, 30])`
- B. `add([10, 20, 30])`
- C. `total([10, 20, 30])`
- D. `math.sum([10, 20, 30])`

---

#### 9. (Domain 2 — State & Backend Operations)
In Terraform v1.5+, what command automatically generates HCL configuration files from resources imported using `import` blocks?
- A. `terraform plan -generate-config-out=generated.tf`
- B. `terraform import -export=hcl`
- C. `terraform generate`
- D. `terraform hcl-convert`

---

#### 10. (Domain 3 — Lifecycle Operations)
What happens if you declare two resources with identical identifiers in the same configuration, e.g.:
```hcl
resource "aws_s3_bucket" "data" {}
resource "aws_s3_bucket" "data" {}
```
- A. Terraform merges the two buckets.
- B. Terraform fails during initial compilation with a duplicate resource definition error before generating a plan.
- C. Terraform creates both buckets in AWS.
- D. Terraform renames the second bucket to `data_1`.

---

#### 11. (Domain 4 — CI/CD Automation Workflows)
In an enterprise multi-tier architecture, what is the recommended way to share outputs (such as VPC ID and Subnet IDs) from the `networking` workspace with the `app-cluster` workspace?
- A. Storing outputs in a shared text file on a network share.
- B. Using HCP Terraform / Terraform Enterprise `Run Triggers` and `terraform_remote_state` data sources or HCP Terraform cross-workspace state sharing.
- C. Hardcoding the VPC ID into the app configuration.
- D. Committing the raw `.tfstate` into GitHub.

---

#### 12. (Domain 5 — Security & Governance)
What is the function of the `tflint` tool in a platform engineering workflow?
- A. It provisions AWS infrastructure.
- B. It is a Pluggable Terraform Linter that catches provider-specific errors (e.g. invalid EC2 instance types, deprecated attributes, naming conventions) that `terraform validate` misses.
- C. It encrypts state files.
- D. It generates Grafana dashboards.

---

#### 13. (Domain 6 — Advanced Modules & Testing)
When authoring integration tests with Terratest, what Go command executes the test suite?
- A. `go test -v -timeout 30m`
- B. `terraform test`
- C. `terratest run`
- D. `go build`

---

#### 14. (Domain 7 — HCP Terraform & Enterprise)
What is the purpose of `Workspace Tagging` in HCP Terraform?
- A. To categorize, filter, and organize workspaces, and to scope Variable Sets and notification webhooks to matching workspace tags.
- B. To tag cloud resources in AWS.
- C. To sign Git commits.
- D. To bill separate credit cards.

---

#### 15. (Domain 1 — Scalable Configuration Design)
Which function converts an encoded JSON string into an HCL object/map?
- A. `jsondecode()`
- B. `jsonencode()`
- C. `parse_json()`
- D. `from_json()`

---

#### 16. (Domain 2 — State & Backend Operations)
Under what scenario would an engineer need to run `terraform state replace-provider`?
- A. When switching from an obsolete or deprecated provider namespace to a new provider namespace (e.g. migrating `hashicorp/template` to `template_file` or moving custom providers to Upbound/community namespaces) in state.
- B. When changing cloud providers from AWS to Azure.
- C. When updating the Terraform CLI version.
- D. When upgrading Linux.

---

#### 17. (Domain 3 — Lifecycle Operations)
Why is `lifecycle { ignore_changes = [all] }` considered an operational anti-pattern for long-term managed infrastructure?
- A. It makes `terraform plan` crash.
- B. It permanently blinds Terraform to any and all drift occurring on the resource in the cloud, defeating the core value proposition of declarative Infrastructure-as-Code.
- C. It costs extra cloud money.
- D. It deletes the resource after 30 days.

---

#### 18. (Domain 4 — CI/CD Automation Workflows)
In a GitHub Actions workflow, what environment variable must be set to authenticate to HCP Terraform when using the remote backend?
- A. `TF_TOKEN_app_terraform_io`
- B. `HCP_PASSWORD`
- C. `TERRAFORM_SECRET_KEY`
- D. `ATLAS_TOKEN`

---

#### 19. (Domain 5 — Security & Governance)
When configuring HashiCorp Sentinel, what is the role of the `sentinel.hcl` file?
- A. It defines the policy set, policy sources, and enforcement levels (`advisory`, `soft-mandatory`, `hard-mandatory`).
- B. It stores AWS root tokens.
- C. It replaces `main.tf`.
- D. It compiles Go code.

---

#### 20. (Domain 6 — Advanced Modules & Testing)
In a `.tftest.hcl` file, how can a test author execute tests against an existing deployed staging environment without creating new resources?
- A. By specifying `command = plan` and providing staging variable definitions.
- B. By disabling state.
- C. By running `terraform destroy`.
- D. By deleting providers.

---

#### 21. (Domain 7 — HCP Terraform & Enterprise)
In Terraform Enterprise, what is the recommended backup strategy for an Active/Active Flexible Deployment Option (FDO) installation?
- A. Backing up the external PostgreSQL database and the external object storage (S3/Azure Blob/GCS bucket).
- B. Taking a screenshot of the web dashboard.
- C. Copying the `/tmp` folder.
- D. Exporting Git commits.

---

#### 22. (Domain 1 — Scalable Configuration Design)
What does `try()` function in HCL do?
- A. It evaluates a list of expressions in sequence and returns the result of the first one that does not produce any errors.
- B. It executes a bash command.
- C. It retries failed cloud API calls.
- D. It validates email addresses.

---

#### 23. (Domain 2 — State & Backend Operations)
What information does the `serial` integer represent in `terraform.tfstate`?
- A. The license number of Terraform.
- B. A monotonic revision counter incremented each time the state file is successfully modified and saved by Terraform.
- C. The CPU serial number of the host machine.
- D. The number of resources in the file.

---

#### 24. (Domain 3 — Lifecycle Operations)
What does `terraform taint` do in legacy configurations (and how is it handled in modern Terraform)?
- A. It marks a resource for recreation on the next apply; in modern Terraform (v1.2+), this is accomplished via `terraform apply -replace=...` or `replace_triggered_by`.
- B. It encrypts the resource.
- C. It deletes the resource immediately from the cloud.
- D. It blocks future applies.

---

#### 25. (Domain 7 — HCP Terraform & Enterprise)
What is the function of `Speculative Plans` in HCP Terraform?
- A. Plans that buy spot instances.
- B. Ephemeral plans generated automatically on pull requests to predict changes and policy compliance without locking the workspace or allowing changes to be applied.
- C. Plans that run only on weekends.
- D. Plans that execute without a backend.

---

## Answer Key & Rationales

1. **A** — `can(cidrnetmask(var.cidr_block))` verifies valid IPv4 CIDR syntax, and `startswith()` verifies the prefix.
2. **B** — Plaintext secrets in state must be immediately invalidated/rotated upstream in the secrets manager, followed by adopting ephemeral/dynamic values.
3. **A** — Terraform's diff engine ignores attributes explicitly declared in `ignore_changes`, while processing updates to other unignored attributes.
4. **B** — `TF_PLUGIN_CACHE_DIR` tells Terraform to store and reuse downloaded provider binaries locally, eliminating redundant downloads.
5. **A** — The Rego rule inspects S3 server-side encryption configurations and verifies `sse_algorithm == "aws:kms"`.
6. **B** — In `terraform test`, a `run` block defaults to `command = apply` if not explicitly specified.
7. **B** — `Auto-apply: true` executes the apply immediately upon a successful plan and passing policy checks.
8. **A** — `sum()` sums a list or set of numbers.
9. **A** — `terraform plan -generate-config-out=generated.tf` generates HCL manifests from imported resources.
10. **B** — Resource blocks must have unique combinations of type and name within the same module scope.
11. **B** — HCP Terraform workspace sharing and `terraform_remote_state` allow controlled, auditable consumption of outputs across boundaries.
12. **B** — `tflint` catches provider-specific validation errors and style violations via AST analysis.
13. **A** — `go test` with a sufficient timeout (e.g., 30m) executes Terratest suites against real cloud APIs.
14. **A** — Workspace tags categorize workspaces and facilitate targeted assignment of variable sets and notification policies.
15. **A** — `jsondecode()` parses JSON into HCL native data structures.
16. **A** — `state replace-provider` updates the provider namespace recorded in existing state entries without recreation.
17. **B** — Ignoring all changes stops Terraform from detecting configuration drift, undermining IaC state fidelity.
18. **A** — `TF_TOKEN_<hostname>` (e.g. `TF_TOKEN_app_terraform_io`) supplies the authentication token for remote operations.
19. **A** — `sentinel.hcl` configures the policy suite and enforcement modes for HCP Terraform Sentinel checks.
20. **A** — `command = plan` against existing state runs assertions without provisioning new infrastructure.
21. **A** — TFE Active/Active stores state and metadata in PostgreSQL and object storage; backing up those two components ensures complete recovery.
22. **A** — `try()` returns the first expression that evaluates without an error.
23. **B** — `serial` is incremented on every state modification to prevent out-of-order writes.
24. **A** — Tainting flags a resource for deletion and recreation; modern Terraform uses `-replace`.
25. **B** — Speculative plans are read-only, non-locking plans generated on pull requests for preview and policy evaluation.
