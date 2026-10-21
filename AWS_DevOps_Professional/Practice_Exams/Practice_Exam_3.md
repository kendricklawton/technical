# AWS DevOps Engineer - Professional (DOP-C02) Practice Exam 3: High Availability, DR & Traffic Failover

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes (≈5 minutes per question; the real exam allows ≈2.4)  
**Passing Target:** 12 / 15 (80%)  
**Domains:** D1 ×3 · D2 ×3 · D3 ×3 · D4 ×2 · D5 ×2 · D6 ×2

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Answer all 15 before opening the answer key at the end.

---

## Question 1 (Domain 3: Resilient Cloud Solutions)

An insurance company runs its claims platform in us-east-1: an Auto Scaling group of 30 EC2 instances behind an Application Load Balancer and an Aurora PostgreSQL cluster with a writer and two readers. Regulators require a disaster recovery capability in us-west-2 with an RPO of 1 minute and an RTO of 30 minutes. Apart from the database, the application is stateless.

The build pipeline already copies launch templates and AMIs to us-west-2, and Route 53 hosts the public zone. Recovery drills show that the application tier launches and passes health checks in about 9 minutes, and that adding a DB instance to an Aurora cluster takes about 12 minutes. The analytics team occasionally asks for a read-only copy of the data in us-west-2 and can tolerate a short setup time.

The company wants to meet its DR objectives at the lowest cost.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Add us-west-2 as an Aurora global database secondary with one provisioned reader instance, keep a scaled-down Auto Scaling group of six instances running there behind an ALB, and switch traffic with Route 53 failover records during a disaster.
- B. Run the full stack active-active in both Regions with an Aurora global database that uses write forwarding from us-west-2, and route users with Route 53 latency records and health checks to the closest healthy Region.
- C. Add us-west-2 as a headless Aurora global database secondary with no DB instances and keep its Auto Scaling group at zero; during a disaster, add a DB instance, fail over, scale out, and switch the Route 53 records.
- D. Add us-west-2 as an Aurora global database secondary with two production-sized reader instances and keep the Auto Scaling group at zero; during a disaster, fail over the global database, scale out, and switch Route 53 records.

---

## Question 2 (Domain 1: SDLC Automation)

A company deploys a CloudFormation stack for its order service to us-east-1, eu-west-1, and ap-southeast-2. Releases must reach us-east-1 first, then eu-west-1, then ap-southeast-2, and before each later Region a regional on-call lead must approve the promotion. The template creates a Regional AWS WAF web ACL and an ACM certificate.

The company already has a CodePipeline (V2) pipeline in us-east-1 that builds the template and deploys it to us-east-1. The team considered CloudFormation StackSets but rejected it because each Region needs its own approver.

The team wants to extend the release process to all three Regions.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create a separate pipeline in each Region, replicate build artifacts with S3 Cross-Region Replication, and use EventBridge rules on pipeline execution success to start the next Region's pipeline, each beginning with a manual approval action.
- B. Add a stage per Region to the existing pipeline with a CodeBuild action that packages the template and runs `aws cloudformation deploy` with the `--region` option, preceded by a manual approval action for that Region's lead.
- C. Add a stage per Region to the existing pipeline containing a manual approval action for that Region's lead and a CloudFormation deploy action configured for that Region, with an artifact store defined for each Region.
- D. Create a self-managed StackSet with one stack instance per Region, and add stages to the existing pipeline with a manual approval action and a Lambda invoke action that calls `UpdateStackInstances` for the next Region.

---

## Question 3 (Domain 2: Configuration Management and IaC)

A company runs 30 CloudFormation stacks that each contain an Aurora cluster, a DynamoDB table, and an S3 bucket alongside compute resources, and teams are heavily refactoring templates this quarter. Last month a renamed property caused CloudFormation to replace a DynamoDB table during an update, and the team restored data from a backup with 40 minutes of data loss.

The company requires that no stack update can replace or delete these stateful resources unless a platform engineer explicitly allows it for that specific update, and that the data must survive if a stateful resource is ever removed from a template. Updates are deployed through CodePipeline with change sets, and termination protection is already enabled on every stack.

The platform team wants the control to need as little ongoing effort as possible.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Attach a stack policy that denies `Update:Replace` and `Update:Delete` on the stateful logical IDs, set `DeletionPolicy` and `UpdateReplacePolicy` to `Snapshot` or `Retain` on them, and supply a temporary override policy for approved updates.
- B. Activate a Lambda hook in each account whose function inspects every resource operation on stateful resource types and fails replacements and deletions unless a platform engineer has added an approval tag to the stack for that update.
- C. Add an SCP that denies delete actions for Aurora clusters, DynamoDB tables, and S3 buckets to the CloudFormation service roles, and have platform engineers detach the SCP temporarily from the account for each approved update.
- D. Add a CodeBuild step before each change set is executed that parses the change set and fails the pipeline whenever a stateful resource shows a replacement or removal, unless an approval flag file is committed for that update.

