# Terraform Associate (004) — Practice Exam 1

**Focus:** Core Workflow, HCL Configuration, Custom Conditions (`4g`), and Ephemeral / Write-Only Secrets (`4h`)  
**Tested Version:** Terraform 1.12+  
**Format:** 25 Scenario-Based Questions with In-Depth Technical Explanations  
**Passing Score:** 80% (20 / 25)

---

### Question 1 (Objective 4h — New in 004)
A security engineer wants to fetch a database master password from AWS Secrets Manager during `terraform apply` and pass it to an `aws_db_instance` resource **without** the password ever being written to `terraform.tfstate` or saved binary plan files. Which combination of Terraform 1.12+ features achieves this?
- A. Use `data "aws_secretsmanager_secret_version"` and set `sensitive = true` on the database resource.
- B. Use an `ephemeral "aws_secretsmanager_secret_version"` block to fetch the secret and pass it to the database resource's write-only `password_wo` argument (alongside `password_wo_version`).
- C. Store the password in `terraform.tfvars` and run `terraform apply -no-state`.
- D. Use a `local` value wrapped in the `base64encode()` function.

---

### Question 2 (Objective 4g — New in 004)
An engineer wants to ensure that before an `aws_instance` resource is created, the subnet passed into `subnet_id` resides in the `us-east-1a` Availability Zone. If it does not, Terraform must immediately abort the plan/apply with a custom error message. Where should this rule be configured?
- A. Inside a top-level `check` block
- B. Inside a `precondition` block nested within the `aws_instance` resource's `lifecycle` block
- C. Inside an `output` block's `description` field
- D. Inside `.terraform.lock.hcl`

---

### Question 3 (Objective 4g — New in 004)
How does a top-level `check` block differ from a `postcondition` block when its assertion evaluates to `false` during `terraform apply`?
- A. A failed `check` block emits a **Warning** without halting execution or blocking downstream resources, whereas a failed `postcondition` raises a **hard error** that halts execution of dependent resources.
- B. A failed `check` block automatically rolls back the entire workspace and runs `terraform destroy`.
- C. A `check` block can only be evaluated in HCP Terraform Enterprise, whereas `postcondition` works in the CLI.
- D. There is no difference; both halt execution immediately.

---

### Question 4 (Objective 4c)
A variable `instance_size` is defined in all of the following places:
1. Default value in `variables.tf`: `"t3.nano"`
2. Environment variable `TF_VAR_instance_size`: `"t3.micro"`
3. `terraform.tfvars`: `"t3.small"`
4. `99-override.auto.tfvars`: `"t3.medium"`
5. `01-base.auto.tfvars`: `"t3.large"`

If the engineer runs `terraform plan` with no CLI `-var` flags, which value is selected for `var.instance_size`?
- A. `"t3.small"`
- B. `"t3.large"`
- C. `"t3.medium"`
- D. `"t3.micro"`

---

### Question 5 (Objective 4f)
An engineer sets `create_before_destroy = true` inside the `lifecycle` block of an `aws_launch_template` resource that is referenced by an `aws_autoscaling_group`. What happens during `terraform apply` when a change forces replacement of the launch template?
- A. Terraform destroys the old launch template first, waits 60 seconds, and then creates the new one.
- B. Terraform creates the new launch template first, updates the Auto Scaling Group to point to the new template, and only then destroys the old launch template.
- C. Terraform refuses to modify the launch template and throws a `prevent_destroy` error.
- D. Terraform ignores all changes to the launch template.

---

### Question 6 (Objective 2a)
Which provider version constraint allows upgrading from `4.15.2` to `4.15.9`, but strictly prohibits upgrading to `4.16.0` or `5.0.0`?
- A. `version = "~> 4.15"`
- B. `version = "~> 4.15.2"`
- C. `version = ">= 4.15.2"`
- D. `version = "!= 4.16.0"`

---

### Question 7 (Objective 3c)
True or False: Running `terraform validate` connects to your cloud provider's API to verify that your IAM user has sufficient permissions and that requested VM quotas are available in the target region.
- A. True
- B. False

---

