# Terraform Associate (004) — Practice Exam 2

**Focus:** State Management, `moved`/`removed`/`import` Blocks (`6d`, `7a`), Modules (`5a`–`5d`), and CLI Operations (`7b`, `7c`)  
**Tested Version:** Terraform 1.12+  
**Format:** 25 Scenario-Based Questions with In-Depth Technical Explanations  
**Passing Score:** 80% (20 / 25)

---

### Question 1 (Objective 6d — New in 004)
An engineer is refactoring a large root module by moving `resource "aws_rds_cluster" "aurora"` into a child module `module "database"`. Instead of running `terraform state mv` manually in the terminal, how can the engineer declare this move directly in HCL so it is peer-reviewed in Git and applied automatically during `terraform apply`?
- A. Add `lifecycle { moved_to = module.database }` inside the RDS resource.
- B. Add `import { from = aws_rds_cluster.aurora to = module.database.aws_rds_cluster.aurora }`.
- C. Set `enable_auto_move = true` inside the `terraform` block.
- D. Add `moved { from = aws_rds_cluster.aurora to = module.database.aws_rds_cluster.aurora }` to the root module configuration.

---

### Question 2 (Objective 6d — New in 004)
A platform team needs to decommission Terraform's management of `aws_route53_zone.public` so a separate DNS team can take ownership of it, **without** deleting the live Route 53 hosted zone in AWS. If they want to accomplish this declaratively in `.tf` code rather than running `terraform state rm`, what should they do?
- A. Delete the `resource` block and add `removed { from = aws_route53_zone.public lifecycle { destroy = true } }`.
- B. Keep the `resource` block and add `lifecycle { prevent_destroy = true }`.
- C. Set `count = 0` on `aws_route53_zone.public`.
- D. Delete the `resource "aws_route53_zone" "public"` block and replace it with:
  ```hcl
  removed {
    from = aws_route53_zone.public
    lifecycle {
      destroy = false
    }
  }
  ```

---

### Question 3 (Objective 7a — New in 004)
You have added the following `import` block to `imports.tf` for an existing S3 bucket:
```hcl
import {
  to = aws_s3_bucket.marketing_assets
  id = "acme-marketing-assets-prod"
}
```
Which command instructs Terraform to query the existing S3 bucket in AWS and automatically generate the corresponding `resource "aws_s3_bucket" "marketing_assets"` HCL block into a file named `s3_generated.tf`?
- A. `terraform plan -generate-config-out=s3_generated.tf`
- B. `terraform import -out=s3_generated.tf`
- C. `terraform fmt -generate=s3_generated.tf`
- D. `terraform init -generate=s3_generated.tf`

---

### Question 4 (Objective 6b & 6c)
In Terraform 1.12, an engineer configures an Amazon S3 remote backend with `use_lockfile = true`. What does `use_lockfile = true` do?
- A. It enables native S3 state locking by writing a `.tflock` object in the S3 bucket using S3 conditional writes, removing the requirement for an external DynamoDB table.
- B. It disables state locking during `terraform apply`.
- C. It locks the local `.tf` files as read-only on the developer's laptop.
- D. It copies `.terraform.lock.hcl` into the S3 bucket.

---

### Question 5 (Objective 5a & 5d)
Examine the following two module blocks:
```hcl
module "vpc_registry" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
}

module "vpc_local" {
  source  = "./modules/vpc"
  version = "1.0.0"
}
```
What will happen when the engineer runs `terraform init` / `terraform validate`?
- A. `module "vpc_registry"` fails because Registry modules must use `?ref=v5.0`.
- B. Both modules initialize cleanly.
- C. `module "vpc_local"` fails validation because the `version` argument is **not supported** for local path modules (`./modules/vpc`); `version` is only valid for modules sourced from a Public or Private Terraform Registry.
- D. Both modules fail because a configuration can only contain one `module` block.

---

### Question 6 (Objective 5b)
Inside `./modules/compute/main.tf`, an EC2 instance `resource "aws_instance" "app" { ... }` is created. Inside `./modules/compute/outputs.tf`, the following output is defined:
```hcl
output "instance_private_ip" {
  value = aws_instance.app.private_ip
}
```
If the root module calls this module as `module "web_cluster" { source = "./modules/compute" }`, what is the ONLY valid syntax for the root module to reference that private IP?
- A. `module.compute.instance_private_ip`
- B. `module.web_cluster.instance_private_ip`
- C. `data.module.web_cluster.instance_private_ip`
- D. `module.web_cluster.aws_instance.app.private_ip`

