# Domain 5 (Incident and Event Response) — Quiz 5: Automated Security Incident Remediation

**Topic:** GuardDuty Alerts to EventBridge, Compromised EC2 Isolation & AWS Config Auto-Remediation  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Automated Isolation of an EC2 Instance Compromised by C2 Malware

Amazon GuardDuty generates a high-severity finding `UnauthorizedAccess:EC2/MaliciousIPCaller.Custom` indicating an EC2 instance is communicating with a known Command and Control (C2) server. The security incident response plan requires automated immediate network quarantine of the instance while preserving its memory and state for forensic investigation.

**Which event-driven architecture fulfills these requirements with the lowest latency?**
- A. Terminate the instance immediately using an Auto Scaling notification.
- B. Create an **EventBridge Rule** matching GuardDuty findings with severity >= 7.0. The rule triggers an **AWS Systems Manager Automation runbook** or **AWS Lambda function** that:
  1. Replaces the instance's Security Groups with an "Isolation Security Group" (no inbound/outbound rules).
  2. Disables termination protection and detaches from any Auto Scaling Group.
  3. Triggers EBS volume snapshots of all attached volumes for forensic analysis.
- C. Send an email alert to the security inbox and wait for manual response during business hours.
- D. Reboot the instance via CloudWatch Alarms.

---

### Question 2: AWS Config Auto-Remediation using SSM Automation Documents

A company policy strictly forbids public S3 buckets. An AWS Config managed rule `s3-bucket-public-read-prohibited` is deployed across all accounts. When a developer makes an S3 bucket publicly readable, the compliance status shifts to `NON_COMPLIANT`.

**How can the DevOps engineer configure AWS Config to automatically remediate this non-compliance immediately without manual intervention?**
- A. Delete the S3 bucket via S3 Lifecycle.
- B. Configure **Automatic Remediation** on the AWS Config rule, targeting the AWS-managed SSM Automation Document `AWS-DisableS3BucketPublicReadWrite`, passing the non-compliant bucket name as the parameter.
- C. Run a cron job on an EC2 instance that runs `aws s3api` every hour.
- D. AWS Config cannot take remediation actions; it is purely a reporting tool.

---

### Question 3: Compromised IAM Access Keys Automated Revocation

Amazon GuardDuty detects an anomalous API call from a Tor exit node using the IAM access keys of an administrator: `UnauthorizedAccess:IAMUser/TorIPCaller`.

**What automated actions should the remediation pipeline execute immediately to neutralize the threat?**
- A. Email the administrator asking if they are traveling.
- B. An EventBridge rule triggers a Lambda function that executes `aws iam update-access-key --status Inactive` (or deletes the key), applies an inline IAM policy denying all actions (`aws:PrincipalArn` deny `*`), and terminates any active STS temporary sessions by revoking sessions prior to the current timestamp.
- C. Delete the AWS account.
- D. Block the Tor IP address in Route 53.

---

### Question 4: Preserving Forensic Evidence on EBS Volumes

During an incident investigation of an EC2 instance suspected of data exfiltration, the digital forensics team needs to analyze the volume data in an isolated forensic AWS account without risking data modification or destruction.

**What is the correct procedure to transfer and analyze the evidence securely?**
- A. Copy the root partition files over public FTP.
- B. Create a point-in-time **Amazon EBS Snapshot** of the compromised volume, encrypt it with a customer-managed KMS key (CMK), share the KMS key and snapshot with the isolated Forensic AWS Account, copy the snapshot in the Forensic Account, and create a volume from the copied snapshot attached to a dedicated forensic analysis instance.
- C. Download the EBS volume to an external USB drive.
- D. Run `dd` over an unencrypted SSH tunnel to an on-premises workstation.

---

### Question 5: GuardDuty Finding Export Architecture for Long-Term SIEM Ingestion

An enterprise needs to retain all Amazon GuardDuty findings across 100 AWS accounts for 7 years and feed them in real time into an external third-party SIEM (such as Splunk or IBM QRadar).

**What is the native, scalable architecture to accomplish this?**
- A. Have developers export CSV files from the GuardDuty console every week.
- B. Configure GuardDuty to **export findings to an Amazon S3 bucket** encrypted with a KMS CMK (which handles high-volume continuous exports natively) and configure an EventBridge rule that streams findings to **Amazon Kinesis Data Firehose** for real-time delivery to the external SIEM HTTP Event Collector (HEC).
- C. Store findings in EC2 instance store volumes.
- D. Run a polling script against the `GetFindings` API every 10 seconds.

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

### Question 1: Automated Isolation of an EC2 Instance Compromised by C2 Malware

**Answer:** B  
**Explanation:** 
Terminating or rebooting a compromised instance destroys volatile RAM memory and evidence critical for forensic analysis. The standard AWS automated incident response pattern is:
1. EventBridge captures the high-severity GuardDuty finding in real time.
2. Lambda or SSM Automation modifies the instance's network interfaces to attach a quarantine security group with 0 ingress and 0 egress rules, cutting off C2 communication immediately.
3. The instance is removed from Auto Scaling (to prevent ASG from killing it for failing health checks).
4. EBS snapshots are created for forensic investigation.

---

### Question 2: AWS Config Auto-Remediation using SSM Automation Documents

**Answer:** B  
**Explanation:** 
AWS Config supports **Remediation Actions** powered by AWS Systems Manager Automation. You can associate an SSM Automation document (either AWS-managed like `AWS-DisableS3BucketPublicReadWrite` or custom runbooks) with any Config rule. When set to Automatic Remediation, Config immediately initiates the SSM Automation document whenever a resource is evaluated as `NON_COMPLIANT`, achieving automated self-healing governance.

---

### Question 3: Compromised IAM Access Keys Automated Revocation

**Answer:** B  
**Explanation:** 
When an IAM user credential compromise is detected:
1. Deactivate the IAM access key immediately (`UpdateAccessKey` to `Inactive`) to prevent further API calls.
2. Revoke active temporary sessions generated via `AssumeRole` / STS by attaching an IAM policy with `aws:CurrentTime < [RevocationTimestamp]` or revoking user policies.
3. Apply an explicit `DenyAll` policy to prevent the identity from performing any actions until investigated.

---

### Question 4: Preserving Forensic Evidence on EBS Volumes

**Answer:** B  
**Explanation:** 
Best practices for cloud forensics dictate maintaining chain of custody and immutability:
1. Take an EBS snapshot of the affected volumes.
2. Encrypt the snapshot with a Customer Managed Key (CMK).
3. Grant the dedicated Forensic AWS Account access to the KMS key and snapshot.
4. In the Forensic Account, copy the snapshot (which re-encrypts with a forensic-account-owned KMS key) to prevent the source account from altering or deleting it.
5. Spin up read-only analysis tools against a volume created from the copied snapshot.

---

### Question 5: GuardDuty Finding Export Architecture for Long-Term SIEM Ingestion

**Answer:** B  
**Explanation:** 
Amazon GuardDuty supports two native delivery mechanisms:
1. **S3 Finding Export**: Automatically delivers all findings (updated within 5 minutes) to a centralized S3 bucket with KMS encryption for long-term immutable compliance storage.
2. **Amazon EventBridge integration**: Streams finding events in real time to targets like Kinesis Data Firehose, which natively integrates with third-party SIEM endpoints (Splunk, Datadog, New Relic) via HTTP Event Collector endpoints.

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

