# AWS DevOps Engineer - Professional (DOP-C02) — Calibration Set

Five questions, one for each common governing constraint: fastest rollback, most cost-effective, fastest recovery, least operational overhead, and least privilege. Use this set to calibrate before you attempt the full practice exams.

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 5 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 25 minutes  
**Passing Target:** 4 / 5

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Answer all five before opening the answer key at the end.

---

## Question 1 (Domain 1: SDLC Automation)

A company's REST API on Amazon API Gateway invokes Lambda functions through stage variables that reference function aliases. Releases often change both the API configuration (mapping templates and integration settings) and the function code. The company needs to canary each release to 10% of production traffic and roll back immediately if errors increase.

The `prod` stage is fronted by a custom domain name, clients cache DNS for up to an hour, and the Lambda functions use published versions.

**Which solution meets these requirements with the FASTEST rollback?**

- A. Deploy each release as a canary on the `prod` stage with 10% of traffic and a stage variable override that points to the new alias version, and roll back by deleting the canary settings.
- B. Deploy each release to a separate `prod-v2` stage behind its own custom domain, and use Route 53 weighted records to send 10% of traffic to it, rolling back by setting its weight to zero.
- C. Deploy each release to a second API, put a CloudFront distribution with continuous deployment and a 10% weight in front of both APIs, and roll back by disabling the continuous deployment policy.
- D. Route 10% of clients to a separate `canary` stage through a client-side feature flag, and roll back by turning the flag off and waiting for clients to refresh their cached flag values.

---

## Question 2 (Domain 2: Configuration Management and IaC)

Lambda functions that handle 3,000 requests per second each call `GetParameter` for the same 20 configuration values on every invocation. During peaks, Parameter Store throttles the calls and requests fail. The values change about once a week, and functions can tolerate values up to 5 minutes old.

The parameters are in the Standard tier, and the functions run in a VPC with an interface endpoint for Systems Manager.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Turn on Parameter Store's higher throughput setting for the account and Region so that the functions can keep calling `GetParameter` on every invocation at peak load without being throttled.
- B. Move the 20 values into AWS Secrets Manager secrets and retrieve them through the Secrets Manager API on each invocation instead of calling Parameter Store.
- C. Add the AWS Parameters and Secrets Lambda Extension with a 300-second cache TTL, and read the values through its local endpoint instead of calling `GetParameter` directly.
- D. Move the values into an AWS AppConfig configuration profile and retrieve them through the AppConfig Agent Lambda extension with a 5-minute poll interval.

---

## Question 3 (Domain 3: Resilient Cloud Solutions)

A company runs an Aurora MySQL global database with its primary cluster in us-east-1 and a secondary cluster in us-west-2. Applications in both Regions connect to the primary cluster's writer endpoint by name. In a Regional failover test, the database switchover completed in about a minute, but writes failed for 25 minutes while teams updated connection strings and redeployed applications.

The company wants writes to resume as quickly as possible after any future global database failover or switchover, without redeploying applications.

**Which solution meets these requirements with the FASTEST recovery?**

- A. Create a Route 53 private hosted zone record for the writer, and have an EventBridge rule on RDS failover events invoke a Lambda function that updates the record to the new primary's writer endpoint with a 60-second TTL.
- B. Store the writer endpoint in Parameter Store, have applications re-read it every minute, and update the parameter with an SSM Automation runbook as the final step of the failover procedure.
- C. During a failover, detach the us-west-2 cluster from the global database and promote it to a standalone cluster, then update applications' connection strings through configuration management.
- D. Point applications at the global database's writer endpoint, which follows the primary cluster after a managed failover or switchover, so applications reconnect without any configuration change.

---

## Question 4 (Domain 4: Monitoring and Logging)

A company must send every CloudWatch Logs log group in 25 accounts — including log groups created later by Lambda functions, ECS tasks, and AWS services — to a central S3 bucket in a log archive account within minutes of ingestion. Teams create about 300 new log groups each week.