---

## Question 4 (Domain 4: Monitoring and Logging)

A gaming company serves players from two Regions with Route 53 failover routing between Application Load Balancers. During a recent Regional impairment, the primary endpoint returned errors for nearly three minutes before Route 53 failed over, and the company wants to detect an unhealthy primary endpoint as quickly as possible.

The application exposes `/health/deep`, which checks the database and cache and responds in under 200 ms. The primary record is an alias with Evaluate Target Health, and the ALB target group uses a 30-second health check interval with an unhealthy threshold of 3. A CloudWatch alarm on the ALB's 5XX count already exists with a 1-minute period and two evaluation periods.

Players connect over a custom TCP protocol after DNS resolution, and DNS TTLs are already set to 30 seconds.

**Which solution meets these requirements with the FASTEST detection?**

- A. Keep Evaluate Target Health on the alias record, and lower the target group's health check interval to 15 seconds with an unhealthy threshold of 2 so that the ALB reports its targets as unhealthy sooner.
- B. Create a Route 53 health check that monitors the state of the existing 5XX CloudWatch alarm, and associate that health check with the primary failover record in place of Evaluate Target Health.
- C. Create three Route 53 HTTPS health checks on `/health/deep` at the standard 30-second interval, combine them in a calculated health check that requires two healthy children, and associate it with the primary record.
- D. Create a Route 53 HTTPS health check on `/health/deep` with the fast 10-second interval and a failure threshold of 1, and associate it with the primary failover record in place of Evaluate Target Health.

---

## Question 5 (Domain 3: Resilient Cloud Solutions)

A document-processing service stores uploads in S3 buckets in us-east-1 and us-east-2 with two-way S3 Replication and Replication Time Control. Clients in 200 branch offices upload and download through an S3 Multi-Region Access Point that was configured last year for active-passive routing, with us-east-1 active.

During a recent us-east-1 S3 impairment, the operations team needed 40 minutes to fail over, because its runbook changed an application configuration value and restarted services at every branch. The branch application is a .NET desktop client that signs requests with SigV4A, and the team can ship a client update if necessary.

The team wants reads and writes to move to us-east-2 as quickly as possible during the next impairment.

**Which solution meets these requirements with the FASTEST recovery?**

- A. Automate the existing runbook with an SSM Automation document that updates the configuration value and restarts the client service on every branch machine through Systems Manager hybrid activations.
- B. Put an Amazon API Gateway upload and download API in each Region in front of its bucket, update the client to use it, and use Route 53 failover records with health checks so clients resolve to the healthy Region.
- C. Keep clients on the Multi-Region Access Point and, during an impairment, use its failover controls to make us-east-1 passive and us-east-2 active, leaving the existing two-way replication in place.
- D. Update the client with a retry policy that switches from the Multi-Region Access Point to the us-east-2 bucket's Regional endpoint after three failed requests, keeping two-way replication so both buckets converge after recovery.

---

## Question 6 (Domain 5: Incident and Event Response)

A financial firm replicates 2 billion objects from a primary bucket in us-east-1 to a DR bucket in us-west-2 with S3 Replication Time Control. Auditors require that the team be alerted whenever an object fails to replicate or misses the 15-minute replication threshold, and that failed objects be re-replicated.

Last month a KMS key policy change caused 30,000 objects to fail replication silently for a day. The buckets use SSE-KMS, replication metrics are enabled as part of Replication Time Control, and the on-call team receives alerts through an SNS topic.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Generate a daily S3 Inventory report for both buckets, run an Athena query that finds objects missing from the DR bucket, alert through the SNS topic, and copy the missing objects with a Lambda function.
- B. Configure S3 Event Notifications for the replication-failed and missed-threshold event types to publish to the SNS topic, and run S3 Batch Replication jobs on demand that re-replicate objects whose replication status is failed.
- C. Replace live replication with a Lambda function invoked for every object creation event that copies the object to the DR bucket and publishes to the SNS topic when a copy fails or takes longer than 15 minutes.
- D. Create a CloudWatch alarm on the replication latency and failed-operations metrics that notifies the SNS topic, and have on-call engineers run `aws s3 sync` between the buckets after each alert to copy objects that failed.

---

## Question 7 (Domain 6: Security and Compliance)

A healthcare company is building a patient-records application that will store data in a DynamoDB global table replicated between us-east-1 and us-west-2. Compliance mandates client-side encryption of sensitive attributes with the AWS Database Encryption SDK before items are written. The application will run active-passive, with us-west-2 as the recovery Region; no data exists yet.

