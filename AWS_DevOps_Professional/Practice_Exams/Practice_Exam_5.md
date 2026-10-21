# AWS DevOps Engineer - Professional (DOP-C02) Practice Exam 5: Event-Driven Incident Response & Chaos Engineering

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes (≈5 minutes per question; the real exam allows ≈2.4)  
**Passing Target:** 12 / 15 (80%)  
**Domains:** D1 ×3 · D2 ×3 · D3 ×2 · D4 ×2 · D5 ×3 · D6 ×2

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Answer all 15 before opening the answer key at the end.

---

## Question 1 (Domain 5: Incident and Event Response)

GuardDuty reports that an EC2 instance in a production Auto Scaling group is communicating with a known command-and-control server over an established TLS session. The incident response plan requires the instance to be isolated with its memory preserved for forensics — no stop, reboot, or termination — its EBS volumes snapshotted, and no other instance in its subnet affected.

An EventBridge rule already routes high-severity GuardDuty findings to a Step Functions workflow. The instance's security group allows all outbound traffic, the subnet hosts 30 healthy instances, and forensic tooling runs from a separate account.

**Which isolation design achieves the FASTEST containment of the existing command-and-control session?**

- A. Have the workflow replace the instance's security groups with an isolation group that has no rules, detach the instance from the Auto Scaling group with replacement, deregister it from its target groups, and snapshot its volumes.
- B. Have the workflow add inbound and outbound deny entries for the instance's IP address to the subnet's network ACL, then replace its security groups with a no-rule isolation group, detach it from the Auto Scaling group, and snapshot its volumes.
- C. Have the workflow replace the security groups with a no-rule isolation group, set the network interface's established TCP connection-tracking timeout to its minimum, detach the instance from the Auto Scaling group, and snapshot its volumes.
- D. Have the workflow snapshot the volumes and copy the snapshots to the forensic account first, then add network ACL deny entries for the instance's IP address and replace its security groups with a no-rule isolation group.

---

## Question 2 (Domain 1: SDLC Automation)

A company's CodePipeline (V2) pipeline deploys a customer-facing service to production about 15 times a day using CodeDeploy. During incidents, the incident commander sets a CloudWatch metric alarm, `incident-active`, to ALARM through a chat command, and on-call engineers then disable the transition into the production stage by hand — a step they forgot twice last quarter, deploying changes into ongoing incidents.

The company wants production deployments blocked automatically whenever `incident-active` is in ALARM; executions that arrive during an incident may fail and be rerun later. Staging deployments must continue during incidents.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Add a Lambda invoke action at the start of the production stage that reads the alarm state with `DescribeAlarms` and fails the action when the alarm is in ALARM, and configure the stage to retry until the alarm clears.
- B. Add an entry condition to the production stage with a CloudWatch alarm rule for `incident-active`, so executions can't enter the stage while the alarm is in ALARM.
- C. Create an EventBridge rule on `incident-active` state changes that invokes a Lambda function, which disables the production stage's inbound transition on ALARM and enables it again on OK.
- D. Add a manual approval action at the start of the production stage, and update the incident runbook so approvers reject any request while `incident-active` is in ALARM.

---

## Question 3 (Domain 2: Configuration Management and IaC)

A company with 150 AWS accounts must ensure that every S3 bucket has versioning enabled and that any noncompliant bucket is corrected automatically within an hour, including in accounts created later. Results must be visible centrally for auditors.

AWS Config is enabled in all accounts and Regions, with an aggregator in the security account, which is also the Config delegated administrator. The company prefers AWS managed rules and runbooks over custom code.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use a StackSet to deploy the `s3-bucket-versioning-enabled` managed rule to every account and Region, plus an EventBridge rule in each account that invokes a Lambda function to enable versioning on noncompliant buckets.
- B. Enable the corresponding AWS Security Hub CSPM control in every account, and add an EventBridge rule on failed findings that invokes a Lambda function to enable versioning on the bucket.
- C. From the delegated administrator, deploy an organization conformance pack with the `s3-bucket-versioning-enabled` rule and automatic remediation through the `AWS-ConfigureS3BucketVersioning` runbook, reporting compliance through the existing aggregator for auditors.
- D. Run a Lambda function every hour in the security account that assumes a role in every account and Region, lists buckets, enables versioning where needed, and writes results to an S3 bucket for auditors.

