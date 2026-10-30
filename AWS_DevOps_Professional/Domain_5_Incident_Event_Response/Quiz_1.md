# Domain 5: Incident and Event Response — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 14% of Exam  
**Format:** 15 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Cross-Account Event Routing with Amazon EventBridge

An enterprise manages an AWS Organization with 120 member accounts. The central Security Operations Center (SOC) operates in Account `999999999999` (Security). Whenever an unauthorized security group modification occurs in any production member account (such as Account `111111111111`), the audit event must be routed in real time to the central security event bus for automated forensic analysis and threat intelligence enrichment.

**How should this cross-account event routing be configured with the LEAST operational overhead?**
- A. In Account `999999999999`, configure an **EventBridge Event Bus** with a resource-based policy granting `events:PutEvents` permission to the organization ID via `aws:PrincipalOrgID`. In Account `111111111111`, create an **EventBridge Rule** on the default bus matching `AuthorizeSecurityGroupIngress`, setting the target to the ARN of the security event bus, and attach an IAM role permitting `events:PutEvents` on that target.
- B. In Account `111111111111`, write an AWS Lambda function triggered by an EventBridge rule that assumes an IAM role in Account `999999999999` and executes the `events:PutEvents` API call over a VPC Peering connection.
- C. In Account `999999999999`, deploy an Amazon SQS queue with a cross-account queue policy; in Account `111111111111`, configure an EventBridge rule targeting the SQS queue directly without IAM roles.
- D. Configure an Amazon Kinesis Data Stream in Account `111111111111`, configure AWS DataSync to replicate stream data to Account `999999999999`, and parse the events using Amazon OpenSearch Service.

---

### Question 2: Chaos Engineering Safety Controls with AWS Fault Injection Service (FIS)

A platform engineering team is conducting chaos experiments on an Amazon EKS cluster to validate multi-AZ pod failover and network resilience under partial connectivity degradation. Corporate security policy mandates that any chaos experiment must execute within an automated safety boundary: if production customer-facing HTTP 5xx error rates exceed 1% or p99 latency breaches 800 milliseconds during the experiment, the chaos actions must be terminated immediately and all injected faults halted without human intervention.

**Which AWS architecture satisfies these experimental and safety requirements?**
- A. Configure an **AWS Fault Injection Service (FIS)** experiment template defining actions like `aws:eks:terminate-nodegroup-instances` and `aws:network:disrupt-connectivity`, and configure **Stop Conditions** linked to Amazon CloudWatch Alarms monitoring ALB HTTP 5xx errors and target response times.
- B. Write a Python script running inside an EC2 instance that issues AWS CLI calls to terminate worker nodes in an infinite loop, while polling CloudWatch alarm status via `DescribeAlarms` every 10 seconds.
- C. Configure AWS Systems Manager Run Command targeting all EKS worker nodes with document `AWS-RunShellScript` to reboot nodes at random intervals, setting `--max-errors 1%`.
- D. Deploy an AWS CloudFormation StackSet that continuously updates EKS node group desired capacities, relying on standard CloudFormation rollback triggers to abort updates if alarms breach.

---

### Question 3: EventBridge Archive and Replay for Downstream Failure Recovery

A critical asynchronous invoice processing architecture uses Amazon EventBridge to route order completion events to an AWS Lambda function that writes to an Amazon Aurora database. A flawed database migration locked table schemas for 4 hours, causing the Lambda function to throw unhandled exceptions and fail all invocations. While retries were attempted, all invoice events generated during the 4-hour window were exhausted and discarded from the active pipeline.

