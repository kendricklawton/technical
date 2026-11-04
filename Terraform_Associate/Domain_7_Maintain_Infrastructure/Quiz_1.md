# Domain 7: Maintain Infrastructure with Terraform — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 7a)
An engineer wants to import an existing AWS VPC (`vpc-0987654321fedcba0`) into Terraform 1.12 using a declarative `import` block and wants Terraform to automatically generate the HCL `resource "aws_vpc"` block so they don't have to write it from scratch. What steps should the engineer take?
- A. Terraform can never generate `.tf` configuration files from existing cloud resources.
- B. Add `import { to = aws_vpc.main id = "vpc-0987654321fedcba0" }` to a `.tf` file and run `terraform plan -generate-config-out=generated_vpc.tf`.
- C. Run `terraform import -generate-code aws_vpc.main vpc-0987654321fedcba0`.
- D. Run `terraform state pull > generated_vpc.tf`.

---

### Question 2 (Objective 7a)
How does the declarative `import` block (`import { to = ... id = ... }`) differ operationally from the imperative `terraform import <ADDRESS> <ID>` CLI command?
- A. `terraform import` supports `for_each`, whereas the `import` block does not.
- B. The `import` block is evaluated as part of the standard `terraform plan` and `terraform apply` workflow (allowing PR review and speculative plans before mutating state), whereas `terraform import` immediately modifies the state file without a prior plan stage.
- C. The `import` block deletes the existing cloud resource and recreates it, whereas `terraform import` preserves it.
- D. The `import` block works only with the local backend.

---

### Question 3 (Objective 7a)
If you choose to use the legacy imperative CLI command `terraform import aws_instance.bastion i-0123456789abcdef0` instead of an `import` block, what must you do **before** running the `terraform import` command?
- A. Delete your `terraform.tfstate` file.
- B. Stop the EC2 instance in the AWS Console.
- C. Run `terraform destroy`.
- D. Manually write a `resource "aws_instance" "bastion" { ... }` block in your `.tf` configuration file.

---

### Question 4 (Objective 7b)
You want to quickly view the list of all resource addresses currently tracked in your workspace's state file without printing every single attribute value. Which command should you run?
- A. `terraform state list`
- B. `terraform state show`
- C. `terraform output`
- D. `terraform state pull`

---

### Question 5 (Objective 7b)
An engineer needs to inspect all recorded state attributes (including computed ARNs, subnet IDs, and security group IDs) for a specific resource instance `aws_instance.app["web-1"]`. Which command displays this information?
- A. `terraform state list aws_instance.app["web-1"]`
- B. `terraform output aws_instance.app["web-1"]`
- C. `terraform state show 'aws_instance.app["web-1"]'`
- D. `terraform validate aws_instance.app["web-1"]`

---

### Question 6 (Objective 7b)
What happens to the real-world Amazon S3 bucket in your AWS account when you run `terraform state rm aws_s3_bucket.customer_uploads`?
- A. The S3 bucket is marked for replacement on the next `terraform apply`.
- B. The S3 bucket is moved into a new CLI workspace named `orphaned`.
- C. The S3 bucket is removed from Terraform's state file so Terraform no longer tracks or manages it, while the actual S3 bucket in AWS remains completely untouched.
- D. The S3 bucket and all objects inside it are immediately deleted from AWS.

---

### Question 7 (Objective 7c)
Which `TF_LOG` environment variable setting produces the **highest level of verbosity** when debugging Terraform execution?
- A. `TF_LOG=VERBOSE`
- B. `TF_LOG=DEBUG`
- C. `TF_LOG=TRACE`
- D. `TF_LOG=INFO`

---

### Question 8 (Objective 7c)
An engineer sets `export TF_LOG_PATH="/var/log/terraform.log"` in their Linux shell, but does **not** set `TF_LOG`, `TF_LOG_CORE`, or `TF_LOG_PROVIDER`. When they run `terraform apply`, what is written to `/var/log/terraform.log`?
- A. Nothing; `TF_LOG_PATH` only specifies where logs are written *when* logging is enabled via `TF_LOG` (or `TF_LOG_CORE` / `TF_LOG_PROVIDER`).
- B. A copy of `terraform.tfstate` is written to the file.
- C. Full `TRACE` logs are written automatically.
- D. Terraform fails with an invalid environment variable error.

---

### Question 9 (Objective 7c)
You suspect a bug in the AzureRM provider's REST API payload serialization rather than in Terraform Core's graph engine. Which environment variable lets you enable `TRACE` logging **exclusively for provider plugins** without flooding the log with internal Terraform Core trace messages?
- A. `TF_LOG_PROVIDER=TRACE`
- B. `TF_LOG=PROVIDER_ONLY`
- C. `TF_LOG_PLUGIN=TRACE`
- D. `TF_PROVIDER_DEBUG=1`

---

### Question 10 (Objective 7b)
Why does HashiCorp strongly advise against manually opening `terraform.tfstate` in a text editor to rename resources or fix state discrepancies?
- A. Because editing `terraform.tfstate` revokes your HashiCorp certification.
- B. Because `terraform.tfstate` is compiled into an unreadable binary format.
- C. Because text editors cannot open files larger than 1 MB.
- D. Because manual edits bypass state locking, risk corrupting the JSON schema or `serial`/`lineage` integrity checks, and can cause catastrophic unintended infrastructure destruction; operators should always use `moved`/`removed` blocks or `terraform state` CLI commands instead.

---

## Answer Key & Rationales

1. **B** — Combining a declarative `import` block with `terraform plan -generate-config-out=generated_vpc.tf` instructs Terraform to query the existing cloud resource and write the matching HCL `resource` block to the specified file.
2. **B** — The declarative `import` block integrates directly into the `plan` $\rightarrow$ `apply` lifecycle (and supports `for_each` and `-generate-config-out`), whereas `terraform import` immediately mutates the state file.
3. **D** — Before running the imperative `terraform import <ADDR> <ID>` command, a matching `resource` block must exist in your `.tf` files so Terraform has a configuration target to bind the state record to.
4. **A** — `terraform state list` prints a concise list of all resource addresses tracked in the state file.
5. **C** — `terraform state show <ADDRESS>` prints all state attributes of the specified resource. (Note: quote addresses containing square brackets in bash/zsh so the shell does not interpret `[]` as glob wildcards.)
6. **C** — `terraform state rm` removes the resource binding from `terraform.tfstate` only; it makes zero API calls to delete the real-world resource in the cloud.
7. **C** — `TRACE` is the most verbose logging level (`TRACE` > `DEBUG` > `INFO` > `WARN` > `ERROR`).
8. **A** — `TF_LOG_PATH` requires `TF_LOG` (or `TF_LOG_CORE` / `TF_LOG_PROVIDER`) to be set to a valid log level before any log output is generated.
9. **A** — `TF_LOG_PROVIDER` controls logging verbosity specifically for provider plugins, while `TF_LOG_CORE` controls logging specifically for Terraform Core.
10. **D** — Hand-editing `terraform.tfstate` risks JSON syntax errors, `serial`/`lineage` mismatches, and race conditions without state locking.
