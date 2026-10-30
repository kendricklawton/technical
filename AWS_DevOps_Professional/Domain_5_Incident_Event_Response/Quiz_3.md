# Domain 5 (Incident and Event Response) — Quiz 3: AWS Systems Manager Incident Manager

**Topic:** Response Plans, Escalation Paths, ChatOps Integration & Post-Incident Analysis  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Automated Critical Incident Mobilization via Incident Manager Response Plans

An enterprise financial trading platform requires an automated incident response workflow when a critical production outage occurs. If a CloudWatch Alarm monitoring API error rate breaches 10% for 3 minutes, the platform must:
1. Immediately mobilize the Primary On-Call SRE via SMS and phone call.
2. If unacknowledged within 5 minutes, automatically escalate to the Secondary On-Call Lead.
3. Automatically launch an AWS Systems Manager Automation diagnostic runbook.
4. Provision a dedicated incident collaboration channel in Slack with pre-populated incident metrics and runbook links.

**Which AWS service and configuration coordinates all four actions natively?**
- A. Configure an **AWS Systems Manager Incident Manager Response Plan**: Link the CloudWatch Alarm as the engagement trigger, configure an **Escalation Plan** with multi-tier contact channels and a 5-minute stage timeout, attach the SSM Automation runbook, and configure an **AWS Chatbot** Slack channel integration.
- B. Configure an Amazon EventBridge Rule that triggers an AWS Step Functions state machine that calls Twilio APIs and Slack webhooks.
- C. Configure an AWS Systems Manager OpsCenter OpsItem rule with automatic deduplication and SNS notification subscriptions.
- D. Deploy an AWS Lambda function that queries the AWS Health API and dispatches PagerDuty alerts via an HTTP REST request.

---

### Question 2: Multi-Region Resilience for Incident Management via Replication Sets

A DevOps architect is implementing AWS Systems Manager Incident Manager across a multi-region environment (`us-east-1` and `us-west-2`). In the event of a total AWS regional failure affecting `us-east-1` (the primary operational region), on-call responders must still be able to access active incident timelines, contact details, escalation plans, and runbook execution histories from `us-west-2`.

**How must Incident Manager be configured to guarantee high availability during a regional disaster?**
- A. Configure an Incident Manager **Replication Set**: Select `us-east-1` and `us-west-2`, provide dedicated Customer Managed Keys (CMKs) in each respective region for data encryption, and ensure response plans reference the replication set.
- B. Deploy AWS Backup cross-region replication to copy Incident Manager metadata daily from `us-east-1` to `us-west-2`.
- C. Create identical duplicate Response Plans in both regions and synchronize them using an Amazon EventBridge cross-region bus.
- D. Enable Amazon Route 53 Application Recovery Controller (ARC) routing controls to redirect incident console DNS endpoints.

---

### Question 3: Secure ChatOps Remediation via AWS Chatbot and IAM Permissions Boundaries

An incident response team collaborates inside a dedicated Slack channel during high-severity outages. Responders want to execute AWS Systems Manager Automation runbooks (such as restarting EC2 services or rebooting RDS instances) directly from Slack using AWS Chatbot commands (e.g., `@aws runbook start ...`). However, corporate security mandates that ChatOps commands cannot be abused to execute unauthorized administrative actions or exfiltrate data.

**How does AWS Chatbot enforce least-privilege security and governance for Slack interactions?**
- A. AWS Chatbot enforces authorization through two intersecting policy controls: the **Chatbot Channel IAM Role** (defining what actions Chatbot can perform) intersected with a **Channel Guardrail Policy** (Service Control Policy or IAM permissions boundary), ensuring commands never exceed authorized boundaries regardless of user Slack permissions.
- B. AWS Chatbot requires every Slack user to paste their AWS Secret Access Key directly into the Slack channel before running any CLI commands.
- C. ChatOps commands can only execute read-only `Describe` and `Get` actions; all mutating actions are strictly prohibited in AWS Chatbot.
- D. An AWS Lambda function intercepts Slack messages, validates Slack user email addresses against Okta via LDAP, and assumes an admin role.

---

### Question 4: Rigorous Blameless Post-Incident Reviews (PIR) via Incident Manager Analysis

