# Domain 2: Configuration Management and IaC — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 17% of Exam  
**Format:** 15 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: CloudFormation CreationPolicy Signaling with `cfn-init` and `cfn-signal`

A Principal Infrastructure Engineer is writing an AWS CloudFormation template that deploys an Auto Scaling Group (ASG) of Amazon EC2 instances running behind an Application Load Balancer. The instances execute a bootstrap sequence via `UserData` that invokes `cfn-init` to configure system packages, deploy proprietary application binaries, and start internal systemd daemons. The CloudFormation stack creation must pause until at least 3 EC2 instances report successful initialization. If any instance fails its bootstrap script or times out after 15 minutes, CloudFormation must halt deployment and initiate an automatic rollback.

**Which combination of CloudFormation and instance configurations satisfies these requirements? (Select TWO)**
- A. Define a `CreationPolicy` attribute on the `AWS::AutoScaling::AutoScalingGroup` resource with `ResourceSignal: { Count: 3, Timeout: PT15M }`.
- B. In the Launch Template `UserData`, run `cfn-init` to parse `AWS::CloudFormation::Init`, followed by `cfn-signal -e $? --stack ${AWS::StackName} --resource AutoScalingGroup --region ${AWS::Region}`.
- C. Attach an `UpdatePolicy` attribute with `AutoScalingRollingUpdate` set to `WaitOnResourceSignals: true` and `PauseTime: PT15M` to govern initial stack creation.
- D. Define an `AWS::CloudFormation::WaitCondition` resource with a `WaitConditionHandle` pre-signed S3 URL configured for a count of 3 and a 900-second timeout.
- E. Ensure the EC2 Instance Profile has IAM permissions for `cloudformation:SignalResource` and configure `cfn-hup` to poll metadata changes every 15 minutes.

---

### Question 2: CloudFormation Custom Resources for Non-Empty S3 Bucket Teardown

A DevOps team manages transient ephemeral test environments provisioned via AWS CloudFormation. Each environment includes an Amazon S3 bucket used by integration test suites to store test payloads and log bundles. When a pipeline initiates stack deletion (`DELETE_STACK`), the CloudFormation operation consistently fails and enters the `DELETE_FAILED` state with an `S3BucketNotEmpty` error because objects and version delete markers remain in the bucket. The solution must ensure fully automated, clean stack deletion under the **LEAST OPERATIONAL OVERHEAD**.

**Which architecture reliably eliminates this failure mode?**
- A. Implement an `AWS::CloudFormation::CustomResource` backed by an AWS Lambda function that triggers when `RequestType` equals `Delete`, iterates through all object versions and delete markers using `ListObjectVersions` and `DeleteObjects`, and posts a `SUCCESS` payload to the pre-signed `ResponseURL`.
- B. Add `DeletionPolicy: Delete` and `UpdateReplacePolicy: Delete` directly to the `AWS::S3::Bucket` resource properties in the CloudFormation template to instruct the S3 service engine to wipe objects prior to bucket deletion.
- C. Configure an S3 Lifecycle Rule on the bucket that expires objects after 1 day, and instruct developers to pause pipeline deletion executions until 24 hours after integration testing completes.
- D. Attach an Amazon EventBridge rule that detects CloudFormation `DELETE_IN_PROGRESS` events for the stack, invokes an AWS Systems Manager Automation runbook that deletes the S3 bucket using the AWS CLI, and deletes the stack again.

---

### Question 3: Systems Manager Patch Manager Baseline Governance & Maintenance Windows

An enterprise platform team oversees security compliance across a fleet of 800 Amazon EC2 instances running Amazon Linux 2, Amazon Linux 2023, and Windows Server 2022. Operating system patches classified as `Critical` security updates must be automatically approved 5 days after vendor publication to prevent zero-day regressions. The actual patch installation and required host reboots must execute exclusively during a weekly maintenance window scheduled every Saturday between 02:00 UTC and 05:00 UTC, leaving non-security updates untouched.

