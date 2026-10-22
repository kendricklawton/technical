# Domain 5: Terraform Security and Governance — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Governance & Compliance Architecture Questions

---

### Question 1: Mitigating Blast Radius in Microservice Platforms
A company runs 80 microservices on a shared Kubernetes platform. If all 80 microservices and the Kubernetes cluster itself are managed in a single Terraform workspace:
- A. The platform will achieve maximum performance.
- B. Any syntax error, state lock contention, or slow cloud API response during an update to a single microservice blocks deployments for all other 79 services, and an accidental destructive change could delete the entire cluster.
- C. AWS automatically splits the state file.
- D. There is no operational risk.

---

### Question 2: Auditing Out-of-Band State Access
How can an enterprise detect if an unauthorized internal user downloaded a production Terraform state file from an S3 backend?
- A. By enabling AWS CloudTrail S3 Data Events logging on the state bucket and monitoring for `GetObject` API calls.
- B. By checking the `.terraform.lock.hcl` file.
- C. S3 logs downloads to the local terminal automatically.
- D. By running `terraform state list`.

---

### Question 3: Enforcing Mandatory Tags with Sentinel
An organization mandates that every provisionable resource must contain tags `Environment`, `Owner`, and `CostCenter`. At what lifecycle phase in HCP Terraform does a Sentinel policy validate these tags?
- A. During `terraform init`.
- B. Post-plan, pre-apply (blocking the run before any cloud resources are created if tags are missing).
- C. After the resources are deployed.
- D. During git commit.

---

### Question 4: Transient Data Sources
What is the primary security benefit of using the `ephemeral` block for data sources introduced in Terraform 1.10+?
- A. It bypasses network firewalls.
- B. Secrets retrieved from external secret stores (such as Vault or AWS Secrets Manager) during execution are never persisted to the plan file or state file.
- C. It makes API calls 10x cheaper.
- D. It deletes the secret from Vault after reading.

---

### Question 5: Preventing Accidental S3 Bucket Public Access
Which Terraform resource configuration prevents an S3 bucket from ever serving public traffic regardless of bucket policies?
- A.
```hcl
resource "aws_s3_bucket_public_access_block" "block" {
  bucket                  = aws_s3_bucket.main.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```
- B. `acl = "public-read"`
- C. `lifecycle { prevent_destroy = true }`
- D. `force_destroy = false`

---

### Question 6: Ephemeral Variable Constraints
Can an `ephemeral = true` variable be referenced in an output block that is NOT marked as `ephemeral = true`?
- A. Yes, outputs ignore variable mutability.
- B. No; if an output depends on an ephemeral value, the output itself must also be marked as `ephemeral = true` to prevent state leakage.
- C. Ephemeral variables cannot be referenced in outputs under any circumstances.
- D. It depends on the cloud provider.

---

### Question 7: Hardening State Lock Tables in DynamoDB
What IAM permission should be strictly granted to CI/CD runners to interact with the DynamoDB lock table without allowing full table deletion?
- A. `dynamodb:*`
- B. Scoped permissions: `dynamodb:GetItem`, `dynamodb:PutItem`, and `dynamodb:DeleteItem` restricted strictly to the table ARN.
- C. `dynamodb:DeleteTable`
- D. `dynamodb:Scan` only

---

### Question 8: Secrets in Remote Execution Environments
In HCP Terraform, what happens when an environment variable is marked as "Sensitive"?
- A. Its value is hidden in the web UI and API responses, and write-only access is enforced for users.
- B. It cannot be used in plans.
- C. It is deleted after 1 hour.
- D. It is sent via email.

---

### Question 9: Automated Guardrails with Run Tasks
What is an HCP Terraform "Run Task"?
- A. A bash script that reboots the runner.
- B. An integration point that passes the speculative plan to an external third-party security or compliance vendor (e.g. Snyk, Infracost, Wiz) and awaits a pass/fail callback before allowing apply.
- C. A cron job that runs `terraform init`.
- D. A task that deletes unused workspaces.

---

### Question 10: Inadvertent Privilege Escalation via IAM Roles
An engineer authors an IAM role for an EC2 instance with `iam:PassRole` and `ec2:RunInstances` on `*`. Why does a security linter flag this as a critical finding?
- A. It allows anyone on that EC2 instance to launch a new instance with administrative roles, effectively escalating privileges to full cloud administrator.
- B. It causes network loops.
- C. It violates GDPR formatting.
- D. EC2 does not support IAM roles.

---

## Answer Key & Explanations

1. **B** — Co-locating 80 microservices in a single monolithic state file creates massive blast radius risk and severe deployment bottlenecks.
2. **A** — CloudTrail S3 Data Events provide an immutable audit trail of every `GetObject` call targeting the state bucket.
3. **B** — Policy as Code evaluates after the plan is computed but before any state or cloud resources are mutated.
4. **B** — Ephemeral data sources ensure that temporary credentials exist only in memory during execution.
5. **A** — `aws_s3_bucket_public_access_block` enforces cloud-level overrides blocking public ACLs and policies.
6. **B** — Ephemeral propagation rules require downstream outputs depending on ephemeral values to also be marked `ephemeral = true`.
7. **B** — Least privilege requires granting only read, write, and delete item permissions on the specific lock table.
8. **A** — Sensitive variables in HCP Terraform are write-only in the UI/API.
9. **B** — Run Tasks integrate external SAST, compliance, and cost engines into the HCP Terraform run pipeline.
10. **A** — `iam:PassRole` without resource ARN constraints is a well-known privilege escalation vector in cloud environments.
