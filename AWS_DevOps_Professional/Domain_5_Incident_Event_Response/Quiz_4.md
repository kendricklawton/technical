# Domain 5 (Incident and Event Response) — Quiz 4: Systems Manager Automation & Multi-Account Remediation

**Topic:** Automation Documents, Rate Control (Concurrency/Error Threshold), Multi-Account Execution & Script Steps  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Cross-Account Multi-Region Automation via Administration & Execution Roles

An enterprise cloud security architect needs to run a critical security patch remediation runbook across 80 AWS member accounts in an AWS Organization. The runbook must execute from a central Management Account (`111122223333`) across 5 AWS Regions simultaneously.

**Which IAM role trust architecture and parameter configuration enables Systems Manager Automation to execute runbooks cross-account?**
- A. Deploy `AWS-SystemsManager-AutomationAdministrationRole` in the central Management Account and `AWS-SystemsManager-AutomationExecutionRole` in all target member accounts with trust to the administration role; invoke the automation specifying target accounts, target regions, and rate control parameters.
- B. Attach the `AdministratorAccess` policy to the central account root user and provide the root access keys in the automation document parameters.
- C. Create an IAM role named `SSMRunbookRole` in the central account and configure an AWS Lambda function to assume roles in a Python loop.
- D. Share the SSM Automation document across the organization via AWS RAM and trigger local executions from each member account independently.

---

### Question 2: Rate Control Mechanics: `MaxConcurrency` and `MaxErrors` Thresholds

A DevOps engineer needs to run an SSM Automation document that restarts services across a fleet of 500 mission-critical production EC2 instances. To ensure application availability is never compromised during the operation, the engineer mandates that:
1. No more than 10% of the fleet can be updated concurrently.
2. If more than 5 individual instances encounter errors, the entire automation execution must immediately stop further deployments.

**Which Systems Manager Automation rate control settings enforce these constraints?**
- A. Set **`MaxConcurrency: "10%"`** (or `50`) and set **`MaxErrors: 5`** (or `"1%"`), instructing Systems Manager to execute across batches of up to 50 instances and abort execution immediately once the 6th error is recorded.
- B. Set `MaxConcurrency: 5` and `MaxErrors: "10%"`, running 5 instances at a time and terminating if 50 fail.
- C. Configure an Auto Scaling Group lifecycle hook with `HeartbeatTimeout: 300` and `DefaultResult: ABANDON`.
- D. Deploy an AWS Step Functions state machine with a `Parallel` state and catch handlers configured for `States.ALL`.

---

### Question 3: Interactive Operator Approvals via `aws:approve` Action Step

A planned database maintenance runbook performs automated schema migrations on an Amazon RDS database. Before applying the changes, the runbook creates a DB snapshot. Corporate change management mandates that before executing the mutating schema changes, two designated senior DBAs must review the snapshot status and explicitly grant interactive approval within a 2-hour window, otherwise the runbook must abort.

**How is this governance checkpoint implemented inside the SSM Automation document?**
- A. Insert an **`aws:approve`** action step into the document specifying the IAM user/role ARNs of the approvers in `Approvers`, set `MinRequiredApprovals: 2`, configure `Timeout: PT2H`, and define an SNS topic ARN in `NotificationInformation` to dispatch approval requests.
- B. Insert an `aws:sleep` step set to 7,200 seconds and have DBAs edit the SSM Automation execution status via the AWS CLI.
- C. Configure an AWS Lambda function step that polls an external Jira ticket every 30 seconds for approval status.
- D. Create an EventBridge rule that intercepts the runbook and triggers an administrative email.

---

### Question 4: Embedded Code Execution via `aws:executeScript` and Payload Extraction

A DevOps team needs an SSM Automation runbook to query an external REST API, extract an authentication token from the JSON response, and pass that token dynamically into a subsequent `aws:runCommand` step executing on target EC2 instances.

