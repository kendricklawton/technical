# AWS DevOps Engineer - Professional (DOP-C02) Practice Exam 4: Enterprise Observability & Distributed Tracing

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes (≈5 minutes per question; the real exam allows ≈2.4)  
**Passing Target:** 12 / 15 (80%)  
**Domains:** D1 ×4 · D2 ×2 · D3 ×2 · D4 ×3 · D5 ×2 · D6 ×2

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Answer all 15 before opening the answer key at the end.

---

## Question 1 (Domain 4: Monitoring and Logging)

A company with 60 AWS accounts in AWS Organizations wants a central monitoring account where SREs can view metrics, run CloudWatch Logs Insights queries, and inspect X-Ray traces across all workload accounts from one console, without copying telemetry. About four new workload accounts are created each month in the `Workloads` OU, and they must appear in the monitoring account automatically.

The company already uses service-managed CloudFormation StackSets from a delegated administrator account. Workloads run on Amazon ECS, AWS Lambda, and Amazon EC2, and log retention policies differ by team.

The SRE lead wants the simplest design to operate.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create metric streams and log subscription filters in every account that deliver to Amazon Data Firehose streams in the monitoring account, and export traces from an ADOT collector in each account to X-Ray in the monitoring account.
- B. In the monitoring account, create an Observability Access Manager sink whose policy lists each source account ID, and have a Lambda function triggered by account-creation events add new accounts to the policy and create their links.
- C. Create an Observability Access Manager sink whose policy allows the organization, and create a link from each existing account in the console, repeating the step for every new account during onboarding.
- D. Create an Observability Access Manager sink whose policy allows the organization, and deploy links for metrics, logs, and traces with a service-managed StackSet that targets the `Workloads` OU with automatic deployment.

---

## Question 2 (Domain 1: SDLC Automation)

A team deploys a high-traffic Lambda function with AWS SAM, using `AutoPublishAlias: live` and a `Linear10PercentEvery1Minute` deployment preference. The rollback alarm watches the function's `Errors` metric with only the `FunctionName` dimension and a threshold of 150 errors per minute. At 20,000 invocations per minute, the function's baseline error rate is about 0.3% because of client validation failures.

During a recent release, the new version failed 2% of its invocations. Diluted by the old version's traffic, the alarm fired only after 3 minutes, when 30% of traffic had shifted. The team wants rollback to begin as early as possible in future deployments, without being triggered by the old version's baseline errors.

**Which solution meets these requirements with the FASTEST rollback?**

- A. Replace the rollback alarm with a metric math error rate for the new version only, using `Errors` and `Invocations` with the `FunctionName`, `Resource`, and `ExecutedVersion` dimensions, taking the version from the function's published version resource.
- B. Lower the existing function-level alarm's threshold to 70 errors per minute so that a smaller increase over the baseline of about 60 errors per minute trips the alarm earlier in the linear deployment.
- C. Add a second rollback alarm on the `Errors` metric with the alias's `Resource` dimension and a 100-error threshold, so that only invocations routed through the `live` alias are counted during deployments.
- D. Run a Lambda function every 5 minutes that queries the function's logs with CloudWatch Logs Insights, computes the new version's error rate, and publishes a custom metric that a rollback alarm watches.

---

## Question 3 (Domain 2: Configuration Management and IaC)

A company runs 2,000 EC2 instances — 1,400 in Auto Scaling groups and 600 standalone — with the CloudWatch agent installed. The observability team must roll out a revised agent configuration that adds memory and disk metrics and new log files, and must keep every instance, including newly launched Auto Scaling instances, on the latest configuration.

All instances are managed by Systems Manager and carry an `Observability=standard` tag, and their instance roles include the `CloudWatchAgentServerPolicy` managed policy. The configuration file is 3 KB, and the team expects to revise it every few weeks.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Store the configuration in a Parameter Store parameter whose name starts with `AmazonCloudWatch-`, and create a tag-targeted State Manager association that runs `AmazonCloudWatch-ManageAgent` with that parameter, which also configures new instances.
- B. Bake the configuration into the AMI with EC2 Image Builder, roll it out with instance refreshes for the Auto Scaling groups, and use Run Command to copy the new file to the standalone instances and restart the agent after every revision.
- C. Add commands to each launch template's user data that download the configuration from S3 and restart the agent, and run a Run Command document against all running instances after every revision.
- D. Manage the configuration with an Ansible playbook run on a schedule from a control node on EC2 that connects to every instance over SSH, using a dynamic inventory built from the tag.

---

## Question 4 (Domain 5: Incident and Event Response)