**How can the DevOps Engineer configure EventBridge to guarantee that historical events can be safely reprocessed after the database issue is resolved?**
- A. Configure an **Amazon EventBridge Archive** on the event bus with an appropriate retention period (e.g. 30 days). Once the database bug is resolved, initiate an **EventBridge Replay** specifying the archive name, start and end timestamps matching the outage window, and the target event bus.
- B. Enable AWS CloudTrail Insights on the event bus, and configure an Amazon EventBridge rule that queries the CloudTrail Event History API to resend raw API calls to the target Lambda function.
- C. Configure an Amazon S3 lifecycle rule on the event bus default artifact bucket to re-upload discarded event JSON files back into the EventBridge API via an SSM Automation runbook.
- D. Rely on Amazon EventBridge's native 7-day automatic buffer, which will automatically re-attempt event delivery to failed Lambda targets once the target function stops throwing errors.

---

### Question 4: Multi-Account Systems Manager Automation Execution with Rate Controls

A critical zero-day vulnerability in OpenSSH requires immediately revoking an insecure security group ingress rule permitting `0.0.0.0/0` on port 22 across 250 member accounts in an enterprise AWS Organization. The security team has authored an approved SSM Automation runbook `RevokeInsecureSSH` in the delegated administrator account. To avoid triggering API throttling or disrupting network connectivity abruptly, the remediation must run across at most 25 accounts concurrently, and must halt if more than 5 accounts fail execution.

**How should this remediation runbook be executed across the organization?**
- A. Execute the SSM Automation runbook from the delegated administrator account using **Multi-Account and Multi-Region Execution**, specifying the target **AWS Organizations OU ID**, the name of an existing execution IAM role deployed across member accounts, `--max-concurrency 25`, and `--max-errors 5`.
- B. Write a Python script in the management account that queries `organizations:ListAccounts`, iterates through the account IDs in a sequential `for` loop, and invokes `aws ssm start-automation-execution` for each account.
- C. Deploy an AWS CloudFormation StackSet with `SELF_MANAGED` permissions targeting all member accounts that deletes and recreates all security groups in each account.
- D. Configure an AWS Config Conformance Pack with an automatic remediation rule across all accounts, configuring the rule evaluation frequency to run on every configuration change.

---

### Question 5: EventBridge Input Transformers for Operational Alert Normalization

An operations center monitors Amazon EC2 state transitions using Amazon EventBridge (`source: aws.ec2`, `detail-type: EC2 Instance State-change Notification`). When an instance enters the `stopped` state, EventBridge sends a notification to an Amazon SNS topic subscribed to an on-call engineer email list. By default, engineers receive complex, unformatted JSON payloads. The team mandates that the notification must be formatted as a clean, single-line text string:  
`"CRITICAL: Instance i-0123456789 in us-east-1 changed state to stopped at 2026-10-07T14:30:00Z."`

**How should the event payload be transformed before delivery to the SNS topic?**
- A. Configure an **Input Transformer** on the EventBridge rule target: in **Input Paths Map**, define `{"instance":"$.detail.instance-id", "state":"$.detail.state", "region":"$.region", "time":"$.time"}`; in **Input Template**, define `"\"CRITICAL: Instance <instance> in <region> changed state to <state> at <time>.\""`.
- B. Deploy an AWS Lambda function between EventBridge and SNS that parses the JSON payload, formats the text string, and calls the `sns:Publish` API directly.
- C. Configure the Amazon SNS topic subscription with **Raw Message Delivery** enabled to strip AWS event metadata headers from the delivered message body.
- D. Enable Amazon CloudWatch Logs metric filtering on the event stream, and use metric dimension formatting to generate the custom email text string.

---

### Question 6: Proactive Infrastructure Maintenance Detection with the AWS Health API

An enterprise operates memory-intensive financial databases on Amazon EC2 Dedicated Instances. AWS occasionally schedules mandatory maintenance on underlying physical hypervisor hardware, requiring instance reboots or retirements. The operations team requires at least 10 days advance automated notice of scheduled hardware retirement events to initiate graceful connection draining, fail over databases, and launch replacement instances before AWS stops the host.