The DR plan requires that during a us-east-1 outage, the us-west-2 application can decrypt every existing item and encrypt new items without calling any us-east-1 endpoint, and that after failback, us-east-1 can read items written in us-west-2.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Encrypt with a multi-keyring that wraps each data key under single-Region KMS keys in both Regions, switch the recovery application to a local-only keyring during an outage, and re-encrypt items written during the outage after failback.
- B. Generate an AES-256 wrapping key, store it in a Secrets Manager secret replicated to us-west-2, and use a raw AES keyring in both Regions so that decryption never depends on AWS KMS in either Region.
- C. Use the AWS KMS Hierarchical keyring with branch keys kept in a key store table that is replicated as a global table and protected by a KMS multi-Region key, and rotate branch keys on a schedule in both Regions.
- D. Create a KMS multi-Region primary key in us-east-1 with a replica in us-west-2, and configure an MRK-aware keyring so each Region's application encrypts and decrypts with its local replica of the key.

---

## Question 8 (Domain 1: SDLC Automation)

A retailer serves its storefront through one CloudFront distribution with several cache behaviors, CloudFront Functions for URL rewrites, and origin request policies. Two configuration changes last quarter caused outages — a cache-key change that served one user's cart to another, and a function bug that returned 404s — and both took effect for all users at once.

The team now wants every distribution configuration change to be validated against a small share of real production traffic, with each viewer session sticking to the configuration it first received, before the change is promoted. A CodePipeline pipeline already deploys distribution changes with CloudFormation.

**Which solution meets these requirements while exposing the FEWEST customer requests to a defective configuration change?**

- A. Create a second distribution with the new configuration and use Route 53 weighted records to send 5% of DNS queries to it, promoting by changing the weights and rolling back by setting its weight to zero.
- B. Apply each change to the production distribution during the lowest-traffic hour, with a CloudFormation rollback trigger on a 4XX and 5XX error-rate alarm so that a bad change rolls back automatically.
- C. Use CloudFront continuous deployment to apply each change to a staging distribution, send 5% of traffic to it through a weight-based policy with session stickiness, and promote it when monitoring is clean.
- D. Use CloudFront continuous deployment to apply each change to a staging distribution, send 15% of traffic to it through a weight-based policy with session stickiness, and promote it after the first clean hour.

---

## Question 9 (Domain 2: Configuration Management and IaC)

A company runs a Java application on AWS Elastic Beanstalk with an external Amazon RDS database that is not part of the environment. Releases go out every Thursday, and defects in the last three releases surfaced only after about 2 hours of production traffic, when a nightly reconciliation job started.

The team uses rolling deployments with an additional batch, and every rollback has meant redeploying the previous application version — about 25 minutes for the 40-instance environment. The team wants to switch production back to the previous release as quickly as possible at any time during the first 6 hours after a release. Database schema changes are always backward compatible.

**Which solution meets these requirements with the FASTEST rollback?**

- A. Switch to immutable deployments so that each release launches a fresh set of instances in a temporary Auto Scaling group, and redeploy the previous application version with another immutable deployment if a defect appears.
- B. Deploy each release to a cloned environment, swap environment URLs when it is healthy, keep the old environment running for 6 hours, and swap the URLs back if a defect appears.
- C. Switch to traffic-splitting deployments that send 10% of traffic to new instances for a 30-minute evaluation period before completing, and redeploy the previous application version if a defect appears afterward.
- D. Keep rolling deployments with an additional batch, raise the batch size to 50% so that redeployments finish faster, and redeploy the previous application version if a defect appears after a release.

---

## Question 10 (Domain 3: Resilient Cloud Solutions)

A company's trading platform runs active-passive across us-east-1 and us-west-2. Failing over requires scaling Auto Scaling groups in us-west-2 from 2 to 40 instances, failing over an Aurora global database, and updating Route 53 records — in that order — with a manual approval from the head of trading before traffic moves.

Today the steps live in a wiki and take 70 minutes to perform. The company wants the procedure codified, rehearsed quarterly, and executable from us-west-2 even if control planes in us-east-1 are impaired. The platform team has no in-house workflow code today and would prefer to keep it that way.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Write an SSM Automation runbook in us-west-2 whose steps scale the groups, call the Aurora global database failover API, pause on an `aws:approve` step, and update the Route 53 records, and run it against a staging copy for drills.
- B. Build a Step Functions state machine in us-west-2 that calls the Auto Scaling, RDS, and Route 53 APIs through SDK integrations with a callback task for the approval, and run it against a staging environment for drills.
- C. Codify the wiki procedure as AWS CLI scripts stored in a us-west-2 S3 bucket, and have on-call engineers run them from AWS CloudShell in us-west-2 during drills and real events, recording each run in a ticket.
- D. Create an Amazon Application Recovery Controller Region switch plan with execution blocks for the Auto Scaling groups, the Aurora global database, Route 53, and a manual approval, and run it from us-west-2 for both drills and real events.

---

## Question 11 (Domain 4: Monitoring and Logging)

A streaming company runs its primary stack in us-west-2, and its monitoring account hosts all CloudWatch alarms in us-east-1. Engineers want a reliable "customer-impacting outage" signal for the playback API that pages the incident commander, but not on a single failed probe, a transient network blip, or an outage of one probe location.