An operations team wants each of its 300 critical CloudWatch alarms to open an AWS Systems Manager OpsCenter OpsItem with a severity and category when the alarm enters ALARM. Responders should work from one queue and run the associated Systems Manager Automation runbooks directly from each OpsItem.

The alarms are defined in CloudFormation, and the team already uses OpsCenter for other operational issues. The alarms notify an SNS topic that pages on-call, and that paging must continue.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create an EventBridge rule for alarm state changes to ALARM that invokes a Lambda function, which looks up each alarm's severity and category in a DynamoDB table and calls `CreateOpsItem`, and keep the SNS action for paging.
- B. Subscribe a Lambda function to the paging SNS topic that parses each alarm message and calls `CreateOpsItem` with a severity derived from the alarm name, and keep the SNS action for paging.
- C. Add a second alarm action to each alarm in the CloudFormation templates that creates a Systems Manager OpsItem with the alarm's severity and category, alongside the existing SNS action.
- D. Create an EventBridge rule for alarm state changes that starts a Step Functions workflow, which creates an OpsItem, looks up the matching runbook in a DynamoDB table, and starts the Automation execution, and keep the SNS action for paging.

---

## Question 5 (Domain 6: Security and Compliance)

A healthcare company stores application logs from 200 services in CloudWatch Logs in one account, and the logs sometimes contain patient names and medical record numbers. Compliance requires that this data be masked for everyone by default, including in log groups created later, and that only the six-person privacy operations team can view unmasked values during investigations.

Engineers use a permission set that includes `logs:GetLogEvents`, `logs:FilterLogEvents`, and `logs:StartQuery`, and the privacy team has its own permission set. Custom data identifiers for medical record numbers are already defined.

**Which solution meets these requirements with the LEAST privilege?**

- A. Create an account-level data protection policy with the managed and custom identifiers, and grant `logs:Unmask` to both the engineers' and the privacy team's permission sets, relying on training to limit its use to approved investigations.
- B. Create an account-level data protection policy with the managed and custom identifiers, add `logs:Unmask` only to the privacy team's permission set, and leave the engineers' existing read permissions unchanged.
- C. Create an account-level data protection policy with the managed and custom identifiers, and give the privacy team's permission set `CloudWatchLogsFullAccess` so it can unmask data and adjust the policy during investigations.
- D. Apply a data protection policy to each log group, with a Lambda function that applies it to new groups, and give the privacy team `logs:Unmask` on all resources plus permission to delete data protection policies.

---

## Question 6 (Domain 3: Resilient Cloud Solutions)

A video company processes transcoding jobs from an SQS standard queue with an Auto Scaling group of EC2 workers. Each worker handles about 4 messages per minute, and a new message must wait no more than 10 minutes, so each instance can tolerate a backlog of about 40 messages. Load varies unpredictably from 50 to 4,000 messages per minute, and the group currently runs a fixed 1,000 instances.

Group metrics collection is already enabled on the Auto Scaling group, and the workers use GPU instance types.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Create a step scaling policy on the queue's `ApproximateNumberOfMessagesVisible` metric with conservative steps that add 100 instances for each 2,000 visible messages and remove capacity only when the queue is nearly empty.
- B. Run a Lambda function every minute that divides the visible message count by the number of in-service instances, publishes the result as a custom metric, and drive a target tracking policy with a target value of 40 from it.
- C. Keep the fixed fleet sized for peak throughput, cover it with a Compute Savings Plan, and add a scheduled action that reduces capacity by 30% overnight, when load has historically stayed below peak.
- D. Create a target tracking policy whose metric math expression divides `ApproximateNumberOfMessagesVisible` by `GroupInServiceInstances`, with a target value of 40 messages per instance, so capacity follows the backlog in both directions while workers stay within the 10-minute wait limit.

---

## Question 7 (Domain 4: Monitoring and Logging)

A company runs 30 Java and Python microservices on Amazon EKS. SREs need a service map, golden-signal dashboards (latency, errors, and volume) per service and operation, and SLOs with burn-rate alerting. Developers can't change application code this quarter.

The cluster uses EKS Pod Identity, and Container Insights isn't enabled yet. Several services call DynamoDB and SQS, and the platform team wants AWS-managed components wherever possible. A few services were instrumented years ago with the AWS X-Ray SDK, which is now in maintenance mode.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Deploy the ADOT Operator with Java and Python auto-instrumentation, send metrics to Amazon Managed Service for Prometheus and traces to X-Ray, and build service maps, dashboards, and burn-rate rules in Amazon Managed Grafana.
- B. Run self-managed Jaeger and Prometheus in the cluster with OpenTelemetry agents injected through a mutating webhook, and compute SLOs with Prometheus recording rules and Alertmanager.
- C. Install the Amazon CloudWatch Observability EKS add-on, enable Application Signals auto-instrumentation for the workloads with pod annotations, and define SLOs on the discovered services and operations.
- D. Install the ADOT EKS add-on with Java and Python auto-instrumentation, send traces to X-Ray for the service map and metrics to CloudWatch, and build per-operation SLO alarms with metric math.

