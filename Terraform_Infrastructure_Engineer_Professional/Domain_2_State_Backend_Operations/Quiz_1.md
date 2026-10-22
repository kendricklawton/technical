# Domain 2: State and Backend Operations — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
During a scheduled CI/CD apply, the runner container is forcefully killed due to an out-of-memory (OOM) error while applying changes to an AWS RDS database. Subsequent pipeline runs fail with `Error acquiring the state lock: ConditionalCheckFailedException`. What is the correct procedure to recover?
- A. Delete the S3 state file and re-import all resources.
- B. Run `terraform force-unlock <Lock-ID>` using the Lock ID displayed in the error message, verify state integrity with `terraform state list`, and re-run the plan.
- C. Run `terraform init -reconfigure` to override the lock.
- D. Delete the DynamoDB table.

---

### Question 2
An engineer notices that an out-of-band change was made to a security group rule in the AWS console. The engineering team agrees that this change was correct and should be kept and reflected in the Terraform state without modifying any infrastructure. Which command achieves this?
- A. `terraform refresh`
- B. `terraform plan -refresh-only` followed by `terraform apply -refresh-only`
- C. `terraform state push`
- D. `terraform import`

---

### Question 3
You want to rename a resource from `aws_instance.web_server` to `aws_instance.frontend_node` in your `.tf` code. If you simply update the code and run `terraform apply`, what will happen?
- A. Terraform renames the instance in place without downtime.
- B. Terraform destroys the running `aws_instance.web_server` and creates a brand-new `aws_instance.frontend_node`.
- C. Terraform returns an error stating the resource already exists.
- D. Terraform updates the tags on the instance.

---

### Question 4
Continuing from Question 3, which command allows you to rename the resource in code without causing Terraform to destroy and recreate the instance?
- A. `terraform state mv aws_instance.web_server aws_instance.frontend_node`
- B. `terraform state rm aws_instance.web_server`
- C. `terraform state pull aws_instance.web_server`
- D. `terraform import aws_instance.frontend_node <id>`

---

### Question 5
What is the primary danger of using local state files (`terraform.tfstate`) in a multi-engineer team?
- A. Local state files cannot store more than 10 resources.
- B. Lack of concurrency locking can cause race conditions where two engineers overwrite each other's changes, corrupting state and causing accidental resource deletion.
- C. Local state files cannot be backed up.
- D. Local state does not support variables.

---

### Question 6
You need to inspect the exact JSON state payload stored in the remote backend without checking out code or modifying the backend. Which command outputs the current remote state directly to stdout?
- A. `terraform state list`
- B. `terraform state pull`
- C. `terraform state show`
- D. `terraform show -json`

---

### Question 7
Which of the following backend types does NOT support native state locking?
- A. AWS S3 (without DynamoDB)
- B. Google Cloud Storage (GCS)
- C. HCP Terraform / Terraform Cloud
- D. Azure Blob Storage (azurerm)

---

### Question 8
What does the `terraform state replace-provider` command do?
- A. It downloads the newest provider binary from the registry.
- B. It updates the provider source namespace for resources in the state file (e.g. migrating from `hashicorp/digitalocean` to `digitalocean/digitalocean`).
- C. It deletes all resources managed by a provider.
- D. It replaces AWS resources with GCP equivalents.

---

### Question 9
Why is storing sensitive database passwords in Terraform input variables not sufficient to protect them from unauthorized access if state is unencrypted?
- A. Variables are logged to `/tmp`.
- B. Every resource attribute (including sensitive values) passed to a resource is saved in plaintext in the `.tfstate` JSON file.
- C. Variables are visible in the DNS query logs.
- D. Cloud providers publish state files publicly by default.

---

### Question 10
When running `terraform graph`, what does each node and edge in the resulting Directed Acyclic Graph (DAG) represent?
- A. Nodes represent cloud accounts; edges represent network routes.
- B. Nodes represent resources, data sources, and provider configurations; edges represent direct dependency relationships between them.
- C. Nodes represent Git commits; edges represent branches.
- D. Nodes represent API endpoints; edges represent HTTP status codes.

---

## Answer Key & Explanations

1. **B** — `terraform force-unlock` manually releases an orphaned state lock using the specific lock ID provided by the lock manager.
2. **B** — `terraform plan -refresh-only` and `terraform apply -refresh-only` provide a safe, plan-driven way to update state to match out-of-band changes without risking unintended infrastructure modifications.
3. **B** — Changing a resource's HCL address causes Terraform to treat the old address as removed (destroy) and the new address as unmanaged (create).
4. **A** — `terraform state mv` updates the resource address in the state file so Terraform recognizes that the existing cloud resource maps to the renamed HCL block.
5. **B** — Without distributed state locking, concurrent runs can execute simultaneous plans/applies on stale state, leading to state corruption and resource loss.
6. **B** — `terraform state pull` reads the state from the configured backend and outputs the raw JSON to stdout.
7. **A** — S3 alone does not support locking; it requires a DynamoDB table with a `LockID` primary key. GCS, Azure Blob, and HCP Terraform support native locking.
8. **B** — `terraform state replace-provider` safely updates provider namespaces in state without re-creating resources.
9. **B** — The `sensitive = true` flag prevents values from printing in CLI stdout, but `.tfstate` records all attribute values in plaintext.
10. **B** — Terraform's DAG models all resources and their explicit/implicit dependencies to determine parallel execution order.
