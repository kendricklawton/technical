# Domain 5: Terraform Security and Governance — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Governance & Compliance Architecture Questions

---

---

### Question 1
A company runs 80 microservices on a shared Kubernetes platform. If all 80 microservices and the Kubernetes cluster itself are managed in a single Terraform workspace:

A. The platform will achieve maximum performance during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. AWS automatically splits the state file through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Any syntax error, state lock contention, or slow cloud API response during an update to a single microservice blocks deployments for all other 79 services, and an accidental destructive change could delete the entire cluster.
D. There is no operational risk while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Co-locating 80 microservices in a single monolithic state file creates massive blast radius risk and severe deployment bottlenecks.
- **Why A is Incorrect:** `The platform will achieve maximum performance during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `AWS automatically splits the state file through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `There is no operational risk while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
How can an enterprise detect if an unauthorized internal user downloaded a production Terraform state file from an S3 backend?

A. By enabling AWS CloudTrail S3 Data Events logging on the state bucket and monitoring for `GetObject` API calls.
B. By checking the `.terraform.lock.hcl` file by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. S3 logs downloads to the local terminal automatically through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By running `terraform state list` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** CloudTrail S3 Data Events provide an immutable audit trail of every `GetObject` call targeting the state bucket.
- **Why B is Incorrect:** `By checking the `.terraform.lock.hcl` file by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `S3 logs downloads to the local terminal automatically through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By running `terraform state list` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
An organization mandates that every provisionable resource must contain tags `Environment`, `Owner`, and `CostCenter`. At what lifecycle phase in HCP Terraform does a Sentinel policy validate these tags?

A. During `terraform init` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. During git commit while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. After the resources are deployed through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Post-plan, pre-apply (blocking the run before any cloud resources are created if tags are missing).

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Policy as Code evaluates after the plan is computed but before any state or cloud resources are mutated.
- **Why A is Incorrect:** `During `terraform init` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `During git commit while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `After the resources are deployed through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
What is the primary security benefit of using the `ephemeral` block for data sources introduced in Terraform 1.10+?

A. It bypasses network firewalls during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Secrets retrieved from external secret stores (such as Vault or AWS Secrets Manager) during execution are never persisted to the plan file or state file.
C. It makes API calls 10x cheaper through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It deletes the secret from Vault after reading while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Ephemeral data sources ensure that temporary credentials exist only in memory during execution.
- **Why A is Incorrect:** `It bypasses network firewalls during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It makes API calls 10x cheaper through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It deletes the secret from Vault after reading while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
Which Terraform resource configuration prevents an S3 bucket from ever serving public traffic regardless of bucket policies?

A. ```hcl
B. `acl = "public-read"` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `lifecycle { prevent_destroy = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `force_destroy = false` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `aws_s3_bucket_public_access_block` enforces cloud-level overrides blocking public ACLs and policies.
- **Why B is Incorrect:** ``acl = "public-read"` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``lifecycle { prevent_destroy = true }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``force_destroy = false` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
Can an `ephemeral = true` variable be referenced in an output block that is NOT marked as `ephemeral = true`?

A. Yes, outputs ignore variable mutability during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Ephemeral variables cannot be referenced in outputs under any circumstances through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. No; if an output depends on an ephemeral value, the output itself must also be marked as `ephemeral = true` to prevent state leakage.
D. It depends on the cloud provider while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Ephemeral propagation rules require downstream outputs depending on ephemeral values to also be marked `ephemeral = true`.
- **Why A is Incorrect:** `Yes, outputs ignore variable mutability during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Ephemeral variables cannot be referenced in outputs under any circumstances through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It depends on the cloud provider while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
What IAM permission should be strictly granted to CI/CD runners to interact with the DynamoDB lock table without allowing full table deletion?

A. `dynamodb:*` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `dynamodb:Scan` only while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `dynamodb:DeleteTable` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Scoped permissions: `dynamodb:GetItem`, `dynamodb:PutItem`, and `dynamodb:DeleteItem` restricted strictly to the table ARN.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Least privilege requires granting only read, write, and delete item permissions on the specific lock table.
- **Why A is Incorrect:** ``dynamodb:*` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``dynamodb:Scan` only while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``dynamodb:DeleteTable` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
In HCP Terraform, what happens when an environment variable is marked as "Sensitive"?

A. It cannot be used in plans by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. Its value is hidden in the web UI and API responses, and write-only access is enforced for users.
C. It is deleted after 1 hour through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It is sent via email while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Sensitive variables in HCP Terraform are write-only in the UI/API.
- **Why A is Incorrect:** `It cannot be used in plans by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It is deleted after 1 hour through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It is sent via email while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
What is an HCP Terraform "Run Task"?

A. A bash script that reboots the runner during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. A cron job that runs `terraform init` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. An integration point that passes the speculative plan to an external third-party security or compliance vendor (e.g. Snyk, Infracost, Wiz) and awaits a pass/fail callback before allowing apply.
D. A task that deletes unused workspaces while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Run Tasks integrate external SAST, compliance, and cost engines into the HCP Terraform run pipeline.
- **Why A is Incorrect:** `A bash script that reboots the runner during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `A cron job that runs `terraform init` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `A task that deletes unused workspaces while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
An engineer authors an IAM role for an EC2 instance with `iam:PassRole` and `ec2:RunInstances` on `*`. Why does a security linter flag this as a critical finding?

A. It allows anyone on that EC2 instance to launch a new instance with administrative roles, effectively escalating privileges to full cloud administrator.
B. It causes network loops by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. It violates GDPR formatting through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. EC2 does not support IAM roles while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `iam:PassRole` without resource ARN constraints is a well-known privilege escalation vector in cloud environments.
- **Why B is Incorrect:** `It causes network loops by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It violates GDPR formatting through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `EC2 does not support IAM roles while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
