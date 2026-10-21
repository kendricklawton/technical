# Domain 2 (Configuration Management & IaC) — Quiz 6: CDK, Guard & Policy-as-Code

**Topic:** AWS CDK Constructs, CloudFormation Guard, Hooks & Policy-as-Code Governance  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: AWS CDK Construct Levels (L1, L2, L3)

A team building infrastructure using the AWS Cloud Development Kit (CDK) in TypeScript needs to understand construct abstractions:
- Construct A: `CfnBucket` (direct 1:1 mapping to CloudFormation resource `AWS::S3::Bucket`, requiring explicit configuration of all attributes).
- Construct B: `Bucket` (provides sensible defaults, automatic KMS key associations, and convenience methods like `bucket.grantRead(role)`).
- Construct C: `ApplicationLoadBalancedFargateService` (provisions a VPC, ALB, ECS cluster, Fargate task definition, target groups, and security groups in a single statement).

**What are the official construct levels for A, B, and C in the AWS CDK?**
- A. A is L3, B is L2, C is L1.
- B. **A is L1 (Cfn primitives), B is L2 (Curated AWS constructs), and C is L3 (High-level architectural patterns)**.
- C. All three are L2 constructs.
- D. Constructs are not categorized into levels.

---

### Question 2: CloudFormation Guard for CI/CD Policy Validation

An enterprise mandates that no CloudFormation template can define an S3 bucket with public read access or an unencrypted EBS volume. The policy evaluation must execute in developer pre-commit hooks and in AWS CodeBuild **before** any CloudFormation stack operation is initiated.

**Which declarative policy-as-code tool fulfills this requirement?**
- A. AWS Systems Manager State Manager
- B. **AWS CloudFormation Guard** (`cfn-guard`)
- C. AWS Config automatic remediation
- D. Amazon Inspector v2

---

### Question 3: CloudFormation Hooks for Proactive Enforcement

A corporate security officer requires that all Amazon S3 buckets created in an AWS account must have S3 Object Lock enabled. Unlike detective controls that flag non-compliant resources after creation, the requirement states that CloudFormation must **refuse to provision** non-compliant buckets.

**Which feature provides proactive, pre-provisioning compliance enforcement inside the CloudFormation engine?**
- A. AWS Config custom rules
- B. **AWS CloudFormation Hooks**
- C. CloudWatch Metric Alarms
- D. S3 Lifecycle Rules

---

### Question 4: AWS CDK Context & Dynamic Environment Lookups

A developer writes a CDK application that looks up an existing VPC in the AWS account using `Vpc.fromLookup(this, 'ExistingVPC', { vpcId: 'vpc-12345' })`. During `cdk synth`, the CDK CLI queries the AWS account and caches the returned VPC subnet and route table details.

**Where does the AWS CDK cache dynamic context lookups to ensure deterministic future syntheses?**
- A. In the local operating system environment variables.
- B. In the **`cdk.context.json`** file in the project root directory, which should be committed to version control.
- C. In Amazon DynamoDB.
- D. In CloudFormation template metadata.

---

### Question 5: CloudFormation Stack Policies vs. IAM Permissions

An administrator with full `AdministratorAccess` attempts to execute a stack update that replaces an Amazon Aurora cluster. The stack update fails with:
`Action denied by stack policy: Statement denies [Update:Replace] on [LogicalResourceId/DBCluster]`

**Can the administrator override this stack policy during the update API call?**
- A. No, stack policies can never be modified or bypassed.
- B. Yes, if the administrator has the `cloudformation:SetStackPolicy` permission, they can pass an explicit temporary override policy during the update call using `--stack-policy-during-update-body` (or `--stack-policy-during-update-url`).
- C. The stack must be deleted and recreated.
- D. Only the AWS root user can override stack policies.

---

### Question 6: AWS CDK Assets Management and Bootstrap Bucket

When deploying a CDK application containing an AWS Lambda function with TypeScript code and local Docker images, `cdk synth` creates local asset bundles, and `cdk deploy` packages and uploads them.

**What is the role of the `cdk bootstrap` command in preparing the AWS account for CDK deployments?**
- A. It starts a dedicated EC2 instance to manage deployments.
- B. It provisions the **CDK bootstrap stack** (`CDKToolkit`), which includes an Amazon S3 bucket for storing file assets (Lambda zip packages) and an Amazon ECR repository for storing container image assets, along with required IAM execution roles.
- C. It compiles TypeScript into JavaScript.
- D. It installs Node.js on the developer's laptop.

---

### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: AWS CDK Construct Levels (L1, L2, L3)

**Answer:** B  
**Explanation:** 
AWS CDK constructs are categorized into three levels:
- **L1 (CFN Resources):** Auto-generated primitives prefixed with `Cfn` representing direct 1:1 mappings to underlying CloudFormation types.
- **L2 (AWS Constructs):** Hand-crafted by AWS, incorporating security defaults, type-safe APIs, and automated IAM grant methods (`grantRead`, `grantWrite`).
- **L3 (Patterns):** Architectural compositions combining multiple resources across services (e.g. `ApplicationLoadBalancedFargateService`).

---

### Question 2: CloudFormation Guard for CI/CD Policy Validation

**Answer:** B  
**Explanation:** 
**AWS CloudFormation Guard** is an open-source command-line tool that evaluates declarative policy-as-code rules against JSON and YAML CloudFormation templates. Running `cfn-guard validate --rules rules.guard --data template.yaml` validates compliance locally or in a CI pipeline, failing the build immediately if non-compliant resource properties are detected before deploying to AWS.

---

### Question 3: CloudFormation Hooks for Proactive Enforcement

**Answer:** B  
**Explanation:** 
**AWS CloudFormation Hooks** provide proactive evaluation of resource properties *before* CloudFormation creates or updates physical resources. A hook is registered at the account or organization level. When a stack operation begins, the hook intercepts `BeforeCreate` or `BeforeUpdate` events, evaluates compliance rules, and if non-compliant, fails the hook and prevents the resource from being provisioned.

---

### Question 4: AWS CDK Context & Dynamic Environment Lookups

**Answer:** B  
**Explanation:** 
The AWS CDK caches results from environmental lookups (VPC IDs, availability zones, SSM parameters) in `cdk.context.json`. Caching guarantees that subsequent syntheses and CI/CD builds produce deterministic, identical CloudFormation templates without relying on active network API calls to AWS. Committing `cdk.context.json` to source control is an AWS best practice.

---

### Question 5: CloudFormation Stack Policies vs. IAM Permissions

**Answer:** B  
**Explanation:** 
While a CloudFormation Stack Policy protects resources from accidental updates by default, an authorized administrator possessing the `cloudformation:SetStackPolicy` permission can explicitly override the policy for a single update execution by providing a temporary override policy via the `--stack-policy-during-update-body` parameter in the `update-stack` command.

---

### Question 6: AWS CDK Assets Management and Bootstrap Bucket

**Answer:** B  
**Explanation:** 
Running `cdk bootstrap` prepares an AWS environment (account/region pair) to receive CDK deployments. It deploys the `CDKToolkit` CloudFormation stack, provisioning an Amazon S3 bucket for file assets, an Amazon ECR repository for container images, and IAM roles granting CDK the permissions required to synthesize and deploy resources.

---


### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

