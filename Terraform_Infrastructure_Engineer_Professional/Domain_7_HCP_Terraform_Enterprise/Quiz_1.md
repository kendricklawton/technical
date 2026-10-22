# Domain 7: HCP Terraform / Enterprise Operations — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An enterprise needs to manage infrastructure inside an isolated on-premises VMware vSphere cluster that has no inbound public internet access. Which HCP Terraform execution mode must be configured?
- A. Remote Execution on HashiCorp-hosted runners.
- B. Agent Execution using self-hosted `tfc-agent` instances deployed within the private on-premises network.
- C. Local Execution only.
- D. Cloud Run.

---

### Question 2
What is the structural relationship between Organizations, Projects, and Workspaces in HCP Terraform?
- A. Workspaces contain Organizations, which contain Projects.
- B. An Organization contains one or more Projects, and each Project contains one or more Workspaces.
- C. Projects are only used for billing.
- D. There is no hierarchy; all are flat lists.

---

### Question 3
An administrator wants to apply AWS region and environment tags across 50 workspaces in the `Payments` project without manually copy-pasting the variables into each workspace. What HCP Terraform feature accomplishes this?
- A. A Project-scoped Variable Set.
- B. A global bash script.
- C. Committing `terraform.tfvars` to a public repository.
- D. Modifying the `.terraform.lock.hcl` file.

---

### Question 4
In HCP Terraform, what happens when Cost Estimation is enabled for an AWS or GCP workspace?
- A. HCP Terraform queries live cloud pricing APIs during the plan phase to calculate the estimated monthly cost increase (or decrease) caused by the proposed plan.
- B. Cloud providers charge a $10 fee per run.
- C. HCP Terraform automatically cancels plans that cost more than $50.
- D. Cost estimation is only available for local state.

---

### Question 5
What is an HCP Terraform "Run Task"?
- A. A scheduled backup of the state file.
- B. An integration point between plan and apply where HCP Terraform sends the plan JSON to an external security or cost vendor (e.g. Snyk, Infracost) and awaits an approval or block response.
- C. A command that restarts failed runners.
- D. A cron job for deleting old workspaces.

---

### Question 6
When an environment variable in an HCP Terraform workspace is marked as "Sensitive", how is it handled?
- A. The value is hashed with MD5 and printed in logs.
- B. The value is masked in the UI and API, write-only access is enforced, and it cannot be viewed by users once saved.
- C. It can only be used once.
- D. It is deleted when the browser closes.

---

### Question 7
In an enterprise with 500 workspaces, how can platform engineers upgrade the default Terraform version across all workspaces systematically?
- A. By deleting and recreating all workspaces.
- B. By updating workspace settings via the HCP Terraform REST API or Terraform provider for TFE (`tfe_workspace.terraform_version`), testing non-production workspaces first.
- C. Terraform versions cannot be upgraded in HCP Terraform.
- D. By clearing the browser cache.

---

### Question 8
What user permission in HCP Terraform allows an engineer to trigger speculative plans and view plan logs, but prohibits them from approving or applying changes to production infrastructure?
- A. Admin permission.
- B. Custom Workspace Permissions with `Runs: Plan` (or standard `Plan` permission).
- C. Read-only permission.
- D. Organization Owner.

---

### Question 9
How do self-hosted `tfc-agent` runners communicate with HCP Terraform?
- A. HCP Terraform initiates inbound SSH connections to the agent on port 22.
- B. The agent initiates outbound HTTPS (port 443) long-polling connections to HCP Terraform APIs to pull queued jobs, eliminating the need for inbound open firewall ports.
- C. Via UDP broadcast.
- D. Through an AWS Direct Connect link only.

---

### Question 10
What is the effect of locking a workspace in HCP Terraform?
- A. All resources in the cloud are immediately stopped.
- B. New plan and apply runs cannot be executed in the workspace until it is manually unlocked, preventing accidental concurrent executions during scheduled maintenance.
- C. The workspace is archived and deleted after 30 days.
- D. The workspace state file is decrypted.

---

## Answer Key & Explanations

1. **B** — `tfc-agent` runners make outbound HTTPS calls to fetch jobs, allowing private/on-prem infrastructure automation without inbound firewall exposure.
2. **B** — HCP Terraform's resource hierarchy is: Organization -> Projects -> Workspaces.
3. **A** — Variable Sets scoped at the Project level automatically cascade shared variables to all child workspaces.
4. **A** — Cost Estimation evaluates planned resources against cloud pricing APIs before apply.
5. **B** — Run Tasks integrate external SAST and FinOps platforms directly into the execution lifecycle.
6. **B** — Sensitive variables in HCP Terraform are write-only to prevent credential exfiltration.
7. **B** — The official `tfe` Terraform provider enables programmatic automation of workspace configurations and engine versions.
8. **B** — Role-Based Access Control allows separating plan trigger privileges from apply execution privileges.
9. **B** — Agents use outbound polling on port 443, making them simple and secure to deploy in restricted network environments.
10. **B** — Locking a workspace halts execution pipelines to allow safe maintenance and debugging.
