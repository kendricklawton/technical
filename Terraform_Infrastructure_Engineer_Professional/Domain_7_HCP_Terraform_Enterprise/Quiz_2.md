# Domain 7: HCP Terraform / Enterprise Operations — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Enterprise Governance & Workspace Management Questions

---

### Question 1: Private Module Publishing in HCP Terraform
When publishing an internal module to the HCP Terraform Private Module Registry:
- A. The repository must be public on GitHub.
- B. The repository must follow the `terraform-<PROVIDER>-<NAME>` naming standard and be tagged with a semantic version (e.g., `v1.0.0`).
- C. The module cannot contain variables.
- D. Modules must be uploaded as zip archives via FTP.

---

### Question 2: Audit Logging in Terraform Enterprise
How do security teams stream audit events (such as workspace deletion, variable modification, or plan approvals) from Terraform Enterprise to an external SIEM (Splunk, Datadog)?
- A. By scraping web pages using Selenium.
- B. By configuring the native Audit Log Streaming API to forward signed JSON payloads to an HTTP event collector endpoint.
- C. Audit logging is not supported.
- D. By reading `/var/log/syslog` on the developer's laptop.

---

### Question 3: Team-Based Workspace Access vs Organization Owners
Why should everyday developers NOT be assigned the "Organization Owner" role in HCP Terraform?
- A. Organization Owners cannot write HCL.
- B. Organization Owners have unrestricted superuser access, including deleting projects, managing billing, revoking SSO, and accessing all sensitive variables across all workspaces.
- C. HCP limits organizations to 1 owner.
- D. It disables cost estimation.

---

### Question 4: Dynamic Provider Credentials in HCP Terraform Workspaces
When using HCP Terraform's native Dynamic Provider Credentials for AWS:
- A. HCP Terraform exchanges its OIDC token directly with AWS STS to assume the configured IAM role for each run, requiring zero static AWS keys in workspace variables.
- B. An administrator must update passwords every 30 days manually.
- C. Static keys are rotated by a nightly cron job.
- D. AWS credentials must be committed to Git.

---

### Question 5: Managing Workspace Destruction
When a workspace is deleted from HCP Terraform via the UI without running `terraform destroy` first:
- A. All real-world cloud resources managed by that workspace are deleted automatically.
- B. The cloud resources remain running unmanaged in the cloud provider, but the state file is permanently destroyed in HCP Terraform, leaving orphaned infrastructure.
- C. HCP Terraform prevents workspace deletion if resources are present in state.
- D. The cloud provider sends an email alert.

---

### Question 6: Workspace Tagging for Governance
What is the primary operational benefit of attaching tags (e.g., `env:prod`, `team:billing`, `pci-dss:true`) to HCP Terraform workspaces?
- A. Tags change the color of the UI buttons.
- B. Tags enable filtering in the UI/API, allow targeted Policy Set enforcement across subsets of workspaces, and facilitate project organization.
- C. Tags reduce state file size.
- D. Tags automatically scale runner memory.

---

### Question 7: High Availability in Terraform Enterprise (FDO vs Active-Active)
In self-hosted Terraform Enterprise Flexible Deployment Options (FDO), how is high availability achieved?
- A. By running a single EC2 instance.
- B. By deploying stateless container nodes behind an application load balancer, backed by external PostgreSQL database and object storage (S3/GCS) with multi-AZ failover.
- C. By syncing laptops via Bluetooth.
- D. FDO does not support high availability.

---

### Question 8: VCS-Driven vs CLI-Driven Workspaces
What is a key difference between a VCS-driven workspace and a CLI-driven workspace in HCP Terraform?
- A. VCS-driven workspaces automatically trigger plans upon Git pull requests and applies upon branch merges; CLI-driven workspaces execute runs initiated from a terminal using `terraform plan/apply`.
- B. CLI-driven workspaces do not use state.
- C. VCS-driven workspaces cannot use modules.
- D. CLI-driven workspaces cannot deploy to AWS.

---

### Question 9: Automated Notifications in HCP Terraform
Which notification destinations are natively supported by HCP Terraform workspace webhooks?
- A. Slack, Microsoft Teams, generic webhooks, and email.
- B. SMS text messages only.
- C. Twitter/X posts.
- D. Fax.

---

### Question 10: Speculative Plan Visibility on PRs
When a GitHub pull request triggers a speculative plan in an HCP Terraform VCS-driven workspace, where is the plan result surfaced?
- A. Only in the developer's local terminal.
- B. Directly on the GitHub Pull Request checks status widget and PR comments, linking back to the detailed HCP Terraform run interface.
- C. In the company Slack general channel.
- D. It is sent via postal mail.

---

## Answer Key & Explanations

1. **B** — Standard 3-part naming and Git SemVer tags are required to register modules in the Private Module Registry.
2. **B** — Audit Log Streaming exports immutable real-time security events directly to enterprise SIEM platforms.
3. **B** — The Organization Owner role represents total administrative authority and must be strictly restricted.
4. **A** — HCP Terraform's native OIDC integration eliminates static credentials through automated STS token exchange.
5. **B** — Deleting a workspace without destroying infrastructure orphans the live resources, requiring manual cloud-level cleanup.
6. **B** — Workspace tags provide metadata hooks for filtering and targeting Policy as Code rules.
7. **B** — Production Terraform Enterprise architectures decouple stateless application containers from clustered storage engines.
8. **A** — VCS-driven runs automate the continuous integration feedback loop based on Git repository events.
9. **A** — HCP Terraform provides built-in webhook notification handlers for chatOps and automated orchestration.
10. **B** — Native VCS status integration surfaces pass/fail indicators and plan diffs directly in developer pull request views.
