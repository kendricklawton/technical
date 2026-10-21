# Domain 1: SDLC Automation — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 22% of Exam  
**Format:** 15 Deep Scenario-Based Professional Practice Questions with Detailed Explanations

---

### Question 1: Cross-Account CodePipeline Artifact Bucket Encryption

A DevOps Engineer is building a centralized CI/CD pipeline in Account `A` (Management/Tools). The pipeline pulls source code from an AWS CodeCommit repository, builds container images using AWS CodeBuild, and deploys tasks to an Amazon ECS cluster running in Account `B` (Production). 

During pipeline execution, the deployment stage in Account `B` fails with an `Access Denied` error when attempting to read the build artifacts from the Amazon S3 artifact bucket located in Account `A`.

**Which combination of actions will resolve this issue? (Select TWO)**
- A. Configure the S3 artifact bucket in Account `A` to use the default `aws/s3` AWS-managed KMS key.
- B. Configure the S3 artifact bucket in Account `A` to use a Customer Managed Key (CMK). Update the KMS key policy to grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account `B`'s deployment IAM role.
- C. Update Account `B`'s deployment IAM role to assume an IAM role in Account `A` to read the S3 bucket using default encryption.
- D. Update the S3 bucket policy in Account `A` to grant `s3:GetObject*` and `s3:ListBucket` permissions to Account `B`'s deployment IAM role.
- E. Enable S3 Object Ownership with Bucket Owner Enforced on the artifact bucket and grant `s3:PutObjectAcl` to the CodeBuild service role in Account `A`.

---

### Question 2: CodeDeploy In-Place Deployment Lifecycle Hook Order

An application running on an Auto Scaling Group of Amazon EC2 instances behind an Application Load Balancer is updated using AWS CodeDeploy with an `In-Place` deployment configuration (`CodeDeployDefault.OneAtATime`). During deployment, the application process must gracefully finish processing active HTTP transactions, terminate cleanly, install the new version, verify local process health, and then resume receiving ALB traffic.

**What is the correct chronological sequence of CodeDeploy lifecycle event hooks for this in-place deployment?**
- A. `BeforeBlockTraffic` $\rightarrow$ `BlockTraffic` $\rightarrow$ `AfterBlockTraffic` $\rightarrow$ `ApplicationStop` $\rightarrow$ `DownloadBundle` $\rightarrow$ `BeforeInstall` $\rightarrow$ `Install` $\rightarrow$ `AfterInstall` $\rightarrow$ `ApplicationStart` $\rightarrow$ `ValidateService` $\rightarrow$ `BeforeAllowTraffic` $\rightarrow$ `AllowTraffic` $\rightarrow$ `AfterAllowTraffic`
- B. `ApplicationStop` $\rightarrow$ `BeforeBlockTraffic` $\rightarrow$ `BlockTraffic` $\rightarrow$ `DownloadBundle` $\rightarrow$ `Install` $\rightarrow$ `ApplicationStart` $\rightarrow$ `ValidateService` $\rightarrow$ `AllowTraffic`
- C. `BeforeInstall` $\rightarrow$ `Install` $\rightarrow$ `AfterInstall` $\rightarrow$ `ApplicationStart` $\rightarrow$ `ValidateService` $\rightarrow$ `BeforeBlockTraffic` $\rightarrow$ `BlockTraffic` $\rightarrow$ `AllowTraffic`
- D. `BlockTraffic` $\rightarrow$ `ApplicationStop` $\rightarrow$ `DownloadBundle` $\rightarrow$ `Install` $\rightarrow$ `ValidateService` $\rightarrow$ `ApplicationStart` $\rightarrow$ `AllowTraffic`

---

### Question 3: Blue/Green ECS Deployment with Canary Validation

