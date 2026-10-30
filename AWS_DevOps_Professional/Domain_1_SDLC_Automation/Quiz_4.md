# Domain 1 (SDLC Automation) — Quiz 4: Testing & Canary Deployments

**Topic:** Automated Pipeline Testing, Canary/Linear Traffic Shifting, Feature Flags & Rollback Gates  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Predefined Traffic Shifting Configurations for Risk-Mitigated Releases

An engineering director establishes a strict release policy for a high-volume payment processing Lambda function: 10% of live production traffic must be routed to the new function version, traffic must remain pegged at exactly 10% for 30 minutes while synthetic alarms evaluate p99 latency and error rates, and if no alarms breach, 100% of traffic must transition to the new version immediately.

**Which predefined AWS CodeDeploy deployment configuration matches this release profile?**
- A. `CodeDeployDefault.LambdaCanary10Percent30Minutes`: Shifts 10% of traffic to the new version in the first increment, pauses for 30 minutes, and shifts the remaining 90% in the second increment.
- B. `CodeDeployDefault.LambdaLinear10PercentEvery10Minutes`: Shifts 10% of traffic immediately, followed by incremental 10% shifts every 10 minutes until 100% is reached after 90 minutes.
- C. `CodeDeployDefault.LambdaAllAtOnce`: Shifts 100% of traffic immediately, while evaluating CloudWatch alarms for 30 minutes before allowing subsequent pipeline stages to proceed.
- D. `CodeDeployDefault.ECSLinear10PercentEvery30Minutes`: Configured with an ALB weighted target group shifting traffic in 10% linear increments every 30 minutes over a 5-hour duration.

---

### Question 2: CloudWatch Alarm Rollback Mechanics During ECS Blue/Green Deployments

An enterprise deploys a containerized microservice to Amazon ECS using AWS CodeDeploy (`ECSBlueGreen`). The deployment group is configured with an automated rollback trigger linked to a CloudWatch Alarm monitoring ALB `HTTPCode_Target_5XX_Count` (`Period: 60s`, `EvaluationPeriods: 1`). During the traffic shifting phase, while 30% of traffic is routed to the replacement task set, a database connection leak causes the target 5xx alarm to transition to `ALARM`.

**What sequence of actions does AWS CodeDeploy execute upon detecting the alarm state?**
- A. CodeDeploy immediately aborts forward traffic shifting, shifts 100% of production traffic back to the original task set, terminates the replacement tasks, and marks the deployment status as `Failed`.
- B. CodeDeploy pauses the deployment at 30% traffic for the configured termination wait time, prompting on-call engineers in the AWS Console to either approve or rollback the release.
- C. CodeDeploy continues shifting traffic to 100% to evaluate whether the 5xx errors resolve under full load, but prevents the original task set from terminating.
- D. CodeDeploy sends a deregistration command to the ALB target group and restarts the Amazon ECS service tasks in-place across all availability zones.

---

### Question 3: Post-Deployment End-to-End Browser Testing in CodePipeline

A continuous delivery pipeline in AWS CodePipeline deploys a web portal to an Amazon ECS staging cluster. Before triggering a manual approval action for production promotion, the pipeline must execute automated Cypress headless browser integration test suites against the staging ALB URL to verify login, shopping cart, and checkout user journeys.

**How should this automated test suite be integrated into the CodePipeline workflow?**
- A. Add an **AWS CodeBuild** test action immediately following the `Deploy-Staging` stage; CodeBuild executes the Cypress headless test runner in a container, and the stage succeeds only if the test process exits with return code `0`.
- B. Configure the CodeDeploy `ValidateService` lifecycle hook script on the ECS container to download the Cypress test suite and run browser simulations locally inside the application container.
- C. Configure an Amazon EventBridge rule that detects the CodeDeploy deployment completion event, invoking an AWS Step Functions state machine that updates pipeline variables.
- D. Configure an Amazon S3 event notification on the artifact bucket that triggers an AWS Lambda function to execute browser tests over an SSH connection to the staging hosts.

