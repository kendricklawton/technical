# Domain 4 (Monitoring and Logging) — Quiz 6: Distributed Tracing with AWS X-Ray & ADOT

**Topic:** X-Ray Daemon, Sampling Rules, Annotations vs Metadata & AWS Distro for OpenTelemetry (ADOT)  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Distinguishing X-Ray Annotations from Metadata

A backend engineer instruments a microservice using the AWS X-Ray SDK. The service processes incoming customer orders. The engineer needs to attach two pieces of information to the trace:
1. `customerTier` (e.g., "Gold", "Silver") which must be **searchable and filterable** in the X-Ray console using filter expressions like `annotation.customerTier = "Gold"`.
2. A large JSON payload of the raw HTTP request body for debugging purposes, which does not need to be indexed.

**How should these two data items be added to the X-Ray trace segment?**
- A. Both should be added as Metadata.
- B. `customerTier` should be added as an **Annotation** (key-value pair indexed for searching and filtering), while the raw HTTP payload should be added as **Metadata** (key-value pair that can store any data type, including objects/arrays, but is NOT indexed for search).
- C. Both should be added as Annotations.
- D. Both should be logged to standard output without SDK calls.

---

### Question 2: X-Ray Sampling Rules Configuration

A high-throughput API gateway handles 20,000 requests per second. Instrumenting and tracing 100% of requests would incur high X-Ray data ingestion costs and network overhead. The engineering lead wants to trace:
- Exactly 50 requests per second unconditionally (guaranteed baseline).
- 5% of any additional requests beyond the 50 req/sec threshold.

**How can this sampling policy be configured dynamically without redeploying microservice code?**
- A. Hardcode a random number generator in the application code.
- B. Configure an **AWS X-Ray Sampling Rule** with `FixedRate: 0.05` (5%) and `ReservoirSize: 50` (first 50 requests per second). The X-Ray SDK polls the X-Ray service for updated sampling rules every 10 seconds.
- C. Use CloudFront caching rules.
- D. Restrict the IAM role of the EC2 instances to only allow 50 `PutTraceSegments` calls per second.

---

### Question 3: Tracing Distributed Transactions Across SQS and Asynchronous Workers

A web application receives an order, writes an order event to Amazon SQS, and an asynchronous worker running on Amazon ECS pulls the message from SQS to process fulfillment. When viewing the X-Ray Service Map, the web service and the ECS worker appear as two completely disjoint, disconnected graphs.

**Why did the trace context break between the web service and the worker microservice?**
- A. SQS does not support message attributes.
- B. The web application did not inject the X-Ray trace header (`AWSTraceHeader`) into the **SQS message attributes**, or the consumer did not retrieve the trace header from the message attributes and pass it to the X-Ray SDK when starting the worker segment.
- C. SQS requires Multi-AZ configuration to support X-Ray.
- D. X-Ray only works with synchronous HTTP calls and cannot trace asynchronous messaging.

---

### Question 4: AWS Distro for OpenTelemetry (ADOT) vs Traditional X-Ray SDK

A modern enterprise is standardizing its containerized microservices (running on Amazon EKS) on open-source, vendor-neutral observability standards. They want to collect metrics, logs, and distributed traces and send them to AWS X-Ray, Amazon CloudWatch, and an external Prometheus instance using a single unified agent.

**Which technology should the DevOps architect recommend?**
- A. Traditional AWS X-Ray Daemon installed on each developer laptop.
- B. **AWS Distro for OpenTelemetry (ADOT)** deployed as a Collector daemonset/sidecar, which implements the open-source CNCF OpenTelemetry standards and exports traces to X-Ray and metrics to CloudWatch/Prometheus simultaneously.
- C. AWS Systems Manager Run Command running `tcpdump`.
- D. CloudWatch Unified Agent configured only for disk metrics.

---

### Question 5: X-Ray Subsegments for Remote Downstream Dependencies

An application running on EC2 queries an external third-party payment gateway via HTTPS and an Amazon Aurora database via SQL. The developer notices that while the overall HTTP request duration is captured in the X-Ray trace, there is no breakdown showing how much time was spent waiting for the third-party payment gateway versus the SQL database.

**How can the developer obtain granular timing breakdowns for external calls within an X-Ray segment?**
- A. Create **Subsegments** for downstream HTTP client calls and database queries using the X-Ray SDK or automatic client instrumentor.
- B. Launch an additional EC2 instance for each external call.
- C. Write log timestamps to a local text file and compare them manually.
- D. Increase the TCP window size on the network interface.

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

### Question 1: Distinguishing X-Ray Annotations from Metadata

**Answer:** B  
**Explanation:** 
In AWS X-Ray:
- **Annotations**: Key-value pairs indexed by the X-Ray service. They support string, number, and boolean values and are designed for filtering traces in the console or API (e.g., `annotation.customerTier = "Gold"`).
- **Metadata**: Key-value pairs that can contain arbitrary objects, lists, or large payloads (serialized as JSON). Metadata is **not indexed** and cannot be used in search filter expressions, but is visible when viewing the specific trace details.

---

### Question 2: X-Ray Sampling Rules Configuration

**Answer:** B  
**Explanation:** 
AWS X-Ray Sampling Rules allow centralized, dynamic control of trace sampling rates. A sampling rule defines:
- **Reservoir**: The minimum number of requests to sample every second before applying the fixed rate (`ReservoirSize: 50`). This guarantees a steady stream of traces even in low-traffic periods.
- **Fixed Rate**: The percentage of requests to sample after the reservoir is exhausted (`FixedRate: 0.05` = 5%).
The X-Ray SDKs continuously poll the X-Ray backend for rule changes, allowing adjustments without application redeployment.

---

### Question 3: Tracing Distributed Transactions Across SQS and Asynchronous Workers

**Answer:** B  
**Explanation:** 
In distributed systems involving message queues (SQS, SNS, EventBridge), the HTTP `X-Amzn-Trace-Id` header is not automatically propagated across asynchronous message boundaries unless explicitly handled. The publisher must include the trace header as an SQS message attribute, and the downstream consumer must extract `AWSTraceHeader` from the received message and pass it to the X-Ray SDK to continue the trace context, linking the services on the X-Ray Service Map.

---

### Question 4: AWS Distro for OpenTelemetry (ADOT) vs Traditional X-Ray SDK

**Answer:** B  
**Explanation:** 
**AWS Distro for OpenTelemetry (ADOT)** is an AWS-supported distribution of the open-source CNCF OpenTelemetry project. With ADOT, developers instrument their applications once using open-source OpenTelemetry APIs/SDKs. The ADOT Collector can then receive, process, and export telemetry data to multiple destinations simultaneously, such as sending traces to AWS X-Ray, metrics to CloudWatch and Amazon Managed Service for Prometheus, avoiding vendor lock-in.

---

### Question 5: X-Ray Subsegments for Remote Downstream Dependencies

**Answer:** A  
**Explanation:** 
Within an AWS X-Ray **Segment** (which represents the compute resource hosting the application), developers can create **Subsegments**. Subsegments provide granular timing, metadata, and error details for downstream HTTP/HTTPS API calls, AWS SDK service calls (DynamoDB, S3), and SQL database queries. Many X-Ray SDKs provide automatic instrumentors (e.g., wrapping `HttpClient` or database drivers) that automatically generate subsegments for all outbound remote calls.

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

