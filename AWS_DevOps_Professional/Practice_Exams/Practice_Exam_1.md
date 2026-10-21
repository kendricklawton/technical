# AWS DevOps Engineer - Professional (DOP-C02) Practice Exam 1: SDLC & Deployment Pipeline Defense

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes (≈5 minutes per question; the real exam allows ≈2.4)  
**Passing Target:** 12 / 15 (80%)  
**Domains:** D1 ×4 · D2 ×2 · D3 ×2 · D4 ×2 · D5 ×2 · D6 ×3

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Don't hunt for the option that "can't work" — ask which one best satisfies the constraint. Answer all 15 before opening the answer key at the end.

---

## Question 1 (Domain 1: SDLC Automation)

A payments company exposes a REST API through Amazon API Gateway backed by 14 AWS Lambda functions defined in a single AWS SAM template. A CodePipeline (V2) pipeline builds the template with AWS CodeBuild and deploys it to production with an AWS CloudFormation deploy action. API Gateway invokes every function through a `live` alias, and the team ships about 20 releases per day.

Last week a release was missing an IAM permission, so every invocation of the new function version failed. An existing CloudWatch alarm on the API's 5XX error rate entered ALARM within two minutes, yet customers saw errors for 31 minutes while an engineer reverted the commit and waited for a full pipeline run. Integration tests in the staging account pass reliably and are not the problem.

The CTO now requires that any release that trips the existing alarm is reverted automatically with no human action, that the pipeline remains the only path for forward deployments, and that the change fits the existing SAM-based workflow.

**Which solution meets these requirements with the FASTEST rollback?**

- A. Add `AutoPublishAlias: live` and a `DeploymentPreference` of type `Canary10Percent5Minutes` to each function, listing the existing 5XX alarm under `Alarms` and keeping the pipeline unchanged. CodeDeploy shifts alias weights and, if the alarm fires, returns all traffic to the previous function version.
- B. Add an `onSuccess` condition to the production stage that evaluates the existing 5XX alarm for 10 minutes with a `ROLLBACK` result. If the alarm fires, CodePipeline rolls the stage back by redeploying the artifacts of the last successful pipeline execution.
- C. Register the existing 5XX alarm as a rollback trigger on the production stack with a 10-minute monitoring time. Pipeline deployments inherit the stack's rollback configuration, so a breach makes CloudFormation roll the stack back to the previous template and function versions.
- D. Create an EventBridge rule for the alarm's ALARM state that invokes a Lambda function. The function finds the source revision of the previous successful execution and starts a new pipeline execution with a source revision override for that commit.

---

## Question 2 (Domain 2: Configuration Management and IaC)

A platform team governs CloudFormation usage across 25 AWS accounts in one AWS Organizations OU. Application teams deploy through CodePipeline, but on-call engineers also create and update stacks from the console during incidents. The security team already maintains 60 AWS CloudFormation Guard rules in a versioned Amazon S3 bucket in the security account; today the rules run only as a pull-request check in CodeBuild.

An audit found 11 non-compliant resources — unencrypted RDS instances and security groups open to 0.0.0.0/0 on port 22 — created by console-driven stack updates that never passed through a pipeline. The pipelines' CodeBuild images run Amazon Linux 2023, and three teams are converting their templates from JSON to YAML.

The new control must evaluate every stack create and update in all 25 accounts (including accounts added to the OU later), block the operation before a non-compliant resource is provisioned, reuse the existing Guard rule files without rewriting them, and pick up rule changes from the security team's bucket automatically.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Deploy an `AWS::CloudFormation::LambdaHook` to every account with a service-managed StackSet. The hook's function bundles the cfn-guard library, downloads the rule files from the security bucket on each invocation, and fails non-compliant targets in FAIL mode.
- B. Deploy an `AWS::CloudFormation::GuardHook` with a service-managed StackSet targeting the OU, pointing its rule location at the security bucket in FAIL mode, and grant the hook's execution role read access through a bucket policy scoped with `aws:PrincipalOrgID`.
- C. Build a private hook type with the CloudFormation CLI whose Python handler evaluates the rules, register it in each account with a StackSet, and activate it in FAIL mode, publishing and re-registering a new version whenever the rules change.
- D. Activate a Guard hook from the console in each account with a per-account rules bucket in FAIL mode, and run a nightly CodeBuild job that copies the latest rule files from the security bucket into all 25 account buckets.

---

## Question 3 (Domain 6: Security and Compliance)

A company's tools account hosts a CodePipeline (V2) pipeline that deploys a CloudFormation stack into a production account in a separate OU. Pipeline artifacts are stored in an S3 bucket in the tools account under the prefix `orders-pipeline/` and are encrypted with a customer managed KMS key in the tools account. The deploy action runs as a cross-account action role in the production account, and CloudFormation uses a separate service role there. The action produces no output artifacts.

The same bucket and KMS key are shared by 30 other pipelines that deploy to 12 other accounts. A security review flagged that production administrators could, in principle, read other teams' artifacts, and that the current setup was copied from a tutorial.

The orders deployment must keep working exactly as it does today. Security wants the cross-account configuration rebuilt to grant only the access this deployment needs.

**Which solution meets these requirements with the LEAST privilege?**

- A. In the key policy and bucket policy, grant the production account root principal `kms:Decrypt`, `kms:DescribeKey`, `s3:GetObject*`, and `s3:ListBucket` on the whole bucket, and let production IAM policies delegate them to the action role, which can pass the CloudFormation service role.
- B. Grant only the production action role `kms:Decrypt` and `kms:DescribeKey` in the key policy with a `kms:ViaService` condition for S3, and `s3:GetObject*` on `orders-pipeline/*` in the bucket policy. It may pass only the CloudFormation service role, scoped to the stack's resources.
- C. Scope the key policy and bucket policy to the production action role and the `orders-pipeline/*` prefix, but attach `AdministratorAccess` to that role and use it as both the action role and the CloudFormation service role so template changes never need IAM updates.
- D. Grant `kms:Decrypt`, `kms:DescribeKey`, and `s3:GetObject*` on `orders-pipeline/*` to any principal whose `aws:PrincipalOrgID` matches the organization, so new accounts need no policy edits, and keep both production roles scoped to the stack's resource types.

