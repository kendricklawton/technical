# Domain 2 (Configuration Management & IaC) — Quiz 4: Systems Manager Fleet Management

**Topic:** Systems Manager State Manager, Patch Manager, Maintenance Windows & Inventory  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Patch Manager Patch Baselines & Auto-Approval Delays

An enterprise requires that all Amazon EC2 Linux instances receive operating system security patches automatically. To prevent faulty upstream vendor packages from causing outages, patches classified as `Critical` must be delayed by 7 days before automated installation, while `Non-Critical` patches must be delayed by 14 days.

**How is this policy configured in AWS Systems Manager Patch Manager?**
- A. By scheduling two separate cron jobs in the Linux OS.
- B. By creating a custom **Patch Baseline** with two **Approval Rules**:
  - Rule 1: Classification `Security`, Severity `Critical`, **Auto-approval delay: 7 days**.
  - Rule 2: Classification `Security`, Severity `Important` and `Medium`, **Auto-approval delay: 14 days**.
- C. By writing an AWS Lambda function to approve patches manually.
- D. By configuring AWS Inspector to delay scans by 7 days.

---

### Question 2: Patch Groups and Maintenance Windows Association

A company operates two environments: `Staging` and `Production`. Staging instances must be patched on Tuesday evenings, and Production instances must be patched on Thursday evenings using the same custom patch baseline.

**How should instances be organized in Patch Manager to enforce these separate schedules?**
- A. Create two separate AWS accounts.
- B. Tag instances with the tag key **`Patch Group`** (e.g. `Patch Group = Staging` and `Patch Group = Production`). Register the patch groups with the custom patch baseline, and create two distinct **SSM Maintenance Windows** targeting each patch group with different schedule expressions.
- C. Hardcode instance IDs into two separate bash scripts.
- D. Use Auto Scaling Groups with different launch templates.

---

### Question 3: Systems Manager State Manager Association Compliance

A company uses AWS Systems Manager State Manager to ensure that the Amazon CloudWatch Agent is always running on all managed EC2 instances. If an instance administrator manually stops the CloudWatch Agent service, how does State Manager detect and remediate this?

**Which mechanism governs State Manager association evaluation?**
- A. State Manager polls the instance once every year.
- B. The **State Manager Association** runs on a user-defined schedule (e.g. every 30 minutes). When the association executes, the SSM Agent on the instance runs the associated document (`AmazonCloudWatch-ManageAgent`), checks service status, detects that the service is stopped, starts the service to restore compliance, and reports compliance status back to AWS Systems Manager.
- C. CloudWatch alarms terminate the non-compliant instance.
- D. State Manager requires a full system reboot to evaluate compliance.

---

### Question 4: Systems Manager Maintenance Window Cutoff Behavior

An SSM Maintenance Window is configured with a duration of 4 hours and a **Cutoff of 1 hour**. A long-running database indexing task is registered to run in this window.

**What happens to new tasks when the maintenance window reaches the 3-hour mark (the cutoff point)?**
- A. All running tasks are immediately terminated.
- B. Systems Manager **stops scheduling and initiating any new tasks**. Any task already running is allowed to continue executing until its specified task timeout or until the 4-hour window ends.
- C. The instances are placed into hibernation.
- D. The maintenance window duration is automatically extended by 1 hour.

---

### Question 5: SSM Inventory for Software License Auditing

An enterprise IT audit requires generating a report listing every version of Python, Java, and OpenSSL installed across 1,000 Amazon EC2 instances running in multiple regions, and tracking changes to installed software over time.

**Which Systems Manager feature collects this software metadata without installing third-party agents?**
- A. Systems Manager **Inventory** with an association configured to collect `AWS:Application` metadata periodically, integrated with AWS Config or Amazon Athena via an S3 Resource Data Sync.
- B. Running an interactive shell script on every instance via SSH.
- C. CloudWatch Logs Insights only.
- D. Amazon GuardDuty.

---

### Question 6: Granular Session Manager Access Control with IAM Tags

An operations team needs to allow developers to open interactive terminal sessions via AWS Systems Manager Session Manager **only** on EC2 instances that have the tag `Environment=Development`. Developers must be denied access to instances tagged `Environment=Production`.

**Which IAM policy condition enforces this tag-based access control?**
- A. `Condition: { "StringEquals": { "ssm:resourceTag/Environment": "Development" } }` on action `ssm:StartSession` targeting resource `arn:aws:ec2:*:*:instance/*`.
- B. Security group port 22 blocking.
- C. Removing the SSM Agent from production instances.
- D. IP address whitelisting.

---

### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Patch Manager Patch Baselines & Auto-Approval Delays

**Answer:** B  
**Explanation:** 
AWS Systems Manager Patch Manager uses **Patch Baselines** to define which patches are approved for installation. Within a baseline, engineers can define granular Approval Rules specifying classification, severity levels, and an **auto-approval delay** (the number of days to wait after patch release before the patch is marked as approved), ensuring adequate soaking time before automated rollout.

---

### Question 2: Patch Groups and Maintenance Windows Association

**Answer:** B  
**Explanation:** 
In SSM Patch Manager, **Patch Groups** allow associating instances with specific patch baselines and maintenance windows using the reserved tag key `Patch Group`. Instances tagged with `Patch Group = Staging` are targeted by the Tuesday maintenance window, while `Patch Group = Production` instances are targeted by the Thursday window, while both share the verified patch baseline rules.

---

### Question 3: Systems Manager State Manager Association Compliance

**Answer:** B  
**Explanation:** 
AWS Systems Manager State Manager maintains managed instances in a defined target state. The association specifies a target (by tags or instance IDs), an SSM document, and a recurrence schedule. When the schedule fires, the local SSM Agent evaluates the configuration, applies fixes if drift is detected, and updates the compliance dashboard in the Systems Manager console.

---

### Question 4: Systems Manager Maintenance Window Cutoff Behavior

**Answer:** B  
**Explanation:** 
In SSM Maintenance Windows:
- **Duration:** The total time window allotted for maintenance (e.g. 4 hours).
- **Cutoff:** The number of hours *before* the end of the window when Systems Manager stops starting new tasks (e.g. 1 hour). When the window reaches the cutoff point (hour 3 of 4), no new task invocations will launch, allowing in-flight tasks adequate runway to complete cleanly before the maintenance window expires.

---

### Question 5: SSM Inventory for Software License Auditing

**Answer:** A  
**Explanation:** 
**AWS Systems Manager Inventory** collects metadata from managed instances, including installed software applications (`AWS:Application`), network configurations, Windows updates, and AWS components via the standard SSM Agent. Configuring a **Resource Data Sync** automatically synchronizes inventory data across all instances, regions, and accounts into an Amazon S3 bucket, enabling SQL querying via Amazon Athena and historical auditing via AWS Config.

---

### Question 6: Granular Session Manager Access Control with IAM Tags

**Answer:** A  
**Explanation:** 
AWS IAM policies support attribute-based access control (ABAC) for Session Manager. By applying the condition `ssm:resourceTag/Environment: "Development"` on the `ssm:StartSession` action for instance ARNs, IAM checks the target instance's tags at the time the session is requested. If the instance is tagged `Production` or lacks the `Development` tag, the API call is denied.

---


### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