---

## Question 4 (Domain 3: Resilient Cloud Solutions)

A Lambda function consumes order messages from an SQS standard queue whose redrive policy moves messages to a dead-letter queue after five receives. A bug in a new release sent 40,000 messages to the DLQ over two hours before the team rolled back.

The fix is now deployed. The team must move the messages back to the source queue so the function processes them, at a controlled rate that won't overwhelm a downstream API. Order processing is idempotent, and message order doesn't matter.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Write a Lambda function that receives messages from the DLQ in batches, sends them to the source queue, and deletes them from the DLQ, and invoke it repeatedly with EventBridge Scheduler until the DLQ is empty.
- B. Temporarily add the DLQ as a second event source for the consumer function with a low maximum concurrency, and remove that event source mapping when the DLQ is empty.
- C. Export the DLQ's messages to an S3 bucket with a script, then replay them into the source queue with a second script that throttles sends to a fixed number per second.
- D. Start a dead-letter queue redrive from the console or with `StartMessageMoveTask`, setting a maximum velocity in messages per second that the downstream API can absorb, with the source queue as the destination.

---

## Question 5 (Domain 4: Monitoring and Logging)

A company has 120 service alarms that page on-call through SNS. When the shared payments gateway fails, most service alarms fire within minutes, producing dozens of pages for one root cause. An alarm on the gateway's health, `payments-gateway-down`, already exists and usually enters ALARM a minute or two after the first service alarms.

The company wants service alarms to stop paging while `payments-gateway-down` is in ALARM — including the window just before it fires — and to page normally otherwise. Notifications also go to Slack through Amazon Q Developer in chat applications.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create a Lambda function triggered by `payments-gateway-down` state changes that disables actions on the 120 service alarms when it enters ALARM and re-enables them when it returns to OK, after a short delay.
- B. Route all service alarm notifications through an EventBridge rule to a Lambda function that checks the state of `payments-gateway-down` and publishes the page to SNS only when it is OK.
- C. Wrap each service alarm in a composite alarm that pages instead, with `payments-gateway-down` as its actions suppressor and a wait period covering the minutes before it fires.
- D. Route service alarm notifications to a Step Functions workflow that waits 3 minutes, checks `payments-gateway-down`, and publishes the page to SNS only if the gateway alarm is still OK.

---

## Question 6 (Domain 6: Security and Compliance)

A security team must run a quarterly least-privilege review across 90 accounts. It must identify IAM roles unused for 90 days and permissions that roles have but don't use, and give each owning team a recommended reduced policy to review. Findings must also be routed automatically to each team's ticket queue.

The company has a delegated administrator account for IAM Access Analyzer, and a central EventBridge bus already integrates with the ticketing system. Teams own their roles and must approve all policy changes.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Generate IAM credential reports and service last-accessed data in every account with a scheduled Lambda function, compute unused roles and services, and write proposed policies with their JSON to tickets through the central bus.
- B. Create an organization-level unused access analyzer with a 90-day window in the delegated administrator account, use its recommended policies for unused-permission findings, and route findings to the central bus.
- C. Query the organization trail in S3 with Athena to find the actions each role used in the last 90 days, generate reduced policies with a script, and open tickets through the central bus.
- D. Run IAM Access Analyzer policy generation for each role in each account from the console every quarter, based on its CloudTrail activity, and attach the generated policies to tickets by hand.

---

## Question 7 (Domain 5: Incident and Event Response)

A company publishes 3 million events a day to a custom EventBridge bus that has an archive retaining all events for 30 days. A bug in one consumer's Lambda function made it silently discard `OrderUpdated` events for 6 hours before the fix was deployed. The consumer writes results to DynamoDB idempotently.

The team must reprocess that consumer's events from the 6-hour window. Twelve other rules on the bus route the same events to consumers that are not idempotent and must not receive duplicates.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Ask the publishing teams to re-publish the window's `OrderUpdated` events from their outbox tables with a `replay` attribute, and add a condition to the twelve other rules' event patterns that excludes events with that attribute.
- B. Start a replay from the archive for the 6-hour window to the bus, restricting the replay to the fixed consumer's rule so that no other rule receives the replayed events.
- C. Write a backfill script that reconstructs the window's `OrderUpdated` events from the order database and invokes the consumer's Lambda function directly with each reconstructed event.
- D. Query a catch-all rule's CloudWatch Logs log group for the window with Logs Insights, export the results, and re-send them with `PutEvents` to a temporary bus that has a copy of only the consumer's rule.

