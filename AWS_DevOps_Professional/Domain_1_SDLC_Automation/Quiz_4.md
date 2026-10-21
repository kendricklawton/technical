# Domain 1 (SDLC Automation) — Quiz 4: Testing & Canary Deployments

**Topic:** Testing Automation, Canary/Linear Deployments & Automated Rollback Triggers  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Linear vs. Canary Deployment Configurations

An e-commerce API must be updated with minimal blast radius. The engineering director mandates: "Route 10% of traffic to the new version. Keep it at 10% for 30 minutes while evaluating error rates. If clean, route all remaining traffic immediately."

**Which AWS CodeDeploy predefined deployment configuration matches this exact requirement?**
- A. `CodeDeployDefault.LambdaLinear10PercentEvery10Minutes`
- B. `CodeDeployDefault.LambdaCanary10Percent30Minutes`
- C. `CodeDeployDefault.LambdaAllAtOnce`
- D. `CodeDeployDefault.ECSLinear10PercentEvery30Minutes`

---

### Question 2: CloudWatch Alarm Rollback Criteria in CodeDeploy

A company configures automated rollbacks for an AWS CodeDeploy deployment group using a CloudWatch alarm tracking HTTP 5xx errors. The alarm is configured with `Period: 60 seconds` and `EvaluationPeriods: 1`.

**What happens if the alarm breaches during the traffic shifting phase of an ECS blue/green deployment?**
- A. CodeDeploy waits for the current batch of requests to finish and then prompts the operator for approval.
- B. CodeDeploy detects the alarm transition to `ALARM` state, halts traffic shifting immediately, re-routes all traffic back to the original task set, terminates the replacement tasks, and records the deployment as failed.
- C. CodeDeploy restarts the ECS cluster.
- D. CodeDeploy ignores the alarm until 100% of traffic has shifted.

---

### Question 3: Running Automated Post-Deployment Verification via CodePipeline

An enterprise pipeline needs to execute automated Cypress end-to-end browser tests against a newly deployed staging environment before the pipeline asks for manual approval for production.

**How should this testing step be integrated into AWS CodePipeline?**
- A. Run the browser tests inside the CodeDeploy `ApplicationStart` hook.
- B. Add an **AWS CodeBuild** test action in the pipeline immediately following the `Deploy-Staging` stage. The CodeBuild action runs Cypress headless tests in a container, asserting UI flows against the staging ALB URL. If tests exit with code 0, the pipeline advances to the manual approval action.
- C. Have developers run Cypress tests on their laptops and push a git tag.
- D. Configure an S3 lifecycle rule to trigger tests.

---

### Question 4: Synthetics Canaries as Continuous Pipeline Deployment Gates

A team wants to run synthetic transaction testing continuously during a 60-minute canary deployment. If the synthetic transaction failure rate drops below 99%, the deployment must abort.

**Which AWS integration achieves this?**
- A. Create an **Amazon CloudWatch Synthetics Canary** that executes synthetic user journeys every 1 minute. Create a CloudWatch Alarm on the canary's `SuccessPercent` metric (< 99%). Link this alarm to the CodeDeploy deployment group's alarm triggers.
- B. CloudWatch Synthetics cannot be linked to CodeDeploy.
- C. Write a cron job in EC2.
- D. Run `curl` commands in a loop in CodeBuild.

---

### Question 5: Safe Database Migrations in Blue/Green Pipelines

A team deploys an application update that requires an additive database schema change (adding a new nullable column). In a blue/green deployment where both old (blue) and new (green) application versions run concurrently during traffic shifting, how must database migrations be structured to prevent errors?

**What is the required database migration architectural discipline?**
- A. Drop old columns immediately before the new code is deployed.
- B. Ensure database changes are **backward-compatible** (Two-Phase / Expand-Contract pattern): deploy the additive schema migration *first* (expanding the schema with nullable columns or defaults) so both old and new code versions can function simultaneously, and contract old columns only after 100% traffic shift and release stabilization.
- C. Take the database offline during the deployment window.
- D. Blue/green deployments cannot be used with relational databases.

---

### Question 6: CodeDeploy Rollback Overrides on Consecutive Failures

A deployment to an Auto Scaling Group fails on instance 1 of 10. CodeDeploy halts the deployment. An engineer fixes the configuration on instance 1 and restarts the deployment. The deployment fails again on instance 2.

**How does CodeDeploy prevent an unhealthy revision from continually churning instances in an ASG?**
- A. CodeDeploy terminates the entire Auto Scaling Group.
- B. CodeDeploy halts the deployment, marks the deployment as `Failed`, notifies Auto Scaling with `ABANDON`, and terminates failed instances without updating healthy instances.
- C. CodeDeploy ignores instance failures and marks the deployment successful.
- D. The ASG will freeze all scaling operations for 24 hours.

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

### Question 1: Linear vs. Canary Deployment Configurations

**Answer:** B  
**Explanation:** 
- **Canary configurations** shift a single percentage of traffic in the first increment (e.g. 10%), wait for a specified duration (e.g. 30 minutes), and then shift the remaining 90% in a single final cutover (`Canary10Percent30Minutes`).
- **Linear configurations** shift traffic in equal increments over multiple intervals (e.g. 10% every 10 minutes for 100 minutes total).

---

### Question 2: CloudWatch Alarm Rollback Criteria in CodeDeploy

**Answer:** B  
**Explanation:** 
When CloudWatch alarms are linked to a CodeDeploy Deployment Group under "Rollback Configuration", CodeDeploy continuously evaluates alarm states throughout the deployment lifecycle. If any configured alarm triggers, CodeDeploy immediately aborts the deployment, reverts load balancer listener rules to 100% original target group traffic, and cleans up replacement tasks, minimizing customer impact.

---

### Question 3: Running Automated Post-Deployment Verification via CodePipeline

**Answer:** B  
**Explanation:** 
AWS CodeBuild is the standard test runner action in AWS CodePipeline. Following a deployment stage, adding a subsequent CodeBuild action allows executing integration tests, end-to-end UI tests (e.g. Cypress or Selenium in headless Chromium), or load tests against the deployed URL. A test failure returns a non-zero exit code, automatically halting the pipeline before it reaches the production approval stage.

---

### Question 4: Synthetics Canaries as Continuous Pipeline Deployment Gates

**Answer:** A  
**Explanation:** 
CloudWatch Synthetics canaries continuously emit metrics into the `CloudWatchSynthetics` namespace, including `SuccessPercent` and `Duration`. Creating a CloudWatch alarm on `SuccessPercent < 99` and adding it to the CodeDeploy Deployment Group allows CodeDeploy to monitor real synthetic end-to-end journey health during the canary phase, automatically initiating a rollback if synthetic user journeys fail.

---

### Question 5: Safe Database Migrations in Blue/Green Pipelines

**Answer:** B  
**Explanation:** 
In blue/green and canary deployments, both the previous version (blue) and new version (green) exist and serve live traffic concurrently during the evaluation window. If a migration destructively alters or renames columns, the still-running blue version will crash. The **Expand-Contract (Parallel Run) pattern** mandates additive, non-breaking migrations first, followed by application deployment, followed by deprecated column cleanup in a separate future release.

---

### Question 6: CodeDeploy Rollback Overrides on Consecutive Failures

**Answer:** B  
**Explanation:** 
CodeDeploy enforces failure thresholds. When the number of failed hosts exceeds the configured limit (or minimum healthy host threshold), CodeDeploy marks the overall deployment as `Failed`. Instances that failed lifecycle hooks are abandoned and terminated by the ASG, while remaining healthy instances running the previous revision continue serving traffic without being touched.

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

