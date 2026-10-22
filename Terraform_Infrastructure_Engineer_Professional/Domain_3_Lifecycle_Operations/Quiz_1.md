# Domain 3: Infrastructure Lifecycle and Operations — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An engineer updates the AMI ID of an auto-scaling group launch template. The launch template has `create_before_destroy = true`. However, the security group referenced by the launch template does NOT have `create_before_destroy = true`. When the security group is also updated, what error occurs?
- A. The plan passes without errors.
- B. A cycle error or dependency inversion error: Terraform cannot destroy the old security group until the old launch template is destroyed, but the new launch template requires the new security group first. To fix this, any dependency of a `create_before_destroy` resource must also have `create_before_destroy = true`.
- C. AWS automatically duplicates the security group.
- D. Terraform ignores the security group update.

---

### Question 2
What happens if you run `terraform destroy` against a configuration containing a resource with `lifecycle { prevent_destroy = true }`?
- A. Terraform ignores `prevent_destroy` during a full destroy.
- B. Terraform prompts for an administrative password.
- C. Terraform immediately halts execution during the planning phase with an error, preventing any destruction.
- D. Terraform deletes all resources except the protected one.

---

### Question 3
An engineer uses a declarative `import` block in Terraform 1.5+:
```hcl
import {
  to = aws_s3_bucket.logs
  id = "prod-corp-access-logs-2026"
}
```
What CLI flag can be passed to `terraform plan` to automatically generate the corresponding resource HCL code if the file does not yet exist?
- A. `-auto-import`
- B. `-generate-config-out=bucket.tf`
- C. `-write-hcl`
- D. `-export-resource`

---

### Question 4
In Kubernetes environments, a Pod Autoscaler constantly alters the `desired_capacity` of an AWS Auto Scaling Group. Every time the CI pipeline runs `terraform plan`, it reports that `desired_capacity` has drifted. How do you prevent Terraform from constantly reverting the autoscaler's changes?
- A. Set `desired_capacity = null`.
- B. Add `lifecycle { ignore_changes = [desired_capacity] }` to the Auto Scaling Group resource.
- C. Delete the Auto Scaling Group from state.
- D. Remove the autoscaler from the Kubernetes cluster.

---

### Question 5
Why should saved plan files (`terraform plan -out=tfplan`) always be used when running Terraform in automated CI/CD pipelines?
- A. Saved plan files are encrypted with RSA 4096-bit keys.
- B. It guarantees that the exact changes reviewed and approved during the PR plan are applied during the deployment step, eliminating race conditions from concurrent changes or external drift.
- C. Saved plan files execute 50% faster.
- D. Plan files automatically roll back on failure.

---

### Question 6
What is the purpose of the `.terraform.lock.hcl` file in source control?
- A. It locks the state file in S3.
- B. It pins exact provider versions and cryptographic package hashes (checksums) across developer workstations and CI runners.
- C. It prevents git merge conflicts.
- D. It stores API access tokens.

---

### Question 7
You have multiple target operating systems in your engineering team (macOS ARM64, Linux AMD64, Windows). When initializing a new provider, only the local architecture's checksum is recorded in `.terraform.lock.hcl`. What command updates the lock file with checksums for all supported OS architectures?
- A. `terraform init -all-platforms`
- B. `terraform providers lock -platform=linux_amd64 -platform=darwin_arm64 -platform=windows_amd64`
- C. `terraform lock refresh`
- D. `terraform get -update`

---

### Question 8
What is the effect of setting `destroy = false` inside a `removed` block?
- A. Terraform deletes the resource from cloud and recreates it.
- B. Terraform removes the resource from the `.tfstate` file, leaving the real infrastructure running untouched in the cloud.
- C. Terraform permanently prevents the resource from ever being destroyed.
- D. Terraform sets the resource to read-only mode.

---

### Question 9
An infrastructure pipeline fails with `HTTP 429 Too Many Requests` when creating 200 DNS records concurrently. What Terraform CLI flag addresses this directly?
- A. `-timeout=60s`
- B. `-parallelism=5` (reducing concurrent graph walking operations from default 10)
- C. `-retry=true`
- D. `-slow-mode`

---

### Question 10
Which `lifecycle` meta-argument allows you to trigger the recreation of a virtual machine when its bootstrap script hash changes?
- A. `create_before_destroy`
- B. `replace_triggered_by`
- C. `precondition`
- D. `ignore_changes`

---

## Answer Key & Explanations

1. **B** — The `create_before_destroy` attribute must be propagated to upstream dependencies; otherwise, Terraform cannot order the creation of new resources ahead of destroying existing dependencies.
2. **C** — `prevent_destroy` acts as a fail-safe that rejects any plan attempting to destroy the resource.
3. **B** — `-generate-config-out=PATH` inspects the imported resource via provider APIs and synthesizes the HCL configuration.
4. **B** — `ignore_changes` instructs Terraform's plan engine to ignore differences between state and code for specified attributes.
5. **B** — Specifying a plan file ensures deterministic, atomic application of the exact recorded plan.
6. **B** — `.terraform.lock.hcl` acts as a security supply-chain barrier by pinning exact provider releases and hashes.
7. **B** — `terraform providers lock -platform=...` fetches package hashes for multiple platform targets.
8. **B** — Declarative `removed { lifecycle { destroy = false } }` removes resources from state safely.
9. **B** — `-parallelism=N` limits concurrent API operations to respect cloud rate limits.
10. **B** — `replace_triggered_by` forces a replacement plan when referenced objects change.