---

## Question 8 (Domain 1: SDLC Automation)

A company deploys a web application with CodeDeploy in-place deployments to an Auto Scaling group behind an Application Load Balancer, and deployments take about 25 minutes. When the group scales out during a deployment, the new instances receive the previous revision, and the fleet runs mixed versions until someone notices and redeploys.

Scaling must stay enabled during deployments because traffic spikes are common. The CodeDeploy deployment group and the Auto Scaling group are managed in CloudFormation.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Enable the deployment group's setting that automatically updates outdated instances, so CodeDeploy deploys the new revision to instances launched during a deployment once it finishes, without suspending scaling.
- B. Add an Auto Scaling launch lifecycle hook with an EventBridge rule that invokes a Lambda function, which checks each new instance's revision and starts a CodeDeploy deployment targeting it if it's outdated.
- C. Bake each application revision into a new AMI with EC2 Image Builder and update the launch template, so instances launched during a deployment already run the new revision.
- D. Create an EventBridge rule for CodeDeploy deployment success events that invokes a Lambda function to compare instance revisions and redeploy the latest revision to any outdated instances.

---

## Question 9 (Domain 2: Configuration Management and IaC)

A company stores 400 partner API tokens as SecureString parameters in Parameter Store, and each token expires 180 days after issuance. The security team wants an alert 14 days before each token expires, and an alert if any token hasn't been changed for 90 days so that stale tokens are investigated.

Alerts must flow into the team's existing EventBridge-based routing. The parameters are in the Standard tier, and partner teams rotate tokens through the console.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Write a Lambda function that runs daily, reads each parameter's metadata, compares `LastModifiedDate` with the expiry and 90-day thresholds, and sends alert events to EventBridge.
- B. Create an AWS Config custom rule with a periodic trigger that evaluates each parameter's last-modified time and an expiry tag, and route its noncompliance events to EventBridge.
- C. Move the parameters to the Advanced tier and attach parameter policies for expiration, a 14-day expiration notification, and a 90-day no-change notification, which emit EventBridge events.
- D. Migrate the tokens to Secrets Manager, write a custom rotation Lambda function for each partner that alerts instead of rotating, and schedule rotation checks to send events 14 days before expiry.

---

## Question 10 (Domain 3: Resilient Cloud Solutions)

A company runs 40 business-critical VMware virtual machines in its data center. Its DR requirements are an RPO of seconds and an RTO under 1 hour in AWS, with failback to the data center after recovery. The applications are commercial off-the-shelf products with no built-in replication, running on Windows and Linux.

The company doesn't want to pay for idle compute in AWS, and recovery drills must be possible without affecting production. It already has an AWS Direct Connect connection to us-east-1.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Use AWS Elastic Disaster Recovery to replicate the servers' disks continuously into a low-cost staging area, launch recovery instances for drills and disasters, and use its failback tooling to return to VMware.
- B. Replicate each server with a third-party block replication tool to an EC2 instance that runs continuously at a scaled-down size in us-east-1, and resize the instances during a disaster.
- C. Keep a warm standby in AWS by running every application on right-sized EC2 instances that receive OS-level replication from the data center, and switch DNS during a disaster.
- D. Run a small EC2 replication server per application that receives block-level replication into attached EBS volumes with an open-source tool, and launch recovery instances from snapshots of those volumes during a disaster.

---

## Question 11 (Domain 4: Monitoring and Logging)

During incidents, SREs need to see within a minute which client IDs and API paths generate the most 5XX responses for an API whose JSON access logs go to CloudWatch Logs — about 200 GB a day. They also want an alarm when any single client generates more than 1,000 errors in 5 minutes.

The log group already has a 30-day retention policy, and the team uses CloudWatch dashboards for incident response. There are hundreds of thousands of client IDs.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Run a CloudWatch Logs Insights query every minute from a Lambda function, write the top contributors to a DynamoDB table shown on a dashboard, and publish a custom metric per client ID for alarming.
- B. Stream the log group to Amazon OpenSearch Service through a subscription filter and Amazon Data Firehose, build top-N visualizations there, and configure OpenSearch alerting monitors for the per-client threshold.
- C. Change the API to emit embedded metric format records with client ID and path as dimensions, and create alarms on the resulting custom metrics for each client ID.
- D. Create a Contributor Insights rule on the log group keyed on client ID and path for 5XX responses, add its graph to the incident dashboard, and alarm on its maximum contributor value.