**Which monitoring architecture provides automated alerts for upcoming AWS hardware maintenance?**
- A. Create an Amazon EventBridge rule matching event source `aws.health`, event type `AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED`, and service `EC2`. Target an Amazon SNS topic and invoke an AWS Systems Manager Automation runbook to orchestrate database migration.
- B. Configure an Amazon CloudWatch Alarm on the EC2 `StatusCheckFailed_System` metric with an evaluation period of 10 days, targeting an SNS notification topic.
- C. Deploy an AWS Lambda function that queries the public AWS Service Health Dashboard RSS feed every 5 minutes and flags regional incident notices.
- D. Enable Amazon Inspector continuous host assessment to detect physical server degradation and hardware maintenance status via the SSM agent.

---

### Question 7: Systems Manager Incident Manager Automated Post-Incident Reviews (PIR)

Following a high-severity customer-facing production outage, the Vice President of Engineering requires a formal blameless Post-Incident Review (PIR). The review must automatically assemble an accurate incident timeline, aggregate all CloudWatch metrics and alarms that triggered during the event, record which on-call responders acknowledged pages, and track preventative action items directly within the engineering workflow.

**Which native capability of AWS Systems Manager supports this workflow?**
- A. Use **AWS Systems Manager Incident Manager Post-Incident Analysis (PIR)**: It automatically compiles the incident timeline, engaged on-call responders, triggered CloudWatch alarms, and executed runbooks into an editable post-incident report with native action item tracking.
- B. Query AWS CloudTrail Event History for the incident timeframe, export the API events to a CSV file in Amazon S3, and generate a timeline using Amazon Athena.
- C. Use AWS Systems Manager Session Manager terminal logs to reconstruct all shell commands executed by engineers during the troubleshooting session.
- D. Enable Amazon GuardDuty incident summarization to generate an automated executive security incident report.

---

### Question 8: Non-Destructive EC2 Incident Forensics & Security Group Quarantine

A security alert from Amazon GuardDuty indicates that an Amazon EC2 instance within an Auto Scaling Group has communicated with a known malicious crypto-mining Command-and-Control (C2) server. The incident response runbook requires that the instance must be immediately isolated from all network traffic, must **NOT be terminated** by the Auto Scaling Group, and must remain running in memory so that forensics investigators can capture a RAM memory dump and disk snapshot.

**What is the correct sequence of actions to isolate the instance safely?**
- A. Enable Instance Scale-In Protection on the instance, call `autoscaling:DetachInstances` with `ShouldDecrementDesiredCapacity: false` (to prevent the ASG from terminating it and to launch a replacement), replace its Security Groups with an **Isolation Security Group** (no ingress/egress rules), and capture an EBS snapshot and RAM dump.
- B. Immediately stop the EC2 instance using `ec2:StopInstances`, detach its primary Elastic Network Interface (ENI), and terminate the Auto Scaling Group.
- C. Terminate the instance immediately using `ec2:TerminateInstances` and restore the root volume from the most recent automated daily backup snapshot.
- D. Modify the VPC Network Access Control List (NACL) to deny all traffic to the entire subnet, and terminate all other instances in the subnet to isolate the blast radius.

---

### Question 9: Automated Remediation of Non-Compliant Resources with AWS Config

An organization uses AWS Config across all accounts. Security policy mandates that all Amazon EBS volumes must be encrypted at rest. When a developer launches an unencrypted EBS volume (`AWS::EC2::Volume`), AWS Config flags the resource as `NON_COMPLIANT` against the managed rule `encrypted-volumes`. The platform team needs non-compliant volumes to be remediated automatically by creating an encrypted snapshot, creating a replacement encrypted volume, and notifying the owner.

**How should this automated remediation be configured in AWS Config?**
- A. Under the AWS Config rule, configure **Automatic Remediation** referencing the SSM Automation document `AWS-CreateEncryptedVolumeFromUnencryptedVolume`, mapping the parameter `VolumeId` to the AWS Config `RESOURCE_ID`.
- B. Write a scheduled cron job in an EC2 instance that executes the AWS CLI command `aws ec2 delete-volume` for any volume without encryption tags.
- C. Create an IAM permissions boundary that denies `ec2:CreateVolume` unless the `--encrypted` flag is present, and configure a CloudWatch Alarm to trigger remediation.
- D. Configure an Amazon S3 Event Notification on the AWS Config delivery bucket that invokes a Lambda function to delete non-compliant EBS volumes.