---

## Question 8 (Domain 1: SDLC Automation)

A company's CodePipeline (V2) pipeline builds a monorepo and deploys it to a shared integration environment, where a 25-minute test suite runs before promotion. Developers merge about 30 times a day. Auditors require that every merged commit be deployed and tested — none skipped — and the integration environment can't run two test suites at once.

The pipeline runs in the default execution mode, and the team noticed that some commits never reached the integration stage. A CodeBuild project runs the tests, and a separate nightly job resets the environment's data.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Switch to `PARALLEL` execution mode and add a DynamoDB-based lock that the integration CodeBuild project acquires before running the tests, retrying until the lock is free and releasing it after the suite finishes.
- B. Disable automatic triggers, send commit events through EventBridge to an SQS FIFO queue, and have a Lambda function start the next pipeline execution only after the previous one finishes.
- C. Change the pipeline's execution mode to `QUEUED` so that each new execution waits for the stage ahead of it to finish instead of being superseded, keeping the existing CodeBuild test stage unchanged.
- D. Switch to `PARALLEL` execution mode and replace the integration stage with a Step Functions workflow that uses a DynamoDB semaphore to run one test suite at a time before reporting back to the pipeline.

---

## Question 9 (Domain 1: SDLC Automation)

A company deploys an order API to 36 EC2 instances across three Availability Zones (12 per AZ) with CodeDeploy in-place deployments behind an ALB. Some defects appear only after about 20 minutes of production traffic, and the deployment group has automatic rollback on two CloudWatch alarms.

The deployment configuration is `CodeDeployDefault.OneAtATime`, and each instance takes about 2 minutes to deploy. Last month a defect that surfaced after 20 minutes had already reached instances in all three AZs before rollback started. The company wants future defects contained to as few AZs as possible before rollback starts.

**Which solution meets these requirements with the SMALLEST blast radius?**

- A. Keep `CodeDeployDefault.OneAtATime`, which limits each step to one instance, and add a third alarm on per-instance memory and latency metrics so that automatic rollback starts sooner after a defect appears.
- B. Create a custom deployment configuration with a zonal configuration that deploys one AZ at a time, monitors each AZ for 30 minutes before moving on, and keeps a minimum number of healthy hosts per AZ.
- C. Switch to `CodeDeployDefault.HalfAtATime` so that each deployment completes in two batches of 18 instances, giving the alarms time to evaluate between batches before the second half of the fleet is updated.
- D. Create a custom deployment configuration that requires 90% of the fleet to stay healthy, so at most three instances across all three AZs are updated at a time and each batch can be observed before the next starts.

---

## Question 10 (Domain 5: Incident and Event Response)

An e-commerce company processes order events with 25 Lambda functions that EventBridge invokes asynchronously. When a function fails after its retries, the event goes to an SQS dead-letter queue. During incidents, engineers spend hours correlating DLQ messages with CloudWatch Logs to find each failure's error and stack trace.

The team wants each failed event captured together with its original payload, error type and message, and stack trace, in one record that can be inspected and later replayed, without changing function code. The functions are written in TypeScript.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Keep the DLQs, and have engineers run CloudWatch Logs Insights queries by the request ID found in each DLQ message's attributes, saving the queries so that every incident uses the same correlation steps.
- B. Create an EventBridge archive on the bus, and during incidents search X-Ray traces for failed invocations, extract their errors, and replay the matching time window from the archive to the functions.
- C. Put a Step Functions Express workflow in front of each function that invokes it synchronously, catches errors, and writes the input, error, and cause to a DynamoDB table used for replays.
- D. Configure an on-failure destination in each function's asynchronous invocation settings that sends invocation records — with the request payload and the error response — to an SQS queue for inspection and replay.

---

## Question 11 (Domain 6: Security and Compliance)

A company's organization trail already delivers management events for 120 accounts to an S3 bucket in a log archive account — about 6 TB per year, retained for 7 years. The security team runs ad hoc SQL investigations a few times a month, usually scoped to a few accounts, Regions, and days.