---

## Question 12 (Domain 5: Incident and Event Response)

A company runs a payments service on Amazon ECS on EC2 and an Aurora cluster across three Availability Zones. Leadership requires a monthly game day that simulates an AZ power interruption affecting compute and the database, stops automatically if customer-facing error rates exceed a threshold, and produces a record of what was run.

CloudWatch alarms on customer error rates already exist. The SRE team has no chaos engineering tooling today and wants to avoid writing and maintaining fault-injection logic.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Write SSM Automation runbooks that stop instances and fail over Aurora in one AZ, add an alarm-driven EventBridge rule that cancels the runbook execution, and record each run in a wiki.
- B. During each game day, have engineers apply network ACL rules that block traffic to one AZ's subnets and reboot the Aurora writer, watching dashboards to decide manually when to roll back.
- C. Create an AWS Fault Injection Service experiment template from the AZ power interruption scenario in the scenario library, targeting the service, with the existing alarms as stop conditions.
- D. Build an AWS FIS experiment template action by action — stopping instances, disrupting network connectivity, and failing over Aurora in one AZ — and add the existing alarms as stop conditions.

---

## Question 13 (Domain 2: Configuration Management and IaC)

A company's CloudFormation stacks use a Lambda-backed custom resource to register services with a third-party CMDB. When the CMDB API hangs, the function times out after 5 minutes without responding, and the stack waits up to an hour before the operation fails and rolls back. This happens during both stack creates and updates.

The team wants failed custom resource operations to fail the stack operation within about 6 minutes, without adding infrastructure to maintain. Another team owns the function code, and it changes slowly.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Set the function's asynchronous retry attempts to zero and configure an on-failure destination that triggers a second Lambda function, which sends a FAILED response to the custom resource's presigned URL.
- B. Replace the Lambda target with an SNS-backed custom resource whose subscriber starts a Step Functions workflow that calls the CMDB with a timeout and always sends a response to CloudFormation.
- C. Add a CloudWatch alarm on the function's timeout errors that notifies the on-call engineer, who follows a runbook to send a FAILED response to the stack's presigned URL with curl.
- D. Set the custom resource's `ServiceTimeout` property to 360 seconds in each template, so CloudFormation fails the operation and rolls back if no response arrives in time, without changing the function code.

---

## Question 14 (Domain 6: Security and Compliance)

A company with 90 AWS accounts wants an automated response whenever AWS detects that one of its IAM access keys has been publicly exposed: deactivate the key, notify the owner identified by an `owner` tag on the IAM user, and open a ticket — within minutes and with no human action.

A central EventBridge bus in the security account already receives events forwarded from every account and starts SSM Automation runbooks that assume a remediation role in member accounts. The company knows AWS also attaches a quarantine policy to exposed keys, but it must still deactivate them.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use a StackSet to add a rule in every account that forwards AWS Health `AWS_RISK_CREDENTIALS_EXPOSED` events to the central bus, where a rule starts a runbook that deactivates the key, notifies the owner, and opens a ticket.
- B. Subscribe a mailbox to each account's root email address, and process the exposure notification emails with an Amazon SES receipt rule and a Lambda function that parses them, deactivates the key, and opens a ticket.
- C. Deploy a Lambda function to every account with a StackSet, triggered by a local EventBridge rule on the Health event, that deactivates the key, notifies the owner through SNS, and opens a ticket through the ticketing API.
- D. Run a Lambda function every 15 minutes in the security account that searches public code repositories for the company's access key IDs and deactivates any matches through a cross-account role.

---

## Question 15 (Domain 1: SDLC Automation)

A company deploys an ECS service on Fargate with rolling updates from a CloudFormation template that a CodePipeline pipeline deploys. The deployment circuit breaker is enabled with rollback and catches tasks that fail to start, but two recent releases started successfully yet doubled p99 latency, and stayed in production for an hour before a manual rollback.