**Which combination of AWS Systems Manager components achieves this architecture? (Select TWO)**
- A. Create custom **Patch Baselines** in SSM Patch Manager for each target operating system, defining approval rules for classification `Security` and severity `Critical` with an auto-approval delay of 5 days, and tag instances into corresponding **Patch Groups**.
- B. Configure an SSM **Maintenance Window** scheduled for Saturdays at 02:00 UTC with a duration of 3 hours and a 1-hour cutoff, registering a `Run Command` task using document `AWS-RunPatchBaseline` with operation `Install` and reboot parameter enabled.
- C. Create an AWS Systems Manager **State Manager** association that targets all managed instances with document `AWS-RunShellScript` scheduled to execute daily at 02:00 UTC.
- D. Enable Amazon Inspector with automated remediation enabled, configuring SSM Automation runbooks to install OS updates immediately upon vulnerability detection.
- E. Create an SSM Maintenance Window that runs document `AWS-ApplyPatchBaseline` targeting all EC2 instances via resource tags with operation `Scan` and an approval threshold of 5 days.

---

### Question 4: Systems Manager State Manager Continuous Drift Remediation

Enterprise security policy mandates that the SSH daemon configuration (`/etc/ssh/sshd_config`) across all Amazon EC2 Linux instances must enforce `PasswordAuthentication no` and `PermitRootLogin no`. An audit found that support engineers occasionally alter these parameters manually during troubleshooting sessions. The DevOps team must implement an automated governance control that enforces compliance continuously, corrects manual configuration drift within 30 minutes of occurrence, and requires **ZERO INFRASTRUCTURE CODE DEPLOYMENT**.

**Which mechanism satisfies these criteria with the LEAST operational overhead?**
- A. Create an AWS Systems Manager **State Manager Association** targeting all managed Linux instances, configured with a custom SSM Document (or Ansible playbook) that validates and updates `sshd_config`, scheduled on a 30-minute cron interval with `ApplyOnlyAtCronInterval: false`.
- B. Configure an Amazon CloudWatch Alarm that monitors EC2 instance CPU utilization spikes, triggering an AWS Lambda function that initiates an instance reboot whenever unauthorized SSH logins are logged.
- C. Deploy an AWS Config Custom Rule that parses instance configuration changes via CloudTrail API events and invokes an AWS Systems Manager Automation runbook that runs a patch script on non-compliant nodes.
- D. Configure an Amazon EventBridge rule that intercepts AWS CloudTrail `AuthorizeSecurityGroupIngress` events on port 22 and triggers a Step Functions workflow to terminate the instance.

---

### Question 5: Elastic Beanstalk High-Availability Zero-Downtime Deployment Strategies

A mission-critical financial ledger application runs on AWS Elastic Beanstalk using an Auto Scaling Group behind an Application Load Balancer. The application requires extensive in-memory cache pre-warming and database connection pool verification during boot, requiring 8 minutes per instance before passing health checks. During production deployments, the system must **NEVER DROP BELOW 100% SERVING CAPACITY**, must ensure zero cross-version traffic mixing on existing instances, and must provide an **INSTANT ZERO-IMPACT ROLLBACK** if replacement instances fail health checks.

**Which Elastic Beanstalk deployment policy should the DevOps Engineer select?**
- A. `Immutable`: Elastic Beanstalk creates a temporary Auto Scaling Group containing a full replacement fleet, deploys the new version, shifts traffic only after all replacement instances pass health checks, and terminates the temporary ASG instantly if deployment fails.
- B. `Rolling with Additional Batch`: Elastic Beanstalk launches an extra batch of instances up front to maintain 100% capacity during updates, but updates instances within the original ASG, requiring an inverse rolling deployment if a rollback is triggered.
- C. `Rolling`: Elastic Beanstalk updates instances in sequential batches within the existing Auto Scaling Group, temporarily reducing serving capacity by the batch size during each deployment cycle.
- D. `Traffic Splitting`: Elastic Beanstalk creates a temporary ASG and routes a percentage of live incoming production user traffic directly to unvalidated instances for canary testing without a pre-deployment ALB health check gate.

---

### Question 6: Enterprise StackSets Automated Deployment Across AWS Organizations

An enterprise platform architecture team manages 120 AWS accounts grouped under a `Workloads` Organizational Unit (OU) in AWS Organizations. Whenever a new AWS account is vended into the `Workloads` OU, baseline governance stacks (IAM permission boundaries, AWS Config conformance packs, CloudWatch logging destinations) must be automatically provisioned into that account. When an account is moved out of the OU, its baseline stacks must be cleaned up automatically.

