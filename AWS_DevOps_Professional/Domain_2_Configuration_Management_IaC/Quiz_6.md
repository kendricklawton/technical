# Domain 2 (Configuration Management & IaC) — Quiz 6: CDK, Guard & Policy-as-Code

**Topic:** AWS CDK Constructs, CloudFormation Guard, Hooks & Policy-as-Code Governance  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: AWS CDK Construct Abstraction Hierarchy & Custom L3 Governance

A platform engineering organization is developing a standardized internal developer platform using the AWS Cloud Development Kit (CDK) in TypeScript. The platform team needs to provide development teams with pre-approved, hardened microservice infrastructure templates that automatically enforce TLS 1.3 termination, provision a multi-AZ Application Load Balancer, configure AWS WAF WebACL associations, and bundle ECS Fargate tasks with AWS Distro for OpenTelemetry sidecars.

**Which CDK construct level and packaging pattern should the platform team author to distribute this standardized capability with the HIGHEST level of architectural encapsulation?**
- A. Author custom Level 3 (L3) Solution Constructs that compose multiple L2 primitives (`alb.ApplicationLoadBalancer`, `ecs.FargateService`, `wafv2.CfnWebACLAssociation`) into a unified, reusable class, distributing it internally via AWS CodeArtifact.
- B. Author custom Level 1 (L1) Cfn Primitives that map 1:1 to CloudFormation resources (`CfnLoadBalancer`, `CfnService`, `CfnWebACL`), overriding resource attributes in a base class distributed via npm.
- C. Author Level 2 (L2) AWS Curated Constructs with custom property overrides, distributing raw synthesized CloudFormation JSON templates into an Amazon S3 bucket.
- D. Author CloudFormation Macros using Python Lambda functions that intercept raw CloudFormation templates during synthesis, injecting required WAF and telemetry properties at deploy time.

---

### Question 2: CloudFormation Guard (`cfn-guard`) CI/CD Shift-Left Validation

An enterprise mandates that no CloudFormation template can define an S3 bucket with public read access, unencrypted EBS volumes, or security groups allowing unrestricted ingress on port 22 (`0.0.0.0/0`). Compliance evaluation must execute inside developer pre-commit hooks and AWS CodeBuild stages *before* initiating any CloudFormation stack deployment, failing the pipeline immediately upon policy violation without consuming AWS API rate limits.

**Which implementation satisfies these requirements with the LEAST operational overhead?**
- A. Integrate AWS CloudFormation Guard (`cfn-guard`) into the build pipeline, executing `cfn-guard validate --rules rules.guard --data template.json` against synthesized templates, and failing the build step if non-zero exit codes are returned.
- B. Deploy AWS Config managed rules (`s3-bucket-public-read-prohibited`, `encrypted-volumes`, `incoming-ssh-disabled`) in the target accounts and evaluate resource compliance asynchronously post-deployment.
- C. Author an AWS Lambda function invoked as a custom CloudFormation Macro that parses the template JSON in the pipeline and sends an SNS alert to security administrators.
- D. Configure an AWS Systems Manager State Manager association that continuously scans deployed EC2 instances and S3 buckets, automatically applying remediation runbooks upon detecting non-compliance.

---

### Question 3: Proactive Stack Enforcement via AWS CloudFormation Hooks

A corporate security officer requires that all Amazon S3 buckets created in an AWS account must have S3 Object Lock and Server-Side Encryption with Customer Managed Keys (SSE-KMS) enabled. Unlike detective controls that alert or remediate after resources are provisioned, CloudFormation must proactively intercept template deployment and **refuse to provision** non-compliant resources before any physical infrastructure is allocated.

**Which mechanism provides proactive, pre-provisioning compliance enforcement natively within the CloudFormation deployment engine?**
- A. Register a CloudFormation Hook targeting `AWS::S3::Bucket` with `TargetFilters` for `BeforeCreate` and `BeforeUpdate`, setting the hook's `FailureMode` to `FAIL` to reject the stack operation if encryption or lock properties are missing.
- B. Create an EventBridge rule matching `CreateBucket` API calls from CloudFormation in CloudTrail, triggering an AWS Lambda function that immediately calls `DeleteBucket` on non-compliant buckets.
- C. Attach a CloudFormation Stack Policy to all stacks with a statement denying `Update:Modify` and `Update:Delete` actions on `AWS::S3::Bucket` resources across the account.
- D. Deploy AWS Config Conformance Packs with proactive evaluation enabled, relying on CloudWatch Alarms to trigger rollback of the deploying CloudFormation stack.

