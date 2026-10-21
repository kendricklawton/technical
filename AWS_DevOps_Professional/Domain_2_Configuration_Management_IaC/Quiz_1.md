# Domain 2: Configuration Management and IaC — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 17% of Exam  
**Format:** 15 Deep Scenario-Based Professional Practice Questions with Detailed Explanations

---

### Question 1: CloudFormation CreationPolicy with `cfn-signal` and `cfn-init`

An infrastructure engineer is writing an AWS CloudFormation template that provisions an Auto Scaling Group of Amazon EC2 instances. The EC2 instances must install Docker, download a proprietary backend package from an internal repository, and start systemd services. The CloudFormation stack creation must not complete until at least 3 instances are completely configured and running. If any instance fails initialization, the stack creation must fail and roll back.

**Which combination of CloudFormation configurations achieves this requirement? (Select TWO)**
- A. Define a `CreationPolicy` on the `AWS::AutoScaling::AutoScalingGroup` resource with `ResourceSignal: { Count: 3, Timeout: PT15M }`.
- B. Define a `WaitCondition` resource with a pre-signed URL and specify `Count: 3`.
- C. In the EC2 Launch Template `UserData`, run `cfn-init` to execute the `AWS::CloudFormation::Init` configuration, followed by `cfn-signal -e $? --stack ${AWS::StackName} --resource AutoScalingGroup --region ${AWS::Region}`.
- D. Configure an `UpdatePolicy` with `AutoScalingRollingUpdate` set to `WaitOnResourceSignals: true`.
- E. Run `cfn-hup` in the UserData with an interval of 5 minutes.

---

### Question 2: CloudFormation Custom Resources with Lambda S3 Bucket Deletion

A company deploys dynamic, temporary development environments using AWS CloudFormation. Each environment includes an Amazon S3 bucket used for test uploads. During automated teardown (`DELETE_STACK`), the CloudFormation stack deletion fails because the S3 bucket is not empty (`BucketNotEmpty` error).

**What is the most automated and reliable way to enable clean stack deletion?**
- A. Add a CloudFormation Custom Resource backed by an AWS Lambda function. Configure the Lambda function to listen for `RequestType == 'Delete'`, empty all object versions and delete markers from the S3 bucket using `s3:ListObjectVersions` and `s3:DeleteObjects`, and send a `SUCCESS` response to the pre-signed S3 URL.
- B. Add `DeletionPolicy: Delete` to the `AWS::S3::Bucket` resource.
- C. Run a local bash script before deleting the stack that executes `aws s3 rm s3://bucket --recursive`.
- D. Enable S3 Object Lock on the bucket with a 1-day retention policy.

---

### Question 3: Systems Manager Patch Manager with Patch Baselines and Maintenance Windows

An enterprise requires automated security patching for 800 Amazon EC2 instances running Amazon Linux 2 and Windows Server 2022. Critical operating system security patches must be approved automatically 5 days after release. Patching must occur exclusively during a Saturday maintenance window between 02:00 UTC and 05:00 UTC. Instances must be rebooted if required by the patch.

**Which combination of AWS Systems Manager components achieves this architecture? (Select TWO)**
- A. Create custom **Patch Baselines** in SSM Patch Manager for Amazon Linux and Windows Server with approval rules set to approve patches with classification `Security` and severity `Critical` with an auto-approval delay of `5 days`.
- B. Create an AWS Systems Manager **State Manager** association that runs every Saturday with document `AWS-RunShellScript`.
- C. Configure an SSM **Maintenance Window** scheduled for Saturdays at 02:00 UTC for a duration of 3 hours with a 1-hour cutoff. Register a `Run Command` task using the `AWS-RunPatchBaseline` document with the `Install` operation and reboot parameter enabled.
- D. Use AWS Inspector to automatically deploy OS patches when vulnerabilities are discovered.
- E. Use SSM Inventory to install updates whenever packages are out of date.

---

### Question 4: Systems Manager State Manager Drift Correction

A compliance policy mandates that the SSH daemon configuration (`sshd_config`) on all Amazon EC2 Linux instances must disable password authentication (`PasswordAuthentication no`) and disable root login (`PermitRootLogin no`). If a system administrator manually alters this file on any instance, the configuration must be automatically remediated and reverted to the compliant state within 60 minutes.

