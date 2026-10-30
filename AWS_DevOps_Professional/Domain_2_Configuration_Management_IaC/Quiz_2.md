# Domain 2 (Configuration Management & IaC) — Quiz 2: CloudFormation Advanced Patterns

**Topic:** CloudFormation Custom Resources, StackSets, Drift Detection, Git Sync & Change Sets  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: CloudFormation Custom Resource Response Signal Timeout

An infrastructure engineer creates an AWS CloudFormation Custom Resource backed by an AWS Lambda function (`Custom::DatabaseSeed`). During stack creation, the Lambda function executes its database insertion logic cleanly within 12 seconds, but the CloudFormation stack remains in the `CREATE_IN_PROGRESS` state for exactly 1 hour before failing with:  
`Custom Resource failed to stabilize in expected time.`

**What caused this 60-minute stack freeze, and how is it resolved?**
- A. The Lambda function failed to issue an HTTPS PUT request containing a valid JSON payload (`Status: "SUCCESS"`) to the pre-signed Amazon S3 `ResponseURL` provided in the CloudFormation event data before the Lambda runtime terminated.
- B. The Lambda function exceeded the maximum AWS Lambda execution timeout of 15 minutes, causing CloudFormation to enter a 45-minute backoff retry loop.
- C. CloudFormation lost the IAM permissions required to execute `lambda:InvokeFunction` on the target function ARN during stack execution.
- D. The Lambda function must be deployed inside the same private subnets as the target database and assigned an Elastic IP address to communicate with CloudFormation.

---

### Question 2: CloudFormation StackSets Self-Managed vs. Service-Managed Governance

An enterprise platform architecture team uses AWS CloudFormation StackSets to deploy security baseline tooling across 120 member accounts. The team needs to choose between **Self-Managed Permissions** and **Service-Managed Permissions**.

**Which architectural characteristic accurately distinguishes Service-Managed StackSets when integrated with AWS Organizations?**
- A. Service-Managed Permissions allow StackSets to target AWS Organizations Organizational Units (OUs) directly, manage cross-account execution roles automatically without deploying IAM roles manually, and support **Automatic Deployment** into newly created member accounts.
- B. Self-Managed Permissions integrate natively with AWS Organizations and automatically remove stacks when accounts leave an Organizational Unit without manual role creation.
- C. Service-Managed Permissions require the administrator to manually provision `AWSCloudFormationStackSetAdministrationRole` in the management account and `AWSCloudFormationStackSetExecutionRole` in each target account.
- D. Service-Managed Permissions are restricted to deploying stacks within a single AWS region, whereas Self-Managed Permissions support multi-region deployment concurrency.

---

### Question 3: Evaluating Destructive Resource Replacement in CloudFormation Change Sets

A developer modifies a production CloudFormation template to update the `Port` property of an Amazon RDS PostgreSQL DB Instance (`AWS::RDS::DBInstance`) from 5432 to 5433. Before executing the update, the engineer creates a CloudFormation Change Set (`aws cloudformation create-change-set`).

**How does CloudFormation Change Sets alert the engineer to the operational blast radius of this modification?**
- A. The change set lists the DB instance resource with `Action: Modify`, `Replacement: True`, and specifies `Evaluation: Static`, warning the engineer that executing the change set will destroy the existing physical DB instance and create a brand-new database.
- B. The change set automatically takes a point-in-time snapshot of the database storage and updates the port parameter in-place with zero downtime.
- C. CloudFormation rejects the change set creation request with a validation error because database port properties cannot be modified after creation.
- D. The change set flags the resource with `Action: Add` and routes traffic through an Amazon Route 53 CNAME record to avoid database downtime.

---

### Question 4: Detecting and Remediating Out-of-Band CloudFormation Configuration Drift

A security compliance scan reveals that a systems engineer manually added an unauthorized ingress rule (TCP port 8080 from `0.0.0.0/0`) to an Amazon EC2 Security Group managed by a production CloudFormation stack. CloudFormation Drift Detection reports the security group resource status as `MODIFIED`.

**What is the correct procedure to remediate this configuration drift while preserving IaC fidelity?**
- A. Execute CloudFormation Drift Detection, inspect the Drift Details report to identify the exact unauthorized rule properties, and either update the Git template to incorporate approved changes via Change Set OR manually delete the unauthorized ingress rule via the EC2 console/CLI.
- B. Initiate a stack deletion using `DeletionPolicy: Retain` on the security group, update the template in Git, and deploy a brand-new CloudFormation stack.
- C. Run the CloudFormation helper script `cfn-hup` on the target EC2 instances to revert the local operating system network configuration.
- D. Execute `aws cloudformation cancel-update-stack` to instruct CloudFormation to forcibly overwrite the security group with the template state.

---

### Question 5: StackSet Multi-Account Deployment Concurrency and Failure Fault Tolerance

A DevOps engineer uses AWS CloudFormation StackSets to deploy an updated IAM security policy across 80 AWS accounts. The rollout must proceed rapidly across accounts, but if more than 4 accounts encounter deployment failures (e.g. due to service limits or missing dependencies), the entire StackSet operation must halt immediately to limit blast radius.

