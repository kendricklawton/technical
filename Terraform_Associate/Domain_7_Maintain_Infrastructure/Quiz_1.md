# Domain 7: Maintain Infrastructure with Terraform — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 7a)
An engineer wants to import an existing AWS VPC (`vpc-0987654321fedcba0`) into Terraform 1.12 using a declarative `import` block and wants Terraform to automatically generate the HCL `resource "aws_vpc"` block so they don't have to write it from scratch. What steps should the engineer take?
- A. Run `terraform import -generate-code aws_vpc.main vpc-0987654321fedcba0`.
- B. Add `import { to = aws_vpc.main id = "vpc-0987654321fedcba0" }` to a `.tf` file and run `terraform plan -generate-config-out=generated_vpc.tf`.
- C. Run `terraform state pull > generated_vpc.tf`.
- D. Terraform can never generate `.tf` configuration files from existing cloud resources.

---

### Question 2 (Objective 7a)
How does the declarative `import` block (`import { to = ... id = ... }`) differ operationally from the imperative `terraform import <ADDRESS> <ID>` CLI command?
- A. The `import` block is evaluated as part of the standard `terraform plan` and `terraform apply` workflow (allowing PR review and speculative plans before mutating state), whereas `terraform import` immediately modifies the state file without a prior plan stage.
- B. The `import` block deletes the existing cloud resource and recreates it, whereas `terraform import` preserves it.
- C. `terraform import` supports `for_each`, whereas the `import` block does not.
- D. The `import` block works only with the local backend.

---

### Question 3 (Objective 7a)
If you choose to use the legacy imperative CLI command `terraform import aws_instance.bastion i-0123456789abcdef0` instead of an `import` block, what must you do **before** running the `terraform import` command?
- A. Manually write a `resource "aws_instance" "bastion" { ... }` block in your `.tf` configuration file.
- B. Delete your `terraform.tfstate` file.
- C. Stop the EC2 instance in the AWS Console.
- D. Run `terraform destroy`.

---

### Question 4 (Objective 7b)
You want to quickly view the list of all resource addresses currently tracked in your workspace's state file without printing every single attribute value. Which command should you run?
- A. `terraform state show`
- B. `terraform state list`
- C. `terraform output`
- D. `terraform state pull`

---

### Question 5 (Objective 7b)
An engineer needs to inspect all recorded state attributes (including computed ARNs, subnet IDs, and security group IDs) for a specific resource instance `aws_instance.app["web-1"]`. Which command displays this information?
- A. `terraform state list aws_instance.app["web-1"]`
- B. `terraform state show 'aws_instance.app["web-1"]'`
- C. `terraform validate aws_instance.app["web-1"]`
- D. `terraform output aws_instance.app["web-1"]`

---

### Question 6 (Objective 7b)
What happens to the real-world Amazon S3 bucket in your AWS account when you run `terraform state rm aws_s3_bucket.customer_uploads`?
- A. The S3 bucket and all objects inside it are immediately deleted from AWS.
- B. The S3 bucket is removed from Terraform's state file so Terraform no longer tracks or manages it, while the actual S3 bucket in AWS remains completely untouched.
- C. The S3 bucket is moved into a new CLI workspace named `orphaned`.
- D. The S3 bucket is marked for replacement on the next `terraform apply`.

---

### Question 7 (Objective 7c)
Which `TF_LOG` environment variable setting produces the **highest level of verbosity** when debugging Terraform execution?
- A. `TF_LOG=DEBUG`
- B. `TF_LOG=TRACE`
- C. `TF_LOG=VERBOSE`
- D. `TF_LOG=INFO`

---

### Question 8 (Objective 7c)
An engineer sets `export TF_LOG_PATH="/var/log/terraform.log"` in their Linux shell, but does **not** set `TF_LOG`, `TF_LOG_CORE`, or `TF_LOG_PROVIDER`. When they run `terraform apply`, what is written to `/var/log/terraform.log`?
- A. Full `TRACE` logs are written automatically.
- B. Nothing; `TF_LOG_PATH` only specifies where logs are written *when* logging is enabled via `TF_LOG` (or `TF_LOG_CORE` / `TF_LOG_PROVIDER`).
- C. A copy of `terraform.tfstate` is written to the file.
- D. Terraform fails with an invalid environment variable error.

---

### Question 9 (Objective 7c)
You suspect a bug in the AzureRM provider's REST API payload serialization rather than in Terraform Core's graph engine. Which environment variable lets you enable `TRACE` logging **exclusively for provider plugins** without flooding the log with internal Terraform Core trace messages?
- A. `TF_LOG_PLUGIN=TRACE`
- B. `TF_LOG_PROVIDER=TRACE`
- C. `TF_PROVIDER_DEBUG=1`
- D. `TF_LOG=PROVIDER_ONLY`

---

### Question 10 (Objective 7b)
Why does HashiCorp strongly advise against manually opening `terraform.tfstate` in a text editor to rename resources or fix state discrepancies?
- A. Because `terraform.tfstate` is compiled into an unreadable binary format.
- B. Because manual edits bypass state locking, risk corrupting the JSON schema or `serial`/`lineage` integrity checks, and can cause catastrophic unintended infrastructure destruction; operators should always use `moved`/`removed` blocks or `terraform state` CLI commands instead.
- C. Because editing `terraform.tfstate` revokes your HashiCorp certification.
- D. Because text editors cannot open files larger than 1 MB.

---

## Answer Key & Rationales

1. **B** — Combining a declarative `import` block with `terraform plan -generate-config-out=generated_vpc.tf` instructs Terraform to query the existing cloud resource and write the matching HCL `resource` block to the specified file.
2. **A** — The declarative `import` block integrates directly into the `plan` $\rightarrow$ `apply` lifecycle (and supports `for_each` and `-generate-config-out`), whereas `terraform import` immediately mutates the state file.
3. **A** — Before running the imperative `terraform import <ADDR> <ID>` command, a matching `resource` block must exist in your `.tf` files so Terraform has a configuration target to bind the state record to.
4. **B** — `terraform state list` prints a concise list of all resource addresses tracked in the state file.
5. **B** — `terraform state show <ADDRESS>` prints all state attributes of the specified resource. (Note: quote addresses containing square brackets in bash/zsh so the shell does not interpret `[]` as glob wildcards.)
6. **B** — `terraform state rm` removes the resource binding from `terraform.tfstate` only; it makes zero API calls to delete the real-world resource in the cloud.
7. **B** — `TRACE` is the most verbose logging level (`TRACE` > `DEBUG` > `INFO` > `WARN` > `ERROR`).
8. **B** — `TF_LOG_PATH` requires `TF_LOG` (or `TF_LOG_CORE` / `TF_LOG_PROVIDER`) to be set to a valid log level before any log output is generated.
9. **B** — `TF_LOG_PROVIDER` controls logging verbosity specifically for provider plugins, while `TF_LOG_CORE` controls logging specifically for Terraform Core.
10. **B** — Hand-editing `terraform.tfstate` risks JSON syntax errors, `serial`/`lineage` mismatches, and race conditions without state locking.