A gaming company deploys microservices to Amazon ECS on AWS Fargate using AWS CodeDeploy with an `ECSBlueGreen` deployment type. The company wants to implement a canary deployment strategy where 10% of production traffic is routed to the new replacement task set for 15 minutes. If no CloudWatch alarms fire during this interval, the remaining 90% of traffic must be routed to the replacement task set. Furthermore, original task sets must be terminated 1 hour after traffic cutover completes.

**How should the DevOps Engineer configure the CodeDeploy deployment group and task definition?**
- A. Configure the deployment group with deployment configuration `CodeDeployDefault.ECSLinear10PercentEvery15Minutes`, set target group pair with two ALB listeners (prod and test), and set original task set termination wait time to 60 minutes.
- B. Configure the deployment group with deployment configuration `CodeDeployDefault.ECSCanary10Percent15Minutes`, configure two target groups on the ALB, set the production listener to forward to the canary, and set `TerminationWaitTimeInMinutes` to 60.
- C. Create a Lambda function that triggers an ALB weighted target group update every 15 minutes and call `UpdateService` on the ECS service.
- D. Configure `CodeDeployDefault.ECSAllAtOnce` with a CloudWatch Alarm trigger on HTTP 500 errors and roll back if alarms trigger within 15 minutes.

---

### Question 4: CodeBuild VPC Egress for Public Packages and Private RDS

A company runs unit tests and automated integration tests in AWS CodeBuild. The build scripts execute Flyway database migrations against an Amazon Aurora PostgreSQL DB cluster deployed in private subnets across two Availability Zones. Simultaneously, the build scripts need to install third-party dependencies from public repositories (`npmjs.org` and `pypi.org`).

Currently, the CodeBuild project fails during the dependency installation step when configured to connect to the VPC.

**What architectural configuration is required to allow CodeBuild to reach both the private database and public dependency registries?**
- A. Attach an Internet Gateway directly to the private subnets containing the CodeBuild VPC elastic network interfaces (ENIs).
- B. Configure CodeBuild to use private subnets that route outbound traffic (`0.0.0.0/0`) through a NAT Gateway located in a public subnet. Ensure CodeBuild security group allows outbound traffic on ports 80/443 and the Aurora security group allows inbound traffic on port 5432 from the CodeBuild security group.
- C. Configure VPC Endpoints for npmjs.org and pypi.org inside the VPC and associate them with the private route tables.
- D. Assign public IP addresses to CodeBuild ENIs and enable dual-stack IPv6 routing in the VPC route table.

---

### Question 5: CodePipeline Monorepo Selective Execution

A development team maintains multiple serverless microservices inside a single AWS CodeCommit monorepo: `/services/orders`, `/services/payments`, and `/services/inventory`. Each microservice has an independent AWS CodePipeline. Currently, any git push to the `main` branch triggers all three pipelines simultaneously, causing redundant builds and deployments.

**Which solution triggers only the relevant pipeline based on the modified folder path with the LEAST operational overhead?**
- A. Create an Amazon EventBridge rule that intercepts CodeCommit pull request merge events, invokes an AWS Lambda function to inspect the commit diff via `git diff-tree`, and uses `codepipeline:StartPipelineExecution` to launch only the affected service's pipeline.
- B. Split the monorepo into three separate AWS CodeCommit repositories and link each repository to its corresponding pipeline.
- C. Configure the CodeCommit source action in CodePipeline with an `IncludedPaths` parameter pointing to `/services/orders/**`.
- D. Write a pre-commit client-side git hook that parses local file paths and pushes to separate branch names per service.

---

### Question 6: Automated Container Vulnerability Scanning in CI/CD

A financial institution requires that all Docker container images built by AWS CodeBuild are scanned for operating system vulnerabilities before being deployed to Amazon EKS. If any vulnerability with a CVSS score $\ge 7.0$ (High/Critical) is detected, the pipeline must immediately halt and notify the security team.