A CloudWatch alarm on the service's p99 latency already exists. The company wants deployments to roll back automatically when that alarm fires during a deployment, while keeping rolling updates and the template-based workflow.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Add the latency alarm to the service's deployment configuration alarms with rollback enabled, alongside the circuit breaker, so ECS rolls back if the alarm fires during a deployment.
- B. Add an `onSuccess` condition to the pipeline's deploy stage with a CloudWatch alarm rule for the latency alarm and a rollback result, so the pipeline redeploys the previous template if the alarm fires.
- C. Create an EventBridge rule on the latency alarm's ALARM state that invokes a Lambda function, which updates the service to the previous task definition revision recorded in a parameter.
- D. Migrate the service to CodeDeploy blue/green deployments with the latency alarm configured for automatic rollback in the deployment group, generating the AppSpec file in the pipeline.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | B | Domain 5: Incident and Event Response | FASTEST containment |
| 2 | B | Domain 1: SDLC Automation | LEAST operational overhead |
| 3 | C | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 4 | D | Domain 3: Resilient Cloud Solutions | LEAST operational overhead |
| 5 | C | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 6 | B | Domain 6: Security and Compliance | LEAST operational overhead |
| 7 | B | Domain 5: Incident and Event Response | LEAST operational overhead |
| 8 | A | Domain 1: SDLC Automation | LEAST operational overhead |
| 9 | C | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 10 | A | Domain 3: Resilient Cloud Solutions | MOST cost-effective |
| 11 | D | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 12 | C | Domain 5: Incident and Event Response | LEAST operational overhead |
| 13 | D | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 14 | A | Domain 6: Security and Compliance | LEAST operational overhead |
| 15 | A | Domain 1: SDLC Automation | LEAST operational overhead |

---

### Question 1

**Answer:** B  
**Constraint:** FASTEST containment · **Services:** Amazon GuardDuty, Amazon EventBridge, AWS Step Functions

**Socratic walkthrough**
1. *Does changing a security group cut an established, tracked connection?* — No. Security group changes don't interrupt existing tracked connections.
2. *What filters traffic statelessly and applies immediately to every packet?* — A network ACL.
3. *Can a network ACL isolate one instance without affecting neighbors?* — Yes, with deny entries scoped to that instance's IP address.

**Why B is correct:** Network ACL deny entries for the instance's IP address drop the existing session at once, and the isolation security group then blocks new connections at the instance level. Memory is untouched, other instances are unaffected, and snapshots proceed in parallel.

**Why the others fail the constraint:**
- **A.** Isolates new traffic, but the existing tracked session can continue.
- **C.** A shorter tracking timeout only ends idle connections; an active beacon keeps its session alive.
- **D.** Correct controls, but waiting for snapshot copies before cutting the session delays containment.

**Exam reflex:** Security groups are stateful, so existing flows survive rule changes; use network ACLs for an immediate cut.

---

### Question 2

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** AWS CodePipeline (stage conditions), Amazon CloudWatch alarms, AWS CodeDeploy

**Socratic walkthrough**
1. *Can a V2 pipeline evaluate an alarm before a stage starts?* — Yes. An entry condition with a CloudWatch alarm rule does.
2. *Does it affect other stages?* — No. It's configured on the production stage only.
3. *What custom code does it need?* — None.

**Why B is correct:** The entry condition checks `incident-active` whenever an execution tries to enter production and blocks it while the alarm is in ALARM. Staging is unaffected, and nothing needs to be built or remembered during incidents.

**Why the others fail the constraint:**
- **A.** Works, but a custom gate function with retry logic does what conditions already do.
- **C.** Works, but transition toggling depends on custom code reacting correctly to every state change.
- **D.** Works, but relies on the same human step that already failed twice.

**Exam reflex:** Gate CodePipeline V2 stages on alarms with conditions, not glue code.

---

### Question 3

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS Config (organization conformance packs), AWS Systems Manager Automation, AWS Organizations

**Socratic walkthrough**
1. *Which AWS managed pieces exist?* — The `s3-bucket-versioning-enabled` rule and the `AWS-ConfigureS3BucketVersioning` runbook.
2. *Can one artifact deploy a rule and its remediation to every account, including new ones?* — Yes. An organization conformance pack can include remediation configurations.
3. *Where do auditors look?* — At the existing aggregator.