The signal must combine two independent views: whether the API's health endpoint is reachable from many internet locations, and whether a scripted login-and-play journey succeeds. The company prefers managed services, and its on-call tooling consumes SNS notifications.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create a Route 53 health check on the health endpoint, which checkers in multiple locations evaluate, and a CloudWatch Synthetics canary in us-east-1 for the journey; alarm on each and page through a composite alarm requiring both.
- B. Run Lambda functions every minute in three Regions that probe the endpoint and run the journey, write results to a DynamoDB global table, and invoke a voting function that publishes to SNS when most probes fail.
- C. Deploy a third-party synthetic monitoring agent on EC2 instances in three Regions, configure it to run both checks, and connect its alerting to SNS through a Lambda function that converts webhooks into SNS messages.
- D. Create Synthetics canaries for the journey in three Regions with an alarm in each, send their state changes to a central EventBridge bus, and have a Lambda function publish to SNS when two of the three alarms are in ALARM.

---

## Question 12 (Domain 1: SDLC Automation)

A company deploys a payments API to Amazon ECS with CodeDeploy blue/green deployments behind an ALB that has a production listener on port 443 and a test listener on port 8443. An integration test suite, packaged as a Lambda function, reliably detects the contract regressions that caused the last four production incidents.

The deployment group uses `CodeDeployDefault.ECSCanary10Percent5Minutes` with alarm-based rollback, yet each regression still produced failed payments during the canary. The team must keep using CodeDeploy for this service because an audit-evidence integration depends on it.

**Which solution meets these requirements while exposing the FEWEST customer requests to a defective release?**

- A. Switch to `CodeDeployDefault.ECSLinear10PercentEvery1Minutes` with the same alarms, so a regression is detected while only 10% of traffic has shifted and the rollback starts sooner.
- B. Keep the canary configuration, but publish the error metric at high resolution and lower the alarm period to 10 seconds so that rollback starts within seconds of the first failed payments.
- C. Switch to `CodeDeployDefault.ECSAllAtOnce` with the same alarms and a 30-minute termination wait, so that a rollback immediately reroutes all traffic back to the original task set.
- D. Register the test function as an `AfterAllowTestTraffic` hook that runs the suite against the test listener and fails the deployment before any production traffic shifts.

---

## Question 13 (Domain 2: Configuration Management and IaC)

A company must patch 6,000 EC2 instances across 60 accounts and 4 Regions. Production instances must use a baseline that approves security updates 7 days after release and installs them on Sundays at 02:00, while development instances use a baseline with no delay and patch daily. Compliance must be reported centrally.

All instances are already managed by Systems Manager, and environment is defined by OU membership. The company has a delegated administrator account for Systems Manager, and a small number of instances run Windows Server.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use StackSets to deploy maintenance windows, patch baselines, and `AWS-RunPatchBaseline` tasks to every account and Region with parameter values per OU, and aggregate compliance with a Resource Data Sync in each account.
- B. From the delegated administrator account, create two Quick Setup patch policies — one per OU with its own baselines and install schedule — targeting all four Regions, and use their central compliance reporting.
- C. Use StackSets to deploy State Manager associations for `AWS-RunPatchBaseline` with schedules per OU and custom baselines per account, and aggregate compliance into a DynamoDB table with a Lambda function.
- D. Run a central Lambda function on EventBridge Scheduler schedules that assumes a role in each account and Region and runs `AWS-RunPatchBaseline` with `SendCommand`, choosing baselines by tag and writing results to S3.

---

## Question 14 (Domain 5: Incident and Event Response)

A logistics company's 40 microservices publish order events to a custom EventBridge event bus in us-east-1, where rules route them to consumers. Event ingestion must continue if EventBridge in us-east-1 becomes impaired: publishers must fail over automatically, and events that reach us-east-1 must also be available in us-west-2 for recovery.

Consumers in both Regions are idempotent. Publishers use the AWS SDK and can be updated once to use a new endpoint. A CloudWatch alarm that detects failed `PutEvents` calls in us-east-1 already exists.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create an EventBridge global endpoint with the us-east-1 bus as primary and an identical us-west-2 bus as secondary, a Route 53 health check based on the existing alarm, and event replication enabled.
- B. Update all publishers to call `PutEvents` on both Regions' buses for every event, and add deduplication logic in every consumer to discard the second copy of each event it receives.
- C. Front the buses with API Gateway APIs in both Regions that call `PutEvents`, and use Route 53 failover records with a health check on the us-east-1 API so that publishers fail over through DNS.
- D. Add retry logic to every publisher that switches to the us-west-2 bus after three failed `PutEvents` calls, and create a rule on the us-east-1 bus that forwards all events to the us-west-2 bus.

---

## Question 15 (Domain 6: Security and Compliance)

