# Domain 7: HCP Terraform / Enterprise Operations — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Enterprise Governance & Workspace Management Questions

---

---

### Question 1
When publishing an internal module to the HCP Terraform Private Module Registry:

A. The repository must be public on GitHub during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. The module cannot contain variables through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. The repository must follow the `terraform-<PROVIDER>-<NAME>` naming standard and be tagged with a semantic version (e.g., `v1.0.0`).
D. Modules must be uploaded as zip archives via FTP while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Standard 3-part naming and Git SemVer tags are required to register modules in the Private Module Registry.
- **Why A is Incorrect:** `The repository must be public on GitHub during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `The module cannot contain variables through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Modules must be uploaded as zip archives via FTP while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
How do security teams stream audit events (such as workspace deletion, variable modification, or plan approvals) from Terraform Enterprise to an external SIEM (Splunk, Datadog)?

A. By configuring the native Audit Log Streaming API to forward signed JSON payloads to an HTTP event collector endpoint.
B. By scraping web pages using Selenium during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Audit logging is not supported through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By reading `/var/log/syslog` on the developer's laptop while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Audit Log Streaming exports immutable real-time security events directly to enterprise SIEM platforms.
- **Why B is Incorrect:** `By scraping web pages using Selenium during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Audit logging is not supported through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `By reading `/var/log/syslog` on the developer's laptop while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
Why should everyday developers NOT be assigned the "Organization Owner" role in HCP Terraform?

A. Organization Owners cannot write HCL during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It disables cost estimation while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. HCP limits organizations to 1 owner through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Organization Owners have unrestricted superuser access, including deleting projects, managing billing, revoking SSO, and accessing all sensitive variables across all workspaces.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** The Organization Owner role represents total administrative authority and must be strictly restricted.
- **Why A is Incorrect:** `Organization Owners cannot write HCL during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It disables cost estimation while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `HCP limits organizations to 1 owner through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
When using HCP Terraform's native Dynamic Provider Credentials for AWS:

A. An administrator must update passwords every 30 days manually by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. HCP Terraform exchanges its OIDC token directly with AWS STS to assume the configured IAM role for each run, requiring zero static AWS keys in workspace variables.
C. Static keys are rotated by a nightly cron job through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. AWS credentials must be committed to Git while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** HCP Terraform's native OIDC integration eliminates static credentials through automated STS token exchange.
- **Why A is Incorrect:** `An administrator must update passwords every 30 days manually by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Static keys are rotated by a nightly cron job through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `AWS credentials must be committed to Git while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
When a workspace is deleted from HCP Terraform via the UI without running `terraform destroy` first:

A. The cloud resources remain running unmanaged in the cloud provider, but the state file is permanently destroyed in HCP Terraform, leaving orphaned infrastructure.
B. All real-world cloud resources managed by that workspace are deleted automatically during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. HCP Terraform prevents workspace deletion if resources are present in state through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. The cloud provider sends an email alert while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Deleting a workspace without destroying infrastructure orphans the live resources, requiring manual cloud-level cleanup.
- **Why B is Incorrect:** `All real-world cloud resources managed by that workspace are deleted automatically during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `HCP Terraform prevents workspace deletion if resources are present in state through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `The cloud provider sends an email alert while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
What is the primary operational benefit of attaching tags (e.g., `env:prod`, `team:billing`, `pci-dss:true`) to HCP Terraform workspaces?

A. Tags change the color of the UI buttons during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Tags reduce state file size through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. Tags enable filtering in the UI/API, allow targeted Policy Set enforcement across subsets of workspaces, and facilitate project organization.
D. Tags automatically scale runner memory while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** Workspace tags provide metadata hooks for filtering and targeting Policy as Code rules.
- **Why A is Incorrect:** `Tags change the color of the UI buttons during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `Tags reduce state file size through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Tags automatically scale runner memory while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
In self-hosted Terraform Enterprise Flexible Deployment Options (FDO), how is high availability achieved?

A. By running a single EC2 instance during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. FDO does not support high availability while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. By syncing laptops via Bluetooth through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. By deploying stateless container nodes behind an application load balancer, backed by external PostgreSQL database and object storage (S3/GCS) with multi-AZ failover.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** Production Terraform Enterprise architectures decouple stateless application containers from clustered storage engines.
- **Why A is Incorrect:** `By running a single EC2 instance during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `FDO does not support high availability while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** `By syncing laptops via Bluetooth through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What is a key difference between a VCS-driven workspace and a CLI-driven workspace in HCP Terraform?

A. CLI-driven workspaces do not use state by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
B. VCS-driven workspaces automatically trigger plans upon Git pull requests and applies upon branch merges; CLI-driven workspaces execute runs initiated from a terminal using `terraform plan/apply`.
C. VCS-driven workspaces cannot use modules through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. CLI-driven workspaces cannot deploy to AWS while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** VCS-driven runs automate the continuous integration feedback loop based on Git repository events.
- **Why A is Incorrect:** `CLI-driven workspaces do not use state by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `VCS-driven workspaces cannot use modules through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `CLI-driven workspaces cannot deploy to AWS while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
Which notification destinations are natively supported by HCP Terraform workspace webhooks?

A. Twitter/X posts through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
B. SMS text messages only by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Slack, Microsoft Teams, generic webhooks, and email.
D. Fax while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** HCP Terraform provides built-in webhook notification handlers for chatOps and automated orchestration.
- **Why A is Incorrect:** `Twitter/X posts through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `SMS text messages only by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Fax while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
When a GitHub pull request triggers a speculative plan in an HCP Terraform VCS-driven workspace, where is the plan result surfaced?

A. Directly on the GitHub Pull Request checks status widget and PR comments, linking back to the detailed HCP Terraform run interface.
B. Only in the developer's local terminal during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. In the company Slack general channel through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. It is sent via postal mail while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Native VCS status integration surfaces pass/fail indicators and plan diffs directly in developer pull request views.
- **Why B is Incorrect:** `Only in the developer's local terminal during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `In the company Slack general channel through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It is sent via postal mail while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---