**Why C is correct:** An organization conformance pack deployed from the delegated administrator places the managed rule and automatic remediation in every member account and Region, including future accounts, and compliance flows to the aggregator. No code is involved.

**Why the others fail the constraint:**
- **A.** Works, but adds per-account Lambda functions and event wiring.
- **B.** Works, but uses custom Lambda remediation and a second service for a Config-native problem.
- **D.** Works, but is a custom cross-account scanner with its own reporting.

**Exam reflex:** For org-wide detection plus auto-remediation, use organization conformance packs with remediation.

---

### Question 4

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** Amazon SQS (DLQ redrive), AWS Lambda, Amazon CloudWatch

**Socratic walkthrough**
1. *Does SQS have a native way to move messages from a DLQ back to its source?* — Yes. DLQ redrive, through the console or the `StartMessageMoveTask` API.
2. *Can it be throttled?* — Yes. Set a maximum number of messages per second.
3. *What would you otherwise build?* — A mover, a scheduler, or temporary event source mappings.

**Why D is correct:** A message move task redrives the 40,000 messages to the source queue at the velocity you choose, so the downstream API is protected, and the existing consumer processes them normally.

**Why the others fail the constraint:**
- **A.** Works, but a custom mover function and schedule duplicate the built-in redrive.
- **B.** Works, but needs temporary event source mapping changes and careful cleanup.
- **C.** Works, but two scripts plus S3 staging is the most manual option.

**Exam reflex:** To reprocess DLQ messages, use SQS DLQ redrive with a velocity limit.

---

### Question 5

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** Amazon CloudWatch (composite alarms), Amazon SNS, Amazon Q Developer in chat applications

**Socratic walkthrough**
1. *Is there a native way to silence alarm actions while another alarm is in ALARM?* — Yes. A composite alarm with an actions suppressor.
2. *How do you handle child alarms that fire before the gateway alarm?* — With the suppressor wait period: the composite waits for the suppressor before acting.
3. *Does it need code?* — No.

**Why C is correct:** Each composite alarm pages in place of its service alarm unless `payments-gateway-down` is in ALARM. The wait period covers the lag before the gateway alarm fires, so pages are suppressed during the race too.

**Why the others fail the constraint:**
- **A.** Works, but mass-toggling alarm actions with custom code is fragile.
- **B.** Works, but every page now depends on a custom function.
- **D.** Works, but a custom delay-and-check workflow sits in the paging path.

**Exam reflex:** To suppress dependent alerts, use a composite alarm with an actions suppressor and wait periods.

---

### Question 6

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** IAM Access Analyzer (unused access), Amazon EventBridge, AWS Organizations

**Socratic walkthrough**
1. *Which feature finds unused roles and unused permissions organization-wide?* — IAM Access Analyzer's unused access analyzer.
2. *Does it suggest reduced policies?* — Yes. It provides recommended policy changes for unused-permission findings.
3. *How do findings reach tickets?* — Access Analyzer publishes findings to EventBridge, which forwards them to the existing bus.

**Why B is correct:** One organization-level analyzer continuously evaluates every account against a 90-day window, produces recommendations teams can review, and emits events that the existing ticket integration consumes, with no data processing code.

**Why the others fail the constraint:**
- **A.** Works, but means custom data collection and policy synthesis across 90 accounts.
- **C.** Works, but log mining and policy generation scripts are custom and costly to maintain.
- **D.** Works, but per-role manual generation every quarter is heavy toil.

**Exam reflex:** For least-privilege reviews across the organization, use the unused access analyzer.

---

### Question 7

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** Amazon EventBridge (archive and replay), AWS Lambda, Amazon DynamoDB

**Socratic walkthrough**
1. *Are the lost events still available?* — Yes. The archive retains 30 days of events.
2. *Can a replay target specific rules?* — Yes. A replay's destination can be limited to particular rules on the bus.
3. *Does any producer or consumer need to change?* — No.

**Why B is correct:** Replaying the 6-hour window from the archive, restricted to the fixed consumer's rule, sends the events only to that rule. The idempotent consumer reprocesses them, and the twelve other rules never see duplicates.

**Why the others fail the constraint:**
- **A.** Works, but needs many publishing teams plus edits to twelve rules.
- **C.** Works, but event reconstruction is custom and error-prone.
- **D.** Works, but means exporting, transforming, and re-publishing through a temporary bus — all custom.