**How should this multi-account infrastructure deployment be implemented with the LEAST administrative overhead?**
- A. Deploy an AWS CloudFormation **StackSet** with **`SERVICE_MANAGED`** permissions targeted to the `Workloads` OU ID, configuring `AutoDeployment: { Enabled: true, RetainStacksOnAccountRemoval: false }`.
- B. Deploy an AWS CloudFormation **StackSet** with **`SELF_MANAGED`** permissions, manually creating `AWSCloudFormationStackSetAdministrationRole` in the management account and deploying execution roles to all member accounts.
- C. Configure an Amazon EventBridge rule that captures `CreateAccount` events from AWS Organizations, triggering an AWS Lambda function that executes `cloudformation create-stack` in the target account.
- D. Deploy an AWS Control Tower Account Factory customization using Terraform Cloud that triggers workspace runs upon new account enrollment via AWS Service Catalog.

---

### Question 7: Decoupled IaC Architecture: Cross-Stack References vs. SSM Dynamic References

A DevOps team is redesigning the infrastructure-as-code architecture for a distributed microservice ecosystem. The core VPC and transit networking is provisioned by a central networking team, while 40 microservices are deployed independently by decentralized product teams. The microservice templates require access to shared subnet IDs and security group IDs. If the central networking stack is updated, CloudFormation must prevent the deletion or modification of exported resources if downstream microservices depend on them, while avoiding tight cross-stack deployment locks.

**Which architectural pattern provides the optimal balance of decoupling and dependency safety?**
- A. Publish networking outputs as **CloudFormation Exports** (`Export: Name: !Sub "${AWS::StackName}-PrivateSubnets"`) and consume them via `Fn::ImportValue` in application stacks, leveraging CloudFormation's native lock that blocks deletion of referenced exports.
- B. Store shared networking IDs in **AWS Systems Manager Parameter Store** as standard parameters, and consume them in application templates using CloudFormation dynamic references (`{{resolve:ssm:/network/vpc-id}}`), allowing fully decoupled stack updates.
- C. Combine all networking resources and 40 microservices into a single monolithic CloudFormation template, managing environment variations using conditions and parameter mappings.
- D. Use CloudFormation **Nested Stacks** where a root template instantiates the VPC stack and passes output parameters directly into each child application stack resource definition.

---

### Question 8: Automated Golden AMI Pipeline with Systems Manager Automation

An organization requires an automated monthly pipeline to build, harden, and validate hardened Amazon Linux 2023 golden AMIs for corporate EC2 workloads. The workflow must launch a base AWS-provided AMI, apply CIS Benchmark Ansible hardening scripts via SSM Run Command, install the CloudWatch and Inspector agents, create an encrypted AMI tagged with the compliance build date, test the AMI by launching a canary instance, and terminate all temporary instances. The solution must provide native retry logic and step-level CloudWatch audit logs.

**Which architecture satisfies these requirements with the LEAST custom glue code?**
- A. Create an **AWS Systems Manager Automation Runbook** (or configure an EC2 Image Builder pipeline) defining sequential automation actions (`aws:runInstances`, `aws:runCommand`, `aws:createImage`, and `aws:terminateInstances`) with built-in rollback steps.
- B. Deploy an AWS Step Functions state machine that invokes Lambda functions to execute SSH shell scripts directly on EC2 instances using credentials retrieved from AWS Secrets Manager.
- C. Launch a persistent EC2 instance running Jenkins, configure a cron job to execute HashiCorp Packer via the AWS CLI, and publish notification alerts to an Amazon SNS topic.
- D. Write an AWS CodeBuild project that downloads base AMI snapshots from Amazon S3, executes chroot modifications inside the container filesystem, and calls `ec2:RegisterImage`.

---

### Question 9: CloudFormation Drift Detection & Live Template Synchronization

During a production incident troubleshooting session, an engineer modified several security group ingress rules and updated an Auto Scaling Group's `DesiredCapacity` directly in the AWS Management Console. The changes resolved the immediate outage, but the underlying CloudFormation template in Git does not reflect the current physical state. The platform team must identify all unrecorded modifications, update the Git-tracked template to match the desired production reality, and ensure future deployments do not overwrite approved operational fixes.