**Which parameters on `aws cloudformation update-stack-set` enforce these operational boundaries?**
- A. Configure `--max-concurrent-count 20` (or `--max-concurrent-percentage 25`) and set `--failure-tolerance-count 4` (or `--failure-tolerance-percentage 5`).
- B. Configure `--concurrency-limit 80` and set `--error-threshold 4` with `--stop-on-failure true`.
- C. Set `--batch-size 20` and set `--max-failure-limit 4` in the StackSet regional parameters.
- D. Configure `--region-concurrency-type PARALLEL` and set `--retain-stacks-on-account-removal true`.

---

### Question 6: Modular CloudFormation Architectures with `AWS::Include` Transforms

A cloud platform engineering team maintains standardized security group rules and VPC endpoint configurations that must be reused across 35 independent CloudFormation templates without duplicating YAML definitions or managing hard-coded parameter files.

**Which native CloudFormation capability allows injecting modular YAML snippets from Amazon S3 into parent templates?**
- A. Use the **`Transform: AWS::Include`** macro in the template `Resources` or `Mappings` section, specifying the S3 URL of the modular snippet in the `Location` property.
- B. Use the `Transform: AWS::Serverless-2016-10-31` macro to automatically compile external YAML files into CloudFormation nested stacks.
- C. Store YAML snippets as string parameters in AWS Systems Manager Parameter Store and resolve them using dynamic references.
- D. Deploy an AWS Step Functions state machine that concatenates YAML text files using a Lambda function before invoking `CreateStack`.

---

### Question 7: Native GitOps Synchronization with CloudFormation Git Sync

A development organization seeks to adopt a GitOps deployment workflow for their CloudFormation infrastructure stacks without provisioning or maintaining AWS CodePipeline pipelines, Jenkins runners, or third-party GitOps controllers. Infrastructure stacks must synchronize automatically whenever pull requests are merged into the `main` branch of their GitHub repository.

**Which native AWS capability delivers this serverless GitOps deployment model?**
- A. Configure **CloudFormation Git Sync**: Establish an AWS CodeConnection to the GitHub repository, link the CloudFormation stack to the repository branch, specify a deployment definition file (`deployment-definition.yaml`), and provide an IAM role assumed by CloudFormation to deploy changes automatically on Git push.
- B. Configure AWS CodeBuild with a GitHub webhook trigger that executes `aws cloudformation deploy` from a custom Docker container on every commit.
- C. Create an Amazon EventBridge rule matching GitHub push webhook events that targets an AWS Systems Manager Automation runbook executing `cfn-cli`.
- D. Deploy HashiCorp Terraform Cloud with an AWS provider integration to synchronize CloudFormation YAML templates via VCS workspaces.

---

### Question 8: Adopting Existing Unmanaged AWS Infrastructure with CloudFormation Resource Import

An enterprise has an existing production Amazon DynamoDB table and S3 bucket provisioned manually in the AWS Console. The platform team needs to bring these unmanaged physical resources under the management of an existing CloudFormation stack without recreating the resources or causing downtime.

**What is the correct procedure to import these existing resources into CloudFormation?**
- A. Update the CloudFormation template to include the logical resource definitions for the DynamoDB table and S3 bucket, create a Change Set of type **`IMPORT`**, provide the physical resource identifiers in the import mapping, and execute the change set.
- B. Delete the existing physical resources in the AWS Console, add their resource definitions to the template, and perform a standard stack update.
- C. Use the `AWS::CloudFormation::CustomResource` type with an AWS Lambda function that calls `dynamodb:UpdateTable` to adopt resource ARNs.
- D. Enable AWS Config continuous recording, select the resources in the AWS Config console, and click "Export to CloudFormation Stack".

---

### Question 9: Protecting Stateful Infrastructure with CloudFormation Termination Protection & Stack Policies

A mission-critical financial ledger database cluster is managed via AWS CloudFormation. The DevOps team must implement safeguards to guarantee that:
1. No administrator or automated pipeline can accidentally delete the CloudFormation stack via the console or CLI.
2. During regular stack updates, the production database cluster resource (`AWS::RDS::DBCluster`) cannot be updated or replaced, even if an engineer modifies the template properties.

**Which combination of CloudFormation security controls satisfies both mandates?**
- A. Enable **Stack Termination Protection** on the CloudFormation stack (to block `DeleteStack` calls), and attach a **Stack Policy** to the stack containing an explicit `Deny` statement on `Update:*` actions targeting the database logical resource ID.
- B. Attach a Service Control Policy (SCP) to the AWS account that denies `cloudformation:DeleteStack`, and set `DeletionPolicy: Retain` on all database resources.
- C. Enable S3 Object Lock on the CloudFormation template bucket in Compliance mode, and configure IAM permission boundaries on all admin roles.
- D. Configure an AWS Config rule that triggers an Amazon SNS alert whenever a stack update change set modifies the database cluster resource.

---

## Answer Key & Detailed Explanations

