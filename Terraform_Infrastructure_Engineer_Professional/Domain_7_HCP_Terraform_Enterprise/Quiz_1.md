# Domain 7: HCP Terraform / Enterprise Operations — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
An enterprise needs to manage infrastructure inside an isolated on-premises VMware vSphere cluster that has no inbound public internet access. Which HCP Terraform execution mode must be configured?

A. Remote Execution on HashiCorp-hosted runners during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Local Execution only through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Agent Execution using self-hosted `tfc-agent` instances deployed within the private on-premises network.
D. Cloud Run while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `tfc-agent` runners make outbound HTTPS calls to fetch jobs, allowing private/on-prem infrastructure automation without inbound firewall exposure.
- **Why A is Incorrect:** `Remote Execution on HashiCorp-hosted runners during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Local Execution only through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Cloud Run while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
What is the structural relationship between Organizations, Projects, and Workspaces in HCP Terraform?

A. An Organization contains one or more Projects, and each Project contains one or more Workspaces.
B. Workspaces contain Organizations, which contain Projects during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Projects are only used for billing through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. There is no hierarchy; all are flat lists while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** HCP Terraform's resource hierarchy is: Organization -> Projects -> Workspaces.
- **Why B is Incorrect:** `Workspaces contain Organizations, which contain Projects during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Projects are only used for billing through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `There is no hierarchy; all are flat lists while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
An administrator wants to apply AWS region and environment tags across 50 workspaces in the `Payments` project without manually copy-pasting the variables into each workspace. What HCP Terraform feature accomplishes this?

A. Modifying the `.terraform.lock.hcl` file while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. A global bash script by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Committing `terraform.tfvars` to a public repository through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. A Project-scoped Variable Set.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Variable Sets scoped at the Project level automatically cascade shared variables to all child workspaces.
- **Why A is Incorrect:** `Modifying the `.terraform.lock.hcl` file while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `A global bash script by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Committing `terraform.tfvars` to a public repository through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
In HCP Terraform, what happens when Cost Estimation is enabled for an AWS or GCP workspace?

A. Cloud providers charge a $10 fee per run by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. HCP Terraform queries live cloud pricing APIs during the plan phase to calculate the estimated monthly cost increase (or decrease) caused by the proposed plan.
C. HCP Terraform automatically cancels plans that cost more than $50 through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Cost estimation is only available for local state while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Cost Estimation evaluates planned resources against cloud pricing APIs before apply.
- **Why A is Incorrect:** `Cloud providers charge a $10 fee per run by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `HCP Terraform automatically cancels plans that cost more than $50 through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Cost estimation is only available for local state while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
What is an HCP Terraform "Run Task"?

A. An integration point between plan and apply where HCP Terraform sends the plan JSON to an external security or cost vendor (e.g. Snyk, Infracost) and awaits an approval or block response.
B. A scheduled backup of the state file during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. A command that restarts failed runners through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. A cron job for deleting old workspaces while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Run Tasks integrate external SAST and FinOps platforms directly into the execution lifecycle.
- **Why B is Incorrect:** `A scheduled backup of the state file during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `A command that restarts failed runners through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `A cron job for deleting old workspaces while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
When an environment variable in an HCP Terraform workspace is marked as "Sensitive", how is it handled?

A. The value is hashed with MD5 and printed in logs during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It can only be used once through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. The value is masked in the UI and API, write-only access is enforced, and it cannot be viewed by users once saved.
D. It is deleted when the browser closes while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Sensitive variables in HCP Terraform are write-only to prevent credential exfiltration.
- **Why A is Incorrect:** `The value is hashed with MD5 and printed in logs during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It can only be used once through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It is deleted when the browser closes while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In an enterprise with 500 workspaces, how can platform engineers upgrade the default Terraform version across all workspaces systematically?

A. By deleting and recreating all workspaces during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. By clearing the browser cache while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. Terraform versions cannot be upgraded in HCP Terraform through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By updating workspace settings via the HCP Terraform REST API or Terraform provider for TFE (`tfe_workspace.terraform_version`), testing non-production workspaces first.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** The official `tfe` Terraform provider enables programmatic automation of workspace configurations and engine versions.
- **Why A is Incorrect:** `By deleting and recreating all workspaces during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `By clearing the browser cache while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Terraform versions cannot be upgraded in HCP Terraform through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What user permission in HCP Terraform allows an engineer to trigger speculative plans and view plan logs, but prohibits them from approving or applying changes to production infrastructure?

A. Admin permission during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Custom Workspace Permissions with `Runs: Plan` (or standard `Plan` permission).
C. Read-only permission through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Organization Owner while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Role-Based Access Control allows separating plan trigger privileges from apply execution privileges.
- **Why A is Incorrect:** `Admin permission during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Read-only permission through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Organization Owner while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
How do self-hosted `tfc-agent` runners communicate with HCP Terraform?