Following a severe payment gateway outage, the Site Reliability Engineering director mandates a comprehensive Post-Incident Analysis (PIR). The review must include an immutable chronological timeline of the incident (including when alarms fired, when engineers joined, and which runbooks were executed), quantify business impact metrics, and convert identified root causes into trackable remediation action items.

**How does Incident Manager support this post-incident workflow natively?**
- A. Incident Manager automatically compiles the **Incident Timeline** populated with CloudWatch alarm timestamps, responder engagements, and runbook execution logs; provides standardized **Post-Incident Analysis Templates** for root-cause analysis; and enables creating trackable **Action Items** integrated with Jira or AWS DevOps tools.
- B. Responders must manually compile Slack chat logs, format them into markdown tables, and store them in an Amazon S3 Glacier vault.
- C. CloudTrail Lake generates an automated post-incident audit PDF report and emails it to all account administrators.
- D. AWS Config automatically detects configuration drift during the incident window and files GitHub pull requests to fix drift.

---

### Question 5: Granular Escalation Logic with Tiered Stage Timeouts and Contact Channels

An organization defines a strict 24/7 on-call policy:
- Stage 1: Alert the primary on-call engineer via push notification and SMS.
- Stage 2: If the incident is unacknowledged after 10 minutes, page the secondary on-call engineer via phone call.
- Stage 3: If still unacknowledged after an additional 15 minutes, notify the VP of Engineering via email and high-priority voice call.

**How is this tiered alerting path constructed within Systems Manager Incident Manager?**
- A. Define an **Escalation Plan**: Configure Stage 1 with the Primary Contact and a 10-minute duration; configure Stage 2 with the Secondary Contact and a 15-minute duration; configure Stage 3 with the Executive Contact; associate each contact with specific verified **Contact Channels** (SMS, voice, email).
- B. Configure three separate Amazon SNS topics linked to three CloudWatch Alarms with evaluation periods set to 10 and 25 minutes.
- C. Author an AWS Step Functions state machine with `Wait` states that queries an Amazon DynamoDB on-call schedule table.
- D. Create an Amazon Connect contact flow that dials engineers sequentially using an automated interactive voice response (IVR) tree.

---

### Question 6: Incident Manager vs Systems Manager OpsCenter Operational Boundaries

An enterprise IT operations team needs to establish guidelines for when to utilize **Systems Manager OpsCenter** versus **Systems Manager Incident Manager**.

**What is the architectural and operational distinction between these two AWS capabilities?**
- A. OpsCenter is designed for day-to-day operational work items (OpsItems) such as tracking software patches, SSL certificate renewals, and non-critical warnings; **Incident Manager** is purpose-built for high-severity, unplanned outages requiring real-time responder mobilization, multi-tiered escalation, ChatOps collaboration, and structured post-incident reviews.
- B. OpsCenter is for managing Amazon EC2 instances exclusively; Incident Manager is for managing AWS Lambda and container workloads.
- C. OpsCenter is a legacy tool replaced completely by Incident Manager; all operational workflows should migrate to Incident Manager.
- D. OpsCenter manages third-party external SaaS incidents; Incident Manager manages internal AWS service incidents.

---

### Question 7: Automated Runbook Execution and Parameter Passing During Incidents

When a database connection pool exhaustion incident is triggered by Incident Manager, the associated response plan automatically executes an SSM Automation runbook (`AWS-RestartService`). The runbook requires dynamic parameters, including the affected DB instance identifier and target VPC ID, which vary depending on which application environment triggered the alarm.

**How does Incident Manager pass contextual alarm parameters to the automated runbook?**
- A. Incident Manager extracts dynamic context variables (such as resource dimensions and alarm properties) directly from the triggering CloudWatch Alarm or EventBridge event payload and maps them to runbook input parameters defined in the Response Plan.
- B. The runbook must query the AWS Systems Manager Parameter Store global parameter `/incident/latest-db-id` at runtime.
- C. The on-call engineer must manually open the Systems Manager console during the outage and type the parameters into the execution prompt.
- D. The CloudWatch Alarm must execute an AWS Lambda function that edits the SSM Automation document YAML code before execution.

---