**How should this requirement be implemented with the LEAST operational effort?**
- A. Create an AWS Systems Manager **State Manager** association that targets all managed Linux instances. Configure the association to run an SSM document (or Ansible playbook) that enforces the desired `sshd_config` parameters, scheduled to run every 30 minutes.
- B. Configure an Amazon CloudWatch alarm on the EC2 CPU utilization to trigger a Lambda function that reboots the instance.
- C. Use Amazon EventBridge to capture SSH connections and terminate non-compliant instances.
- D. Run a cron job on each EC2 instance that runs `systemctl restart sshd`.

---

### Question 5: Elastic Beanstalk Immutable Deployments vs. Rolling with Additional Batch

A mission-critical financial reporting application is hosted on AWS Elastic Beanstalk using an Auto Scaling Group behind an Application Load Balancer. The application connects to a database and runs stateful in-memory cache warming on boot, which takes 8 minutes per instance. During software releases, the platform must never drop below 100% serving capacity, and if a deployment fails, the rollback must occur with zero impact on the existing production instances.

**Which Elastic Beanstalk deployment policy should the DevOps Engineer select?**
- A. `All at Once`
- B. `Rolling`
- C. `Rolling with Additional Batch`
- D. `Immutable`

---

### Question 6: CloudFormation StackSets Organizational Auto-Deployment

An enterprise uses AWS Organizations with hundreds of AWS accounts organized into Organizational Units (OUs). Whenever a new member account is created inside the `Workloads` OU, standard security infrastructure (IAM roles, AWS Config rules, CloudWatch Log groups) must be provisioned into that account automatically without manual intervention.

**How should this automated provisioning be configured?**
- A. Deploy an AWS CloudFormation **StackSet** with service-managed permissions targeted at the `Workloads` OU. Enable **Automatic deployment** (`AutoDeployment: { Enabled: true, RetainStacksOnAccountRemoval: false }`).
- B. Write an AWS Lambda function triggered on account creation events in EventBridge that calls `CreateStack` via the AWS CLI.
- C. Create a CloudFormation template in S3 and send an email to the account owner with a launch link.
- D. Use AWS Control Tower Account Factory with manual account configuration scripts.

---

### Question 7: CloudFormation Nested Stacks vs. Cross-Stack References

An organization is designing a modular CloudFormation architecture for an enterprise web application. The core VPC and networking infrastructure is managed by the network team, while the microservices are deployed frequently by development teams. The networking outputs (`VpcId`, `SubnetIds`) must be shared with the application templates. If the application team attempts to reference a subnet that is later deleted, CloudFormation must prevent the deletion of the networking resources while dependent applications exist.

**Which architectural approach best balances separation of concerns and dependency safety?**
- A. Use CloudFormation **Nested Stacks** where the root template includes both VPC and Application resources.
- B. Use CloudFormation **Exported Outputs** in the VPC stack (`Export: Name: !Sub "${AWS::StackName}-VpcId"`), and use the `!ImportValue` intrinsic function in the application stacks.
- C. Store all output values in an unencrypted S3 text file and download it via UserData.
- D. Hardcode the VPC and Subnet IDs into the application templates.

---

### Question 8: Systems Manager Automation Runbook for AMI Baking

A company bakes golden AMIs monthly for security compliance. The process requires launching a base Amazon Linux 2023 EC2 instance, running Ansible playbooks to harden the OS, installing security agents, capturing a new AMI, testing the AMI by launching a test instance, and terminating all temporary instances.

**How should this workflow be automated with the LEAST operational complexity?**
- A. Create a custom **AWS Systems Manager Automation Runbook** (or use EC2 Image Builder). The automation document defines sequential steps using actions like `aws:runInstances`, `aws:runCommand` to execute hardening scripts, `aws:createImage` to capture the AMI, and `aws:terminateInstances`.
- B. Write a multi-threaded Python script deployed to AWS Elastic Beanstalk.
- C. Use AWS Step Functions that SSHs into instances using passwords stored in DynamoDB.
- D. Manually launch EC2 instances every month, run commands via terminal, and click "Create Image" in the AWS Management Console.

---

### Question 9: CloudFormation Drift Detection and Remediation

A security audit reveals that several security group ingress rules in a production CloudFormation stack were manually modified in the AWS Management Console during an emergency troubleshooting session. The CloudFormation template in source control does not reflect these unauthorized changes.

**How can the DevOps Engineer identify the exact changes and restore template fidelity?**
- A. Delete the CloudFormation stack and redeploy it from scratch.
- B. Run CloudFormation **Drift Detection** on the stack. Review the drift status and details in the console to inspect modified resource properties, update the CloudFormation template to match the desired state, and apply a stack update (or manually revert the drifted properties to match the template).
- C. Use AWS Trusted Advisor to download the security group history.
- D. Query AWS CloudTrail to revert all API calls made in the last 24 hours.