A company runs an Amazon RDS for MySQL primary in eu-west-1 with a cross-Region read replica in eu-central-1 for disaster recovery. The application's database credentials are in a Secrets Manager secret in eu-west-1 that rotates every 30 days through a rotation Lambda function.

The DR runbook requires that, after the replica is promoted, the eu-central-1 application can retrieve current credentials locally even if eu-west-1 is unavailable, and that rotation resumes in eu-central-1. Secrets must be encrypted with a customer managed KMS key in the Region where they are stored.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Replicate the secret to eu-central-1 with a customer managed key in that Region; during DR, promote the replica to a standalone secret and configure rotation on it against the promoted database.
- B. Create a separate secret in eu-central-1, and add an EventBridge rule on the primary secret's rotation events that invokes a Lambda function to copy each new password into the eu-central-1 secret.
- C. Store a copy of the credentials as a SecureString parameter in eu-central-1 that a Lambda function updates after each rotation, and switch the application to read Parameter Store during DR.
- D. Switch the application to IAM database authentication in both Regions so that no secret is needed, creating database users mapped to IAM roles and generating authentication tokens in the application.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | C | Domain 3: Resilient Cloud Solutions | MOST cost-effective |
| 2 | C | Domain 1: SDLC Automation | LEAST operational overhead |
| 3 | A | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 4 | D | Domain 4: Monitoring and Logging | FASTEST detection |
| 5 | C | Domain 3: Resilient Cloud Solutions | FASTEST recovery |
| 6 | B | Domain 5: Incident and Event Response | LEAST operational overhead |
| 7 | D | Domain 6: Security and Compliance | LEAST operational overhead |
| 8 | C | Domain 1: SDLC Automation | FEWEST customer requests exposed |
| 9 | B | Domain 2: Configuration Management and IaC | FASTEST rollback |
| 10 | D | Domain 3: Resilient Cloud Solutions | LEAST operational overhead |
| 11 | A | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 12 | D | Domain 1: SDLC Automation | FEWEST customer requests exposed |
| 13 | B | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 14 | A | Domain 5: Incident and Event Response | LEAST operational overhead |
| 15 | A | Domain 6: Security and Compliance | LEAST operational overhead |

---

### Question 1

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** Amazon Aurora Global Database, Amazon EC2 Auto Scaling, Amazon Route 53

**Socratic walkthrough**
1. *Which objective drives the database design: RPO or RTO?* — RPO. Aurora global database storage replication typically lags by under a second, whether or not the secondary has instances.
2. *Does the 30-minute RTO leave time to add a DB instance?* — Yes. About 12 minutes for the instance, in parallel with about 9 for the app tier.
3. *So what are you paying for in each option while nothing is wrong?* — DB instances and EC2 instances that sit idle.

**Why C is correct:** A headless secondary cluster receives storage-level replication but runs no DB instances, so you pay for replicated storage and I/O, not compute. At failover, you add an instance, fail over the global database, scale the Auto Scaling group from zero, and switch Route 53 — comfortably inside 30 minutes.

**Why the others fail the constraint:**
- **A.** Meets the objectives, but pays continuously for a reader instance, six EC2 instances, and an ALB in us-west-2.
- **B.** Meets the objectives with near-zero RTO, but doubles the entire production footprint.
- **D.** Meets the objectives, but pays for two production-sized readers that the RTO doesn't require.

**Exam reflex:** With an RTO measured in tens of minutes, a headless Aurora secondary is the cheapest pilot light that still keeps RPO in seconds.

---

### Question 2

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS CodePipeline, AWS CloudFormation, Amazon S3 (artifact stores)

**Socratic walkthrough**
1. *Can one pipeline deploy to other Regions?* — Yes. Actions can target another Region, and CodePipeline copies artifacts to an artifact store in each Region it uses.
2. *Where do per-Region approvals fit?* — As manual approval actions in each Region's stage.
3. *What custom code or extra pipelines does C need?* — None.

**Why C is correct:** Cross-Region CloudFormation actions inside the existing pipeline keep one release flow with ordered stages and per-Region approvers. CodePipeline handles artifact replication to the per-Region stores, so nothing new needs to be built.

**Why the others fail the constraint:**
- **A.** Works, but three pipelines, artifact replication, and event chaining are more infrastructure to keep consistent.
- **B.** Works, but scripted deploys need packaging, credentials, and error handling that the native action already provides.
- **D.** Works, but adds StackSet administration and custom Lambda code just to sequence Regions.

**Exam reflex:** Multi-Region deployments with per-Region gates mean one pipeline with cross-Region actions.

---

### Question 3

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation (stack policies, DeletionPolicy, UpdateReplacePolicy), Amazon Aurora/DynamoDB, Amazon S3

**Socratic walkthrough**
1. *What blocks a stack update from replacing a specific resource?* — A stack policy that denies `Update:Replace` on that logical ID.
2. *How do you allow an approved exception for one update?* — Pass a temporary stack policy for that update only; the original policy then applies again.
3. *What protects the data if a resource is removed from the template?* — `DeletionPolicy` (and `UpdateReplacePolicy` for replacements) set to `Retain` or `Snapshot`.