---

### Question 7 (Objective 6d)
Why did HashiCorp deprecate the standalone `terraform refresh` command in favor of `terraform plan -refresh-only` and `terraform apply -refresh-only`?
- A. Because `terraform refresh` did not work with AWS.
- B. Because `terraform refresh` immediately overwrote `terraform.tfstate` without showing a plan preview or asking for confirmation, whereas `-refresh-only` lets operators review proposed state changes before committing them to the state file.
- C. Because `terraform refresh` could only refresh 1 resource at a time.
- D. Because `terraform refresh` deleted cloud resources.

---

### Question 8 (Objective 7c)
An engineer wants to capture full `TRACE`-level diagnostic logs from **both** Terraform Core and Provider plugins and save them to `./crash-debug.log`. Which environment variables must be set before running `terraform apply`?
- A. `export TF_LOG="TRACE"` and `export TF_LOG_PATH="./crash-debug.log"`
- B. `export TF_LOG_PATH="./crash-debug.log"` only
- C. `export TERRAFORM_TRACE="./crash-debug.log"`
- D. `export TF_DEBUG="true"` and `export TF_OUTPUT="./crash-debug.log"`

---

### Question 9 (Objective 7b)
Which `terraform state` subcommand prints the complete, raw JSON contents of the current state file from a remote backend directly to standard output (`stdout`)?
- A. `terraform state export`
- B. `terraform state show`
- C. `terraform state pull`
- D. `terraform state list`

---

### Question 10 (Objective 6c)
You have an existing `backend "s3"` configuration pointing to `bucket = "old-state-bucket"`. You update the configuration to `bucket = "new-state-bucket"` and want to initialize the new bucket **without** copying or migrating the existing state from the old bucket. Which command flag should you use?
- A. `terraform init -migrate-state`
- B. `terraform state rm --all`
- C. `terraform init -upgrade`
- D. `terraform init -reconfigure`

---

### Question 11 (Objective 5a)
An engineer wants to source a Terraform module located inside a subfolder named `modules/eks` of a GitHub repository `github.com/acme/infra-modules` pinned to the Git tag `v3.2.0`. Which `source` string syntax is correct?
- A. `source = "github.com/acme/infra-modules//modules/eks?ref=v3.2.0"`
- B. `source = "registry.terraform.io/acme/infra-modules/eks"`
- C. `source = "github.com/acme/infra-modules:modules/eks@v3.2.0"`
- D. `source = "./github.com/acme/infra-modules/modules/eks"`

---

### Question 12 (Objective 6a)
When using CLI workspaces (`terraform workspace`), how can you dynamically append the active workspace name (e.g., `dev`, `stage`, `prod`) to an S3 bucket name in HCL?
- A. `bucket = "acme-app-logs-${local.workspace}"`
- B. `bucket = "acme-app-logs-${terraform.workspace}"`
- C. `bucket = "acme-app-logs-${var.workspace}"`
- D. `bucket = "acme-app-logs-${env.TF_WORKSPACE}"`

---

### Question 13 (Objective 6a)
Which CLI workspace exists by default in every new Terraform working directory and **cannot** be deleted?
- A. `main`
- B. `local`
- C. `default`
- D. `master`

---

### Question 14 (Objective 7b)
You have refactored `aws_s3_bucket.logs` into `aws_s3_bucket.audit_logs` and want to update the state file immediately via the CLI instead of using a `moved` block. Which command performs this operation?
- A. `terraform state rename aws_s3_bucket.logs aws_s3_bucket.audit_logs`
- B. `terraform state cp aws_s3_bucket.logs aws_s3_bucket.audit_logs`
- C. `terraform state mv aws_s3_bucket.logs aws_s3_bucket.audit_logs`
- D. `terraform import aws_s3_bucket.audit_logs aws_s3_bucket.logs`

---