---

### Question 4: Synthetics Canaries as Automated Continuous Pipeline Deployment Gates

A financial platform runs continuous canary releases across Amazon ECS and AWS Lambda. To protect against edge-case functional bugs that do not generate 5xx server errors (such as broken JavaScript forms or third-party iframe failures), the team requires continuous real-user journey verification that executes every minute and aborts deployments if transaction success drops below 99%.

**Which architecture implements this continuous validation gate?**
- A. Deploy an **Amazon CloudWatch Synthetics Canary** executing Node.js Puppeteer scripts every 60 seconds against critical user journeys, create a CloudWatch Alarm on `SuccessPercent < 99`, and add the alarm to the CodeDeploy Deployment Group **Deployment Alarms**.
- B. Configure an AWS CodeBuild project to loop executing `curl` commands against the API endpoint for 60 minutes, passing the build status back to CodePipeline via `PutJobSuccessResult`.
- C. Create an Amazon Route 53 health check that queries the root domain HTTP status, and configure Route 53 to fail over to a static maintenance page hosted on Amazon S3.
- D. Write an AWS Systems Manager Automation runbook that executes every minute via EventBridge Scheduler to query CloudWatch Logs Insights for frontend error strings.

---

### Question 5: Two-Phase Database Migrations in Concurrent Blue/Green Deployments

An engineering team is releasing an application update that modifies a relational database schema (adding a new column and restructuring user profile queries). During the CodeDeploy blue/green deployment, both the original application version and the replacement version run concurrently and serve live customer traffic simultaneously for 45 minutes.

**How must the database migration and application release be structured to prevent production errors?**
- A. Follow the **Expand-Contract (Two-Phase) pattern**: deploy an additive, backward-compatible migration *first* (adding nullable columns or defaults) so both old and new code versions function concurrently, and execute the contract phase (dropping legacy columns) only after 100% traffic cutover.
- B. Execute the schema migration script during the CodeDeploy `BeforeInstall` lifecycle hook, dropping outdated tables and columns before the replacement application containers launch.
- C. Place the database into read-only maintenance mode using Amazon RDS Multi-AZ failover, execute the migration on the read replica, and promote the replica to primary writer.
- D. Deploy an Amazon ElastiCache Redis caching layer in front of the database to intercept and convert mismatched SQL queries originating from the legacy application version.

---

### Question 6: CodeDeploy Failure Handling & Instance Health Churn Prevention

A CodeDeploy in-place deployment to an Auto Scaling Group of 12 instances experiences an unhandled exception during the `ValidateService` hook on instance 1. CodeDeploy halts the deployment. An engineer applies an emergency hotfix to instance 1's local files and manually retries the deployment, but instance 2 immediately fails with the same error.

**How does CodeDeploy prevent an unhealthy application revision from continually churning instances across an Auto Scaling Group?**
- A. CodeDeploy halts forward progression, marks the overall deployment as `Failed`, notifies Auto Scaling with an `ABANDON` action to terminate the failing instance, and leaves the remaining healthy instances untouched.
- B. CodeDeploy automatically scales the Auto Scaling Group capacity to zero instances to isolate the application until an engineer acknowledges the failure alert.
- C. CodeDeploy continues deploying to the remaining instances in the fleet, logging the failed instances into an Amazon SQS dead-letter queue for deferred inspection.
- D. Auto Scaling freezes all scaling activities for 24 hours and prevents any new instances from launching until the deployment group is recreated.

---

### Question 7: Long-Running Integration Testing with AWS Step Functions in CodePipeline

A healthcare platform requires multi-hour automated load, endurance, and compliance test suites to run against a dedicated performance test environment before code can be approved for production. The test execution takes approximately 4 hours, exceeding standard AWS Lambda execution timeouts (15 minutes).

