# Domain 1: Infrastructure as Code (IaC) with Terraform — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 1a)
An engineering team is migrating from custom Python Boto3 scripts to HashiCorp Terraform. What is the primary paradigm shift when moving from imperative scripts to Terraform's declarative HCL model?
- A. Declarative configurations do not require cloud provider API credentials.
- B. Terraform compiles HCL into kernel-level C binaries that run inside the guest operating system.
- C. Engineers define the desired end state of the infrastructure, and Terraform automatically calculates the dependency graph and necessary CRUD actions to reach that state.
- D. Engineers must manually specify the exact chronological sequence of every REST API call and rollback step.

---

### Question 2 (Objective 1b)
A DevOps engineer applies a Terraform configuration that creates an AWS VPC and three subnets. Ten minutes later, without modifying any `.tf` files or cloud resources, a colleague runs `terraform apply` against the same state. Why does Terraform report `No changes. Your infrastructure matches the configuration` instead of creating a second VPC and three duplicate subnets?
- A. Because Terraform ignores resources that do not have a `count` meta-argument.
- B. Because AWS accounts are limited to a single VPC per region.
- C. Because Terraform is idempotent and reconciles desired configuration against tracked state before taking action.
- D. Because `terraform apply` can only be executed once per day.

---

### Question 3 (Objective 1c)
Which statement accurately describes how Terraform supports multi-cloud and hybrid-cloud workflows?
- A. Terraform provides a consistent HCL syntax and unified lifecycle workflow (`init`, `plan`, `apply`, `destroy`) across cloud, on-premises, and SaaS APIs via provider plugins.
- B. A single generic resource block (`resource "cloud_vm" "app"`) automatically deploys to AWS, Azure, and VMware vSphere without changing resource types.
- C. Terraform requires AWS CloudFormation to act as a proxy for Azure and Google Cloud deployments.
- D. Terraform installs a persistent background agent on every managed server to pull configurations.

---

### Question 4 (Objective 1b)
Which of the following is NOT an advantage of adopting Infrastructure as Code (IaC) with Version Control (Git)?
- A. Automated reproducibility of identical staging and production environments.
- B. Auditable commit history showing who changed infrastructure definitions, when, and why.
- C. Peer review of infrastructure changes via Pull Requests and speculative plans before production changes occur.
- D. Complete elimination of cloud provider compute and storage billing charges.

---

### Question 5 (Objective 1c)
An organization uses VMware vSphere in its on-premises data center, Amazon EKS in AWS for container workloads, and Cloudflare for global DNS and WAF protection. How can Terraform manage this hybrid architecture?
- A. By declaring the `vsphere`, `aws`, `kubernetes`, and `cloudflare` providers within the Terraform configuration, enabling cross-platform references in a single workflow.
- B. Terraform can only manage the AWS resources; vSphere and Cloudflare must be configured manually.
- C. By converting the HCL files into PowerShell scripts for vSphere and YAML for Cloudflare.
- D. By installing the HCP Terraform operating system on the physical data center routers.

---

### Question 6 (Objective 1a)
How does Terraform's communication architecture differ from pull-based configuration management tools like Chef or Puppet?
- A. Terraform requires a long-running `terraformd` agent inside every virtual machine.
- B. Terraform is agentless and push-based, communicating directly with target platform APIs over HTTPS via provider plugins.
- C. Terraform cannot manage virtual machines once they are booted.
- D. Terraform communicates exclusively over SSH port 22 to configure cloud VPCs.

---

### Question 7 (Objective 1b)
What term describes the condition when an administrator manually modifies a security group rule in the cloud provider's web console, causing the live cloud resource to differ from the codified `.tf` configuration?
- A. Provider aliasing
- B. State locking
- C. Resource idempotence
- D. Configuration drift

---

### Question 8 (Objective 1c)
In an enterprise toolset, why do platform teams frequently pair **Terraform** with **Ansible** or **Packer**?
- A. Because Terraform cannot provision virtual machines or networking.
- B. Because Packer builds immutable machine images, Terraform provisions Day 1 cloud infrastructure using those images, and Ansible handles Day 2 in-guest OS configuration management.
- C. Because Terraform is only licensed for development environments.
- D. Because Ansible is required to generate Terraform's `.tfstate` file.

---

### Question 9 (Objective 1b)
Why does "ClickOps" (manually provisioning production infrastructure through web consoles) increase operational risk in large enterprises?
- A. Web consoles do not support TLS encryption.
- B. Cloud providers charge double for resources created via the web console.
- C. Web consoles cannot provision relational databases.
- D. Manual clicks lack pre-execution peer review, cannot be automatically linted or policy-checked, and create undocumented "snowflake" configurations that are difficult to reproduce during disaster recovery.

---

### Question 10 (Objective 1c)
True or False: Because Terraform is service-agnostic, a single root module can provision an AWS S3 bucket, pass its domain name to a Cloudflare DNS record, and create a Datadog synthetic monitor for that endpoint in one `terraform apply`.
- A. True
- B. False

---

## Answer Key & Rationales

1. **C** — Terraform is **declarative**: you declare the target end state in HCL, and Terraform's core engine builds a dependency graph to compute and execute the exact delta.
2. **C** — **Idempotence** ensures that applying the same configuration repeatedly yields the identical infrastructure state without duplicating existing resources.
3. **A** — Terraform unifies the workflow (`init/plan/apply/destroy`) and language (HCL) across clouds and SaaS platforms via provider plugins, though resource schemas remain specific to each provider.
4. **D** — While IaC helps optimize cloud costs (e.g., tearing down ephemeral test environments), cloud providers still bill for active compute and storage resources.
5. **A** — Terraform's provider ecosystem supports hybrid and multi-cloud topologies (`vsphere`, `aws`, `kubernetes`, `cloudflare`) within a unified workflow.
6. **B** — Terraform is **push-based and agentless**; it invokes cloud/service APIs directly via provider plugins.
7. **D** — Out-of-band manual changes create **configuration drift** between real-world infrastructure and the desired state in code.
8. **B** — Packer builds golden VM images (Day 0), Terraform provisions cloud infrastructure (Day 1), and Ansible manages in-guest OS configuration (Day 2).
9. **D** — Manual console configuration is unversioned, error-prone, impossible to validate automatically before execution, and leads to non-reproducible snowflake environments.
10. **A (True)** — Cross-provider attribute references allow Terraform to chain outputs from one provider (`aws_s3_bucket`) directly into resources of other providers (`cloudflare_record`, `datadog_synthetics_test`).