---

### Question 4: Deterministic CDK Deployments & `cdk.context.json` Management

A DevOps team manages multi-account AWS CDK infrastructure. A pipeline deploys a CDK application that provisions an Amazon ECS service inside an existing shared VPC looked up using `Vpc.fromLookup({ vpcId })`. The pipeline succeeds in the test account but fails unexpectedly when deployed via an automated CI/CD runner because the runner environment lacks permissions to call `ec2:DescribeVpcs`.

**Why does this failure occur, and how should the team architect their CDK project to guarantee deterministic, isolated builds in CI/CD?**
- A. The `Vpc.fromLookup` call performs dynamic environment queries during CDK synthesis; the resulting environment context must be cached in `cdk.context.json` and committed to version control so synthesis in CI/CD is fully deterministic without requiring live AWS API credentials.
- B. Dynamic lookups require runtime IAM credentials with `AdministratorAccess` on the CI/CD runner; granting temporary STS credentials to the build agent resolves the lookup failure.
- C. CDK forbids querying existing infrastructure via constructs; the team must refactor the application to define the VPC directly within the same CDK stack.
- D. The `cdk.context.json` file contains sensitive account credentials that must never be checked into version control; the pipeline must run `cdk context --clear` before every synthesis.

---

### Question 5: Granular Stack Updates Protected by CloudFormation Stack Policies

A mission-critical CloudFormation stack contains a production Amazon RDS Multi-AZ DB instance and an Auto Scaling Group. To prevent accidental database downtime, the engineering director applies a Stack Policy denying `Update:*` actions on the `AWS::RDS::DBInstance` logical resource. During a scheduled maintenance window, an urgent database parameter change requires updating the DB instance parameter group without recreating or replacing the database.

**How can an authorized DevOps engineer execute this specific update while keeping the baseline Stack Policy active for all other operations?**
- A. Execute the `aws cloudformation update-stack` CLI command passing the `--stack-policy-during-update-body` parameter with a temporary policy granting `Update:Modify` on the specific RDS resource, which overrides the stack policy exclusively for the duration of that single update operation.
- B. Delete the existing Stack Policy using `aws cloudformation delete-stack-policy`, perform the stack update, and recreate the original Stack Policy post-deployment.
- C. Modify the IAM permissions of the deployment role to include `cloudformation:IgnoreStackPolicy`, which automatically bypasses all stack policy restrictions.
- D. Edit the CloudFormation template directly to remove the stack policy metadata block and deploy the template using `--force-update`.

---

### Question 6: AWS CDK Assets Management, ECR Publishing & Bootstrap Roles

A DevOps engineer is implementing an AWS CDK pipeline deploying containerized microservices to Amazon ECS across multiple staging and production accounts. During `cdk synth`, the CDK application packages local Dockerfiles into container assets. However, during `cdk deploy` to the production account, the deployment fails with an `AccessDeniedException` while pushing container images to Amazon ECR.

**What is the root cause of this failure, and how should the multi-account CDK bootstrapping be configured?**
- A. The target production account was bootstrapped with standard legacy bootstrapping; modern CDK asset deployments require modern bootstrapping (`cdk bootstrap aws://PROD_ACCOUNT/REGION --trust CICD_ACCOUNT`) to provision the `ImagePublishingRole` with cross-account trust to the CI/CD pipeline execution role.
- B. The CDK deployment role requires `ecr:CreateRepository` permissions in the CI/CD management account rather than the target workload account.
- C. Docker buildkit daemon is not running on the target ECS host instances in the production account to receive the pushed images.
- D. CDK container assets cannot be deployed cross-account natively; container images must be manually built and pushed using external bash scripts prior to invoking `cdk deploy`.