---

## Question 4 (Domain 3: Resilient Cloud Solutions)

A media company runs a Java web tier on 24 EC2 instances in an Auto Scaling group behind an Application Load Balancer. Instances take about 8 minutes to become healthy because the application warms a 40 GB local cache from Amazon S3. Releases ship weekly through AWS CodeDeploy in-place deployments, and the deployment group already has alarm-based automatic rollback.

Two of the last five releases contained memory leaks that surfaced roughly 40 minutes after the deployment had completed, when latency alarms fired. Each time, restoring the previous release took more than 30 minutes. The instances run on Graviton-based c7g instances, and the team is evaluating newer instance families.

The team needs a release process that can return production traffic to the previous release within minutes at any point during the first 2 hours after a release, without rebuilding capacity from scratch.

**Which solution meets these requirements with the FASTEST rollback?**

- A. Switch to blue/green deployments that copy the Auto Scaling group and terminate the original instances 2 hours after traffic is rerouted. If a defect appears, stop the deployment with rollback so CodeDeploy reroutes the ALB to the still-running originals.
- B. Replace CodeDeploy with Auto Scaling instance refreshes that roll out a new launch template version per release with checkpoints and automatic rollback, and start a new refresh with the previous launch template version if a defect appears after a refresh completes.
- C. Manage the group with a CloudFormation `AutoScalingReplacingUpdate` policy so each release creates a new Auto Scaling group behind the ALB, and run a stack update that points back to the previous AMI and application version if a defect appears after the update completes.
- D. Keep in-place deployments, change the deployment configuration to `CodeDeployDefault.HalfAtATime`, and when a defect appears after a deployment completes, redeploy the last known good revision with the same configuration and automatic rollback enabled.

---

## Question 5 (Domain 1: SDLC Automation)

A logistics company is launching a new Amazon ECS service on AWS Fargate behind an existing Application Load Balancer. The service, its two target groups, and its listener rules are defined in a CloudFormation template that a CodePipeline CloudFormation action deploys. The company has no CodeDeploy resources and prefers not to add any.

Each release must route a test listener rule to the new tasks before production traffic, run a Lambda function that executes contract tests against the test listener and can block the release, shift production traffic all at once after the tests pass, keep the previous tasks running for a 30-minute bake period, and roll back automatically if either of two existing CloudWatch alarms fires during the release.

The service runs four tasks of a Go binary, and the team expects traffic to grow fivefold next year.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Set the service's deployment controller to `CODE_DEPLOY`, create a CodeDeploy application and deployment group with both target groups and the alarms, generate an AppSpec file with an `AfterAllowTestTraffic` hook, and replace the pipeline's CloudFormation action with an Amazon ECS (Blue/Green) action.
- B. Add the `AWS::CodeDeployBlueGreen` transform and an `AWS::CodeDeploy::BlueGreen` hook so CloudFormation runs a CodeDeploy blue/green deployment when the task definition changes, configuring the test listener, the test function, and a 30-minute wait in the hook, with both alarms as stack rollback triggers.
- C. Configure the service's deployment strategy in the template as Amazon ECS blue/green, adding the alternate target group, production and test listener rules, a 30-minute bake time, a lifecycle hook that invokes the test function, and both alarms with rollback enabled.
- D. Run two ECS services behind weighted target groups on the production listener, and have the pipeline invoke a Step Functions state machine that deploys the idle service, tests it through a test listener, flips the weights, waits 30 minutes, and reverts on alarm.

---

## Question 6 (Domain 4: Monitoring and Logging)

A SaaS company deploys 70 microservices to Amazon ECS with CodeDeploy blue/green deployments that send 10% of production traffic to the replacement task set for 15 minutes. Each service has its own pair of ALB target groups, which swap roles on every deployment. CodeDeploy currently rolls back only on a CPU alarm, and two recent defects that returned HTTP 5XX errors from the replacement tasks reached 100% of traffic.

Each deployment group must roll back automatically when the replacement target group's 5XX responses exceed 2% of its requests during the canary window, regardless of the original target group's traffic volume. Application code can't change this quarter. Every service already runs an nginx sidecar whose access logs are shipped to CloudWatch Logs; most services are written in Kotlin.

Leadership has asked for the option that is cheapest to run across all 70 services.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Create a CloudWatch Logs metric filter on each service's nginx access logs that counts 5XX and total responses with a dimension for the task set's target group, and alarm on the ratio of the two custom metrics for each target group in the deployment group.
- B. Add a sidecar that tails the nginx access logs and emits CloudWatch embedded metric format records with per-target-group request and 5XX counts, and alarm on a metric math ratio of those custom metrics for each target group in the deployment group.
- C. For each target group, create an alarm on a metric math expression that divides the ALB's `HTTPCode_Target_5XX_Count` by `RequestCount` for that target group over 1-minute periods and three evaluation periods, and add both alarms of each pair to the service's deployment group rollback configuration.
- D. Enable ALB access logs to Amazon S3 and run a scheduled Lambda function every minute that queries the newest log objects with Amazon Athena, publishes a custom 5XX-ratio metric per target group, and drives an alarm attached to each deployment group.

---

## Question 7 (Domain 5: Incident and Event Response)

