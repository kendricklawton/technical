# Domain 4: Terraform Configuration — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 4g — New in 004!)
A cloud architect wants to verify that an HTTPS load balancer returns an HTTP `200 OK` status code after `terraform apply` finishes, but does **NOT** want a transient `503` response to halt or fail the Terraform execution. Which HCL construct is specifically designed for non-blocking continuous assertions?
- A. A `postcondition` block inside the load balancer's `lifecycle` block
- A `precondition` block inside the load balancer's `lifecycle` block
- C. A top-level `check` block containing a scoped `data "http"` block and an `assert` block
- D. A `validation` block inside an input variable

---

### Question 2 (Objective 4g — New in 004!)
An engineer adds the following block inside an `aws_ebs_volume` resource:
```hcl
resource "aws_ebs_volume" "data" {
  availability_zone = "us-east-1a"
  size              = 100
  encrypted         = var.enable_encryption

  lifecycle {
    postcondition {
      condition     = self.encrypted == true
      error_message = "Production EBS volumes must be encrypted!"
    }
  }
}
```
What happens if `var.enable_encryption` evaluates to `false`?
- A. Terraform logs a warning message and continues creating downstream EC2 instances.
- B. Terraform evaluates `self.encrypted`, fails the `postcondition` with a hard error, and prevents any downstream resources that depend on `aws_ebs_volume.data` from executing.
- C. Terraform automatically changes `encrypted` to `true` to satisfy the postcondition.
- D. The `self` object is invalid inside a `postcondition` block.

---

### Question 3 (Objective 4h — New in 004!)
Why did HashiCorp introduce **ephemeral resources** (`ephemeral "..."`) and **write-only arguments** (such as `password_wo`) in Terraform 1.10 / 1.11+?
- A. Because `sensitive = true` only redacts values in CLI terminal output while still persisting secrets in plaintext JSON inside `terraform.tfstate` and saved plan files; ephemeral values and write-only arguments are never written to state or plan files.
- B. To make `terraform init` download provider plugins faster.
- C. To allow Terraform to run without a state file.
- D. To replace `terraform.tfvars` files with encrypted binary files.

---

### Question 4 (Objective 4h — New in 004!)
Because a write-only argument like `password_wo` on `aws_db_instance` is never stored in `terraform.tfstate`, Terraform cannot compare the prior password value to detect when you want to rotate the secret. How do you trigger Terraform to send a new `password_wo` value to the cloud provider?
- A. Delete the database and recreate it from scratch
- B. Run `terraform apply -refresh-only`
- C. Run `terraform force-unlock`
- D. Increment the companion integer attribute `password_wo_version` (e.g., from `1` to `2`) in the resource block

---

### Question 5 (Objective 4h — New in 004!)
An engineer declares an input variable with `ephemeral = true`:
```hcl
variable "vault_token" {
  type      = string
  ephemeral = true
}
```
Where is `var.vault_token` permitted to be referenced?
- A. Nowhere; `ephemeral` is only valid on `output` blocks
- B. Inside any regular persistent attribute of a managed resource (e.g., `tags = { Token = var.vault_token }`)
- C. Only in contexts that are not persisted to state or plan files, such as a `provider` block, a `provisioner` connection block, a resource write-only argument (`*_wo`), or another `ephemeral` block/variable/output
- D. Inside `.terraform.lock.hcl`

---

### Question 6 (Objective 4c)
Given the following variable definitions for `var.region`:
- `export TF_VAR_region="us-west-1"`
- `terraform.tfvars`: `region = "us-east-1"`
- `prod.auto.tfvars`: `region = "eu-west-1"`
- CLI command: `terraform plan -var="region=ap-south-1"`

Which value is used during `terraform plan`?
- A. `"eu-west-1"`
- B. `"ap-south-1"`
- C. `"us-east-1"`
- D. `"us-west-1"`

---