### Question 8: On-Call Schedule Rotations and Shift Handoffs in Incident Manager

An enterprise SRE team operates a follow-the-sun on-call model spanning engineering hubs in North America, Europe, and Asia. Shifts rotate every 8 hours, and engineers frequently swap shifts for vacation coverage. The platform lead needs an integrated on-call scheduling mechanism that automatically routes incident notifications to the active engineer without modifying response plan definitions.

**Which capability in Systems Manager Incident Manager natively manages shifting responder schedules?**
- A. Configure **On-Call Schedules** within Incident Manager: Define rotation patterns (daily, weekly, or custom shifts), assign engineer contacts to rotation tiers, support shift overrides for temporary swaps, and link the schedule directly to Escalation Plans.
- B. Integrate an external Google Calendar using AWS AppFlow to update IAM user tags every 8 hours.
- C. Schedule an Amazon EventBridge Scheduler task that invokes `aws ssm-incidents update-response-plan` three times daily.
- D. Maintain an Amazon DynamoDB table with employee phone numbers and query it via an AWS Lambda function.

---

### Question 9: Incident Metric Capture and Graphing in Incident Timelines

During a severe distributed denial-of-service (DDoS) incident, the incident commander needs all responders in the Incident Manager console and Slack channel to view synchronized, real-time CloudWatch metric graphs (such as ALB request count, target response time, and WAF blocked requests) embedded directly in the active incident dashboard.

**How are live metric dashboards integrated into an active Incident Manager incident?**
- A. In the Response Plan configuration, define **Incident Metrics** by specifying CloudWatch metric namespaces, metric names, and dimensions; Incident Manager automatically renders live, interactive metric graphs within the incident overview and posts snapshot links to the ChatOps channel.
- B. Responders must take screenshots of CloudWatch dashboards and manually drag and drop image files into the Incident Manager web console.
- C. Deploy an Amazon QuickSight dashboard and embed it into an iframe hosted on an EC2 instance.
- D. Configure CloudWatch Metric Streams to stream metrics to an Amazon S3 bucket for viewing via Athena.

---

## Answer Key & Explanations

### Question 1: Automated Critical Incident Mobilization via Incident Manager Response Plans
**Answer:** A  
**Explanation:** 
**AWS Systems Manager Incident Manager Response Plans** are purpose-built to orchestrate rapid, end-to-end incident mobilization:
- **Triggers:** Automatically triggered by Amazon CloudWatch Alarms or Amazon EventBridge events.
- **Escalation Plans:** Automatically engages responders via verified contact channels (SMS, voice, email) and escalates through defined stages if unacknowledged within configured timeouts (e.g., 5 minutes).
- **Runbook Execution:** Automatically triggers AWS Systems Manager Automation runbooks to gather diagnostics or initiate failover.
- **Collaboration:** Integrates with **AWS Chatbot** to automatically create or populate dedicated incident channels in Slack or Microsoft Teams with real-time links, timelines, and metric graphs.

---

### Question 2: Multi-Region Resilience for Incident Management via Replication Sets
**Answer:** A  
**Explanation:** 
Incident Manager relies on **Replication Sets** to guarantee high availability during regional outages:
- When configuring Incident Manager, you define a Replication Set comprising multiple AWS Regions (e.g., `us-east-1` and `us-west-2`).
- You provide a Customer Managed Key (CMK) in each region to encrypt incident metadata, response plans, contacts, and timelines at rest.
- Incident data is continuously replicated across all regions in the replication set.
- If an entire AWS region (such as `us-east-1`) suffers an catastrophic impairment, incident responders can seamlessly access and manage active incidents, escalation plans, and runbooks from `us-west-2`.

---

### Question 3: Secure ChatOps Remediation via AWS Chatbot and IAM Permissions Boundaries
**Answer:** A  
**Explanation:** 
AWS Chatbot provides enterprise-grade governance for ChatOps using two intersecting layers of authorization:
1. **Chatbot Channel IAM Role:** Defines the baseline permissions that can be executed from that specific chat channel.
2. **Channel Guardrail Policy:** An IAM permissions boundary or Service Control Policy that acts as an upper limit on what actions can be performed. The effective permissions are the **intersection** of the Channel IAM Role, the Guardrail Policy, and (if configured) the individual user's mapped IAM credentials.
This ensures that even if an unauthorized user attempts to issue destructive commands (e.g., deleting S3 buckets or terminating DB clusters) via Slack, the guardrail policy strictly blocks the action.