Today analysts download log files and parse them locally. The log archive bucket uses S3 Object Lock, and lifecycle rules move objects older than 90 days to S3 Glacier Instant Retrieval.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Create a second organization trail that delivers to CloudWatch Logs in the log archive account with 7-year retention, and run CloudWatch Logs Insights queries across the log group for each investigation.
- B. Create an Athena table over the existing trail bucket with partition projection on account, Region, and date, so that each investigation scans only the partitions it needs and no log data is copied.
- C. Load the logs into an Amazon OpenSearch Service domain with UltraWarm and cold storage tiers sized for 7 years of data, and run investigations with SQL in the query workbench.
- D. Run a nightly AWS Glue job that converts new log files to Parquet in a separate bucket and loads them into an Amazon Redshift Serverless table, retaining 7 years of data in managed storage.

---

## Question 12 (Domain 3: Resilient Cloud Solutions)

A payments API runs on Lambda functions that connect to an Amazon RDS for PostgreSQL Multi-AZ DB instance. During the last two automatic failovers, the API returned errors for about 2 minutes: the failover itself took most of that time, and thousands of new Lambda connections then overwhelmed the new primary.

The company wants to minimize the time that the API is unavailable during a database failover, without changing database engines or SQL. The functions read the database endpoint from an environment variable, and reports run nightly against a read replica.

**Which solution meets these requirements with the FASTEST recovery?**

- A. Keep the Multi-AZ DB instance, put RDS Proxy in front of it, and point the functions at the proxy endpoint so that connections are pooled and the proxy reconnects to the new primary without waiting on DNS.
- B. Keep the Multi-AZ DB instance, and add exponential backoff with jitter to the functions' connection logic plus a lower DNS cache TTL in the runtime so that clients reconnect smoothly after failover.
- C. Promote the existing read replica during failures with an SSM Automation runbook triggered by RDS failover events, and update the functions' environment variable to the new endpoint.
- D. Migrate to an RDS Multi-AZ DB cluster with two readable standby instances, put RDS Proxy in front of its writer, and point the functions at the proxy endpoint.

---

## Question 13 (Domain 4: Monitoring and Logging)

A company must retain VPC Flow Logs for 120 VPCs for one year and analyze them ad hoc with SQL — usually one VPC and a few hours at a time — about twice a week. Flow logs currently go to CloudWatch Logs, which has become a top-five cost item, and logs are rarely queried after 30 days.

An Athena workgroup already exists in the security account. The VPCs use a custom flow log format that includes `pkt-srcaddr` and `flow-direction`.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Keep CloudWatch Logs as the destination with one-year retention, move the log groups to the Infrequent Access log class, and run CloudWatch Logs Insights queries for investigations.
- B. Publish flow logs directly to S3 in Parquet format with Hive-compatible prefixes and hourly partitions, add a lifecycle rule for one-year retention, and query them with Athena.
- C. Send flow logs to Amazon Data Firehose, which delivers them to an Amazon OpenSearch Service domain sized for one year of storage, and query them with SQL through OpenSearch during investigations.
- D. Publish flow logs directly to S3 in the default text format with daily partitions, add a lifecycle rule for one-year retention, and query them with Athena.

---

## Question 14 (Domain 2: Configuration Management and IaC)

A company runs 300 Python and Node.js Lambda functions that log at DEBUG level, producing about 40 TB of CloudWatch Logs ingestion per month. Operations needs only WARN and above during normal operation, retained for 30 days, but must be able to raise a single function to DEBUG within minutes during an incident, without changing or redeploying code.

All functions are deployed with CloudFormation and already use their runtime's standard logging library. The team investigates with CloudWatch Logs Insights.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Set each function's `LoggingConfig` in CloudFormation to JSON format with application and system log levels of `WARN`, and raise a function's application log level to `DEBUG` with a configuration update during incidents.
- B. Keep DEBUG logging, move every log group to the Infrequent Access log class with 30-day retention, and continue to use CloudWatch Logs Insights for investigations.
- C. Keep DEBUG logging, add a subscription filter on every log group that forwards only WARN and above to Amazon Data Firehose for delivery to S3, and set one-day retention on the log groups.
- D. Keep DEBUG logging with 30-day retention, and create CloudWatch Logs field indexes on log level and request ID so that Logs Insights queries scan less data during investigations.

---

## Question 15 (Domain 1: SDLC Automation)

A company runs about 5,000 short CodeBuild builds per day for linting and unit tests across 200 repositories. Each build takes under 3 minutes, needs less than 4 GB of memory, uses a Node.js or Python runtime, and doesn't build container images or need VPC access. Builds arrive in bursts after merges and stop overnight.