### Question 7 (Objective 4f)
You have an application server `aws_instance.app` that downloads packages from the internet during its initial boot script (`user_data`). It sits in a public subnet, but `aws_instance.app` does not reference `aws_internet_gateway.igw` in any of its attributes. How do you ensure Terraform creates and attaches `aws_internet_gateway.igw` **before** booting `aws_instance.app`?
- A. Run `terraform apply` in single-threaded mode.
- B. Place the `aws_internet_gateway.igw` block at the top of `main.tf` above `aws_instance.app`.
- C. Add `lifecycle { create_before_destroy = true }` to the internet gateway.
- D. Add `depends_on = [aws_internet_gateway.igw]` inside the `aws_instance.app` resource block.

---

### Question 8 (Objective 4e)
A developer has a variable `variable "subnet_ids" { type = list(string) }` and wants to iterate over it using `for_each` inside an `aws_instance` resource block. What must the developer do?
- A. Convert the list to a set of strings using `for_each = toset(var.subnet_ids)`, because `for_each` only accepts a `map` or a `set of strings`.
- B. Pass `for_each = var.subnet_ids` directly, because `for_each` natively accepts lists.
- C. Use a `dynamic` block to create multiple top-level `resource` blocks.
- D. Use `count = var.subnet_ids`.

---

### Question 9 (Objective 4d)
How do you declare and reference a local value named `common_prefix` in HCL?
- A. Declare inside `variable "common_prefix" {}` and reference as `local.common_prefix`
- B. Declare inside `output "common_prefix" {}` and reference as `var.common_prefix`
- C. Declare inside `locals { common_prefix = "prod" }` and reference as `local.common_prefix`
- D. Declare inside `local { common_prefix = "prod" }` and reference as `locals.common_prefix`

---

### Question 10 (Objective 4a & 4b)
What is the exact HCL expression to reference the `arn` attribute of an existing IAM role fetched via `data "aws_iam_role" "eks_admin" { name = "EKSAdminRole" }`?
- A. `module.aws_iam_role.eks_admin.arn`
- B. `aws_iam_role.eks_admin.arn`
- C. `data.aws_iam_role.eks_admin.arn`
- D. `ephemeral.aws_iam_role.eks_admin.arn`

---

## Answer Key & Rationales

1. **C** — Top-level `check` blocks (introduced in TF 1.5 and tested on 004 Objective `4g`) execute `assert` conditions at the end of `plan`/`apply` and emit **Warnings only** without failing or halting the run.
2. **B** — Unlike `check` blocks, a failed `postcondition` (or `precondition`) inside a `lifecycle` block raises a **hard error** and immediately blocks dependent resources from running.
3. **A** — `ephemeral` blocks/variables and write-only (`_wo`) arguments solve the long-standing security challenge where `sensitive = true` secrets were still stored in plaintext inside `terraform.tfstate` and binary plan files.
4. **D** — Because write-only arguments (`*_wo`) store `null` in state, they are paired with a version integer (`*_wo_version`, e.g., `password_wo_version = 2`) stored in state. Incrementing the version integer tells Terraform to re-send the write-only secret to the provider.
5. **C** — Ephemeral values cannot be stored in state, so Terraform enforces at `validate`/`plan` time that they are only passed to non-persisted targets (`provider` configs, `provisioner` connections, `*_wo` write-only arguments, or other `ephemeral` objects).
6. **B** — CLI `-var` and `-var-file` flags sit at the very top of the variable precedence hierarchy (Priority 5), overriding `.auto.tfvars` (4), `terraform.tfvars` (2), and `TF_VAR_*` environment variables (1).
7. **D** — Order of blocks in `.tf` files has zero effect on execution order. When no direct attribute reference exists, use `depends_on = [aws_internet_gateway.igw]` to declare an explicit dependency.
8. **A** — `for_each` on a `resource` or `data` block strictly requires a `map` or a `set of strings`. Passing `toset(var.subnet_ids)` converts `list(string)` to `set(string)`.
9. **C** — Local values are declared in a plural `locals { ... }` block and referenced using the singular `local.<NAME>` prefix.
10. **C** — Data sources are always referenced with the `data.` prefix: `data.<TYPE>.<NAME>.<ATTRIBUTE>`.
