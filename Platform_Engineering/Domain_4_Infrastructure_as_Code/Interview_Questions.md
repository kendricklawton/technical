# PE Domain 4: Infrastructure as Code — 25 Core Interview Questions

Master these questions across Technical Screening, IaC Deep Dive (Mock 4), and Staff Architecture rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What is the purpose of the Terraform state file, and why must it be secured?
- **Short Answer:** The state file maps declarative HCL configurations to real-world cloud resource IDs and attributes. It must be secured because it contains sensitive plaintext data (database passwords, private keys) and is critical to preventing duplicate or destructive resource provisioning.
- **Deep Dive:** Without state, Terraform cannot know if an HCL resource block already exists in the cloud or needs to be created. State should be stored in an encrypted remote backend (e.g., S3 with SSE-KMS and bucket versioning) with strict IAM least-privilege policies.
- **Trade-offs & Realities:** Even if variables are marked `sensitive = true`, their plaintext values are still written to `.tfstate`. Access to the state bucket must be treated with the same severity as root cloud access.
- **Follow-up Probes:** "How do you protect state files from concurrent execution corruption?" (Distributed locking via DynamoDB or S3 Object Lock).

### Q2: What is the difference between `terraform plan` and `terraform apply`?
- **Short Answer:** `plan` refreshes existing state, evaluates configuration, and produces a predictive diff of proposed changes. `apply` executes mutating cloud API calls to align real infrastructure with desired state and updates the state file.
- **Deep Dive:** Best practice decouples them in CI: `plan` outputs an immutable binary plan file (`-out=tfplan`), which is reviewed, validated via Policy as Code, and then passed explicitly to `apply` to ensure zero drift occurs between review and execution.
- **Trade-offs & Realities:** Running `apply` without reviewing a plan risks unintended destruction of stateful infrastructure.
- **Follow-up Probes:** "What does exit code 2 mean in `terraform plan -detailed-exitcode`?" (The plan succeeded and identified a non-empty diff / infrastructure drift).

### Q3: How do declarative `moved` blocks work in Terraform 1.1+?
- **Short Answer:** They allow engineers to record resource renames or module refactorings directly in HCL code. When running `plan`, Terraform recognizes the move and updates the state file address without destroying or recreating the physical resource.
- **Deep Dive:** Syntax: `moved { from = aws_s3_bucket.old; to = module.storage.aws_s3_bucket.new }`. This eliminates the need to execute error-prone manual `terraform state mv` commands across distributed engineering teams.
- **Trade-offs & Realities:** `moved` blocks should remain in the codebase for at least one release cycle so all environments and developer branches absorb the state migration.
- **Follow-up Probes:** "What tool was used before `moved` blocks?" (Manual imperative CLI command `terraform state mv`).

### Q4: Explain the difference between implicit and explicit dependencies in Terraform.
- **Short Answer:** An implicit dependency is created automatically when one resource references an attribute of another (e.g., `vpc_id = aws_vpc.main.id`). An explicit dependency is declared manually using `depends_on = [aws_vpc.main]`.
- **Deep Dive:** Terraform builds its execution DAG from implicit dependencies. Explicit dependencies should only be used when an ordering constraint exists that is invisible to Terraform (e.g., an S3 bucket policy requiring an IAM role to exist first, even if no attribute is referenced).
- **Trade-offs & Realities:** Overusing `depends_on` serializes graph execution, drastically slowing down plan and apply times.
- **Follow-up Probes:** "Can explicit dependencies cause dependency cycles?" (Yes, if circular references are created).

### Q5: What is Policy as Code (PaC), and why is it used in IaC pipelines?
- **Short Answer:** The practice of writing programmatic rules (e.g., using OPA Rego or Sentinel) that validate infrastructure definitions against organizational security, cost, and compliance baselines before provisioning.
- **Deep Dive:** PaC intercepts pull requests, evaluating the JSON output of `terraform plan`. It automatically blocks changes that violate rules (e.g., unencrypted disks, open SSH ports, missing mandatory billing tags), shifting security left.
- **Trade-offs & Realities:** Policies must provide clear, actionable remediation messages; otherwise, they frustrate developers and slow down delivery velocity.
- **Follow-up Probes:** "Name two popular open-source PaC tools." (Open Policy Agent / Conftest and Trivy).