**How should this security gate be incorporated into the pipeline?**
- A. In the CodeBuild `buildspec.yml`, push the image to Amazon ECR. Enable enhanced scanning with Amazon Inspector in ECR. Configure an Amazon EventBridge rule for `Inspector Finding` with severity `HIGH` or `CRITICAL` to trigger an AWS Lambda function that invokes `codepipeline:PutJobFailureResult`.
- B. In the CodeBuild `buildspec.yml` `post_build` phase, run an open-source static vulnerability scanner (e.g. Trivy) against the local Docker image. Parse the JSON scan report using `jq`, and if vulnerabilities with score $\ge 7.0$ exist, exit the build step with a non-zero exit code (`exit 1`).
- C. Allow the image to deploy to EKS and use an Admission Controller to delete pods with high vulnerabilities after deployment.
- D. Configure an S3 artifact scanner that inspects the container zip files before deployment.

---

### Question 7: CodeDeploy Auto Scaling Lifecycle Hook Rollback

A company uses AWS CodeDeploy to deploy revisions to an Auto Scaling Group (ASG). During a scale-out event, new instances are launched, and CodeDeploy automatically deploys the application revision to the new instance. The application startup script fails on the newly launched EC2 instance.

**What happens to the new instance and the Auto Scaling Group?**
- A. The instance is left in `InService` state and CodeDeploy logs the error in the console.
- B. CodeDeploy marks the deployment as failed on that instance. The Auto Scaling lifecycle hook terminates the failed instance, launches a new instance, and retries the deployment indefinitely.
- C. CodeDeploy notifies Auto Scaling of deployment failure via the lifecycle hook (`ABANDON` result). Auto Scaling terminates the failed instance, decrements the desired capacity, and marks the ASG unhealthy.
- D. CodeDeploy sends an `ABANDON` response to the Auto Scaling lifecycle hook. The Auto Scaling Group terminates the failing instance, launches a replacement instance, and re-attempts the deployment. If consecutive failures exceed the deployment group threshold, CodeDeploy fails the deployment.

---

### Question 8: Secrets Management in CodeBuild without Transcript Exposure

An enterprise application built in AWS CodeBuild requires an API token to publish test coverage reports to a private SaaS analytics vendor. The security team mandates that the token must never be visible in plain text in source control, buildspec files, or CodeBuild CloudWatch console logs.

**How should the DevOps Engineer inject this token into the build environment?**
- A. Store the token in AWS Systems Manager Parameter Store as a `String` parameter and reference it in the buildspec under `env: variables`.
- B. Store the token in AWS Secrets Manager. In the `buildspec.yml`, reference the token under `env: secrets-manager: TOKEN_SECRET: "secret-name:secret-key"`.
- C. Store the token in an encrypted S3 bucket, download it using `aws s3 cp` in the `pre_build` phase, and print it to the terminal.
- D. Hardcode the base64-encoded token into the `buildspec.yml` under `env: parameter-store`.

---

### Question 9: Automated Rollbacks with CloudWatch Alarms in CodeDeploy

A critical financial microservice is deployed using AWS CodeDeploy to an Auto Scaling Group using the `CodeDeployDefault.OneAtATime` deployment configuration. To safeguard against application bugs that do not crash the process but generate high HTTP 500 error rates, the team wants to initiate an automated rollback whenever the Application Load Balancer reports HTTP 5xx errors $> 1\%$ for 3 consecutive minutes during or immediately after deployment.

**How should the DevOps Engineer configure this automated safety net?**
- A. Create an Amazon CloudWatch Alarm tracking `HTTPCode_Target_5XX_Count` divided by `RequestCount` over 1-minute evaluation periods. In the CodeDeploy Deployment Group settings, add this alarm to the **Deployment Alarms** section and enable **Roll back when alarms detect issues**.
- B. Write an AWS Lambda function that polls ALB metrics every minute and executes `aws deploy stop-deployment` if the metric threshold is exceeded.
- C. Configure CodeDeploy lifecycle hook `ValidateService` with a bash script that sleeps for 3 minutes and curls the endpoint.
- D. Configure an Amazon EventBridge rule on ALB 5xx errors that targets an AWS Systems Manager Automation runbook.