**Exam reflex:** To reprocess events for one consumer, replay the archive scoped to that rule.

---

### Question 8

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS CodeDeploy, Amazon EC2 Auto Scaling, Elastic Load Balancing

**Socratic walkthrough**
1. *Why do new instances get the old revision?* — Scale-out during a deployment installs the last successful revision.
2. *Does CodeDeploy know which instances are outdated?* — Yes. It tracks them.
3. *Is there a setting to fix them automatically?* — Yes. The deployment group's outdated-instances option triggers a follow-up deployment.

**Why A is correct:** With automatic updating of outdated instances enabled, CodeDeploy deploys the new revision to instances launched during the deployment once it finishes. That's one property in the CloudFormation deployment group, with no code or new pipeline steps.

**Why the others fail the constraint:**
- **B.** Works, but a lifecycle hook, rule, and function rebuild a native feature.
- **C.** Works, but an AMI per revision changes the whole release process.
- **D.** Works, but custom revision comparison and redeployment logic is needed.

**Exam reflex:** For mixed versions after scale-out during CodeDeploy deployments, enable automatic updates for outdated instances.

---

### Question 9

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS Systems Manager Parameter Store (parameter policies), Amazon EventBridge, Parameter Store Advanced tier

**Socratic walkthrough**
1. *Can Parameter Store track expiration and staleness itself?* — Yes, through parameter policies: Expiration, ExpirationNotification, and NoChangeNotification.
2. *What do they need?* — The Advanced tier.
3. *Where do notifications go?* — EventBridge, which feeds the existing routing.

**Why C is correct:** Parameter policies attach the expiry date, a 14-day warning, and a 90-day no-change alert to each parameter. Systems Manager evaluates them and emits EventBridge events, so there's no scheduler or code.

**Why the others fail the constraint:**
- **A.** Works, but is a custom daily scanner.
- **B.** Works, but is a custom Config rule plus tag discipline for expiry dates.
- **D.** Works, but is a migration plus a custom function per partner.

**Exam reflex:** For expiration and staleness alerts on parameters, use Advanced-tier parameter policies.

---

### Question 10

**Answer:** A  
**Constraint:** MOST cost-effective · **Services:** AWS Elastic Disaster Recovery, Amazon EC2, Amazon EBS

**Socratic walkthrough**
1. *Which options keep application compute running in AWS 24×7?* — B and C.
2. *How does Elastic Disaster Recovery keep RPO in seconds without running replicas?* — Continuous block-level replication into a low-cost staging area with lightweight replication servers.
3. *Does it support drills and failback?* — Yes. Drills launch recovery instances without affecting replication, and failback replicates back to the source environment.

**Why A is correct:** Elastic Disaster Recovery charges mainly for the staging area — cheap storage plus shared, lightweight replication servers. Full-size recovery instances run only during drills or disasters, while still meeting an RPO of seconds and an RTO under an hour.

**Why the others fail the constraint:**
- **B.** Works, but pays for 40 scaled-down instances continuously.
- **C.** Works, but a full warm standby is the most expensive option.
- **D.** Works, but 40 per-application replication servers and do-it-yourself tooling cost more than the managed staging area.

**Exam reflex:** For on-premises-to-AWS DR with seconds of RPO and no idle compute, use Elastic Disaster Recovery.

---

### Question 11

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** CloudWatch Contributor Insights, Amazon CloudWatch Logs, CloudWatch alarms

**Socratic walkthrough**
1. *What's the problem shape?* — Top-N contributors over high-cardinality keys, in near real time.
2. *Which CloudWatch feature computes exactly that from log events?* — Contributor Insights rules.
3. *Can you alarm on it?* — Yes, with `INSIGHT_RULE_METRIC` on the rule's maximum contributor value.

**Why D is correct:** A Contributor Insights rule on the existing log group continuously ranks client ID and path pairs by 5XX count. It feeds a dashboard graph and an alarm without code, extra pipelines, or per-client custom metrics.

**Why the others fail the constraint:**
- **A.** Works, but means a polling function, a table, and high-cardinality custom metrics.
- **B.** Works, but requires an OpenSearch cluster and streaming pipeline to run.
- **C.** Works, but requires code changes and creates hundreds of thousands of custom metrics.

**Exam reflex:** For top talkers in logs, use Contributor Insights.