### Q6: What is a Terraform remote backend?
- **Short Answer:** A centralized storage location (such as AWS S3, Google Cloud Storage, or HCP Terraform, formerly Terraform Cloud) where the state file is stored and shared among team members and CI runners, typically featuring encryption and locking.
- **Deep Dive:** Remote backends prevent team members from stepping on each other's changes. They eliminate the risk of committing state files containing plaintext secrets to Git repositories.
- **Trade-offs & Realities:** Misconfiguring backend credentials blocks all team deployments.
- **Follow-up Probes:** "Why should S3 state buckets have versioning enabled?" (To allow rollbacks if a state file is accidentally corrupted or deleted).

### Q7: What does `terraform validate` verify?
- **Short Answer:** It verifies syntactic correctness, internal schema validity, attribute types, and argument names across HCL files without accessing remote state or cloud provider APIs.
- **Deep Dive:** It runs fast and offline, making it ideal as a pre-commit hook or early linting stage in CI pipelines before running expensive `plan` operations.
- **Trade-offs & Realities:** `validate` does not check if an AMI ID exists or if you have cloud permissions; it only checks code correctness.
- **Follow-up Probes:** "Does `terraform validate` detect provider authentication errors?" (No; provider authentication is evaluated during plan/apply).

### Q8: What are dynamic blocks in HCL, and when should you use them?
- **Short Answer:** A construct that dynamically generates repeated nested configuration blocks inside a resource based on a complex collection variable (list or map).
- **Deep Dive:** Useful for resources with repeatable nested blocks, such as `ingress` rules in security groups or `rule` blocks in S3 lifecycle configurations.
- **Trade-offs & Realities:** Overusing dynamic blocks makes HCL difficult to read and reason about. Standard static declarations should be preferred when the structure is fixed.
- **Follow-up Probes:** "Can you use dynamic blocks on top-level resource blocks?" (No; dynamic blocks only work on nested blocks inside resources or data sources).

### Q9: What is the purpose of `terraform.lock.hcl`?
- **Short Answer:** It pins the exact versions and multi-platform cryptographic checksums of provider plugins to guarantee deterministic, tamper-proof builds across all environments.
- **Deep Dive:** When a provider is downloaded, Terraform records its SHA256 checksums in the lock file. If a malicious party attempts to compromise a provider binary, Terraform detects the hash mismatch and halts.
- **Trade-offs & Realities:** The lock file must be committed to Git. Upgrades must be executed intentionally via `terraform init -upgrade`.
- **Follow-up Probes:** "How do you generate checksums for both Linux and macOS?" (`terraform providers lock -platform=linux_amd64 -platform=darwin_arm64`).

### Q10: What is configuration drift in cloud infrastructure?
- **Short Answer:** The divergence between the real-world state of cloud infrastructure and the declared configuration recorded in the IaC codebase.
- **Deep Dive:** Drift occurs when engineers make manual console changes, emergency CLI hotfixes, or when cloud providers alter default attributes. Detected via `terraform plan -detailed-exitcode` and corrected by applying IaC or importing changes.
- **Trade-offs & Realities:** Unmonitored drift leads to unexpected resource destruction when Terraform is subsequently applied.
- **Follow-up Probes:** "How does Crossplane handle drift compared to Terraform?" (Crossplane continuously reconciles and corrects drift 24/7, whereas Terraform only detects drift during scheduled pipeline runs).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: How do you design and structure Terraform repositories to minimize blast radius and execution latency?
- **Short Answer:** Partition infrastructure into decoupled layers (e.g., networking, security, compute, data) with separate state files, using Terraform remote state data sources or SSM Parameter Store / Consul for cross-layer references.
- **Deep Dive:** Instead of a single 10,000-line root module, split into:
  - `layers/01-network/` (VPCs, Transit Gateways) — modified quarterly.
  - `layers/02-iam-security/` (Roles, KMS) — modified monthly.
  - `layers/03-eks-clusters/` (EKS, node pools) — modified bi-weekly.
  - `layers/04-apps/` (App S3 buckets, RDS) — modified daily.
  Each layer has its own backend and lock, preventing changes in apps from risking core network integrity.
- **Trade-offs & Realities:** Managing cross-layer dependencies requires discipline; sharing data via SSM parameters or remote state keys must be versioned.
- **Follow-up Probes:** "What tool orchestrates multi-layer Terraform execution?" (Terragrunt or Atmos).