---

### Question 10: Multi-Branch Pipeline Automation with GitHub and AWS CodePipeline

A company is moving its application repositories to GitHub. Developers work on feature branches and create pull requests into the `main` branch. The DevOps team needs an automated CI solution where:
1. Every push to any feature branch runs unit tests in AWS CodeBuild.
2. Status checks (Pass/Fail) are reported back to GitHub PR commit statuses.
3. Merges to `main` trigger a full deployment pipeline in AWS CodePipeline.

**What is the most maintainable and serverless architecture for this requirement?**
- A. Use AWS CodePipeline with a GitHub source action configured to poll all branches using a wildcard `*`.
- B. Configure AWS CodeBuild with a GitHub webhook source. Set the webhook filter group to trigger on `PULL_REQUEST_CREATED`, `PULL_REQUEST_UPDATED`, and `PULL_REQUEST_REOPENED`. Enable **Report build statuses to source provider**. Configure AWS CodePipeline with a separate GitHub source action listening only on the `main` branch.
- C. Deploy a Jenkins cluster on Amazon EC2 with the GitHub Multi-Branch Pipeline plugin.
- D. Write an AWS Lambda function invoked by API Gateway to clone the repository, run tests locally, and post comments on GitHub.

---

### Question 11: Custom Actions in AWS CodePipeline

A software team uses an on-premises enterprise static code analysis tool (SonarQube) running inside a private corporate data center. The team needs to introduce a code analysis stage into their AWS CodePipeline workflow before artifacts are approved for production. The on-premises SonarQube server cannot be accessed directly from the public internet.

**How should this integration be implemented within AWS CodePipeline?**
- A. Expose the on-premises SonarQube server over an AWS Site-to-Site VPN or AWS Direct Connect, and configure an AWS CodePipeline Custom Action with a job worker polling CodePipeline using `PollForJobs` API from on-premises.
- B. Configure an S3 event notification that sends an email to the security team to manually run the scan.
- C. Open port 9000 on the corporate firewall to allow public AWS CodePipeline IPs to execute remote scans.
- D. Use AWS CodeBuild in a public subnet to SSH directly into the on-premises SonarQube server.

---

### Question 12: CodeBuild Artifact Caching Strategies

A Java-based enterprise application built with Maven in AWS CodeBuild takes 28 minutes to compile. An investigation reveals that 22 minutes of each build are spent re-downloading thousands of Maven dependencies from Maven Central because the build environment is completely ephemeral.

**Which caching strategy reduces build time with the LEAST latency?**
- A. Configure CodeBuild to use Amazon S3 caching pointing to a bucket in the same region.
- B. Configure CodeBuild to use **Local Caching** with `custom` cache mode and specify the local Maven cache directory (`/root/.m2/repository`) in `buildspec.yml`.
- C. Create a custom Amazon Linux AMI with all Maven dependencies pre-installed and specify it in CodeBuild.
- D. Mount an Amazon EFS filesystem directly to the CodeBuild project and run the build inside EFS.

---

### Question 13: Zero-Downtime Blue/Green Lambda Deployments with Traffic Shifting

A serverless application uses AWS CodeDeploy and AWS SAM to manage AWS Lambda deployments. The team wants to test the new Lambda version in production using 10% of real user traffic for 10 minutes. If error metrics remain normal, traffic must shift completely to the new version. If errors spike, the deployment must automatically roll back immediately.

