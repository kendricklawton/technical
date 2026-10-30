# Domain 1: SDLC Automation — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 22% of Exam  
**Format:** 15 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Cross-Account CodePipeline Artifact Bucket Encryption

A Principal DevOps Engineer is designing a multi-account CI/CD pipeline hosted in Account `A` (Central Tools). The pipeline compiles code using AWS CodeBuild and deploys containerized services to an Amazon ECS cluster in Account `B` (Production). All pipeline artifacts stored in an Amazon S3 bucket in Account `A` must remain encrypted at rest, and deployments must operate under the principle of **LEAST PRIVILEGE**.

During pipeline execution, the deployment stage in Account `B` fails with an `AccessDeniedException` when attempting to fetch build artifacts from Account `A`'s S3 bucket.

**Which combination of actions will resolve this issue while enforcing least privilege? (Select TWO)**
- A. Configure the S3 artifact bucket in Account `A` to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions in its key policy specifically to Account `B`'s deployment IAM role.
- B. Attach a bucket policy to the S3 artifact bucket in Account `A` that grants `s3:GetObject*` and `s3:ListBucket` permissions scoped specifically to Account `B`'s deployment IAM role ARN.
- C. Enable the default `aws/s3` AWS-managed KMS key on the S3 bucket in Account `A`, and attach an IAM policy to Account `B`'s deployment role granting `kms:Decrypt` on `*`.
- D. Update Account `B`'s deployment role trust policy to allow `sts:AssumeRole` from the S3 service principal and attach the `AmazonS3ReadOnlyAccess` managed policy.
- E. Configure Amazon S3 Cross-Region Replication (CRR) to replicate all artifacts from Account `A` to an unencrypted staging bucket in Account `B`.

---

### Question 2: CodeDeploy In-Place Deployment Lifecycle Hook Order

An enterprise e-commerce application runs on an Auto Scaling Group of Amazon EC2 instances registered to an Application Load Balancer (ALB). The application is updated using AWS CodeDeploy with an `In-Place` deployment configuration (`CodeDeployDefault.OneAtATime`). To avoid dropping active user checkout sessions, instances must be deregistered from the target group and allow inflight requests to drain before the existing process is stopped. After updating files and starting the service, local health checks must pass before the instance re-registers with the ALB.

**What is the exact chronological sequence of CodeDeploy lifecycle event hooks for this in-place deployment?**
- A. `BeforeBlockTraffic` > `BlockTraffic` > `AfterBlockTraffic` > `ApplicationStop` > `DownloadBundle` > `BeforeInstall` > `Install` > `AfterInstall` > `ApplicationStart` > `ValidateService` > `BeforeAllowTraffic` > `AllowTraffic` > `AfterAllowTraffic`
- B. `ApplicationStop` > `BeforeBlockTraffic` > `BlockTraffic` > `AfterBlockTraffic` > `DownloadBundle` > `BeforeInstall` > `Install` > `AfterInstall` > `ApplicationStart` > `ValidateService` > `BeforeAllowTraffic` > `AllowTraffic` > `AfterAllowTraffic`
- C. `BeforeBlockTraffic` > `BlockTraffic` > `AfterBlockTraffic` > `DownloadBundle` > `BeforeInstall` > `Install` > `AfterInstall` > `ApplicationStop` > `ApplicationStart` > `ValidateService` > `BeforeAllowTraffic` > `AllowTraffic` > `AfterAllowTraffic`
- D. `BeforeInstall` > `Install` > `AfterInstall` > `BeforeBlockTraffic` > `BlockTraffic` > `AfterBlockTraffic` > `ApplicationStop` > `DownloadBundle` > `ApplicationStart` > `ValidateService` > `BeforeAllowTraffic` > `AllowTraffic` > `AfterAllowTraffic`

---

### Question 3: Blue/Green ECS Deployment with Canary Validation

A real-time trading microservice deploys to Amazon ECS on AWS Fargate using AWS CodeDeploy (`ECSBlueGreen`). The engineering organization mandates a canary release policy: exactly 10% of production traffic must be directed to the replacement task set for 15 minutes while synthetic canary monitors verify API error rates. If no CloudWatch alarms fire during this period, 100% of production traffic must transition to the replacement task set. The original task set must be kept running for 60 minutes after traffic cutover to allow an instant rollback if edge-case anomalies emerge.