---

### Question 10: Elastic Beanstalk `.ebextensions` Execution Order

A developer is configuring an Elastic Beanstalk environment using multiple configuration files inside the `.ebextensions/` folder:
- `01_packages.config`
- `02_database_migration.config`
- `03_environment.config`

The developer needs to understand when `commands` and `container_commands` execute relative to application extraction.

**Which statement accurately describes the execution lifecycle of commands in `.ebextensions`?**
- A. `commands` run after application source extraction; `container_commands` run before any packages are installed.
- B. `commands` run early in the deployment process before the application and web server are set up and before application source is unpacked. `container_commands` run from the staging directory **after** the application source code has been extracted, but before the application version is deployed to its final location.
- C. `container_commands` run inside the Docker container only, whereas `commands` run on the host OS.
- D. Both `commands` and `container_commands` run concurrently in random alphabetical order.

---

### Question 11: Systems Manager Parameter Store Tiering and Cross-Account Sharing

A financial services company needs to store 15,000 application parameters and configuration keys in AWS Systems Manager Parameter Store. Some configuration values exceed 6 KB in size. The company also needs to share certain non-sensitive configuration parameters across 10 AWS accounts in the same AWS Organization.

**Which Parameter Store configuration satisfies these requirements?**
- A. Use the **Standard** tier of Parameter Store, which supports up to 20,000 parameters and 8 KB payload sizes.
- B. Upgrade to the **Advanced** tier of Parameter Store to support parameter values up to 8 KB and up to 100,000 parameters. Use **AWS Resource Access Manager (AWS RAM)** to share the parameters across accounts in the organization.
- C. Store the parameters in Amazon DynamoDB instead of Parameter Store.
- D. Compress the values using gzip and store them as base64 strings in the Standard tier.

---

### Question 12: CloudFormation DeletionPolicy vs. UpdateReplacePolicy

A CloudFormation stack manages an Amazon Aurora PostgreSQL DB cluster (`AWS::RDS::DBCluster`) and an Amazon DynamoDB table (`AWS::DynamoDB::Table`). The DevOps Engineer wants to ensure that:
1. If the stack is deleted, a snapshot of the Aurora cluster is created, while the DynamoDB table is retained.
2. If an update causes the Aurora DB cluster or DynamoDB table to be replaced, the old resources are preserved rather than deleted.

**Which resource attributes should be configured?**
- A. Set `DeletionPolicy: Snapshot` on the Aurora cluster, `DeletionPolicy: Retain` on the DynamoDB table, and set `UpdateReplacePolicy: Retain` on both resources.
- B. Set `TerminationProtection: true` on the stack.
- C. Set `DeletionPolicy: Delete` on all resources and take manual snapshots before updating.
- D. Set `UpdatePolicy: Retain` on the resources.

---

### Question 13: AWS CDK Synth, Diff, and Deploy Workflow

A development team adopts the AWS Cloud Development Kit (AWS CDK) in TypeScript to manage application infrastructure. During their local development workflow, engineers need to inspect the generated CloudFormation template, verify the difference between local changes and the currently deployed stack, and deploy the stack without prompt confirmation in CI/CD.

**Which sequence of AWS CDK CLI commands represents this workflow?**
- A. `cdk build` $\rightarrow$ `cdk compare` $\rightarrow$ `cdk push`
- B. `cdk synth` $\rightarrow$ `cdk diff` $\rightarrow$ `cdk deploy --require-approval never`
- C. `cdk generate` $\rightarrow$ `cdk test` $\rightarrow$ `cdk execute`
- D. `cdk compile` $\rightarrow$ `cdk inspect` $\rightarrow$ `cdk apply`

---

### Question 14: Systems Manager Run Command Concurrency and Error Controls

An operations team needs to execute a maintenance script across a fleet of 500 Amazon EC2 instances using AWS Systems Manager Run Command (`AWS-RunShellScript`). The operation must run concurrently on at most 50 instances at a time. If more than 5 instances fail to execute the command successfully, the entire Run Command execution must halt immediately to prevent cascading issues.

**Which parameters of `aws ssm send-command` should be configured?**
- A. `--max-concurrency 50` and `--max-errors 5`
- B. `--batch-size 50` and `--failure-threshold 5`
- C. `--concurrency-limit 50` and `--stop-on-error 5`
- D. `--rate-control "50 instances"` and `--timeout 300`

---

### Question 15: CloudFormation Dynamic Parameter Resolution with SSM