The builds currently use the on-demand `BUILD_GENERAL1_MEDIUM` EC2 compute type, and engineers complain about provisioning delays. The company has no reserved capacity fleets.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Create a reserved capacity fleet sized for peak concurrency so that builds start immediately on warm hosts, and attach all 200 projects to the fleet.
- B. Move the builds to self-hosted GitHub Actions runners on an Auto Scaling group of EC2 Spot Instances that scales on job queue depth, while keeping CodeBuild only for release builds.
- C. Switch the projects to the AWS Lambda compute type for CodeBuild, choosing a 4 GB memory size and the Node.js or Python Lambda images that match each repository.
- D. Keep on-demand EC2 compute but switch to `BUILD_GENERAL1_SMALL` with local caching, and batch the linting and unit tests of related repositories into shared builds.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | D | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 2 | A | Domain 1: SDLC Automation | FASTEST rollback |
| 3 | A | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 4 | C | Domain 5: Incident and Event Response | LEAST operational overhead |
| 5 | B | Domain 6: Security and Compliance | LEAST privilege |
| 6 | D | Domain 3: Resilient Cloud Solutions | MOST cost-effective |
| 7 | C | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 8 | C | Domain 1: SDLC Automation | LEAST operational overhead |
| 9 | B | Domain 1: SDLC Automation | SMALLEST blast radius |
| 10 | D | Domain 5: Incident and Event Response | LEAST operational overhead |
| 11 | B | Domain 6: Security and Compliance | MOST cost-effective |
| 12 | D | Domain 3: Resilient Cloud Solutions | FASTEST recovery |
| 13 | B | Domain 4: Monitoring and Logging | MOST cost-effective |
| 14 | A | Domain 2: Configuration Management and IaC | MOST cost-effective |
| 15 | C | Domain 1: SDLC Automation | MOST cost-effective |

---

### Question 1

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** CloudWatch cross-account observability (OAM), AWS Organizations, AWS X-Ray

**Socratic walkthrough**
1. *Which option shares telemetry without copying it?* — Observability Access Manager (OAM) sinks and links.
2. *How can a sink accept every account in the organization?* — With a sink policy that allows the organization.
3. *What creates links in new accounts without anyone acting?* — A service-managed StackSet with automatic deployment on the OU.

**Why D is correct:** An organization-scoped sink accepts links from any member account, and the StackSet creates links for metrics, logs, and traces in every current and future `Workloads` account. SREs then query everything from the monitoring account.

**Why the others fail the constraint:**
- **A.** Works, but copies all telemetry through Firehose streams, filters, and per-account collectors, all of which you operate and pay for.
- **B.** Works, but a custom Lambda function maintains the sink policy and creates links.
- **C.** Works, but manual console linking for every new account is recurring toil that's easy to miss.

**Exam reflex:** Cross-account CloudWatch with no data copy means OAM; for "future accounts", use StackSets auto-deployment.

---

### Question 2

**Answer:** A  
**Constraint:** FASTEST rollback · **Services:** AWS CodeDeploy, AWS Lambda (versions and aliases), Amazon CloudWatch

**Socratic walkthrough**
1. *Why did the alarm fire late?* — The function-level metric mixes the new version's 2% error rate with the old version's 0.3% baseline.
2. *Can CloudWatch separate the two?* — Yes. Invocations through an alias publish metrics with the `ExecutedVersion` dimension.
3. *What does a version-only error rate look like at the 10% step?* — 2% versus the 0.3% baseline, which is detectable in the first minute.

**Why A is correct:** Alarming on the new version's own error rate (Errors ÷ Invocations with `FunctionName`, `Resource`, and `ExecutedVersion`) makes the signal independent of traffic share. CodeDeploy rolls back at the first shift step instead of at 30%.

**Why the others fail the constraint:**
- **B.** Fires earlier, but is still diluted by traffic share and is close enough to the baseline to cause false rollbacks.
- **C.** Alias-level metrics still include both versions, so dilution remains.
- **D.** A 5-minute query cycle is slower than the original alarm.

**Exam reflex:** For Lambda canary alarms, scope metrics to the new version with `ExecutedVersion`.

---

### Question 3

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS Systems Manager State Manager, AWS Systems Manager Parameter Store, Amazon CloudWatch agent

**Socratic walkthrough**
1. *Where can the agent read its configuration with permissions that instances already have?* — From a Parameter Store parameter named `AmazonCloudWatch-*`, which `CloudWatchAgentServerPolicy` allows.
2. *What applies the configuration to every instance, including new ones?* — A tag-targeted State Manager association, which runs when a new instance registers and on its schedule.
3. *Is there anything to build or bake?* — No.