### Q12: Walk through the mechanics of declarative `import` blocks in Terraform 1.5+.
- **Short Answer:** Declare `import { to = <resource_address>; id = "<cloud_id>" }` in HCL. Run `terraform plan` to preview the import. If the resource code is not yet written, use `-generate-config-out=generated.tf` to have Terraform auto-generate the HCL.
- **Deep Dive:** This shifts imports from an imperative CLI command (`terraform import`) into the standard declarative Git review workflow. Reviewers can inspect the imported attributes in the PR diff before the state is updated during apply.
- **Trade-offs & Realities:** Auto-generated HCL often contains verbose default attributes that should be manually cleaned up for maintainability.
- **Follow-up Probes:** "What happens to the `import` block after apply completes?" (It can remain in code safely or be removed once recorded in state).

### Q13: Explain how `terraform test` works natively in Terraform 1.6+ and how mock providers are utilized.
- **Short Answer:** Native tests are written in `.tftest.hcl` files using `run` blocks with assertions. `mock_provider` blocks allow testing module logic and preconditions completely in memory without cloud credentials or network calls.
- **Deep Dive:** Tests support two execution commands:
  - `command = plan`: Dry-run unit tests validating planned attributes against assertions.
  - `command = apply`: Integration tests that create temporary infrastructure, assert postconditions, and automatically tear down resources when the test finishes.
- **Trade-offs & Realities:** Replaces heavyweight external frameworks like Terratest for standard module verification, drastically speeding up CI test pipelines.
- **Follow-up Probes:** "What can you assert in a `tftest` block?" (Any boolean expression evaluating resource attributes, variables, or outputs).