### Question 1: CloudFormation Custom Resource Response Signal Timeout
**Answer:** A  
**Explanation:**  
When CloudFormation invokes an `AWS::CloudFormation::CustomResource` backed by Lambda, it passes a pre-signed S3 URL in the event payload under `ResponseURL`. CloudFormation enters `CREATE_IN_PROGRESS` and waits for an HTTPS PUT request containing a JSON response payload (`Status: "SUCCESS"` or `"FAILED"`, `PhysicalResourceId`, etc.). If the Lambda function crashes, throws an unhandled exception, or exits without sending this response, CloudFormation will hang for exactly **1 hour** before timing out. Developers should use the `cfn-response` module wrapped in a `try/finally` block.

---

### Question 2: CloudFormation StackSets Self-Managed vs. Service-Managed Governance
**Answer:** A  
**Explanation:**  
**Service-Managed StackSets** integrate natively with AWS Organizations. They allow deploying stacks across entire Organizational Units (OUs), manage cross-account execution roles automatically without requiring manual IAM configuration in target accounts, and feature **Automatic Deployment**, which automatically deploys stacks to new accounts as soon as they join an OU. `Self-Managed` StackSets require manual deployment of administration and execution roles across accounts.

---

### Question 3: Evaluating Destructive Resource Replacement in CloudFormation Change Sets
**Answer:** A  
**Explanation:**  
In AWS CloudFormation, changing immutable properties of certain resources (such as the `Port` on an `AWS::RDS::DBInstance` or the `AvailabilityZone` on an `AWS::EC2::Instance`) requires **Replacement** (`Requires: Replacement`). CloudFormation Change Sets explicitly warn engineers of destructive actions by setting `Action: Modify` and `Replacement: True`. If applied, CloudFormation provisions a new instance and terminates the old one, leading to total data loss unless a snapshot or `UpdateReplacePolicy: Retain` is configured.

---

### Question 4: Detecting and Remediating Out-of-Band CloudFormation Configuration Drift
**Answer:** A  
**Explanation:**  
CloudFormation **Drift Detection** compares the physical property values of live AWS resources against the expected property values defined in the stack template. When drift is detected (`MODIFIED`), the recommended remediation is either: 1) update the Git template to incorporate the approved live configuration changes and apply a stack update; or 2) manually alter the live resource back to the properties declared in the template, bringing the resource back into `IN_SYNC` status.

---

### Question 5: StackSet Multi-Account Deployment Concurrency and Failure Fault Tolerance
**Answer:** A  
**Explanation:**  
In CloudFormation StackSets operations (`create-stack-instances`, `update-stack-set`):  
- `--max-concurrent-count` (or `--max-concurrent-percentage`): Controls how many accounts/regions are deployed to simultaneously.  
- `--failure-tolerance-count` (or `--failure-tolerance-percentage`): Sets the maximum number of account deployments that can fail before CloudFormation halts the entire StackSet operation, containing the failure blast radius.

---

### Question 6: Modular CloudFormation Architectures with `AWS::Include` Transforms
**Answer:** A  
**Explanation:**  
The **`AWS::Include`** transform is a native CloudFormation macro that allows inserting external boilerplate YAML or JSON templates hosted in Amazon S3 directly into parent templates at execution time. This allows organizations to maintain centralized, versioned snippets (such as common Security Groups or standard tagging mappings) in S3 and reference them across dozens of templates via `Transform: AWS::Include` and `Location: s3://...`.

---

### Question 7: Native GitOps Synchronization with CloudFormation Git Sync
**Answer:** A  
**Explanation:**  
**CloudFormation Git Sync** provides native GitOps management directly within CloudFormation. By establishing a connection via AWS CodeConnections to a Git repository (GitHub, GitLab, Bitbucket) and configuring a `deployment-definition.yaml` file in the repo, CloudFormation monitors repository branch commits, creates change sets automatically, and synchronizes infrastructure changes on Git push without needing CodePipeline or external runner infrastructure.

---

### Question 8: Adopting Existing Unmanaged AWS Infrastructure with CloudFormation Resource Import
**Answer:** A  
**Explanation:**  
CloudFormation supports importing existing unmanaged AWS resources into a stack via **Resource Import**. The engineer updates the CloudFormation template to include the logical definitions matching the existing physical resources, initiates a Change Set of type **`IMPORT`**, maps the logical resource IDs to the physical resource identifiers (e.g. table name or bucket name), and executes the change set. CloudFormation adopts the resources without recreating or interrupting them.

---

### Question 9: Protecting Stateful Infrastructure with CloudFormation Termination Protection & Stack Policies
**Answer:** A  
**Explanation:**  
To provide defense-in-depth for stateful production stacks:  
1. **Stack Termination Protection** explicitly blocks `DeleteStack` API actions, preventing accidental stack deletion until protection is manually disabled.  
2. A **Stack Policy** is a JSON policy attached to the stack that defines which update actions can be performed on specific resources. Defining `Effect: Deny, Action: Update:*, Principal: *, Resource: LogicalResourceId/DatabaseCluster` ensures that even during template updates, CloudFormation blocks any modification or replacement of the protected database resource.
