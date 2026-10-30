# Domain 7: HCP Terraform & Terraform Enterprise Operations — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
In HCP Terraform / Terraform Enterprise, you have an isolated private data center with no inbound Internet access. How can Terraform Enterprise workspaces execute plans and applies against private on-premises vSphere and VMware clusters?

A. By opening public firewall ports to the Internet during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. By configuring VPN tunnels to public DNS servers through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. By deploying HCP Terraform Agents (containerized workers) inside the private data center network that poll HCP Terraform outbound over HTTPS for execution jobs.
D. By using SSH port forwarding from engineer laptops while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** HCP Terraform Agents run inside private networks and establish outbound-only polling connections to execute runs securely behind corporate firewalls.
- **Why A is Incorrect:** `By opening public firewall ports to the Internet during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `By configuring VPN tunnels to public DNS servers through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By using SSH port forwarding from engineer laptops while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
What is the difference between a `CLI-driven run` and a `VCS-driven run` in HCP Terraform?

A. CLI-driven runs execute locally on developer laptops and push state only; VCS-driven runs trigger automatically on Git pull requests or commits, executing plans and applies inside remote HCP Terraform runners.
B. VCS-driven runs do not support variables by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. CLI-driven runs cannot use Sentinel through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. There is no difference while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** CLI-driven runs are triggered manually from the local terminal (`terraform plan/apply`); VCS-driven runs trigger automatically on webhook events from GitHub, GitLab, etc.
- **Why B is Incorrect:** `VCS-driven runs do not support variables by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `CLI-driven runs cannot use Sentinel through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `There is no difference while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
An enterprise platform engineering team wants to share a common set of AWS credentials and environment variables across 50 distinct workspaces without manually copying them into each workspace. Which HCP Terraform feature solves this?

A. Workspace tags while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. Committed `.env` files in Git by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Hardcoded credentials in root modules through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Variable Sets (varsets) scoped globally or attached to specific workspaces / projects.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Variable Sets enable centralized definition and reuse of environment/Terraform variables across projects and workspaces.
- **Why A is Incorrect:** `Workspace tags while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Committed `.env` files in Git by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Hardcoded credentials in root modules through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
In HCP Terraform, what occurs during a `Cost Estimation` stage before Sentinel policy checks?

A. Terraform automatically purchases cloud discounts during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform analyzes the planned resource changes against public cloud provider pricing APIs, calculating hourly and monthly delta cost estimates and total predicted spend.
C. The organization's credit card is charged through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The run is cancelled if the estimate is over \$10 while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Cost estimation evaluates planned delta changes against cloud pricing data, feeding cost data into Sentinel policies and PR UI summaries.
- **Why A is Incorrect:** `Terraform automatically purchases cloud discounts during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `The organization's credit card is charged through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The run is cancelled if the estimate is over \$10 while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
How does `Run Triggers` enable multi-workspace orchestration in HCP Terraform?

A. It connects a downstream workspace to an upstream workspace, automatically initiating a plan in the downstream workspace whenever an apply completes successfully in the upstream workspace.
B. It sends Slack notifications on run failures during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. It restarts cloud VMs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It merges Git branches automatically while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Run Triggers establish sequential execution pipelines between interconnected workspaces (e.g., `networking` $\rightarrow$ `database` $\rightarrow$ `app`).
- **Why B is Incorrect:** `It sends Slack notifications on run failures during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It restarts cloud VMs through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It merges Git branches automatically while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
What is the primary role of `Projects` in HCP Terraform?

A. To compile Go code during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. To host Docker images through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. To organize and group workspaces hierarchically, enabling centralized team access control and permission management across multiple workspaces.
D. To bill separate credit cards while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Projects provide logical containerization and RBAC boundaries for collections of workspaces within an HCP Terraform organization.
- **Why A is Incorrect:** `To compile Go code during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `To host Docker images through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `To bill separate credit cards while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In Terraform Enterprise, which high-availability architecture provides active-active cluster nodes with an external PostgreSQL database and shared object storage (S3/Azure Blob)?

A. SQLite clustering while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
B. Standalone Docker engine by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Local disk replication through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Active/Active clustered deployment (FDO / Flexible Deployment Options).

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Terraform Enterprise Active/Active architecture distributes worker nodes across multiple VMs/instances backed by managed database and object storage tiers.
- **Why A is Incorrect:** `SQLite clustering while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Standalone Docker engine by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `Local disk replication through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
When configuring Dynamic Provider Credentials in HCP Terraform for AWS, how does the runner authenticate to AWS without static access keys?

A. Through an SSH tunnel during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. HCP Terraform generates a short-lived OIDC token signed by its private key; AWS STS validates the token against the configured OIDC Identity Provider and assumes an IAM role.
C. By reading AWS keys from a shared S3 bucket through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By running a local HashiCorp Vault container while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Dynamic Provider Credentials use OIDC federation with cloud STS/IAM endpoints to issue ephemeral, temporary credentials per run.
- **Why A is Incorrect:** `Through an SSH tunnel during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `By reading AWS keys from a shared S3 bucket through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By running a local HashiCorp Vault container while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
In HCP Terraform, what is the effect of configuring a workspace's `Execution Mode` to `Local`?

A. The workspace is deleted during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. State is saved on a USB drive through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Plan and apply operations run directly on the user's local machine, while state is stored remotely in the HCP Terraform workspace.
D. Sentinel policies run on the user's workstation while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** In `Local` execution mode, CLI operations execute on the engineer's workstation, using HCP Terraform purely as a remote state and locking backend.
- **Why A is Incorrect:** `The workspace is deleted during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `State is saved on a USB drive through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Sentinel policies run on the user's workstation while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
What capability does `Continuous Validation` provide in HCP Terraform?

A. It periodically runs `check` blocks and drift detection assertions (every 24 hours or on a schedule) on managed infrastructure, alerting teams when external drift or assertion failures occur outside of manual runs.
B. It runs syntax checks on every git push during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. It reboots the cloud hypervisor daily through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It forces daily code refactoring while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Continuous Validation periodically evaluates `check` blocks on live infrastructure without requiring an engineer to trigger a run, alerting to post-deployment drift.
- **Why B is Incorrect:** `It runs syntax checks on every git push during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `It reboots the cloud hypervisor daily through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It forces daily code refactoring while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