**Why A is correct:** Stack policies and resource policy attributes are declarative CloudFormation features with no code or infrastructure. Together they block replacement or deletion, provide a per-update override mechanism, and preserve data even when someone bypasses or misconfigures them.

**Why the others fail the constraint:**
- **B.** Works, but you build and maintain a hook function and an approval-tag workflow.
- **C.** Works, but detaching SCPs for each approval is a heavy, account-wide, error-prone process.
- **D.** Works, but it's custom change set parsing and only covers pipeline-driven updates.

**Exam reflex:** To protect stateful resources from CloudFormation updates, combine stack policies with DeletionPolicy and UpdateReplacePolicy.

---

### Question 4

**Answer:** D  
**Constraint:** FASTEST detection · **Services:** Amazon Route 53 health checks, Amazon CloudWatch alarms, Elastic Load Balancing

**Socratic walkthrough**
1. *How long does each signal take to flip?* — It's roughly interval × threshold, plus any metric period.
2. *What are the numbers?* — A: 15 s × 2 = 30 s via the ALB. B: two 1-minute alarm periods = 2 min. C: 30 s × 3 (default threshold) = 90 s. D: 10 s × 1 = about 10 s.
3. *Does the deep endpoint catch what matters?* — Yes. It checks the database and cache, not just the load balancer.

**Why D is correct:** A Route 53 HTTPS health check with the fast (10-second) interval and a failure threshold of 1 marks the endpoint unhealthy after one failed interval. Route 53 checkers in multiple locations evaluate it, so failover begins far sooner than any metric-based signal.

**Why the others fail the constraint:**
- **A.** Works, but needs two failed 15-second ALB checks — about three times slower.
- **B.** Works, but an alarm-based health check inherits the alarm's two 1-minute evaluation periods.
- **C.** Works, but standard intervals with the default threshold, plus a calculated parent, add latency rather than remove it.

**Exam reflex:** For the fastest DNS failover detection, use a fast-interval Route 53 health check with a low failure threshold on a deep health endpoint.

---

### Question 5

**Answer:** C  
**Constraint:** FASTEST recovery · **Services:** Amazon S3 Multi-Region Access Points, S3 Replication (RTC), S3 failover controls

**Socratic walkthrough**
1. *What made the last failover slow?* — Changes and restarts at 200 branches.
2. *Which option needs no client-side action at failover time?* — The Multi-Region Access Point, whose routing is changed centrally.
3. *Is the data already in us-east-2?* — Yes. Two-way replication with RTC keeps both buckets current.

**Why C is correct:** Multi-Region Access Point failover controls let you flip which Region is active with one control-plane change; clients keep using the same access point alias and SigV4A signing. Reads and writes move to us-east-2 within minutes, with no DNS TTLs, restarts, or client logic.

**Why the others fail the constraint:**
- **A.** Works, but still restarts services on 200 machines — faster than by hand, but slower than a routing flip.
- **B.** Works, but adds a new API layer and depends on health-check detection and DNS TTL expiry.
- **D.** Works, but every client must first hit three failures before switching, per process, every time.

**Exam reflex:** For S3 active-passive across Regions, Multi-Region Access Point failover controls give the fastest, client-transparent failover.

---

### Question 6

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** Amazon S3 Replication (RTC), Amazon SNS, S3 Batch Replication

**Socratic walkthrough**
1. *Does S3 emit events for individual replication failures and SLA misses?* — Yes. With RTC and replication metrics enabled, S3 can publish replication failed and missed-threshold event notifications.
2. *Is there a native way to retry failed objects?* — Yes. S3 Batch Replication replicates objects whose replication status is failed.
3. *What's left to build?* — Nothing beyond configuration.

**Why B is correct:** Per-object replication event notifications go straight to the existing SNS topic, so the team learns about failures within minutes. Batch Replication re-replicates exactly the failed objects, using the existing replication configuration, permissions, and KMS keys.

**Why the others fail the constraint:**
- **A.** Works, but detection is daily, and inventory diffs plus a copy function are custom code over 2 billion objects.
- **C.** Works, but replaces a managed feature with a custom copier you must scale and monitor.
- **D.** Works, but metrics alert in aggregate without object keys, and manual syncs don't scale.

**Exam reflex:** For S3 replication failures, use replication event notifications for alerts and Batch Replication for remediation.

---

### Question 7

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS KMS (multi-Region keys), Amazon DynamoDB global tables, AWS Database Encryption SDK

**Socratic walkthrough**
1. *Why does client-side encryption complicate DR?* — Ciphertext replicates with the global table, but it can only be decrypted with the key that wrapped it.
2. *What makes two Regional keys interchangeable?* — Multi-Region keys share key material and key ID.
3. *What does the SDK need?* — An MRK-aware keyring that uses the local replica.

