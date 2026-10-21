# AWS DevOps Engineer - Professional (DOP-C02) Practice Exam 2: Configuration Drift & Infrastructure Orchestration

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes (≈5 minutes per question; the real exam allows ≈2.4)  
**Passing Target:** 12 / 15 (80%)  
**Domains:** D1 ×3 · D2 ×3 · D3 ×2 · D4 ×2 · D5 ×2 · D6 ×3

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Answer all 15 before opening the answer key at the end.

---

## Question 1 (Domain 2: Configuration Management and IaC)

A company operates 140 AWS accounts in AWS Organizations, grouped into three workload OUs. Every account in those OUs must contain a standard set of IAM roles — break-glass, security audit, and CI deployment — defined in one CloudFormation template. Today a platform engineer runs a script after each account is created, and roles drift: accounts moved out of the OUs keep roles they should no longer have. The company does not use AWS Control Tower.

About 10 accounts are created each month. The platform team already has a delegated administrator account for CloudFormation StackSets, and most of the roles trust a centrally managed SAML identity provider.

The roles must appear in every current and future account in the three OUs within minutes of the account joining, and must be removed automatically when an account leaves those OUs.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use a self-managed StackSet whose execution role is created by an account-vending script, and add an EventBridge rule in the management account on Organizations account-creation and move events that invokes a Lambda function to add or remove stack instances.
- B. From the delegated administrator account, create a service-managed StackSet that targets the three OUs with automatic deployment enabled and stack retention on account removal disabled, deploying the existing role template to a single Region because IAM roles are global.
- C. Create a CodePipeline pipeline per OU with one CloudFormation action per account, and add a Lambda function triggered by Organizations events that regenerates each pipeline definition whenever accounts are created or moved.
- D. Deploy the roles with a service-managed StackSet targeting the three OUs without automatic deployment, and schedule a Lambda function every 5 minutes that compares OU membership with stack instances and creates or deletes instances.

---

## Question 2 (Domain 1: SDLC Automation)

A team deploys a CDK application from a tools account to a production account with CDK Pipelines. The application creates an S3 bucket, a DynamoDB table, Lambda functions, API Gateway REST APIs, CloudWatch alarms, and the IAM execution roles that its functions use. Every resource name starts with `orders-`.

The production account was bootstrapped years ago with the default `cdk bootstrap --trust <tools-account>` command. A security review found that the CloudFormation execution role created by bootstrapping has `AdministratorAccess`. The security team requires the permissions used to deploy this application to production to be as narrow as possible, and the pipeline must keep its normal `cdk deploy` behavior and asset publishing.

Developers use CDK v2 with TypeScript, and the team plans to add a second Region next year.

**Which solution meets these requirements with the LEAST privilege?**

- A. Re-bootstrap production with `--trust` and set `--cloudformation-execution-policies` to the AWS managed `PowerUserAccess` policy plus a customer managed policy that allows IAM actions on roles named `orders-*`.
- B. Keep the default bootstrap, but create a separate `orders-deployer` role with a scoped policy and configure the application's synthesizer to use it as the CloudFormation execution role for every deployment.
- C. Re-bootstrap production with `--trust <tools-account>` and `--cloudformation-execution-policies` set to a customer managed policy, created beforehand, that is limited to the actions the stack needs on `orders-*` resources.
- D. Re-bootstrap production with trust granted to every account in the tools OU and the same scoped customer managed execution policy, so future pipelines in other tools accounts deploy without re-bootstrapping.

---

## Question 3 (Domain 6: Security and Compliance)

A company records all supported resource types continuously with AWS Config in 60 accounts and aggregates them in a security account. Batch analytics accounts launch and terminate thousands of EC2 instances and network interfaces every hour, and those two resource types account for 82% of the Config bill. Compliance reviewers need only a daily view of those two types through the aggregator, but security groups, IAM roles, and S3 buckets must still be recorded as soon as they change because several detective controls use change-triggered rules.

The recorder settings are deployed through a CloudFormation StackSet. Several analytics accounts use Spot Instances with capacity rebalancing.

The company wants to reduce its Config costs while keeping these compliance and detection requirements.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Keep continuous recording for every resource type, and convert the change-triggered rules that evaluate EC2 instances and network interfaces into periodic rules that run once every 24 hours to cut evaluation charges.
- B. In the StackSet, keep continuous recording as the default and add recording-mode overrides that record EC2 instances and network interfaces with a daily recording frequency.
- C. Keep continuous recording for every resource type, and stop delivering configuration snapshots and configuration history files to the S3 bucket, relying on the aggregator for compliance reviews.
- D. Keep continuous recording for every resource type, and move the detective rules from per-account StackSet deployments into organization conformance packs managed from the delegated administrator account.

---

## Question 4 (Domain 3: Resilient Cloud Solutions)