**Which configuration satisfies these requirements with the LEAST custom scripting?**
- A. Set deployment configuration to `CodeDeployDefault.ECSLinear10PercentEvery15Minutes`, configure two ALB target groups, set production listener to forward to the canary, and set task set termination wait time to 60 minutes.
- B. Set deployment configuration to `CodeDeployDefault.ECSCanary10Percent15Minutes`, configure an ALB target group pair with production and test listeners, and configure the deployment group with a `TerminationWaitTimeInMinutes` of 60.
- C. Deploy an AWS Step Functions state machine that updates ALB target group listener weights to 10/90, sleeps for 15 minutes while checking CloudWatch alarms, shifts weights to 0/100, and triggers task termination after 60 minutes.
- D. Set deployment configuration to `CodeDeployDefault.ECSAllAtOnce`, attach an `AfterAllowTraffic` lifecycle hook Lambda function that routes 10% traffic via Route 53 weighted records, and sleeps for 15 minutes.

---

### Question 4: CodeBuild Private VPC Connectivity for Database Migrations & Package Ingestion

An automated CI/CD pipeline uses AWS CodeBuild to execute database schema migrations and run integration test suites. The build container must connect to an Amazon Aurora PostgreSQL DB cluster deployed across private subnets in an existing VPC. At the same time, the build scripts must download required language dependencies from public registries (`npmjs.org` and `pypi.org`). The project currently fails during dependency resolution when connected to the VPC.

**Which architectural pattern provides the necessary connectivity with the LEAST operational overhead?**
- A. Configure the CodeBuild project to attach to the VPC's private subnets whose route tables direct `0.0.0.0/0` outbound traffic to an AWS NAT Gateway in a public subnet, and allow outbound ports 80/443 in the CodeBuild security group.
- B. Assign public IPv4 addresses to the CodeBuild elastic network interfaces (ENIs) inside the public subnet, attach an Internet Gateway, and update the Aurora security group to permit inbound traffic from the ENI public IPs.
- C. Establish an AWS Site-to-Site VPN from the CodeBuild VPC to an external proxy server, and configure the `buildspec.yml` HTTP proxy environment variables to route package manager traffic over the tunnel.
- D. Deploy AWS PrivateLink VPC interface endpoints for `npmjs.org` and `pypi.org` within the private subnets, and configure private DNS hosted zones in Route 53 to resolve the registries locally.

---

### Question 5: CodePipeline Monorepo Selective Microservice Execution

An engineering division maintains three decoupled backend services (`/services/orders`, `/services/payments`, and `/services/inventory`) in a single monorepo hosted on GitHub. Each microservice has an independent continuous delivery pipeline. Currently, pushing any commit to the `main` branch triggers all three pipelines, creating resource contention, redundant deployments, and excessive CodeBuild costs.

**Which approach triggers only the relevant pipeline for the modified service with the LEAST operational overhead?**
- A. Upgrade the pipelines to CodePipeline V2 and configure declarative Git Triggers with `FilePaths.Includes` and `FilePaths.Excludes` matching each service's directory prefix on the `CodeStarSourceConnection` source action.
- B. Create an Amazon EventBridge rule that intercepts GitHub push webhook events, invokes an AWS Lambda function to inspect commit diffs via the GitHub API, and executes `codepipeline:StartPipelineExecution` for matching services.
- C. Add a pre-build validation script to each CodeBuild project that runs `git diff-tree -r --no-commit-id --name-only HEAD`, and calls `aws codebuild stop-build` with status `SUCCESS` if no changes exist in the service folder.
- D. Decompose the monorepo into three distinct GitHub repositories and configure branch-level triggers on each repository independently.

---

### Question 6: Automated Shift-Left Container Vulnerability Gate in CI/CD

A regulated fintech company requires that all container images built by AWS CodeBuild are scanned for Common Vulnerabilities and Exposures (CVEs). If any vulnerability with a CVSS score $\ge 7.0$ (High or Critical) is detected, the pipeline must immediately halt before the image is pushed to the production Amazon ECR repository, and zero vulnerable images may be stored in ECR.

