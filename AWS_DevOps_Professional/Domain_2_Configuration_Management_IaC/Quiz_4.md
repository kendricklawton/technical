# Domain 2 (Configuration Management & IaC) — Quiz 4: Systems Manager Fleet Management Deep Dive

**Topic:** Systems Manager State Manager, Patch Manager, Session Manager, Distributor & Inventory  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Patch Manager Baseline Auto-Approval Rules & Severity Filtering

An enterprise platform architecture mandates automated operating system patching across 1,200 Amazon EC2 Linux instances. To guard against faulty upstream package vendor regressions, patches classified as `Critical` must be delayed by exactly 7 days before automated installation, while `Important` and `Medium` security updates must be delayed by 14 days. Non-security bugfix updates must be ignored.

**How should this governance policy be declared in AWS Systems Manager Patch Manager?**
- A. Create a custom **Patch Baseline** with two distinct **Approval Rules**: Rule 1 matches product `*`, classification `Security`, severity `Critical` with `AutoApprovalDelay: 7 days`; Rule 2 matches classification `Security`, severity `Important` and `Medium` with `AutoApprovalDelay: 14 days`, setting the compliance level appropriately.
- B. Configure an AWS Systems Manager State Manager association that executes two sequential cron jobs inside the Linux operating system using custom yum shell scripts.
- C. Deploy an AWS Lambda function triggered on an Amazon EventBridge schedule that queries AWS Inspector CVE alerts and updates repository package manifests.
- D. Create an Amazon EventBridge rule matching AWS Health security advisory events, and target an SSM Automation runbook that schedules EC2 host reboots.

---

### Question 2: Decoupled Multi-Environment Patch Scheduling via Patch Groups

An organization operates two distinct application tiers: `Staging` and `Production`. All instances must adhere to the same custom patch baseline, but Staging instances must undergo automated patching every Tuesday at 02:00 UTC, while Production instances must undergo patching every Thursday at 02:00 UTC.

**How should instances and maintenance windows be structured in Systems Manager to enforce these decoupled schedules?**
- A. Tag target EC2 instances with the standard tag key **`Patch Group`** (e.g. `Patch Group = Staging` and `Patch Group = Production`), register both patch groups with the custom patch baseline, and create two distinct **SSM Maintenance Windows** targeting each patch group tag independently.
- B. Provision two separate AWS Organizations member accounts, maintaining independent default patch baselines in each regional account endpoint.
- C. Hardcode instance IDs into two separate AWS Systems Manager Automation runbooks, triggering them via Amazon EventBridge Scheduler cron expressions.
- D. Configure two distinct EC2 Auto Scaling Groups using separate launch templates, and terminate instances in batches during scheduled maintenance windows.

---

### Question 3: State Manager Continuous Drift Remediation and Service Enforcement

An enterprise compliance standard requires that the Amazon CloudWatch Agent daemon (`amazon-cloudwatch-agent`) must be running continuously on all managed Amazon EC2 instances. If an instance administrator accidentally or maliciously stops the service, the configuration must be automatically detected, restarted, and reported as compliant within 30 minutes without deploying new infrastructure code.

**Which Systems Manager mechanism enforces this continuous state reconciliation?**
- A. Create a **State Manager Association** targeting instances by tag, configured with document `AmazonCloudWatch-ManageAgent` (action: `configure`, mode: `ec2`, optional_restart: `yes`), scheduled on a 30-minute cron interval to automatically restart the daemon if stopped.
- B. Configure an Amazon CloudWatch Alarm that monitors EC2 CPU utilization drops, triggering an AWS Systems Manager Automation runbook that executes an instance reboot.
- C. Write an Amazon EventBridge rule that captures AWS CloudTrail `StopInstances` API calls and invokes an AWS Lambda function to verify host daemon processes.
- D. Enable AWS Config continuous recording on the EC2 instances, configuring the managed rule `ec2-managedinstance-patch-compliance-status-check` with automated reboot remediation.

---

### Question 4: Systems Manager Maintenance Window Cutoff Execution Semantics

An enterprise maintenance window is defined in AWS Systems Manager with a total **Duration of 4 hours** and a **Cutoff of 1 hour**. A long-running database index maintenance task that typically takes 90 minutes is registered as a Run Command task within the window.

**What is the precise execution behavior when the maintenance window reaches the 3-hour mark (the cutoff point)?**
- A. Systems Manager **stops scheduling and initiating any new tasks**; any task that is already executing is allowed to continue running until its specified task timeout expires or the 4-hour window concludes.
- B. Systems Manager immediately sends a `SIGTERM` signal to all currently running tasks, terminating in-flight scripts to prevent exceeding the maintenance window.
- C. The maintenance window duration is automatically extended by an additional hour to allow in-flight tasks to complete gracefully.
- D. All registered EC2 instances are automatically placed into standby mode, preventing network traffic from reaching the instances until the window resets.

---

### Question 5: Multi-Region Software License & Package Auditing via SSM Inventory

