# Domain 4 (Monitoring and Logging) — Quiz 3: CloudWatch Logs Architecture & Streaming Pipelines

**Topic:** Log Groups, Retention Policies, Metric Filters & Subscription Filters (Kinesis, OpenSearch, Lambda)  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Log Metric Filter Extraction for HTTP 5xx Response Codes

An Apache web server pushes access logs formatted in JSON to Amazon CloudWatch Logs:
`{"timestamp": "2026-10-07T12:00:00Z", "status": 500, "responseTime": 0.450, "endpoint": "/api/checkout"}`.
The DevOps engineer wants to create a custom metric that counts the number of 5xx HTTP server errors.

**Which CloudWatch Logs Metric Filter pattern correctly matches HTTP 5xx errors?**
- A. `{ $.status = 5* }`
- B. `{ ($.status >= 500) && ($.status <= 599) }`
- C. `SELECT count(*) WHERE status LIKE '5%'`
- D. `[status = 500]`

---

### Question 2: CloudWatch Logs Subscription Filter to Kinesis Data Firehose

A security compliance mandate requires that all application log groups in CloudWatch Logs be streamed in near real-time to an Amazon S3 data lake in Parquet format, partitioned by year/month/day/hour.

**What is the most cost-effective and scalable serverless architecture to achieve this?**
- A. A cron job on an EC2 instance running `aws logs get-log-events` and writing to S3.
- B. A **CloudWatch Logs Subscription Filter** pointing to an **Amazon Kinesis Data Firehose** delivery stream, which performs automated format conversion to Apache Parquet (using an AWS Glue Data Catalog table) and delivers to the S3 bucket with dynamic partitioning.
- C. Exporting logs manually using the `CreateExportTask` API every 5 minutes.
- D. Writing a custom Lambda function triggered by S3 bucket events.

---

### Question 3: Subscription Filter Cross-Account Delivery Permissions

An enterprise wants to stream CloudWatch Logs from multiple development AWS accounts into a centralized Amazon OpenSearch Service cluster located in a dedicated Security AWS Account via Amazon Kinesis Data Streams.

**What AWS IAM and resource policy configuration is required for CloudWatch Logs in Account A to deliver logs to Kinesis Data Streams in Account B?**
- A. The Kinesis Data Stream must have public access enabled.
- B. A **CloudWatch Logs Destination** (`PutDestination`) must be created in Account B with an access policy allowing Account A (`logs.amazonaws.com` principal) to call `PutSubscriptionFilter`, and a role allowing CloudWatch Logs to write to the Kinesis Data Stream (`kinesis:PutRecord`). Account A then attaches its subscription filter to this destination.
- C. Both accounts must share the same root password.
- D. VPC Peering alone automatically authorizes CloudWatch Logs streaming without IAM.

---

### Question 4: Automated Retention Policy Enforcement on New Log Groups

Developers frequently deploy serverless microservices via AWS CDK and SAM that create new CloudWatch Log Groups without setting a retention period (defaulting to "Never Expire"). Over time, this leads to thousands of dollars in unnecessary log storage costs for 3-year-old test logs.

**How can a DevOps engineer automatically enforce a 30-day retention period on all newly created CloudWatch Log Groups across an AWS Region?**
- A. Write an SCP that blocks the `logs:CreateLogGroup` API.
- B. Create an **Amazon EventBridge Rule** matching the AWS API Call via CloudTrail for `CreateLogGroup` from `logs.amazonaws.com`, triggering an **AWS Lambda function** or **Systems Manager Automation runbook** that executes `PutRetentionPolicy` with `retentionInDays: 30`.
- C. Change the default AWS account-level log retention setting in the Billing Console.
- D. Configure an S3 Lifecycle rule targeting CloudWatch internal storage.

---

### Question 5: CloudWatch Logs Batch Export Limitations

A team needs to back up 500 GB of historical CloudWatch Logs to Amazon S3 for an annual audit. An engineer writes a shell script invoking `aws logs create-export-task` concurrently across 20 log groups in parallel, but the script fails immediately with `LimitExceededException`.

**Why did the export tasks fail, and what is the architectural constraint?**
- A. S3 buckets can only accept one log file per day.
- B. CloudWatch Logs supports only **one active export task per account per Region** at any given time; parallel `create-export-task` calls are rejected until the running task reaches `COMPLETED`, `CANCELLED`, or `FAILED`.
- C. S3 export tasks cannot exceed 10 MB in total size.
- D. The KMS key expired during execution.

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

### Question 1: Log Metric Filter Extraction for HTTP 5xx Response Codes

**Answer:** B  
**Explanation:** 
CloudWatch Logs metric filter syntax for JSON logs allows numeric comparisons using JSON path syntax: `{ ($.status >= 500) && ($.status <= 599) }`. Metric filters can extract values to publish as metric values or increment a counter metric (e.g., metric value `1`) on each matching log event.

---

### Question 2: CloudWatch Logs Subscription Filter to Kinesis Data Firehose

**Answer:** B  
**Explanation:** 
CloudWatch Logs Subscription Filters provide real-time streaming of log events to Kinesis Data Firehose, Kinesis Data Streams, or AWS Lambda. Kinesis Data Firehose integrates natively with AWS Glue Data Catalog to convert incoming JSON logs to columnar Apache Parquet or ORC formats on the fly and buffer/batch deliveries into Amazon S3 with custom partition keys. `CreateExportTask` is not continuous or real-time (it is designed for cold historical batch exports).

---

### Question 3: Subscription Filter Cross-Account Delivery Permissions

**Answer:** B  
**Explanation:** 
For cross-account CloudWatch Logs subscriptions, the receiving account (Account B) creates a CloudWatch Logs Destination via `aws logs put-destination`. The destination specifies an IAM role that grants permissions to write to the recipient stream (Kinesis Data Streams). Account B attaches an access policy (`put-destination-policy`) granting the source account (Account A) permission to publish to the destination. Account A then creates a subscription filter referencing the destination ARN.

---

### Question 4: Automated Retention Policy Enforcement on New Log Groups

**Answer:** B  
**Explanation:** 
There is no native global account setting to change the default "Never Expire" policy of newly created CloudWatch Log Groups. The standard enterprise pattern is event-driven automation: EventBridge captures the `CreateLogGroup` event from CloudTrail and triggers a Lambda function or SSM Automation runbook to immediately apply `PutRetentionPolicy` with `retentionInDays=30`.

---

### Question 5: CloudWatch Logs Batch Export Limitations

**Answer:** B  
**Explanation:** 
The CloudWatch Logs `create-export-task` API has a hard quota of **1 active (running or pending) export task per AWS account per Region**. Attempting to start concurrent export tasks throws `LimitExceededException`. To export multiple log groups or large date ranges, tasks must be queued and executed sequentially, or streaming subscription filters (Kinesis Firehose -> S3) should be used instead.

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

