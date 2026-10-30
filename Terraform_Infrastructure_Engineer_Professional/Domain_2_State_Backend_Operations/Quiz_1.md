# Domain 2: State and Backend Operations — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
During a scheduled CI/CD apply, the runner container is forcefully killed due to an out-of-memory (OOM) error while applying changes to an AWS RDS database. Subsequent pipeline runs fail with `Error acquiring the state lock: ConditionalCheckFailedException`. What is the correct procedure to recover?

A. Delete the S3 state file and re-import all resources during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Run `terraform init -reconfigure` to override the lock through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Run `terraform force-unlock <Lock-ID>` using the Lock ID displayed in the error message, verify state integrity with `terraform state list`, and re-run the plan.
D. Delete the DynamoDB table while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform force-unlock` manually releases an orphaned state lock using the specific lock ID provided by the lock manager.
- **Why A is Incorrect:** `Delete the S3 state file and re-import all resources during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Run `terraform init -reconfigure` to override the lock through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Delete the DynamoDB table while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
An engineer notices that an out-of-band change was made to a security group rule in the AWS console. The engineering team agrees that this change was correct and should be kept and reflected in the Terraform state without modifying any infrastructure. Which command achieves this?

A. `terraform plan -refresh-only` followed by `terraform apply -refresh-only`
B. `terraform refresh` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `terraform state push` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform import` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `terraform plan -refresh-only` and `terraform apply -refresh-only` provide a safe, plan-driven way to update state to match out-of-band changes without risking unintended infrastructure modifications.
- **Why B is Incorrect:** ``terraform refresh` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform state push` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform import` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
You want to rename a resource from `aws_instance.web_server` to `aws_instance.frontend_node` in your `.tf` code. If you simply update the code and run `terraform apply`, what will happen?

A. Terraform renames the instance in place without downtime during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform updates the tags on the instance while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. Terraform returns an error stating the resource already exists through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Terraform destroys the running `aws_instance.web_server` and creates a brand-new `aws_instance.frontend_node`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Changing a resource's HCL address causes Terraform to treat the old address as removed (destroy) and the new address as unmanaged (create).
- **Why A is Incorrect:** `Terraform renames the instance in place without downtime during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Terraform updates the tags on the instance while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Terraform returns an error stating the resource already exists through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
Continuing from Question 3, which command allows you to rename the resource in code without causing Terraform to destroy and recreate the instance?

A. `terraform state rm aws_instance.web_server` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. `terraform state mv aws_instance.web_server aws_instance.frontend_node`
C. `terraform state pull aws_instance.web_server` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform import aws_instance.frontend_node <id>` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `terraform state mv` updates the resource address in the state file so Terraform recognizes that the existing cloud resource maps to the renamed HCL block.
- **Why A is Incorrect:** ``terraform state rm aws_instance.web_server` by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``terraform state pull aws_instance.web_server` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform import aws_instance.frontend_node <id>` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
What is the primary danger of using local state files (`terraform.tfstate`) in a multi-engineer team?

A. Lack of concurrency locking can cause race conditions where two engineers overwrite each other's changes, corrupting state and causing accidental resource deletion.
B. Local state files cannot store more than 10 resources during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Local state files cannot be backed up through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Local state does not support variables while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Without distributed state locking, concurrent runs can execute simultaneous plans/applies on stale state, leading to state corruption and resource loss.
- **Why B is Incorrect:** `Local state files cannot store more than 10 resources during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Local state files cannot be backed up through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Local state does not support variables while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
You need to inspect the exact JSON state payload stored in the remote backend without checking out code or modifying the backend. Which command outputs the current remote state directly to stdout?

