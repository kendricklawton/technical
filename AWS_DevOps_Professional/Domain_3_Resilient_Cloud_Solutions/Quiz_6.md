# Domain 3 (Resilient Cloud Solutions) — Quiz 6: Resilient Storage, Messaging & Networking

**Topic:** S3 CRR & MRAP, EFS Multi-AZ Mounts, SQS Dead-Letter Queues & ALB Connection Draining  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Amazon S3 Cross-Region Replication with KMS CMK Encrypted Objects

A security enterprise configures Amazon S3 Cross-Region Replication (CRR) between `bucket-primary` in `us-east-1` and `bucket-secondary` in `eu-central-1`. Objects uploaded to `bucket-primary` are encrypted using AWS KMS Customer Managed Keys (CMKs). After enabling CRR, the security team notices that newly uploaded objects are **not** replicating to the secondary bucket.

**What two configuration steps are required to enable S3 CRR for objects encrypted with KMS CMKs?**
- A. Switch the primary bucket encryption to S3 Managed Keys (SSE-S3), as KMS is unsupported in replication.
- B. Enable **KMS encryption support in the S3 replication rule**, specify the destination KMS key ARN, and grant the S3 replication IAM role permissions to decrypt with the source KMS key (`kms:Decrypt`) and encrypt with the destination KMS key (`kms:GenerateDataKey` / `kms:Encrypt`).
- C. Add a public bucket policy allowing `*` access to S3 GetObject.
- D. Mount both buckets to an EC2 instance and run `rsync` via cron.

---

### Question 2: Amazon S3 Multi-Region Access Points (MRAP) with Failover Controls

A global media company uses Amazon S3 Multi-Region Access Points (MRAP) to provide a single global endpoint (`mrap-alias.accesspoint.s3-global.amazonaws.com`) routing client traffic to buckets in `us-east-1` and `eu-west-1` based on AWS Global Accelerator anycast routing. During a maintenance window in `us-east-1`, the operations team needs to dynamically shift 100% of S3 PUT and GET requests to `eu-west-1`.

**How can the team execute this traffic shift with minimal disruption and deterministic control?**
- A. Update the public DNS records of the client applications to resolve to the raw bucket name.
- B. Use **S3 Multi-Region Access Point Failover Controls** to set the routing status of `us-east-1` to **Inactive** and ensure `eu-west-1` is **Active**.
- C. Delete the S3 bucket in `us-east-1`.
- D. Disable AWS Global Accelerator in the AWS account.

---

### Question 3: Amazon SQS Dead-Letter Queue (DLQ) and Redrive Policy Configuration

A backend payment processing microservice consumes messages from an Amazon SQS standard queue (`PaymentQueue`). Some malformed messages crash the worker threads repeatedly, blocking the queue and consuming excessive compute cycles. The team decides to configure a Dead-Letter Queue (`PaymentDLQ`).

**How does SQS determine when to move a problematic message from `PaymentQueue` to `PaymentDLQ`?**
- A. When the message age exceeds 24 hours.
- B. When the message's `ApproximateReceiveCount` exceeds the configured `maxReceiveCount` in the queue's redrive policy.
- C. When the worker process writes an error log to CloudWatch Logs.
- D. When the message payload exceeds 256 KB.

---

### Question 4: SQS DLQ Redrive to Source Queue via API / CLI

After fixing the bug in the payment processing worker microservice, the DevOps team needs to reprocess 15,000 messages that accumulated in the `PaymentDLQ`.

**What is the most efficient, native AWS mechanism to move these messages back into the source queue?**
- A. Write a custom Python script that runs `ReceiveMessage`, `SendMessage`, and `DeleteMessage` in a loop.
- B. Use the **SQS Dead-Letter Queue (DLQ) Redrive** feature (via AWS Console or `aws sqs start-message-move-task`), which moves messages natively between the DLQ and the source queue (or custom destination queue) at scale without deploying compute or custom code.
- C. Export messages to S3 and reload them via AWS Glue.
- D. Purge the DLQ and request customers resubmit their transactions.

---

### Question 5: Application Load Balancer Deregistration Delay (Connection Draining)

A fleet of microservices runs on Amazon EC2 behind an Application Load Balancer (ALB). During an Auto Scaling scale-in event or deployment, instances receive an in-flight HTTP POST request that takes 45 seconds to complete. By default, instances were being terminated immediately upon deregistration, causing HTTP 502 / 504 errors for active users.

**How should the DevOps engineer configure the ALB Target Group to prevent terminating instances with active transactions?**
- A. Set `deregistration_delay.timeout_seconds` on the Target Group to a value greater than 45 seconds (e.g., 60 seconds).
- B. Increase the ALB idle timeout to 3600 seconds.
- C. Enable Sticky Sessions (affinity cookies) on the Target Group.
- D. Disable ALB cross-zone load balancing.

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

### Question 1: Amazon S3 Cross-Region Replication with KMS CMK Encrypted Objects

**Answer:** B  
**Explanation:** 
By default, S3 Replication does not replicate objects encrypted with AWS KMS CMKs. To replicate KMS-encrypted objects, you must:
1. Explicitly enable KMS replication in the bucket replication rule and provide the target AWS KMS CMK in the destination region.
2. Grant the S3 Replication IAM service role permissions to decrypt using the source KMS key in Region 1 and generate data keys/encrypt using the destination KMS key in Region 2.

---

### Question 2: Amazon S3 Multi-Region Access Points (MRAP) with Failover Controls

**Answer:** B  
**Explanation:** 
S3 Multi-Region Access Point (MRAP) Failover Controls allow DevOps engineers to configure active-active or active-passive traffic routing across multiple regional S3 buckets behind a single global MRAP endpoint. Setting a region's routing status to Inactive shifts all incoming S3 requests to the remaining Active regional bucket(s) within minutes via AWS Global Accelerator anycast network controls, without changing client configuration or endpoints.

---

### Question 3: Amazon SQS Dead-Letter Queue (DLQ) and Redrive Policy Configuration

**Answer:** B  
**Explanation:** 
SQS tracks the number of times a message has been delivered to consumers using the `ApproximateReceiveCount` system attribute. When an application fails to process and delete a message before the visibility timeout expires, the message becomes visible again, and `ApproximateReceiveCount` increments on the next receive call. Once `ApproximateReceiveCount > maxReceiveCount`, SQS automatically moves the message to the configured Dead-Letter Queue (DLQ).

---

### Question 4: SQS DLQ Redrive to Source Queue via API / CLI

**Answer:** B  
**Explanation:** 
AWS SQS includes native Dead-Letter Queue (DLQ) Redrive functionality (and the `start-message-move-task` API). It allows administrators to move messages directly from a DLQ back to their original source queue or a custom destination queue asynchronously at high throughput, completely eliminating the need to write and maintain custom polling/redrive scripts.

---

### Question 5: Application Load Balancer Deregistration Delay (Connection Draining)

**Answer:** A  
**Explanation:** 
Deregistration delay (connection draining) tells the Application Load Balancer how long to wait before forcefully terminating connections to a deregistering target. When a target enters the deregistration process, the ALB stops routing new requests to it while allowing in-flight requests to complete up to `deregistration_delay.timeout_seconds` (default is 300 seconds; can be set between 0 and 3600 seconds). Setting this value above the maximum request duration (45 seconds) prevents dropped connections during deployments and scale-in events.

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