A ticketing company runs a Java application in an EC2 Auto Scaling group behind an Application Load Balancer. A new instance needs about 7 minutes before it can serve traffic because it downloads a 30 GB seating dataset from S3 and builds in-memory indexes; the data changes once a day. Demand spikes are unpredictable — a popular show can go on sale with 10 minutes' notice — and new capacity must serve traffic within 90 seconds of a scale-out decision.

The company runs 60 instances at all times to absorb spikes, although steady-state load needs only 15. The application can persist its indexes to the EBS root volume, and testing shows a stopped instance with persisted indexes serves traffic within 80 seconds of being started. An engineer suggests moving to a larger instance family.

The company wants to meet the 90-second target at the lowest cost.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Keep 15 instances in service and add a warm pool of 45 instances in the `Running` state, with a lifecycle hook that completes after the dataset is loaded, so that scale-out moves fully initialized instances into service.
- B. Keep 15 instances in service and add a warm pool of 45 instances in the `Stopped` state, with a lifecycle hook that completes after the dataset is loaded and the indexes are persisted to disk.
- C. Keep all 60 instances in service, purchase a Compute Savings Plan for the full fleet, and use target tracking to maintain at least 45 instances of headroom above current demand at all times.
- D. Keep 15 On-Demand Instances and 45 Spot Instances in service at all times through a mixed instances policy, so spare capacity is always initialized but billed at a lower hourly rate.

---

## Question 5 (Domain 4: Monitoring and Logging)

A company manages 200 security groups across 15 accounts exclusively with CloudFormation. Every stack is deployed with the same CloudFormation service role, `cfn-deployer`, which already exists in each account. During incidents, engineers sometimes edit security group rules in the console, and the change persists unnoticed until the next stack update reverts it or fails.

The security team wants an alert within about a minute whenever any security group rule is changed by anything other than CloudFormation. A central SNS topic in the security account is already subscribed by the paging tool, and every account already forwards selected events to a central EventBridge bus. The security groups belong to EKS clusters and ECS services.

**Which solution meets these requirements with the FASTEST detection?**

- A. In each account, create an EventBridge rule that matches CloudTrail management events for security group rule changes, excluding events whose session issuer is the `cfn-deployer` role, and forward them to the central bus that publishes to the SNS topic.
- B. Deploy the `cloudformation-stack-drift-detection-check` AWS Config managed rule to every account at its shortest periodic frequency, and send its noncompliance notifications through the central bus to the SNS topic.
- C. Run a scheduled Lambda function every 15 minutes in each account that calls `DetectStackDrift` on every stack that contains security groups and publishes any drifted results to the SNS topic.
- D. Create an AWS Config custom rule triggered by configuration changes to `AWS::EC2::SecurityGroup` that compares each group's rules with its stack's template and publishes mismatches to the SNS topic.

---

## Question 6 (Domain 2: Configuration Management and IaC)

A company has 3,000 EC2 instances across 40 accounts in AWS Organizations. Application teams own their instance profiles and IAM roles, and any change to those roles requires a change request approved by the owning team; there are 40 teams. The platform team wants every instance managed by AWS Systems Manager for patching and Session Manager access.

All instances already run a recent SSM Agent and require IMDSv2. The platform team has a delegated administrator account for Systems Manager, and about 300 instances run Windows. New accounts join the organization monthly and must be covered automatically.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use a StackSet to deploy a Lambda function that attaches `AmazonSSMManagedInstanceCore` to every instance role, raise the 40 change requests, and add an AWS Config rule that reattaches the policy whenever teams recreate their roles.
- B. Use Quick Setup host management to create a Systems Manager instance profile in every account and swap it onto each instance with `ReplaceIamInstanceProfileAssociation`, merging each team's permissions into the new roles through change requests.
- C. Register every instance as a hybrid managed node by creating activation codes per account with a StackSet, distributing them through the teams' configuration management tools, and re-registering the SSM Agent with each code.
- D. From the delegated administrator account, use Quick Setup to enable Default Host Management Configuration across the organization in every AWS Region that the company uses so Systems Manager manages instances without touching their instance profiles.

---

## Question 7 (Domain 5: Incident and Event Response)

A company operates 1,200 Amazon Linux 2023 instances that Systems Manager already manages. During incidents, operators sometimes edit `/etc/ssh/sshd_config` or stop `auditd`, and those changes persist for weeks. Compliance requires that any drift from the approved configuration be reverted within 30 minutes and that each instance's compliance status be visible centrally. The approved configuration already exists as an Ansible playbook in an S3 bucket.