**Which native CodePipeline action type should the DevOps Engineer configure to orchestrate this long-running test stage?**
- A. Configure an **AWS Step Functions Invoke Action** in CodePipeline: The pipeline invokes a Step Functions state machine that orchestrates distributed load test tasks, monitors completion, and returns the final `SUCCEEDED` or `FAILED` status back to CodePipeline asynchronously.
- B. Configure an AWS Lambda Invoke Action in CodePipeline, configuring the Lambda function execution timeout to 240 minutes in the function properties.
- C. Use a CodeDeploy `AfterInstall` lifecycle hook configured with a custom shell script that executes a 4-hour `sleep` loop while checking background process status.
- D. Add a CodePipeline Manual Approval action with an SNS topic that sends an email to the performance testing team, requiring an engineer to manually click approve after 4 hours.

---

### Question 8: Feature Flag Management and Progressive Rollouts with AWS AppConfig

An engineering team wants to decouple software deployments from feature releases. They need to deploy code dark (disabled), and subsequently roll out new capabilities to 10% of users, increasing linearly to 100% over 40 minutes. If user error rates spike at any point during rollout or during a 15-minute observation bake time, the feature must be disabled automatically.

**Which architecture satisfies these feature management requirements with the LEAST custom infrastructure?**
- A. Use **AWS AppConfig**: Create an Application, Environment, and Feature Flag Configuration Profile; define a **Deployment Strategy** with linear growth (25% every 10 minutes) and a 15-minute **Bake Time** monitored by a CloudWatch Alarm that triggers automated rollback on breach.
- B. Store feature flag boolean flags in an Amazon DynamoDB table, and write an AWS Lambda function that polls CloudWatch metrics every minute and overwrites the table values.
- C. Create Route 53 Weighted DNS records pointing to two separate API Gateway stages, updating record weights via an AWS Systems Manager Automation runbook.
- D. Use AWS CodeDeploy with a custom deployment configuration, specifying linear shifting on EC2 host tags without modifying application code.

---

### Question 9: Automated Chaos Resilience Validation Gates with AWS FIS in CI/CD

A platform engineering team requires that all staging environments undergo automated chaos experimentation before production deployment. The pipeline must trigger an AWS Fault Injection Service (FIS) experiment that injects 50% packet drop and terminates one random container task. If the staging application fails to maintain a sub-second response time during the chaos test, the CI/CD pipeline must fail immediately.

**How should this chaos testing gate be integrated into AWS CodePipeline?**
- A. Add an **AWS CodeBuild** action in the pipeline that executes `aws fis start-experiment --experiment-template-id <id>`, polls `aws fis get-experiment` until status is `completed`, asserts that CloudWatch synthetic alarms remained `OK`, and exits with code 1 if alarms breached.
- B. Configure AWS FIS to monitor the CodePipeline execution state directly, configuring the pipeline execution ARN as an automated FIS experiment trigger.
- C. Deploy an AWS CloudFormation Hook that evaluates the FIS experiment template YAML file during change set execution, blocking deployment if faults are detected.
- D. Configure the Amazon ECS service task definition with an embedded chaos agent container that runs packet disruption scripts continuously on boot.

---

## Answer Key & Detailed Explanations

### Question 1: Predefined Traffic Shifting Configurations for Risk-Mitigated Releases
**Answer:** A  
**Explanation:**  
`CodeDeployDefault.LambdaCanary10Percent30Minutes` is an AWS-managed deployment configuration designed specifically for canary deployments. It shifts exactly 10% of traffic to the new Lambda version in the first increment, pauses for 30 minutes while monitoring CloudWatch alarms, and shifts the remaining 90% in the second increment. Linear configurations (Option B) shift traffic incrementally across multiple steps (e.g. 10% every 10 minutes), while `AllAtOnce` (Option C) shifts 100% immediately.

---

### Question 2: CloudWatch Alarm Rollback Mechanics During ECS Blue/Green Deployments
**Answer:** A  
**Explanation:**  
When CodeDeploy is configured with CloudWatch Deployment Alarms, it continuously monitors alarm states during the traffic shifting lifecycle. If an alarm breaches (such as ALB target 5xx errors), CodeDeploy detects the state change, immediately aborts the deployment, reverts 100% of traffic back to the original task set, terminates the newly created replacement tasks, and marks the deployment as failed.