A fintech team's CodePipeline (V2) pipeline has an `IntegrationTest` stage with a CodeBuild action that calls a partner's sandbox API. About 5% of runs fail because the sandbox times out, and simply rerunning the stage almost always succeeds. Engineers currently watch a Slack channel, open the console, and retry the stage by hand, which delays overnight releases by up to an hour.

The team already uses Amazon Q Developer in chat applications with that Slack workspace for other AWS notifications. The pipeline runs in `QUEUED` execution mode, and the CodeBuild project uses a custom Ubuntu image.

A failed `IntegrationTest` stage must be retried automatically, and the team wants a Slack message, with a link to the execution, only when the pipeline execution ultimately fails.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create an EventBridge rule for `IntegrationTest` stage failures that invokes a Lambda function. The function calls `RetryStageExecution` with `FAILED_ACTIONS` once per execution, tracks attempts in DynamoDB, and posts to a Slack incoming webhook if the retried stage fails again.
- B. Wrap the test command in a buildspec loop that reruns the suite up to three times, and route CodeBuild build-failure state changes through EventBridge to an Amazon SNS topic that forwards email to the Slack channel's email address.
- C. Replace the CodeBuild action with a Step Functions invoke action whose state machine starts the CodeBuild project, retries it once on failure, and returns the result to CodePipeline, with final failures posted to Slack by a Lambda webhook function.
- D. Configure the `IntegrationTest` stage's failure condition to retry failed actions automatically in the same execution, and create a notification rule on the pipeline for failed pipeline executions that targets the existing Amazon Q Developer in chat applications Slack channel.

---

## Question 8 (Domain 1: SDLC Automation)

A company has 14 product teams, each in its own AWS account. Every team runs a self-managed Nexus repository on an EC2 instance that proxies npmjs.com and PyPI for its CodeBuild projects. An analysis shows that 85% of cached packages are identical across the teams, and the 14 instances and their EBS volumes are now the platform's third-largest cost line.

The company is moving to AWS CodeArtifact. CodeBuild in all 14 accounts must resolve public npm and PyPI packages through CodeArtifact; each team must publish private packages to a repository that only that team can write to; and access must be limited to the organization. All builds run in us-east-1, and two teams use Yarn Berry instead of npm.

Finance has asked for the design with the lowest ongoing cost.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Create one domain in a shared services account whose domain policy allows the organization via `aws:PrincipalOrgID`, npm and PyPI repositories with external connections, and per-team repositories that use them as upstreams under team-scoped policies.
- B. Create a domain in each team's account containing an npm and a PyPI repository with external connections plus a private repository for the team's packages, sharing nothing across accounts so that each team's costs remain isolated and attributable.
- C. Create one domain per language ecosystem in the shared services account, each with an external-connection repository and per-team repositories, and copy private packages between the two domains nightly with a CodeBuild job for polyglot projects.
- D. Build the shared domain design, but keep each team's Nexus instance as a pull-through cache in front of the team's CodeArtifact repository to reduce CodeArtifact request charges, pointing builds at Nexus.

---

## Question 9 (Domain 6: Security and Compliance)

A healthcare company builds 120 container images with CodeBuild and pushes them to Amazon ECR repositories in a shared services account, from which production Amazon EKS clusters pull. Amazon Inspector is activated with ECR enhanced scanning (continuous scanning) on the production repositories, and that setting stays in place in every design under consideration.

A compliance finding requires that no image with CRITICAL or HIGH vulnerabilities is ever pushed to a production repository. Today, vulnerable images land in the repositories and Inspector findings appear minutes later — sometimes after the image has been deployed. The builds use multi-stage Dockerfiles with distroless base images.

The security team wants the gate enforced inside the build, with as little new infrastructure and custom code as possible.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Push each image to a staging repository with enhanced scanning, and add an EventBridge rule on Inspector's initial-scan completion event that invokes a Lambda function to check the image's findings and copy compliant images into the production repository.
- B. Install an open-source scanner in the CodeBuild image, mirror its vulnerability database to Amazon S3 with a scheduled job so builds avoid the internet, and fail the build before `docker push` when the scanner reports CRITICAL or HIGH vulnerabilities.
- C. Generate an SBOM in CodeBuild and upload it to Amazon S3, where an S3 event invokes a Lambda function that submits it to the Amazon Inspector scan API and records a verdict that a later build phase polls before running `docker push`.
- D. In the buildspec, run the Amazon Inspector SBOM generator against the built image, submit the SBOM to the Amazon Inspector scan API, and fail the build before `docker push` if the response contains CRITICAL or HIGH findings for any package.

---

## Question 10 (Domain 2: Configuration Management and IaC)

An online retailer keeps checkout feature flags and tuning parameters for 120 Lambda functions in one JSON document in Amazon S3. Functions read it during initialization, so changes take effect only as execution environments recycle. Last month a bad discount-rule flag broke checkout: the existing CloudWatch alarm on checkout errors fired within 90 seconds, but the bad configuration stayed active for 50 minutes while engineers redeployed functions to force a refresh.

The functions are written in Python and Node.js and run behind API Gateway. Configuration changes are made by product managers several times a day.

The company requires that if the existing alarm fires after a configuration change, the previous configuration is restored on every function automatically — with no code redeployment and no human in the loop.

**Which solution meets these requirements with the FASTEST rollback?**

- A. Move the document into an AWS AppConfig feature flag profile whose deployments list the existing alarm as a monitor, and add the AppConfig Agent Lambda extension to each function with a short poll interval so automatic rollbacks propagate quickly.
- B. Move the values into Lambda environment variables managed by CloudFormation, apply changes through stack updates, and register the existing alarm as a stack rollback trigger so a breach makes CloudFormation restore the previous variables on all 120 functions.
- C. Store the document as a Systems Manager parameter that functions read through the AWS Parameters and Secrets Lambda Extension with a 300-second cache, and add an EventBridge rule on the alarm that invokes a Lambda function to restore the previous parameter version.
- D. Store each flag as a DynamoDB item that functions cache for five minutes, and apply changes from a Step Functions workflow that watches the existing alarm and writes the previous item values back if the alarm enters ALARM after a change.