A Resource Data Sync already sends Systems Manager compliance data to a central S3 bucket that auditors query with Athena. Half of the fleet runs in Auto Scaling groups.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create an AWS Config custom rule that runs every 30 minutes, uses Run Command to check the files and service state on each instance, and triggers an SSM Automation remediation that reruns the playbook when it finds drift.
- B. Create an EventBridge Scheduler schedule that runs every 30 minutes and invokes a Lambda function, which runs the playbook through Run Command on all instances and records results with `PutComplianceItems`.
- C. Bake a cron job into the AMI that reapplies the playbook locally every 30 minutes and ships its output to CloudWatch Logs, with a metric filter and an alarm that report noncompliant instances.
- D. Create a State Manager association that runs the playbook with the `AWS-ApplyAnsiblePlaybooks` document on a 30-minute rate schedule, targeting the fleet by tag, so association compliance flows into the existing sync.

---

## Question 8 (Domain 1: SDLC Automation)

A company builds a golden Amazon Linux AMI every month and distributes it to four workload accounts in two Regions each. Today a CodeBuild project runs Packer, a Lambda function copies and shares the AMI, and engineers update the launch template in each account and Region by hand — eight updates a month, two of which were missed last quarter. Amazon Inspector is activated in every account, and the security team wants each new image scanned for vulnerabilities as part of the build.

Every Auto Scaling group uses its launch template's default version, and an existing weekly instance refresh rolls new AMIs into the groups. The images include a 4 GB machine learning runtime.

The company wants new AMIs to reach every launch template automatically.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Keep Packer in CodeBuild, and replace the manual updates with a Lambda function in each account and Region, triggered by an EventBridge rule on AMI share events, that creates a new default launch template version with the shared AMI ID.
- B. Replace Packer with an EC2 Image Builder pipeline that has Inspector image scanning enabled and a distribution configuration that shares the AMI with the four accounts in both Regions and sets it as each launch template's new default version.
- C. Replace Packer with an EC2 Image Builder pipeline, and add an EventBridge rule on image state changes that starts a Step Functions workflow to copy the AMI, update every launch template, and wait for Inspector findings in each account.
- D. Keep Packer in CodeBuild with an added Inspector scan step, and have the build assume a role in each account and Region to copy the AMI and create a new default launch template version with the copied AMI ID.

---

## Question 9 (Domain 6: Security and Compliance)

A company wants 300 developers to provision approved data stacks — an S3 bucket encrypted with a customer managed KMS key, a DynamoDB table, and an RDS for PostgreSQL instance — in their team accounts. Each stack is defined in a CloudFormation template owned by the security team, with allowed instance classes and mandatory encryption settings. Developers must be able to launch, update, and terminate their own copies of the approved stacks.

Developers sign in through IAM Identity Center with a `Developer` permission set. The templates are stored in an S3 bucket in a shared account. The security team wants developers to hold the minimum permissions needed for this workflow.

**Which solution meets these requirements with the LEAST privilege?**

- A. Publish the templates as AWS Service Catalog products in a portfolio shared to the team accounts, add a launch constraint role scoped to the products' resources, and grant the `Developer` permission set only Service Catalog end-user permissions.
- B. Grant the `Developer` permission set `PowerUserAccess` with a permissions boundary that allows only S3, DynamoDB, RDS, and KMS actions, and enforce encryption and instance classes with AWS Config rules that remediate noncompliant resources.
- C. Grant the `Developer` permission set `AWSCloudFormationFullAccess` and `iam:PassRole` on an administrator deployment role in each account, and instruct developers to launch only the approved templates from the shared bucket.
- D. Grant the `Developer` permission set full access to S3, DynamoDB, RDS, and KMS on resources tagged with the team's name, and require that every stack change uses the approved templates through a pull-request review process.

---

## Question 10 (Domain 2: Configuration Management and IaC)

A company's monolithic CloudFormation stack contains 400 resources owned by four teams, including 12 DynamoDB tables and 3 RDS instances that hold production data. The company wants to split it into four team-owned stacks so that each team can deploy independently. Stateful resources must not be replaced or deleted, there can be no downtime, and logical IDs in the new stacks must follow a new naming convention.

A single CodePipeline pipeline deploys the stack today and will be split into four pipelines afterward. Several Lambda functions in the stack are packaged as container images.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Add `DeletionPolicy: Retain` to every moving resource and update the stack, remove those resources from the source template, then import them into four new stacks with `IMPORT` change sets that use the new logical IDs.
- B. Use the CloudFormation IaC generator to scan the account and generate four templates for the moving resources, then retain and remove the resources from the source stack and import them into new stacks from the generated templates.
- C. Use CloudFormation stack refactoring to move the resources from the source stack into four new stacks that the refactor operation creates, supplying a logical ID mapping so the renamed resources keep their existing physical resource IDs.
- D. Build a Step Functions workflow that automates the retain, remove, and import sequence for each team's resources in turn, validating every change set before execution and restoring the source template if an import fails.

---

## Question 11 (Domain 3: Resilient Cloud Solutions)