---

### Question 4: Rigorous Blameless Post-Incident Reviews (PIR) via Incident Manager Analysis
**Answer:** A  
**Explanation:** 
Systems Manager Incident Manager streamlines the Post-Incident Review (PIR) process:
- **Automated Incident Timeline:** Automatically records key chronological milestones (when alarms fired, when responders acknowledged, when runbook steps started and completed, when severity was updated).
- **PIR Templates:** Provides structured, AWS-best-practice post-incident analysis templates covering detection, timeline, root cause, impact assessment, and lessons learned.
- **Action Items:** Allows incident commanders to generate actionable remediation tasks directly within the console and assign them to engineers or export them to enterprise issue trackers (such as Jira), ensuring operational findings are resolved.

---

### Question 5: Granular Escalation Logic with Tiered Stage Timeouts and Contact Channels
**Answer:** A  
**Explanation:** 
In Systems Manager Incident Manager:
- An **Escalation Plan** defines sequential stages of engagement.
- Each stage specifies one or more **Contacts** (or on-call schedules) and a **Duration** (timeout in minutes) before the next stage triggers.
- Each Contact defines multiple verified **Contact Channels** (SMS, voice call, email) with customizable retry intervals.
- If Stage 1 contacts do not acknowledge the engagement within the specified timeout (e.g., 10 minutes), Incident Manager automatically advances to Stage 2, paging secondary contacts, and subsequently to Stage 3 for executive escalation.

---

### Question 6: Incident Manager vs Systems Manager OpsCenter Operational Boundaries
**Answer:** A  
**Explanation:** 
AWS defines clear operational boundaries between OpsCenter and Incident Manager:
- **OpsCenter:** Focuses on operational issue management (OpsItems). It aggregates non-critical operational warnings, software patch reminders, resource quota alerts, and configuration drifts that need tracking and remediation during standard business hours.
- **Incident Manager:** Specifically engineered for **high-severity, real-time crisis response**. It mobilizes human on-call responders, coordinates bridge calls and ChatOps, executes emergency automation runbooks, and facilitates formal post-incident analysis for critical service outages.

---

### Question 7: Automated Runbook Execution and Parameter Passing During Incidents
**Answer:** A  
**Explanation:** 
When an Incident Manager Response Plan triggers an SSM Automation runbook:
- Incident Manager automatically captures the context of the triggering event (e.g., the CloudWatch Alarm's dimensions like `InstanceId`, `DBInstanceIdentifier`, or `ClusterName`).
- In the Response Plan configuration, administrators map these dynamic contextual variables directly to the input parameters expected by the SSM Automation document.
- This allows generic, reusable automation runbooks (e.g., service restart, traffic rerouting) to execute against the specific faulty infrastructure without manual operator intervention.

---

### Question 8: On-Call Schedule Rotations and Shift Handoffs in Incident Manager
**Answer:** A  
**Explanation:** 
Systems Manager Incident Manager features native **On-Call Schedules**:
- Allows engineering organizations to model complex shifts (e.g., follow-the-sun rotations across time zones, weekly rotations, or custom weekend shifts).
- Responders can set up **Shift Overrides** to trade shifts or cover for colleagues during PTO without altering core escalation plans.
- When an incident occurs, the Escalation Plan dynamically queries the On-Call Schedule to determine who is currently on duty and dispatches pages accordingly.

---

### Question 9: Incident Metric Capture and Graphing in Incident Timelines
**Answer:** A  
**Explanation:** 
In Incident Manager Response Plans, administrators can declare **Incident Metrics**:
- You specify key CloudWatch metrics (e.g., Application Load Balancer target 5xx rate, backend CPU utilization, p99 latency).
- When an incident is active, Incident Manager continuously graphs these metrics directly within the incident overview screen, synchronizing the time range to the incident lifecycle.
- Responders collaborating in Slack/Teams can instantly view embedded metric snapshots, enabling shared situational awareness without navigating away to separate monitoring consoles.