Each account already has an Amazon Data Firehose delivery stream that delivers to the central bucket. The company uses service-managed StackSets.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Deploy a StackSet that adds a subscription filter to every existing log group, plus an EventBridge rule on `CreateLogGroup` events that invokes a Lambda function to add the filter to each new log group.
- B. Use a StackSet to create an account-level subscription filter policy in each account that sends every log group, including future ones, to that account's existing Firehose delivery stream.
- C. Run a Lambda function every 5 minutes in each account that lists the log groups and adds a subscription filter to the Firehose delivery stream for any log group that doesn't have one yet.
- D. Configure each Lambda function, ECS task definition, and AWS service integration to log directly to Firehose instead of CloudWatch Logs, using log routers where direct delivery isn't supported.

---

## Question 5 (Domain 6: Security and Compliance)

An application role, `reports-app`, writes and reads objects in the S3 bucket `acme-reports` using SSE-KMS with a customer managed key, `reports-key`, that is dedicated to that bucket. S3 Bucket Keys are enabled on the bucket. The security team wants the key policy to let `reports-app` use the key only through S3 and only for this bucket, even if other buckets are later configured to use the same key.

The key policy already gives the account's key administrators full administrative access.

**Which solution meets these requirements with the LEAST privilege?**

- A. Grant `reports-app` `kms:*` on `reports-key` in the key policy, and rely on the role's IAM policy to limit its S3 access to the `acme-reports` bucket.
- B. Grant the account root principal `kms:GenerateDataKey` and `kms:Decrypt` on `reports-key`, and grant those actions to `reports-app` in its IAM policy with a `kms:ViaService` condition for S3.
- C. Grant `reports-app` `kms:GenerateDataKey` and `kms:Decrypt` on `reports-key` with a `kms:ViaService` condition for Amazon S3 in the bucket's Region.
- D. Grant `reports-app` `kms:GenerateDataKey` and `kms:Decrypt` with a `kms:ViaService` condition for S3 and a `kms:EncryptionContext:aws:s3:arn` condition equal to the bucket ARN.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | A | Domain 1: SDLC Automation | FASTEST rollback |
| 2 | C | Domain 2: Configuration Management and IaC | MOST cost-effective |
| 3 | D | Domain 3: Resilient Cloud Solutions | FASTEST recovery |
| 4 | B | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 5 | D | Domain 6: Security and Compliance | LEAST privilege |

---

### Question 1

**Answer:** A  
**Constraint:** FASTEST rollback · **Services:** Amazon API Gateway (canary release), AWS Lambda, Amazon CloudWatch

**Socratic walkthrough**
1. *What must the canary cover?* — Both the API configuration and the function code.
2. *Which mechanisms depend on caches outside your control?* — DNS weighting (an hour of client caching) and client-side flags.
3. *Which rollback is a single stage update that takes effect at once?* — Deleting an API Gateway stage canary.

**Why A is correct:** A stage canary deployment routes a percentage of requests to a new deployment of the API — including mapping templates and integrations — and its stage variable override points the canary at the new function alias. Deleting the canary settings sends 100% of traffic back to the base deployment immediately.

**Why the others fail the constraint:**
- **B.** Works, but clients keep resolving to `prod-v2` for up to an hour after rollback.
- **C.** Works, but rollback is a distribution configuration change that must propagate.
- **D.** Works, but rollback waits for every client to refresh its flag cache.

**Exam reflex:** To canary API Gateway configuration and code together, use stage canary settings; rollback is deleting the canary.

---

### Question 2

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** AWS Systems Manager Parameter Store, AWS Lambda (Parameters and Secrets extension), Amazon CloudWatch

**Socratic walkthrough**
1. *Why is there throttling?* — Every invocation calls the API, so request volume tracks traffic.
2. *Do the values need to be fetched on every request?* — No. They change weekly, and 5-minute staleness is fine.
3. *Which option removes most of the API calls without adding a new paid service?* — Caching with the extension.

**Why C is correct:** The AWS Parameters and Secrets Lambda Extension caches values for each execution environment for the configured TTL, which collapses thousands of requests per second into occasional refreshes. Standard-tier parameters at standard throughput have no added charges, so throttling and cost both drop.

