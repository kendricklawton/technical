# Domain 8: HCP Terraform — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 8c — New in 004!)
An enterprise has 120 workspaces in its HCP Terraform organization across 6 engineering departments. The platform team wants to grant the "Payments Engineering" team administrative access to all 20 Payments workspaces and automatically share a common set of AWS OIDC role variables across only those 20 workspaces—including any new Payments workspaces created in the future—without managing permissions on each workspace individually. Which HCP Terraform feature accomplishes this?
- A. CLI Workspaces (`terraform workspace new payments`)
- B. HCP Terraform **Projects** combined with a **Project-scoped Variable Set** and Project-level team permissions
- C. Sentinel Advisory Policies
- D. Storing a shared `terraform.tfvars` file in the `/tmp` directory

---

### Question 2 (Objective 8c — New in 004!)
In HCP Terraform, the variable `aws_region` is defined in three places:
1. A **Global Variable Set** applied to all workspaces: `aws_region = "us-east-1"`
2. A **Project-scoped Variable Set** applied to the `EU-Retail` Project: `aws_region = "eu-west-1"`
3. A **Workspace Variable** set directly inside the `eu-checkout-dr` workspace (which belongs to the `EU-Retail` Project): `aws_region = "eu-central-1"`

When a run executes in the `eu-checkout-dr` workspace, what value of `aws_region` is used?
- A. `"us-east-1"`
- B. `"eu-west-1"`
- C. `"eu-central-1"`
- D. The run fails due to a duplicate variable collision error

---

### Question 3 (Objective 8b)
What is the primary security advantage of configuring **Dynamic Provider Credentials** in HCP Terraform over storing `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as sensitive environment variables?
- A. Dynamic Provider Credentials use OpenID Connect (OIDC) Workload Identity Federation to request short-lived, single-run credentials from the cloud provider (AWS, GCP, Azure, or Vault) at runtime, eliminating the risk of static, long-lived cloud credentials being leaked or rotated manually.
- B. Dynamic Provider Credentials bypass all cloud IAM permission checks.
- C. Dynamic Provider Credentials allow running Terraform without an internet connection.
- D. Dynamic Provider Credentials encrypt `.tf` source code files in GitHub.

---

### Question 4 (Objective 8b)
At what point in the HCP Terraform run lifecycle are **Sentinel** and **Open Policy Agent (OPA)** policy checks evaluated?
- A. Before `terraform init` downloads providers
- B. **After `terraform plan` completes (and after Cost Estimation), but before `terraform apply` executes**
- C. Only after `terraform apply` has already created the cloud resources
- D. Only when `terraform destroy` is executed

---

### Question 5 (Objective 8b)
Which Sentinel policy enforcement level halts a run when a policy fails, but allows an authorized team leader or organization owner with override permissions to manually approve and proceed with the `apply`?
- A. `Advisory`
- B. `Soft Mandatory`
- C. `Hard Mandatory`
- D. `Warning-Only`

---

### Question 6 (Objective 8b)
How do **Health Assessments** (Drift Detection and Continuous Validation) work in HCP Terraform?
- A. They continuously reboot production EC2 instances every 24 hours.
- B. They periodically run background checks on a schedule (without needing a new Git commit or manual run) to detect if real-world cloud resources have drifted from state and to re-evaluate `check` blocks / custom conditions, alerting operators when drift or assertion failures occur.
- C. They automatically run `terraform destroy` on idle workspaces.
- D. They replace the need for cloud load balancers.

---

### Question 7 (Objective 8c)
A platform team manages a `shared-network-vpc` workspace and an `eks-app-cluster` workspace in HCP Terraform. Whenever a network change is successfully applied in `shared-network-vpc`, the team wants HCP Terraform to automatically queue a run in `eks-app-cluster`. Which feature should they configure?
- A. Run Triggers
- B. Speculative Plans
- C. Private Registry Webhooks
- D. Local Execution Mode

---

### Question 8 (Objective 8a & 8d)
Which HCL block inside `terraform { ... }` is the modern standard for connecting a configuration directory to HCP Terraform?
- A. `backend "s3" { ... }`
- B. `cloud { organization = "..." workspaces { ... } }`
- C. `provider "hcp_terraform" { ... }`
- D. `module "cloud" { ... }`

---

### Question 9 (Objective 8d)
An enterprise needs HCP Terraform to provision resources inside an air-gapped on-premises VMware vSphere cluster and an isolated private AWS VPC that do not allow inbound internet traffic from HCP Terraform's SaaS IPs. Which execution mode solves this requirement?
- A. Local Execution Mode on every developer's personal laptop
- B. **Agent Execution Mode** using **HCP Terraform Agents** (`tfc-agent`) deployed inside the private network, which poll HCP Terraform outbound over HTTPS
- C. Opening port 22 (`0.0.0.0/0`) on the corporate perimeter firewall
- D. Disabling state locking

---

### Question 10 (Objective 8a)
When a developer opens a Pull Request in GitHub against a branch connected to a **VCS-driven HCP Terraform workspace**, what type of run is automatically triggered?
- A. An immediate `terraform apply -auto-approve` in production
- B. A **Speculative Plan**, which shows the proposed diff, cost estimate, and policy results on the Pull Request but cannot be applied until the PR is merged
- C. A `terraform destroy` dry run
- D. A state migration to the developer's local laptop

---

## Answer Key & Rationales

1. **B** — **HCP Terraform Projects** (tested in 004 Objective `8c`) group workspaces logically so teams can be granted RBAC permissions and **Variable Sets** at the Project level rather than managing individual workspaces.
2. **C** — More specific scopes always override broader scopes in HCP Terraform: **Workspace Variable** (`"eu-central-1"`) > **Workspace-scoped Variable Set** > **Project-scoped Variable Set** (`"eu-west-1"`) > **Global Variable Set** (`"us-east-1"`).
3. **A** — **Dynamic Provider Credentials** exchange a short-lived workload identity OIDC token signed by HCP Terraform for temporary cloud provider credentials on each individual run.
4. **B** — Policy checks (Sentinel / OPA) execute **post-plan, pre-apply** so they can inspect the planned resource changes, configuration, and cost estimate before any real-world infrastructure is mutated.
5. **B** — **Soft Mandatory** blocks the run on failure but allows authorized users to override. **Hard Mandatory** cannot be overridden by anyone; **Advisory** logs a warning without blocking.
6. **B** — HCP Terraform **Health Assessments** run scheduled background drift detection and continuous validation (`check` blocks, `precondition`/`postcondition`) to catch out-of-band drift and broken health assertions between deployments.
7. **A** — **Run Triggers** link upstream source workspaces to downstream target workspaces, automatically queuing a downstream run when an upstream `apply` completes.
8. **B** — The `cloud` block inside `terraform { ... }` is the dedicated syntax for configuring HCP Terraform workspaces (by `name` or by `tags`/`project`).
9. **B** — **HCP Terraform Agents** run inside the customer's private network boundary and initiate outbound-only HTTPS connections to HCP Terraform to pull and execute jobs.
10. **B** — Pull Requests trigger non-destructive **Speculative Plans** so reviewers can verify the execution plan and policy checks before merging.