**How should this security gate be integrated into the CI/CD pipeline?**
- A. In the CodeBuild `buildspec.yml` `build` phase, scan the locally compiled Docker image using an open-source scanner (e.g. Trivy) before running `docker push`; if vulnerabilities $\ge 7.0$ are detected, exit with code 1.
- B. Push the image to Amazon ECR, enable Amazon Inspector enhanced scanning, and use an Amazon EventBridge rule on Inspector findings to trigger an AWS Lambda function that invokes `codepipeline:PutJobFailureResult`.
- C. Deploy the container image to a staging Amazon EKS cluster, configure the AWS Systems Manager distributor to run vulnerability checks on the running pods, and terminate any pod with high severity CVEs.
- D. Configure an S3 artifact scanner with AWS Lambda to inspect the compressed container tarballs in the artifact bucket and invoke an SSM Automation runbook to quarantine the repository on detection.

---

### Question 7: CodeDeploy Auto Scaling Lifecycle Hook Rollback Mechanics

An application is deployed to an Auto Scaling Group using AWS CodeDeploy. The deployment group is configured to automatically deploy revisions to new EC2 instances during scale-out events. During an unexpected traffic surge, the ASG launches a replacement instance, but the application initialization script configured in `appspec.yml` fails on the newly provisioned instance.

**What is the precise sequence of actions taken by AWS CodeDeploy and EC2 Auto Scaling?**
- A. CodeDeploy notifies Auto Scaling by sending an `ABANDON` result to the lifecycle hook. Auto Scaling terminates the failed instance, launches a replacement, and retries the deployment up to the configured max attempts.
- B. CodeDeploy sends a `CONTINUE` result to the lifecycle hook. Auto Scaling places the instance into service, but the ALB health check fails and triggers an immediate termination and replacement cycle.
- C. CodeDeploy notifies Auto Scaling by sending an `ABANDON` result to the lifecycle hook. Auto Scaling terminates the failing instance and permanently decrements the ASG Desired Capacity by 1.
- D. Auto Scaling marks the instance unhealthy, immediately deletes the Launch Template, and initiates a reverse rolling update across all existing instances in the Auto Scaling Group.

---

### Question 8: Secrets Management in CodeBuild without Transcript Exposure

An enterprise build running in AWS CodeBuild requires an administrative token to publish test coverage reports to a private SaaS analytics vendor. Security policy mandates that the secret token must never be committed to source code, must not appear in `buildspec.yml` in plain text, and must never be exposed in plaintext within CodeBuild CloudWatch execution logs, even if verbose logging is enabled.

**How should the DevOps Engineer inject this credential securely into the build environment?**
- A. Store the secret in AWS Secrets Manager and map it under `env: secrets-manager: SAAS_TOKEN: "arn:aws:secretsmanager:...:secret:saas-token:token"` in `buildspec.yml`, which natively masks the secret in console and CloudWatch logs.
- B. Store the secret in AWS Systems Manager Parameter Store as a `String` parameter and reference it under `env: variables: SAAS_TOKEN: "{{resolve:ssm:saas-token}}"` in `buildspec.yml`.
- C. In the `buildspec.yml` `pre_build` phase, execute `aws secretsmanager get-secret-value --secret-id saas-token --query SecretString --output text` and export it as a shell variable without `set -x`.
- D. Store the token as an encrypted environment variable override in the CodePipeline deploy stage action configuration and pass it into CodeBuild via parameter injection.

---

### Question 9: Automated Rollbacks with CloudWatch Alarms in CodeDeploy

A latency-sensitive payments service deploys across an Auto Scaling Group using AWS CodeDeploy with the `CodeDeployDefault.OneAtATime` configuration. While new code versions compile cleanly and pass local systemd health checks, subtle database connection leaks under load cause the Application Load Balancer to emit HTTP 5xx responses. The DevOps team needs CodeDeploy to automatically halt deployments and restore previous code if target 5xx rates exceed 1% over 3 minutes.

**How should the DevOps Engineer configure this automated safety mechanism?**
- A. Create a CloudWatch Alarm monitoring ALB `HTTPCode_Target_5XX_Count` divided by `RequestCount` using Metric Math, add the alarm to the CodeDeploy Deployment Group under **Deployment Alarms**, and enable **Roll back when alarms detect issues**.
- B. Write a scheduled AWS Lambda function that polls ALB CloudWatch metrics every minute and executes `aws deploy stop-deployment` with `--auto-rollback-enabled` if 5xx errors exceed 1%.
- C. Configure the CodeDeploy `ValidateService` lifecycle hook script to loop for 3 minutes executing HTTP requests against the ALB endpoint and return exit code 1 if any request returns 5xx.
- D. Create an Amazon EventBridge rule triggered by ALB CloudWatch alarms that targets an AWS Systems Manager Automation runbook to detach instances from the ALB target group.