---

### Question 7: CDK Pipelines (`pipelines.CodePipeline`) Self-Mutation & Multi-Stage Promotion

A fintech organization uses the modern AWS CDK `pipelines.CodePipeline` construct to deploy applications across `Alpha`, `Beta`, and `Prod` accounts. A developer updates the CDK pipeline code itself to add a new security scanning stage between `Beta` and `Prod`.

**How does `pipelines.CodePipeline` handle modifications to its own pipeline structure and stages?**
- A. The pipeline executes an automated `UpdatePipeline` (self-mutation) stage immediately following the synthesis stage, which applies CloudFormation updates to the pipeline stack itself and restarts pipeline execution if structural changes occurred.
- B. The pipeline execution terminates with a `PipelineDefinitionModifiedException`, requiring an administrator to manually run `cdk deploy PipelineStack` from a local CLI workstation.
- C. CDK pipelines cannot modify their own architecture; all pipeline structure modifications must be deployed via AWS CloudFormation StackSets from the organization management account.
- D. The pipeline provisions a shadow pipeline stack with the new stages, performs a blue/green swap of CodePipeline ARN identifiers, and deletes the legacy pipeline stack.

---

### Question 8: Implementing Custom CloudFormation Registry Extensions via Private Hooks

An enterprise cloud security team wants to enforce tagging standards (`CostCenter`, `Environment`, `Owner`) across all AWS CloudFormation deployments in 50 AWS accounts. If any resource supporting tags is declared without these required keys, the CloudFormation engine must immediately abort stack creation before initiating any resource provisioning API calls.

**Which architecture implements this proactive governance model across all organization accounts with the LEAST ongoing operational overhead?**
- A. Develop a private CloudFormation Hook using the CloudFormation CLI (`cfn`), publish the hook to the private CloudFormation registry, and use CloudFormation StackSets to activate and configure the hook with `FailureMode: FAIL` across all organization member accounts.
- B. Deploy an AWS Config rule that triggers an SSM Automation runbook to delete non-compliant CloudFormation stacks after resources enter `CREATE_COMPLETE`.
- C. Write an Amazon EventBridge rule in the management account listening for `CreateStack` API calls in CloudTrail, triggering a Lambda function to inject missing tags via the AWS CLI.
- D. Author an AWS Service Catalog portfolio containing only approved CloudFormation templates, and restrict IAM users from invoking CloudFormation directly.

---

### Question 9: CDK Aspects for Organization-Wide Security Compliance Validation

A lead cloud architect wants to verify that all Amazon S3 buckets declared across 30 distinct CDK applications enforce SSL/TLS (`aws:SecureTransport`) in their bucket policies before generating CloudFormation templates. The platform team wants to enforce this rule programmatically within the CDK synthesis tree without manually editing every individual bucket construct.

**Which AWS CDK feature should the team utilize to inspect and mutate construct tree nodes during synthesis?**
- A. Implement a custom CDK `Aspect` by defining a class implementing the `IAspect` interface and `visit(node: IConstruct)` method, checking if `node instanceof s3.CfnBucket`, and applying the aspect to the root `App` or `Stage` via `Aspects.of(app).add(...)`.
- B. Create a custom CDK Context provider that intercepts `cdk.json` queries and injects policy statements into the global context cache.
- C. Configure a CloudFormation Drift Detection scheduled task that periodically scans synthesized JSON templates stored in S3.
- D. Author an AWS Step Functions state machine that parses CDK TypeScript source files using regular expressions in CodePipeline.

---

## Answer Key & Explanations

### Question 1: AWS CDK Construct Abstraction Hierarchy & Custom L3 Governance
**Answer:** A  
**Explanation:** 
AWS CDK constructs exist in three distinct abstraction tiers:
- **Level 1 (L1 / Cfn primitives):** Direct 1:1 representations of CloudFormation resources (prefixed with `Cfn`, e.g., `CfnBucket`). They provide no default configuration and require explicit manual specification of all CloudFormation properties.
- **Level 2 (L2 / AWS Curated Constructs):** AWS-curated resource constructs (e.g., `s3.Bucket`, `ecs.FargateService`) that incorporate sensible security defaults, handle IAM policy generation via convenience methods (`grantRead()`), and abstract boilerplate configuration.
- **Level 3 (L3 / Solution Constructs / Architectural Patterns):** High-level compositions of multiple L1/L2 constructs designed to implement complete architectural solutions (e.g., `ApplicationLoadBalancedFargateService`). 

