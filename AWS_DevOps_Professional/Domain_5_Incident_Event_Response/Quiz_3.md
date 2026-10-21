# Domain 5 (Incident and Event Response) — Quiz 3: AWS Systems Manager Incident Manager

**Topic:** Response Plans, Escalation Paths, ChatOps Integration & Post-Incident Analysis  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Incident Manager Response Plan Components

An enterprise wants to automate critical incident engagement for severe production outages. When a P1 CloudWatch alarm triggers:
1. The on-call primary engineer must be paged immediately via SMS and voice call.
2. If the primary engineer does not acknowledge within 5 minutes, page the secondary engineering lead.
3. Automatically launch a predefined AWS Systems Manager Automation runbook to collect diagnostics.
4. Create a dedicated Slack incident collaboration channel via AWS Chatbot.

**Which AWS Systems Manager Incident Manager construct orchestrates all of these actions together?**
- A. SSM Patch Baseline.
- B. **SSM Incident Manager Response Plan**, which links CloudWatch Alarms/EventBridge events to engagement plans, escalation plans, SSM Automation runbooks, and ChatOps collaboration channels.
- C. AWS Organizations Service Control Policy.
- D. AWS Trusted Advisor check.

---

### Question 2: Engagement Plans vs Escalation Plans in Incident Manager

In AWS Systems Manager Incident Manager, what is the architectural distinction between an **Engagement Plan** and an **Escalation Plan**?
- A. An Engagement Plan defines how an individual contact is notified (e.g., call mobile after 0 min, SMS after 2 min), whereas an Escalation Plan defines the sequence and timing of paging across multiple contacts or teams (e.g., Page On-Call Team A, then escalate to Manager B after 15 minutes).
- B. Escalation Plans are for billing issues; Engagement Plans are for security issues.
- C. Engagement Plans can only send emails; Escalation Plans can only execute bash scripts.
- D. There is no distinction; they are identical aliases.

---

### Question 3: Post-Incident Analysis (Post-Mortem) Workflow

Following a major production database outage, executive leadership requires a formal Post-Incident Analysis (post-mortem) report detailing the timeline of events, root cause, metrics observed during the event, and actionable remediation tasks to prevent recurrence.

**How does AWS Systems Manager Incident Manager streamline this process?**
- A. It automatically sends an email to the AWS CEO.
- B. It generates a **Post-Incident Analysis** based on incident data, providing a structured template with a pre-populated timeline of CloudWatch alarms and responder actions, a 5-Whys root cause analysis framework, and direct integration to create Jira/SSM OpsCenter follow-up action items.
- C. It rewrites git commit history to remove the faulty commit.
- D. It restores all databases to factory defaults.

---

### Question 4: AWS Chatbot ChatOps Integration for Incident Collaboration

During an active incident, engineers collaborate in a dedicated Slack channel `#incident-prod-down`. The on-call engineer needs to run diagnostic commands against EC2 instances and execute SSM Automation runbooks directly from Slack without opening the AWS Management Console.

**How should this secure ChatOps workflow be configured?**
- A. Post AWS IAM root access keys directly into the Slack channel.
- B. Configure **AWS Chatbot** integrated with the Slack workspace, map the channel to a restricted IAM role with least-privilege permissions, and execute AWS CLI / SSM commands using `@aws run ssm ...` within the channel.
- C. Install an untrusted third-party bot with full administrator privileges.
- D. AWS Chatbot is read-only and cannot execute any operational actions.

---

### Question 5: Incident Manager Replication Sets for High Availability

Why does AWS Systems Manager Incident Manager require the creation of a **Replication Set** during initial service setup?
- A. To replicate EC2 AMI images across accounts for disaster recovery.
- B. To replicate incident response data (contacts, escalation plans, response plans, and active incident records) across multiple specified AWS Regions, ensuring that incident management remains operational even if the primary region hosting the workload suffers a catastrophic failure.
- C. To synchronize Git repositories with CodeCommit.
- D. To duplicate S3 buckets across AWS partitions.

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

### Question 1: Incident Manager Response Plan Components

**Answer:** B  
**Explanation:** 
An **Incident Manager Response Plan** predefines the automated response to an incident. A Response Plan brings together:
- **Triggers**: CloudWatch Alarms or EventBridge events that initiate the incident.
- **Engagement & Escalation Plans**: Contacts, paging channels (voice, SMS, email), and escalation timelines.
- **Automation Runbooks**: SSM Automation documents executed automatically at incident creation.
- **ChatOps**: Integration with AWS Chatbot to create collaborative channels in Slack, Microsoft Teams, or Chime.

---

### Question 2: Engagement Plans vs Escalation Plans in Incident Manager

**Answer:** A  
**Explanation:** 
- **Contact & Engagement Plan**: Configured for an individual contact, specifying their personal devices and wait times (e.g., notify Contact Alice via SMS at 0 minutes, then phone call at 3 minutes).
- **Escalation Plan**: Defines the team-level escalation hierarchy across multiple contacts (e.g., Tier 1 on-call engineer contacted at Stage 1; if unacknowledged after 10 minutes, advance to Stage 2 and engage the Tier 2 team and Engineering Director).

---

### Question 3: Post-Incident Analysis (Post-Mortem) Workflow

**Answer:** B  
**Explanation:** 
Incident Manager includes a dedicated **Post-Incident Analysis** capability. When an incident is resolved, Incident Manager automatically compiles a chronological timeline of alerts, metrics, runbook executions, and chat messages. It provides a structured post-mortem framework including questions for root cause analysis (such as the "5 Whys") and enables teams to track action items directly in Systems Manager OpsCenter or external issue trackers like Jira.

---

### Question 4: AWS Chatbot ChatOps Integration for Incident Collaboration

**Answer:** B  
**Explanation:** 
AWS Chatbot enables ChatOps for Slack and Microsoft Teams. It allows team members to receive real-time CloudWatch and Incident Manager notifications and execute commands directly from chat (e.g., `@aws ssm start-automation-execution ...`). Access is strictly controlled via IAM roles associated with the chat channel and user identity mapping with guardrail policies.

---

### Question 5: Incident Manager Replication Sets for High Availability

**Answer:** B  
**Explanation:** 
Incident Manager is designed to manage incidents even when entire AWS Regions are experiencing major outages. During setup, an administrator creates a **Replication Set**, selecting one or more AWS Regions. Incident Manager encrypts (via KMS CMKs) and synchronizes contacts, response plans, and incident state across these Regions. If an incident disables one Region, responders can seamlessly access and manage the incident from any other Region in the replication set.

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

