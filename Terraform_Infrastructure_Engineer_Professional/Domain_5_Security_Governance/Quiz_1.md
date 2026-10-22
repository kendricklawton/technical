# Domain 5: Terraform Security and Governance — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An engineer marks a variable as `sensitive = true` in Terraform:
```hcl
variable "token" {
  type      = string
  sensitive = true
}
```
Does this prevent the token value from being written in plaintext inside the remote `.tfstate` file?
- A. Yes, `sensitive = true` encrypts the value in the state file using AES-256.
- B. No; `sensitive = true` only redacts the value in CLI output (`terraform plan` and `terraform apply`). The raw value is still written in plaintext inside the `.tfstate` JSON document unless the variable is declared as `ephemeral = true` or stored in an encrypted backend.
- C. Yes, Terraform deletes the value from state immediately after apply.
- D. It hashes the token using SHA-256 in state.

---

### Question 2
What is the function of the `ephemeral = true` variable attribute introduced in modern Terraform?
- A. The variable is automatically deleted after 24 hours.
- B. It guarantees that the value exists only in memory during graph evaluation and is strictly prohibited from being persisted to state files or saved plan files on disk.
- C. It allows the variable to be shared across all workspaces.
- D. It turns the variable into a random string generator.

---

### Question 3
In HCP Terraform, what is the key operational difference between a **Soft Mandatory** Sentinel policy and a **Hard Mandatory** Sentinel policy?
- A. Soft Mandatory policies only apply to staging; Hard Mandatory policies only apply to production.
- B. Soft Mandatory policies halt the run on violation, but allow users with override permissions (e.g. SecOps) to approve and proceed; Hard Mandatory policies cannot be overridden under any circumstances and immediately terminate the run.
- C. Soft Mandatory runs in advisory mode only.
- D. Hard Mandatory policies cost more money.

---

### Question 4
An infrastructure team needs to generate short-lived, dynamic PostgreSQL database credentials for each deployment rather than hardcoding static admin passwords. Which tool natively integrates with Terraform to issue rotated, temporary database credentials on-demand?
- A. HashiCorp Vault
- B. Docker Compose
- C. NGINX
- D. Git LFS

---

### Question 5
When writing Open Policy Agent (OPA) policies in Rego to inspect planned infrastructure, what command converts a binary plan file (`tfplan`) into the JSON input format required by OPA?
- A. `terraform export -json`
- B. `terraform show -json tfplan`
- C. `terraform plan -rego`
- D. `terraform convert tfplan.json`

---

### Question 6
Which of the following configurations violates the CIS Infrastructure Security Benchmark for cloud networks?
- A. Security group allowing inbound TCP 443 from `0.0.0.0/0`.
- B. Security group allowing inbound TCP 22 (SSH) from `0.0.0.0/0`.
- C. Private subnets without Internet Gateways.
- D. Storage buckets with server-side encryption enabled.

---

### Question 7
In Terraform 1.11+, how do **write-only arguments** enhance provider security?
- A. They allow write access without read access on cloud IAM.
- B. They enable providers to send sensitive values directly to remote APIs without Terraform storing those values in the state schema.
- C. They prevent engineers from typing commands in the CLI.
- D. They disable all `terraform destroy` commands.

---

### Question 8
What security risk arises if an engineer creates an S3 remote backend bucket without configuring `aws_s3_bucket_public_access_block`?
- A. State files cannot be locked.
- B. An accidental misconfiguration or overly permissive IAM policy could expose sensitive credentials, API keys, and infrastructure topology contained in state to the public internet.
- C. Terraform will refuse to run `init`.
- D. DynamoDB tables will be deleted.

---

### Question 9
How should IAM policies for CI/CD Terraform service roles be designed according to the principle of least privilege?
- A. Grant `AdministratorAccess` to ensure builds never fail due to permission errors.
- B. Scope permissions strictly to the specific resource actions (e.g. `ec2:RunInstances`, `s3:CreateBucket`) and resource ARNs required by that specific workspace.
- C. Grant read-only access to all resources.
- D. Assign the `root` account credentials to the runner.

---

### Question 10
What tool can be executed in local developer pre-commit hooks to automatically scan HCL code for security misconfigurations (such as unencrypted buckets or open security groups) before code is pushed to Git?
- A. Trivy, tfsec, or Checkov
- B. Webpack
- C. ESLint
- D. Kubernetes Kustomize

---

## Answer Key & Explanations

1. **B** — `sensitive = true` is purely an output masking tool for CLI logs. It provides zero protection against state leakage; state encryption or `ephemeral = true` is required.
2. **B** — `ephemeral` ensures sensitive data vanishes after execution and is never saved to the state file.
3. **B** — Soft Mandatory policies enforce a governance gate that can be bypassed by authorized personnel with explicit audit logging; Hard Mandatory policies strictly block.
4. **A** — HashiCorp Vault dynamically generates database credentials with automatic Time-to-Live (TTL) expiration.
5. **B** — `terraform show -json <planfile>` outputs the complete machine-readable plan representation.
6. **B** — Exposing administrative SSH (port 22) or RDP (port 3389) to the entire internet (`0.0.0.0/0`) breaches standard security benchmarks.
7. **B** — Write-only arguments prevent state persistence for provider-supported sensitive attributes.
8. **B** — State files are high-value targets containing complete architecture blueprints and embedded secrets.
9. **B** — Scoped, least-privilege IAM policies contain the blast radius if CI runner credentials are leaked.
10. **A** — Static application security testing (SAST) tools like Trivy, tfsec, and Checkov analyze HCL for known security anti-patterns.