---

### Question 10: Multi-Branch Pull Request Testing with GitHub and AWS CodeBuild

An enterprise software team is adopting trunk-based development on GitHub. Developers push feature branches and open pull requests into `main`. The DevOps team needs an automated CI solution that:
1. Automatically executes unit and integration test suites in AWS CodeBuild on every pull request update.
2. Posts granular status checks (Pass/Fail with build log URL) back to the GitHub PR commit status API.
3. Automatically triggers the production AWS CodePipeline workflow only when pull requests are merged into `main`.

**Which architecture satisfies these criteria with the LEAST operational overhead?**
- A. Configure AWS CodeBuild with a GitHub webhook source, set webhook filter groups for `PULL_REQUEST_CREATED`, `PULL_REQUEST_UPDATED`, and `PULL_REQUEST_REOPENED`, enable **Report build statuses to source provider**, and configure CodePipeline to trigger on `main` branch pushes.
- B. Deploy an AWS CodePipeline pipeline configured with a GitHub source action using branch wildcard matching (`*`), and attach a Lambda action to report commit status checks back to GitHub.
- C. Configure Amazon EventBridge to capture GitHub webhook notifications via an Amazon API Gateway endpoint, and invoke an AWS Step Functions state machine to manage branch builds and PR updates.
- D. Launch an Amazon EC2 instance hosting Jenkins with the GitHub Multi-Branch Pipeline plugin to run builds and orchestrate CodePipeline invocations via AWS CLI.

---

### Question 11: Integrating On-Premises Static Analysis via CodePipeline Custom Actions

A corporate security standard requires that all source code pass static analysis through an on-premises enterprise SonarQube scanner located inside a restricted private data center before deployment artifacts are published. The on-premises scanner cannot accept inbound internet traffic, but outbound HTTPS connections to AWS endpoints are permitted through an egress proxy.

**How should this security gate be incorporated into an AWS CodePipeline workflow?**
- A. Define an AWS CodePipeline Custom Action of category `Test`, and run a background worker daemon on the on-premises server that continuously polls `codepipeline:PollForJobs`, executes the local scan, and reports results via `PutJobSuccessResult` or `PutJobFailureResult`.
- B. Configure an AWS CodeBuild project inside an egress-only private subnet to establish a reverse SSH tunnel to the on-premises server and execute the SonarQube CLI over the tunnel.
- C. Deploy an AWS Lambda function inside a VPC connected to the on-premises data center via AWS Direct Connect, and configure the Lambda function to trigger the scan and wait synchronously for 30 minutes.
- D. Add a Manual Approval action to the pipeline that generates a temporary presigned S3 URL for the source code bundle, and send an email alert to the on-premises administrator to download and scan the bundle.

---

### Question 12: Optimizing CodeBuild Maven Build Durations with Local Caching

A mission-critical Java application built with Apache Maven in AWS CodeBuild currently takes 26 minutes per build execution. Analysis shows that over 20 minutes are consumed downloading recurring third-party dependencies from Maven Central because CodeBuild provisions fresh execution containers for each build. The team requires the **LOWEST BUILD DURATION** for consecutive builds on the same host.

**Which caching strategy should the DevOps Engineer configure?**
- A. Enable **Local Caching** on the CodeBuild project with `CUSTOM` cache mode, and configure `cache.paths` in `buildspec.yml` to store `/root/.m2/repository`.
- B. Enable Amazon S3 caching on the CodeBuild project pointing to a versioned bucket in the same region, and specify `/root/.m2/repository` in the `buildspec.yml` cache block.
- C. Create an Amazon Elastic File System (EFS) filesystem, mount it to the CodeBuild project environment, and configure Maven's `localRepository` path to target the EFS mount point.
- D. Pre-bake an Amazon EC2 AMI containing the complete Maven repository dependencies and use it as a custom EC2 builder instance for CodePipeline.

---

### Question 13: Zero-Downtime Serverless Deployments with AWS SAM and Canary Shifting

