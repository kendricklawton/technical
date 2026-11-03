# Domain 3: Core Terraform Workflow — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 3b)
Which of the following actions is **NOT** performed when you execute `terraform init`?
- A. Downloading required provider plugins into `.terraform/providers/`
- B. Initializing the configured state backend
- C. Downloading child modules referenced in `module` blocks
- D. Provisioning initial VPC and IAM resources in the target cloud account

---

### Question 2 (Objective 3c)
A DevOps engineer wants to run `terraform validate` in an isolated CI container that has no cloud credentials and no network access to the remote S3 state bucket. How should the CI pipeline initialize the directory so `terraform validate` succeeds?
- A. Run `terraform validate` directly without running `terraform init`
- B. Run `terraform init -backend=false` followed by `terraform validate`
- C. Run `terraform apply -validate-only`
- D. `terraform validate` always requires valid AWS IAM credentials and remote state access

---

### Question 3 (Objective 3d)
When reviewing the output of `terraform plan`, you see the symbol `+/-` next to `aws_launch_template.app`. What does `+/-` indicate compared to `-/+`?
- A. `+/-` indicates the resource will be updated in-place without replacement.
- B. `+/-` indicates the replacement resource will be **created first and then the old resource destroyed** (due to `create_before_destroy = true`), whereas `-/+` destroys the old resource before creating the replacement.
- C. `+/-` indicates the resource is being imported into state.
- D. `+/-` indicates a data source read operation.

---

### Question 4 (Objective 3d & 3e)
Why is running `terraform plan -out=prod.tfplan` followed by `terraform apply prod.tfplan` considered the gold standard for production CI/CD pipelines over running `terraform apply -auto-approve`?
- A. Because `-auto-approve` is deprecated in Terraform 1.12.
- B. Because saving a binary plan file guarantees that `terraform apply prod.tfplan` executes the **exact** set of changes reviewed and approved during the plan stage, preventing any out-of-band state or variable changes from sneaking in between plan and apply.
- C. Because `prod.tfplan` encrypts the cloud provider's root account password.
- D. Because `terraform apply prod.tfplan` still prompts for an interactive `yes` confirmation.

---

### Question 5 (Objective 3e)
During a `terraform apply` that plans to create 6 resources, 4 resources are created successfully, but the 5th resource (`aws_db_instance.analytics`) fails because the requested instance class is unavailable in that Availability Zone. What does Terraform do?
- A. Automatically rolls back and destroys the 4 resources that were just created.
- B. Marks the 4 successfully created resources in the state file, halts execution of the remaining dependent resources, and outputs the error message.
- C. Deletes the state file so the engineer can start fresh.
- D. Automatically switches the database instance class to a smaller size and retries.

---

### Question 6 (Objective 3g)
You are configuring a GitHub Actions pull request check that must fail the build if any developer commits unformatted `.tf` files in either the root module or any nested `./modules/` subdirectory, without modifying files in the runner. Which command should you use?
- A. `terraform validate -recursive`
- B. `terraform fmt -check -recursive`
- C. `terraform fmt -write=true`
- D. `terraform plan -fmt`

---

### Question 7 (Objective 3f)
An engineer wants to preview the exact list of resources that would be terminated if the environment were decommissioned, without actually deleting anything. Which command accomplishes this?
- A. `terraform plan -destroy`
- B. `terraform destroy -auto-approve`
- C. `terraform state rm --all`
- D. `terraform validate -destroy`

---

### Question 8 (Objective 3e)
An EC2 instance `aws_instance.worker` has become unresponsive at the OS level, even though its AWS API attributes look normal. What is the recommended Terraform 1.12 command to force Terraform to terminate and recreate only that instance during the next apply?
- A. `terraform taint aws_instance.worker`
- B. `terraform apply -replace="aws_instance.worker"`
- C. `terraform state rm aws_instance.worker`
- D. `terraform init -reconfigure`

---

### Question 9 (Objective 3d)
By default, what does Terraform do with the state file at the very beginning of a standard `terraform plan`?
- A. It deletes the remote state file and recreates it after the plan.
- B. It acquires a state lock (if supported by the backend) and refreshes the in-memory state against real-world cloud APIs to detect any drift before calculating the plan diff.
- C. It commits the state file to the active Git branch.
- D. It skips reading state unless `-refresh-only` is passed.

---

### Question 10 (Objective 3f)
Which command is an exact functional alias for `terraform destroy`?
- A. `terraform apply -destroy`
- B. `terraform plan -delete`
- C. `terraform init -clean`
- D. `terraform state purge`

---

## Answer Key & Rationales

1. **D** — `terraform init` prepares the local working directory (backend, modules, providers, lock file); it never provisions or modifies cloud infrastructure.
2. **B** — `terraform validate` requires installed provider schemas and modules (from `init`), but does not need remote state. Running `terraform init -backend=false` initializes plugins/modules without connecting to the remote state backend.
3. **B** — `-/+` is standard destroy-then-create replacement; `+/-` indicates `create_before_destroy = true`, where the replacement is provisioned before the existing resource is destroyed.
4. **B** — Applying a saved plan file (`terraform apply prod.tfplan`) guarantees deterministic execution of the exact previewed changes and runs non-interactively without prompting.
5. **B** — Terraform is **not** an atomic rollback engine. Resources created prior to an error remain active and are recorded in `terraform.tfstate` so the next `terraform apply` can resume cleanly.
6. **B** — `terraform fmt -check -recursive` checks formatting across the current directory and all subdirectories without modifying files, returning a non-zero exit code if any file is unformatted.
7. **A** — `terraform plan -destroy` generates a speculative destruction plan without deleting any resources.
8. **B** — `terraform apply -replace="aws_instance.worker"` is the modern command that replaces the deprecated `terraform taint` subcommand.
9. **B** — Standard `terraform plan` locks the backend state and queries provider APIs to refresh the in-memory state before computing the execution plan.
10. **A** — `terraform destroy` is a convenience alias for `terraform apply -destroy`.
