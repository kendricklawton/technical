# Domain 6: Terraform State Management — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 6d — New in 004!)
A module author renames `resource "aws_kms_key" "main"` to `resource "aws_kms_key" "primary"` inside a shared module used by 15 application teams. How can the module author ensure that when all 15 teams upgrade the module and run `terraform plan` / `apply`, Terraform automatically updates the resource address in their state files instead of destroying and recreating their production KMS keys?
- A. Instruct all 15 teams to run `terraform taint` before upgrading.
- B. Add a declarative `moved { from = aws_kms_key.main to = aws_kms_key.primary }` block inside the module configuration.
- C. Add `lifecycle { create_before_destroy = true }` to the KMS key.
- D. Delete the `.terraform.lock.hcl` file in each repository.

---

### Question 2 (Objective 6d — New in 004!)
An engineering team wants to stop managing `aws_db_instance.legacy_reporting` with Terraform while leaving the live RDS database running untouched in AWS. Instead of running an imperative CLI command against remote state, they want to perform this refactoring declaratively via a Git Pull Request. How should they modify their HCL configuration?
- A. Remove the `resource "aws_db_instance" "legacy_reporting"` block and add a `removed` block with `from = aws_db_instance.legacy_reporting` and `lifecycle { destroy = false }`.
- B. Set `count = 0` inside `resource "aws_db_instance" "legacy_reporting"`.
- C. Add `lifecycle { prevent_destroy = true }` and delete the resource block.
- D. Add `lifecycle { ignore_changes = all }` and run `terraform destroy`.

---

### Question 3 (Objective 6c)
A developer writes the following `backend "s3"` block:
```hcl
terraform {
  backend "s3" {
    bucket = var.state_bucket_name
    key    = "${var.environment}/terraform.tfstate"
    region = "us-east-1"
  }
}
```
Why does `terraform init` fail on this configuration?
- A. The `s3` backend only supports the `us-west-2` region.
- B. A `backend` block cannot reference input variables (`var.*`), locals (`local.*`), or data sources because backend initialization happens during `terraform init` prior to variable evaluation; dynamic values must be supplied viapartial configuration (`-backend-config`).
- C. The `key` attribute must end in `.json`.
- D. `encrypt = true` is mandatory in all S3 backend blocks.

---

### Question 4 (Objective 6b & 6c)
In Terraform 1.10+ / 1.12+, how can you enable state locking on the Amazon S3 backend **without** provisioning or paying for a separate Amazon DynamoDB table?
- A. Set `use_lockfile = true` inside the `backend "s3"` block, which uses S3 conditional writes to create a `.tflock` object in the S3 bucket.
- B. Set `lock = "memory"` inside the `provider "aws"` block.
- C. Commit `.terraform.lock.hcl` to the S3 bucket.
- D. S3 state locking still strictly requires an Amazon RDS instance.

---

### Question 5 (Objective 6d)
During an on-call incident, an engineer manually increases the Desired Capacity of an Auto Scaling Group in the AWS Console. The team wants to inspect this out-of-band change and update `terraform.tfstate` to match the live AWS environment **without** modifying any cloud resources, and they want to preview the exact state changes before committing them. Which command should they run?
- A. `terraform refresh`
- B. `terraform apply -refresh-only` (or `terraform plan -refresh-only`)
- C. `terraform init -migrate-state`
- D. `terraform apply -replace`

---

### Question 6 (Objective 6a)
When using the default `local` backend, if you create a new CLI workspace using `terraform workspace new dev`, where does Terraform store the state file for the `dev` workspace?
- A. In `./dev.tfstate` in the root directory
- B. In `./terraform.tfstate.d/dev/terraform.tfstate`
- C. Inside `.terraform/providers/dev/state.json`
- D. In the user's home directory `~/.terraform.d/dev.tfstate`

---

### Question 7 (Objective 6b)
A GitLab CI runner loses power in the middle of a `terraform apply`, leaving the remote state locked. After verifying in the GitLab UI and AWS Console that no Terraform process is actively running, which command should an administrator run to clear the stale lock?
- A. `terraform init -reconfigure`
- B. `terraform force-unlock <LOCK_ID>`
- C. `terraform state rm lock`
- D. `terraform apply -lock=false`

---

### Question 8 (Objective 6c)
You are migrating a project from a `local` state file to an Azure Blob Storage (`azurerm`) remote backend. After adding the `backend "azurerm" { ... }` block to your configuration, what command copies your existing local state into the Azure storage container?
- A. `terraform init -reconfigure`
- B. `terraform init -migrate-state`
- C. `terraform state push azurerm`
- D. `terraform validate -migrate`

---

### Question 9 (Objective 6a)
What is the difference between **CLI Workspaces** (`terraform workspace new`) and **HCP Terraform Workspaces**?
- A. CLI Workspaces only create alternate state files for the same working directory and configuration files, whereas HCP Terraform Workspaces are independent cloud management units with their own access controls, variable sets, run queues, and VCS repository bindings.
- B. CLI Workspaces automatically encrypt state with AWS KMS, whereas HCP Terraform Workspaces store state unencrypted.
- C. CLI Workspaces require a paid license.
- D. They are completely identical.

---

### Question 10 (Objective 6c)
What is a **partial backend configuration**, and why is it used?
- A. Omitting sensitive or environment-specific arguments from the `backend` block in `.tf` files and supplying them at initialization time via `terraform init -backend-config=...` or environment variables, allowing the same `.tf` code to initialize against different state files per environment.
- B. Storing half of the state file locally and the other half in S3.
- C. Running `terraform apply` on only 50% of the resources.
- D. Using `count = 0` on backend resources.

---

## Answer Key & Rationales

1. **B** — A declarative `moved` block (`moved { from = ... to = ... }`) embedded in the module records the refactoring in HCL so every consumer's state is automatically updated during their next `plan`/`apply` without destroying resources.
2. **A** — Setting `count = 0` or deleting a resource block normally **destroys** the live cloud resource. Adding a `removed` block with `lifecycle { destroy = false }` after deleting the `resource` block tells Terraform to forget the resource from state without deleting the live database.
3. **B** — `backend` blocks cannot interpolate `var.*`, `local.*`, or `data.*` because backend initialization precedes variable loading. Use `terraform init -backend-config=...` instead.
4. **A** — Terraform 1.10+ introduced `use_lockfile = true` on the `s3` backend, leveraging native S3 conditional writes to manage a `.tflock` file in the bucket without needing DynamoDB (though `dynamodb_table` is also still understood for legacy questions).
5. **B** — `terraform plan -refresh-only` / `terraform apply -refresh-only` replaces the deprecated `terraform refresh` command by allowing operators to preview state updates before writing them to `terraform.tfstate`.
6. **B** — The `local` backend stores non-default CLI workspace state files under `terraform.tfstate.d/<WORKSPACE_NAME>/terraform.tfstate`.
7. **B** — `terraform force-unlock <LOCK_ID>` releases a stuck state lock using the Lock ID displayed in the lock error output.
8. **B** — `terraform init -migrate-state` migrates existing state data to the newly configured backend. (`-reconfigure` skips migration and starts with an empty state.)
9. **A** — CLI Workspaces simply swap state file paths within the same configuration directory, whereas HCP Terraform Workspaces provide full RBAC, variable, VCS, and run isolation.
10. **A** — Partial backend configuration keeps credentials and environment-specific state keys out of static `.tf` code and injects them via `-backend-config` during `terraform init`.