### Question 15 (Objective 6b)
What happens if you run `terraform plan` against an Azure Blob Storage (`azurerm`) remote backend while a colleague's `terraform apply` is actively running against that same state?
- A. `terraform plan` never locks state under any circumstances.
- B. Both operations proceed concurrently without interference.
- C. `terraform plan` automatically terminates the colleague's running `apply`.
- D. `terraform plan` attempts to acquire a state lock (Azure Blob Lease), sees the state is already locked by the running `apply`, and fails with a state lock error (unless `-lock=false` or `-lock-timeout` is specified).

---

### Question 16 (Objective 4e)
Which built-in function combines two or more maps into a single map, where keys from later arguments overwrite matching keys from earlier arguments?
- A. `concat()`
- B. `zipmap()`
- C. `merge()`
- D. `join()`

---

### Question 17 (Objective 4d)
What is the structural difference between a `list(string)` and a `set(string)` in Terraform?
- A. There is no difference.
- B. A `list(string)` can be passed directly to a resource's `for_each` argument without conversion.
- C. A `set(string)` can contain numbers and booleans, whereas a `list(string)` cannot.
- D. A `list(string)` maintains a strict 0-indexed ordering and allows duplicate values, whereas a `set(string)` is an unordered collection of unique strings that removes duplicates and cannot be indexed by position (`[0]`).

---

### Question 18 (Objective 3b)
You have cloned a repository from GitHub that calls two Registry modules (`terraform-aws-modules/vpc/aws` and `terraform-aws-modules/eks/aws`). When you immediately run `terraform plan`, Terraform reports `Error: Module not installed`. What command must you run?
- A. `terraform fmt`
- B. `terraform validate`
- C. `terraform state list`
- D. `terraform init` (or `terraform get`)

---

### Question 19 (Objective 7c)
If you set `export TF_LOG="JSON"`, what does Terraform output to the log stream?
- A. Nothing; `JSON` is not a valid setting for `TF_LOG`.
- B. Machine-parseable JSON-formatted diagnostic log lines at the `TRACE` verbosity level.
- C. Only syntax errors in `.tf.json` files.
- D. Only the contents of `terraform.tfstate`.

---

### Question 20 (Objective 4c)
An engineer wants to read the value of a single root output named `eks_cluster_endpoint` in a bash script without surrounding quotes (`"..."`) so it can be passed directly to `curl`. Which command flag should be used?
- A. `terraform plan -output eks_cluster_endpoint`
- B. `terraform output -raw eks_cluster_endpoint`
- C. `terraform output -json eks_cluster_endpoint`
- D. `terraform state show eks_cluster_endpoint`

---

### Question 21 (Objective 6c)
Why is it recommended to enable **Object Versioning** and **Server-Side Encryption (SSE-KMS)** on an Amazon S3 bucket used as a Terraform remote state backend?
- A. Because S3 Object Versioning automatically upgrades Terraform provider plugins.
- B. Because KMS encryption eliminates the need for state locking.
- C. Because `terraform.tfstate` can contain sensitive data in plaintext JSON (protected at rest by SSE-KMS), and Object Versioning preserves prior state revisions for disaster recovery if state is accidentally corrupted or deleted.
- D. Because Terraform refuses to initialize an S3 backend unless Versioning and KMS are enabled.

---

### Question 22 (Objective 4f)
An engineer configures `replace_triggered_by = [aws_security_group.web.id]` inside the `lifecycle` block of `aws_instance.web`. When will Terraform force the replacement (destroy and recreate) of `aws_instance.web`?
- A. Whenever `aws_security_group.web.id` changes (i.e., when the security group is replaced).
- B. Never; `replace_triggered_by` prevents replacement.
- C. Every time `terraform plan` is executed.
- D. Only when `aws_instance.web` is manually tainted.

---

### Question 23 (Objective 5b)
Where must provider `required_providers` source mappings be declared when a root module calls multiple child modules?
- A. Only in the root module; child modules automatically inherit provider source mappings.
- B. **Each module** (both the root module and every child module) should declare its own `terraform { required_providers { ... } }` block specifying the provider local names, source addresses, and minimum compatible version constraints it relies on.
- C. Inside `outputs.tf`.
- D. Inside `terraform.tfvars`.

---

### Question 24 (Objective 7a)
Can a declarative `import` block in Terraform 1.12 use the `for_each` meta-argument to import dozens of existing cloud resources simultaneously in a single `terraform apply`?
- A. No, `import` blocks can only import one resource per `terraform apply`.
- B. Only for `local_file` resources.
- C. Yes, `import` blocks support `for_each` (e.g., iterating over a map of resource keys and cloud IDs).
- D. Only if using `count` instead of `for_each`.