**How should this be configured in the SAM template (`template.yaml`)?**
- A. Under the `AWS::Serverless::Function` resource, add `AutoPublishAlias: live` and `DeploymentPreference: { Type: Linear10PercentEvery10Minutes, Alarms: [ !Ref ErrorAlarm ] }`.
- B. Under the `AWS::Serverless::Function` resource, add `AutoPublishAlias: live` and `DeploymentPreference: { Type: Canary10Percent10Minutes, Alarms: [ !Ref ErrorAlarm ], Hooks: { PreTraffic: !Ref PreTrafficLambda } }`.
- C. Configure Route 53 Weighted records pointing to two separate API Gateway stages.
- D. Use an Application Load Balancer with weighted target groups pointing to Lambda function versions.

---

### Question 14: Secure Cross-Region CodePipeline Replication

An international enterprise operates an active-active deployment across `us-east-1` (N. Virginia) and `eu-west-1` (Ireland). The central CodePipeline runs in `us-east-1`. The pipeline must build the application once and deploy the exact same build artifact to both regions simultaneously.

**What is the required artifact store configuration in the CodePipeline definition?**
- A. Specify a single Amazon S3 bucket located in `us-east-1` for all pipeline actions across both regions.
- B. Configure an `artifactStore` for `us-east-1` and an `artifactStores` mapping in the pipeline definition that specifies a separate Amazon S3 artifact bucket and region-specific Customer Managed Key (CMK) in `eu-west-1`.
- C. Configure Amazon S3 Cross-Region Replication (CRR) on the `us-east-1` bucket to replicate to `eu-west-1` and have CodeDeploy read the replica bucket outside the pipeline.
- D. Configure an AWS Snowball Edge device to transfer artifacts between regions.

---

### Question 15: Pipeline Stage Gate Approval with SNS and Slack

A regulated company requires manual approval by the Security Officer before a build artifact can be deployed to the production environment. The security officer must receive an alert on Slack with details of the pending release, the commit hash, and a link to approve or reject the deployment directly.

**What is the most secure and automated way to construct this workflow?**
- A. Configure a CodePipeline Manual Approval action with an Amazon SNS topic. Subscribe an AWS Lambda function to the SNS topic that formats the release details and posts an interactive message to a Slack channel with approval buttons. The buttons invoke an API Gateway endpoint backed by a Lambda function that calls `codepipeline:PutApprovalResult`.
- B. Have the security officer log into the AWS Management Console with AdministratorAccess and manually approve the pipeline stage.
- C. Use a CodeBuild step that prompts for user input in the terminal and waits for 24 hours.
- D. Create an Amazon EventBridge rule that pauses the pipeline execution by modifying the pipeline IAM role.

---

## Answer Key & Detailed Explanations

### Question 1: Cross-Account CodePipeline Artifact Bucket Encryption

**Answer:** B, D  
**Explanation:** 
When artifacts are passed cross-account in AWS CodePipeline, default AWS-managed KMS keys (`aws/s3`) cannot be used because AWS-managed key policies cannot be modified to grant cross-account access. A Customer Managed Key (CMK) is strictly required. Account `B`'s deployment role must have permission to use the CMK (`kms:Decrypt` and `kms:DescribeKey` in the KMS key policy) AND must have read access to the bucket in Account `A` via the S3 Bucket Policy (`s3:GetObject*`, `s3:ListBucket`).

---

### Question 2: CodeDeploy In-Place Deployment Lifecycle Hook Order

**Answer:** A  
**Explanation:** 
When an instance is registered to an ALB/ELB target group, CodeDeploy executes traffic deregistration hooks first: `BeforeBlockTraffic` $\rightarrow$ `BlockTraffic` $\rightarrow$ `AfterBlockTraffic`. Once the ALB deregistration delay completes and no new traffic reaches the instance, CodeDeploy runs the host-level lifecycle: `ApplicationStop` (running scripts from the *previous* bundle) $\rightarrow$ `DownloadBundle` $\rightarrow$ `BeforeInstall` $\rightarrow$ `Install` $\rightarrow$ `AfterInstall` $\rightarrow$ `ApplicationStart` $\rightarrow$ `ValidateService`. Finally, it executes traffic registration hooks: `BeforeAllowTraffic` $\rightarrow$ `AllowTraffic` $\rightarrow$ `AfterAllowTraffic`.