A serverless payment processing service uses AWS SAM and AWS CodeDeploy to manage AWS Lambda deployments. The team needs to roll out new function versions gradually: 10% of production traffic must be served by the new Lambda version for 10 minutes. If error alarms breach or if integration smoke tests fail prior to traffic shifting, the release must abort immediately without impacting any live customer transactions.

**How should this deployment strategy be declared in the SAM template (`template.yaml`)?**
- A. Under `AWS::Serverless::Function`, configure `AutoPublishAlias: live`, `DeploymentPreference: { Type: Canary10Percent10Minutes, Alarms: [ !Ref ErrorAlarm ], Hooks: { PreTraffic: !Ref PreTrafficLambda } }`.
- B. Under `AWS::Serverless::Function`, configure `AutoPublishAlias: live`, `DeploymentPreference: { Type: Linear10PercentEvery10Minutes, Alarms: [ !Ref ErrorAlarm ] }`, and configure a custom CloudWatch event rule.
- C. Provision an Application Load Balancer with weighted target groups pointing to function version aliases, and use an AWS Lambda function to update target weights every 10 minutes.
- D. Configure Route 53 Weighted DNS records pointing to two separate API Gateway stage endpoints, each referencing distinct Lambda function alias versions.

---

### Question 14: Cross-Region Multi-Site CodePipeline Artifact Distribution

An enterprise maintains an active-active disaster recovery architecture across `us-east-1` (Primary) and `eu-west-1` (Secondary). A single centralized AWS CodePipeline in `us-east-1` must compile code once and deploy identical build artifacts to target compute resources in both regions simultaneously.

**What is the mandatory artifact storage architecture in the CodePipeline definition?**
- A. Define an `artifactStores` map in the pipeline declaration specifying an S3 bucket and Customer Managed Key (CMK) in `us-east-1`, alongside a distinct S3 bucket and regional CMK in `eu-west-1`.
- B. Define a single `artifactStore` pointing to an S3 bucket in `us-east-1`, and enable Amazon S3 Cross-Region Replication (CRR) to replicate artifacts asynchronously to `eu-west-1`.
- C. Configure a global Amazon S3 Multi-Region Access Point (MRAP) as the pipeline's single `artifactStore` location, and grant cross-region IAM permissions to the deployment roles.
- D. Use a single S3 bucket in `us-east-1` for all actions and grant cross-region KMS decrypt permissions on the `us-east-1` default `aws/s3` key to the `eu-west-1` execution role.

---

### Question 15: Automated Multi-Stage Release Governance with Slack Approvals

A financial institution requires an automated approval workflow for production deployments. When a build reaches the production stage gate in AWS CodePipeline, the pipeline must pause. Release metadata (commit author, changelog summary, and commit SHA) must be posted to a restricted Slack channel with interactive "Approve" and "Reject" buttons. The decision must be securely recorded back to CodePipeline within 24 hours.

**Which architecture implements this workflow with the LEAST operational maintenance?**
- A. Add a CodePipeline Manual Approval action targeting an Amazon SNS topic. Subscribe an AWS Lambda function that posts the interactive message to Slack. The button click invokes an Amazon API Gateway endpoint backed by a Lambda function that validates the caller and calls `codepipeline:PutApprovalResult`.
- B. Configure AWS Chatbot with an Amazon EventBridge rule that captures CodePipeline stage state change events and posts release notices to Slack; have operators log in to the AWS Console via SSO to approve.
- C. Use a CodeBuild test action that executes a long-polling Python daemon waiting for an approval file to be uploaded to an Amazon S3 control bucket by the release manager.
- D. Create an Amazon EventBridge rule that intercepts CodePipeline pipeline execution state changes, disables the transition to the production stage, and triggers an SSM Automation runbook.

---

## Answer Key & Detailed Explanations

### Question 1: Cross-Account CodePipeline Artifact Bucket Encryption
**Answer:** A, B  
**Explanation:**  
In cross-account CodePipeline architectures, default AWS-managed KMS keys (`aws/s3`) cannot be used because their key policies are immutable and cannot grant access to external accounts. A Customer Managed Key (CMK) is mandatory in Account `A`. Account `B`'s deployment role must be granted `kms:Decrypt` and `kms:DescribeKey` in the CMK key policy (Action A). Additionally, Account `A`'s S3 bucket policy must explicitly permit `s3:GetObject*` and `s3:ListBucket` for Account `B`'s role (Action B). Options C, D, and E fail because `aws/s3` cannot be shared cross-account, S3 cannot assume external roles, and unencrypted replication introduces security and consistency risks.

