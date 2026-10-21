# Domain 5: Incident and Event Response — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 14% of Exam  
**Format:** 15 Deep Scenario-Based Professional Practice Questions with Detailed Explanations

---

### Question 1: Cross-Account Event Routing with Amazon EventBridge

An enterprise uses a multi-account AWS Organization. The central Security Operations Center (SOC) operates in Account `999999999999` (Security). Whenever an unauthorized port is opened in any Security Group in Account `111111111111` (Production), an event must be routed in real-time to Account `999999999999`'s event bus for automated forensic analysis.

**How should this cross-account event routing be configured?**
- A. In Account `111111111111`, write a Lambda function that calls `events:PutEvents` directly to the security account's bus using assumed IAM credentials.
- B. In Account `999999999999`, configure an **EventBridge Event Bus** with a resource-based policy that grants `events:PutEvents` permission to Account `111111111111`. In Account `111111111111`, create an **EventBridge Rule** on the default bus matching AWS API call `AuthorizeSecurityGroupIngress`, and set the target to the ARN of the event bus in Account `999999999999`.
- C. Use Amazon SQS to link the accounts over a VPC Peering connection.
- D. Cross-account event bus routing is not supported natively in Amazon EventBridge.

---

### Question 2: AWS Fault Injection Service (FIS) for Resilience Testing

A company is designing a chaos engineering drill to validate that their Amazon EKS application can survive sudden node terminations and simulated network packet drop without impacting customer transactions. The security team mandates that chaos experiments must have an automated safety stop mechanism that cancels the experiment immediately if customer-facing error rates increase.

**Which AWS service and feature satisfies these requirements?**
- A. **AWS Fault Injection Service (FIS)** with an experiment template defining actions (e.g. `aws:eks:terminate-nodegroup-instances`, `aws:network:disrupt-connectivity`) and configured with **Stop Conditions** linked to Amazon CloudWatch Alarms monitoring application HTTP 5xx errors.
- B. A Python script running on an EC2 instance that executes `pkill` in an infinite loop.
- C. AWS Systems Manager Run Command that executes random reboots.
- D. CloudWatch Synthetics canaries with destructive payloads.

---

### Question 3: EventBridge Archive and Replay for Incident Recovery

A critical billing ingestion pipeline powered by Amazon EventBridge, AWS Lambda, and Amazon Aurora failed for 4 hours due to an unhandled database schema migration bug. During the outage, EventBridge successfully matched thousands of incoming invoice events, but the target Lambda function threw exceptions, causing the events to be lost after exhausting retries.

**How can the DevOps Engineer design the EventBridge architecture to ensure past events can be safely reprocessed after the bug is resolved?**
- A. Enable **Amazon EventBridge Archive** on the event bus with a retention period of 30 days. When the downstream bug is patched, initiate an **EventBridge Replay** specifying the archive name, start time, end time, and target event bus.
- B. Instruct upstream clients to re-send all invoices via email.
- C. Query CloudTrail management events and replay API requests manually.
- D. S3 bucket versioning will automatically restore the events.

---

### Question 4: Systems Manager Automation Multi-Account Execution with Rate Control

A security vulnerability requires immediately revoking a compromised security group rule (`0.0.0.0/0` on port 22) across 200 AWS accounts in an AWS Organization. The operations team has an approved SSM Automation runbook `RevokeSSHIngress` in the management account.

**How should this remediation runbook be executed across all 200 accounts simultaneously with rate controls?**
- A. Log into each of the 200 accounts individually and run the script.
- B. Execute the SSM Automation runbook using **Multi-Account and Multi-Region execution** from the management or delegated administrator account. Specify the target **AWS Organizations OU**, an execution IAM role name deployed across member accounts, set `--max-concurrency 20`, and `--max-errors 5`.
- C. Use AWS CloudFormation to delete all security groups.
- D. Write an AWS Lambda function that loops through accounts using a `for` loop.

---

### Question 5: EventBridge Input Transformers for Alert Formatting

An operations team configures an Amazon EventBridge rule that captures Amazon EC2 instance state changes (`source: aws.ec2`, `detail-type: EC2 Instance State-change Notification`). The team needs to send a clean, human-readable notification to an Amazon SNS topic that states:
`"Alert: Instance i-0123456789 in us-east-1 changed state to stopped at 2026-10-07T10:00:00Z."`