**Why D is correct:** A multi-Region primary and replica decrypt each other's ciphertext. Each Region encrypts and decrypts with its local key, so failover needs no keyring switch, outage-time re-encryption, or extra key management.

**Why the others fail the constraint:**
- **A.** Works, but keyring switching during an outage and post-failback re-encryption jobs are operationally heavy.
- **B.** Works, but you own raw key material — rotation, protection, and access control — outside KMS.
- **C.** Works, but adds a key store table, branch key lifecycle, and rotation for a problem MRKs already solve.

**Exam reflex:** Client-side encrypted data that must be decrypted in another Region during an outage calls for KMS multi-Region keys.

---

### Question 8

**Answer:** C  
**Constraint:** FEWEST customer requests exposed · **Services:** Amazon CloudFront (continuous deployment), AWS CodePipeline, AWS CloudFormation

**Socratic walkthrough**
1. *How precisely does each option control the share of traffic?* — A CloudFront policy controls it per request; DNS weights are distorted by resolver caching.
2. *What happens to clients once a change is reverted?* — CloudFront stops routing to staging; DNS clients keep cached answers until TTLs expire.
3. *Between 5% and 15%, which exposes fewer requests if the change is bad?* — 5%.

**Why C is correct:** Continuous deployment routes a precise 5% of requests — sticky per viewer session — to the staging distribution's configuration. A defect affects only that slice until the policy is disabled, and promotion copies the staging configuration to production.

**Why the others fail the constraint:**
- **A.** Works, but resolver caching makes the exposed share unpredictable, and rollback waits for TTLs.
- **B.** Works, but 100% of requests are exposed until the alarm and stack rollback complete.
- **D.** Works, but exposes three times as many requests during validation.

**Exam reflex:** Use CloudFront continuous deployment with a small weight for CDN configuration canaries; DNS weighting is imprecise.

---

### Question 9

**Answer:** B  
**Constraint:** FASTEST rollback · **Services:** AWS Elastic Beanstalk, Amazon Route 53 (CNAME swap), Amazon RDS

**Socratic walkthrough**
1. *Where must the previous release be running for a rollback to be fast?* — In an environment that still exists 2–6 hours after release.
2. *Which deployment type keeps the old fleet alive after completion?* — Only blue/green with a URL swap; immutable and traffic splitting discard old instances once they finish.
3. *Why is the external database important?* — It lets both environments share data safely.

**Why B is correct:** Swapping environment URLs moves traffic by updating the environments' CNAMEs. Swapping back points users to the still-running previous environment within the DNS TTL — no instances to launch and no version to deploy.

**Why the others fail the constraint:**
- **A.** Works, but rollback is a full immutable deployment of 40 new instances.
- **C.** Works, but the defect appears after the evaluation window, so rollback is a redeployment.
- **D.** Works, but it's still a redeployment, just with larger batches and less capacity during it.

**Exam reflex:** If defects surface hours later, keep the old environment running (blue/green) so rollback is a swap.

---

### Question 10

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** Amazon Application Recovery Controller (Region switch), Amazon Aurora Global Database, Amazon EC2 Auto Scaling

**Socratic walkthrough**
1. *What do A, B, and C have in common?* — Your team writes and maintains the orchestration logic.
2. *Is there a managed service that orchestrates multi-step Region failover?* — Yes. ARC Region switch plans, with prebuilt execution blocks.
3. *What about running it during a us-east-1 impairment?* — Region switch plans are designed to be executed from the recovery Region.

**Why D is correct:** A Region switch plan declares the ordered steps as managed execution blocks — scaling, Aurora global database failover, Route 53 changes — plus a manual approval. The plan can be exercised regularly and run from us-west-2 without custom code.

**Why the others fail the constraint:**
- **A.** Works, but you author, test, and maintain each API call and its error handling in the runbook.
- **B.** Works, but it's custom workflow code — exactly what the team wants to avoid.
- **C.** Works, but manual scripts are slow and error-prone under pressure.

**Exam reflex:** For orchestrated multi-Region failover with approvals, consider ARC Region switch before building runbooks.

---

### Question 11

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** Amazon Route 53 health checks, Amazon CloudWatch Synthetics, CloudWatch composite alarms

**Socratic walkthrough**
1. *Which managed check already has built-in multi-location consensus?* — A Route 53 health check, evaluated by checkers in many locations.
2. *Where do Route 53 health check metrics live?* — In us-east-1, which is the monitoring account's Region.
3. *How do you require two independent signals to agree?* — A composite alarm with an AND rule over two alarms in the same Region.

**Why A is correct:** The Route 53 health check covers broad reachability with built-in consensus, and the Synthetics canary covers the user journey. Both alarms live in us-east-1, so a composite alarm pages only when both agree — all managed, with no code.

**Why the others fail the constraint:**
- **B.** Works, but means probes, a global table, and voting logic to build and run.
- **C.** Works, but means EC2 agents, licensing, and a webhook bridge.
- **D.** Works, but composite alarms can't span Regions, so cross-Region voting becomes custom Lambda code.