A CloudFormation template needs to look up the latest approved golden AMI ID for Amazon Linux 2023. The golden AMI ID is updated monthly by the central security team and published into AWS Systems Manager Parameter Store under the key `/golden-ami/al2023/latest`.

**What is the most maintainable way to reference this AMI in the CloudFormation template without updating the template code monthly?**
- A. Hardcode the AMI ID as a default parameter in the template.
- B. Define a CloudFormation parameter of type `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` with default value `/golden-ami/al2023/latest`.
- C. Use a custom Bash script in UserData that queries the parameter using the AWS CLI.
- D. Use `!Ref AWS::ImageId`.

---

## Answer Key & Detailed Explanations

### Question 1: CloudFormation CreationPolicy with `cfn-signal` and `cfn-init`

**Answer:** A, C  
**Explanation:** 
`CreationPolicy` is specifically designed for `AWS::AutoScaling::AutoScalingGroup` and `AWS::EC2::Instance` resources. It pauses CloudFormation stack creation until the specified number of success signals (`Count: 3`) are received within the given timeout period (`Timeout: PT15M`). To deliver those signals, the instance `UserData` must run `cfn-init` to process the metadata, followed by `cfn-signal` passing the exit code (`-e $?`) of the initialization commands to the stack and ASG resource.

---

### Question 2: CloudFormation Custom Resources with Lambda S3 Bucket Deletion

**Answer:** A  
**Explanation:** 
CloudFormation cannot delete an S3 bucket if it contains objects or delete markers. CloudFormation does not natively empty buckets upon stack deletion (`DeletionPolicy: Delete` still fails if objects exist). The canonical AWS solution is a Lambda-backed CloudFormation Custom Resource (`Custom::EmptyBucket`). When CloudFormation sends a `Delete` request, the Lambda function programmatically purges all objects and object versions, and posts a `SUCCESS` status back to the CloudFormation response URL, allowing the bucket and stack to delete cleanly.

---

### Question 3: Systems Manager Patch Manager with Patch Baselines and Maintenance Windows

**Answer:** A, C  
**Explanation:** 
AWS Systems Manager Patch Manager uses **Patch Baselines** to define auto-approval rules (e.g. severity `Critical`, delay `5 days`). To schedule execution, an **SSM Maintenance Window** defines the time window, duration (3h), and cutoff (1h, preventing tasks from starting in the final hour). The registered task invokes the `AWS-RunPatchBaseline` SSM document with `Operation: Install` and `RebootOption: RebootIfNeeded`.

---

### Question 4: Systems Manager State Manager Drift Correction

**Answer:** A  
**Explanation:** 
AWS Systems Manager **State Manager** is designed to maintain instances in a defined state and correct configuration drift automatically. By associating an SSM document (or Ansible playbook via `AWS-ApplyAnsiblePlaybooks`) that enforces configuration settings on a recurring schedule (e.g., every 30 minutes), State Manager checks compliance and automatically re-applies the required configuration if any drift is detected.

---

### Question 5: Elastic Beanstalk Immutable Deployments vs. Rolling with Additional Batch

**Answer:** D  
**Explanation:** 
The `Immutable` deployment policy creates an entirely new, temporary Auto Scaling Group behind the load balancer, provisions the new version, and tests instance health. If the deployment succeeds, the new instances are moved to the main ASG and the old instances are terminated. If health checks fail, the temporary ASG is simply terminated, leaving the existing production environment completely untouched with zero downtime and instant recovery. While `Rolling with Additional Batch` maintains capacity, a failure mid-roll leaves a mixed version state requiring a rollback deployment.

---

### Question 6: CloudFormation StackSets Organizational Auto-Deployment

**Answer:** A  
**Explanation:** 
AWS CloudFormation StackSets with **Service-Managed Permissions** integrates directly with AWS Organizations. By enabling **Automatic Deployment** on the StackSet targeted at an Organizational Unit (OU), CloudFormation automatically deploys the stack into any newly created or invited account that enters that OU, and deletes the stacks when an account is removed from the OU, with zero custom scripting.

---

### Question 7: CloudFormation Nested Stacks vs. Cross-Stack References

**Answer:** B  
**Explanation:** 
CloudFormation **Cross-Stack References** using `Export` and `!ImportValue` are explicitly designed for decoupling stacks managed by different teams with different lifecycles. Crucially, CloudFormation enforces dependency safety: you **cannot delete or modify an exported output** in a parent stack while another stack is actively importing that value (`!ImportValue`), protecting shared networking resources from accidental deletion.

