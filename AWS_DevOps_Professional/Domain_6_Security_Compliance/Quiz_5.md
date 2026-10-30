# Domain 6 (Security and Compliance) — Quiz 5: AWS Config Governance & Conformance Packs

**Topic:** Configuration Items, Managed vs Custom Rules, Conformance Packs, Aggregators & Auto-Remediation  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Organization Conformance Packs for Immutable Governance Baselines

An enterprise Chief Information Security Officer (CISO) requires a standardized baseline of 40 security compliance rules (e.g., encrypted EBS, S3 public access blocked, IAM root MFA enforced) and their associated automated SSM remediation runbooks deployed across 120 member accounts. Member account administrators must NOT be able to delete, modify, or bypass these rules.

**Which AWS architecture fulfills this organization-wide compliance enforcement?**
- A. Author an **Organization Conformance Pack** YAML template declaring the Config rules and SSM remediation documents; deploy the conformance pack from the **Organizations Management Account** (or delegated administrator) across all member accounts; AWS Config automatically deploys the rules and prevents member accounts from modifying or deleting them.
- B. Deploy CloudFormation StackSets to deploy independent AWS Config rules in each member account with stack termination protection enabled.
- C. Write an Amazon EventBridge rule that detects when member accounts delete Config rules and triggers an automated Lambda function to recreate them.
- D. Attach a Service Control Policy (SCP) denying `config:*` to all member accounts and deploy rules via the root user credentials.

---

### Question 2: Custom Policy-as-Code Rules via AWS Config Guard vs Lambda Custom Rules

A cloud compliance team needs to author a custom AWS Config rule to enforce that all Amazon DynamoDB tables have Point-in-Time Recovery (PITR) enabled and use Customer Managed Keys (CMKs) for encryption. The team wants to author declarative policy-as-code rules without deploying, patching, and maintaining custom AWS Lambda execution runtimes.

**Which AWS Config rule authoring mechanism satisfies this requirement?**
- A. Deploy an **AWS Config Custom Policy Rule** authored using the **CloudFormation Guard (`cfn-guard`)** domain-specific language: Store the declarative rule directly in AWS Config, which evaluates the rule natively within the Config managed runtime without provisioning Lambda functions.
- B. Deploy an AWS Config Custom Rule backed by an AWS Lambda function authored in Node.js that invokes the `PutEvaluations` API.
- C. Configure an AWS Systems Manager State Manager association that polls DynamoDB API endpoints every 15 minutes.
- D. Create an Amazon EventBridge rule with an input transformer that executes bash commands via SSM Run Command.

---

### Question 3: Centralized Enterprise Compliance Visibility via Config Aggregators

An enterprise operates 200 AWS member accounts across 6 AWS Regions. The central compliance team needs a single, unified dashboard to query compliance status, search inventory, and identify non-compliant resources across all accounts and regions without granting the compliance team direct IAM access into spoke accounts.

**Which AWS Config feature provides this unified multi-account and multi-region view?**
- A. Deploy an **AWS Config Multi-Account Multi-Region Aggregator** in a dedicated Audit account: Authorize the aggregator to collect compliance and inventory data from all accounts in the AWS Organization across all enabled regions, enabling organization-wide advanced SQL queries via AWS Config Advanced Query.
- B. Export AWS Config snapshot files from all accounts into a central Amazon S3 bucket and query them daily using Amazon Athena.
- C. Configure Amazon QuickSight to connect directly to the AWS Config API in each individual member account.
- D. Create an Amazon EventBridge cross-account bus in each member account forwarding compliance events to the audit account.

---

### Question 4: Optimizing AWS Config Costs via Continuous vs Periodic Recording Modes

A fintech startup experiences an unexpected surge in AWS Config costs, reaching \$15,000 per month. An audit reveals that ephemeral Amazon Elastic Network Interfaces (ENIs) created and deleted dynamically by AWS Lambda and Amazon ECS tasks generate millions of Configuration Items (CIs) daily. However, the startup only requires continuous recording on stateful resources (S3, RDS, EBS) for compliance, while non-critical resources can be evaluated periodically.