### Q14: How does Crossplane achieve self-service infrastructure for developers without exposing raw cloud credentials?
- **Short Answer:** Crossplane allows platform teams to define Composite Resource Definitions (XRDs) representing simplified abstractions (e.g., `kind: Database`). Product teams declare the custom resource in their Kubernetes namespace, and Crossplane reconciles it using cluster-level cloud credentials.
- **Deep Dive:** XRDs define the API schema; Compositions define the underlying cloud resources (AWS RDS, security groups, subnet groups). The developer never sees Terraform or AWS IAM; they interact solely with standard Kubernetes manifests via GitOps.
- **Trade-offs & Realities:** High initial setup complexity and steep learning curve for platform engineers.
- **Follow-up Probes:** "How does Crossplane handle secret distribution?" (It publishes database credentials directly into a Kubernetes Secret in the developer's namespace).

### Q15: How do you prevent and resolve stuck state locks in automated CI pipelines?
- **Short Answer:** Configure CI jobs with graceful termination timeouts, avoid hard killing runners (`kill -9`), investigate the running process before unlocking, and use `terraform force-unlock <LOCK-ID>` with audit logging.
- **Deep Dive:** When a lock error occurs, the output displays the Lock ID, owner, and timestamp. The platform engineer must verify via CI runner metrics that the process is truly dead before forcing the lock open.
- **Trade-offs & Realities:** Forcing a lock while a process is actively writing corrupts the state file irreparably.
- **Follow-up Probes:** "Can S3 Object Lock replace DynamoDB locking?" (Yes, native S3 locking with conditional writes was introduced in recent Terraform/AWS provider releases).

### Q16: How do you enforce security guardrails using Conftest and OPA Rego in a GitHub Actions pipeline?
- **Short Answer:** Generate the Terraform plan as JSON (`terraform show -json tfplan.binary > plan.json`), execute `conftest test plan.json -p policy/`, and fail the CI job with exit code 1 if any `deny` rules match.
- **Deep Dive:** Policies inspect `resource_changes`. Rego rules can enforce encryption, restrict CIDR blocks, require resource tags, and limit VM sizes. Conftest outputs violation messages directly in the PR comments.
- **Trade-offs & Realities:** Rego has a learning curve. Teams must write tests for their Rego policies to prevent policy bugs.
- **Follow-up Probes:** "What is the difference between Conftest and Checkov?" (Checkov has hundreds of pre-built static rules; Conftest/OPA is a generic policy engine for custom organization rules).

### Q17: What are preconditions and postconditions in HCL, and how do they differ from variable validations?
- **Short Answer:** Variable validations check input syntax and values before graph construction. Preconditions validate assumptions before a specific resource is evaluated. Postconditions validate the actual attributes returned by the cloud provider after resource creation.
- **Deep Dive:** Postconditions can verify dynamic data returned by the cloud (e.g., asserting that an allocated EBS volume has `encrypted == true` or that an IP address is within an approved corporate CIDR block). If a postcondition fails, Terraform halts immediately, preventing downstream resources from being provisioned.
- **Trade-offs & Realities:** Halting on a failed postcondition leaves the resource created in the cloud but marks it tainted in state.
- **Follow-up Probes:** "Where are preconditions defined?" (Inside the `lifecycle` block of resources, data sources, or outputs).

### Q18: Compare Atlantis and Spacelift / HCP Terraform for PR-driven GitOps workflows.
- **Short Answer:** Atlantis is a self-hosted open-source bot that executes `plan` and `apply` via PR comments (`atlantis plan`). Spacelift/HCP Terraform are managed SaaS platforms with rich UI, built-in policy engines (OPA/Sentinel), drift detection, and role-based access control.
- **Deep Dive:** Atlantis is lightweight, cost-free, and runs inside your own VPC, keeping cloud credentials internal. However, it requires maintaining the runner infrastructure and lacks built-in governance dashboards. SaaS solutions provide audit trails, state history, and automated concurrency queues out of the box.
- **Trade-offs & Realities:** Enterprise compliance often demands SaaS governance or dedicated internal platform teams to maintain Atlantis.
- **Follow-up Probes:** "What is a speculative plan in HCP Terraform?" (A plan executed on an unmerged PR to preview diffs without locking state).

### Q19: How do you design an automated drift detection system across 100+ Terraform states?
- **Short Answer:** Run a nightly scheduled CI matrix job that iterates across all state directories, executing `terraform plan -detailed-exitcode`. Capture exit code 2, extract the resource diff, publish metrics to Prometheus/Datadog, and alert owning teams via Slack.
- **Deep Dive:** Architecture:
  1. GitHub Actions scheduled workflow triggered at 02:00 UTC.
  2. Runs `terraform init` and `terraform plan -detailed-exitcode -no-color`.
  3. If exit code is 2, trigger a webhook sending the plan summary to `#platform-alerts` and increment a metric `terraform_drift_detected{state="networking"}`.
- **Trade-offs & Realities:** Running 100 plans concurrently can trigger cloud API rate limits (e.g., AWS Describe API throttling). Implement rate limiting and randomized jitter.
- **Follow-up Probes:** "What AWS feature helps avoid API rate limits during plans?" (AWS Config or caching read requests).

### Q20: Explain the concept and design of an AWS Multi-Account Landing Zone.
- **Short Answer:** An enterprise multi-account topology structured via AWS Organizations into distinct functional accounts (Management, Security, Network, Shared Services, and isolated Workload accounts) with centralized billing, egress routing, and Service Control Policies (SCPs).
- **Deep Dive:** Blast-radius isolation is enforced at the account boundary. A compromise in `dev-checkout` has zero network or IAM access to `prod-payments`. Central egress proxies in the Network account inspect all outbound traffic, while the Security account aggregates CloudTrail and GuardDuty findings.
- **Trade-offs & Realities:** High initial setup complexity; requires automated account vending machines (AVM) to scale.
- **Follow-up Probes:** "What is an SCP?" (Service Control Policy: organization-level boundary limiting maximum allowable permissions for member accounts).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Design an automated Account Vending Machine (AVM) architecture capable of vending fully-compliant cloud accounts in under 15 minutes.
- **Candidate Framework:**
  - *Trigger:* Developer requests a new account via Backstage/Portal, specifying team owner, environment (`dev`/`prod`), and billing cost center.
  - *Orchestration:* Portal triggers an EventBridge event that invokes a step function or HCP Terraform workspace.
  - *Account Creation:* Invokes AWS Organizations API to create account under the appropriate Organizational Unit (OU).
  - *Baseline Bootstrapping:* Automatically runs Terraform to provision account baselines:
    - VPC with Transit Gateway attachment.
    - Default security groups (blocking all ingress).
    - Cross-account IAM roles for deployment runners.
    - KMS customer managed keys (CMK) with automated rotation.
    - Service Control Policies (SCPs) preventing CloudTrail tampering and restricting regions.
  - *Catalog Registration:* Registers account metadata in the Software Catalog and notifies the developer via Slack with credentials/access instructions.
- **Red Flags:** Requiring manual tickets or failing to automate VPC network attachments and security baselines.

### Q22: A developer accidentally deletes an entire VPC and its subnets outside of Terraform. How do you lead disaster recovery and state realignment?
- **Candidate Framework:**
  - *Mitigate First:* Assess affected workloads. If active production traffic is impacted, execute the disaster recovery plan (fail over DNS via Route53 to secondary region or backup cluster).
  - *Assess State Discrepancy:* Run `terraform plan`. Terraform will recognize the missing VPC and attempt to recreate it.
  - *Evaluate Dependencies:* Re-creating the VPC generates *new* VPC and Subnet IDs. All dependent resources (EKS clusters, RDS databases) will either fail or plan recreations if tied to the old IDs.
  - *Recovery Options:*
    - Option A: Reapply Terraform to generate the new VPC, update routing references, and redeploy compute workloads.
    - Option B: If stateful databases still exist in other networks, reconstruct VPC with identical CIDRs, update peering, and patch references.
  - *Prevent Recurrence:* Apply AWS termination protection, delete protection, and attach an SCP forbidding `ec2:DeleteVpc` for all non-platform IAM roles.
- **Red Flags:** Panicking and manually creating resources in AWS Console without recording them in Terraform.

### Q23: How would you architect a zero-trust credential delivery model for Terraform pipelines running across multi-cloud environments?
- **Candidate Framework:**
  - *Eliminate Static Credentials:* Ban long-lived AWS Access Keys and GCP Service Account JSON keys entirely from CI/CD runners.
  - *OIDC Federation:* Configure GitHub Actions / GitLab CI with OpenID Connect (OIDC) identity provider integration directly with AWS IAM and GCP Workload Identity.
  - *Ephemeral Token Exchange:* When a CI job runs:
    1. Runner requests a cryptographically signed OIDC JWT token from GitHub.
    2. Runner exchanges JWT with AWS STS via `sts:AssumeRoleWithWebIdentity`, asserting claims (`repository: org/repo`, `ref: refs/heads/main`).
    3. AWS STS returns temporary credentials valid for 1 hour.
  - *Least Privilege Scoping:* Role permissions scoped strictly to the resources managed by that specific repository and environment.
- **Red Flags:** Storing static AWS access keys in GitHub Secrets.

### Q24: Evaluate the architectural trade-offs between Terraform/OpenTofu and Crossplane for an enterprise building an Internal Developer Platform (IDP).
- **Candidate Framework:**
  - *Terraform / OpenTofu:*
    - *Strengths:* Massive ecosystem, mature providers for every SaaS/cloud, well-known HCL syntax, battle-tested plan review workflows.
    - *Weaknesses:* Passive execution (drift only detected when runs are triggered); static state files; requires complex pipeline orchestration (Atlantis/Spacelift) to expose as self-service.
  - *Crossplane:*
    - *Strengths:* Kubernetes-native; active continuous reconciliation heals drift instantly; exposes simple CRDs to developers; perfect fit for teams already living in Kubernetes.
    - *Weaknesses:* Heavy control plane footprint; managing XRD/Composition complexity; smaller community than Terraform; provider crashes can destabilize the Kubernetes control plane.
  - *Hybrid Recommendation:* Use Terraform for foundational landing zones, VPCs, and Kubernetes clusters; deploy Crossplane inside those clusters to vend self-service application resources (databases, queues, S3 buckets) to developers.
- **Red Flags:** Dogmatically asserting that one tool completely eliminates the need for the other.

### Q25: How do you design an automated migration strategy to transfer 300 microservice databases from an old Terraform module to a new v2 module without downtime?
- **Candidate Framework:**
  - *Avoid Monolithic Big-Bang:* Never attempt to upgrade 300 services simultaneously.
  - *Maintain Backward Compatibility:* Support Module v1 and Module v2 concurrently in the module registry.
  - *Declarative `moved` Blocks:* Ensure Module v2 includes internal `moved` blocks so internal resource address renames are transparent to consumers.
  - *Lighthouse Validation:* Test migration on 2 non-critical internal services first, validating zero-downtime plan and apply diffs.
  - *Automated Bot PR Campaign:* Write a script (or use Renovate) that opens PRs bumping the module version across all 300 repositories, running automated `terraform plan` to verify `0 to destroy`.
  - *Scheduled Rollout Waves:* Merge and apply in staged cohorts (e.g., 20 services per day during maintenance windows).
- **Red Flags:** Proposing manual state surgery across 300 repos or failing to test `plan` diffs before merging.