---

### Question 25 (Objective 6d)
If an administrator manually deletes an EC2 instance (`i-0aaa111bbb222ccc3`) in the AWS Console that was managed by Terraform, what happens during the next standard `terraform plan`?
- A. During the refresh phase of `terraform plan`, Terraform detects that `i-0aaa111bbb222ccc3` no longer exists in AWS, removes it from the refreshed in-memory state, and proposes creating (`+`) a new `aws_instance` to restore the desired state defined in `.tf` code.
- B. Terraform deletes the `resource "aws_instance"` block from `main.tf` automatically.
- C. Terraform ignores the missing instance unless `-replace` is passed.
- D. Terraform throws a fatal crash error because the instance ID is missing.

---

## Answer Key & Explanations

1. **D** — Declarative `moved` blocks (`moved { from = ... to = ... }`) record resource address changes in HCL so `terraform plan`/`apply` seamlessly updates state without destroying the resource.
2. **D** — Deleting the `resource` block and adding a `removed` block with `lifecycle { destroy = false }` removes the resource from Terraform state during `apply` while leaving the live cloud resource running.
3. **A** — `terraform plan -generate-config-out=s3_generated.tf` reads resources targeted by `import` blocks and writes their generated HCL `resource` definitions to the specified file.
4. **A** — Introduced in TF 1.10 and tested on the 004 exam, `use_lockfile = true` enables native S3 state locking via a `.tflock` object using S3 conditional writes.
5. **C** — The `version` argument inside a `module` block is **only** valid for Registry modules; local path modules (`./...`) do not support `version`.
6. **B** — Child module outputs are referenced via `module.<MODULE_CALL_NAME>.<OUTPUT_NAME>` (`module.web_cluster.instance_private_ip`).
7. **B** — `terraform plan -refresh-only` / `terraform apply -refresh-only` allows operators to inspect proposed state updates before modifying `terraform.tfstate`.
8. **A** — Both `TF_LOG="TRACE"` and `TF_LOG_PATH="./crash-debug.log"` must be set to write trace logs to a file.
9. **C** — `terraform state pull` fetches the raw state JSON from the active backend and prints it to stdout.
10. **D** — `terraform init -reconfigure` initializes the new backend configuration without migrating existing state.
11. **A** — In module `source` addresses, a double slash `//` separates the repository root URL from a subdirectory path (`github.com/acme/infra-modules//modules/eks?ref=v3.2.0`).
12. **B** — `${terraform.workspace}` evaluates to the name of the currently active workspace.
13. **C** — Every working directory starts with the `default` workspace, which cannot be deleted.
14. **C** — `terraform state mv <SRC> <DEST>` renames or moves a resource address in the state file via CLI.
15. **D** — Standard `terraform plan` acquires a state lock before refreshing state; if another process holds the lock, `plan` fails to acquire the lock and exits safely.
16. **C** — `merge(map1, map2, ...)` combines multiple maps into a single map.
17. **D** — Lists are ordered and indexable (`list[0]`); sets are unordered collections of unique elements and cannot be indexed by integer position.
18. **D** — `terraform init` (or `terraform get`) downloads and installs child modules into `.terraform/modules/`.
19. **B** — `TF_LOG="JSON"` outputs `TRACE`-level diagnostic logs structured as JSON objects.
20. **B** — `terraform output -raw <NAME>` prints the raw unquoted string value of an output, ideal for shell scripts and pipes.
21. **C** — Encryption at rest (`SSE-KMS`) protects sensitive values stored in `terraform.tfstate`, while S3 Object Versioning enables point-in-time recovery if state is corrupted.
22. **A** — `replace_triggered_by` forces replacement of the containing resource whenever any referenced resource or attribute changes in the plan.
23. **B** — Provider source mappings (`required_providers`) are scoped per module; HashiCorp best practice requires every module to declare its own `required_providers` block.
24. **C** — In Terraform 1.7+ / 1.12+, `import` blocks support `for_each` for bulk declarative importing.
25. **A** — During `plan`, Terraform refreshes state against AWS, sees the instance was deleted out-of-band, and plans to create (`+`) a new instance to match the `.tf` configuration.