**Exam reflex:** Composite alarms require same-account, same-Region alarms — design the signal so that both inputs land there.

---

### Question 12

**Answer:** D  
**Constraint:** FEWEST customer requests exposed · **Services:** AWS CodeDeploy (ECS), Elastic Load Balancing (test listener), AWS Lambda

**Socratic walkthrough**
1. *Can any traffic-shifting configuration expose zero customers to a defect?* — No. Every canary or linear step shifts some real traffic.
2. *Where can the suite run before production traffic shifts?* — Against the test listener, which already routes to the replacement task set.
3. *Which ECS hook runs at that moment?* — `AfterAllowTestTraffic`.

**Why D is correct:** Once test traffic is routed to the replacement task set, CodeDeploy invokes the hook function. If the suite finds a regression, the hook reports failure and the deployment rolls back before any production request reaches the new version.

**Why the others fail the constraint:**
- **A.** Works, but still sends 10% of real traffic to the defect before the alarm reacts.
- **B.** Works, but faster alarms still need failed payments to detect anything.
- **C.** Works, but exposes 100% of traffic until the alarm fires.

**Exam reflex:** To catch detectable regressions with zero customer exposure, test through the test listener in `AfterAllowTestTraffic`.

---

### Question 13

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** AWS Systems Manager Patch Manager, Systems Manager Quick Setup, AWS Organizations

**Socratic walkthrough**
1. *What varies by environment?* — The baseline and the schedule.
2. *Which feature deploys baselines, schedules, and targeting across an organization's OUs and Regions in one configuration?* — Quick Setup patch policies.
3. *How is compliance aggregated?* — Patch policies provide organization-level compliance reporting.

**Why B is correct:** Two patch policies — production and development — each define baselines per OS, the scan and install schedule, and target OUs and Regions. Quick Setup deploys and maintains the supporting resources in every account and Region, including new accounts in those OUs.

**Why the others fail the constraint:**
- **A.** Works, but you maintain StackSet templates and per-account aggregation.
- **C.** Works, but per-account baselines and custom aggregation are duplicated configuration.
- **D.** Works, but it's a homegrown scheduler with cross-account role management and result handling.

**Exam reflex:** For organization-wide patching with different baselines per OU, use Quick Setup patch policies.

---

### Question 14

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** Amazon EventBridge (global endpoints), Amazon Route 53 health checks, Amazon CloudWatch

**Socratic walkthrough**
1. *Is there a managed failover for custom event buses?* — Yes. EventBridge global endpoints fail publishers over between two Regions.
2. *What drives the failover decision?* — A Route 53 health check, which can be based on the existing CloudWatch alarm.
3. *How do events reach the secondary Region?* — Event replication, which consumers handle idempotently.

**Why A is correct:** Publishers send to the global endpoint once, and EventBridge routes to the healthy Region's bus based on the health check. Replication sends events to the secondary bus, so recovery data already exists in us-west-2, with no client-side retry or dual-write logic.

**Why the others fail the constraint:**
- **B.** Works, but every publisher and consumer changes, and dual writes double costs and complexity.
- **C.** Works, but adds a proxy tier to run and scale, plus DNS failover timing.
- **D.** Works, but custom failover logic in 40 services and a forwarding rule duplicate what global endpoints provide.

**Exam reflex:** For Regional resilience of custom event buses, use EventBridge global endpoints with replication.

---

### Question 15

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS Secrets Manager (replica secrets), Amazon RDS (cross-Region read replica), AWS KMS

**Socratic walkthrough**
1. *Can Secrets Manager keep a copy of a secret in another Region automatically?* — Yes. Replica secrets are updated whenever the primary changes, including after rotation.
2. *Can each replica use its own Region's KMS key?* — Yes. Each replica specifies a key in its Region.
3. *What happens at DR?* — Promote the replica to a standalone secret, then configure rotation in that Region.

**Why A is correct:** Replication keeps current credentials available locally with no code, encrypted under a eu-central-1 key. Promotion turns the replica into a normal secret that can rotate against the newly promoted database.

**Why the others fail the constraint:**
- **B.** Works, but it's a custom sync pipeline duplicating built-in replication.
- **C.** Works, but adds a second store, sync code, and an application change at DR time.
- **D.** Works, but requires application rewrites and IAM-user mapping in both databases.

**Exam reflex:** For DR of secrets, use replica secrets and promote at failover.

---

## Scoring & Performance Breakdown

| Score | Band | Next step |
|---|---|---|
| **14–15 / 15** | Exam ready | Proceed to Practice Exam 4. |
| **12–13 / 15** | Passing | For each miss, re-read the distractor breakdown and note which constraint signal you overlooked. |
| **≤ 11 / 15** | Needs remediation | Use the domain column in the key table to pick study guides; retake this exam in a week. |