---

## Question 11 (Domain 3: Resilient Cloud Solutions)

A travel company's booking API runs on six Java 21 Lambda functions behind API Gateway, packaged as .zip archives with default ephemeral storage and no Amazon EFS. Framework initialization makes cold starts take 6–9 seconds, so the p99 latency SLO of 1.5 seconds is violated whenever partner promotions cause traffic bursts, which arrive without warning at any hour.

Steady-state concurrency is about 40; bursts reach 900 concurrent executions within a minute and last 10–20 minutes. API Gateway invokes a `prod` alias. In testing, environments restored from a snapshot were ready in under 600 ms once the code primed its HTTP clients in a runtime hook.

The company wants to meet the SLO at the lowest cost.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Configure provisioned concurrency of 900 on the `prod` alias of each function so that enough initialized execution environments always exist for the largest burst, and leave on-demand concurrency available as overflow capacity.
- B. Enable Lambda SnapStart on published versions behind the `prod` alias, using the runtime hook to prime HTTP clients before the snapshot is taken, so new execution environments resume from the snapshot during bursts.
- C. Move the API to Amazon ECS on AWS Fargate behind an Application Load Balancer, running enough tasks continuously to serve 900 concurrent requests so that no request ever waits for application initialization during a burst.
- D. Enable SnapStart on the four functions that tolerate snapshot restores, and configure provisioned concurrency of 900 on the `prod` alias of the two most latency-critical functions so they never depend on snapshot restores.

---

## Question 12 (Domain 4: Monitoring and Logging)

A company runs 600 CodeBuild projects that together produce about 2 TB of build logs per month. All projects send logs to CloudWatch Logs with 400-day retention to satisfy an audit requirement. Engineers read logs only for failed builds, almost always within a day, and auditors occasionally request the complete log for a specific build ID.

An S3 bucket with Object Lock and lifecycle rules already stores other audit artifacts in the same account. The builds run in a VPC with an S3 gateway endpoint, and 40% of the projects use ARM compute.

Every build log must be retained for 400 days and be retrievable by build ID within minutes. The company wants the lowest cost.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Keep CloudWatch Logs delivery but switch every build log group to the Infrequent Access log class with 400-day retention, because engineers and auditors only need to retrieve occasional build logs by build ID.
- B. Keep CloudWatch Logs delivery in the Standard log class with 400-day retention, and add a saved CloudWatch Logs Insights query that auditors run with the build ID to retrieve a complete log on request.
- C. Disable CloudWatch Logs in each project's logs configuration and write build logs to the existing audit bucket under a build-ID prefix, with a lifecycle rule moving objects to S3 Glacier Instant Retrieval after 30 days.
- D. Keep CloudWatch Logs delivery with 7-day retention, and add a subscription filter on every log group that streams events to Amazon Data Firehose, which writes compressed objects to the audit bucket for 400-day retention.

---

## Question 13 (Domain 1: SDLC Automation)

A product team keeps 40 repositories in GitHub, connected to CodeBuild through AWS CodeConnections. Reviewers need a full preview environment — an ECS service, an Aurora Serverless v2 cluster, and an ALB — deployed from each repository's CloudFormation template as stack `pr-<number>` for every pull request. About 300 pull requests are opened per month; most stay open for two days, some for three weeks. Team policy allows each developer only one open pull request at a time.

Today each of the 35 developers owns a long-lived preview environment that runs around the clock and is redeployed by hand. Aurora is the largest cost line, and engineers use a mix of ARM and x86 laptops.

A preview environment must exist whenever its pull request is open and must reflect the latest commit. Leadership asks for the lowest cost.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Keep one long-lived preview environment per developer with Aurora Serverless v2 at its minimum capacity, and add a CodeBuild webhook on pull request created and updated events that redeploys the developer's stack with the latest commit.
- B. Create stack `pr-<number>` from a CodeBuild webhook on pull request created, updated, and reopened events, and run a weekly EventBridge Scheduler job that deletes every preview stack whose pull request is no longer open.
- C. Use one CodeBuild webhook filter group for pull request created, updated, and reopened events that deploys stack `pr-<number>`, and a second filter group for pull request merged and closed events that deletes it.
- D. Create stack `pr-<number>` from a CodeBuild webhook on pull request created, updated, and reopened events, and delete each stack 24 hours after a GitHub closed-event webhook delivered through API Gateway reaches a Lambda function.

---

## Question 14 (Domain 6: Security and Compliance)

A bank runs 200 on-premises Jenkins agents on autoscaled VMware virtual machines. Build jobs push images to Amazon ECR and artifacts to Amazon S3 using an IAM user's access keys stored in Jenkins credentials. An internal audit requires that all long-term AWS access keys be removed from build infrastructure within 30 days.

The bank's enterprise PKI already issues every agent VM an X.509 client certificate through automatic enrollment when the VM is created, and renews it every 30 days. The Jenkins controller has no inbound internet access, and the agents run a mix of RHEL 9 and Windows Server 2022.

