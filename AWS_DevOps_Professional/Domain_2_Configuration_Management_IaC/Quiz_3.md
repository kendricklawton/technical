# Domain 2 (Configuration Management & IaC) — Quiz 3: Parameter Store & Secrets Management Deep Dive

**Topic:** Systems Manager Parameter Store Tiers, Hierarchies, Dynamic Resolution, Throughput & Policies  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Systems Manager Parameter Store Tiering Limits & Parameter Policies

An enterprise needs to store a 6.5 KB configuration payload and enable automated expiration for temporary access tokens in AWS Systems Manager Parameter Store. When attempting to create the parameter in the Standard tier, the API returns a `ValidationException`.

**Why does this error occur, and how is it resolved?**
- A. The **Standard tier** enforces a maximum parameter size limit of **4 KB** and does not support parameter policies; the parameter must be created using the **Advanced tier**, which supports values up to **8 KB**, parameter policies (such as expiration TTLs), and larger total parameter capacity.
- B. Systems Manager Parameter Store only supports string payloads up to 1 KB; payloads exceeding 1 KB must be compressed as base64 archives and stored within Amazon DynamoDB.
- C. The Parameter Store API enforces client-side encryption using AWS KMS Customer Managed Keys for any payload exceeding 4 KB, requiring the `--kms-key-id` flag during creation.
- D. Advanced tier parameters cannot be created via the AWS CLI; the configuration must be authored as a secret within AWS Secrets Manager to support payloads up to 10 KB.

---

### Question 2: CloudFormation Dynamic Resolution of Encrypted SecureString Parameters

A production CloudFormation template needs to retrieve a sensitive database master password stored as an encrypted `SecureString` in AWS Systems Manager Parameter Store. The password must be decrypted dynamically during stack creation and passed into an `AWS::RDS::DBInstance` resource property without ever exposing the plaintext string in CloudFormation console transcripts, change sets, or template files.

**Which CloudFormation dynamic reference syntax retrieves and passes the decrypted password securely?**
- A. Reference the parameter using the dynamic reference pattern: `{{resolve:ssm-secure:my-db-password:1}}` directly in the database `MasterUserPassword` property.
- B. Define a CloudFormation parameter of type `AWS::SSM::Parameter::Value<String>` with a default value of `my-db-password`, and reference it via `!Ref`.
- C. Use the CloudFormation intrinsic function `!GetAtt SSM::Parameter.Value` with decryption flags configured in the template metadata.
- D. Reference the parameter using the dynamic reference string: `{{resolve:ssm:my-db-password:decrypted}}` in the template parameter mapping.

---

### Question 3: Multi-Account Parameter Sharing Architecture with AWS Resource Access Manager (RAM)

A central cloud engineering account (`111111111111`) maintains standardized AMI IDs, corporate proxy endpoints, and global logging configurations in Systems Manager Parameter Store. These configuration parameters must be shared securely with 60 member accounts in an AWS Organization without copying or synchronizing duplicate parameter records into each account.

**How should multi-account Parameter Store sharing be implemented with the LEAST operational overhead?**
- A. Ensure the parameters are configured in the **Advanced tier** in Account `111111111111`, create a resource share in **AWS Resource Access Manager (AWS RAM)** specifying the parameter ARNs, and share the resource share with the AWS Organization ID or target Organizational Units.
- B. Deploy an AWS Lambda function in Account `111111111111` triggered on EventBridge parameter change events that copies updated parameter values into all 60 member accounts using cross-account STS credentials.
- C. Configure a custom resource policy on the Parameter Store parameters granting `ssm:GetParameter` to `*` with an `aws:PrincipalOrgID` condition.
- D. Deploy an Amazon S3 bucket with Cross-Region Replication enabled, instructing application workloads to fetch configuration objects over HTTP rather than querying Parameter Store.

---

### Question 4: Recursive Hierarchical Parameter Retrieval and Path Traversal

A containerized billing microservice deployed on Amazon ECS needs to fetch all configuration parameters associated with the `billing` module in the `production` environment during application boot. The parameters are organized under `/app/prod/billing/db_host`, `/app/prod/billing/db_port`, and `/app/prod/billing/api_key` (an encrypted `SecureString`). The application must retrieve all keys and their decrypted values in a single API operation.

**Which AWS Systems Manager CLI command and parameters achieves this?**
- A. `aws ssm get-parameters-by-path --path "/app/prod/billing/" --recursive --with-decryption`
- B. `aws ssm get-parameter --name "/app/prod/billing/*" --with-decryption`
- C. `aws ssm describe-parameters --parameter-filters "Key=Name,Option=BeginsWith,Values=/app/prod/billing/"`
- D. `aws ssm get-parameters --names "/app/prod/billing"`

---

### Question 5: Event-Driven Change Notifications for Parameter Store Modifications

A financial compliance control mandates that whenever any application configuration key under the namespace `/production/compliance/*` is created, updated, or deleted in AWS Systems Manager Parameter Store, a security alert must be dispatched to an Amazon SNS topic, and an automated compliance verification audit must execute immediately.