Currently, SNS sends the raw, complex JSON event payload to subscribers.

**How can the event payload be transformed into the desired string format before reaching SNS?**
- A. Configure an **Input Transformer** on the EventBridge rule target. In the **Input Path**, map variables: `{"instance": "$.detail.instance-id", "state": "$.detail.state", "region": "$.region", "time": "$.time"}`. In the **Input Template**, define the formatted string: `"Alert: Instance <instance> in <region> changed state to <state> at <time>."`
- B. Write a Lambda function between EventBridge and SNS to parse the JSON.
- C. Change the SNS topic subscription type to Raw Message Delivery.
- D. CloudWatch Metrics can automatically format SNS text strings.

---

### Question 6: AWS Health API Integration for Scheduled Maintenance

An enterprise runs latency-sensitive database workloads on Amazon EC2 dedicated hosts. AWS periodically schedules maintenance events on underlying hardware hosts, which can require instance reboots. The operations team needs automated notification 14 days in advance to drain traffic and migrate workloads proactively.

**Which service architecture detects upcoming AWS infrastructure maintenance events?**
- A. Amazon Inspector scanning host firmware.
- B. **AWS Health Dashboard (AWS Health API)** emitting events to **Amazon EventBridge** for event source `aws.health`. Create an EventBridge rule matching `detail.service: ["EC2"]` and `detail.eventTypeCode: ["AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED"]` that targets an SNS topic and opens an operational Jira ticket.
- C. CloudWatch Metric Alarms on EC2 CPU Utilization.
- D. Polling the AWS status webpage (`status.aws.amazon.com`) every 5 minutes.

---

### Question 7: Systems Manager Incident Manager Post-Incident Analysis (PIR)

Following a major customer-facing outage, the engineering director mandates a formal Post-Incident Analysis (PIR) to document the root cause, timeline of events, metrics impact, and preventive action items.

**Which native capability of AWS Systems Manager Incident Manager supports this workflow?**
- A. SSM Session Manager terminal replay.
- B. **Incident Manager Post-Incident Analysis (PIR)**, which automatically compiles the incident timeline, engaged responders, executed runbooks, and CloudWatch metrics into an editable post-incident report with action item tracking.
- C. AWS CloudTrail Event History export to CSV.
- D. Amazon GuardDuty incident summarization.

---

### Question 8: Automated Security Group Quarantine with Lambda

A security tool flags that an Amazon EC2 instance has communicated with a known malicious command-and-control (C2) IP address. An automated incident response workflow must immediately isolate the instance from the network while preserving the EC2 instance in its running state for memory forensics.

**What is the correct action to isolate the instance?**
- A. Terminate the EC2 instance immediately.
- B. Detach the Elastic Network Interface (ENI) from the EC2 instance.
- C. Replace all security groups attached to the instance's network interface with an **Isolation Security Group** that has zero inbound rules and zero outbound rules.
- D. Disable the VPC Internet Gateway.

---

### Question 9: AWS Config Automated Remediation of Unencrypted EBS Volumes

An organization uses AWS Config to monitor compliance across all member accounts. When a developer creates an unencrypted Amazon EBS volume (`AWS::EC2::Volume`), AWS Config flags the resource as `NON_COMPLIANT` against the rule `encrypted-volumes`. The security team needs non-compliant unencrypted volumes to be automatically deleted immediately upon detection.

**How should this automated remediation be configured in AWS Config?**
- A. Under the AWS Config rule, configure an **Automatic Remediation** action referencing the SSM Automation document `AWS-DeleteVolume` with the parameter `VolumeId` mapped to `RESOURCE_ID`.
- B. Write a cron job on an EC2 instance that runs `aws ec2 delete-volume`.
- C. Create an IAM policy that deletes EBS volumes.
- D. Enable S3 Object Lock on EBS snapshots.

---

### Question 10: EventBridge API Destinations for Third-Party Webhook Paging

An enterprise needs to automatically trigger an incident in an external third-party Incident Management SaaS tool (e.g. PagerDuty, Opsgenie, or a custom internal webhook) whenever a high-severity AWS Health event occurs. The third-party API requires OAuth 2.0 client credential authentication and rate-limits incoming webhooks to 10 requests per second.