Builds must obtain short-lived credentials for a single IAM role automatically, without storing AWS secrets on the agents.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Configure the Jenkins controller as an OpenID Connect provider, publish its discovery document and signing keys on a public Amazon S3 website, register an IAM OIDC identity provider for it, and have jobs exchange signed job tokens through `AssumeRoleWithWebIdentity`.
- B. Create an IAM Roles Anywhere trust anchor from the enterprise CA bundle and a profile mapped to the build role, and configure each agent's AWS CLI to use the AWS signing helper as a `credential_process` with the VM's existing certificate.
- C. Create a new AWS Private CA hierarchy, issue each agent a client certificate at VM creation through a provisioning script, and configure IAM Roles Anywhere with the private CA as the trust anchor and the AWS signing helper as a `credential_process`.
- D. Build an internal credential-vending API on Amazon API Gateway with mutual TLS that trusts the enterprise CA, backed by a Lambda function that calls `AssumeRole` for the build role and returns temporary credentials that jobs request when they start.

---

## Question 15 (Domain 5: Incident and Event Response)

An IoT platform processes telemetry from an Amazon Kinesis data stream with 40 shards through a Lambda function and an event source mapping with a batch size of 500. Malformed records from a few devices occasionally make the function throw. Each time, the batch is retried until its records expire at the end of the 7-day retention period, and the shard's iterator age climbs for hours while valid records behind it wait.

Engineers must keep processing valid records with minimal delay, quarantine only the failing records, and keep enough information to inspect and replay them later. Processing is idempotent. The function is written in Rust and uses 1,024 MB of memory.

The team wants the simplest solution to run.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Wrap the processing loop in error handling that writes failed records to an S3 bucket, stores the last good sequence number for each shard in DynamoDB, and returns success so that the event source mapping advances past the failed batch.
- B. Replace the Lambda consumer with a Kinesis Client Library application on Amazon ECS that checkpoints after each record, sends records that fail three times to an SQS queue, and scales its task count with the shard count.
- C. Set the event source mapping's maximum retry attempts to zero with an SQS on-failure destination, and add a second Lambda function that reads each failed batch's sequence range from the stream and reprocesses its records one at a time.
- D. Enable bisect batch on function error on the event source mapping, limit the maximum record age to one hour, and add an SQS on-failure destination that receives the shard ID and sequence-number range of each batch that still fails.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | A | Domain 1: SDLC Automation | FASTEST rollback |
| 2 | B | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 3 | B | Domain 6: Security and Compliance | LEAST privilege |
| 4 | A | Domain 3: Resilient Cloud Solutions | FASTEST rollback |
| 5 | C | Domain 1: SDLC Automation | LEAST operational overhead |
| 6 | C | Domain 4: Monitoring and Logging | MOST cost-effective |
| 7 | D | Domain 5: Incident and Event Response | LEAST operational overhead |
| 8 | A | Domain 1: SDLC Automation | MOST cost-effective |
| 9 | D | Domain 6: Security and Compliance | LEAST operational overhead |
| 10 | A | Domain 2: Configuration Management and IaC | FASTEST rollback |
| 11 | B | Domain 3: Resilient Cloud Solutions | MOST cost-effective |
| 12 | C | Domain 4: Monitoring and Logging | MOST cost-effective |
| 13 | C | Domain 1: SDLC Automation | MOST cost-effective |
| 14 | B | Domain 6: Security and Compliance | LEAST operational overhead |
| 15 | D | Domain 5: Incident and Event Response | LEAST operational overhead |

---

### Question 1

**Answer:** A  
**Constraint:** FASTEST rollback · **Services:** AWS CodePipeline, AWS CodeDeploy, AWS Lambda

**Socratic walkthrough**
1. *All four options revert automatically — so what actually differs?* — What each must do to put the old code back in front of users.
2. *Where does the previous code live at the moment the alarm fires?* — With alias-based deployments, the previous Lambda version is still published and still serving most traffic. B, C, and D must redeploy or rebuild it.
3. *Which rollback is a pointer change rather than a provisioning operation?* — Only the alias weight change in A.

**Why A is correct:** SAM's `DeploymentPreference` makes CodeDeploy shift the `live` alias between two immutable published versions. When a listed alarm fires during the deployment, CodeDeploy stops and routes 100% of the alias back to the original version — a single routing-configuration update that takes effect in seconds. A release where every invocation fails trips the alarm inside the 5-minute canary window.

**Why the others fail the constraint:**
- **B.** Stage rollback is automatic, but it re-runs the deploy action with the previous execution's artifacts — a full stack update of 14 functions that takes minutes.
- **C.** Rollback triggers work, but CloudFormation must roll back every changed resource (code, versions, alias) through a stack rollback, which is again minutes of provisioning.
- **D.** A revision-override execution rebuilds and redeploys the old commit through every stage — the slowest path, plus custom code.

**Exam reflex:** For Lambda, the fastest rollback is an alias weight change; anything that redeploys or reprovisions is slower.

---

### Question 2

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** AWS CloudFormation Hooks, Amazon S3, AWS CodePipeline

**Socratic walkthrough**
1. *Why can't the existing PR check solve this?* — Console updates bypass pipelines; only a hook evaluates every stack operation before provisioning.
2. *Which hook types can run the existing Guard files unchanged?* — A managed Guard hook natively; a Lambda or custom hook only if you write and maintain code around cfn-guard.
3. *What keeps 25 accounts — and future ones — in sync with no per-account work?* — A service-managed StackSet targeting the OU, reading rules from one central bucket.

**Why B is correct:** A Guard hook evaluates Guard rules stored in S3 against resource configurations before provisioning and fails the operation in FAIL mode — no code. The service-managed StackSet automatically deploys it to new OU accounts, and every account reads the same rules location, so rule changes apply without redeployment.

**Why the others fail the constraint:**
- **A.** Works, but the team now owns a Lambda function: packaging and patching the cfn-guard library, handling timeouts, and maintaining hook response logic.
- **C.** Works, but a custom hook type means a schema, handler code, testing, and publishing a new registered version across 25 accounts on every rule change.
- **D.** Works, but console activation is manual per account (including future accounts), and the nightly copy job and 25 duplicated buckets are extra moving parts that also delay rule updates.