A. `terraform state list` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `terraform state show` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `terraform state pull`
D. `terraform show -json` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `terraform state pull` reads the state from the configured backend and outputs the raw JSON to stdout.
- **Why A is Incorrect:** ``terraform state list` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform state show` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``terraform show -json` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
Which of the following backend types does NOT support native state locking?

A. Azure Blob Storage (azurerm) while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. Google Cloud Storage (GCS) by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. HCP Terraform / Terraform Cloud through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. AWS S3 (without DynamoDB)

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** S3 alone does not support locking; it requires a DynamoDB table with a `LockID` primary key. GCS, Azure Blob, and HCP Terraform support native locking.
- **Why A is Incorrect:** `Azure Blob Storage (azurerm) while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Google Cloud Storage (GCS) by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `HCP Terraform / Terraform Cloud through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What does the `terraform state replace-provider` command do?

A. It downloads the newest provider binary from the registry during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It updates the provider source namespace for resources in the state file (e.g. migrating from `hashicorp/digitalocean` to `digitalocean/digitalocean`).
C. It deletes all resources managed by a provider through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It replaces AWS resources with GCP equivalents while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `terraform state replace-provider` safely updates provider namespaces in state without re-creating resources.
- **Why A is Incorrect:** `It downloads the newest provider binary from the registry during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It deletes all resources managed by a provider through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It replaces AWS resources with GCP equivalents while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
Why is storing sensitive database passwords in Terraform input variables not sufficient to protect them from unauthorized access if state is unencrypted?

A. Variables are logged to `/tmp` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Variables are visible in the DNS query logs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Every resource attribute (including sensitive values) passed to a resource is saved in plaintext in the `.tfstate` JSON file.
D. Cloud providers publish state files publicly by default while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** The `sensitive = true` flag prevents values from printing in CLI stdout, but `.tfstate` records all attribute values in plaintext.
- **Why A is Incorrect:** `Variables are logged to `/tmp` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Variables are visible in the DNS query logs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Cloud providers publish state files publicly by default while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
When running `terraform graph`, what does each node and edge in the resulting Directed Acyclic Graph (DAG) represent?

A. Nodes represent resources, data sources, and provider configurations; edges represent direct dependency relationships between them.
B. Nodes represent cloud accounts; edges represent network routes during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Nodes represent Git commits; edges represent branches through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Nodes represent API endpoints; edges represent HTTP status codes while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Terraform's DAG models all resources and their explicit/implicit dependencies to determine parallel execution order.
- **Why B is Incorrect:** `Nodes represent cloud accounts; edges represent network routes during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Nodes represent Git commits; edges represent branches through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Nodes represent API endpoints; edges represent HTTP status codes while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
Your enterprise Landing Zone stores all Terraform state files in a centralized Security/Shared-Services AWS Account (`111122223333`), while your workspace provisions EKS workloads in a Spoke Production Account (`999988887777`). How should you configure authentication so Terraform accesses the central S3 state bucket using `RoleA` in Account `111122223333` while provisioning EKS resources using `RoleB` in Account `999988887777`?

A. Configure `assume_role = { role_arn = "...RoleA" }` inside the `backend "s3"` block and configure `assume_role { role_arn = "...RoleB" }` inside the default `provider "aws"` block.
B. Define two `provider "aws"` blocks with aliases `state` and `workload`, and pass `provider = aws.state` inside the `terraform { backend "s3" {} }` block.
C. Export both role ARNs into `AWS_ROLE_ARN` separated by a comma so the AWS SDK automatically routes S3 calls to Account `111122223333`.
D. Use `data "terraform_remote_state"` with `assume_role` instead of a `backend "s3"` block because `backend` blocks only support default environment credentials.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** The `s3` backend supports its own dedicated `assume_role = { role_arn = ... }` configuration block, completely independent of the `provider "aws"` `assume_role` block used for managing workload resources.
- **Why B is Incorrect:** `backend` blocks cannot reference `provider` blocks (`backend` initializes during `terraform init` before providers are loaded).
- **Why C & D are Incorrect:** `AWS_ROLE_ARN` only accepts a single ARN, and `backend "s3"` natively supports `assume_role`.

</details>

---

### Question 12
You are splitting `aws_rds_cluster.ledger` out of a monolithic workspace (`prod-monolith`) into a dedicated database workspace (`prod-ledger-db`) on Terraform 1.8 using a pure GitOps Pull Request workflow (no manual CLI `terraform state mv` access is permitted). Which combination of declarative blocks across the two repositories achieves this migration with zero database downtime?