**Why A is correct:** `AmazonCloudWatch-ManageAgent` configures the agent from the parameter. The association keeps all 2,000 instances — whether in Auto Scaling groups or standalone, existing or new — on the current configuration, and a revision means updating one parameter.

**Why the others fail the constraint:**
- **B.** Works, but each revision triggers an AMI build, instance refreshes, and a separate push to standalone instances.
- **C.** Works, but launch templates and manual Run Command executions must be kept in sync on every revision.
- **D.** Works, but means a control node, SSH access, and inventory to maintain.

**Exam reflex:** CloudWatch agent configuration at scale means a Parameter Store parameter (`AmazonCloudWatch-` prefix) plus a State Manager association.

---

### Question 4

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** Amazon CloudWatch alarms, Systems Manager OpsCenter, Systems Manager Automation

**Socratic walkthrough**
1. *Can an alarm create an OpsItem by itself?* — Yes. CloudWatch alarm actions include creating a Systems Manager OpsItem with a severity and category.
2. *Can an alarm have more than one action?* — Yes, so SNS paging continues.
3. *Where do runbooks come in?* — Responders run Automation runbooks from the OpsItem in OpsCenter.

**Why C is correct:** Adding an OpsItem action to each alarm is a declarative template change: OpsCenter receives the alarm details, severity, and category, and the existing SNS action keeps paging.

**Why the others fail the constraint:**
- **A.** Works, but adds a function and a DynamoDB lookup table to maintain.
- **B.** Works, but message parsing and severity-by-naming conventions are fragile custom code.
- **D.** Works, but a state machine plus a lookup table is the heaviest option.

**Exam reflex:** Alarm → OpsItem is a native alarm action; you don't need glue code.

---

### Question 5

**Answer:** B  
**Constraint:** LEAST privilege · **Services:** CloudWatch Logs data protection, AWS IAM Identity Center permission sets, AWS IAM

**Socratic walkthrough**
1. *What covers log groups created in the future?* — An account-level data protection policy.
2. *Which permission reveals masked data?* — `logs:Unmask`.
3. *Who needs it?* — Only the privacy team; engineers keep their existing read access, which returns masked values.

**Why B is correct:** The account-level policy masks the identifiers in every current and future log group. Granting `logs:Unmask` only to the privacy team's permission set gives exactly one group the ability to view unmasked data, with nothing more.

**Why the others fail the constraint:**
- **A.** Works, but gives unmasking to every engineer.
- **C.** Works, but `logs:*` also lets the privacy team delete or weaken the masking policy.
- **D.** Works, but unmasking on all resources plus policy deletion is broader, and per-group policies need custom code.

**Exam reflex:** For sensitive logs, use account-level data protection and grant `logs:Unmask` narrowly.

---

### Question 6

**Answer:** D  
**Constraint:** MOST cost-effective · **Services:** Amazon SQS, Amazon EC2 Auto Scaling, Amazon CloudWatch (metric math)

**Socratic walkthrough**
1. *What's the right scaling metric for queue workers?* — Backlog per instance, compared with the acceptable backlog (40).
2. *Do you need a custom metric to get it?* — No. Target tracking accepts a metric math expression.
3. *Which options pay for idle capacity or extra components?* — A, B, and C.

**Why D is correct:** Target tracking on queue depth ÷ in-service instances keeps backlog per instance near 40, scaling both out and in with demand. That pays only for needed GPU instances and uses free queue and group metrics.

**Why the others fail the constraint:**
- **A.** Works, but conservative steps overprovision and scale in late — expensive with GPU instances.
- **B.** Works, but adds a custom metric and a Lambda function to produce a value that metric math computes for free.
- **C.** Works, but pays for peak capacity most of the day.

**Exam reflex:** Scale SQS workers on backlog per instance with target tracking and metric math.

---

### Question 7

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** CloudWatch Application Signals, Amazon EKS, AWS Distro for OpenTelemetry

**Socratic walkthrough**
1. *How do you instrument without code changes?* — Auto-instrumentation agents injected through pod annotations.
2. *Which option also gives a service map, golden signals, and SLOs as managed features?* — Application Signals.
3. *What about the X-Ray SDK?* — It's in maintenance mode; don't build new work on it.

**Why C is correct:** The CloudWatch Observability EKS add-on deploys the agent and the auto-instrumentation operator. Application Signals discovers services and operations and provides service maps, standard dashboards, and SLOs with burn-rate alerting out of the box.