A. HCP Terraform initiates inbound SSH connections to the agent on port 22 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Via UDP broadcast through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. The agent initiates outbound HTTPS (port 443) long-polling connections to HCP Terraform APIs to pull queued jobs, eliminating the need for inbound open firewall ports.
D. Through an AWS Direct Connect link only while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Agents use outbound polling on port 443, making them simple and secure to deploy in restricted network environments.
- **Why A is Incorrect:** `HCP Terraform initiates inbound SSH connections to the agent on port 22 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Via UDP broadcast through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Through an AWS Direct Connect link only while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
What is the effect of locking a workspace in HCP Terraform?

A. New plan and apply runs cannot be executed in the workspace until it is manually unlocked, preventing accidental concurrent executions during scheduled maintenance.
B. All resources in the cloud are immediately stopped during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. The workspace is archived and deleted after 30 days through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The workspace state file is decrypted while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Locking a workspace halts execution pipelines to allow safe maintenance and debugging.
- **Why B is Incorrect:** `All resources in the cloud are immediately stopped during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `The workspace is archived and deleted after 30 days through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The workspace state file is decrypted while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
In an HCP Terraform Organization, the variable `aws_region` is set in four places (none marked with the Priority override flag):
1. Global Organization Variable Set: `"us-east-1"`
2. Project Variable Set (`Platform-Prod`): `"us-west-2"`
3. Workspace Variable Set (`eks-prod`): `"eu-west-1"`
4. Workspace-specific Variable defined directly on `eks-prod`: `"ap-northeast-1"`
When a run executes in workspace `eks-prod`, which value of `aws_region` is used?

A. `"us-east-1"` because Global Organization Variable Sets enforce organization-wide baseline defaults.
B. `"us-west-2"` because Project-level governance overrides workspace-level settings.
C. `"eu-west-1"` because Workspace Variable Sets are evaluated after individual workspace variables.
D. `"ap-northeast-1"` because more specific scopes override broader scopes (Workspace Variable > Workspace Variable Set > Project Variable Set > Global Variable Set).

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Unless a Variable Set has the explicit **Priority** override flag enabled, HCP Terraform resolves variable conflicts from most specific to least specific: Workspace Variable (`ap-northeast-1`) wins over Workspace Variable Set (`eu-west-1`), which wins over Project Variable Set (`us-west-2`), which wins over Global Variable Set (`us-east-1`).
- **Why A, B, and C are Incorrect:** Broader variable sets only win if an administrator explicitly checks the Priority override option on the variable set.

</details>

---

### Question 12
Your Platform Engineering team manages two separate **Terraform Stacks** in HCP Terraform: an upstream `network-foundation` stack and a downstream `eks-platform` stack. How do you pass the regional VPC IDs created by `network-foundation` into `eks-platform` using native Stack primitives?

A. Declare a `publish_output` block in the `network-foundation` stack's `.tfdeploy.hcl` file, and declare an `upstream_input` block referencing the `network-foundation` stack in `eks-platform`'s `.tfdeploy.hcl` file.
B. Use `data "terraform_remote_state"` inside `eks-platform`'s `.tfstack.hcl` file pointing to the S3 bucket of `network-foundation`.
C. Configure a workspace `tfe_run_trigger` resource linking the two `.tfstack.hcl` files.
D. Symlink the `.tfstate` file between the two stack repositories in Git.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Linked Terraform Stacks use `publish_output` in the producer stack's `.tfdeploy.hcl` configuration and `upstream_input` in the consumer stack's `.tfdeploy.hcl` configuration to pass typed values and automatically trigger downstream stack replans when upstream outputs change.
- **Why B, C, and D are Incorrect:** Stacks manage component state internally in HCP Terraform and use `publish_output`/`upstream_input` rather than `terraform_remote_state` or workspace-level `tfe_run_trigger`.

</details>

---

### Question 13
You are architecting a self-hosted **Terraform Enterprise (TFE)** deployment on Amazon EKS using Flexible Deployment Options (FDO) in **Active-Active** mode (2 stateless TFE pods behind an AWS Application Load Balancer) to satisfy a 99.99% availability SLA. Which four external dependencies are strictly mandatory for Active-Active TFE operation?

A. External PostgreSQL (e.g., RDS), External Object Storage (e.g., S3), External Redis (e.g., ElastiCache), and an identical `TFE_ENCRYPTION_PASSWORD` configured across all TFE pods.
B. Mounted Disk EFS shared storage, Amazon DynamoDB state locking table, and an internal Consul cluster.
C. External MySQL database, local container disk storage for `.tfstate`, and sticky Layer-4 TCP sessions.
D. Three TFE pods running embedded PostgreSQL with Raft consensus and local NVMe volumes.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Active-Active TFE requires stateless compute pods backed by three external persistence/coordination services—**External PostgreSQL**, **External S3-compatible Object Storage**, and **External Redis**—plus an identical **`TFE_ENCRYPTION_PASSWORD`** across all nodes to decrypt the internal Vault keyring.
- **Why B, C, and D are Incorrect:** Mounted Disk mode cannot run Active-Active; TFE requires PostgreSQL (not MySQL or DynamoDB); and embedded DB/Redis cannot scale across multiple Active-Active nodes.

</details>

---

### Question 14
Your Platform Engineering team is building an Internal Developer Portal (Backstage) that enables product engineers to provision a pre-approved database and dedicated HCP Terraform workspace by filling out a web form—without writing any HCL or creating a Git repository. Which HCP Terraform feature provides this capability natively?

A. Private Registry **No-Code Provisioning** enabled on a vetted registry module with variable option lists.
B. Speculative CLI-driven runs using `terraform login`.
C. Local Execution Mode workspaces connected to an S3 backend.
D. Sentinel `tfconfig/v2` auto-generation policies.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** **No-Code Provisioning** allows platform teams to publish a vetted module in the HCP Terraform Private Registry, mark it as No-Code enabled, and define allowed variable options so users (or an IDP API call) can instantiate a managed workspace directly without a VCS repository or HCL authoring.
- **Why B, C, and D are Incorrect:** None of these provision VCS-less self-service workspaces from a module catalog.

</details>

---

### Question 15
A disaster strikes your primary region hosting Terraform Enterprise (TFE) in External Services mode. You have recent backups of your External PostgreSQL database and your External S3 Object Storage bucket in the DR region, and you launch a replacement TFE instance connected to both restored stores. However, TFE fails to start and reports that it cannot unseal or decrypt stored workspace state and variables. What critical artifact is missing?

A. The original `.terraform.lock.hcl` file from the TFE host container.
B. The original `TFE_ENCRYPTION_PASSWORD` environment variable used by the primary TFE deployment to derive the root encryption key for PostgreSQL and Object Storage data.
C. The SSH host key of the terminated Kubernetes worker node.
D. The `terraform.tfstate.tflock` file from the S3 bucket.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** TFE encrypts sensitive data in PostgreSQL and all state/plan blobs in Object Storage using an internal Vault keyring protected by a root key derived from `TFE_ENCRYPTION_PASSWORD`. Without the exact original `TFE_ENCRYPTION_PASSWORD`, database and S3 backups are cryptographically unreadable!
- **Why A, C, and D are Incorrect:** Container lockfiles, node SSH keys, and `.tflock` files are not used to encrypt TFE persistence stores.

</details>

---