---

### Question 2: CodeDeploy In-Place Deployment Lifecycle Hook Order
**Answer:** A  
**Explanation:**  
When instances are behind an ALB/ELB, CodeDeploy initiates traffic deregistration hooks first: `BeforeBlockTraffic` > `BlockTraffic` (agent deregisters instance) > `AfterBlockTraffic` (waits for connection draining). Next, host-level lifecycle hooks run: `ApplicationStop` (executing scripts from the *previous* deployment bundle) > `DownloadBundle` (agent fetches new bundle) > `BeforeInstall` > `Install` > `AfterInstall` > `ApplicationStart` > `ValidateService`. Finally, traffic re-registration hooks execute: `BeforeAllowTraffic` > `AllowTraffic` (agent re-registers instance with target group) > `AfterAllowTraffic`.

---

### Question 3: Blue/Green ECS Deployment with Canary Validation
**Answer:** B  
**Explanation:**  
`CodeDeployDefault.ECSCanary10Percent15Minutes` shifts exactly 10% of traffic to the replacement task set on the first increment, pauses for 15 minutes while monitoring CloudWatch alarms, and shifts the remaining 90% in the second increment. An ALB target group pair (one for production traffic, one optional for test listener) is required. Setting `TerminationWaitTimeInMinutes: 60` keeps the original task set alive for 1 hour after full cutover, allowing immediate zero-downtime rollback if needed. Option A uses a linear configuration that shifts 10% every 15 minutes (taking 150 minutes). Options C and D introduce unnecessary custom scripting.

---

### Question 4: CodeBuild Private VPC Connectivity for Database Migrations & Package Ingestion
**Answer:** A  
**Explanation:**  
When AWS CodeBuild is configured to attach to a VPC, its Elastic Network Interfaces (ENIs) reside in the specified subnets and never receive public IPv4 addresses. Even if placed in a public subnet, CodeBuild cannot communicate directly through an Internet Gateway. To access both private VPC resources (Aurora) and public package repositories (`npmjs.org`), CodeBuild ENIs must be deployed in **private subnets** that route `0.0.0.0/0` outbound traffic through a **NAT Gateway** located in a public subnet. AWS PrivateLink (Option D) does not exist for arbitrary public domain names like `npmjs.org`.

---

### Question 5: CodePipeline Monorepo Selective Microservice Execution
**Answer:** A  
**Explanation:**  
CodePipeline V2 introduces native declarative Git triggers supporting branch, tag, and file path filtering (`FilePaths.Includes` and `FilePaths.Excludes`) directly on GitHub/GitLab source connections. This provides a zero-code, fully serverless solution for monorepo microservice filtering. Option B (EventBridge + Lambda) was the legacy workaround for CodePipeline V1, but carries significantly higher operational overhead. Option C wastes build minutes, and Option D forces an unneeded repository refactoring.

---

### Question 6: Automated Shift-Left Container Vulnerability Gate in CI/CD
**Answer:** A  
**Explanation:**  
True "Shift-Left" security mandates scanning artifacts before they are published to shared registries. Running a container scanner (such as Trivy, Grype, or Snyk) during the CodeBuild `build` or `post_build` phase inspects the locally built Docker image. If vulnerabilities exceeding the CVSS 7.0 threshold are discovered, the script exits with a non-zero exit code (`exit 1`), terminating the build before `docker push` can execute. Option B pushes vulnerable images to ECR before detecting them, violating the requirement that no vulnerable images be stored in ECR.

---

### Question 7: CodeDeploy Auto Scaling Lifecycle Hook Rollback Mechanics
**Answer:** A  
**Explanation:**  
When CodeDeploy is integrated with Auto Scaling lifecycle hooks during scale-out events, a failure of the installation script on a newly launched instance causes the CodeDeploy agent to report failure. CodeDeploy then sends an `ABANDON` action to the Auto Scaling lifecycle hook. Upon receiving `ABANDON`, Auto Scaling immediately terminates the non-functional instance, launches a fresh replacement instance, and re-triggers the lifecycle deployment hook. Desired capacity is not decremented.

---