**What is the correct procedure to achieve template and resource fidelity?**
- A. Execute **CloudFormation Drift Detection** on the stack, review drifted resource properties in the Drift Details report, update the Git CloudFormation template to incorporate the approved changes, and apply a stack update using an executed Change Set.
- B. Initiate a stack deletion using `DeletionPolicy: Retain` on all modified resources, modify the template in source control to include the new security group rules, and deploy a brand-new CloudFormation stack.
- C. Query AWS CloudTrail for `AuthorizeSecurityGroupIngress` and `ModifyAutoScalingGroup` events over the past 7 days, and run an SSM Automation runbook to revert the live AWS resources back to the template state.
- D. Create a CloudFormation Change Set of type `CREATE` targeting the existing stack, and execute the change set to automatically import untracked resource modifications into the template.

---

### Question 10: Elastic Beanstalk Lifecycle Execution Order: `.ebextensions` vs. `.platform/hooks`

A DevOps engineer is modernizing deployment hooks for an application running on Elastic Beanstalk on the Amazon Linux 2023 platform. The deployment requires executing database schema migrations (`alembic upgrade head`) on exactly one EC2 instance prior to traffic cutover, installing host-level security agents before the web server launches, and running an application warm-up script immediately after the application server starts listening on its port.

**How should these scripts be structured across `.ebextensions/` and `.platform/hooks/`?**
- A. Place host security packages under `.ebextensions/*.config` `commands:`, run the database migration in `container_commands:` with `leader_only: true` (executing in staging prior to deployment), and place the warm-up script in `.platform/hooks/postdeploy/`.
- B. Place the database migration in `.ebextensions/*.config` `commands:`, place host security packages in `container_commands:` with `leader_only: true`, and place the warm-up script in `.platform/confighooks/predeploy/`.
- C. Place the database migration script in `.platform/hooks/prebuild/`, configure host security packages under `container_commands:`, and execute the application warm-up script inside `.ebextensions/*.config` `commands:`.
- D. Place all three scripts inside `.platform/hooks/postdeploy/`, using a distributed DynamoDB lock to ensure that the database migration executes on only one instance.

---

### Question 11: Systems Manager Parameter Store Tiering, Quotas, & Multi-Account Sharing

An enterprise platform provisions 18,000 configuration keys across 15 AWS accounts in an AWS Organization. Several configuration strings exceed 5 KB in size, and specific core microservices require high-throughput parameter access exceeding 2,500 transactions per second (TPS). Additionally, non-sensitive global configuration parameters must be accessible across all 15 member accounts without duplicating storage.

**Which Systems Manager Parameter Store configuration satisfies all architectural constraints?**
- A. Use the **Advanced Tier** of Parameter Store (supporting up to 100,000 parameters and 8 KB payload sizes), enable **Higher Throughput** (up to 10,000 TPS) in account settings, and share parameters across accounts using **AWS Resource Access Manager (AWS RAM)**.
- B. Use the **Standard Tier** of Parameter Store, configure client-side gzip compression to keep payloads under 4 KB, and use AWS Secrets Manager cross-account resource policies to share parameters.
- C. Store the parameters in the Standard Tier of Parameter Store, deploy an Amazon ElastiCache Redis cluster in each member account to cache values, and enable VPC peering across all 15 accounts.
- D. Store configuration keys in Amazon DynamoDB with DynamoDB Accelerator (DAX) in the management account, and configure cross-account IAM roles for all microservice access.

---

### Question 12: Dual Protection of Stateful CloudFormation Resources: Deletion vs. Update Replacement

A CloudFormation stack manages an Amazon Aurora PostgreSQL DB cluster (`AWS::RDS::DBCluster`) and an Amazon DynamoDB table (`AWS::DynamoDB::Table`). The DevOps team must ensure that:
1. If the stack is deleted, a final backup snapshot of the Aurora DB cluster is taken, while the DynamoDB table is retained as an active standalone resource.
2. If a stack update modifies a property that requires **Replacement** (such as changing an immutable database engine parameter or encryption attribute), the existing physical resource is preserved rather than destroyed.