**Why the others fail the constraint:**
- **A.** Works, but higher throughput is billed per API interaction at this volume.
- **B.** Works, but Secrets Manager bills per secret and per API call.
- **D.** Works, but AppConfig adds per-request and per-configuration-received charges that cached Parameter Store doesn't have.

**Exam reflex:** To fix hot-path parameter throttling, cache with the Lambda extension before paying for throughput.

---

### Question 3

**Answer:** D  
**Constraint:** FASTEST recovery · **Services:** Amazon Aurora Global Database (writer endpoint), Amazon Route 53, AWS Lambda

**Socratic walkthrough**
1. *What actually took 25 minutes?* — Reconfiguring and redeploying applications, not the database.
2. *Does Aurora offer an endpoint that moves with the primary?* — Yes. The global database writer endpoint.
3. *Which options still depend on DNS TTLs, polling intervals, or manual steps?* — A, B, and C.

**Why D is correct:** The global writer endpoint always resolves to the current primary cluster's writer. After a managed failover or switchover it follows the new primary, so applications reconnect without configuration changes or redeployments.

**Why the others fail the constraint:**
- **A.** Works, but adds event detection, Lambda execution, and a 60-second TTL to every recovery.
- **B.** Works, but adds a final manual step plus a one-minute polling delay.
- **C.** Works, but detach-and-promote breaks the global topology and still needs connection string updates.

**Exam reflex:** For Aurora global database failover without app changes, use the global writer endpoint.

---

### Question 4

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** Amazon CloudWatch Logs (account-level subscription filter policy), Amazon Data Firehose, Amazon S3

**Socratic walkthrough**
1. *What makes the per-log-group approach expensive to operate?* — About 300 new log groups a week, each needing a filter.
2. *Can CloudWatch Logs apply a subscription to every log group in an account automatically?* — Yes. An account-level subscription filter policy.
3. *Does it cover future log groups?* — Yes.

**Why B is correct:** One account-level subscription filter policy per account — deployed by a StackSet — streams all current and future log groups to the existing Firehose stream. There are no functions, events, or per-group work.

**Why the others fail the constraint:**
- **A.** Works, but custom functions must catch every log group creation.
- **C.** Works, but it's a polling function with gaps of up to 5 minutes.
- **D.** Works, but reconfigures every workload and service — the most effort by far.

**Exam reflex:** To stream all log groups in an account, use an account-level subscription filter policy.

---

### Question 5

**Answer:** D  
**Constraint:** LEAST privilege · **Services:** AWS KMS (key policy conditions), Amazon S3 (SSE-KMS with S3 Bucket Keys), AWS IAM

**Socratic walkthrough**
1. *Which principal should the key policy name?* — The `reports-app` role, not the account root.
2. *How do you require that the key is used only through S3?* — With `kms:ViaService` for S3.
3. *How do you tie use to one bucket when S3 Bucket Keys are enabled?* — The encryption context is the bucket ARN, so add a `kms:EncryptionContext:aws:s3:arn` condition.

**Why D is correct:** D narrows the principal, the actions, the calling service, and the bucket. Even if another bucket later uses `reports-key`, `reports-app` can't use the key for that bucket's objects.

**Why the others fail the constraint:**
- **A.** Works, but `kms:*` includes direct and administrative key actions.
- **B.** Works, but the root principal delegates key use to any identity the account grants.
- **C.** Works, but without the encryption-context condition, the role can use the key for any bucket encrypted with it.

**Exam reflex:** For least-privilege SSE-KMS, use the specific role principal, `kms:ViaService`, and an encryption-context condition on the bucket ARN (with S3 Bucket Keys).

---

## Scoring

| Score | Band | Next step |
|---|---|---|
| **5 / 5** | Calibrated | Start the full practice exams. |
| **4 / 5** | On track | Re-read the distractor breakdown for the miss, then start Practice Exam 1. |
| **≤ 3 / 5** | Recalibrate | For each miss, re-read the Socratic walkthrough and the study guide for that domain before starting the full exams. |
