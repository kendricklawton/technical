# Domain 2: State and Backend Operations — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced State Operations & Architecture Questions

---

---

### Question 1
A security engineering team manages all VPCs and KMS keys in a dedicated state file `security/vpc.tfstate`. An application team needs to read the VPC ID and Subnet IDs without having write permissions to the security state. How should the application team configure access?

A. By importing the VPC resource into their own state file during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. By hardcoding the VPC ID into `terraform.tfvars` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. By defining a `terraform_remote_state` data source pointing to the security team's remote S3 backend and reading its published `outputs`.
D. By combining both state files using `terraform state push` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** The `terraform_remote_state` data source allows cross-workspace consumption of published outputs while preserving strict read-only access boundaries.
- **Why A is Incorrect:** `By importing the VPC resource into their own state file during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `By hardcoding the VPC ID into `terraform.tfvars` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By combining both state files using `terraform state push` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
Which of the following architectures represents the best practice for organizing state files in a high-scale enterprise?

A. Decoupled workspaces or backend keys separated by lifecycle, team ownership, and blast radius (e.g., `networking`, `identity`, `kubernetes-clusters`, `payment-service`).
B. A single monolithic `main.tfstate` storing 1,500 resources across 4 AWS regions during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Committing `terraform.tfstate` directly into the application's Git repository through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Storing state in local `/tmp` directories on CI runners while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Segmenting state by functional layer and lifecycle limits blast radius so a failure in an application rollout cannot corrupt the core VPC.
- **Why B is Incorrect:** `A single monolithic `main.tfstate` storing 1,500 resources across 4 AWS regions during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Committing `terraform.tfstate` directly into the application's Git repository through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Storing state in local `/tmp` directories on CI runners while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
When upgrading an enterprise environment from Terraform v1.8 to v1.15 across 40 state files, what is the safest progression?

A. Update the CI pipeline runner to v1.15 and apply all workspaces simultaneously during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Delete the `.terraform.lock.hcl` file while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. Run `terraform state rm` on all resources before upgrading through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Upgrade one non-production workspace first, verify provider compatibility, review `terraform plan` output for breaking schema deprecations, run `terraform apply`, test workloads, and then sequentially promote through staging and production.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Systematic promotion across environments with compatibility verification is required for enterprise Terraform version upgrades.
- **Why A is Incorrect:** `Update the CI pipeline runner to v1.15 and apply all workspaces simultaneously during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Delete the `.terraform.lock.hcl` file while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Run `terraform state rm` on all resources before upgrading through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
A junior engineer ran `terraform state push` with a truncated, unvalidated backup file, resulting in an invalid JSON error on subsequent runs. What is the fastest way to recover if the S3 backend has bucket versioning enabled?

A. Delete the S3 bucket during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Restore the prior S3 object version using the AWS S3 Console or CLI, and verify with `terraform state pull | jq .serial`.
C. Re-run `terraform init -upgrade` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Run `terraform force-unlock` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Enabling S3 bucket versioning on state buckets provides point-in-time recovery from accidental state overwrites or corruption.
- **Why A is Incorrect:** `Delete the S3 bucket during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Re-run `terraform init -upgrade` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Run `terraform force-unlock` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
A company is decommissioning an internal microservice. The engineer wants to remove the Kubernetes deployment from Terraform management because it will now be managed by ArgoCD, but the deployment must NOT be deleted from the live Kubernetes cluster. Which command must the engineer run?

A. `terraform state rm kubernetes_deployment.app`
B. `terraform destroy -target=kubernetes_deployment.app` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `terraform state mv kubernetes_deployment.app argocd_app` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform taint kubernetes_deployment.app` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform state rm` removes the object from Terraform tracking without issuing a delete call to the cloud provider.
- **Why B is Incorrect:** ``terraform destroy -target=kubernetes_deployment.app` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform state mv kubernetes_deployment.app argocd_app` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform taint kubernetes_deployment.app` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
What happens if Resource A references an attribute of Resource B, and Resource B simultaneously references an attribute of Resource A?

A. Terraform automatically resolves the reference at runtime during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform applies Resource A first and Resource B second through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Terraform throws a `Cycle error` during graph construction and halts execution without modifying infrastructure.
D. The cloud provider detects the loop and creates an asynchronous task while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Terraform's DAG requires an acyclic graph; circular references cause an immediate cycle error.
- **Why A is Incorrect:** `Terraform automatically resolves the reference at runtime during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terraform applies Resource A first and Resource B second through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The cloud provider detects the loop and creates an asynchronous task while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In AWS, an `aws_security_group` resource can declare inline `ingress` rules, or you can use standalone `aws_security_group_rule` resources. If two security groups need to reference each other's IDs, how do you prevent a cycle error?

A. Define both rules inside the security group blocks during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Security groups cannot reference each other in AWS while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. Use `lifecycle { ignore_changes = [ingress] }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Create the security groups first without inline rules, and use separate `aws_security_group_rule` resources to link them bidirectionally.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Using standalone `aws_security_group_rule` resources breaks the circular dependency because the parent security group IDs exist before the rules are attached.
- **Why A is Incorrect:** `Define both rules inside the security group blocks during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Security groups cannot reference each other in AWS while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Use `lifecycle { ignore_changes = [ingress] }` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What is the difference between `terraform init -migrate-state` and `terraform init -reconfigure`?

A. `-reconfigure` deletes all cloud infrastructure by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. `-migrate-state` attempts to copy the existing state from the old backend to the new backend; `-reconfigure` ignores existing backend state configuration and initializes the new backend from scratch.
C. `-migrate-state` only works with local backends through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. There is no difference while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `-migrate-state` copies state data between backends, whereas `-reconfigure` reinitializes without copying.
- **Why A is Incorrect:** ``-reconfigure` deletes all cloud infrastructure by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``-migrate-state` only works with local backends through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `There is no difference while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
Do Terraform backend locks automatically expire after a set time (e.g. 15 minutes)?

A. Yes, all backends release locks after 10 minutes during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Only HCP Terraform has persistent locks through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. No; in systems like DynamoDB, GCS, and S3, locks are persistent records that remain in place until the applying process releases them or an administrator runs `terraform force-unlock`.
D. Locks only last while the TCP connection is open while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** State locks are explicit persistent records designed to prevent concurrent writes even across long-running applies.
- **Why A is Incorrect:** `Yes, all backends release locks after 10 minutes during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Only HCP Terraform has persistent locks through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Locks only last while the TCP connection is open while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
You have 400 resources in state. Which command lists only the DigitalOcean Droplet resources without printing disks, firewalls, or VPCs?

A. `terraform state list | grep droplet` or `terraform state list digitalocean_droplet`
B. `terraform show droplet` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. `terraform get droplets` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform plan -filter=droplets` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform state list [address]` filters state resources by address prefix or type.
- **Why B is Incorrect:** ``terraform show droplet` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform get droplets` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform plan -filter=droplets` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
