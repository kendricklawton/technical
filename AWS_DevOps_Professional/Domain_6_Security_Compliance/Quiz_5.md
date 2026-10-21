# Domain 6 (Security and Compliance) — Quiz 5: AWS Config Governance & Conformance Packs

**Topic:** Configuration Items, Managed vs Custom Rules, Conformance Packs & Remediation  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: AWS Config Evaluation Trigger Types (Change-Triggered vs Periodic)

An organization creates a custom AWS Config rule using AWS Lambda to enforce specific tagging standards and security group rules across all EC2 instances.

**What are the two distinct trigger types supported by AWS Config rules, and when should each be used?**
- A. Push and Pull.
- B. **Configuration Changes** (evaluates immediately whenever a supported resource is created, modified, or deleted) and **Periodic** (evaluates at scheduled frequency intervals such as 1 hour, 3 hours, 6 hours, 12 hours, or 24 hours).
- C. Manual CLI execution and CloudTrail API call only.
- D. HTTP POST and WebSocket.

---

### Question 2: Deploying Organization-Wide Compliance with Conformance Packs

A financial enterprise needs to deploy a baseline of 40 AWS Config rules and 15 automated SSM remediation actions across all 250 AWS accounts in their AWS Organization. The compliance baseline must be managed as version-controlled code and automatically applied to any new accounts joined to the organization.

**Which AWS Config feature satisfies this requirement?**
- A. Deploying individual CloudFormation stacks manually into each account.
- B. Deploying an **Organization Conformance Pack**, which packages a collection of AWS Config rules and remediation actions into a single YAML template and deploys it across the entire organization from the management or delegated administrator account.
- C. Writing a bash script that loops through AWS accounts using SSH.
- D. Using AWS Trusted Advisor.

---

### Question 3: Configuration Item (CI) Delivery and Storage

How does AWS Config track resource configuration history, and where are Configuration Items (CIs) stored for long-term audit purposes?
- A. CIs are stored in volatile RAM inside AWS data centers and erased after 24 hours.
- B. AWS Config delivers **Configuration History files** and **Configuration Snapshot files** periodically to an **Amazon S3 bucket**, and can optionally stream configuration changes in real time to an **Amazon SNS topic**.
- C. CIs are emailed as PDF attachments to the root user.
- D. CIs can only be viewed in the AWS Console for 7 days.

---

### Question 4: Multi-Account Multi-Region Data Aggregation in AWS Config

A compliance officer needs a single unified dashboard to view compliance status and resource inventory across 50 AWS accounts and 6 Regions without logging into individual accounts.

**What feature of AWS Config enables centralized multi-account, multi-region compliance visibility?**
- A. VPC Peering mesh.
- B. **AWS Config Aggregators**, which aggregate configuration and compliance data from multiple accounts and multiple Regions into a designated central monitoring account using AWS Organizations authorization.
- C. AWS CloudTrail Lake.
- D. Route 53 Resolver query logging.

---

### Question 5: Handling AWS Config Rule Evaluation Timeouts and Permissions

A custom AWS Config rule evaluates compliance using an AWS Lambda function. When resources change, the rule is triggered, but the rule status displays `EVALUATION_FAILURE`. CloudWatch Logs show that the Lambda function completed execution, but the compliance status never updated in AWS Config.

**What is the most likely cause of this evaluation failure?**
- A. The Lambda function lacked permission to call the `config:PutEvaluations` API, or failed to return `PutEvaluations` within the required execution window.
- B. The S3 bucket was full.
- C. AWS Config only works with Python 2.7.
- D. The EC2 instance was stopped.

---

### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: AWS Config Evaluation Trigger Types (Change-Triggered vs Periodic)

**Answer:** B  
**Explanation:** 
AWS Config rules support two trigger types:
1. **Configuration Changes**: Triggered when AWS Config records a configuration change event (e.g., an EC2 security group rule is added, an S3 bucket ACL is modified). This provides near real-time compliance evaluation.
2. **Periodic**: Triggered at specified time frequencies (e.g., every 24 hours). This is used for rules that evaluate time-dependent criteria (e.g., checking if IAM access keys are older than 90 days) or resources that do not generate configuration items.

---

### Question 2: Deploying Organization-Wide Compliance with Conformance Packs

**Answer:** B  
**Explanation:** 
An **AWS Config Conformance Pack** is a collection of AWS Config rules and remediation actions packaged in a single YAML template. When deployed as an **Organization Conformance Pack** from the management account or delegated administrator, it automatically deploys and enforces the compliance baseline across all member accounts in the organization, including any newly created or joined accounts.

---

### Question 3: Configuration Item (CI) Delivery and Storage

**Answer:** B  
**Explanation:** 
AWS Config records changes to resource configurations as **Configuration Items (CIs)**. It periodically aggregates these CIs into Configuration History files and writes them to an Amazon S3 bucket for long-term immutable storage. It also allows exporting periodic Configuration Snapshots to S3 and streaming change notifications in real time to an Amazon SNS topic or EventBridge.

---

### Question 4: Multi-Account Multi-Region Data Aggregation in AWS Config

**Answer:** B  
**Explanation:** 
An **AWS Config Aggregator** collects AWS Config configuration and compliance data from multiple AWS accounts and multiple Regions into a single central account. When integrated with AWS Organizations, the aggregator automatically discovers all member accounts in the organization, providing a single consolidated view of compliance and resource inventory.

---

### Question 5: Handling AWS Config Rule Evaluation Timeouts and Permissions

**Answer:** A  
**Explanation:** 
When AWS Config triggers a custom rule's Lambda function, the function evaluates the configuration item and must explicitly deliver the compliance result back to AWS Config by calling the **`config:PutEvaluations`** API. If the Lambda execution role lacks the `config:PutEvaluations` permission, or if the Lambda function fails before calling this API, AWS Config marks the rule evaluation as failed.

---


### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