A company runs a stateless web application on an EC2 Auto Scaling group across three Availability Zones behind an Application Load Balancer. During a recent AZ impairment, latency stayed elevated for 50 minutes until an engineer manually removed the affected AZ from the Auto Scaling group. The company wants traffic to move away from an impaired AZ automatically, without building or maintaining custom tooling, and wants the mechanism exercised regularly.

The group already runs enough capacity to serve peak traffic from any two AZs. The team has CloudWatch alarms on ALB 5XX rates and latency, and the business forbids failover tests on the last business day of each month.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Enable zonal autoshift for the ALB in Amazon Application Recovery Controller, configure practice runs with the existing alarms as outcome alarms and month-end blocked dates, and let AWS shift traffic away from an AZ when it detects an impairment.
- B. Create CloudWatch alarms per AZ on target health and latency that invoke a Lambda function, which deregisters targets in the impaired AZ and removes its subnet from the Auto Scaling group, restoring both after the alarms clear.
- C. Split the application into three single-AZ stacks, each with its own ALB, behind Route 53 weighted records whose health checks evaluate per-AZ alarms, so DNS stops returning an impaired AZ's load balancer.
- D. Create an EventBridge rule for AWS Health events about AZ operational issues that invokes a Lambda function to start an ARC zonal shift on the ALB, and schedule a monthly SSM Automation runbook that tests the function.

---

## Question 12 (Domain 4: Monitoring and Logging)

A company must report which software packages and versions are installed on 8,000 EC2 instances across 50 accounts and 3 Regions. Auditors run SQL queries weekly, and package data can be up to 24 hours old. All instances are already managed by Systems Manager. AWS Config is enabled in every account, recording security-related resource types only.

The audit account already has an S3 bucket and an Athena workgroup for auditors. Package lists change frequently because of nightly CI-driven updates.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Enable AWS Config recording of the `AWS::SSM::ManagedInstanceInventory` resource type in every account and Region, and have auditors run advanced queries against an organization aggregator in the audit account.
- B. Run a daily Run Command execution that lists installed packages on every instance, with a Lambda function that writes the output to a DynamoDB table that auditors query through Athena's DynamoDB connector.
- C. Deploy a third-party CMDB on EC2 in the audit account that connects to every instance over SSH each day and collects package lists into its own SQL database for the auditors.
- D. Create a State Manager inventory association that collects application data daily, and an organization-wide Resource Data Sync that writes inventory to the audit bucket for Athena queries.

---

## Question 13 (Domain 1: SDLC Automation)

A platform group maintains 60 small CloudFormation stacks — one per team repository in GitHub — for shared resources such as SNS topics, IAM roles, and S3 buckets. Teams want changes merged to `main` to deploy automatically, with environment parameters kept in the repository. There is no build step: the templates are plain YAML. An AWS CodeConnections connection to the GitHub organization already exists.

Today an engineer applies changes from a laptop with the AWS CLI. The group expects 120 stacks by next year, and some repositories also contain Terraform modules owned by another team.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create a CodePipeline pipeline per repository with a GitHub source action that uses the connection and a CloudFormation deploy action that reads a template configuration file for the parameters.
- B. Add a GitHub Actions workflow to each repository that assumes an IAM role through OIDC and runs a CloudFormation deploy step with the parameters file on every push to `main`.
- C. Enable CloudFormation Git sync for each stack through the existing connection, with a deployment file in the repository that names the template and parameters.
- D. Create a CodeBuild project for each repository with a webhook on pushes to `main` that runs `aws cloudformation deploy` with parameter overrides read from a file in the repository.

---

## Question 14 (Domain 6: Security and Compliance)

A company runs 120 Amazon RDS for PostgreSQL and MySQL instances across 20 accounts. Master user passwords are stored as SecureString parameters and rotated manually once a year, and an audit found 40 instances whose passwords no longer match the stored values. The security team now requires automatic rotation at least every 30 days, encryption with each account's customer managed KMS key, and a single place where applications retrieve current credentials.

All instances are deployed with CloudFormation. Some run in private subnets with no internet access and no VPC endpoints.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create a Secrets Manager secret for each instance and configure rotation with the AWS single-user rotation Lambda function, deployed into each VPC with a Secrets Manager interface endpoint so the function can reach both the database and the service.
- B. Keep the SecureString parameters, and create an EventBridge Scheduler schedule that invokes a Lambda function in each VPC every 30 days to generate a new password, update the instance, and write the parameter.
- C. Reference the parameters from the templates through dynamic references, and run a scheduled pipeline every 30 days that generates new passwords, updates the parameters, and runs stack updates for all 120 instances.
- D. Set `ManageMasterUserPassword` with a customer managed `MasterUserSecret` KMS key on each instance in the CloudFormation templates so RDS stores and rotates the password in Secrets Manager, and set the secret's rotation schedule to 30 days.

---

## Question 15 (Domain 5: Incident and Event Response)