**Exam reflex:** If the rules already exist in Guard, the managed Guard hook beats any code-based hook.

---

### Question 3

**Answer:** B  
**Constraint:** LEAST privilege · **Services:** AWS CodePipeline, AWS KMS, AWS CloudFormation

**Socratic walkthrough**
1. *Which identity actually reads the artifact and decrypts it?* — The cross-account action role in production, via S3 (so KMS sees the call arriving through S3).
2. *Does CloudFormation's service role need artifact or key access?* — No; it only needs permissions for the stack's resources, and the action role must be able to pass it.
3. *Which grant can be narrowed by principal, resource, and path at the same time?* — Name the one role, add `kms:ViaService`, and restrict S3 to `orders-pipeline/*`.

**Why B is correct:** It grants exactly the decrypt path the action role uses (KMS calls made through S3), object reads limited to this pipeline's prefix, and `iam:PassRole` for one specific role whose own permissions cover only the stack's resources. Nothing else in production, and nothing outside the orders artifacts, gains access.

**Why the others fail the constraint:**
- **A.** The tutorial pattern: the account root principal delegates access to any production identity an administrator chooses, across every team's artifacts in the bucket.
- **C.** Correct resource scoping, but `AdministratorAccess` on a combined role grants every action in the account to whoever can trigger the pipeline.
- **D.** Correct actions and prefix, but `aws:PrincipalOrgID` opens the orders artifacts to every principal in every account in the organization.

**Exam reflex:** Least privilege = specific role principal + specific actions + specific resource path + a service-path condition (`kms:ViaService`).

---

### Question 4

**Answer:** A  
**Constraint:** FASTEST rollback · **Services:** Amazon EC2 Auto Scaling, Elastic Load Balancing, AWS CodeDeploy

**Socratic walkthrough**
1. *What is the most expensive step in this application's recovery?* — Booting and warming 24 instances: about 8 minutes each.
2. *Which designs still have the old release running 40 minutes after a release?* — Only blue/green with a 2-hour wait before original instances are terminated.
3. *What does rollback then mean?* — Rerouting the ALB to instances that are already warm — no boot, no install.

**Why A is correct:** In an EC2 blue/green deployment, CodeDeploy provisions a replacement fleet, reroutes the ALB, and keeps the original instances running for the configured wait time. Stopping the deployment with rollback during that window reroutes traffic back to the original, fully warmed instances, typically within the time it takes them to pass ALB health checks.

**Why the others fail the constraint:**
- **B.** An instance refresh's automatic rollback only covers failures during the refresh; after completion you must start another refresh, which relaunches and re-warms instances.
- **C.** `AutoScalingReplacingUpdate` deletes the old group after a successful update, so rolling back creates yet another group and pays the 8-minute warm-up again.
- **D.** No instances boot, but the previous revision must be reinstalled and restarted across two batches, and restarted processes must rewarm the cache before passing health checks.

**Exam reflex:** The fastest rollback keeps the previous fleet running and moves traffic, not code.

---

### Question 5

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** Amazon ECS, Elastic Load Balancing, AWS CloudFormation

**Socratic walkthrough**
1. *Which requirements are pure blue/green features?* — Test listener, pre-shift validation hook, all-at-once shift, bake time, and alarm rollback.
2. *Does Amazon ECS still need CodeDeploy to do this?* — No. Since 2025, ECS has a built-in blue/green deployment strategy with bake time, lifecycle hooks, and alarm-based rollback.
3. *Which option adds no new service, artifact, or deploy action?* — Native ECS blue/green inside the existing template and CloudFormation action.

**Why C is correct:** ECS built-in blue/green deployments are configured on the service itself: the alternate target group and listener rules, a bake time, Lambda lifecycle hooks (including one that runs while test traffic is routed), and deployment alarms with rollback. The existing CloudFormation action keeps working, and no CodeDeploy resources are introduced.

**Why the others fail the constraint:**
- **A.** Works, but adds a CodeDeploy application, deployment group, AppSpec generation, and a different pipeline action — exactly the dependency the company wants to avoid.
- **B.** Works, but the CloudFormation blue/green hook brings CodeDeploy back in through a transform and comes with stack-update restrictions to manage.
- **D.** Works, but two services and a custom state machine for traffic shifting, testing, and rollback are bespoke code to build and maintain.

**Exam reflex:** For new ECS services, check native ECS blue/green before reaching for CodeDeploy.

---

### Question 6

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** Amazon CloudWatch, Elastic Load Balancing, AWS CodeDeploy

**Socratic walkthrough**
1. *Does the signal already exist somewhere for free?* — Yes. ALBs publish `RequestCount` and `HTTPCode_Target_5XX_Count` per target group at no charge.
2. *Why alarm on both target groups?* — The groups swap roles each deployment, so whichever is the replacement is covered.
3. *What does every other option add to the bill?* — Custom metrics, extra log ingestion, or per-minute Athena scans plus Lambda.

**Why C is correct:** A metric math alarm per target group divides 5XX count by request count for that group, which gives the replacement task set's error ratio independent of the original group's volume. The cost is just two standard alarms per service, using free ALB metrics; CodeDeploy rolls back when either listed alarm fires.

**Why the others fail the constraint:**
- **A.** Works, but each metric filter creates custom metrics billed per metric per month — across 70 services × 2 target groups × 2 metrics.
- **B.** Works, but EMF records are additional log ingestion, and the extracted values are custom metrics as well.
- **D.** Works, but runs 1,440 Athena queries a day (billed per data scanned) plus Lambda invocations and custom metrics — the most expensive option.

**Exam reflex:** Before creating custom metrics, check what the load balancer already publishes per target group.

---

### Question 7

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS CodePipeline, Amazon EventBridge, Amazon Q Developer in chat applications

