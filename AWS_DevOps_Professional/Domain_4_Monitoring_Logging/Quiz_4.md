# Domain 4 (Monitoring and Logging) — Quiz 4: Logs Insights, Contributor Insights & Data Protection

**Topic:** CloudWatch Logs Insights Syntax, Contributor Insights Top-Talkers & PII Masking Data Protection  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: CloudWatch Logs Insights Query Syntax for Percentile Latency

A DevOps engineer needs to analyze application response times from a CloudWatch Log Group containing millions of API access records. The logs include fields `@timestamp`, `requestId`, `status`, and `durationMs`. The engineer needs to calculate the 95th percentile (`p95`) and 99th percentile (`p99`) latency, grouped in 5-minute time buckets for requests returning HTTP status 200.

**Which CloudWatch Logs Insights query achieves this?**
- A. `SELECT p95(durationMs), p99(durationMs) FROM logs GROUP BY 5m WHERE status = 200`
- B. 
```sql
filter status = 200
| stats pct(durationMs, 95) as p95, pct(durationMs, 99) as p99 by bin(5m)
```
- C. 
```sql
filter status == 200
| group by 5min
| calculate percentiles(durationMs)
```
- D. `parse "durationMs" | aggregate 95%, 99%`

---

### Question 2: CloudWatch Contributor Insights for VPC Flow Logs Top Talkers

A network security team observes an unexpected spike in outbound data transfer costs across their VPC NAT Gateways. They need to identify the top internal source IP addresses transmitting the highest volume of bytes to external destinations in real time.

**Which service feature provides real-time ranking and visualization of top talkers from log data?**
- A. Amazon Athena scheduled queries running every 24 hours.
- B. **CloudWatch Contributor Insights**, by creating a rule that analyzes VPC Flow Logs to track the top contributors for `srcAddr`, summing the `bytes` field over time.
- C. AWS Trusted Advisor data transfer recommendations.
- D. AWS Cost Explorer monthly reports.

---

### Question 3: CloudWatch Logs Data Protection Policies for Sensitive PII Masking

A healthcare application logs user activity. An accidental application bug starts printing Social Security Numbers (SSNs), Credit Card Numbers, and email addresses into application log streams. The compliance officer requires that all sensitive PII be masked immediately upon ingestion without changing application source code or discarding entire log messages.

**How should the DevOps engineer implement real-time PII masking on the CloudWatch Log Group?**
- A. Deploy an intermediate proxy server to rewrite logs.
- B. Attach a **CloudWatch Logs Data Protection Policy** to the log group, defining managed data identifiers (such as `SSN`, `CreditCardNumber`, `EmailAddress`) and audit/de-identify masking actions.
- C. Delete the log group every 2 hours.
- D. Encrypt the entire log group with an asymmetric KMS RSA key.

---

### Question 4: Multi-Log Group Queries in CloudWatch Logs Insights

An engineer needs to investigate an incident where a user transaction touched three different microservices: `OrderService`, `PaymentService`, and `NotificationService`. Each service writes to its own separate CloudWatch Log Group in `us-east-1`.

**How can the engineer correlate events for a specific `correlationId` across all three services using CloudWatch Logs Insights?**
- A. Merge all three log groups into a single log group using the AWS CLI.
- B. Select all three log groups simultaneously in the CloudWatch Logs Insights console (or pass multiple log group names in the `StartQuery` API call) and run a single query: `filter correlationId = "corr-12345" | sort @timestamp asc`.
- C. Download all logs to Excel spreadsheets and run VLOOKUP.
- D. CloudWatch Logs Insights cannot query more than one log group per query.

---

### Question 5: Live Tail in CloudWatch Logs for Deployment Troubleshooting

During a blue/green deployment, an engineer notices that several newly launched container instances are failing internal health checks. The engineer needs to view incoming application log records instantaneously as they are being written to CloudWatch Logs, similar to `tail -f`.

**Which feature allows real-time interactive streaming of ingested logs directly in the CloudWatch console or CLI?**
- A. CloudWatch Metric Filters.
- B. **CloudWatch Logs Live Tail**.
- C. S3 Select.
- D. CloudWatch Metric Math.

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

### Question 1: CloudWatch Logs Insights Query Syntax for Percentile Latency

**Answer:** B  
**Explanation:** 
CloudWatch Logs Insights query syntax uses pipe operators (`|`). The `filter` directive filters log records. The `stats` command supports percentile aggregation functions (`pct(field, percentile)`), and `bin(period)` groups timestamps into discrete time intervals (e.g., `bin(5m)`).

---

### Question 2: CloudWatch Contributor Insights for VPC Flow Logs Top Talkers

**Answer:** B  
**Explanation:** 
**CloudWatch Contributor Insights** analyzes log data in real time to generate time-series metrics showing top contributors (e.g., top talkers, highest-frequency error codes, heaviest API callers). By writing a Contributor Insights rule targeting VPC Flow Logs with keys `srcAddr` and aggregate operation `sum(bytes)`, the network team gets continuous real-time ranking of top IP addresses driving bandwidth consumption.

---

### Question 3: CloudWatch Logs Data Protection Policies for Sensitive PII Masking

**Answer:** B  
**Explanation:** 
**CloudWatch Logs Data Protection** enables data protection policies on log groups using machine learning and pattern matching against managed data identifiers (SSNs, credit cards, bank account numbers, emails, IP addresses, AWS secret keys) or custom regex patterns. When enabled, matching sensitive data in log events is dynamically masked with asterisks (`***`) at ingestion time for unauthorized viewers, while allowing privileged IAM principals with `logs:Unmask` permissions to view the original values if authorized.

---

### Question 4: Multi-Log Group Queries in CloudWatch Logs Insights

**Answer:** B  
**Explanation:** 
CloudWatch Logs Insights natively supports querying up to **50 log groups simultaneously** in a single query (via both the AWS Management Console and the `StartQuery` API). Logs from all specified log groups are queried, unified, and filtered in a single operational view, making cross-microservice tracing and transaction correlation straightforward.

---

### Question 5: Live Tail in CloudWatch Logs for Deployment Troubleshooting

**Answer:** B  
**Explanation:** 
**CloudWatch Logs Live Tail** provides a real-time, interactive streaming view of ingested logs directly in the AWS Console or via the AWS CLI (`aws logs start-live-tail`). It functions like `tail -f`, allowing developers and DevOps engineers to view, filter, and highlight incoming log events across multiple log groups in real time during deployments and live debugging sessions.

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