A platform team supports 70 accounts across three Regions in which application teams deploy CloudFormation stacks through many tools, including CodePipeline, the CDK CLI, and the console. The team must open a ticket automatically, typically within a minute, whenever any stack in any account enters `UPDATE_ROLLBACK_FAILED` or `ROLLBACK_COMPLETE`.

A central EventBridge bus in the platform account already receives other events from every account and invokes the ticketing system through an API destination. The platform team has a delegated administrator account for StackSets.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use a service-managed StackSet to create an EventBridge rule in every account and Region that matches CloudFormation stack status change events for the two statuses and sends them to the existing central bus, which already invokes ticketing.
- B. Require every deployment tool to set `NotificationARNs` to an SNS topic in its account, and subscribe a Lambda function to each topic that filters for the two statuses and forwards matching notifications to the ticketing system.
- C. Run a Lambda function every minute in the platform account that assumes a role in each account and Region, calls `ListStacks` with a status filter for the two statuses, and opens tickets for stacks it hasn't seen before.
- D. Create an EventBridge rule in every account and Region that sends matching stack status events to a per-account SNS topic, and subscribe the ticketing system's email ingestion address to each of the 210 topics.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | B | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 2 | C | Domain 1: SDLC Automation | LEAST privilege |
| 3 | B | Domain 6: Security and Compliance | MOST cost-effective |
| 4 | B | Domain 3: Resilient Cloud Solutions | MOST cost-effective |
| 5 | A | Domain 4: Monitoring and Logging | FASTEST detection |
| 6 | D | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 7 | D | Domain 5: Incident and Event Response | LEAST operational overhead |
| 8 | B | Domain 1: SDLC Automation | LEAST operational overhead |
| 9 | A | Domain 6: Security and Compliance | LEAST privilege |
| 10 | C | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 11 | A | Domain 3: Resilient Cloud Solutions | LEAST operational overhead |
| 12 | D | Domain 4: Monitoring and Logging | MOST cost-effective |
| 13 | C | Domain 1: SDLC Automation | LEAST operational overhead |
| 14 | D | Domain 6: Security and Compliance | LEAST operational overhead |
| 15 | A | Domain 5: Incident and Event Response | LEAST operational overhead |

---

### Question 1

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation StackSets, AWS Organizations, AWS IAM

**Socratic walkthrough**
1. *What needs to happen when an account joins or leaves an OU?* — Its stack instance has to be created or deleted.
2. *Can StackSets do that by itself?* — Yes. Service-managed StackSets with automatic deployment react to OU membership changes.
3. *What controls removal?* — The retain-stacks-on-account-removal setting. Turning it off deletes the stack, and with it the roles.

**Why B is correct:** A service-managed StackSet uses Organizations-managed permissions, so no per-account execution role is needed. With automatic deployment on the three OUs, it deploys to accounts as they join and, with retention disabled, removes stacks as they leave. There is no code, schedule, or script.

**Why the others fail the constraint:**
- **A.** Works, but self-managed StackSets need an execution role in every account, plus a Lambda function and event plumbing to track membership.
- **C.** Works, but per-account actions and a pipeline-regenerating function are bespoke orchestration.
- **D.** Works, but re-implements automatic deployment with a polling reconciler that the team must maintain.

**Exam reflex:** "Every current and future account in an OU" means a service-managed StackSet with automatic deployment.

---

### Question 2

**Answer:** C  
**Constraint:** LEAST privilege · **Services:** AWS CDK (CDK Pipelines), AWS CodePipeline, AWS CloudFormation

**Socratic walkthrough**
1. *Which role actually creates the resources?* — The CloudFormation execution role created by bootstrapping.
2. *Which bootstrap flag controls its permissions?* — `--cloudformation-execution-policies`.
3. *What else defines privilege?* — Who is trusted to use the bootstrap roles (`--trust`) and whether an admin-level role remains assumable.

**Why C is correct:** Re-bootstrapping with a scoped execution policy replaces `AdministratorAccess` on the role that CDK deployments use, and trust stays limited to the single tools account. `cdk deploy` and asset publishing work unchanged.

**Why the others fail the constraint:**
- **A.** Works, but `PowerUserAccess` grants nearly every non-IAM action in the account.
- **B.** Works, but the default `AdministratorAccess` execution role still exists and stays assumable through the trusted tools account.
- **D.** Works, but extends trust to every account in the tools OU.

**Exam reflex:** Least-privilege CDK deployments: bootstrap with a custom `--cloudformation-execution-policies` and the narrowest `--trust`.

---

### Question 3

**Answer:** B  
**Constraint:** MOST cost-effective · **Services:** AWS Config, AWS CloudFormation StackSets, AWS Organizations

