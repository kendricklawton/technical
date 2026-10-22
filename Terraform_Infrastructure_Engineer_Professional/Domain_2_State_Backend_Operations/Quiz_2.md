# Domain 2: State and Backend Operations — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced State Operations & Architecture Questions

---

### Question 1: Cross-Project State Sharing with `terraform_remote_state`
A security engineering team manages all VPCs and KMS keys in a dedicated state file `security/vpc.tfstate`. An application team needs to read the VPC ID and Subnet IDs without having write permissions to the security state. How should the application team configure access?
- A. By importing the VPC resource into their own state file.
- B. By defining a `terraform_remote_state` data source pointing to the security team's remote S3 backend and reading its published `outputs`.
- C. By hardcoding the VPC ID into `terraform.tfvars`.
- D. By combining both state files using `terraform state push`.

---

### Question 2: Blast Radius Reduction
Which of the following architectures represents the best practice for organizing state files in a high-scale enterprise?
- A. A single monolithic `main.tfstate` storing 1,500 resources across 4 AWS regions.
- B. Decoupled workspaces or backend keys separated by lifecycle, team ownership, and blast radius (e.g., `networking`, `identity`, `kubernetes-clusters`, `payment-service`).
- C. Committing `terraform.tfstate` directly into the application's Git repository.
- D. Storing state in local `/tmp` directories on CI runners.

---

### Question 3: Upgrading Terraform Version Across Workspaces
When upgrading an enterprise environment from Terraform v1.8 to v1.15 across 40 state files, what is the safest progression?
- A. Update the CI pipeline runner to v1.15 and apply all workspaces simultaneously.
- B. Upgrade one non-production workspace first, verify provider compatibility, review `terraform plan` output for breaking schema deprecations, run `terraform apply`, test workloads, and then sequentially promote through staging and production.
- C. Run `terraform state rm` on all resources before upgrading.
- D. Delete the `.terraform.lock.hcl` file.

---

### Question 4: State File Corruption Recovery
A junior engineer ran `terraform state push` with a truncated, unvalidated backup file, resulting in an invalid JSON error on subsequent runs. What is the fastest way to recover if the S3 backend has bucket versioning enabled?
- A. Delete the S3 bucket.
- B. Restore the prior S3 object version using the AWS S3 Console or CLI, and verify with `terraform state pull | jq .serial`.
- C. Re-run `terraform init -upgrade`.
- D. Run `terraform force-unlock`.

---

### Question 5: `terraform state rm` Use Case
A company is decommissioning an internal microservice. The engineer wants to remove the Kubernetes deployment from Terraform management because it will now be managed by ArgoCD, but the deployment must NOT be deleted from the live Kubernetes cluster. Which command must the engineer run?
- A. `terraform destroy -target=kubernetes_deployment.app`
- B. `terraform state rm kubernetes_deployment.app`
- C. `terraform state mv kubernetes_deployment.app argocd_app`
- D. `terraform taint kubernetes_deployment.app`

---

### Question 6: Dependency Inversion & Cycles
What happens if Resource A references an attribute of Resource B, and Resource B simultaneously references an attribute of Resource A?
- A. Terraform automatically resolves the reference at runtime.
- B. Terraform throws a `Cycle error` during graph construction and halts execution without modifying infrastructure.
- C. Terraform applies Resource A first and Resource B second.
- D. The cloud provider detects the loop and creates an asynchronous task.

---

### Question 7: Mitigating Cycle Errors with Decoupled Resources
In AWS, an `aws_security_group` resource can declare inline `ingress` rules, or you can use standalone `aws_security_group_rule` resources. If two security groups need to reference each other's IDs, how do you prevent a cycle error?
- A. Define both rules inside the security group blocks.
- B. Create the security groups first without inline rules, and use separate `aws_security_group_rule` resources to link them bidirectionally.
- C. Use `lifecycle { ignore_changes = [ingress] }`.
- D. Security groups cannot reference each other in AWS.

---

### Question 8: Backend Migration with `-reconfigure` vs `-migrate-state`
What is the difference between `terraform init -migrate-state` and `terraform init -reconfigure`?
- A. `-migrate-state` attempts to copy the existing state from the old backend to the new backend; `-reconfigure` ignores existing backend state configuration and initializes the new backend from scratch.
- B. `-reconfigure` deletes all cloud infrastructure.
- C. `-migrate-state` only works with local backends.
- D. There is no difference.

---

### Question 9: Lock Expiry in Remote Backends
Do Terraform backend locks automatically expire after a set time (e.g. 15 minutes)?
- A. Yes, all backends release locks after 10 minutes.
- B. No; in systems like DynamoDB, GCS, and S3, locks are persistent records that remain in place until the applying process releases them or an administrator runs `terraform force-unlock`.
- C. Only HCP Terraform has persistent locks.
- D. Locks only last while the TCP connection is open.

---

### Question 10: State Inspection with Filter (`terraform state list`)
You have 400 resources in state. Which command lists only the DigitalOcean Droplet resources without printing disks, firewalls, or VPCs?
- A. `terraform state list | grep droplet` or `terraform state list digitalocean_droplet`
- B. `terraform show droplet`
- C. `terraform get droplets`
- D. `terraform plan -filter=droplets`

---

## Answer Key & Explanations

1. **B** — The `terraform_remote_state` data source allows cross-workspace consumption of published outputs while preserving strict read-only access boundaries.
2. **B** — Segmenting state by functional layer and lifecycle limits blast radius so a failure in an application rollout cannot corrupt the core VPC.
3. **B** — Systematic promotion across environments with compatibility verification is required for enterprise Terraform version upgrades.
4. **B** — Enabling S3 bucket versioning on state buckets provides point-in-time recovery from accidental state overwrites or corruption.
5. **B** — `terraform state rm` removes the object from Terraform tracking without issuing a delete call to the cloud provider.
6. **B** — Terraform's DAG requires an acyclic graph; circular references cause an immediate cycle error.
7. **B** — Using standalone `aws_security_group_rule` resources breaks the circular dependency because the parent security group IDs exist before the rules are attached.
8. **A** — `-migrate-state` copies state data between backends, whereas `-reconfigure` reinitializes without copying.
9. **B** — State locks are explicit persistent records designed to prevent concurrent writes even across long-running applies.
10. **A** — `terraform state list [address]` filters state resources by address prefix or type.