**Which AWS Config configuration adjustment optimizes costs while preserving regulatory compliance?**
- A. Configure **AWS Config Recording Modes**: Keep **Continuous Recording** enabled for critical stateful resource types (`AWS::S3::Bucket`, `AWS::RDS::DBInstance`, `AWS::EBS::Volume`), switch transient resource types to **Periodic Recording** (recording changes only once every 24 hours), and explicitly exclude high-churn transient ENIs from recording.
- B. Disable AWS Config recording entirely and rely on CloudTrail Management Events for inventory tracking.
- C. Delete the AWS Config delivery channel and store configuration items exclusively in local memory.
- D. Set the AWS Config retention period to 1 day to reduce data storage fees.

---

### Question 5: Trigger Types for Config Rules: Change-Triggered vs Periodic Evaluations

A DevOps engineer is designing an AWS Config rule to verify that Amazon EC2 instances do not have unencrypted root EBS volumes attached. The security team insists that non-compliant instances must be flagged immediately upon creation, rather than waiting for an hourly or daily batch scan.

**Which Trigger Type must be configured on the AWS Config rule?**
- A. Configure the rule with a **Configuration Change** trigger type: AWS Config evaluates the rule immediately whenever a Configuration Item (CI) is generated or updated for `AWS::EC2::Instance` or `AWS::EC2::Volume`, flagging violations within seconds.
- B. Configure the rule with a **Periodic** trigger type set to a maximum frequency of 1 hour.
- C. Configure an AWS Lambda function triggered by Amazon CloudWatch Metrics.
- D. Configure an Auto Scaling lifecycle hook that invokes the AWS Config API during instance launch.

---

### Question 6: Automated Closed-Loop Remediation Execution Order and Retry Logic

An automated remediation action is configured on an AWS Config rule that checks whether Amazon EBS volumes are encrypted (`encrypted-volumes`). When an unencrypted volume is detected, Config triggers the SSM Automation document `AWS-EncryptEBSVolume`. However, the remediation fails on the first attempt because an attached EC2 instance was in the `stopping` state.

**How does AWS Config handle failed automated remediation executions?**
- A. AWS Config supports **Automatic Retries**: You can configure the maximum number of retry attempts (up to 5 retries) and a retry interval; Config automatically re-invokes the SSM Automation document if previous execution attempts fail before marking the remediation as failed.
- B. AWS Config deletes the non-compliant resource immediately upon remediation failure.
- C. AWS Config permanently disables the rule and sends an SNS email alert to the account root user.
- D. AWS Config restarts the underlying EC2 instance and resets the configuration recorder.

---

### Question 7: Advanced Compliance Analytics via Config SQL Query Syntax

A security auditor needs to find all unattached Amazon EBS volumes (`status = "available"`) larger than 500 GB across 50 AWS accounts that have not been modified in the last 30 days, to identify abandoned data disks.

**Which AWS Config feature and syntax facilitates this multi-account discovery?**
- A. Use **AWS Config Advanced Query** pointing to an enterprise Aggregator, executing a standard SQL query: `SELECT resourceId, accountId, awsRegion, configuration.size WHERE resourceType = 'AWS::EBS::Volume' AND configuration.state = 'available' AND configuration.size > 500`.
- B. Run an Amazon Athena query against raw CloudTrail `.json.gz` log archives stored in Amazon S3.
- C. Write a Python script using `boto3` that assumes cross-account IAM roles and calls `ec2:DescribeVolumes` in a nested loop.
- D. Query Amazon CloudWatch Logs Insights across all VPC Flow Log groups.

---

### Question 8: Handling Exceptions and Suppressing False Positives via Conformance Pack Exclusions

An enterprise deploys an Organization Conformance Pack enforcing EBS volume encryption across all accounts. A specialized legacy machine learning workload in the Research OU requires unencrypted local scratch disks to achieve raw physical I/O performance. The security team agrees to grant a compliance exemption for these specific resources without altering the baseline conformance pack for other teams.