---

### Question 8: Systems Manager Automation Runbook for AMI Baking

**Answer:** A  
**Explanation:** 
AWS Systems Manager Automation runbooks (and EC2 Image Builder, which uses SSM under the hood) provide native, declarative, and auditable orchestration for EC2 lifecycle tasks. SSM Automation supports native actions such as `aws:runInstances`, `aws:runCommand`, `aws:createImage`, and `aws:terminateInstances`, with built-in error handling, rollbacks, and IAM role security.

---

### Question 9: CloudFormation Drift Detection and Remediation

**Answer:** B  
**Explanation:** 
AWS CloudFormation **Drift Detection** enables engineers to detect unmanaged configuration changes made outside of CloudFormation management. It compares the current live state of resources against the template's expected properties and reports exact property-level drift (`MODIFIED`, `DELETED`, `NOT_CHECKED`). Engineers can either manually revert the resource to match the template or update the template to reflect the intentional change.

---

### Question 10: Elastic Beanstalk `.ebextensions` Execution Order

**Answer:** B  
**Explanation:** 
In Elastic Beanstalk:
- `commands`: Execute on the host EC2 instance before the application bundle is unzipped, useful for global host setup and OS packages.
- `container_commands`: Execute **after** the application source code has been extracted to the staging directory (`/var/app/ondeck`), with access to environment variables. They run right before the application is moved to its final live directory (`/var/app/current`), making them ideal for database migrations (`leader_only: true`).

---

### Question 11: Systems Manager Parameter Store Tiering and Cross-Account Sharing

**Answer:** B  
**Explanation:** 
SSM Parameter Store Standard tier supports up to 10,000 parameters and a maximum value size of 4 KB per parameter. The **Advanced** tier increases the parameter size limit to 8 KB, supports up to 100,000 parameters, and enables parameter policies (e.g. expiration). Furthermore, Advanced parameters can be shared across AWS accounts in an AWS Organization using **AWS Resource Access Manager (AWS RAM)**.

---

### Question 12: CloudFormation DeletionPolicy vs. UpdateReplacePolicy

**Answer:** A  
**Explanation:** 
In CloudFormation:
- `DeletionPolicy`: Controls what happens to a resource when the stack is **deleted**. For supported databases (RDS, Aurora), `Snapshot` creates a final snapshot before deletion. For DynamoDB, `Retain` leaves the table intact in the account.
- `UpdateReplacePolicy`: Controls what happens to the existing physical resource when a stack update requires the resource to be **replaced** (e.g., changing immutable properties). Setting `UpdateReplacePolicy: Retain` prevents the old resource from being deleted during the replacement update.

---

### Question 13: AWS CDK Synth, Diff, and Deploy Workflow

**Answer:** B  
**Explanation:** 
The AWS CDK CLI standard commands are:
- `cdk synth`: Synthesizes and outputs the AWS CloudFormation template from CDK construct code.
- `cdk diff`: Compares the local CDK state with the deployed stack in AWS, highlighting added, modified, or deleted resources.
- `cdk deploy --require-approval never`: Deploys the stack into AWS, bypassing interactive approval prompts for security-sensitive changes (such as IAM statement additions), which is essential for non-interactive CI/CD pipelines.

---

### Question 14: Systems Manager Run Command Concurrency and Error Controls

**Answer:** A  
**Explanation:** 
AWS Systems Manager Run Command natively provides velocity and error threshold controls:
- `--max-concurrency`: Specifies the maximum number of instances (or percentage) that can run the command simultaneously (e.g., `50` or `10%`).
- `--max-errors`: Specifies the number of errors allowed before SSM automatically stops sending commands to additional targets (e.g., `5` or `1%`).

---

### Question 15: CloudFormation Dynamic Parameter Resolution with SSM

**Answer:** B  
**Explanation:** 
AWS CloudFormation provides native parameter types for AWS Systems Manager Parameter Store (e.g. `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>`). When a stack is created or updated, CloudFormation dynamically queries SSM Parameter Store for the latest value of `/golden-ami/al2023/latest` at stack execution time, ensuring that the latest golden AMI is consumed automatically without modifying the template.

---

### Scoring Guide
- **14–15 Correct:** Mastery (Ready for DOP-C02 Configuration Management & IaC section)
- **12–13 Correct:** Solid Proficiency (Review specific `.ebextensions` execution order and StackSets parameters)
- **$\le 11$ Correct:** Needs Reinforcement (Review `Domain_2_Configuration_Management_IaC/Study_Guide.md` and `Exercises.md`)