For enterprise platform engineering teams, authoring custom L3 constructs that bundle compute, load balancing, WAF rules, and observability sidecars into a single reusable TypeScript construct distributed via an internal registry (such as AWS CodeArtifact) encapsulates governance and accelerates developer onboarding. Distributing synthesized raw CloudFormation templates (Option C) defeats the programming model of CDK. L1 primitives (Option B) provide no abstraction or sensible defaults.

---

### Question 2: CloudFormation Guard (`cfn-guard`) CI/CD Shift-Left Validation
**Answer:** A  
**Explanation:** 
**AWS CloudFormation Guard** (`cfn-guard`) is an open-source, purpose-built CLI tool and domain-specific language (DSL) for writing policy-as-code rules to validate CloudFormation templates, Terraform JSON, and Kubernetes manifests. Running `cfn-guard validate` operates purely client-side or within a CI build runner (such as AWS CodeBuild). It evaluates declarative `.guard` rules against template data without initiating stack deployments or calling AWS APIs. If violations are detected, it returns a non-zero exit code that halts the CI/CD pipeline immediately ("shift-left").

- Option B (AWS Config) is a detective control that evaluates resources *after* deployment, incurring AWS Config recording charges and potential window-of-vulnerability exposure.
- Option C (Macros) runs during stack execution inside AWS CloudFormation, which is too late to catch errors in local pre-commit or early build stages.
- Option D (SSM State Manager) runs asynchronously against deployed infrastructure and does not prevent non-compliant template deployments.

---

### Question 3: Proactive Stack Enforcement via AWS CloudFormation Hooks
**Answer:** A  
**Explanation:** 
**AWS CloudFormation Hooks** provide proactive compliance enforcement directly inside the AWS CloudFormation provisioning engine. When a stack operation (`CreateStack`, `UpdateStack`, `ChangeSet`) is executed, CloudFormation invokes registered hooks at the `BeforeCreate`, `BeforeUpdate`, or `BeforeDelete` lifecycle phases. If a hook's target filter matches the resource being provisioned (e.g., `AWS::S3::Bucket`) and the resource configuration violates the hook's validation logic, setting `FailureMode: FAIL` instructs CloudFormation to immediately fail and abort the stack deployment before any physical infrastructure is created.

- Option B is detective: CloudTrail events are delivered with a 5–15 minute delay, meaning the bucket already exists and may be exposed.
- Option C (Stack Policies) controls whether existing stack resources can be updated or deleted; it cannot validate specific resource property values (such as SSE-KMS or Object Lock) during creation.
- Option D evaluates after deployment starts and relies on alarms rather than native engine gating.

---

### Question 4: Deterministic CDK Deployments & `cdk.context.json` Management
**Answer:** A  
**Explanation:** 
CDK methods such as `Vpc.fromLookup`, `HostedZone.fromLookup`, and `SSMParameterLookup` perform environmental lookups by querying the AWS account during synthesis. CDK caches the responses from these queries in a local file named `cdk.context.json`. 

AWS best practices dictate that `cdk.context.json` **must be committed to version control**. This guarantees:
1. Synthesizing the CDK application in automated CI/CD build environments is fully deterministic.
2. The CI/CD build runner does not require live AWS network access or extensive IAM read permissions to synthesize templates.
3. Templates do not unexpectedly mutate between test and production deployments due to external environment drift.

Clearing context or failing to commit `cdk.context.json` forces the CI/CD runner to attempt live lookups, leading to permission failures (`ec2:DescribeVpcs`) or non-deterministic syntheses.

---

### Question 5: Granular Stack Updates Protected by CloudFormation Stack Policies
**Answer:** A  
**Explanation:** 
A CloudFormation Stack Policy is an IAM-like JSON document applied directly to a stack that defines which update actions can be performed on specific logical resources. By default, once a stack policy is associated with a stack, all resources are protected by an implicit deny (`Deny *`).

