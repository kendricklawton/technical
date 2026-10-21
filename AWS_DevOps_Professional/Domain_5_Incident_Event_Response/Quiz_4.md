# Domain 5 (Incident and Event Response) — Quiz 4: Systems Manager Automation & Multi-Account Remediation

**Topic:** Automation Documents, Rate Control (Concurrency/Error Threshold), Multi-Account Execution & Script Steps  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Multi-Account and Multi-Region SSM Automation Execution

A centralized security engineering team needs to remediate a critical zero-day vulnerability by updating a software package across 5,000 EC2 instances spread across 20 AWS accounts and 4 Regions. The execution must be orchestrated from a central management account.

**Which AWS Systems Manager Automation feature natively enables centralized multi-account, multi-region execution?**
- A. SSH looping through all public IP addresses from an admin workstation.
- B. Using **SSM Automation Multi-Account and Multi-Region execution**, defining target AWS Accounts (or AWS Organizations OUs), target Regions, an administration role in the central account, and an execution role (`AWS-SystemsManager-AutomationExecutionRole`) in spoke accounts.
- C. CloudFormation StackSets with manual parameter overrides.
- D. VPC Peering mesh connecting all 20 accounts.

---

### Question 2: Rate Control Configuration: Concurrency and Error Thresholds

When executing a patch remediation runbook across 1,000 production EC2 instances behind load balancers, the DevOps engineer must ensure that:
1. No more than 10% of the instances are updated simultaneously, preserving application capacity.
2. If more than 5 instances fail the update, the entire automation execution must halt immediately to prevent widespread fleet failure.

**How should the engineer configure the Rate Control parameters in the Automation execution?**
- A. `MaxConcurrency: 10%` and `MaxErrors: 5`
- B. `MaxConcurrency: 100` and `MaxErrors: 100%`
- C. `BatchSize: 1` and `Timeout: 300`
- D. `Parallelism: 10` and `CircuitBreaker: True`

---

### Question 3: Running Python or PowerShell Code in SSM Automation Documents

A DevOps engineer needs to write a custom SSM Automation document that calls third-party REST APIs, parses the returned JSON payload, and passes an extracted authorization token to subsequent AWS API steps in the document.

**Which action step in an SSM Automation document allows embedding and executing custom Python or PowerShell scripts?**
- A. `aws:runCommand`
- B. `aws:executeScript`
- C. `aws:invokeLambdaFunction`
- D. `aws:branch`

---

### Question 4: Human-in-the-Loop Manual Approval Steps in Automation Runbooks

A disaster recovery failover runbook executes automated pre-checks and database snapshotting. Before promoting the secondary database and modifying DNS records, the Chief Architect must manually review the pre-check outputs and grant explicit approval.

**Which action step must be placed in the SSM Automation document to pause execution pending human approval?**
- A. `aws:sleep`
- B. `aws:approve`
- C. `aws:pauseExecution`
- D. `aws:waitForCallback`

---

### Question 5: SSM Automation Document Conditional Branching

An automated incident remediation runbook checks whether an affected EC2 instance is tagged `Environment=Production` or `Environment=Development`. If Development, the runbook terminates the instance immediately; if Production, it attaches an isolating security group and creates an AMI snapshot for forensic analysis.

**Which SSM Automation action type is used to execute different steps based on conditional evaluations?**
- A. `aws:executeAutomation`
- B. `aws:branch`
- C. `aws:switchCase`
- D. `aws:conditionalGoto`

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

### Question 1: Multi-Account and Multi-Region SSM Automation Execution

**Answer:** B  
**Explanation:** 
AWS Systems Manager Automation supports native **Multi-Account and Multi-Region** executions. From a central orchestration account, you can trigger an Automation document targeting multiple accounts (via account IDs or AWS Organizations OU paths) and multiple Regions simultaneously. The central account uses an Administration Role to assume an execution role (`AWS-SystemsManager-AutomationExecutionRole`) in each spoke target account, executing the tasks securely and aggregating status back to the central console.

---

### Question 2: Rate Control Configuration: Concurrency and Error Thresholds

**Answer:** A  
**Explanation:** 
SSM Automation (and SSM Run Command) provides **Rate Control** using two parameters:
- **MaxConcurrency** (or Targets concurrency): Limits the number or percentage of targets that can run the automation step simultaneously (e.g., `10%` ensures 90% of instances remain online).
- **MaxErrors**: The maximum number or percentage of errors allowed before the automation stops sending tasks to remaining targets (e.g., `MaxErrors: 5` halts execution as soon as 6 failures occur, limiting the blast radius).

---

### Question 3: Running Python or PowerShell Code in SSM Automation Documents

**Answer:** B  
**Explanation:** 
The `aws:executeScript` action allows embedding inline Python (using Python 3 runtime) or PowerShell Core scripts directly inside the SSM Automation YAML/JSON document without provisioning external Lambda functions. The script receives inputs from previous document steps and returns JSON outputs that downstream document steps can reference via `{{ scriptStep.outputName }}`.

---

### Question 4: Human-in-the-Loop Manual Approval Steps in Automation Runbooks

**Answer:** B  
**Explanation:** 
The `aws:approve` action pauses the execution of an SSM Automation document until one or more designated approvers (specified by IAM user/role ARNs) explicitly approve or reject the step via the AWS Console or `aws ssm send-automation-signal`. You can also configure an SNS topic to notify approvers when the step is reached, and specify a timeout period after which the execution fails if not approved.

---

### Question 5: SSM Automation Document Conditional Branching

**Answer:** B  
**Explanation:** 
The `aws:branch` action provides dynamic choice/branching logic within an SSM Automation document. It evaluates choices based on boolean expressions, string comparisons, or variable values (e.g., checking the output of a previous step or parameter value) and jumps execution to a designated target step (`NextStep`), enabling complex multi-path remediation workflows.

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