**Which configuration on the CloudFormation resources satisfies both requirements?**
- A. Set `DeletionPolicy: Snapshot` and `UpdateReplacePolicy: Snapshot` on the Aurora cluster; set `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain` on the DynamoDB table.
- B. Enable `TerminationProtection: true` on the CloudFormation stack, and attach `DeletionPolicy: Retain` to all resources in the template.
- C. Set `DeletionPolicy: Retain` on both resources, and configure an Amazon EventBridge rule that intercepts CloudFormation `ResourceUpdateFailed` events to trigger an AWS Backup job.
- D. Set `UpdatePolicy: AutoScalingRollingUpdate` on both database resources, and configure `DeletionPolicy: Delete` with automated pre-delete Lambda snapshots.

---

### Question 13: AWS CDK CI/CD Pipeline Automation & Synthesis Validation

A platform team is building a continuous deployment pipeline for an AWS Cloud Development Kit (AWS CDK v2) application written in TypeScript. In the CI/CD pipeline, the build phase must synthesize the CloudFormation template, validate that synth succeeds without errors, verify infrastructure diffs against the production environment without hanging for interactive confirmation, and safely deploy changes.

**Which sequence of AWS CDK CLI commands should be executed in the CI/CD pipeline runner?**
- A. `cdk synth` (validates synthesis and generates CloudFormation artifacts) > `cdk diff` (evaluates infrastructure delta) > `cdk deploy --require-approval never` (executes deployment non-interactively).
- B. `cdk build` (compiles TypeScript code) > `cdk test` (runs Jest unit tests) > `cdk push --force` (deploys templates directly to the CloudFormation engine).
- C. `cdk compile` (generates JavaScript) > `cdk inspect --no-prompt` (validates drift) > `cdk apply --auto-approve` (provisions cloud resources).
- D. `cdk package` (bundles CloudFormation templates) > `cdk validate` (evaluates Guard rules) > `cdk deploy --strict` (deploys with rollback triggers).

---

### Question 14: Systems Manager Run Command Rate Control & Fault Tolerance

A systems operations team needs to execute a critical security patch script across a fleet of 600 Amazon EC2 instances using AWS Systems Manager Run Command (`AWS-RunShellScript`). To avoid overloading internal database backends during service restarts, the command must execute concurrently on at most 40 instances simultaneously. If more than 6 instances fail execution, the entire command invocation must halt immediately to limit the failure blast radius.

**Which parameters of `aws ssm send-command` should be configured?**
- A. Set `--max-concurrency 40` (or `"40"`) and set `--max-errors 6` (or `"6"`).
- B. Set `--batch-size 40` and set `--failure-threshold 6`.
- C. Set `--concurrency-limit 40` and set `--stop-after-errors 6`.
- D. Set `--rate-control "40 instances"` and set `--error-tolerance "1%"` in the command document parameters.

---

### Question 15: CloudFormation Dynamic AMI Resolution via Systems Manager Parameter Types

A central Cloud Operations team updates approved Amazon Linux 2023 golden AMI IDs monthly in AWS Systems Manager Parameter Store under `/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64`. Workload engineering teams maintain dozens of independent CloudFormation templates. When application teams deploy or update their stacks, CloudFormation must automatically resolve the current AMI ID directly from Parameter Store at stack execution time without modifying template code or passing manual parameter overrides.

**How should this requirement be implemented in the CloudFormation templates?**
- A. Define a template parameter of type `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` with a default value of `/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64`.
- B. Use a dynamic reference string in the template properties: `{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64:1}}`.
- C. Configure a CloudFormation Custom Resource backed by an AWS Lambda function that calls `ssm:GetParameter` during stack deployment and returns the AMI ID.
- D. Include an EC2 Launch Template `UserData` script that executes `aws ssm get-parameter` using the AWS CLI and updates the running kernel image at boot.

---

## Answer Key & Detailed Explanations

### Question 1: CloudFormation CreationPolicy Signaling with `cfn-init` and `cfn-signal`
**Answer:** A, B  
**Explanation:**  
`CreationPolicy` with `ResourceSignal` is natively supported on `AWS::AutoScaling::AutoScalingGroup` and `AWS::EC2::Instance`. Setting `Count: 3` and `Timeout: PT15M` instructs CloudFormation to hold the stack in `CREATE_IN_PROGRESS` until 3 successful signals arrive within 15 minutes (Action A). The instances must execute `cfn-signal -e $?` with `--stack`, `--resource`, and `--region` upon completing `cfn-init` in `UserData` (Action B). `UpdatePolicy` (Option C) governs stack *updates*, not initial stack creation. `WaitCondition` (Option D) is legacy and unnecessary for native ASG resources.