---

### Question 3: Blue/Green ECS Deployment with Canary Validation

**Answer:** B  
**Explanation:** 
CodeDeploy provides built-in deployment configurations for ECS. `CodeDeployDefault.ECSCanary10Percent15Minutes` shifts 10% of traffic in the first increment, pauses for 15 minutes, and then shifts the remaining 90% in the second increment. The deployment group requires an ALB with two listeners (production traffic listener and optional test traffic listener) and two target groups (blue and green). Specifying a termination wait time of 60 minutes keeps the original task set alive for rollback safety for 1 hour after full traffic routing.

---

### Question 4: CodeBuild VPC Egress for Public Packages and Private RDS

**Answer:** B  
**Explanation:** 
When AWS CodeBuild runs inside a VPC, it attaches ENIs to the specified subnets. VPC ENIs in private subnets cannot access the public internet unless their route table directs default outbound traffic (`0.0.0.0/0`) to a NAT Gateway located in a public subnet with an Internet Gateway. AWS does not provide VPC endpoints for arbitrary public sites like npm or PyPI. The security group on the Aurora cluster must permit ingress from the security group attached to CodeBuild.

---

### Question 5: CodePipeline Monorepo Selective Execution

**Answer:** A  
**Explanation:** 
CodeCommit source actions in AWS CodePipeline do not natively support file-path or directory filtering (`IncludedPaths` is not an S3/CodeCommit source parameter in CodePipeline v1/v2). The standard AWS architectural pattern for monorepos is to trigger an EventBridge rule on CodeCommit repository state changes, which invokes a lightweight Lambda function that calls `codecommit:GetDifferences` (or `GetCommit`), inspects which directory paths were touched, and programmatically initiates `StartPipelineExecution` only on the affected pipeline.

---

### Question 6: Automated Container Vulnerability Scanning in CI/CD

**Answer:** B  
**Explanation:** 
CodeBuild commands that exit with a non-zero exit code (`exit 1`) immediately fail the build phase, preventing the pipeline from advancing to the deployment stage. Running container vulnerability scanners (like Trivy or Grype) locally within the buildspec `post_build` phase provides a synchronous, deterministic gating mechanism. Option A is asynchronous and introduces race conditions because ECR Inspector scans take several minutes to complete, during which CodePipeline would have already transitioned to the deployment stage.

---

### Question 7: CodeDeploy Auto Scaling Lifecycle Hook Rollback

**Answer:** D  
**Explanation:** 
When CodeDeploy is integrated with an Auto Scaling Group, it installs an ASG lifecycle hook (`CodeDeploy-managed-lifecycle-hook`). During scale-out, instances pause in `Pending:Wait`. If the CodeDeploy deployment fails on the new instance, CodeDeploy issues an `ABANDON` lifecycle action. Auto Scaling immediately terminates the unhealthy instance and launches a replacement instance to satisfy desired capacity, re-triggering the deployment.

---

### Question 8: Secrets Management in CodeBuild without Transcript Exposure

**Answer:** B  
**Explanation:** 
AWS CodeBuild provides native integration with AWS Secrets Manager and AWS Systems Manager Parameter Store via the `env: secrets-manager` and `env: parameter-store` stanzas in `buildspec.yml`. When secrets are fetched using these stanzas, CodeBuild automatically masks their values with asterisks (`***`) in the CloudWatch build logs and console outputs, preventing credential leakage in transcripts.

---

### Question 9: Automated Rollbacks with CloudWatch Alarms in CodeDeploy