### Question 8: Secrets Management in CodeBuild without Transcript Exposure
**Answer:** A  
**Explanation:**  
AWS CodeBuild provides native integration with AWS Secrets Manager via the `env: secrets-manager` block in `buildspec.yml`. Secrets retrieved through this mechanism are automatically masked in build execution transcripts and CloudWatch Logs with asterisks (`***`). Option C (using the AWS CLI directly) will expose the plain text secret in stdout and CloudWatch logs if shell debugging (`set -x`) or verbose command echoing is active. Option B uses Parameter Store `String` parameters which are unencrypted and unmasked.

---

### Question 9: Automated Rollbacks with CloudWatch Alarms in CodeDeploy
**Answer:** A  
**Explanation:**  
CodeDeploy Deployment Groups feature built-in **Deployment Alarms**. When configured with CloudWatch Alarms (such as an alarm on ALB 5xx rate calculated via Metric Math), CodeDeploy continuously monitors the alarms during the deployment. If an alarm breaches, CodeDeploy automatically halts the deployment, marks it as failed, and initiates an immediate rollback to the previous application revision. This requires zero custom Lambda functions or EventBridge rules.

---

### Question 10: Multi-Branch Pull Request Testing with GitHub and AWS CodeBuild
**Answer:** A  
**Explanation:**  
AWS CodeBuild natively supports GitHub pull request webhooks and status reporting. By setting webhook filter groups to match `PULL_REQUEST_CREATED`, `PULL_REQUEST_UPDATED`, and `PULL_REQUEST_REOPENED`, CodeBuild triggers builds automatically on every PR update. Checking **Report build statuses to source provider** makes CodeBuild communicate build results directly back to GitHub commit status checks without any intermediate Lambda functions or plugins. CodePipeline triggers separately upon PR merge into `main`.

---

### Question 11: Integrating On-Premises Static Analysis via CodePipeline Custom Actions
**Answer:** A  
**Explanation:**  
When integrating third-party or on-premises systems that cannot receive inbound internet traffic, AWS CodePipeline supports **Custom Actions**. A job worker agent running on-premises calls `codepipeline:PollForJobs` over an outbound HTTPS connection. When a job is detected, the worker acknowledges it, downloads the artifact, runs the SonarQube scan locally, and reports the final status back using `PutJobSuccessResult` or `PutJobFailureResult`. This requires zero inbound firewall openings.

---

### Question 12: Optimizing CodeBuild Maven Build Durations with Local Caching
**Answer:** A  
**Explanation:**  
CodeBuild supports **Local Caching**, which stores cached directories directly on the build host instance. For consecutive builds scheduled on the same host, local caching provides near-instantaneous cache retrieval with zero network transfer overhead, significantly outperforming S3 caching (which must upload and download compressed tar archives over the network for every build execution).

---

### Question 13: Zero-Downtime Serverless Deployments with AWS SAM and Canary Shifting
**Answer:** A  
**Explanation:**  
AWS SAM integrates natively with AWS CodeDeploy for Lambda functions. Specifying `AutoPublishAlias: live` creates published versions and alias pointers. `DeploymentPreference: { Type: Canary10Percent10Minutes }` instructs CodeDeploy to shift 10% of traffic to the new version for 10 minutes before shifting the remaining 90%. Specifying `Hooks.PreTraffic` executes a validation Lambda function against the new version *before* any production traffic shifts, guaranteeing zero-downtime safety.

---

### Question 14: Cross-Region Multi-Site CodePipeline Artifact Distribution
**Answer:** A  
**Explanation:**  
When an AWS CodePipeline executes actions across multiple AWS regions, a single S3 bucket cannot be used. CodePipeline requires the plural `artifactStores` configuration in the pipeline definition. This must specify an S3 bucket and Customer Managed Key in the primary region (`us-east-1`) AND an S3 bucket and regional CMK in the target deployment region (`eu-west-1`). CodePipeline automatically synchronizes the artifacts across regions before executing the regional action.

---

### Question 15: Automated Multi-Stage Release Governance with Slack Approvals
**Answer:** A  
**Explanation:**  
The standard serverless approval pattern in CodePipeline uses a `ManualApproval` action configured with an SNS topic. The SNS notification triggers a Lambda function that formats the release details (parsed from the approval token payload) and sends an interactive Slack message containing approval buttons. When clicked, Slack sends an HTTPS POST request to Amazon API Gateway, invoking a callback Lambda function that verifies the security signature and calls `codepipeline:PutApprovalResult` (`Approved` or `Rejected`).