**Which Systems Manager Automation action step allows executing inline Python code and passing outputs to downstream steps?**
- A. Use the **`aws:executeScript`** action step: Declare the `Runtime` (e.g., `python3.11`), embed the Python script directly in the `Script` property or reference an S3 file, specify input parameters in `InputPayload`, and extract the return value using the `Outputs` selector with JSONPath (e.g., `$.Payload.token`).
- B. Use the `aws:executeAutomation` step to launch an external CloudFormation template that provisions a temporary Lambda function.
- C. Use the `aws:invokeLambdaFunction` action step passing hardcoded static credentials.
- D. Embed shell scripts directly into an `aws:branch` action step and write the token to `/tmp/token.txt`.

---

### Question 5: Conditional Branching Logic in Automation Documents via `aws:branch`

An automated incident response runbook inspects an Amazon EC2 instance suspected of compromise. If the instance has a tag `Environment: Production`, the runbook must isolate the instance by attaching a quarantine security group and snapshotting EBS volumes. If the tag is `Environment: Development`, the runbook must terminate the instance immediately.

**How should conditional execution paths be declared inside the SSM Automation document?**
- A. Use the **`aws:branch`** action step: Define conditional rules matching the string value of the instance tag (e.g., `StringEquals: "Production"`), specifying `NextStep: IsolateProductionInstance`, and define an alternate rule or default step pointing to `TerminateDevInstance`.
- B. Define two separate SSM Automation documents and execute both concurrently from Amazon EventBridge.
- C. Write an external bash script using `if/else` statements executed via the `aws:runCommand` action.
- D. Use CloudFormation conditions inside the SSM Document metadata block.

---

### Question 6: Automated Security Remediation via AWS Config and SSM Automation

Corporate compliance requires that any Amazon S3 bucket found to have public read access enabled must be remediated automatically within 60 seconds of detection. The remediation must apply S3 Public Access Block settings to the non-compliant bucket without manual human intervention.

**Which architecture provides this automated closed-loop compliance remediation?**
- A. Deploy the AWS Config managed rule `s3-bucket-public-read-prohibited`, configure an **Automatic Remediation** action linking the rule to the AWS-managed SSM Automation document **`AWS-DisableS3BucketPublicReadWrite`**, and map the non-compliant resource ID dynamically to the `BucketName` parameter.
- B. Write a cron job on an EC2 instance that runs `aws s3api get-bucket-acl` every minute and removes public permissions.
- C. Configure an S3 Bucket Policy with an explicit deny on `s3:GetObject` applied across all organization accounts via SCP.
- D. Deploy an Amazon GuardDuty detector that invokes an AWS Lambda function to delete non-compliant S3 buckets.

---

### Question 7: Passing Dynamic Event Context from EventBridge to SSM Automation

An enterprise needs an automated remediation pipeline that runs whenever an EC2 instance state changes to `stopping` unexpectedly during business hours. An EventBridge rule detects the `EC2 Instance State-change Notification` event and must trigger an SSM Automation runbook that automatically restarts the instance.

**How does EventBridge pass the dynamic EC2 instance ID from the event payload to the SSM Automation runbook?**
- A. In the EventBridge Rule target configuration, configure an **Input Transformer**: Use the **Input Paths Map** to extract the instance ID (`{"instance": "$.detail.instance-id"}`), and format the **Input Template** to pass the JSON parameter expected by the runbook: `{"InstanceId": [<instance>]}`.
- B. EventBridge automatically injects all root JSON attributes into the SSM runbook parameters without configuration.
- C. The SSM runbook must execute `aws ec2 describe-instances` upon startup to guess which instance triggered the alert.
- D. Configure an intermediate Amazon SQS queue that buffers events for an external Python daemon.

---

### Question 8: Continuous Drift Enforcement via Systems Manager State Manager Associations

An enterprise security policy dictates that all managed Linux EC2 instances across 10 VPCs must continuously enforce CIS Level 1 benchmark configurations. Rather than reacting to alerts post-drift, the system must evaluate and re-apply compliant configuration states automatically every 6 hours, recording full compliance history.