**How should this compliance exemption be managed within AWS Config?**
- A. Configure **Rule Exceptions / Exclusions** in AWS Config: Authorize an exception for the specific resource ARNs or account IDs within the Conformance Pack configuration, ensuring that the exempted scratch volumes do not count against organizational compliance scores.
- B. Delete the AWS Config recorder in the Research account.
- C. Attach an IAM policy denying `config:PutEvaluations` to the Config service-linked role in that account.
- D. Modify the EC2 instance launch template to spoof the EBS encryption flag via metadata injection.

---

### Question 9: AWS Config Delivery Channel Resilience and S3 Bucket Policies

An administrator sets up AWS Config in a new member account. Config successfully records Configuration Items, but configuration snapshots and change history files fail to deliver to the centralized compliance S3 bucket in the Logging Account, reporting delivery failures.

**What is the required S3 Bucket Policy configuration to allow AWS Config to deliver configuration history files?**
- A. The central S3 bucket policy must grant `s3:PutObject` and `s3:GetBucketAcl` to the AWS Config service principal (`config.amazonaws.com`), specifying the bucket prefix path `AWSLogs/<account-id>/Config/*` with condition `StringEquals: { "aws:SourceAccount": "<member-account-id>" }`.
- B. The S3 bucket policy must grant `s3:*` to `Principal: "*"` without any conditions.
- C. The member account must assume an IAM role in the Logging Account before starting the configuration recorder.
- D. The S3 bucket must have public read and write access enabled.

---

## Answer Key & Explanations

### Question 1: Organization Conformance Packs for Immutable Governance Baselines
**Answer:** A  
**Explanation:** 
**AWS Config Organization Conformance Packs** provide immutable compliance governance across an AWS Organization:
- Authored as a single YAML template that bundles multiple AWS Config rules and SSM Automation remediation documents.
- Deployed from the **Organizations Management Account** (or a delegated administrator account).
- Once deployed, AWS Config automatically provisions the conformance pack across all existing and newly joined member accounts.
- **Tamper-Proof:** Member account administrators (even those with root or `AdministratorAccess`) **cannot modify or delete** organization conformance packs or their member rules. They can only be updated or deleted from the central management account.

---

### Question 2: Custom Policy-as-Code Rules via AWS Config Guard vs Lambda Custom Rules
**Answer:** A  
**Explanation:** 
AWS Config supports two methods for authoring custom compliance rules:
1. **Lambda Custom Rules:** Requires authoring custom code in Python/Node.js, packaging dependencies, provisioning Lambda functions, managing IAM execution roles, and handling Lambda runtime lifecycle updates.
2. **AWS Config Custom Policy Rules (powered by CloudFormation Guard):** Allows writing declarative, human-readable policy-as-code rules using the **`cfn-guard`** DSL directly inside AWS Config (e.g., `let status = configuration.pointInTimeRecoveryDescription.pointInTimeRecoveryStatus; status == 'ENABLED'`). Config evaluates the rule natively within its managed runtime with **zero Lambda functions to deploy or maintain**, drastically reducing operational overhead.

---

### Question 3: Centralized Enterprise Compliance Visibility via Config Aggregators
**Answer:** A  
**Explanation:** 
An **AWS Config Aggregator** is an AWS Config resource that collects configuration and compliance data from multiple accounts and regions:
- In an organization aggregator, you authorize Config to gather data from the entire AWS Organization across all regions.
- It provides a centralized, single-pane-of-glass dashboard displaying overall compliance percentages and non-compliant resource counts.
- **Advanced Query:** Integrates with **AWS Config Advanced Query**, allowing security analysts to run standard SQL `SELECT` queries across the aggregate inventory of all 200 accounts in seconds without configuring cross-account IAM roles or writing custom polling scripts.

---