**Which serverless EventBridge feature connects directly to this external API without writing Lambda proxy code?**
- A. Amazon EventBridge Pipes with SQS.
- B. **Amazon EventBridge API Destinations** with an **EventBridge Connection** configured for OAuth 2.0 Client Credentials and an invocation rate limit of `10 requests per second`.
- C. Amazon SNS HTTP subscription with plain text basic auth.
- D. An EC2 instance running NGINX proxying requests to the SaaS tool.

---

### Question 11: Amazon SQS Message Redrive to Source Queue with Filter

A payment processor discovers that an upstream bug caused 10,000 transactions to be rejected and sent to an SQS Dead Letter Queue (DLQ). The 10,000 messages consist of two types: `TypeA` (which are now fixable) and `TypeB` (which remain invalid). The team wants to move only `TypeA` messages back to the source queue for immediate processing, while leaving `TypeB` messages in the DLQ.

**Can SQS Dead Letter Queue Redrive filter messages during transfer, and how should this be executed?**
- A. SQS DLQ Redrive only moves 100% of messages; filtering requires an AWS Lambda function to inspect message attributes or body, call `SendMessage` to the source queue for `TypeA`, and call `DeleteMessage` on the DLQ.
- B. SQS DLQ Redrive supports SQL where clauses directly in the console.
- C. Use Route 53 to filter SQS messages by DNS record.
- D. Increase the message retention period on `TypeB` messages.

---

### Question 12: Systems Manager Session Manager Audit Logging

An enterprise security policy forbids opening port 22 on any EC2 instance and mandates that all administrative terminal access must be performed via AWS Systems Manager Session Manager. Furthermore, every keystroke and shell command executed during an administrative session must be logged to an encrypted, tamper-evident location for auditing.

**How should Session Manager logging be configured?**
- A. Configure Session Manager preferences to stream session logs to **Amazon CloudWatch Logs** and/or an **Amazon S3 bucket** encrypted with an AWS KMS Customer Managed Key (CMK). Apply an S3 Bucket Policy preventing deletion and enable S3 Object Lock.
- B. Install a keylogger on the EC2 instances and email logs to the administrator.
- C. Store session recordings in the local `/tmp` folder on each instance.
- D. Session Manager does not support logging interactive shell commands.

---

### Question 13: Amazon GuardDuty Automated Suppression Rules

A DevOps team deploys continuous automated integration testing pipelines that launch temporary EC2 instances. These test instances frequently run automated vulnerability assessment tools, causing Amazon GuardDuty to trigger hundreds of `Recon:EC2/Portscan` findings, cluttering the security console and paging on-call engineers.

**How can the team prevent these expected testing findings from generating alerts without disabling GuardDuty?**
- A. Disable GuardDuty in the development AWS account.
- B. Create **Amazon GuardDuty Suppression Rules** with criteria matching the finding type `Recon:EC2/Portscan` and the specific IAM Role or tag assigned to the testing instances. Configure the rule to automatically archive matching findings.
- C. Write an EventBridge rule that deletes GuardDuty findings.
- D. Add the testing instances' private IPs to the VPC route table.

---

### Question 14: Emergency IAM Access Revocation with AWS Organizations

During a critical security incident, an administrator discovers that an IAM role in a member account has been compromised and is actively exfiltrating data. The incident commander needs to terminate all active sessions established by that compromised role across the entire organization within 30 seconds.

**Which combination of actions immediately revokes all active sessions? (Select TWO)**
- A. Delete the IAM role in the member account.
- B. In the IAM console, select the compromised IAM role, navigate to the **Revoke sessions** tab, and click **Revoke active sessions**. This attaches an inline policy with an `aws:CurrentTime` condition that denies all actions taken by sessions issued before the revocation timestamp.
- C. Apply a Service Control Policy (SCP) at the root level denying all actions for that role ARN.
- D. Reboot all EC2 instances in the account.
- E. Request AWS Support to revoke the STS credentials.

---

### Question 15: Cross-Region EventBridge Replication for Disaster Recovery