---

### Question 10: Serverless Webhook Integration with EventBridge API Destinations

An enterprise platform must automatically trigger incident alerts in an external third-party Incident Management SaaS platform (e.g. PagerDuty, Opsgenie, or an internal ITSM tool) whenever a critical AWS Health or GuardDuty event occurs. The external SaaS webhook requires authentication via OAuth 2.0 Client Credentials (grant type `client_credentials`) and strictly enforces a rate limit of 15 requests per second to avoid API throttling.

**Which serverless architecture fulfills these integration requirements with the LEAST custom code?**
- A. Configure **Amazon EventBridge API Destinations**: Create an **EventBridge Connection** storing OAuth 2.0 credentials securely with AWS Secrets Manager, define an API Destination pointing to the SaaS webhook URL with an invocation rate limit of `15 requests per second`, and set it as the rule target.
- B. Deploy an AWS Lambda function triggered by EventBridge that retrieves OAuth tokens, manages an internal rate-limiting token bucket algorithm, and issues HTTPS POST requests to the SaaS endpoint.
- C. Configure an Amazon SNS topic with an HTTPS subscription pointing to the external SaaS URL, storing the OAuth client secret in the SNS subscription attributes.
- D. Launch an Amazon EC2 instance running NGINX configured as a reverse proxy with an `ngx_http_limit_req_module` rate limiter, proxying requests to the external SaaS provider.

---

### Question 11: Amazon SQS Dead Letter Queue (DLQ) Selective Message Redrive

A transient downstream API timeout caused 12,000 transaction messages to fail processing and land in an Amazon SQS Dead Letter Queue (DLQ). The 12,000 messages consist of two distinct message types identified by the message attribute `TransactionType`: `TypeA` (which can now be safely reprocessed) and `TypeB` (which require manual schema remediation). The engineering team must move only the `TypeA` messages back to the source queue immediately, while leaving `TypeB` messages in the DLQ.

**How can this selective redrive be accomplished?**
- A. Deploy an AWS Lambda function that polls the DLQ, parses the `TransactionType` attribute, calls `sqs:SendMessage` to the source queue for `TypeA` messages, and calls `sqs:DeleteMessage` on the DLQ; `TypeB` messages are left in the DLQ. (Standard SQS DLQ Redrive moves 100% of messages and does not filter by attribute).
- B. Initiate an **SQS Dead Letter Queue Redrive** from the AWS Management Console, configuring a custom SQL WHERE filter: `WHERE TransactionType = 'TypeA'`.
- C. Configure an Amazon EventBridge Pipe with an input filter on `TransactionType = 'TypeA'` targeting the source queue, leaving the DLQ unmodified.
- D. Increase the `VisibilityTimeout` of `TypeB` messages to 14 days and initiate a standard SQS DLQ redrive to the source queue.

---

### Question 12: Point-to-Point Stream Filtering and Enrichment with EventBridge Pipes

A high-scale telemetry ingestion platform streams IoT sensor state changes through Amazon DynamoDB Streams. Critical temperature anomaly events must be filtered, enriched with geolocation data from an internal microservice, and delivered to an Amazon SQS processing queue. Previously, this required maintaining a dedicated AWS Lambda function for polling, filtering, enriching, and pushing records.

**Which serverless architecture replaces this custom polling glue code with the LEAST operational overhead?**
- A. Configure an **Amazon EventBridge Pipe**: Set the Source to the DynamoDB Stream, define a **Filter Pattern** matching temperature threshold breaches, configure an **Enrichment** step referencing an AWS Lambda function or API Destination to fetch geolocation data, and set the Target to the Amazon SQS queue.
- B. Configure an Amazon Kinesis Data Firehose delivery stream reading from DynamoDB Streams, routing records through an AWS Lambda transformation before writing to SQS.
- C. Deploy an Amazon EventBridge custom event bus that subscribes to DynamoDB Streams via an EventBridge rule, using input transformers for enrichment.
- D. Configure Amazon SQS to poll DynamoDB Streams directly using native SQS stream triggers with dead-letter queue routing.