### Question 8 (Objective 4e)
Examine the following configuration:
```hcl
variable "team_members" {
  type    = list(string)
  default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "dev" {
  for_each = toset(var.team_members)
  name     = each.value
}
```
If `"alice"` leaves the company and is removed from `var.team_members`, what will `terraform plan` propose?
- A. Destroy `aws_iam_user.dev["alice"]`, and destroy/recreate `"bob"` and `"charlie"` because their indices shifted.
- B. Destroy **only** `aws_iam_user.dev["alice"]`, leaving `aws_iam_user.dev["bob"]` and `aws_iam_user.dev["charlie"]` untouched.
- C. Throw an error because `toset()` cannot be used inside `for_each`.
- D. Do nothing until `-replace` is specified.

---

### Question 9 (Objective 4h — New in 004)
An engineer configures an `aws_db_instance` using `password_wo = ephemeral.random_password.db.result` and `password_wo_version = 1`. Six months later, compliance requires rotating the database password in-place. Because `password_wo` is not stored in `terraform.tfstate`, what must the engineer change in the HCL configuration so `terraform apply` generates and sends a new password to AWS RDS?
- A. Change `sensitive = true` to `sensitive = false`.
- B. Increment `password_wo_version` from `1` to `2`.
- C. Run `terraform validate -rotate`.
- D. Delete `terraform.tfstate`.

---

### Question 10 (Objective 3d & 3e)
An engineer runs `terraform plan -out=release.tfplan`. After peer review, the engineer runs `terraform apply release.tfplan`. Which statement is true about the `apply` step?
- A. Terraform will prompt `Do you want to perform these actions? Enter a value:` and wait for `yes`.
- B. Terraform will execute the exact actions recorded in `release.tfplan` immediately **without** prompting for interactive confirmation and **without** performing a new plan diff.
- C. Terraform will ignore `release.tfplan` unless `-auto-approve` is also passed.
- D. `release.tfplan` is a human-readable HCL file that can be edited in Vim before apply.

---

### Question 11 (Objective 1a & 1b)
Which property of Terraform guarantees that running `terraform apply` five times in a row against an unchanged `.tf` configuration and unchanged cloud environment will result in the resources being provisioned on the first run and zero modifications occurring on the subsequent four runs?
- A. Polymorphism
- B. Idempotence
- C. Eventual consistency
- D. Imperative looping

---

### Question 12 (Objective 2c)
You need to deploy an ACM SSL certificate specifically in `us-east-1` for an AWS CloudFront distribution, while the rest of your infrastructure is deployed in `eu-west-1` via your default `provider "aws"` block. After adding `provider "aws" { alias = "us_east_1" region = "us-east-1" }`, how do you bind the `aws_acm_certificate` resource to the `us-east-1` provider?
- A. Add `provider = aws.us_east_1` inside the `aws_acm_certificate` resource block.
- B. Add `alias = "us_east_1"` inside the `aws_acm_certificate` resource block.
- C. Rename the resource to `resource "aws_us_east_1_acm_certificate" "cert"`.
- D. Place the resource in a file named `us-east-1.tf`.

---

### Question 13 (Objective 4d)
Which HCL type constraint allows you to define a variable that accepts a single structured map with explicitly named attributes of **different data types** (for example: `name = string`, `port = number`, `public = bool`)?
- A. `map(string)`
- B. `list(any)`
- C. `object({ name = string, port = number, public = bool })`
- D. `set(string)`

---

### Question 14 (Objective 4e)
Inside a `dynamic "ebs_block_device"` block that does **not** specify a custom `iterator` argument, how do you reference the current element's value inside the `content { ... }` block?
- A. `each.value`
- B. `ebs_block_device.value`
- C. `self.value`
- D. `dynamic.value`

---

### Question 15 (Objective 3g)
A developer runs `terraform fmt` in the root directory of a project. What output does `terraform fmt` print to the terminal by default?
- A. The full contents of every `.tf` file in the repository.
- B. The filenames of the `.tf` configuration files that were actually modified/reformatted (or no output if all files were already properly formatted).
- C. A dependency graph in DOT format.
- D. The estimated monthly cloud cost.

---