**Socratic walkthrough**
1. *Does CodePipeline V2 have a native way to react to a stage failure?* — Yes. A stage's failure condition can retry the failed actions automatically.
2. *Which event should trigger Slack: the stage failure or the execution failure?* — The execution failure, which only happens if the retry also fails.
3. *Is there already a managed path from pipeline events to Slack?* — Yes: notification rules targeting the existing Q Developer chat channel.

**Why D is correct:** The stage-level retry is configuration, not code, and preserves the execution in its queue order. A notification rule filtered on failed pipeline executions posts to the already-configured Slack channel with a link to the execution. Nothing new needs to be deployed or maintained.

**Why the others fail the constraint:**
- **A.** Works, but adds a Lambda function, a DynamoDB attempt tracker, webhook secrets, and EventBridge plumbing that the team owns.
- **B.** Works, but in-build retry loops mask flakiness and burn build minutes, and email-to-Slack forwarding is an extra integration to maintain.
- **C.** Works, but replaces a simple CodeBuild action with a state machine plus a Lambda webhook — more components to deploy and debug.

**Exam reflex:** For V2 pipelines, use stage conditions (retry/rollback) and notification rules before writing glue code.

---

### Question 8

**Answer:** A  
**Constraint:** MOST cost-effective · **Services:** AWS CodeArtifact, AWS CodeBuild, AWS Organizations

**Socratic walkthrough**
1. *How does CodeArtifact bill storage when the same package is in many repositories?* — Assets are stored once per domain, no matter how many repositories in that domain contain them.
2. *So what drives storage cost here?* — The number of domains holding the 85% overlap.
3. *Is any extra infrastructure needed for performance?* — No; CodeArtifact is a managed, regional service.

**Why A is correct:** A single domain stores each shared package once. The npm and PyPI repositories with external connections fetch public packages once for everyone, and team repositories use them as upstreams. Repository policies restrict publishing per team, and the domain policy restricts access to the organization.

**Why the others fail the constraint:**
- **B.** Works, but 14 domains store the same 85% of packages 14 times.
- **C.** Works, but two domains duplicate every copied private package, and the nightly copy job adds build minutes.
- **D.** Works, but keeps 14 EC2 instances and EBS volumes running to save on CodeArtifact request charges that are small by comparison.

**Exam reflex:** CodeArtifact deduplicates within a domain — consolidate domains, not repositories, to cut storage.

---

### Question 9

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** Amazon ECR, Amazon Inspector, AWS CodeBuild

**Socratic walkthrough**
1. *Can ECR enhanced scanning stop a push?* — No; it scans images after they are pushed, so the gate has to run before `docker push`.
2. *Does Inspector offer scanning without pushing to ECR?* — Yes: the Inspector SBOM generator plus the Inspector scan API, designed for CI/CD.
3. *Which option keeps everything inside the build, with no new services?* — Calling the scan API synchronously from the buildspec.

**Why D is correct:** The build generates an SBOM locally, sends it to the Inspector scan API, and gets vulnerability results back in the same step, so it can fail before the push. Inspector maintains the vulnerability intelligence, and continuous scanning stays on the production repositories.

**Why the others fail the constraint:**
- **A.** Works, but needs a staging repository, an EventBridge rule, a promotion Lambda, and image-copy logic.
- **B.** Works, but the team must patch the scanner and keep a mirrored vulnerability database current — a scheduled job and a second source of truth.
- **C.** Works, but splits a synchronous call into S3, an S3 event, Lambda, a verdict store, and polling — custom glue around the same API that D calls directly.

**Exam reflex:** To block before push, use Inspector's CI/CD scan API in the build; enhanced scanning covers images after push.

---

### Question 10

**Answer:** A  
**Constraint:** FASTEST rollback · **Services:** AWS AppConfig, AWS Lambda, Amazon CloudWatch

**Socratic walkthrough**
1. *How long does each design take to notice a revert?* — It depends on how each function re-reads configuration: poll interval, cache TTL, or a new execution environment.
2. *Which design rolls back as a side effect of the deployment itself, rather than via extra automation?* — AppConfig: its deployment monitors trigger an automatic rollback.
3. *What then bounds propagation?* — The AppConfig agent extension's poll interval, which you can set to seconds.

**Why A is correct:** An AppConfig deployment monitors the listed alarm and automatically rolls back to the previous configuration version when it fires. Each function's AppConfig Agent extension polls on a short interval and serves the reverted configuration on its next poll. No redeploy, no glue code, no cold starts.

**Why the others fail the constraint:**
- **B.** Works, but changing environment variables on 120 functions is a stack rollback, and new variables force new execution environments (cold starts).
- **C.** Works, but the revert waits on an EventBridge-to-Lambda hop, and functions keep serving the bad value until their 300-second cache expires.
- **D.** Works, but the revert depends on a custom workflow, and the five-minute cache delays propagation.

**Exam reflex:** For runtime configuration with alarm-driven rollback, AppConfig deployment monitors plus the agent extension are the fastest native path.

---

### Question 11

**Answer:** B  
**Constraint:** MOST cost-effective · **Services:** AWS Lambda, Amazon API Gateway, Amazon CloudWatch

**Socratic walkthrough**
1. *What causes the SLO breach?* — Initialization time on new execution environments during bursts.
2. *Which options remove initialization without paying for idle capacity?* — SnapStart restores from a snapshot on demand; provisioned concurrency and Fargate pay for capacity around the clock.
3. *Do the restrictions rule SnapStart out?* — No. The functions use .zip packages, default ephemeral storage, no EFS, and an alias of published versions.

**Why B is correct:** SnapStart snapshots the initialized execution environment of a published version, and new environments resume from it, which testing shows takes under 600 ms. For Java runtimes, SnapStart carries no additional charge, so bursts are served at normal on-demand pricing.

