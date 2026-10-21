# Domain 5 (Incident and Event Response) — Quiz 2: Enterprise EventBridge Patterns

**Topic:** Cross-Account / Cross-Region Event Routing, Archive & Replay & EventBridge Pipes  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Cross-Account EventBridge Event Routing

An enterprise maintains multiple spoke AWS accounts (Account A, Account B) that generate security and operational events. The enterprise needs to consolidate all events into a central security incident response bus in Account Central (`111122223333`).

**What combination of configurations is required to allow Account A to forward events to the central event bus in Account Central?**
- A. Open all security groups between the VPCs in Account A and Account Central.
- B. In Account Central, configure a **Resource-Based Policy on the custom event bus** granting `events:PutEvents` permission to Account A. In Account A, create an EventBridge rule whose target is the ARN of the custom event bus in Account Central, backed by an IAM execution role in Account A.
- C. Establish AWS Site-to-Site VPN between the two accounts.
- D. Share the root credentials of Account Central with Account A.

---

### Question 2: EventBridge Archive and Replay for Incident Recovery

A critical microservice encountered a software bug that caused all consumer Lambda functions to drop orders received between 14:00 and 15:30 UTC. The bug in the Lambda code has been patched and redeployed. The DevOps team needs to reprocess all orders that occurred during that 90-minute window without asking customers to re-submit orders.

**Which EventBridge feature natively resolves this issue?**
- A. AWS Snowball Edge device data transfer.
- B. **EventBridge Archive and Replay**, by creating a Replay targeting the event bus or rule, specifying the start time (14:00 UTC) and end time (15:30 UTC) from the pre-configured event archive.
- C. Running a MySQL dump on the Lambda function memory.
- D. EventBridge does not retain past events; they are permanently lost.

---

### Question 3: EventBridge Pipes Point-to-Point Integration

An engineering team needs to consume stream records from an Amazon DynamoDB Streams source, filter out all events except `INSERT` events where `totalAmount > 100`, transform the JSON payload using an AWS Lambda function, and deliver the result into an Amazon SQS queue. They want zero custom polling infrastructure or custom consumer code.

**Which AWS capability is designed specifically for this point-to-point stream processing pipeline?**
- A. AWS Direct Connect.
- B. **Amazon EventBridge Pipes**, configuring the DynamoDB Stream as the Source, applying a filter criteria pattern on event type and totalAmount, configuring the Lambda function as the Enrichment step, and setting the SQS queue as the Target.
- C. Writing a daemon on an EC2 instance that runs the DynamoDB Kinesis Client Library (KCL).
- D. AWS Batch with Spot instances running every 10 seconds.

---

### Question 4: Dead-Letter Queues (DLQ) for EventBridge Rule Targets

An EventBridge rule routes critical order events to an external third-party payment vendor via an API Destination target. If the external payment vendor experiences an API outage lasting 4 hours, EventBridge retries sending events for up to 24 hours. If the target continues to fail after all retries are exhausted, the events could be dropped.

**How can the DevOps team ensure that dropped events are captured for manual inspection and audit?**
- A. Configure a **Dead-Letter Queue (DLQ)** using Amazon SQS on the specific EventBridge target configuration.
- B. EventBridge automatically writes failed events to a local text file on the AWS console.
- C. Set the retry policy to 100 years.
- D. Add an Elastic IP to the API destination.

---

### Question 5: EventBridge Schema Registry and Code Bindings

A large distributed organization has dozens of teams publishing custom JSON events to shared company event buses. Teams frequently break downstream consumer applications by changing event schemas, renaming fields, or removing required attributes without notice.

**How can the organization automate event contract validation and developer tooling?**
- A. Require all developers to communicate via Slack before modifying code.
- B. Enable **EventBridge Schema Registry** and **Schema Discovery** to automatically detect and register event schemas from the bus, and generate strongly-typed **code bindings** (for Java, Python, TypeScript) to validate event models in CI/CD pipelines.
- C. Store JSON schemas in S3 and manually inspect git diffs.
- D. Convert all JSON events to plain text unformatted strings.

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

### Question 1: Cross-Account EventBridge Event Routing

**Answer:** B  
**Explanation:** 
To send events across AWS accounts using Amazon EventBridge:
1. The receiving event bus in the target account (Account Central) must have a resource-based policy granting permission (`events:PutEvents`) to the sender account ID or organization.
2. The sender account (Account A) creates an EventBridge rule that specifies the ARN of the target event bus as its destination.
3. The sender account rule requires an IAM role with permission to put events onto the target bus.

---

### Question 2: EventBridge Archive and Replay for Incident Recovery

**Answer:** B  
**Explanation:** 
Amazon EventBridge **Archive and Replay** allows you to record all events matching specific filters or all events on an event bus into an Archive with configurable retention (including indefinite). If an application failure, downstream outage, or bug occurs, you can initiate a **Replay** specifying a past time range. EventBridge will re-publish those archived events to the specified event bus or rules, allowing downstream systems to reprocess failed messages cleanly.

---

### Question 3: EventBridge Pipes Point-to-Point Integration

**Answer:** B  
**Explanation:** 
**Amazon EventBridge Pipes** provides a simpler, consistent, and cost-effective way to create point-to-point integrations between event producers and consumers. A Pipe consists of:
1. **Source** (DynamoDB Streams, Kinesis, SQS, Kafka, MQ).
2. **Filtering** (built-in event pattern filtering without compute).
3. **Enrichment** (optional Lambda, Step Functions, API Gateway, or API Destinations to modify or validate data).
4. **Target** (SQS, SNS, Step Functions, Kinesis, EventBridge Bus, etc.).

---

### Question 4: Dead-Letter Queues (DLQ) for EventBridge Rule Targets

**Answer:** A  
**Explanation:** 
EventBridge allows attaching an Amazon SQS Dead-Letter Queue (DLQ) directly to individual targets of an EventBridge rule. If EventBridge exhausts its retry attempts (configurable up to 24 hours and up to 185 retries) or if the message retention window expires, it delivers the failed event payload along with error details (such as HTTP response status codes and failure reasons) directly into the configured SQS DLQ.

---

### Question 5: EventBridge Schema Registry and Code Bindings

**Answer:** B  
**Explanation:** 
The Amazon EventBridge **Schema Registry** stores event schemas. With **Schema Discovery** enabled, EventBridge automatically inspects events sent to an event bus and creates schema definitions. Developers can download strongly-typed code bindings (classes/interfaces in Java, Python, TypeScript, or Go) directly into their IDEs and codebases. This enables schema validation at build time in CI/CD pipelines, preventing contract breakages across teams.

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