### Question 4: Optimizing AWS Config Costs via Continuous vs Periodic Recording Modes
**Answer:** A  
**Explanation:** 
AWS Config pricing is based on the number of Configuration Items (CIs) recorded ($0.003 per CI):
- Ephemeral resources (such as dynamic ENIs created by Lambda or ECS) churn constantly, generating millions of CIs and driving high costs.
- **AWS Config Recording Modes:**
  - **Continuous Recording:** Records a CI immediately whenever a change occurs. Essential for stateful, compliance-sensitive resources (S3 buckets, IAM roles, RDS databases, security groups).
  - **Periodic Recording:** Records changes only once every 24 hours if a change occurred, capturing end-of-day configuration while filtering out intra-day transient churn.
- Switching low-priority resources to Periodic mode and excluding ephemeral ENIs preserves compliance visibility while reducing Config billing by up to 80%.

---

### Question 5: Trigger Types for Config Rules: Change-Triggered vs Periodic Evaluations
**Answer:** A  
**Explanation:** 
AWS Config rules support two trigger types:
- **Configuration Changes:** Triggered immediately when AWS Config detects a change in the configuration of a resource type (e.g., when an EC2 instance launches or a security group changes). This allows real-time, event-driven compliance detection within seconds.
- **Periodic:** Triggered at a fixed scheduled frequency (e.g., 1 hour, 3 hours, 6 hours, 12 hours, 24 hours). Used for rules that evaluate time-based criteria (e.g., checking if IAM access keys have not been rotated in 90 days).
To catch unencrypted EBS volumes immediately upon creation, the rule must be configured with a **Configuration Change** trigger.

---

### Question 6: Automated Closed-Loop Remediation Execution Order and Retry Logic
**Answer:** A  
**Explanation:** 
When configuring automated remediation in AWS Config:
- You specify the target SSM Automation document and map parameters (such as `RESOURCE_ID`).
- Because cloud environments are dynamic, initial remediation attempts can fail due to resource states (e.g., an EC2 instance is currently rebooting or stopping).
- Config supports **Automatic Retries**: You configure the maximum number of retry attempts (e.g., 5 retries) and a retry interval in seconds.
- Config automatically retries the SSM document execution upon transient failures before flagging the remediation as failed, ensuring high remediation success rates.

---

### Question 7: Advanced Compliance Analytics via Config SQL Query Syntax
**Answer:** A  
**Explanation:** 
**AWS Config Advanced Query** provides a managed SQL query engine:
- It queries the current configuration state of AWS resources across accounts and regions using standard SQL `SELECT` syntax.
- Schema properties match the Configuration Item JSON structure (e.g., `configuration.size`, `configuration.state`, `tags.Environment`).
- Pointing Advanced Query at an enterprise Config Aggregator allows finding unattached volumes larger than 500 GB across 50 accounts in a single query execution without writing custom API scripts.

---

### Question 8: Handling Exceptions and Suppressing False Positives via Conformance Pack Exclusions
**Answer:** A  
**Explanation:** 
In enterprise governance, legitimate business exceptions arise:
- AWS Config supports configuring **Exclusions and Exceptions** within Conformance Packs.
- You can specify individual resource identifiers (ARNs) or specific member accounts that are exempted from specific rules.
- When an exception is active, AWS Config suppresses compliance evaluation for those resources, preventing them from skewing enterprise compliance scores or triggering automated remediation actions.

---

### Question 9: AWS Config Delivery Channel Resilience and S3 Bucket Policies
**Answer:** A  
**Explanation:** 
For AWS Config to deliver configuration snapshots and history files to an Amazon S3 bucket:
- The destination S3 bucket policy must explicitly grant permissions to the AWS Config service principal (`config.amazonaws.com`).
- Required permissions: `s3:PutObject` (for delivery) and `s3:GetBucketAcl` (to verify bucket permissions).
- Best practice requires conditioning the grant on `aws:SourceAccount` matching the member account ID to prevent unauthorized accounts from writing to the centralized compliance bucket.