**Why the others fail the constraint:**
- **A.** Works, but you build and maintain Grafana dashboards, maps, and burn-rate logic.
- **B.** Works, but a self-managed tracing and metrics stack is the heaviest option.
- **D.** Works, but SLOs and per-operation views are custom metric math you have to maintain.

**Exam reflex:** For managed APM with SLOs on EKS, use the CloudWatch Observability add-on with Application Signals.

---

### Question 8

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS CodePipeline (V2 execution modes), AWS CodeBuild, Amazon ECS

**Socratic walkthrough**
1. *Why are commits being skipped?* — The default `SUPERSEDED` mode replaces waiting executions with newer ones.
2. *Which mode keeps every execution and still prevents overlapping runs in a stage?* — `QUEUED`.
3. *Is anything custom needed?* — No; it's a pipeline setting.

**Why C is correct:** In `QUEUED` mode, executions wait their turn instead of being superseded, and a stage still processes one execution at a time. Every commit is tested, with no concurrent suites in the shared environment.

**Why the others fail the constraint:**
- **A.** Works, but adds a custom lock with retry logic and failure handling.
- **B.** Works, but replaces native triggers with a custom queueing system.
- **D.** Works, but a semaphore workflow re-creates what `QUEUED` provides.

**Exam reflex:** "Don't skip commits, don't overlap" in CodePipeline V2 means `QUEUED` execution mode.

---

### Question 9

**Answer:** B  
**Constraint:** SMALLEST blast radius · **Services:** AWS CodeDeploy (zonal configuration), Amazon EC2 Auto Scaling, Amazon CloudWatch

**Socratic walkthrough**
1. *How many instances does `OneAtATime` update in 20 minutes?* — About 10, chosen without regard to AZ.
2. *Which options still mix AZs before the defect appears?* — A, C, and D: every fleet-wide batching scheme.
3. *Can CodeDeploy deploy and bake one AZ at a time?* — Yes. Deployment configurations support zonal configuration with per-AZ monitoring durations.

**Why B is correct:** A zonal configuration finishes one AZ, then waits the monitoring duration — longer than the 20-minute latency — before the next AZ. Alarm-based rollback therefore starts while only one AZ is affected.

**Why the others fail the constraint:**
- **A.** Works, but even with earlier alarms, instance-by-instance rollout crosses AZs within minutes.
- **C.** Works, but the first batch of 18 already spans all three AZs.
- **D.** Works, but three-instance batches still spread across AZs over 20 minutes.

**Exam reflex:** To limit blast radius by AZ, use CodeDeploy zonal configuration with monitoring durations longer than defect latency.

---

### Question 10

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS Lambda (destinations), Amazon EventBridge, Amazon SQS

**Socratic walkthrough**
1. *What does a DLQ message contain?* — The event payload plus a few attributes, but not the full error response.
2. *What does an on-failure destination record contain?* — Request context, the request payload, and the response payload with the error type, message, and stack trace.
3. *Does it need code?* — No; it's a configuration setting.

**Why D is correct:** On-failure destinations capture the complete invocation record when retries are exhausted. Engineers get payload and error details together and can replay events from the stored payloads.

**Why the others fail the constraint:**
- **A.** Works, but keeps the manual correlation that caused the problem.
- **B.** Works, but combining trace search with time-window replays is manual and imprecise.
- **C.** Works, but adds 25 workflows and a table in front of the functions.

**Exam reflex:** For async Lambda failures with context, use on-failure destinations rather than DLQs.

---

### Question 11

**Answer:** B  
**Constraint:** MOST cost-effective · **Services:** AWS CloudTrail (organization trail), Amazon S3, Amazon Athena

**Socratic walkthrough**
1. *Is the data already somewhere queryable?* — Yes. Every log is already in S3, under lifecycle management.
2. *What keeps Athena scans small?* — Partition projection on the account, Region, and date path components.
3. *Can Athena read Glacier Instant Retrieval objects?* — Yes. That storage class allows immediate access.

**Why B is correct:** Querying in place adds no ingestion, duplicate storage, or always-on clusters — just Athena charges per data scanned, which partition projection keeps small for scoped investigations.

**Why the others fail the constraint:**
- **A.** Works, but pays CloudWatch Logs ingestion and 7 years of storage for data that already exists.
- **C.** Works, but keeps a cluster running for 7 years of data queried a few times a month.
- **D.** Works, but adds daily ETL, duplicate storage, and a warehouse.

**Exam reflex:** CloudTrail logs already in S3 call for Athena with partition projection.

---

### Question 12

**Answer:** D  
**Constraint:** FASTEST recovery · **Services:** Amazon RDS (Multi-AZ DB cluster), Amazon RDS Proxy, AWS Lambda

