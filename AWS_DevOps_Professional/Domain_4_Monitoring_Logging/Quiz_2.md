# Domain 4 (Monitoring and Logging) — Quiz 2: Advanced CloudWatch Alarms & Cross-Account Telemetry

**Topic:** High-Resolution Metrics, Composite Alarms, Anomaly Detection & Cross-Account Observability  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: High-Resolution Custom Metrics and Rapid Alarm Evaluation

A financial trading platform requires detection of application latency spikes within 10 seconds of occurrence to trigger an automated rollback. Standard CloudWatch metrics publish at 1-minute or 5-minute intervals, which is too slow.

**How should the DevOps engineer publish metrics and configure CloudWatch Alarms to meet this sub-minute detection requirement?**
- A. Publish metrics with `StorageResolution=1` (High-Resolution Metric) using `PutMetricData`, and configure the CloudWatch Alarm with a period of 10 seconds or 30 seconds.
- B. Use CloudWatch Synthetics running every 1 second.
- C. Publish log lines to CloudWatch Logs and rely on standard Metric Filters.
- D. Stream metrics to S3 and run Athena queries every 5 seconds.

---

### Question 2: CloudWatch Composite Alarms for Alert Fatigue Reduction

An operations team manages an e-commerce microservice. When an underlying RDS database slows down, the team receives 15 distinct alerts simultaneously: EC2 CPU spike, ECS task memory pressure, ALB 504 Gateway Timeouts, queue depth buildup, and latency alarms. The on-call engineers suffer from alert fatigue.

**How should the DevOps engineer configure CloudWatch to consolidate these alerts into a single actionable incident trigger?**
- A. Delete all CloudWatch alarms and rely only on daily email digests.
- B. Create a **CloudWatch Composite Alarm** that uses boolean logic (`ALARM("RDS_CPU") AND ALARM("ALB_Latency") AND ALARM("ECS_504s")`) to evaluate multiple metric alarms together and trigger a single SNS notification only when the combined systemic failure condition occurs.
- C. Group the alarms into an AWS Systems Manager maintenance window.
- D. Lower the metric thresholds so the alarms never trigger.

---

### Question 3: CloudWatch Anomaly Detection for Cyclical Traffic Patterns

An online streaming platform experiences high user traffic every weekday evening from 7 PM to 11 PM and very low traffic at 4 AM. Using fixed-threshold CloudWatch Alarms results in false-positive alerts at night and missed performance anomalies during peak evening traffic.

**What CloudWatch feature should the DevOps engineer implement to dynamically alert on abnormal deviations?**
- A. Configure a cron expression on the EC2 instances to restart them every night.
- B. Enable **CloudWatch Anomaly Detection** on the metric, which applies machine learning algorithms to past metric data to model hourly, daily, and weekly seasonality, and alert when the metric breaches the predicted normal band (e.g., beyond 2 standard deviations).
- C. Use AWS Step Functions to adjust the static alarm threshold every hour.
- D. Disable monitoring during peak evening hours.

---

### Question 4: Cross-Account and Cross-Region CloudWatch Observability

An enterprise has 50 AWS accounts organized under AWS Organizations. The Central Monitoring Account (`Account-Central`) needs visibility into CloudWatch metrics, logs, and traces from all 49 spoke member accounts without requiring on-call engineers to log into individual spoke accounts.

**What is the AWS recommended architectural solution for centralized observability?**
- A. Write a Lambda script in each account that calls `AssumeRole` and exports metrics to CSV files.
- B. Set up **CloudWatch Cross-Account Observability** by configuring the central account as a monitoring account and the spoke accounts as source accounts via AWS Organizations, enabling seamless cross-account dashboarding, log searching, and X-Ray distributed trace aggregation.
- C. Deploy third-party monitoring agents on all EC2 instances to bypass CloudWatch.
- D. Use VPC Peering between all 50 accounts and run Prometheus.

---

### Question 5: CloudWatch Alarm Missing Data Treatment

A batch processing microservice runs once every Sunday morning. A CloudWatch Alarm monitors the error count metric. During the rest of the week, the microservice does not run, and no metrics are published (`-` null data points). The alarm keeps flipping to `INSUFFICIENT_DATA`, triggering confused alerts from junior operators.

**How should the DevOps engineer configure the alarm's missing data evaluation?**
- A. Set `TreatMissingData` to `ignore` (maintain current alarm state) or `notBreaching` (treat missing data as within threshold).
- B. Set `TreatMissingData` to `breaching`.
- C. Force EC2 instances to publish dummy `0` values 24/7 using an infinite bash while loop.
- D. Delete the alarm after each batch execution.

---

### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: High-Resolution Custom Metrics and Rapid Alarm Evaluation

**Answer:** A  
**Explanation:** 
AWS CloudWatch supports **High-Resolution Metrics** with resolution down to 1 second (`StorageResolution=1`). When high-resolution metrics are published via `PutMetricData`, you can configure CloudWatch Alarms with evaluation periods of 10 seconds or 30 seconds (or multiples of 1 second/10 seconds), enabling rapid incident detection and sub-minute automated response.

---

### Question 2: CloudWatch Composite Alarms for Alert Fatigue Reduction

**Answer:** B  
**Explanation:** 
**CloudWatch Composite Alarms** evaluate the state of multiple underlying standard metric alarms using rule expressions with boolean operators (`AND`, `OR`, `NOT`). This enables teams to trigger notifications (SNS, PagerDuty, Incident Manager) only when true multi-symptom outages occur, suppressing redundant noisy alerts and drastically reducing alert fatigue.

---

### Question 3: CloudWatch Anomaly Detection for Cyclical Traffic Patterns

**Answer:** B  
**Explanation:** 
CloudWatch Anomaly Detection uses statistical and machine learning models trained on historical metric trends. It automatically generates expected metric bands that account for predictable hourly, daily, or weekly cyclical patterns. Alarms can be set to trigger when the metric is greater than the upper band, lower than the lower band, or outside both bands (`ANOMALY_DETECTION_BAND(m1, 2)`).

---

### Question 4: Cross-Account and Cross-Region CloudWatch Observability

**Answer:** B  
**Explanation:** 
AWS CloudWatch provides native **Cross-Account Observability**. By defining a monitoring account and sharing source accounts (which can be automated across an entire AWS Organization or specific OUs), users in the monitoring account can view cross-account dashboards, execute CloudWatch Logs Insights queries across multiple log groups in multiple accounts, and view end-to-end distributed traces in AWS X-Ray without custom export infrastructure.

---

### Question 5: CloudWatch Alarm Missing Data Treatment

**Answer:** A  
**Explanation:** 
CloudWatch Alarms support configuring how missing data points are treated via `TreatMissingData`:
- `notBreaching`: Missing data is treated as normal/healthy (ideal for batch jobs or sparse error metrics).
- `ignore`: The alarm maintains its current state and does not evaluate missing data points.
- `breaching`: Missing data is treated as an alarm breach (useful if failure to report indicates instance death).
- `missing`: Standard default, flips to `INSUFFICIENT_DATA`.

---


### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

