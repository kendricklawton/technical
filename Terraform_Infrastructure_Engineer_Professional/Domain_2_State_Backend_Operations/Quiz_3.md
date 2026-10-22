# Domain 2: State & Backend Operations — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
You are running a terraform operation against an S3/DynamoDB remote backend, but the terminal session crashes midway through execution. Subsequent runs fail with `Error acquiring the state lock: ConditionalCheckFailedException`. What is the correct and safe recovery command?
- A. `rm -rf .terraform`
- B. `terraform force-unlock <LOCK-ID>` after verifying that no other engineer or pipeline is actively running.
- C. `terraform state push -force`
- D. Delete the DynamoDB table from AWS console.

---

### Question 2
You need to remove a legacy resource from Terraform management without destroying the actual physical cloud infrastructure. Which command achieves this?
- A. `terraform destroy -target=aws_s3_bucket.legacy`
- B. `terraform state rm aws_s3_bucket.legacy`
- C. `terraform state delete aws_s3_bucket.legacy`
- D. `terraform taint aws_s3_bucket.legacy`

---

### Question 3
In Terraform v1.1+, which block allows engineers to declare that a resource was moved or renamed inside the configuration without modifying the state file manually?
- A. `import {}`
- B. `moved {}`
- C. `alias {}`
- D. `replace {}`

---

### Question 4
What happens if you run `terraform state pull > state.json`, edit `state.json` to change a resource attribute, and run `terraform state push state.json` without incrementing `serial`?
- A. Terraform automatically increments the serial.
- B. Terraform rejects the push with a state lineage or serial mismatch error unless `-force` is supplied.
- C. The physical cloud infrastructure updates instantly.
- D. DynamoDB deletes the lock table.

---

### Question 5
You are migrating an existing remote state from an S3 backend to Terraform Cloud / HCP Terraform. Which command initiates the migration process?
- A. `terraform refresh`
- B. `terraform init -migrate-state`
- C. `terraform apply -migrate`
- D. `terraform backend push`

---

### Question 6
Why does standard Terraform state locking NOT prevent state corruption if two engineers run `terraform apply` using a backend that does not support locking (such as standard HTTP without lock headers or basic local file storage)?
- A. Terraform automatically switches to Git for locking.
- B. The second apply will overwrite the state file written by the first apply, causing a race condition and losing tracked resource metadata.
- C. The cloud provider rejects the API calls.
- D. The Terraform binary exits with an error code.

---

### Question 7
In Terraform v1.5+, what block syntax allows importing existing cloud infrastructure into Terraform state declaratively during `terraform plan`?
- A. `resource_import {}`
- B. `import { to = aws_s3_bucket.example, id = "my-bucket-name" }`
- C. `moved {}`
- D. `fetch {}`

---

### Question 8
When using `terraform_remote_state` data source to read outputs from a networking state file, what is a key architectural risk?
- A. It creates an explicit tight coupling between workspaces and requires the consumer workspace to have read access to the entire remote state file, exposing all sensitive data contained in that state.
- B. It causes AWS bill charges for every plan.
- C. It only works with Terraform 0.11.
- D. It deletes the producer workspace.

---

### Question 9
What is the effect of running `terraform refresh` (or running `terraform plan -refresh-only`)?
- A. It destroys all tainted resources.
- B. It queries the cloud provider APIs to update the state file with the current real-world status of managed infrastructure without proposing changes to configuration.
- C. It deletes local lock files.
- D. It downloads new provider plugins.

---

### Question 10
Under what condition will `terraform plan` fail with `Lineage mismatch`?
- A. When you try to push a state file that originated from a completely different state initialization (different `lineage` UUID).
- B. When you have unused variables in `variables.tf`.
- C. When you use AWS provider v5.0.
- D. When the state file contains more than 100 resources.

---

## Answer Key & Explanations

1. **B** — `terraform force-unlock <LOCK-ID>` releases a stale lock once it is confirmed no execution process is holding it.
2. **B** — `terraform state rm` deletes the resource from state tracking without invoking the provider's `Delete` API.
3. **B** — The declarative `moved { from = ... to = ... }` block records refactoring in code, avoiding manual CLI state interventions.
4. **B** — Remote backends compare `lineage` and `serial` numbers to reject stale or concurrently modified state uploads unless `-force` is specified.
5. **B** — Changing the backend block and running `terraform init -migrate-state` prompts Terraform to copy existing state to the new backend.
6. **B** — Without locking, concurrent writes cause last-write-wins collisions, corrupting or wiping state entries.
7. **B** — Declarative `import` blocks (`import { to = ... id = ... }`) generate configuration and import state during standard plan/apply workflows.
8. **A** — `terraform_remote_state` exposes the entire target state file (including all sensitive outputs and resource attributes) to the consumer, violating least privilege.
9. **B** — Refresh queries external APIs to reconcile drift between state metadata and actual cloud infrastructure.
10. **A** — `lineage` is a unique UUID generated at backend initialization; mismatched lineages indicate distinct, unrelated state histories.