### Question 16 (Objective 4f)
What happens if a developer runs `terraform destroy` when one of the resources in `main.tf` contains `lifecycle { prevent_destroy = true }`?
- A. Terraform destroys all other resources and quietly skips the protected resource without error.
- B. Terraform halts with an error refusing to generate the destroy plan for that resource as long as `prevent_destroy = true` remains in the configuration.
- C. `prevent_destroy` only applies to `terraform apply`, never to `terraform destroy`.
- D. Terraform prompts the user to enter their cloud root password to override it.

---

### Question 17 (Objective 2b)
Where does `terraform init` record the cryptographic SHA-256 hashes of the provider plugins it selects so that future `terraform init` executions across team machines verify plugin integrity?
- A. `terraform.tfstate`
- B. `.terraform.lock.hcl`
- C. `providers.tf.json`
- D. `.terraform/environment`

---

### Question 18 (Objective 4e)
Which built-in Terraform function reads a template file from disk and renders its interpolated variables using a provided map of key/value pairs?
- A. `file()`
- B. `templatefile(path, vars)`
- C. `lookup()`
- D. `formatlist()`

---

### Question 19 (Objective 4g — New in 004)
Where can a `precondition` block be placed in Terraform 1.12+?
- A. Inside the `lifecycle` block of a `resource`, `data` source, or `output` block
- B. Inside a `terraform { required_providers { ... } }` block
- C. Inside a `locals` block
- D. Inside a `backend` block

---

### Question 20 (Objective 4h)
Even when an input variable is marked `sensitive = true`, why is it still considered a critical security risk to commit `terraform.tfstate` to a Git repository?
- A. Because `sensitive = true` only suppresses terminal/log output during `plan` and `apply`; the actual value is still written in **plaintext JSON** inside `terraform.tfstate` if used by a regular resource attribute.
- B. Because Git automatically strips `sensitive = true` from `.tf` files.
- C. Because `terraform.tfstate` files exceed GitHub's 100 GB file limit.
- D. Because `sensitive = true` only works on Windows.

---

### Question 21 (Objective 3e)
Instead of the deprecated `terraform taint` command, which flag should you pass to `terraform plan` or `terraform apply` to force the replacement of a single degraded resource (`aws_instance.cache[0]`)?
- A. `-recreate="aws_instance.cache[0]"`
- B. `-replace="aws_instance.cache[0]"`
- C. `-target="aws_instance.cache[0]"`
- D. `-force-new="aws_instance.cache[0]"`

---

### Question 22 (Objective 4b)
If a resource is provisioned using `count = 3` (`resource "aws_instance" "web" { count = 3 ... }`), which splat expression returns a list of the private IP addresses of all 3 instances?
- A. `aws_instance.web.private_ip`
- B. `aws_instance.web[*].private_ip`
- C. `aws_instance.web[all].private_ip`
- D. `each.aws_instance.web.private_ip`

---

### Question 23 (Objective 4f)
An EC2 instance's `tags` attribute is modified out-of-band by an AWS Lambda cost-allocation function every night. How do you configure the `aws_instance` resource in Terraform so that `terraform plan` and `apply` ignore changes to `tags` while continuing to manage all other attributes of the instance?
- A. Add `lifecycle { ignore_changes = [tags] }` inside the `aws_instance` block.
- B. Add `lifecycle { prevent_destroy = true }` inside the `aws_instance` block.
- C. Run `terraform state rm aws_instance.app`.
- D. Set `tags = null`.

---

### Question 24 (Objective 1c)
Which statement best explains why Terraform is considered **Day 1** infrastructure provisioning tooling rather than a traditional **Day 2** OS configuration management tool?
- A. Terraform focuses on provisioning and managing the lifecycle of cloud/network/SaaS infrastructure resources via provider APIs, rather than managing ongoing inside-the-OS package updates and daemon files.
- B. Terraform can only be run on the first day an AWS account is created.
- C. Terraform cannot modify infrastructure after the initial `terraform apply`.
- D. Terraform requires a pull agent inside every Linux VM.

---

### Question 25 (Objective 4h — New in 004)
Which of the following statements about an `ephemeral` resource block (e.g., `ephemeral "vault_kv_secret_v2" "api_key" { ... }`) is TRUE?
- A. Its attributes are saved permanently in `terraform.tfstate` so they can be viewed with `terraform state show`.
- B. Terraform opens/fetches the ephemeral resource during the active `plan`/`apply` cycle and closes/discards it when the run finishes, **never** storing its data in `terraform.tfstate` or plan files.
- C. It can only be used with the `local` backend.
- D. It destroys the secret in HashiCorp Vault when `terraform plan` finishes.