An enterprise operates an event-driven architecture with microservices in `us-east-1` and `us-west-2`. Core business events published to the default event bus in `us-east-1` must be mirrored in real-time to the event bus in `us-west-2` so the secondary region has full event history and can process failover workloads seamlessly.

**How should cross-region event mirroring be configured in EventBridge?**
- A. Write a Lambda function in `us-east-1` that sends HTTP POST requests to `us-west-2`.
- B. In `us-east-1`, create an **Amazon EventBridge Rule** that matches the required events and sets the **Target** directly to the ARN of the event bus in `us-west-2`.
- C. Enable S3 Cross-Region Replication on the EventBridge schema registry.
- D. Use AWS Direct Connect between regions.

---

## Answer Key & Detailed Explanations

### Question 1: Cross-Account Event Routing with Amazon EventBridge

**Answer:** B  
**Explanation:** 
Amazon EventBridge natively supports cross-account event delivery. The receiving event bus in the central account (`999999999999`) attaches a resource-based policy permitting `events:PutEvents` from member accounts or the entire organization. In the source account (`111111111111`), an EventBridge rule matches the security group modification event and targets the central event bus ARN directly, using an execution role that permits cross-account delivery.

---

### Question 2: AWS Fault Injection Service (FIS) for Resilience Testing

**Answer:** A  
**Explanation:** 
**AWS Fault Injection Service (FIS)** is a fully managed chaos engineering service. It allows teams to safely run fault injection experiments (CPU stress, node termination, network packet loss, API throttling) across AWS infrastructure. Crucially, FIS enforces **Stop Conditions**: you attach CloudWatch alarms to the experiment, and if an alarm breaches threshold during the drill (e.g., customer 5xx error spike), FIS automatically aborts the experiment and rolls back the injected faults immediately.

---

### Question 3: EventBridge Archive and Replay for Incident Recovery

**Answer:** A  
**Explanation:** 
**Amazon EventBridge Archive and Replay** allows capturing all (or filtered) events processed by an event bus into an encrypted, long-term archive. When downstream microservices fail, operators can fix the code and trigger an **Event Replay**, which re-dispatches the historical events in chronological order through the event bus and rules, recovering state with zero loss of transaction history.

---

### Question 4: Systems Manager Automation Multi-Account Execution with Rate Control

**Answer:** B  
**Explanation:** 
AWS Systems Manager Automation supports **Multi-Account and Multi-Region execution**. From a central account, an automation runbook can target multiple AWS accounts (using account IDs or entire Organizational Units) and regions simultaneously. It includes built-in rate controls (`MaxConcurrency` and `MaxErrors`) to govern execution velocity and halt operations automatically if error thresholds are exceeded.

---

### Question 5: EventBridge Input Transformers for Alert Formatting

**Answer:** A  
**Explanation:** 
Amazon EventBridge **Input Transformers** allow extracting JSON properties from incoming event payloads using JSONPath (`Input Path`) and injecting them into a customizable text template (`Input Template`). This transforms raw AWS event JSON into concise, human-readable notifications before delivering them to SNS, SQS, or webhook targets without requiring an intermediary Lambda function.

---

### Question 6: AWS Health API Integration for Scheduled Maintenance

**Answer:** B  
**Explanation:** 
The **AWS Health** service delivers personalized notifications about AWS service events, planned maintenance, and account changes. AWS Health events stream natively into Amazon EventBridge under `source: aws.health`. By filtering on specific event type codes (such as instance retirement or scheduled maintenance), teams can automate proactive workload migration and trigger incident tickets weeks before AWS initiates forced reboots.

---

### Question 7: Systems Manager Incident Manager Post-Incident Analysis (PIR)

**Answer:** B  
**Explanation:** 
AWS Systems Manager Incident Manager includes a dedicated **Post-Incident Analysis** capability. When an incident is resolved, Incident Manager automatically generates an analysis draft populated with the chronological incident timeline, responder engagements, chat channels, executed automation steps, and key metric snapshots. Teams can document root causes and assign tracked remediation tasks directly in AWS Systems Manager.

---

### Question 8: Automated Security Group Quarantine with Lambda