---

### Question 12

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS Fault Injection Service, Amazon CloudWatch alarms (stop conditions), Amazon ECS

**Socratic walkthrough**
1. *Is there a prebuilt definition of an AZ power interruption?* — Yes. The FIS scenario library has an AZ power interruption scenario.
2. *How does the experiment stop safely?* — Alarms configured as stop conditions.
3. *Where is the record?* — FIS keeps experiment history.

**Why C is correct:** Starting from the scenario library gives a maintained, multi-service AZ-impairment experiment. You only target your resources and attach the existing alarms as stop conditions — no fault-injection logic to write.

**Why the others fail the constraint:**
- **A.** Works, but you build and maintain the fault logic and the abort mechanism.
- **B.** Works, but it's manual, unsafe, and unrecorded.
- **D.** Works, but hand-building the template re-creates what the scenario already defines.

**Exam reflex:** For AZ game days, start from the FIS scenario library and use alarm stop conditions.

---

### Question 13

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation (custom resources), AWS Lambda, Amazon S3 (response URL)

**Socratic walkthrough**
1. *Why does the stack wait an hour?* — By default, CloudFormation waits up to an hour for a custom resource response.
2. *Can the wait be shortened per resource?* — Yes, with the custom resource's `ServiceTimeout` property.
3. *Does that require changing the function or adding components?* — No; it's a template property.

**Why D is correct:** Setting `ServiceTimeout` to 360 seconds makes CloudFormation fail the operation if no response arrives in that time, for creates and updates alike, which triggers rollback within the target window.

**Why the others fail the constraint:**
- **A.** Works, but adds a responder function and asynchronous invocation tuning.
- **B.** Works, but replaces the integration with SNS, a subscriber, and a workflow.
- **C.** Works, but depends on a human with curl.

**Exam reflex:** For hung custom resources, set `ServiceTimeout` before writing compensating code.

---

### Question 14

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS Health (exposed-credential events), Amazon EventBridge, AWS IAM

**Socratic walkthrough**
1. *How does AWS tell you a key was exposed?* — With an AWS Health event, `AWS_RISK_CREDENTIALS_EXPOSED`, delivered to the account's EventBridge.
2. *What already exists centrally?* — A bus that runs cross-account remediation runbooks.
3. *So what's missing?* — Only a forwarding rule in each account.

**Why A is correct:** A StackSet deploys one forwarding rule per account. The central bus starts an Automation runbook that deactivates the key, looks up the `owner` tag to notify, and opens the ticket. The runbook is maintained in one place.

**Why the others fail the constraint:**
- **B.** Works, but email parsing is brittle and requires mailbox infrastructure.
- **C.** Works, but deploys and maintains code in 90 accounts.
- **D.** Works, but duplicates AWS's own detection with a scanner that you run.

**Exam reflex:** Use AWS Health exposed-credential events with EventBridge for automatic key response.

---

### Question 15

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** Amazon ECS (deployment alarms and circuit breaker), AWS CloudFormation, Amazon CloudWatch

**Socratic walkthrough**
1. *What does the circuit breaker miss?* — Tasks that start fine but perform badly.
2. *Can ECS rolling deployments roll back on CloudWatch alarms?* — Yes. Deployment configuration alarms with rollback enabled.
3. *How much changes?* — One property in the existing service definition.

**Why A is correct:** ECS monitors the listed alarms during the rolling deployment and rolls back to the last completed deployment if any fires, alongside the circuit breaker. The template-based workflow and rolling updates stay the same.

**Why the others fail the constraint:**
- **B.** Works, but redeploys through the pipeline — more moving parts than the ECS-native rollback.
- **C.** Works, but custom code must track and restore the previous revision.
- **D.** Works, but means a deployment controller migration and new AppSpec tooling.

**Exam reflex:** For ECS rolling deployments, use the circuit breaker for start failures and deployment alarms for performance regressions.

---

## Scoring & Performance Breakdown

| Score | Band | Next step |
|---|---|---|
| **14–15 / 15** | Exam ready | Proceed to Practice Exam 6. |
| **12–13 / 15** | Passing | For each miss, re-read the distractor breakdown and note which constraint signal you overlooked. |
| **≤ 11 / 15** | Needs remediation | Use the domain column in the key table to pick study guides; retake this exam in a week. |