---

## Answer Key & Explanations

1. **B** — In Terraform 1.12+ (Objective `4h`), combining an `ephemeral` resource block (`ephemeral "aws_secretsmanager_secret_version"`) with a resource's write-only argument (`password_wo` + `password_wo_version`) guarantees the secret is never written to `.tfstate` or binary plan files.
2. **B** — A `precondition` block inside the resource's `lifecycle` block validates assumptions before the resource is created and halts execution with a custom `error_message` if `condition` is `false`.
3. **A** — `check` blocks emit non-blocking **Warnings** when assertions fail, whereas `precondition` and `postcondition` blocks halt execution with **hard errors**.
4. **C** — Precedence order: `.auto.tfvars` files (Priority 4) override `terraform.tfvars` (Priority 2), `TF_VAR_*` (Priority 1), and `default`. When multiple `.auto.tfvars` files exist, they are loaded in **alphabetical (lexical) order**, so `99-override.auto.tfvars` (`"t3.medium"`) is loaded after and overrides `01-base.auto.tfvars` (`"t3.large"`).
5. **B** — `create_before_destroy = true` provisions the replacement resource first and updates dependents before destroying the prior resource.
6. **B** — `~> 4.15.2` allows only the patch segment (`2`) to increment (`>= 4.15.2, < 4.16.0`).
7. **B (False)** — `terraform validate` is strictly an offline syntax and schema check; it never contacts cloud APIs or checks IAM permissions/quotas.
8. **B** — Because `for_each` indexes instances by string key (`"alice"`, `"bob"`, `"charlie"`), removing `"alice"` destroys only `aws_iam_user.dev["alice"]` without touching the other instances.
9. **B** — Write-only arguments (`*_wo`) are paired with a state-tracked version integer (`*_wo_version`). Incrementing `password_wo_version` triggers Terraform to send the updated write-only value to the provider.
10. **B** — Passing a pre-saved binary plan file (`terraform apply release.tfplan`) executes those exact changes immediately without re-planning or asking for interactive confirmation.
11. **B** — **Idempotence** guarantees that applying an unchanged configuration repeatedly produces no duplicate resources or unintended side effects.
12. **A** — Reference the aliased provider via `provider = <PROVIDER>.<ALIAS>` (`provider = aws.us_east_1`).
13. **C** — `object({ ... })` is a structural type that groups named attributes of different data types.
14. **B** — Inside a `dynamic "<LABEL>"` block, the default iterator name matches the block label (`ebs_block_device.value` and `ebs_block_device.key`).
15. **B** — `terraform fmt` outputs the names of the files it actually modified.
16. **B** — As long as `prevent_destroy = true` is present in the `.tf` configuration, Terraform will error out and refuse to destroy the resource.
17. **B** — `.terraform.lock.hcl` stores provider version locks and cryptographic checksums.
18. **B** — `templatefile(path, vars)` reads a file at `path` and renders its template expressions using the `vars` map.
19. **A** — `precondition` blocks can be defined inside the `lifecycle` block of a `resource`, `data` source, or `output` block.
20. **A** — `sensitive = true` only redacts CLI output; unless an attribute is write-only (`_wo`) or ephemeral, its value is stored in plaintext JSON in `terraform.tfstate`.
21. **B** — `-replace="<ADDRESS>"` is the modern flag on `terraform plan` / `apply` that replaces `terraform taint`.
22. **B** — The splat expression `aws_instance.web[*].private_ip` collects the `private_ip` attribute across all `count` instances.
23. **A** — `ignore_changes = [tags]` tells Terraform to ignore external drift on the `tags` attribute.
24. **A** — Terraform is designed for Day 1 infrastructure lifecycle provisioning via APIs, whereas tools like Ansible/Chef/Puppet manage Day 2 in-guest OS configuration.
25. **B** — `ephemeral` resources exist solely in memory for the duration of the current Terraform operation and are never persisted to state or plan files.