**Socratic walkthrough**
1. *What drives the Config bill?* — The number of configuration items recorded, which is enormous for high-churn types.
2. *Can recording frequency be set per resource type?* — Yes. The recorder supports continuous or daily recording with per-type overrides.
3. *Does daily recording break the change-triggered security controls?* — No. Security groups, IAM, and S3 remain continuous.

**Why B is correct:** Overriding only the two high-churn types to daily recording collapses thousands of configuration items per hour into at most one per resource per day. Everything that change-triggered rules depend on keeps recording continuously, and the StackSet already distributes the recorder settings.

**Why the others fail the constraint:**
- **A.** Works, but rule evaluations aren't the main cost; configuration-item volume is unchanged.
- **C.** Works, but S3 delivery adds only a little storage cost; the 82% driver remains.
- **D.** Works, but changes how rules are deployed, not what is recorded.

**Exam reflex:** For high-churn resources where daily state is enough, use daily recording-mode overrides.

---

### Question 4

**Answer:** B  
**Constraint:** MOST cost-effective · **Services:** Amazon EC2 Auto Scaling (warm pools), Elastic Load Balancing, Amazon CloudWatch

**Socratic walkthrough**
1. *What must be paid for today to meet 90 seconds?* — 45 extra instances running 24×7.
2. *What does a stopped warm pool instance cost?* — Mainly its EBS storage, not compute.
3. *Does a stopped instance meet the target?* — Yes. With persisted indexes, testing shows it serves within 80 seconds of starting.

**Why B is correct:** A `Stopped` warm pool keeps 45 fully prepared instances on disk. The lifecycle hook ensures each instance persists its indexes before stopping. Scale-out starts them — compute is billed only while they're in service — and they serve within the 90-second target.

**Why the others fail the constraint:**
- **A.** Meets the target, but `Running` warm pool instances are billed for compute around the clock.
- **C.** Meets the target, but keeps paying for 60 instances, only at a discount.
- **D.** Pays for 45 always-on Spot Instances, and interruptions can remove spare capacity right when a sale starts.

**Exam reflex:** Long initialization plus unpredictable spikes plus a cost focus points to a `Stopped` (or `Hibernated`) warm pool.

---

### Question 5

**Answer:** A  
**Constraint:** FASTEST detection · **Services:** AWS CloudTrail, Amazon EventBridge, AWS CloudFormation

**Socratic walkthrough**
1. *What is the earliest signal of a manual rule change?* — The API call itself, recorded by CloudTrail.
2. *How do you tell CloudFormation's own changes apart?* — Every stack uses `cfn-deployer`, so its session issuer identifies them.
3. *Which options wait for a schedule or a configuration recorder?* — B, C, and D.

**Why A is correct:** CloudTrail management events reach EventBridge in near real time. A rule with an `anything-but` filter on the session issuer passes only non-CloudFormation changes, and the existing central bus and SNS topic page on-call. No drift computation is needed.

**Why the others fail the constraint:**
- **B.** Works, but the managed drift rule is periodic — at best hourly.
- **C.** Works, but a 15-minute schedule plus asynchronous drift detection runs adds tens of minutes.
- **D.** Works, but waits for Config to record the change and then runs custom template comparison logic.

**Exam reflex:** For "who changed it, right now", use CloudTrail events in EventBridge; drift detection is for reconciliation, not alerting.

---

### Question 6

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS Systems Manager (Default Host Management Configuration), Systems Manager Quick Setup, AWS Organizations

**Socratic walkthrough**
1. *What normally gives the SSM Agent permission to register?* — The instance profile.
2. *Is there a way that doesn't involve instance profiles at all?* — Yes. Default Host Management Configuration (DHMC) lets Systems Manager use an account-level service role.
3. *What are the prerequisites?* — IMDSv2 and a recent SSM Agent, which this fleet already meets.

**Why D is correct:** Once DHMC is enabled through Quick Setup across the organization, Systems Manager manages every instance in every Region and account in scope, including new accounts, using its own role. Application teams' profiles and roles are untouched, so no change requests are needed.

**Why the others fail the constraint:**
- **A.** Works, but requires 40 change requests, custom code, and drift remediation on roles you don't own.
- **B.** Works, but replaces every instance profile and merges permissions — the most change-request-heavy option.
- **C.** Works, but hybrid activations are built for non-EC2 machines, and per-account codes and agent re-registration are pure toil.

**Exam reflex:** "Manage EC2 with SSM without touching instance profiles" means Default Host Management Configuration.

---

### Question 7

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS Systems Manager State Manager, Systems Manager Compliance, Systems Manager Resource Data Sync

**Socratic walkthrough**
1. *What is the job?* — Reapply a known desired state on a schedule and report compliance.
2. *Which Systems Manager capability is built for exactly that?* — State Manager associations, which record association compliance.
3. *Is the playbook reusable as-is?* — Yes. `AWS-ApplyAnsiblePlaybooks` runs playbooks from S3.