**How should this change detection architecture be configured?**
- A. Create an **Amazon EventBridge Rule** matching `source: aws.ssm`, `detail-type: Parameter Store Change`, and filtering `detail.name` with prefix `/production/compliance/`, targeting an Amazon SNS topic and an AWS Lambda function.
- B. Configure Amazon CloudWatch Logs metric filters on the AWS CloudTrail log group to match `PutParameter` and `DeleteParameter` API calls, setting up an alarm on metric increments.
- C. Schedule an AWS Lambda function via EventBridge Scheduler running every 60 seconds that executes `ssm:GetParametersByPath` and compares hashes against an Amazon DynamoDB state table.
- D. Enable S3 Server Access Logging on the internal Parameter Store storage bucket and configure an S3 Event Notification to publish alerts.

---

### Question 6: Parameter Store Versioning Mechanics and Zero-Downtime Rollbacks

An engineer accidentally updates an application database connection pool parameter in Parameter Store from `50` to `5000`, causing target databases to exhaust connection limits. The operations team must immediately revert the parameter to its previous stable value without losing historical version audit trails.

**How does Systems Manager Parameter Store manage parameter versions and history?**
- A. Parameter Store **automatically increments the parameter version number** on every write operation; past configurations can be inspected using `GetParameterHistory`, and reverting is achieved by updating the parameter value with the contents of the prior version (creating a new incremented version).
- B. Parameter Store maintains only the currently active parameter value; reverting an accidental update requires restoring the parameter from an automated Amazon EBS volume snapshot.
- C. Parameter Store allows administrators to execute an in-place version pointer swap using `aws ssm set-active-version --name parameter-name --version 1`.
- D. Parameter Store integrates with AWS CodeCommit; rolling back a parameter requires executing a `git revert` command against the underlying repository.

---

### Question 7: Mitigating API Throttling at Scale with Parameter Store Higher Throughput

A serverless fleet consisting of 2,500 concurrently executing AWS Lambda functions each calls `ssm:GetParameter` during cold starts to retrieve global feature flags. During morning peak traffic spikes, hundreds of Lambda invocations fail with `ThrottlingException` (`Rate exceeded`). An inspection shows the account is using default Parameter Store settings.

**What architectural adjustment eliminates this API throttling bottleneck with the LEAST latency?**
- A. Enable **Higher Throughput** in the Systems Manager Parameter Store settings (increasing the account transaction limit from 40 TPS up to 10,000 TPS) AND implement in-memory client-side caching (or use the AWS Parameters and Secrets Lambda Extension) to cache values across warm container invocations.
- B. Migrate the configuration parameters into an Amazon DynamoDB table configured with On-Demand capacity mode, querying the table on every Lambda invocation.
- C. Deploy an Amazon ElastiCache Redis cluster inside the VPC and configure Lambda functions to read parameters exclusively from Redis.
- D. Add exponential backoff and jitter algorithms to the Lambda application code without altering Parameter Store account throughput settings.

---

### Question 8: Enforcing Parameter Lifecycles with Parameter Policies

A security team issues temporary database maintenance credentials that must automatically expire and be deleted from Systems Manager Parameter Store exactly 8 hours after creation. Additionally, the team requires an automated notification to be published to an Amazon SNS topic 15 minutes before the parameter expires so on-call engineers can extend the credential window if maintenance is ongoing.

**Which Systems Manager Parameter Store feature satisfies these operational lifecycle requirements?**
- A. Create the parameter in the **Advanced tier** and attach a **Parameter Policy** specifying an `Expiration` policy (setting the UTC timestamp or duration for deletion) alongside an `ExpirationNotification` policy that emits an EventBridge event 15 minutes prior to expiration.
- B. Create a standard Parameter Store parameter and configure an AWS Lambda function triggered by Amazon EventBridge Scheduler to delete the parameter after 8 hours.
- C. Attach an IAM condition to the parameter ARN evaluating `aws:CurrentTime`, which revokes read permissions 8 hours after creation.
- D. Deploy an AWS Config custom rule that evaluates parameter age continuously and triggers an SSM Automation runbook to delete parameters older than 8 hours.

---

### Question 9: Architectural Decision Framework: Parameter Store vs. AWS Secrets Manager

An enterprise architectural board is defining technical standards for configuration management. The board needs to establish formal guidelines determining when to use **AWS Systems Manager Parameter Store** versus **AWS Secrets Manager**.

**Which architectural comparison accurately reflects the capabilities and cost profiles of these two services?**
- A. Use **Parameter Store** for hierarchical configuration data, environment variables, and non-rotating secrets (offering free Standard tier storage and pay-per-request Advanced tier); use **Secrets Manager** when workloads mandate native automated rotation (e.g. RDS, Redshift), automatic cross-account resource policies, or cross-region secret replication.
- B. Use Parameter Store exclusively for unencrypted data, and use Secrets Manager for all encrypted data, because Parameter Store does not support AWS KMS Customer Managed Keys.
- C. Use Secrets Manager for high-volume configuration retrieval because it is free of charge, whereas Parameter Store incurs charges per API request.
- D. Parameter Store supports automatic password rotation for relational databases natively without Lambda, whereas Secrets Manager requires custom shell scripts.

