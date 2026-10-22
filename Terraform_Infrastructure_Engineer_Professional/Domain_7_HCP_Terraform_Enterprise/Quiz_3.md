# Domain 7: HCP Terraform & Terraform Enterprise Operations — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
In HCP Terraform / Terraform Enterprise, you have an isolated private data center with no inbound Internet access. How can Terraform Enterprise workspaces execute plans and applies against private on-premises vSphere and VMware clusters?
- A. By opening public firewall ports to the Internet.
- B. By deploying HCP Terraform Agents (containerized workers) inside the private data center network that poll HCP Terraform outbound over HTTPS for execution jobs.
- C. By configuring VPN tunnels to public DNS servers.
- D. By using SSH port forwarding from engineer laptops.

---

### Question 2
What is the difference between a `CLI-driven run` and a `VCS-driven run` in HCP Terraform?
- A. CLI-driven runs execute locally on developer laptops and push state only; VCS-driven runs trigger automatically on Git pull requests or commits, executing plans and applies inside remote HCP Terraform runners.
- B. VCS-driven runs do not support variables.
- C. CLI-driven runs cannot use Sentinel.
- D. There is no difference.

---

### Question 3
An enterprise platform engineering team wants to share a common set of AWS credentials and environment variables across 50 distinct workspaces without manually copying them into each workspace. Which HCP Terraform feature solves this?
- A. Variable Sets (varsets) scoped globally or attached to specific workspaces / projects.
- B. Committed `.env` files in Git.
- C. Hardcoded credentials in root modules.
- D. Workspace tags.

---

### Question 4
In HCP Terraform, what occurs during a `Cost Estimation` stage before Sentinel policy checks?
- A. Terraform automatically purchases cloud discounts.
- B. Terraform analyzes the planned resource changes against public cloud provider pricing APIs, calculating hourly and monthly delta cost estimates and total predicted spend.
- C. The organization's credit card is charged.
- D. The run is cancelled if the estimate is over \$10.

---

### Question 5
How does `Run Triggers` enable multi-workspace orchestration in HCP Terraform?
- A. It sends Slack notifications on run failures.
- B. It connects a downstream workspace to an upstream workspace, automatically initiating a plan in the downstream workspace whenever an apply completes successfully in the upstream workspace.
- C. It restarts cloud VMs.
- D. It merges Git branches automatically.

---

### Question 6
What is the primary role of `Projects` in HCP Terraform?
- A. To compile Go code.
- B. To organize and group workspaces hierarchically, enabling centralized team access control and permission management across multiple workspaces.
- C. To host Docker images.
- D. To bill separate credit cards.

---

### Question 7
In Terraform Enterprise, which high-availability architecture provides active-active cluster nodes with an external PostgreSQL database and shared object storage (S3/Azure Blob)?
- A. Active/Active clustered deployment (FDO / Flexible Deployment Options).
- B. Standalone Docker engine.
- C. Local disk replication.
- D. SQLite clustering.

---

### Question 8
When configuring Dynamic Provider Credentials in HCP Terraform for AWS, how does the runner authenticate to AWS without static access keys?
- A. Through an SSH tunnel.
- B. HCP Terraform generates a short-lived OIDC token signed by its private key; AWS STS validates the token against the configured OIDC Identity Provider and assumes an IAM role.
- C. By reading AWS keys from a shared S3 bucket.
- D. By running a local HashiCorp Vault container.

---

### Question 9
In HCP Terraform, what is the effect of configuring a workspace's `Execution Mode` to `Local`?
- A. The workspace is deleted.
- B. Plan and apply operations run directly on the user's local machine, while state is stored remotely in the HCP Terraform workspace.
- C. State is saved on a USB drive.
- D. Sentinel policies run on the user's workstation.

---

### Question 10
What capability does `Continuous Validation` provide in HCP Terraform?
- A. It runs syntax checks on every git push.
- B. It periodically runs `check` blocks and drift detection assertions (every 24 hours or on a schedule) on managed infrastructure, alerting teams when external drift or assertion failures occur outside of manual runs.
- C. It reboots the cloud hypervisor daily.
- D. It forces daily code refactoring.

---

## Answer Key & Explanations

1. **B** — HCP Terraform Agents run inside private networks and establish outbound-only polling connections to execute runs securely behind corporate firewalls.
2. **A** — CLI-driven runs are triggered manually from the local terminal (`terraform plan/apply`); VCS-driven runs trigger automatically on webhook events from GitHub, GitLab, etc.
3. **A** — Variable Sets enable centralized definition and reuse of environment/Terraform variables across projects and workspaces.
4. **B** — Cost estimation evaluates planned delta changes against cloud pricing data, feeding cost data into Sentinel policies and PR UI summaries.
5. **B** — Run Triggers establish sequential execution pipelines between interconnected workspaces (e.g., `networking` $\rightarrow$ `database` $\rightarrow$ `app`).
6. **B** — Projects provide logical containerization and RBAC boundaries for collections of workspaces within an HCP Terraform organization.
7. **A** — Terraform Enterprise Active/Active architecture distributes worker nodes across multiple VMs/instances backed by managed database and object storage tiers.
8. **B** — Dynamic Provider Credentials use OIDC federation with cloud STS/IAM endpoints to issue ephemeral, temporary credentials per run.
9. **B** — In `Local` execution mode, CLI operations execute on the engineer's workstation, using HCP Terraform purely as a remote state and locking backend.
10. **B** — Continuous Validation periodically evaluates `check` blocks on live infrastructure without requiring an engineer to trigger a run, alerting to post-deployment drift.