**Why D is correct:** A tag-targeted association applies the playbook every 30 minutes to every instance, including new ones launched by Auto Scaling. Association compliance flows automatically into the existing Resource Data Sync for Athena, with no new code or infrastructure.

**Why the others fail the constraint:**
- **A.** Works, but Config can't see file contents natively, so the custom rule calls Run Command and Automation — layered glue.
- **B.** Works, but the team writes and maintains a Lambda function and compliance-item reporting that State Manager provides natively.
- **C.** Works, but changes require AMI rebuilds, and log-based compliance is a custom pipeline.

**Exam reflex:** For desired-state enforcement plus compliance on managed instances, use State Manager associations.

---

### Question 8

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** EC2 Image Builder, Amazon Inspector, Amazon EC2 launch templates

**Socratic walkthrough**
1. *Which steps are manual or custom today?* — Build, scan, copy, share, and launch template updates.
2. *Which service does all of those as configuration?* — EC2 Image Builder: pipelines, Inspector scanning, and distribution settings that can update launch templates in target accounts and Regions.
3. *How do Auto Scaling groups pick up the change?* — They already use `$Default`, and the weekly refresh rolls it out.

**Why B is correct:** Image Builder's distribution configuration copies and shares the AMI and sets it as the new default version of each specified launch template across accounts and Regions. Inspector scanning is enabled in the pipeline, so no glue code remains.

**Why the others fail the constraint:**
- **A.** Works, but keeps Packer and adds eight Lambda deployments that must stay in sync.
- **C.** Works, but rebuilds in Step Functions what distribution settings already do.
- **D.** Works, but cross-account scripting inside the build is custom code to maintain and secure.

**Exam reflex:** Golden AMI build, scan, and distribute to launch templates across accounts means an EC2 Image Builder distribution configuration.

---

### Question 9

**Answer:** A  
**Constraint:** LEAST privilege · **Services:** AWS Service Catalog, AWS CloudFormation, AWS IAM

**Socratic walkthrough**
1. *Do developers need direct permissions on S3, KMS, DynamoDB, and RDS?* — Not if something else provisions on their behalf.
2. *What lets an end user launch a product without holding the underlying permissions?* — A Service Catalog launch constraint role.
3. *What do developers then need?* — Only end-user permissions for Service Catalog.

**Why A is correct:** Service Catalog provisions the security team's templates using the launch constraint role, and template constraints limit parameter values. Developers can launch, update, and terminate their provisioned products without any direct data-service or CloudFormation permissions.

**Why the others fail the constraint:**
- **B.** Works, but developers can create, modify, and delete these services directly.
- **C.** Works, but `iam:PassRole` on an admin role lets developers deploy any template with administrator permissions.
- **D.** Works, but grants full data-plane and control-plane access to four services on tagged resources.

**Exam reflex:** For self-service with least privilege, use Service Catalog with launch constraints.

---

### Question 10

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation (stack refactoring), Amazon DynamoDB, Amazon RDS

**Socratic walkthrough**
1. *What is the classic way to move resources between stacks?* — Retain, remove, then import, carefully and in order.
2. *Is there now a native operation for moving and renaming?* — Yes: CloudFormation stack refactoring, including logical ID mapping.
3. *Does it replace resources?* — No. Resources move between stacks without being recreated.

**Why C is correct:** Stack refactoring computes and executes the move as a single managed operation, can create the destination stacks, and remaps logical IDs. Nothing is deleted or replaced, and there are no manual retain or import steps.

**Why the others fail the constraint:**
- **A.** Works, but this multi-step manual procedure is error-prone across 400 resources and four stacks.
- **B.** Works, but the IaC generator only generates templates; the retain, remove, and import work remains.
- **D.** Works, but automates the legacy procedure with custom orchestration that has to be built and tested.

**Exam reflex:** "Split or move resources between stacks without replacement" — try stack refactoring first.

---

### Question 11

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** Amazon Application Recovery Controller (zonal autoshift), Elastic Load Balancing, Amazon EC2 Auto Scaling

**Socratic walkthrough**
1. *Who detects the AZ impairment in each option?* — In A, AWS; in the others, your own alarms, code, or Health event handling.
2. *Which option has built-in, schedulable rehearsal?* — Zonal autoshift practice runs, with outcome alarms and blocked dates.
3. *Is capacity a concern?* — No. The group already handles peak traffic from two AZs, which is exactly what autoshift expects.

**Why A is correct:** With zonal autoshift, AWS shifts the ALB's traffic away from an AZ when AWS telemetry shows an impairment, and returns it afterward. Required practice runs regularly exercise the shift, are evaluated against your alarms, and respect blocked dates.

**Why the others fail the constraint:**
- **B.** Works, but it's custom detection and recovery code with a risk of flapping.
- **C.** Works, but triples the load balancers and adds DNS failover logic — an architectural rebuild.
- **D.** Works, but it's custom glue around zonal shift, and Health-event timing is less direct than autoshift.