---

## Answer Key & Detailed Explanations

### Question 1: Systems Manager Parameter Store Tiering Limits & Parameter Policies
**Answer:** A  
**Explanation:**  
AWS Systems Manager Parameter Store provides two tiers:  
1. **Standard Tier**: Max size 4 KB per parameter, max 10,000 parameters per account/region, free of charge for standard throughput, zero parameter policy support.  
2. **Advanced Tier**: Max size 8 KB per parameter, up to 100,000 parameters per account/region, billed per parameter-month and per API interaction, and supports **Parameter Policies** (such as automated expiration TTLs and expiration notifications). Storing a 6.5 KB payload strictly requires the Advanced tier.

---

### Question 2: CloudFormation Dynamic Resolution of Encrypted SecureString Parameters
**Answer:** A  
**Explanation:**  
AWS CloudFormation supports dynamic references for secrets and encrypted parameters using the pattern `{{resolve:ssm-secure:parameter-name:version}}`. Using `ssm-secure` instructs CloudFormation to decrypt the parameter value using AWS KMS during stack operations without printing the secret in plaintext in templates, change sets, or stack event logs. Standard `{{resolve:ssm:...}}` references do not support `SecureString` types.

---

### Question 3: Multi-Account Parameter Sharing Architecture with AWS Resource Access Manager (RAM)
**Answer:** A  
**Explanation:**  
AWS Systems Manager Parameter Store integrates with **AWS Resource Access Manager (AWS RAM)**. To share parameters across accounts, parameters must be in the **Advanced tier**. The administrator creates a resource share in AWS RAM, specifies the parameter ARNs, and shares them with the entire AWS Organization or specific Organizational Units. Member accounts access the shared parameters directly via their ARNs without copying data.

---

### Question 4: Recursive Hierarchical Parameter Retrieval and Path Traversal
**Answer:** A  
**Explanation:**  
The AWS CLI command `aws ssm get-parameters-by-path` allows retrieving all parameters under a specified hierarchical path. Specifying `--recursive` ensures all nested child paths (e.g. `/app/prod/billing/*`) are returned in a single batch API call. Specifying `--with-decryption` automatically decrypts any `SecureString` parameters in the hierarchy before returning the response.

---

### Question 5: Event-Driven Change Notifications for Parameter Store Modifications
**Answer:** A  
**Explanation:**  
Amazon EventBridge natively captures Parameter Store operations under `source: aws.ssm` with `detail-type: Parameter Store Change`. The event payload contains `name` (the parameter path) and `operation` (`Create`, `Update`, `Delete`). An EventBridge rule matching these criteria triggers downstream targets (such as SNS topics and Lambda functions) in real time without polling.

---

### Question 6: Parameter Store Versioning Mechanics and Zero-Downtime Rollbacks
**Answer:** A  
**Explanation:**  
Parameter Store maintains an immutable version history for every parameter. Each time a parameter value or attribute is modified, Parameter Store automatically increments the version number (`1`, `2`, `3`...). While historical versions cannot be "activated" directly as pointers, engineers can inspect past values using `GetParameterHistory` and submit a `PutParameter` request containing the previous value, safely creating a new version with the restored configuration.

---

### Question 7: Mitigating API Throttling at Scale with Parameter Store Higher Throughput
**Answer:** A  
**Explanation:**  
By default, Parameter Store enforces a regional limit of 40 transactions per second (TPS) on `GetParameter` / `GetParameters`. Enabling **Higher Throughput** raises this limit up to 10,000 TPS (incurring a nominal charge per 10,000 requests). Combining Higher Throughput with client-side in-memory caching (such as the AWS Parameters and Secrets Lambda Extension) drastically reduces API call volume during traffic spikes.

---

### Question 8: Enforcing Parameter Lifecycles with Parameter Policies
**Answer:** A  
**Explanation:**  
Advanced tier parameters support **Parameter Policies**. An `Expiration` policy specifies a timestamp or duration after which Parameter Store automatically deletes the parameter. An `ExpirationNotification` policy emits an event to Amazon EventBridge at a specified threshold prior to deletion (e.g. 15 minutes before expiration), enabling automated alerts or renewal workflows without external cron jobs.

---

### Question 9: Architectural Decision Framework: Parameter Store vs. AWS Secrets Manager
**Answer:** A  
**Explanation:**  
**Parameter Store** is optimized for general configuration settings, hierarchical strings, and static application parameters (with free Standard tier storage). **Secrets Manager** is purpose-built for secrets lifecycle governance: it provides built-in native rotation using Lambda functions (with out-of-the-box support for RDS, Aurora, and Redshift), cross-account resource-based access policies, and automated cross-region secret replication.