**Which Systems Manager feature satisfies this requirement with MINIMAL custom engineering?**
- A. Configure a **Systems Manager State Manager Association**: Target instances by tag (e.g., `OS: Linux`), bind the association to an SSM Document (e.g., `AWS-RunAnsiblePlaybook` or custom CIS document), set a recurring schedule of 6 hours, and enable compliance reporting in SSM Compliance.
- B. Deploy AWS Systems Manager Patch Manager with a daily patch baseline scanning window.
- C. Author an AWS Step Functions state machine that invokes SSM Run Command every 6 hours via EventBridge Scheduler.
- D. Create an Amazon EC2 Auto Scaling scheduled action to terminate and relaunch the entire fleet every 6 hours.

---

### Question 9: Resilient Error Handling in Runbooks using `onFailure` and `isCritical`

A multi-step disaster recovery SSM Automation runbook performs failover operations:
1. Step 1: Promote database replica.
2. Step 2: Clear CloudFront cache.
3. Step 3: Update Route 53 DNS records.

If Step 2 (cache clearing) fails or times out, the failover process must NOT halt; it should log a warning and proceed to Step 3. However, if Step 1 (database promotion) fails, the entire runbook must halt immediately and execute an emergency cleanup step.

**How are step failure behaviors controlled within the SSM Automation document specification?**
- A. On Step 1, set **`isCritical: true`** and configure **`onFailure: "step:EmergencyCleanup"`**; on Step 2, set **`isCritical: false`** and configure **`onFailure: "Continue"`** (or set `nextStep: Step3`).
- B. Wrap all three steps in a Python `try/except` block inside an `aws:executeScript` step.
- C. Configure a CloudWatch Alarm on runbook execution duration and abort via the AWS CLI.
- D. Split the runbook into three separate documents and execute them sequentially using AWS CodePipeline.

---

## Answer Key & Explanations

### Question 1: Cross-Account Multi-Region Automation via Administration & Execution Roles
**Answer:** A  
**Explanation:** 
AWS Systems Manager Automation enables centralized multi-account and multi-region execution using standard cross-account IAM roles:
- **Management Account:** Configured with `AWS-SystemsManager-AutomationAdministrationRole`, which possesses permissions to assume execution roles in spoke accounts.
- **Member Accounts:** Each target account contains `AWS-SystemsManager-AutomationExecutionRole` with a trust policy allowing the administration role in the central account to assume it.
- When invoking the automation, you specify target accounts (or an AWS Organizations OU ID), target AWS regions, and rate control parameters (`MaxConcurrency`, `MaxErrors`). Systems Manager automatically assumes the execution roles across target accounts and executes the runbook steps in parallel.

---

### Question 2: Rate Control Mechanics: `MaxConcurrency` and `MaxErrors` Thresholds
**Answer:** A  
**Explanation:** 
Systems Manager Automation provides fine-grained rate controls:
- **`MaxConcurrency`**: Dictates how many targets execute the runbook simultaneously. Specifying `"10%"` (or `50` instances out of 500) ensures that at least 90% of the fleet remains active and unaffected while batches are being processed.
- **`MaxErrors`**: Dictates the failure threshold before the entire execution is aborted. Specifying `5` (or `"1%"`) means that as soon as 6 instances fail, Systems Manager halts all pending and in-flight executions, preventing a faulty runbook or bad patch from rolling across the remainder of the fleet.

---

### Question 3: Interactive Operator Approvals via `aws:approve` Action Step
**Answer:** A  
**Explanation:** 
The **`aws:approve`** action step pauses an SSM Automation execution until designated approvers grant or deny permission:
- `Approvers`: Specifies a list of IAM user or role ARNs authorized to approve.
- `MinRequiredApprovals`: Number of approvals required to proceed (e.g., `2`).
- `Timeout`: ISO 8601 duration (e.g., `PT2H` for 2 hours). If the timeout elapses without sufficient approvals, the step fails and execution halts.
- `NotificationInformation`: Points to an Amazon SNS topic ARN to alert approvers via email or SMS that an approval request is pending.