---

### Question 2: CloudFormation Custom Resources for Non-Empty S3 Bucket Teardown
**Answer:** A  
**Explanation:**  
Amazon S3 rejects `DeleteBucket` API requests if the bucket contains any objects, object versions, or delete markers. Setting `DeletionPolicy: Delete` (Option B) does not empty the bucket; CloudFormation simply attempts standard bucket deletion, which fails. The proven architectural pattern is an `AWS::CloudFormation::CustomResource` backed by a Lambda function. On `RequestType == 'Delete'`, the function iterates over all object versions and delete markers via `ListObjectVersions` and issues batch `DeleteObjects` calls before returning `SUCCESS` to the pre-signed S3 URL.

---

### Question 3: Systems Manager Patch Manager Baseline Governance & Maintenance Windows
**Answer:** A, B  
**Explanation:**  
SSM Patch Manager requires two core pillars: 1) Custom **Patch Baselines** defining auto-approval rules (e.g. classification `Security`, severity `Critical`, approval delay 5 days) associated with target instances via Patch Groups (Action A); and 2) an SSM **Maintenance Window** defining the schedule (Saturday 02:00 UTC, 3-hour duration, 1-hour cutoff) registering a task using `AWS-RunPatchBaseline` with the `Install` operation and reboot enabled (Action B). State Manager (Option C) lacks native maintenance window cutoff governance.

---

### Question 4: Systems Manager State Manager Continuous Drift Remediation
**Answer:** A  
**Explanation:**  
AWS Systems Manager **State Manager** is specifically designed for continuous, automated configuration drift detection and remediation. A State Manager Association targeting managed instances can run an SSM document (or Ansible playbook) on a scheduled frequency (e.g. every 30 minutes) to inspect `/etc/ssh/sshd_config`, re-apply compliant configuration values if drift is detected, and restart the daemon. This requires zero ongoing infrastructure pipeline runs.

---

### Question 5: Elastic Beanstalk High-Availability Zero-Downtime Deployment Strategies
**Answer:** A  
**Explanation:**  
The `Immutable` deployment policy creates an entirely separate, temporary Auto Scaling Group behind the same Application Load Balancer and launches a full duplicate fleet of replacement instances. Because the original instances remain untouched, production capacity never drops below 100%. Replacement instances warm up caches and pass health checks over their 8-minute boot window. If deployment fails or alarms breach, Elastic Beanstalk terminates the temporary ASG, providing an immediate zero-impact rollback. `Rolling with Additional Batch` (Option B) updates instances within the original ASG and requires an inverse rolling deployment to roll back.

---

### Question 6: Enterprise StackSets Automated Deployment Across AWS Organizations
**Answer:** A  
**Explanation:**  
CloudFormation StackSets with **`SERVICE_MANAGED`** permissions integrate directly with AWS Organizations. When targeted to an Organizational Unit (OU) with `AutoDeployment: Enabled`, StackSets automatically deploys stacks to newly created or moved member accounts entering the OU without manual intervention. Setting `RetainStacksOnAccountRemoval: false` ensures stacks are automatically deleted when an account leaves the OU. `SELF_MANAGED` StackSets (Option B) require manual IAM role management in every member account.

---

### Question 7: Decoupled IaC Architecture: Cross-Stack References vs. SSM Dynamic References
**Answer:** A  
**Explanation:**  
CloudFormation cross-stack references created via `Export` and consumed via `Fn::ImportValue` explicitly enforce dependency safety. CloudFormation locks exported outputs: a stack exporting an output cannot be deleted, and its exported value cannot be modified as long as another stack imports it. This prevents breaking 40 microservices accidentally. Parameter Store dynamic references (Option B) decouple templates but provide zero deletion protection against accidental resource destruction in the networking stack.

---