---

### Question 13: High-Resilience Regional Failover with EventBridge Global Endpoints

An international financial enterprise requires an Active-Passive event ingestion architecture spanning `us-east-1` (Primary) and `us-west-2` (Secondary). External payment partners publish business events to an Amazon EventBridge endpoint. If the primary AWS region experiences an infrastructure disruption, event ingestion must automatically fail over to `us-west-2` within minutes without requiring external partners to update endpoint URLs or reconfigure credentials.

**Which EventBridge architecture fulfills this high-availability requirement?**
- A. Deploy **Amazon EventBridge Global Endpoints**: Configure an EventBridge Global Endpoint backed by Amazon Route 53 health checks that evaluate primary bus metric alarms; external partners publish to the global endpoint hostname, and traffic fails over automatically to `us-west-2` if health checks fail.
- B. Create Route 53 Latency-Based routing records pointing to custom domain names associated with regional API Gateway endpoints fronting each regional event bus.
- C. Configure S3 Cross-Region Replication to mirror event bus archives between `us-east-1` and `us-west-2` in real time.
- D. Instruct external partners to implement client-side dual publishing, sending every event payload simultaneously to both `us-east-1` and `us-west-2` event buses.

---

### Question 14: Interactive Incident Remediation via AWS Chatbot in Slack

During a production database failover incident, an incident commander needs to query Amazon CloudWatch metrics, view current RDS cluster status, and invoke an approved AWS Systems Manager Automation runbook (`AWS-RestartDBInstance`) directly from a dedicated Slack incident triage channel without logging into the AWS Management Console.

**How should this ChatOps integration be configured securely?**
- A. Configure **AWS Chatbot** integrated with the Slack workspace and channel, subscribed to the incident Amazon SNS topic, and assign an IAM Role to the Chatbot channel with least-privilege permissions permitting `cloudwatch:GetMetricData`, `rds:DescribeDBClusters`, and `ssm:StartAutomationExecution`.
- B. Build a custom Slack bot application hosted on an EC2 instance that stores AWS administrator credentials in environment variables and executes shell commands.
- C. Configure an Amazon API Gateway endpoint that receives Slack outgoing webhooks, verifies incoming tokens, and calls AWS STS to generate long-lived root access keys.
- D. Use Amazon EventBridge API Destinations to poll Slack channel messages every 5 seconds and invoke Lambda functions to parse CLI commands.

---

### Question 15: Proactive Spot Instance Replacement with EC2 Capacity Rebalance

A data processing cluster runs on Amazon EC2 Spot Instances inside an Auto Scaling Group. Previously, when AWS issued a **Spot Instance Interruption Notice** (which provides a 2-minute warning before instance termination), the time was insufficient for workloads to checkpoint state and launch replacement capacity, leading to processing drops. The team needs to proactively launch replacement Spot instances before existing instances are interrupted.

**Which Auto Scaling configuration provides the earliest proactive replacement of Spot instances?**
- A. Enable **Capacity Rebalance** on the Auto Scaling Group: It listens for the **EC2 Instance Rebalance Recommendation** event (emitted minutes before the 2-minute interruption notice when a Spot instance is at elevated risk of interruption), proactively launches a replacement instance, warms it up, and only then terminates the at-risk instance.
- B. Set the ASG Default Cooldown period to 0 seconds and create a CloudWatch Alarm monitoring Spot market price changes.
- C. Configure an `AWS::AutoScaling::LifecycleHook` for `EC2_INSTANCE_TERMINATING` with a heartbeat timeout of 120 seconds.
- D. Switch all cluster instances from Spot instances to On-Demand instances with Savings Plans.

---

## Answer Key & Detailed Explanations