---

### Question 4: Embedded Code Execution via `aws:executeScript` and Payload Extraction
**Answer:** A  
**Explanation:** 
The **`aws:executeScript`** action step allows running inline Python or PowerShell scripts natively within the Systems Manager Automation service:
- You declare the `Runtime` (e.g., `python3.11`).
- The script code can make HTTP requests, parse JSON, or call AWS APIs.
- The script returns a dictionary. You define `Outputs` using JSONPath selectors (e.g., `Name: AuthToken, Selector: $.Payload.token, Type: String`).
- Subsequent steps in the document can reference this output dynamically using variable substitution syntax: `{{ aws:executeScriptStepName.AuthToken }}`.

---

### Question 5: Conditional Branching Logic in Automation Documents via `aws:branch`
**Answer:** A  
**Explanation:** 
The **`aws:branch`** action step provides dynamic conditional branching inside Systems Manager Automation documents:
- It evaluates conditions against document inputs, step outputs, or system variables.
- Supported operations include `StringEquals`, `NumericEquals`, `BooleanEquals`, etc.
- If a condition evaluates to true, execution jumps to the designated `NextStep` (e.g., `IsolateProductionInstance`).
- A `DefaultStep` can be defined to handle unmatched conditions (e.g., `TerminateDevInstance`). This allows a single document to handle multiple environment tiers or recovery scenarios cleanly.

---

### Question 6: Automated Security Remediation via AWS Config and SSM Automation
**Answer:** A  
**Explanation:** 
AWS Config paired with Systems Manager Automation provides automated compliance remediation:
- The managed rule `s3-bucket-public-read-prohibited` detects non-compliant buckets.
- In AWS Config, you configure **Remediation**: choose **Automatic Remediation**, select the AWS-managed document `AWS-DisableS3BucketPublicReadWrite`, and map the parameter `BucketName` to the non-compliant resource ID via `RESOURCE_ID`.
- As soon as a bucket is evaluated as non-compliant, Config automatically triggers the SSM Automation runbook, enabling S3 Block Public Access within seconds.

---

### Question 7: Passing Dynamic Event Context from EventBridge to SSM Automation
**Answer:** A  
**Explanation:** 
When an Amazon EventBridge Rule triggers an SSM Automation runbook:
- The event payload contains dynamic metadata (e.g., `{"detail": {"instance-id": "i-0123456789abcdef0"}}`).
- To map this into the specific parameter expected by the SSM document (e.g., `InstanceId`), you configure an **Input Transformer** on the EventBridge rule target:
  - **Input Path:** `{"instance": "$.detail.instance-id"}`
  - **Input Template:** `{"InstanceId": [<instance>]}`
- EventBridge formats the JSON payload and passes it directly to the Systems Manager `StartAutomationExecution` API.

---

### Question 8: Continuous Drift Enforcement via Systems Manager State Manager Associations
**Answer:** A  
**Explanation:** 
**AWS Systems Manager State Manager** is a configuration management service that automates the process of keeping managed instances in a defined state:
- An **Association** binds a target set of instances (defined by tags, resource groups, or instance IDs) to an SSM Document (such as an Ansible playbook or shell script).
- You define a recurring schedule (e.g., every 6 hours via cron or rate).
- State Manager automatically re-applies the configuration, remediating configuration drift without human intervention, and reports compliance status to AWS Systems Manager Compliance.

---

### Question 9: Resilient Error Handling in Runbooks using `onFailure` and `isCritical`
**Answer:** A  
**Explanation:** 
Systems Manager Automation documents support robust step-level failure handling:
- **`isCritical`**: When set to `true` (default), failure of that step causes the entire automation to fail. When set to `false`, execution can continue to subsequent steps even if this step fails.
- **`onFailure`**: Defines the failure action. Setting `onFailure: "Continue"` or `onFailure: "step:NextStepName"` directs execution forward. Setting `onFailure: "step:EmergencyCleanup"` jumps immediately to an error-handling or rollback routine.