**Exam reflex:** For automatic AZ evacuation without tooling, use ARC zonal autoshift with practice runs.

---

### Question 12

**Answer:** D  
**Constraint:** MOST cost-effective · **Services:** AWS Systems Manager Inventory, Systems Manager Resource Data Sync, Amazon Athena

**Socratic walkthrough**
1. *Is software inventory collection on managed EC2 instances billed?* — Inventory collection itself carries no extra charge for EC2.
2. *How does data get centralized across accounts?* — An organization-wide Resource Data Sync to an S3 bucket.
3. *Why is Config costly here?* — Frequent package changes produce many configuration items.

**Why D is correct:** A daily inventory association collects package data, and an org-wide Resource Data Sync lands it in the audit bucket in a format Athena can query. Costs are just S3 storage and Athena scans.

**Why the others fail the constraint:**
- **A.** Works, but nightly package churn turns into a large number of billed configuration items.
- **B.** Works, but adds Run Command parsing, Lambda, DynamoDB, and connector costs.
- **C.** Works, but needs EC2, licensing, and SSH key management.

**Exam reflex:** For org-wide software inventory at low cost, use SSM Inventory with Resource Data Sync, S3, and Athena.

---

### Question 13

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation (Git sync), GitHub, AWS CodeConnections

**Socratic walkthrough**
1. *Is there anything to build?* — No. These are plain templates plus parameters.
2. *Can CloudFormation watch a branch directly?* — Yes. Git sync updates a stack when the tracked branch changes.
3. *What scales better to 120 stacks: 120 sync configurations or 120 pipelines and projects?* — The sync configurations.

**Why C is correct:** Git sync links each stack to its repository and branch through the existing connection. A deployment file defines the template path and parameters, so merges to `main` deploy with no pipeline, build project, or workflow to maintain.

**Why the others fail the constraint:**
- **A.** Works, but means 60 to 120 pipelines to create and maintain for zero build steps.
- **B.** Works, but adds per-repository workflows, OIDC roles, and runner dependencies.
- **D.** Works, but means a CodeBuild project, webhook, and script per repository.

**Exam reflex:** For template-only stacks deployed from Git, consider CloudFormation Git sync before pipelines.

---

### Question 14

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS Secrets Manager, Amazon RDS, AWS KMS

**Socratic walkthrough**
1. *Who must reach the database to rotate the password?* — With rotation Lambda functions, you do, so isolated subnets need endpoints or networking.
2. *Can RDS manage the master password itself?* — Yes. With `ManageMasterUserPassword`, RDS stores the secret in Secrets Manager and rotates it.
3. *Can you choose the KMS key and the schedule?* — Yes. Specify the KMS key for the secret and adjust the rotation schedule.

**Why D is correct:** RDS-managed master passwords remove rotation functions, VPC networking, and endpoint requirements entirely. Applications read the secret from Secrets Manager, which is the single place for credentials.

**Why the others fail the constraint:**
- **A.** Works, but means 120 rotation configurations with Lambda functions inside VPCs and new interface endpoints.
- **B.** Works, but it's custom rotation code, scheduling, and failure handling.
- **C.** Works, but stack updates for every instance every 30 days is heavy and risky orchestration.

**Exam reflex:** For RDS master passwords, prefer RDS-managed secrets in Secrets Manager over custom rotation.

---

### Question 15

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation, Amazon EventBridge, AWS Organizations

**Socratic walkthrough**
1. *Does CloudFormation emit stack status changes as events?* — Yes, to EventBridge in each account and Region.
2. *Which component already reaches the ticketing system?* — The central bus with its API destination.
3. *What's left to deploy?* — One forwarding rule per account and Region, which a service-managed StackSet handles.

**Why A is correct:** Stack status change events capture every tool's deployments, including the console. One StackSet deploys a filtered forwarding rule everywhere, and the existing bus and API destination open tickets.

**Why the others fail the constraint:**
- **B.** Works, but every tool and stack must set notification ARNs, and per-account Lambda functions filter messages.
- **C.** Works, but it's a polling function across 210 account-Region pairs with its own deduplication state.
- **D.** Works, but means 210 SNS topics and email ingestion instead of the existing API destination.

**Exam reflex:** Use native service events, routed through EventBridge cross-account buses, rather than polling or per-resource notifications.

---

## Scoring & Performance Breakdown

| Score | Band | Next step |
|---|---|---|
| **14–15 / 15** | Exam ready | Proceed to Practice Exam 3. |
| **12–13 / 15** | Passing | For each miss, re-read the distractor breakdown and note which constraint signal you overlooked. |
| **≤ 11 / 15** | Needs remediation | Use the domain column in the key table to pick study guides; retake this exam in a week. |