**Socratic walkthrough**
1. *What are the two parts of the outage?* — Failover time and the connection storm afterward.
2. *Which deployment type fails over faster than a Multi-AZ DB instance?* — A Multi-AZ DB cluster, which typically fails over in under 35 seconds to a readable standby.
3. *What fixes the connection storm?* — RDS Proxy pooling.

**Why D is correct:** A Multi-AZ DB cluster shortens the failover itself, and RDS Proxy absorbs Lambda's connection churn and reconnects to the new writer. Together they address both parts of the outage.

**Why the others fail the constraint:**
- **A.** Helps the connection storm, but the Multi-AZ DB instance failover itself still takes most of the 2 minutes.
- **B.** Smooths reconnection, but doesn't shorten failover.
- **C.** Manual replica promotion is far slower than an automatic failover.

**Exam reflex:** Fastest RDS failover with Lambda: Multi-AZ DB cluster plus RDS Proxy.

---

### Question 13

**Answer:** B  
**Constraint:** MOST cost-effective · **Services:** Amazon VPC Flow Logs, Amazon S3, Amazon Athena

**Socratic walkthrough**
1. *Where are flow logs cheapest to land?* — Directly in S3, without CloudWatch Logs ingestion.
2. *What reduces Athena cost per query?* — Columnar Parquet plus fine-grained (hourly, Hive-compatible) partitions.
3. *Which option differs only in format and partitioning?* — D, the "almost right" choice.

**Why B is correct:** Flow logs support S3 delivery in Parquet with Hive-compatible, per-hour partitions. Queries for one VPC over a few hours scan a tiny, compressed slice, and storage is S3.

**Why the others fail the constraint:**
- **A.** Works, but you still pay CloudWatch Logs ingestion, just at a lower rate.
- **C.** Works, but adds Firehose plus an always-on search cluster.
- **D.** Works, but text files with daily partitions scan far more data per query.

**Exam reflex:** Flow logs for ad hoc SQL: S3, Parquet, Hive-compatible hourly partitions, and Athena.

---

### Question 14

**Answer:** A  
**Constraint:** MOST cost-effective · **Services:** AWS Lambda (advanced logging controls), AWS CloudFormation, Amazon CloudWatch Logs

**Socratic walkthrough**
1. *What drives the bill?* — 40 TB a month of ingestion, mostly DEBUG.
2. *Which options stop DEBUG logs from being ingested at all?* — Only filtering at the source.
3. *Can Lambda filter by log level without code changes?* — Yes. Advanced logging controls with JSON format set application and system log levels.

**Why A is correct:** With `LoggingConfig` at `WARN`, Lambda drops lower-level logs before ingestion. During incidents, a single configuration update raises one function to `DEBUG` without redeploying code.

**Why the others fail the constraint:**
- **B.** Works, but you still ingest all 40 TB, just at a lower rate.
- **C.** Works, but subscription filters act after ingestion, then add Firehose and S3 costs.
- **D.** Works, but field indexes lower query costs, not ingestion.

**Exam reflex:** To reduce Lambda log cost without code changes, set the log level in `LoggingConfig`.

---

### Question 15

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** AWS CodeBuild (compute types), AWS CodePipeline, Amazon CloudWatch

**Socratic walkthrough**
1. *What fits a short, bursty, non-Docker, non-VPC build profile?* — CodeBuild's Lambda compute type.
2. *What does it cost when idle overnight?* — Nothing; you pay only for build time.
3. *Which options pay for idle capacity or add infrastructure?* — A and B.

**Why C is correct:** Lambda compute starts builds almost immediately, which removes provisioning delays, and bills at lower per-unit rates than EC2 compute for short builds. These builds meet its limitations: no Docker, no VPC, and short durations.

**Why the others fail the constraint:**
- **A.** Works, but a fleet sized for peak bills for its instances even when idle overnight.
- **B.** Works, but adds runner infrastructure and operations for marginal Spot savings.
- **D.** Works, but keeps EC2 provisioning overhead, and batching couples unrelated repositories.

**Exam reflex:** For short, bursty builds without Docker or VPC, choose CodeBuild Lambda compute.

---

## Scoring & Performance Breakdown

| Score | Band | Next step |
|---|---|---|
| **14–15 / 15** | Exam ready | Proceed to Practice Exam 5. |
| **12–13 / 15** | Passing | For each miss, re-read the distractor breakdown and note which constraint signal you overlooked. |
| **≤ 11 / 15** | Needs remediation | Use the domain column in the key table to pick study guides; retake this exam in a week. |