To perform a planned, authorized modification on a protected resource without permanently deleting or revoking the stack policy, AWS CloudFormation provides the `--stack-policy-during-update-body` (or `--stack-policy-during-update-url`) parameter for the `update-stack` command. This temporary policy overrides the permanent stack policy **only for the scope and duration of that single update operation**. Once the update completes, the permanent stack policy resumes full enforcement. Deleting the stack policy (Option B) creates an operational window where all critical resources are unprotected.

---

### Question 6: AWS CDK Assets Management, ECR Publishing & Bootstrap Roles
**Answer:** A  
**Explanation:** 
Modern AWS CDK bootstrapping (`cdk bootstrap`) provisions a standardized suite of cross-account IAM roles and infrastructure in each target account/region, including:
- `DeploymentActionRole`: Assumed by CloudFormation to execute deployments.
- `FilePublishingRole`: Grants access to upload S3 file assets.
- `ImagePublishingRole`: Grants access to build and push container images to the bootstrap Amazon ECR repository.
- `LookupRole`: Grants read-only access to query environment metadata.

In multi-account CI/CD pipelines, target deployment accounts must be bootstrapped with trust relationships established to the pipeline deployment account (`--trust <pipeline-account-id>`). Without this trust, the pipeline's CodeBuild runner cannot assume the target account's `ImagePublishingRole`, resulting in an `AccessDeniedException` when pushing container images to ECR.

---

### Question 7: CDK Pipelines (`pipelines.CodePipeline`) Self-Mutation & Multi-Stage Promotion
**Answer:** A  
**Explanation:** 
The AWS CDK `pipelines.CodePipeline` construct features a built-in **Self-Mutation** stage (named `UpdatePipeline`). Whenever pipeline source code is updated (such as adding stages, changing build waves, or altering construct configurations), the pipeline runs `cdk synth`, generating an updated CloudFormation template for the pipeline stack itself. 

The `UpdatePipeline` stage runs immediately after synthesis and deploys this updated template to AWS CloudFormation. If the pipeline definition changed, CloudFormation updates the CodePipeline resource in place and immediately restarts pipeline execution from the beginning to ensure subsequent stages execute under the newly defined structure. No manual CLI redeployment or shadow stacks are required.

---

### Question 8: Implementing Custom CloudFormation Registry Extensions via Private Hooks
**Answer:** A  
**Explanation:** 
AWS CloudFormation Registry allows organizations to build and register custom extensions, including Resource Types, Modules, and Hooks. 
- A **Private Hook** developed using the CloudFormation CLI (`cfn hook`) executes natively within the CloudFormation deployment pipeline.
- Hooks can be authored in Java or Python to validate properties (such as mandatory tags) on target resources prior to provisioning.
- By publishing the hook into the private CloudFormation registry and utilizing **CloudFormation StackSets** across the AWS Organization, administrators can enforce the hook in all member accounts with `FailureMode: FAIL`.

If any template across the organization attempts to create a resource missing mandatory tags, CloudFormation rejects the stack creation before making any underlying resource API calls.

---

### Question 9: CDK Aspects for Organization-Wide Security Compliance Validation
**Answer:** A  
**Explanation:** 
**CDK Aspects** are a powerful mechanism for applying operations to all constructs within a given scope. Aspects implement the Visitor pattern:
```typescript
export class EnforceTlsOnBuckets implements cdk.IAspect {
  public visit(node: IConstruct): void {
    if (node instanceof s3.CfnBucket) {
      // Validate or inject secure transport policy
    }
  }
}
// Application:
Aspects.of(app).add(new EnforceTlsOnBuckets());
```
When `app.synth()` is called, CDK traverses the construct tree and invokes `visit()` on every construct node. Aspects can be used to validate compliance (throwing compilation/synthesis errors if rules are violated) or mutate constructs (such as automatically injecting mandatory tags or bucket policy statements). This enforces organization-wide governance across entire CDK applications without modifying individual construct instantiations.