**Why the others fail the constraint:**
- **A.** Meets the SLO, but pays for 900 provisioned environments per function 24×7 to serve a steady state of about 40.
- **C.** Meets the SLO, but runs peak-sized Fargate capacity continuously, plus an ALB.
- **D.** Meets the SLO, but still pays 24×7 provisioned concurrency for two functions that SnapStart already covers.

**Exam reflex:** Java cold starts plus unpredictable bursts point to SnapStart; provisioned concurrency pays for idle capacity.

---

### Question 12

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** AWS CodeBuild, Amazon CloudWatch Logs, Amazon S3

**Socratic walkthrough**
1. *What is the dominant cost of 2 TB/month in CloudWatch Logs?* — Ingestion, billed per GB, plus storage for 400 days.
2. *Can CodeBuild skip CloudWatch Logs entirely?* — Yes. A project's logs configuration can disable CloudWatch Logs and write directly to S3.
3. *Does S3 meet the retrieval requirement?* — Yes. Objects are keyed by build ID, and Glacier Instant Retrieval returns them in milliseconds.

**Why C is correct:** Writing logs straight to the existing audit bucket removes all CloudWatch Logs ingestion and storage charges. A lifecycle rule moves objects older than 30 days to a cheaper storage class that still allows immediate retrieval by build ID.

**Why the others fail the constraint:**
- **A.** Works, and Infrequent Access lowers the ingestion rate, but you still pay ingestion and storage for 2 TB a month — far above S3 storage costs.
- **B.** Works, but it is the current, most expensive pattern; Logs Insights queries add per-GB-scanned charges on top.
- **D.** Works, but pays full CloudWatch Logs ingestion and then Firehose processing just to reach S3.

**Exam reflex:** If logs are rarely read, send them directly to S3; every hop through CloudWatch Logs is billed ingestion.

---

### Question 13

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** AWS CodeBuild, AWS CodeConnections, AWS CloudFormation

**Socratic walkthrough**
1. *What drives cost?* — Hours that preview stacks, especially Aurora, exist without an open pull request.
2. *Which event marks the moment a pull request stops needing its environment?* — Merged or closed.
3. *Can CodeBuild react to those events natively?* — Yes. Webhook filter groups support pull request merged and closed events.

**Why C is correct:** Two filter groups create or update `pr-<number>` while the pull request is open and delete it the moment the pull request is merged or closed. Environments exist only while needed, with no scheduler, API, or extra function.

**Why the others fail the constraint:**
- **A.** Works under the one-open-PR policy, but 35 environments run continuously, including evenings, weekends, and days with no open pull request.
- **B.** Works, but stacks linger up to a week after their pull requests close.
- **D.** Works, but each stack lives 24 extra hours, and API Gateway and Lambda are added.

**Exam reflex:** Tie ephemeral environment lifetime to source-control events, not schedules.

---

### Question 14

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** IAM Roles Anywhere, an existing enterprise CA (or AWS Private CA), Amazon ECR

**Socratic walkthrough**
1. *What identity does every agent already have?* — An auto-enrolled, auto-renewed X.509 certificate from the enterprise PKI.
2. *Which AWS service exchanges an X.509 certificate for temporary role credentials?* — IAM Roles Anywhere, which accepts an external CA as a trust anchor.
3. *What is left to build?* — Only the trust anchor, the profile, and a `credential_process` line on each agent.

**Why B is correct:** Roles Anywhere validates the existing certificates against the enterprise CA bundle and issues short-lived credentials for the mapped role. Certificate lifecycle stays with the PKI that already exists, and the signing helper supports both Linux and Windows.

**Why the others fail the constraint:**
- **A.** Works, but the team must run Jenkins as an OIDC issuer and publish and rotate its signing keys on a public site that AWS STS can reach.
- **C.** Works, but duplicates the existing PKI with a new private CA hierarchy and provisioning scripts — more to run and pay for.
- **D.** Works, but the team builds and secures a custom credential broker.

**Exam reflex:** Workloads outside AWS that already have X.509 certificates should use IAM Roles Anywhere with the existing CA.

---

### Question 15

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** Amazon Kinesis Data Streams, AWS Lambda event source mappings, Amazon SQS

**Socratic walkthrough**
1. *Why does one bad record stall a shard today?* — Lambda retries the whole batch until the records expire, and processing is ordered per shard.
2. *Which built-in settings stop the stall and narrow it to the bad records?* — Bisect on error splits batches to isolate failures, and the record-age limit bounds retries.
3. *How do you keep replay information without code?* — An on-failure destination that records the shard and sequence-number range.

**Why D is correct:** Bisecting lets valid halves of a batch succeed while the failing portion shrinks. The age limit stops endless retries, and the SQS destination captures metadata so engineers can re-read those records from the stream. All of this is event source mapping configuration.

**Why the others fail the constraint:**
- **A.** Works, but the team owns error handling, an S3 quarantine format, and a DynamoDB checkpoint table.
- **B.** Works, but replaces a serverless consumer with a KCL application on ECS that the team must scale, patch, and checkpoint.
- **C.** Works, but sends entire 500-record batches to the destination and needs a second function to re-read and replay them.

**Exam reflex:** For stream poison pills, configure bisect, retry/age limits, and an on-failure destination before writing code.

---

## Scoring & Performance Breakdown

| Score | Band | Next step |
|---|---|---|
| **14–15 / 15** | Exam ready | Proceed to Practice Exam 2. |
| **12–13 / 15** | Passing | For each miss, re-read the distractor breakdown and note which constraint signal you overlooked. |
| **≤ 11 / 15** | Needs remediation | Use the domain column in the key table to pick study guides; retake this exam in a week. |
