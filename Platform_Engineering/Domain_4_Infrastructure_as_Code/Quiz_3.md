# PE Domain 4: Infrastructure as Code — Scenario Quiz 3

Test your knowledge on Policy as Code (OPA/Conftest), Crossplane, and enterprise Landing Zones.

---

### Question 1
Your company requires that no AWS S3 bucket can ever be provisioned without server-side encryption. Where in the engineering lifecycle is it most cost-effective and secure to enforce this policy using Policy as Code (PaC)?

A) By manually reviewing AWS CloudTrail audit logs 30 days after deployment.  
B) In the pull request CI/CD pipeline, by using Conftest / OPA to evaluate the JSON output of `terraform plan` and blocking the PR merge if an unencrypted bucket is detected.  
C) By asking developers to promise they enabled encryption.  
D) By writing a Python cron job that deletes unencrypted buckets at midnight.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Shifting security left into the PR pipeline prevents non-compliant resources from ever being created. Evaluating the planned state with OPA/Conftest provides immediate feedback to developers before code merges, eliminating security risks and remediation toil.
</details>

---

### Question 2
What is the primary architectural difference between how **Crossplane** manages cloud infrastructure and how **Terraform** manages it?

A) Terraform executes transiently via CLI or CI pipelines using static state files; Crossplane is a continuous Kubernetes control plane that stores state in etcd and runs active reconciliation loops to automatically heal out-of-band drift in real time.  
B) Terraform does not support cloud databases.  
C) Crossplane cannot manage AWS resources.  
D) Terraform runs on Linux; Crossplane only runs on Windows.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Terraform is execution-based: it runs when triggered by a pipeline and ceases running once complete. Crossplane extends Kubernetes: it continuously reconciles declared desired state against the cloud provider 24/7, actively correcting configuration drift without human intervention.
</details>

---

### Question 3
In a PR-driven IaC workflow using **Atlantis**, what security vulnerability arises if Atlantis is configured to run `terraform plan` automatically on unreviewed pull requests from public or untrusted forks?

A) Atlantis will automatically merge the code.  
B) The runner will consume too much disk space.  
C) Malicious actors can submit a PR modifying HCL or using external data sources/scripts to exfiltrate the CI runner's AWS IAM credentials during the automated plan execution.  
D) The GitHub repository will be deleted.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Terraform configurations can execute arbitrary local shell commands via `local-exec` provisioners or external data sources during the plan phase. Running automated plans on untrusted code allows attackers to execute shell payloads that steal runner IAM credentials. Atlantis should only run on internal branches or after maintainer approval.
</details>

---

### Question 4
In an enterprise multi-account AWS Organization, what is the role of a **Transit Gateway (TGW)** in the Core Network Account?

A) It acts as a centralized cloud router connecting hundreds of VPCs across multiple workload accounts, on-premises data centers, and centralized egress inspection proxies in a hub-and-spoke topology.  
B) It manages employee email accounts.  
C) It hosts public marketing websites.  
D) It replaces all Linux operating systems.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In complex multi-account environments, full VPC peering meshes ($N(N-1)/2$) become unmanageable. An AWS Transit Gateway provides a centralized hub-and-spoke router, enabling controlled routing, centralized firewall inspection, and simplified connectivity across hundreds of VPCs.
</details>

---

### Question 5
What is an **Account Vending Machine (AVM)** in enterprise platform engineering?

A) A physical hardware dispenser for laptops.  
B) An automated self-service capability that provisions fully hardened, pre-configured cloud accounts (with standard VPCs, IAM baselines, KMS keys, GuardDuty, and billing alerts) via infrastructure-as-code in response to a developer request.  
C) An ATM for cryptocurrency.  
D) A tool that creates fake user accounts on social media.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** An AVM automates account provisioning. Rather than filing tickets and waiting weeks for cloud accounts, developers request an account via the platform portal, and the AVM provisions an AWS/GCP account with security guardrails and network attachments in minutes.
</details>

---

### Question 6
In Open Policy Agent (OPA) Rego language, what does the following rule do?
```rego
package terraform.limits
deny[msg] {
    instance := input.resource_changes[_]
    instance.type == "aws_instance"
    not startswith(instance.change.after.instance_type, "t3.")
    msg := "Only t3-series instances are permitted in development environments."
}
```

A) It reboots all t3 instances.  
B) It monitors CPU temperature on t3 instances.  
C) It forces all EC2 instances to upgrade to t3.metal.  
D) It parses a Terraform plan JSON and rejects any configuration attempting to provision an EC2 instance whose instance type does not begin with `t3.`.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** The Rego rule iterates over all `aws_instance` resources in the plan. If any instance specifies a type not starting with `t3.`, it populates the `deny` set, causing the policy check to fail with the violation message.
</details>

---

### Question 7
What is the purpose of **Service Control Policies (SCPs)** in AWS Organizations?

A) They monitor server fan speeds.  
B) They replace Terraform code entirely.  
C) They act as organizational guardrails that set the maximum allowable permissions for member accounts, establishing boundaries that even root or administrative users within member accounts cannot bypass.  
D) They manage employee vacation requests.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** SCPs define the absolute perimeter of allowed actions across an AWS Organization. Even an account administrator with `AdministratorAccess` cannot execute actions blocked by an SCP (e.g., leaving the organization, disabling CloudTrail, or creating resources in unauthorized AWS regions).
</details>

---

### Question 8
Why is separating the `plan` phase from the `apply` phase essential in enterprise IaC continuous delivery pipelines?

A) Because `apply` requires a reboot of developer laptops.  
B) Because Terraform cannot run both in the same month.  
C) It decreases network bandwidth by 90%.  
D) To decouple review and verification from execution: the generated plan file (`tfplan.binary`) acts as a locked, immutable artifact that can be inspected, policy-validated (OPA), and approved before executing the exact same operations during apply.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Generating a plan file creates an immutable representation of the intended changes. Applying that specific plan file (`terraform apply tfplan.binary`) guarantees that exactly what was reviewed and policy-checked is executed, preventing out-of-band changes from slipping in between review and apply.
</details>

---

### Question 9
What is a **Composite Resource Definition (XRD)** in Crossplane?

A) An encryption key generated by Vault.  
B) A legacy virtual machine format.  
C) A Docker compose configuration file.  
D) A custom API specification defined by the platform team that declares a simplified, high-level resource (e.g., `CompositePostgres`) and maps it to underlying compositions of real cloud infrastructure primitives.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** XRDs define the platform API interface. Product teams interact with simple XRD schemas (e.g., choosing `db_size: small`), while Crossplane's Composition behind the scenes orchestrates the VPC subnet groups, security groups, KMS keys, and RDS instances.
</details>

---

### Question 10
In a high-maturity platform engineering team, how is infrastructure drift typically handled?

A) By disabling all cloud monitoring tools.  
B) By running automated daily drift detection pipelines (e.g., `terraform plan -detailed-exitcode`), generating telemetry metrics, posting alerts to owning teams, and employing continuous reconciliation engines (like Crossplane) to self-heal state.  
C) By manually logging into the cloud console to fix things.  
D) By ignoring it until production crashes.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Mature teams treat drift as an operational defect. They detect drift automatically via scheduled CI runs or continuous control planes, maintaining tight alignment between declared code and live infrastructure.
</details>