### Question 8: Automated Golden AMI Pipeline with Systems Manager Automation
**Answer:** A  
**Explanation:**  
AWS Systems Manager Automation Runbooks provide a serverless, managed orchestration framework for EC2 workflows without maintaining external servers. A custom runbook (or AWS EC2 Image Builder) chains actions like `aws:runInstances`, `aws:runCommand` (running Ansible hardening scripts), `aws:createImage`, and `aws:terminateInstances` with native error handling, step retries, and CloudWatch execution logging.

---

### Question 9: CloudFormation Drift Detection & Live Template Synchronization
**Answer:** A  
**Explanation:**  
CloudFormation Drift Detection compares the current physical configuration of deployed resources against the expected properties recorded in the stack template. By reviewing the Drift Details report, engineers identify exact attribute modifications (e.g., added security group ingress rules, changed ASG capacity). The DevOps engineer then updates the template in Git to incorporate these approved operational changes and applies a stack update with a Change Set, bringing code and infrastructure back into synchronization.

---

### Question 10: Elastic Beanstalk Lifecycle Execution Order: `.ebextensions` vs. `.platform/hooks`
**Answer:** A  
**Explanation:**  
In Elastic Beanstalk, `.ebextensions/*.config` `commands:` execute early in the provisioning cycle before the web server is configured and before the application code is unpacked. `container_commands:` execute from the application staging directory *after* the source bundle is extracted, and specifying `leader_only: true` ensures database migrations execute on only a single instance. On Amazon Linux 2023 platforms, `.platform/hooks/postdeploy/` executes after the application service has been deployed and started, making it ideal for final warm-up scripts.

---

### Question 11: Systems Manager Parameter Store Tiering, Quotas, & Multi-Account Sharing
**Answer:** A  
**Explanation:**  
The **Advanced Tier** of SSM Parameter Store increases parameter payload limits from 4 KB to 8 KB and capacity up to 100,000 parameters. Enabling **Higher Throughput** raises the regional transaction limit from 40 TPS to up to 10,000 TPS. Furthermore, Parameter Store integrates with **AWS Resource Access Manager (AWS RAM)**, allowing parameters to be shared securely across accounts in an AWS Organization without cross-account IAM role assumptions or data duplication.

---

### Question 12: Dual Protection of Stateful CloudFormation Resources: Deletion vs. Update Replacement
**Answer:** A  
**Explanation:**  
`DeletionPolicy` governs physical resource handling when a resource is deleted from a template or the stack is deleted. Setting `DeletionPolicy: Snapshot` on the Aurora cluster creates a final snapshot, while `DeletionPolicy: Retain` on DynamoDB preserves the live table. `UpdateReplacePolicy` governs the fate of the *old* resource when an update requires replacement (`Requires: Replacement`). Setting `UpdateReplacePolicy: Snapshot` and `UpdateReplacePolicy: Retain` guarantees old physical databases are not destroyed during replacement updates.

---

### Question 13: AWS CDK CI/CD Pipeline Automation & Synthesis Validation
**Answer:** A  
**Explanation:**  
The standard AWS CDK deployment workflow consists of: 1) `cdk synth`, which compiles CDK code and synthesizes CloudFormation templates, validating construct schemas; 2) `cdk diff`, which compares local synthesized templates against deployed CloudFormation stacks to surface infrastructure modifications; and 3) `cdk deploy --require-approval never`, which provisions the stacks non-interactively in automated CI/CD runners without waiting for human console prompts for IAM or security group changes.

---

### Question 14: Systems Manager Run Command Rate Control & Fault Tolerance
**Answer:** A  
**Explanation:**  
In `aws ssm send-command`, rate and concurrency control is configured via `--max-concurrency` (which defines the maximum number or percentage of managed instances allowed to execute the command concurrently, e.g. `40` or `"10%"`). Error threshold control is configured via `--max-errors` (which defines the maximum number or percentage of failed executions before Run Command halts further execution, e.g. `6` or `"5%"`).

---

### Question 15: CloudFormation Dynamic AMI Resolution via Systems Manager Parameter Types
**Answer:** A  
**Explanation:**  
CloudFormation provides dedicated AWS-specific parameter types such as `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>`. When defined with a default value pointing to an SSM parameter key (such as the public AWS-maintained AL2023 parameter), CloudFormation queries Systems Manager Parameter Store directly at stack execution time and resolves the latest underlying AMI ID without requiring template modifications or external Lambda lookups.