**Answer:** A  
**Explanation:** 
AWS CodeDeploy includes native integration with Amazon CloudWatch alarms. In the Deployment Group configuration under "Alarms", you can specify one or more CloudWatch alarms. Under "Rollbacks", you can check "Roll back when alarms detect issues". If the specified alarm transitions to the `ALARM` state while a deployment is active, CodeDeploy immediately stops the deployment and initiates an automatic rollback to the last known good revision.

---

### Question 10: Multi-Branch Pipeline Automation with GitHub and AWS CodePipeline

**Answer:** B  
**Explanation:** 
AWS CodeBuild directly supports GitHub webhooks with advanced event filter groups. It can filter on PR creation, update, and re-open events, as well as specific branch regex patterns. CodeBuild has a native setting: "Report build statuses to source provider", which uses GitHub Commit Status API to display green checkmarks or red X marks directly on the pull request. Meanwhile, AWS CodePipeline handles the `main` branch CD deployment cleanly.

---

### Question 11: Custom Actions in AWS CodePipeline

**Answer:** A  
**Explanation:** 
AWS CodePipeline supports **Custom Actions**. A custom action defines an action type with custom provider details. A background job worker running on-premises (or in an EC2 instance connected via VPN/Direct Connect) periodically polls CodePipeline for jobs using the `PollForJobs` API, acknowledges the job (`AcknowledgeJob`), downloads the artifact from S3, runs the scan, and reports results back using `PutJobSuccessResult` or `PutJobFailureResult`.

---

### Question 12: CodeBuild Artifact Caching Strategies

**Answer:** B  
**Explanation:** 
AWS CodeBuild supports two caching modes: S3 caching and Local caching. Local caching caches dependencies directly on the build host (supported modes: Source cache, Docker layer cache, Custom cache). For package managers like Maven, npm, or pip, Local Custom Caching stores the `.m2` folder on the underlying Docker host, eliminating the network latency of downloading and uploading tarballs to Amazon S3 on every build.

---

### Question 13: Zero-Downtime Blue/Green Lambda Deployments with Traffic Shifting

**Answer:** B  
**Explanation:** 
AWS SAM simplifies CodeDeploy traffic shifting via the `DeploymentPreference` attribute. Setting `AutoPublishAlias: live` directs SAM to maintain an alias and publish new versions. Specifying `Type: Canary10Percent10Minutes` shifts 10% of traffic to the new version, waits 10 minutes, and shifts the remaining 90%. CloudWatch alarms linked under `Alarms` trigger instantaneous rollback if thresholds are breached, and `PreTraffic` hooks run validation tests before any traffic shifts.

---

### Question 14: Secure Cross-Region CodePipeline Replication

**Answer:** B  
**Explanation:** 
When an AWS CodePipeline contains actions in regions different from the pipeline itself, the pipeline must define `artifactStores` (plural) as a mapping of regions to artifact buckets. Each region where an action executes must have its own S3 bucket located within that specific region, encrypted with a region-specific Customer Managed Key (CMK). CodePipeline handles replicating the artifact between the regional buckets transparently.

---

### Question 15: Pipeline Stage Gate Approval with SNS and Slack

**Answer:** A  
**Explanation:** 
AWS CodePipeline Manual Approval actions publish a JSON notification payload to an Amazon SNS topic containing approval tokens, stage names, and pipeline execution IDs. By subscribing a Lambda function to this topic, you can format a rich Slack message using Slack Block Kit and interactive buttons. When clicked, the interaction invokes an Amazon API Gateway endpoint that triggers a second Lambda function calling `codepipeline:PutApprovalResult` with `Approved` or `Rejected` and the approval token.

---

### Scoring Guide
- **14–15 Correct:** Mastery (Ready for DOP-C02 SDLC section)
- **12–13 Correct:** Solid Proficiency (Review specific CodeDeploy lifecycle hooks and cross-account KMS policies)
- **$\le 11$ Correct:** Needs Reinforcement (Review `Domain_1_SDLC_Automation/Study_Guide.md` and `Exercises.md`)