A corporate software audit requires generating a comprehensive, historical inventory of all versions of OpenSSL, Python, and Java packages installed across 1,500 Amazon EC2 instances spanning 4 AWS regions, with the ability to query installed versions using SQL.

**Which native Systems Manager capability collects and centralizes this inventory data with the LEAST operational management?**
- A. Configure **Systems Manager Inventory** across all managed nodes to collect `AWS:Application` package metadata, and configure a **Resource Data Sync** to centralize all regional inventory data into an Amazon S3 bucket for querying via **Amazon Athena**.
- B. Execute an AWS Systems Manager Run Command script (`rpm -qa` / `dpkg -l`) every morning across all instances, appending output files to an encrypted EBS volume.
- C. Configure Amazon CloudWatch Logs subscription filters on `/var/log/messages` across all instances, streaming log events to an Amazon OpenSearch Service cluster.
- D. Enable Amazon Inspector continuous scanning on EC2 instances, and write a Python script that polls the Inspector API to reconstruct installed package lists.

---

### Question 6: Least-Privilege Interactive Terminal Access with Session Manager Tag Conditions

A security policy permits software developers to start interactive bash sessions via AWS Systems Manager Session Manager **ONLY** on Amazon EC2 instances tagged `Environment=Development`. Developers must be strictly blocked from starting sessions on instances tagged `Environment=Production`.

**Which IAM policy condition enforces this tag-based access boundary?**
- A. Attach an IAM policy granting `ssm:StartSession` on resource `arn:aws:ec2:*:*:instance/*` with condition:  
  `"StringEquals": { "ssm:resourceTag/Environment": "Development" }`.
- B. Configure the EC2 instance Security Group to block inbound TCP traffic on port 22 for all IP addresses in the corporate office.
- C. Remove the AWS Systems Manager Agent (SSM Agent) from all production EC2 instances to prevent session registration.
- D. Attach an IAM session policy that allows `ssm:DescribeInstanceInformation` with condition `"StringEquals": { "aws:PrincipalTag/Role": "Developer" }`.

---

### Question 7: Secure TCP Tunneling & Database Access via Session Manager Port Forwarding

A database administrator needs to connect a local GUI client (e.g. pgAdmin) running on their workstation to an Amazon Aurora PostgreSQL database deployed in an isolated private VPC subnet. Security mandates prohibit provisioning bastion hosts (jump boxes), opening inbound port 5432 to the corporate network, or provisioning public IPv4 addresses.

**How should this administrative access be architected using Systems Manager?**
- A. Use **Session Manager Port Forwarding**: Execute the AWS CLI command `aws ssm start-session --target <private-ec2-instance-id> --document-name AWS-StartPortForwardingSessionToRemoteHost --parameters '{"host":["aurora-cluster.endpoint"],"portNumber":["5432"],"localPortNumber":["5432"]}'`.
- B. Deploy an AWS Client VPN endpoint in the public subnet, and configure route tables to allow direct routing from the client laptop to the Aurora subnet.
- C. Configure an Application Load Balancer in a public subnet with a TLS listener forwarding TCP traffic to the Aurora database endpoints.
- D. Provision an AWS Direct Connect dedicated connection with a private virtual interface (VIF) directly into the Aurora database subnet.

---

### Question 8: Automated Security Agent Distribution with Systems Manager Distributor

An enterprise security operations team must distribute and maintain a third-party endpoint detection and response (EDR) agent across thousands of Amazon EC2 instances running Amazon Linux 2, Amazon Linux 2023, and Windows Server. The agent software is updated quarterly, and new instance launches must install the agent automatically at boot.

**Which AWS Systems Manager capability manages this software distribution lifecycle?**
- A. Publish the software packages into **AWS Systems Manager Distributor**, and create a **State Manager Association** targeting all managed instances that schedules installation and automatic updates using the `AWS-ConfigureAWSPackage` document.
- B. Bake the security agent directly into an Amazon Machine Image (AMI) monthly, and trigger rolling Auto Scaling updates to replace all instances.
- C. Write an Amazon EventBridge rule that detects EC2 instance launch events and executes an `AWS-RunShellScript` Run Command task on the new instance ID.
- D. Store the agent binaries in a public Amazon S3 bucket, and configure an EC2 Launch Template `UserData` script to download and execute the installer on every reboot.

---

### Question 9: Centralized Multi-Account Compliance Telemetry with Resource Data Sync

A central security and governance team oversees 150 AWS member accounts. The team needs unified, real-time dashboards tracking OS patch compliance, installed inventory packages, and State Manager association statuses across all accounts and regions in an AWS Organization without logging into individual account consoles.