### Question 1: Cross-Account Event Routing with Amazon EventBridge
**Answer:** A  
**Explanation:**  
Amazon EventBridge natively supports cross-account event routing. In the destination security account, an event bus is created with a resource-based policy granting `events:PutEvents` permission to the organization principal (`aws:PrincipalOrgID`) or specific account IDs. In the source account, an EventBridge rule matches the desired events and specifies the destination bus ARN as the target. The rule requires an IAM role in the source account that grants `events:PutEvents` on the target bus ARN.

---

### Question 2: Chaos Engineering Safety Controls with AWS Fault Injection Service (FIS)
**Answer:** A  
**Explanation:**  
**AWS Fault Injection Service (FIS)** is AWS's managed chaos engineering service. FIS experiment templates allow injecting targeted faults (e.g. node group terminations, container CPU stress, network packet latency and drops). Crucially, FIS includes built-in **Stop Conditions** linked directly to Amazon CloudWatch Alarms. If an alarm fires (e.g. 5xx rate > 1%), FIS immediately halts the experiment and rolls back the injected fault actions automatically to safeguard production availability.

---

### Question 3: EventBridge Archive and Replay for Downstream Failure Recovery
**Answer:** A  
**Explanation:**  
**Amazon EventBridge Archive and Replay** allows capturing all events sent to an event bus into a durable archive with a configurable retention policy. When downstream processing fails (e.g. due to database schema locking or Lambda bugs), past events can be safely reprocessed by initiating a **Replay**. The replay delivers the exact historical event payloads over a specified time window back to the target bus, ensuring zero lost events without custom backup scripts.

---

### Question 4: Multi-Account Systems Manager Automation Execution with Rate Controls
**Answer:** A  
**Explanation:**  
AWS Systems Manager Automation supports **Multi-Account and Multi-Region Execution**. When executed from an AWS Organizations management or delegated administrator account, administrators can target entire Organizational Units (OUs), specify an execution IAM role deployed in member accounts, and enforce granular rate controls using `--max-concurrency` (e.g. 25 accounts) and `--max-errors` (e.g. 5 failures before halting).

---

### Question 5: EventBridge Input Transformers for Operational Alert Normalization
**Answer:** A  
**Explanation:**  
**EventBridge Input Transformers** allow restructuring and formatting event payloads directly at the rule target level without custom Lambda code. The **Input Paths Map** extracts specific JSON attributes using JSONPath syntax (e.g. `{"instance": "$.detail.instance-id"}`). The **Input Template** defines the desired string structure and interpolates variables using `<variable>` syntax, transforming verbose JSON into clean, human-readable notifications.

---

### Question 6: Proactive Infrastructure Maintenance Detection with the AWS Health API
**Answer:** A  
**Explanation:**  
AWS Health publishes proactive maintenance notifications to **Amazon EventBridge** under source `aws.health`. Scheduled hardware retirement events arrive as `AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED`, typically 10–14 days prior to hardware retirement. Capturing this event in EventBridge allows automated runbooks to launch replacement instances, drain connections, and fail over databases well in advance of the deadline.

---

### Question 7: Systems Manager Incident Manager Automated Post-Incident Reviews (PIR)
**Answer:** A  
**Explanation:**  
**AWS Systems Manager Incident Manager Post-Incident Analysis (PIR)** provides a structured, automated post-mortem workflow. It automatically compiles key incident artifacts: timeline of alarm state changes, engaged responders, chat transcripts, and executed automation runbooks. It allows engineering teams to identify root causes and track preventative action items directly within the AWS ecosystem.

---

### Question 8: Non-Destructive EC2 Incident Forensics & Security Group Quarantine
**Answer:** A  
**Explanation:**  
To isolate a compromised instance in an Auto Scaling Group without destroying forensic evidence: 1) Enable Instance Scale-In Protection; 2) Call `autoscaling:DetachInstances` with `ShouldDecrementDesiredCapacity: false` (this separates the instance from the ASG and triggers the ASG to launch a clean replacement instance to maintain service capacity); 3) Replace existing security groups with an **Isolation Security Group** (no ingress/egress rules) to sever C2 communications; and 4) Snapshot the EBS volumes and take a memory dump while the host remains powered on.