**Answer:** C  
**Explanation:** 
To isolate an EC2 instance without rebooting or destroying ephemeral RAM contents needed for live forensics, the standard incident response procedure is to replace its assigned security groups with an **Isolation / Quarantine Security Group**. A security group with no inbound rules and no outbound rules immediately severs all established and new TCP/UDP connections at the hypervisor boundary while keeping the instance alive and accessible via AWS Systems Manager Session Manager (if configured with private VPC endpoints).

---

### Question 9: AWS Config Automated Remediation of Unencrypted EBS Volumes

**Answer:** A  
**Explanation:** 
AWS Config supports **Remediation Actions** backed by AWS Systems Manager Automation documents. For the managed rule `encrypted-volumes`, you can attach the document `AWS-DeleteVolume` with an execution mode of `Automatic`. When a non-compliant volume is detected, AWS Config automatically executes the SSM document, passing the non-compliant volume's ID as `VolumeId`, terminating the non-compliant resource without human intervention.

---

### Question 10: EventBridge API Destinations for Third-Party Webhook Paging

**Answer:** B  
**Explanation:** 
**Amazon EventBridge API Destinations** allow EventBridge rules to route events directly to external HTTP webhooks and REST APIs. Coupled with **EventBridge Connections**, it securely manages authentication (OAuth, API Key, or Basic Auth) and handles token refreshes automatically. API Destinations also include built-in rate-limiting controls (invocations per second), preventing outgoing bursts from overwhelming or violating rate limits of downstream third-party endpoints.

---

### Question 11: Amazon SQS Message Redrive to Source Queue with Filter

**Answer:** A  
**Explanation:** 
While Amazon SQS Dead-Letter Queue Redrive allows moving messages from a DLQ back to a source queue (or custom destination), native DLQ Redrive acts on message batches without inspecting message content or filtering by custom payload attributes. Selective redrive (filtering `TypeA` vs `TypeB`) requires a custom consumer script or an AWS Lambda function that inspects message attributes/body and re-publishes matching messages.

---

### Question 12: Systems Manager Session Manager Audit Logging

**Answer:** A  
**Explanation:** 
AWS Systems Manager Session Manager provides native session logging. In Session Manager preferences, administrators can configure streaming of full session transcripts (including all shell inputs and outputs) directly to Amazon CloudWatch Logs and/or an Amazon S3 bucket. Encrypting the destination bucket with a KMS CMK and applying S3 Object Lock ensures that administrative session logs are immutable and tamper-proof.

---

### Question 13: Amazon GuardDuty Automated Suppression Rules

**Answer:** B  
**Explanation:** 
Amazon GuardDuty **Suppression Rules** allow organizations to automatically archive findings that match specific criteria (such as low-severity findings, specific finding types, or findings originating from known testing environments and specific tagged instances). Suppressed findings are still logged and stored in finding history for auditability, but they do not generate alerts or trigger downstream notifications.

---

### Question 14: Emergency IAM Access Revocation with AWS Organizations

**Answer:** B, C  
**Explanation:** 
Merely deleting an IAM role (A) does **not** invalidate active temporary security credentials already issued by AWS STS (they remain valid until expiration, up to 12 hours). Clicking **Revoke active sessions** in IAM attaches an inline policy that explicitly denies any request where `aws:TokenIssueTime` is earlier than the revocation timestamp, instantly invalidating active sessions. Applying a Service Control Policy (C) denying the role ARN provides an immediate organization-wide barrier that overrides all local permissions.

---

### Question 15: Cross-Region EventBridge Replication for Disaster Recovery

**Answer:** B  
**Explanation:** 
Amazon EventBridge natively supports cross-region targets. An EventBridge rule created in `us-east-1` can specify an event bus located in another AWS region (e.g. `arn:aws:events:us-west-2:123456789012:event-bus/default`) as its target. EventBridge handles cross-region transport, encryption in transit, and reliable delivery automatically without custom Lambda forwarders.

---

### Scoring Guide
- **14–15 Correct:** Mastery (Ready for DOP-C02 Incident & Event Response section)
- **12–13 Correct:** Solid Proficiency (Review specific EventBridge input transformers and FIS stop conditions)
- **$\le 11$ Correct:** Needs Reinforcement (Review `Domain_5_Incident_Event_Response/Study_Guide.md` and `Exercises.md`)