A. Add `moved { from = aws_rds_cluster.ledger, to = workspace.prod_ledger_db.aws_rds_cluster.ledger }` in `prod-monolith`.
B. Replace the resource in `prod-monolith` with `removed { from = aws_rds_cluster.ledger lifecycle { destroy = false } }`, and add the resource block plus `import { to = aws_rds_cluster.ledger id = "prod-ledger-cluster" }` in `prod-ledger-db`.
C. Set `lifecycle { prevent_destroy = true }` in `prod-monolith`, delete the resource block from `prod-monolith`, and run `terraform apply -refresh-only` in `prod-ledger-db`.
D. Add `import { from_state = "s3://.../monolith.tfstate" }` in `prod-ledger-db` so it automatically deletes the entry from the upstream state.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** A `removed` block with `lifecycle { destroy = false }` declaratively forgets the resource from the source workspace's state during apply without deleting the cloud resource, while the `import` block in the destination workspace adopts the live cloud resource into the new state.
- **Why A is Incorrect:** `moved` blocks only work within a single state file/workspace; they cannot move resources across separate workspaces.
- **Why C is Incorrect:** Deleting a resource block removes its `prevent_destroy` rule and destroys the database!
- **Why D is Incorrect:** `import` blocks do not have a `from_state` attribute and never mutate another workspace's state.

</details>

---

### Question 13
During a disaster recovery drill, you download a historical state backup (`backup.tfstate`) from S3 Object Versioning whose internal `serial` is `84` and `lineage` is `f47ac10b-58cc-4372-a567-0e02b2c3d479`. The corrupted state currently sitting in the remote S3 backend has `serial` `89` and the exact same `lineage`. What happens if you run `terraform state push backup.tfstate` without any flags?

A. Terraform overwrites the remote state immediately because the `lineage` UUIDs match.
B. Terraform refuses to push the state and fails with an error because `backup.tfstate` has a lower `serial` (`84`) than the remote state (`89`); you must either increment `serial` to `90` in the file or pass `-force`.
C. Terraform automatically merges the resources from `serial 84` and `serial 89` and writes `serial 90`.
D. Terraform generates a new `lineage` UUID and creates a parallel workspace state file.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `terraform state push` enforces two optimistic concurrency checks: matching `lineage` AND a strictly higher `serial` number than the destination state. Pushing serial `84` over serial `89` is rejected unless you increment `serial` in the JSON payload or pass `terraform state push -force`.
- **Why A, C, and D are Incorrect:** Matching `lineage` alone is insufficient when `serial` is lower; Terraform never auto-merges state JSONs or alters `lineage` on push.

</details>

---

### Question 14
Your scheduled GitHub Actions drift detection workflow runs `terraform plan -refresh-only -detailed-exitcode -input=false` against production every hour. In the latest run, the command exits with return code `2`. What does exit code `2` signify in this exact invocation?

A. A fatal provider authentication or backend lock error prevented the plan from completing.
B. The plan succeeded and detected zero differences between `.tfstate` and the live cloud environment.
C. The plan succeeded without errors and detected out-of-band drift between the cached `.tfstate` attributes and live cloud API resources.
D. Two concurrent `terraform plan` processes collided on the S3 `.tflock` lockfile.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** With `-detailed-exitcode`, exit code `0` means succeeded with empty diff (no drift), exit code `1` means an error occurred, and exit code `2` means the plan succeeded and there is a non-empty diff (drift detected).
- **Why A, B, and D are Incorrect:** Errors always return exit code `1`, and zero diff returns exit code `0`.

</details>

---

### Question 15
You want the `payments-app` workspace in HCP Terraform to consume the `vpc_id` and `private_subnet_ids` outputs from the `core-network` workspace while enforcing strict least privilege so `payments-app` CANNOT read sensitive database passwords or internal attributes stored inside `core-network`'s raw `.tfstate` file. Which configuration achieves this?

A. Configure `core-network` workspace settings to share outputs with `payments-app`, and query `data "tfe_outputs" "net"` inside `payments-app`.
B. Use `data "terraform_remote_state" "net"` in `payments-app` and set `sensitive = true` on all database outputs in `core-network`.
C. Grant the `payments-app` execution role read-only `s3:GetObject` permission to the `core-network` state file and filter attributes using `local` expressions.
D. Copy the `core-network` `.tfstate` file into the `payments-app` Git repository prior to running `terraform plan`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `data "tfe_outputs"` queries only the exported workspace output values via the HCP Terraform Outputs API without granting the consumer workspace access to read the producer's raw `.tfstate` snapshot.
- **Why B & C are Incorrect:** `data "terraform_remote_state"` requires full read access to the entire raw `.tfstate` file, exposing all state attributes regardless of `sensitive = true`.
- **Why D is Incorrect:** Committing `.tfstate` to Git is a severe security and operational anti-pattern.

</details>

---