---

### Question 9: Automated Remediation of Non-Compliant Resources with AWS Config
**Answer:** A  
**Explanation:**  
AWS Config supports **Automatic Remediation** using AWS Systems Manager Automation runbooks. When a resource is flagged as non-compliant against a rule (e.g. `encrypted-volumes`), AWS Config automatically invokes the configured runbook (e.g. `AWS-CreateEncryptedVolumeFromUnencryptedVolume`), passing the non-compliant resource identifier via the `RESOURCE_ID` parameter mapping to execute remediation without human intervention.

---

### Question 10: Serverless Webhook Integration with EventBridge API Destinations
**Answer:** A  
**Explanation:**  
**Amazon EventBridge API Destinations** allow EventBridge to send events directly to any external HTTP/REST endpoint. An **EventBridge Connection** securely manages authentication credentials (including OAuth 2.0 Client Credentials, API keys, and Basic auth) integrated with AWS Secrets Manager. API Destinations also include built-in **Rate Limiting** (invocations per second), protecting external SaaS APIs from being overwhelmed during event storms with zero Lambda proxy code.

---

### Question 11: Amazon SQS Dead Letter Queue (DLQ) Selective Message Redrive
**Answer:** A  
**Explanation:**  
The native AWS SQS DLQ Redrive feature moves all messages back to the source queue (or a specified custom queue) in bulk; it does not support filtering messages by attribute or body content. To achieve selective redrive where only `TypeA` messages are returned to the source queue while `TypeB` messages remain in the DLQ, an AWS Lambda function or worker application must read messages, evaluate the `TransactionType` attribute, call `SendMessage` for `TypeA`, and delete them from the DLQ.

---

### Question 12: Point-to-Point Stream Filtering and Enrichment with EventBridge Pipes
**Answer:** A  
**Explanation:**  
**Amazon EventBridge Pipes** provides direct, point-to-point serverless integration between event sources (such as DynamoDB Streams, Kinesis, or SQS) and targets (SQS, Step Functions, Lambda). Pipes includes built-in declarative **Event Filtering** (evaluating payload attributes) and an optional **Enrichment** step (invoking a Lambda function or API Destination to augment payloads) before delivering records to the target, completely eliminating boilerplate poller Lambda functions.

---

### Question 13: High-Resilience Regional Failover with EventBridge Global Endpoints
**Answer:** A  
**Explanation:**  
**Amazon EventBridge Global Endpoints** provide a single, highly available global hostname that routes incoming events across primary and secondary AWS regions. Backed by Amazon Route 53 health checks monitoring primary event bus metrics, Global Endpoints automatically shift event ingestion to the secondary region if the primary region encounters disruption, ensuring continuous ingestion without partner reconfiguration.

---

### Question 14: Interactive Incident Remediation via AWS Chatbot in Slack
**Answer:** A  
**Explanation:**  
**AWS Chatbot** provides secure, interactive ChatOps integration with Slack and Microsoft Teams. By mapping IAM roles with least-privilege permissions to specific Slack channels, incident responders can run AWS CLI read commands (`describe`, `get`) and trigger approved AWS Systems Manager Automation runbooks (`AWS-RestartDBInstance`) directly from chat, facilitating rapid collaborative incident response.

---

### Question 15: Proactive Spot Instance Replacement with EC2 Capacity Rebalance
**Answer:** A  
**Explanation:**  
Amazon EC2 Auto Scaling **Capacity Rebalance** leverages the **EC2 Instance Rebalance Recommendation** signal, which is emitted when an instance is at elevated risk of interruption, significantly earlier than the standard 2-minute Spot Interruption Notice. Capacity Rebalance proactively launches a replacement Spot instance, waits for it to pass health checks and warm up, and only then terminates the at-risk instance, minimizing workload disruption.