---

### Question 3: Post-Deployment End-to-End Browser Testing in CodePipeline
**Answer:** A  
**Explanation:**  
The AWS-recommended pattern for executing integration and browser-based test suites (such as Cypress, Selenium, or Playwright) in continuous delivery pipelines is an **AWS CodeBuild** test action. Running after the staging deployment stage, CodeBuild executes the headless test runner inside a container environment, points requests to the staging load balancer, and passes or fails the pipeline stage based on the process exit code (`0` for success, non-zero for failure).

---

### Question 4: Synthetics Canaries as Automated Continuous Pipeline Deployment Gates
**Answer:** A  
**Explanation:**  
**Amazon CloudWatch Synthetics Canaries** execute automated scripts (using Node.js Puppeteer or Python Selenium) that simulate real user clicks, form submissions, and authentication flows on a scheduled cadence (e.g. every minute). Linking an alarm on the canary's `SuccessPercent` metric (< 99%) directly to CodeDeploy's deployment alarms provides an automated safety net that catches client-side rendering bugs, CSS failures, and broken API workflows, automatically aborting deployments if user journeys fail.

---

### Question 5: Two-Phase Database Migrations in Concurrent Blue/Green Deployments
**Answer:** A  
**Explanation:**  
During blue/green and canary deployments, both old and new application versions run concurrently and process live production traffic during the shifting window. Applying breaking schema changes (such as dropping or renaming columns) breaks the running legacy version. The **Expand-Contract (Two-Phase) pattern** is strictly required: 1) Expand the schema in an additive, backward-compatible manner (new nullable columns or default values) so both application versions work concurrently; 2) Contract the schema (removing old columns) only in a subsequent release after 100% traffic cutover is complete and stable.

---

### Question 6: CodeDeploy Failure Handling & Instance Health Churn Prevention
**Answer:** A  
**Explanation:**  
When a CodeDeploy deployment fails on an Auto Scaling instance, CodeDeploy sends an `ABANDON` action to the underlying Auto Scaling lifecycle hook. Auto Scaling terminates the failed instance. CodeDeploy halts forward progression across the rest of the deployment group, preserving the remaining healthy instances running the previous revision, and records the deployment as failed to prevent churning through the entire fleet.

---

### Question 7: Long-Running Integration Testing with AWS Step Functions in CodePipeline
**Answer:** A  
**Explanation:**  
AWS Lambda actions in CodePipeline are constrained by Lambda's 15-minute maximum execution timeout. For long-running workflows (such as 4-hour performance, soak, or load tests), CodePipeline provides a native **AWS Step Functions Invoke Action**. CodePipeline starts the state machine execution asynchronously and pauses the pipeline stage until Step Functions reports execution completion (`SUCCEEDED` or `FAILED`).

---

### Question 8: Feature Flag Management and Progressive Rollouts with AWS AppConfig
**Answer:** A  
**Explanation:**  
**AWS AppConfig** provides native feature flag management and progressive rollout capabilities decoupled from code deployment. An AppConfig Deployment Strategy defines the growth type (e.g. Linear 25% every 10 minutes) and a **Bake Time** (e.g. 15 minutes) during which AppConfig monitors CloudWatch Alarms. If an alarm breaches during deployment or bake time, AppConfig automatically rolls back the feature flag to its previous state.

---

### Question 9: Automated Chaos Resilience Validation Gates with AWS FIS in CI/CD
**Answer:** A  
**Explanation:**  
To integrate chaos experimentation as a quality gate in CI/CD, an **AWS CodeBuild** action in CodePipeline can trigger an AWS Fault Injection Service (FIS) experiment via the AWS CLI or SDK (`aws fis start-experiment`). CodeBuild polls the experiment state until completion, verifies that CloudWatch synthetic monitoring alarms remained healthy during fault injection, and returns exit code 0 or 1, automatically gating the pipeline based on chaos resilience.