**Which architecture centralizes this compliance telemetry with the LEAST operational overhead?**
- A. Configure **Systems Manager Resource Data Sync** in each member account (or deploy it via CloudFormation StackSets) to continuously stream inventory, patch compliance, and association data to an encrypted central Amazon S3 bucket in the management account, querying the data lake using **Amazon Athena**.
- B. Deploy an AWS Lambda function in each member account scheduled every 30 minutes that queries `DescribeInstancePatchStates` and pushes records to Amazon DynamoDB.
- C. Configure an Amazon Kinesis Data Stream in every account, routing patch compliance logs to an Amazon OpenSearch Service cluster in the central security account.
- D. Create cross-account IAM roles in all 150 accounts, and build a custom web dashboard that assumes each role sequentially to query SSM compliance APIs.

---

## Answer Key & Detailed Explanations

### Question 1: Patch Manager Baseline Auto-Approval Rules & Severity Filtering
**Answer:** A  
**Explanation:**  
In AWS Systems Manager Patch Manager, a custom **Patch Baseline** defines granular **Approval Rules**. An approval rule can target specific operating system products, classifications (such as `Security`), and severity levels (such as `Critical`, `Important`, `Medium`). Specifying an `AutoApprovalDelay` (e.g. 7 days for `Critical` and 14 days for `Important`) ensures patches are thoroughly vetted in the open-source community before automated installation occurs.

---

### Question 2: Decoupled Multi-Environment Patch Scheduling via Patch Groups
**Answer:** A  
**Explanation:**  
**Patch Groups** allow associating specific managed instances with distinct patch baselines and maintenance windows. By tagging instances with the reserved key `Patch Group` (e.g. `Patch Group = Staging` and `Patch Group = Production`) and registering the patch group names in the patch baseline, administrators can create separate **SSM Maintenance Windows** with independent schedules targeting each patch group tag.

---

### Question 3: State Manager Continuous Drift Remediation and Service Enforcement
**Answer:** A  
**Explanation:**  
**AWS Systems Manager State Manager** maintains instances in a defined state. An association can target instances using tags or resource groups, running document `AmazonCloudWatch-ManageAgent` on a defined recurring schedule (e.g. every 30 minutes). When executed, the SSM agent verifies service status and restarts the daemon if it has been stopped, restoring compliance automatically.

---

### Question 4: Systems Manager Maintenance Window Cutoff Execution Semantics
**Answer:** A  
**Explanation:**  
In an SSM Maintenance Window, the **Cutoff** parameter specifies the number of hours before the end of the maintenance window when Systems Manager stops scheduling and initiating any *new* tasks. Any task that began executing prior to the cutoff point is permitted to continue executing until its defined execution timeout expires or until the overall maintenance window concludes.

---

### Question 5: Multi-Region Software License & Package Auditing via SSM Inventory
**Answer:** A  
**Explanation:**  
**Systems Manager Inventory** collects detailed metadata from managed instances (including installed applications via `AWS:Application`, network configurations, and OS details). Using **Resource Data Sync**, inventory data collected across multiple accounts and regions is synchronized into a centralized Amazon S3 bucket. Once aggregated in S3, organizations query package versions and historical configuration changes using **Amazon Athena**.

---

### Question 6: Least-Privilege Interactive Terminal Access with Session Manager Tag Conditions
**Answer:** A  
**Explanation:**  
IAM policies governing `ssm:StartSession` can evaluate resource tags using the condition key `ssm:resourceTag/<TagKey>`. Granting `ssm:StartSession` with condition `StringEquals: {"ssm:resourceTag/Environment": "Development"}` allows opening interactive sessions exclusively on instances tagged `Environment=Development`, while access to production instances is denied by default.

---

### Question 7: Secure TCP Tunneling & Database Access via Session Manager Port Forwarding
**Answer:** A  
**Explanation:**  
**AWS Systems Manager Session Manager Port Forwarding** allows securely tunneling traffic between a local machine and remote ports on private instances or remote database hosts (such as Amazon RDS/Aurora) using the AWS CLI and Session Manager plugin. Document `AWS-StartPortForwardingSessionToRemoteHost` establishes an encrypted multiplexed WebSocket tunnel over HTTPS (port 443), eliminating the need for bastion hosts or open inbound security group rules.

---

### Question 8: Automated Security Agent Distribution with Systems Manager Distributor
**Answer:** A  
**Explanation:**  
**AWS Systems Manager Distributor** simplifies package management across fleets. Packages containing software binaries, installation scripts, and metadata for various operating systems are published to Distributor. Associating the package with managed nodes via a **State Manager Association** running document `AWS-ConfigureAWSPackage` ensures the software is automatically installed on new instances and updated when new package versions are published.

---

### Question 9: Centralized Multi-Account Compliance Telemetry with Resource Data Sync
**Answer:** A  
**Explanation:**  
**Resource Data Sync** centralizes Systems Manager Inventory, patch compliance, and State Manager association compliance telemetry across hundreds of AWS accounts and multiple regions into an Amazon S3 data lake. This provides a unified, serverless architecture where security and operations teams execute SQL queries in **Amazon Athena** or build Amazon QuickSight compliance visualizations with zero custom poller infrastructure.
