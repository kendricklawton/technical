# Domain 5 (Incident and Event Response) — Quiz 2: Enterprise EventBridge Patterns

**Topic:** Cross-Account / Cross-Region Event Routing, Archive & Replay, EventBridge Pipes & Scheduler  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Cross-Account Multi-Region Event Ingestion with Organization Condition

An enterprise with 100 AWS member accounts operates a centralized Security Operations event bus named `sec-ops-bus` in a dedicated Security Account (`555566667777`) in `us-east-1`. All member accounts must route critical security events to this central bus. The security team mandates that:
1. Any account belonging to the AWS Organization must be permitted to publish events without modifying the bus resource policy for every new account.
2. Non-organization accounts must be strictly denied.
3. Spoke accounts in `eu-west-1` and `ap-southeast-1` must route events to `us-east-1` seamlessly.

**Which EventBridge architecture fulfills these multi-account and multi-region routing requirements?**
- A. Attach a Resource Policy to `sec-ops-bus` granting `events:PutEvents` to principal `*` with a condition `StringEquals: { "aws:PrincipalOrgID": "o-exampleorg123" }`; in each spoke account region, create an event rule on the default bus matching security events with a target pointing directly to the `sec-ops-bus` ARN in `us-east-1`.
- B. Share the `sec-ops-bus` across the AWS Organization using AWS Resource Access Manager (RAM), allowing spoke accounts to publish events directly to the local RAM share.
- C. Create an IAM role in the Security Account granting `events:PutEvents` and configure spoke accounts to assume this role via AWS STS before publishing.
- D. Deploy Amazon SQS queues in each spoke account, replicating messages across regions using AWS DataSync into a centralized SQS queue subscribed to EventBridge.

---

### Question 2: Point-to-Point Streaming Integration via EventBridge Pipes with Enrichment

A payments microservice writes transaction records to an Amazon DynamoDB table with DynamoDB Streams enabled (`NEW_AND_OLD_IMAGES`). Downstream fraud detection systems require real-time notifications, but only for transactions where `amount > 5000`. Furthermore, before the event reaches the fraud system's Amazon SQS queue, it must be enriched with external geolocation metadata queried from a legacy REST API.

**Which architecture satisfies these filtering, enrichment, and delivery requirements with the LEAST custom code?**
- A. Deploy an **Amazon EventBridge Pipe**: Configure DynamoDB Streams as the **Source**, specify an event filter pattern matching `{"dynamodb": {"NewImage": {"amount": {"N": [{"numeric": [">", 5000]}]}}}}`, configure an **Enrichment** step targeting an AWS Lambda function that queries the legacy API, and set the **Target** to the fraud SQS queue.
- B. Write a custom AWS Lambda function triggered by DynamoDB Streams that parses records, executes filtering in Python, calls the legacy API, and writes to SQS.
- C. Create an Amazon Kinesis Data Firehose delivery stream connected to DynamoDB Streams with inline AWS Lambda data transformation.
- D. Configure an Amazon EventBridge Rule on the default bus with an input transformer passing data to an Amazon SNS topic.

---

### Question 3: Non-Disruptive Failure Recovery via EventBridge Archive and Replay

A major software regression in an inventory accounting microservice caused it to drop incoming `OrderPlaced` events for 6 hours between 08:00 UTC and 14:00 UTC. The bug has been patched and deployed. The platform architect needs to redrive the exact historical `OrderPlaced` events that occurred during those 6 hours to the inventory service without republishing events to other active consumers (such as Billing, Shipping, and Email Notifications) and without requiring upstream order producers to re-emit events.

**Which EventBridge feature achieves this targeted historical event redrive?**
- A. Create an **EventBridge Replay**: Specify the event bus archive as the source, set the start time to `08:00 UTC` and end time to `14:00 UTC`, and configure the replay target to match only the specific EventBridge Rule associated with the inventory service.
- B. Trigger an AWS Step Functions execution that queries CloudTrail Lake for `PutEvents` calls and replays payloads to the inventory service SQS queue.
- C. Restore the EventBridge default event bus from an automated AWS Backup recovery point to a staging event bus.
- D. Instruct the order processing database to execute a rollback transaction and replay the write-ahead log.

---

### Question 4: Dispersing Load Spikes via EventBridge Scheduler Flexible Time Windows

An enterprise runs automated compliance audit jobs across 20,000 Amazon EC2 instances every night. If all 20,000 jobs execute at precisely 00:00 UTC, the backend audit API suffers severe connection exhaustion and throttles requests. The compliance team mandates that each instance must run its audit job once per night between 00:00 UTC and 00:30 UTC, dispersing the execution load evenly across that 30-minute window.

**Which scheduling mechanism fulfills this requirement with the LEAST operational overhead?**
- A. Configure an **Amazon EventBridge Scheduler** schedule with a cron expression (`cron(0 0 * * ? *)`) and enable a **Flexible Time Window** with `Mode: "FLEXIBLE"` and `MaximumWindowInMinutes: 30`.
- B. Create a standard EventBridge Rule with a cron expression and deploy an intermediate SQS queue with a message delay of 1,800 seconds.
- C. Write an AWS Step Functions state machine that generates a random sleep interval between 0 and 1,800 seconds before executing each job.
- D. Configure AWS Systems Manager State Manager associations with an EC2 rate control setting of 5% concurrency.

---

### Question 5: Dead-Letter Queues (DLQ) for Asynchronous EventBridge Rule Delivery Failures

An EventBridge rule matches critical invoice payment events and routes them to an AWS Lambda function. During an unexpected traffic surge, the Lambda function exceeds its account-level reserved concurrency limit, returning `TooManyRequestsException` throttling errors. After exhausting EventBridge's standard 24-hour retry duration and maximum retry attempts, events are permanently discarded.

**How should the EventBridge rule be configured to guarantee zero event loss during target delivery failures?**
- A. Configure a **Dead-Letter Queue (DLQ)** on the EventBridge Rule's target configuration pointing to a standard Amazon SQS queue, and attach an SQS queue policy permitting the EventBridge service principal to send messages.
- B. Configure a Dead-Letter Queue on the target AWS Lambda function's asynchronous invocation configuration.
- C. Increase the EventBridge event bus retention period to 14 days and configure CloudWatch Alarms on throttled events.
- D. Enable EventBridge Schema Discovery to validate that incoming event payloads match the Lambda function input schema.

---

### Question 6: Secure Third-Party Webhook Delivery via EventBridge API Destinations

A DevOps team needs to send real-time deployment status events from AWS CodePipeline to an external third-party project management SaaS platform via an HTTPS webhook (`https://api.thirdparty.com/v1/events`). The third-party API requires OAuth 2.0 client credentials authentication and enforces a strict rate limit of 50 requests per second.

**Which serverless architecture satisfies this requirement without deploying custom proxy compute instances?**
- A. Configure an **EventBridge API Destination**: Create an EventBridge **Connection** managing the OAuth 2.0 client credentials via AWS Secrets Manager, define the API Destination with the endpoint URL and an invocation rate limit of 50 requests per second, and attach it as the target of an EventBridge Rule.
- B. Deploy an AWS Lambda function that fetches OAuth tokens from Secrets Manager, throttles calls using an in-memory token bucket algorithm, and makes HTTP requests.
- C. Create an Amazon API Gateway HTTP API configured with an integration pointing to the third-party URL and attach an AWS WAF rate-limiting rule.
- D. Stream events to an Amazon Kinesis Data Firehose delivery stream configured with a custom HTTP endpoint destination.

---

### Question 7: EventBridge Advanced Pattern Matching: Numeric Ranges and Prefix Matching

An IoT device telemetry pipeline receives sensor data on an EventBridge custom event bus. An SRE needs to route alerts to an emergency SNS topic ONLY when:
- The device location prefix starts with `factory-us-`
- The sensor temperature reading is strictly greater than 85.5 degrees
- The event source is `corp.iot.sensors`

**Which EventBridge Rule event pattern syntax correctly isolates these specific events?**
- A.
```json
{
  "source": ["corp.iot.sensors"],
  "detail": {
    "location": [{"prefix": "factory-us-"}],
    "temperature": [{"numeric": [">", 85.5]}]
  }
}
```
- B.
```json
{
  "source": "corp.iot.sensors",
  "detail": {
    "location": "factory-us-*",
    "temperature": "> 85.5"
  }
}
```
- C.
```json
{
  "source": ["corp.iot.sensors"],
  "detail": {
    "location": [{"regex": "^factory-us-"}],
    "temperature": [{"value": "> 85.5"}]
  }
}
```
- D.
```json
{
  "source": ["corp.iot.sensors"],
  "filter": "location LIKE 'factory-us-%' AND temperature > 85.5"
}
```

---

### Question 8: Safe Testing in CI/CD via EventBridge Schema Registry & Code Bindings

A development organization with 30 microservice teams relies heavily on EventBridge for inter-service communication. Teams frequently introduce breaking changes by modifying event payload schemas without notifying downstream consumers, leading to deserialization crashes in production.

**How should the platform team integrate EventBridge with developer CI/CD workflows to prevent schema drift and breaking contract changes?**
- A. Enable **EventBridge Schema Discovery** on development event buses, publish approved schemas to the **EventBridge Schema Registry**, and generate strongly typed **Code Bindings** (Java/TypeScript/Python) integrated into developer build pipelines to enforce contract validation during compilation.
- B. Deploy an AWS Config rule that validates JSON payloads against OpenAPI specifications before allowing events to be published.
- C. Configure CloudWatch Metric Filters that trigger alarms if unparsed JSON strings appear in application logs.
- D. Require all microservices to serialize events into Apache Avro binary formats using custom Lambda pre-processors.

---

### Question 9: Multi-Region Event Bus Ingress via Global Event Routing

A multinational banking platform operates workloads in `us-east-1` and `eu-west-1`. Applications in both regions emit transaction events that must be processed locally when healthy, but automatically fail over to the secondary region if the primary region's event bus or ingestion endpoints experience degraded health.

**Which AWS EventBridge feature provides managed multi-region event failover?**
- A. Deploy an **EventBridge Global Endpoint**: Provides a single global endpoint hostname that routes incoming events to the primary region's event bus, automatically failing over to the secondary region's event bus when Route 53 health checks detect regional impairment.
- B. Create standard Route 53 Geolocation DNS records pointing to the public EventBridge service endpoints in each region.
- C. Configure Amazon CloudFront with an Origin Group pointing to regional EventBridge VPC Endpoints.
- D. Deploy an AWS Transit Gateway multi-region peering mesh connecting regional event buses with static routes.

---

## Answer Key & Explanations

### Question 1: Cross-Account Multi-Region Event Ingestion with Organization Condition
**Answer:** A  
**Explanation:** 
For scalable cross-account event routing into a centralized event bus:
1. **Central Bus Resource Policy:** Attaching a resource-based policy to `sec-ops-bus` that grants `events:PutEvents` to `Principal: "*"` with the condition `"StringEquals": { "aws:PrincipalOrgID": "o-exampleorg123" }` allows any account in the AWS Organization to publish events. When new member accounts are added to the organization, they immediately inherit access with zero bus policy updates.
2. **Cross-Region / Cross-Account Routing:** In spoke accounts across different regions, an EventBridge rule on the local default bus matches security events and specifies the central bus ARN (`arn:aws:events:us-east-1:555566667777:event-bus/sec-ops-bus`) as the target. EventBridge natively handles cross-account and cross-region event delivery over the secure AWS internal backbone.
RAM (Option B) does not support sharing EventBridge event buses.

---

### Question 2: Point-to-Point Streaming Integration via EventBridge Pipes with Enrichment
**Answer:** A  
**Explanation:** 
**Amazon EventBridge Pipes** provides a serverless point-to-point integration framework that eliminates glue code:
- **Source:** Pulls events from streaming sources (DynamoDB Streams, Kinesis) or message queues (SQS).
- **Filtering:** Filters records before processing using native EventBridge pattern syntax (e.g., matching only `amount > 5000`). Filtered-out records do not invoke downstream compute or enrichment steps, saving cost.
- **Enrichment:** Invokes an intermediate step (AWS Lambda, Step Functions, or API Gateway) to enrich, transform, or enhance the event payload with external data (such as querying a legacy geolocation REST API).
- **Target:** Delivers the enriched event directly to the destination SQS queue.
This fulfills the requirement natively without managing custom consumer fleets or long-running worker code.

---

### Question 3: Non-Disruptive Failure Recovery via EventBridge Archive and Replay
**Answer:** A  
**Explanation:** 
**EventBridge Archive & Replay** provides native disaster recovery and operational remediation for event-driven architectures:
- An **Archive** records all events published to an event bus and retains them for a configurable retention window.
- When a consumer service suffers an outage or bug, you initiate a **Replay** specifying the archive, the exact time window (08:00 to 14:00 UTC), and the **specific EventBridge Rule** associated with the impacted consumer.
- EventBridge replays the historical events *only* to the targeted rule, ensuring the patched inventory service receives the missing transactions while active consumers (Billing, Shipping, Email) are completely unaffected and do not receive duplicate events.

---

### Question 4: Dispersing Load Spikes via EventBridge Scheduler Flexible Time Windows
**Answer:** A  
**Explanation:** 
When executing large numbers of concurrent scheduled jobs (such as 20,000 nightly compliance tasks), firing all jobs simultaneously causes the "thundering herd" problem, overwhelming downstream APIs and databases.

**Amazon EventBridge Scheduler** solves this with **Flexible Time Windows**:
- Setting `Mode: "FLEXIBLE"` with a `MaximumWindowInMinutes` (e.g., 30 minutes) instructs the scheduler to execute the scheduled task at a randomized point within that time window.
- For a cron schedule starting at 00:00 UTC, the 20,000 tasks are automatically distributed smoothly between 00:00 and 00:30 UTC.
- This disperses API request rates evenly, eliminating throttling and connection exhaustion with zero custom sleep or jitter logic.

---

### Question 5: Dead-Letter Queues (DLQ) for Asynchronous EventBridge Rule Delivery Failures
**Answer:** A  
**Explanation:** 
When EventBridge attempts to deliver an event to a target (such as an AWS Lambda function) and the target is throttled or fails:
- EventBridge automatically retries delivery for up to 24 hours with exponential backoff and jitter.
- If the target remains unavailable after exhausting maximum retries or maximum event age, EventBridge drops the event unless a **Dead-Letter Queue (DLQ)** is configured on the **Rule Target**.
- Setting an SQS queue as the target's DLQ ensures that failed events are preserved along with error metadata (error code, reason, and timestamp), preventing data loss and enabling automated dead-letter redrive once the Lambda concurrency issue is resolved.

---

### Question 6: Secure Third-Party Webhook Delivery via EventBridge API Destinations
**Answer:** A  
**Explanation:** 
**EventBridge API Destinations** enables direct HTTP webhook delivery from EventBridge to external endpoints:
- **Connection:** Securely manages authentication credentials (Basic, API Key, or OAuth 2.0 client credentials grant). OAuth client secrets are automatically stored and retrieved using AWS Secrets Manager.
- **API Destination:** Configures the external HTTPS URL (`https://api.thirdparty.com/v1/events`) and defines an **Invocation Rate Limit** (e.g., 50 requests/second). EventBridge buffers incoming events and delivers them smoothly without exceeding the third-party endpoint's rate limit.
- This provides an enterprise-grade, serverless integration without writing custom token-refresh or throttling code.

---

### Question 7: EventBridge Advanced Pattern Matching: Numeric Ranges and Prefix Matching
**Answer:** A  
**Explanation:** 
EventBridge event patterns use structured JSON syntax for advanced matching:
- **Prefix Matching:** Matches strings beginning with a specific prefix: `[{"prefix": "factory-us-"}]`. Wildcards like `*` (Option B) are not supported in standard EventBridge string patterns.
- **Numeric Matching:** Supports mathematical comparisons (`>`, `<`, `>=`, `<=`, `=`, `range`): `[{"numeric": [">", 85.5]}]`.
- Option A correctly declares both the prefix matcher and the numeric comparison inside an array format conforming to the official EventBridge rule pattern specification.

---

### Question 8: Safe Testing in CI/CD via EventBridge Schema Registry & Code Bindings
**Answer:** A  
**Explanation:** 
**EventBridge Schema Registry and Code Bindings** bridge event-driven architecture with software engineering discipline:
- **Schema Discovery:** Automatically inspects events traversing the event bus and maps their structure.
- **Schema Registry:** Stores version-controlled OpenAPI/JSON schema definitions of approved events.
- **Code Bindings:** Generates strongly typed data transfer objects (DTOs) for Java, TypeScript, or Python.
- Integrating these bindings into developer CI/CD pipelines ensures that any unauthorized schema changes cause compilation or unit test failures before code can be merged, eliminating breaking runtime contract errors across microservices.

---

### Question 9: Multi-Region Event Bus Ingress via Global Event Routing
**Answer:** A  
**Explanation:** 
**EventBridge Global Endpoints** provides high-availability multi-region event ingestion:
- Provides a global DNS endpoint backed by Amazon Route 53.
- Applications publish events to this single global hostname.
- Under normal operations, events are routed to the primary region's event bus.
- If the primary region experiences service degradation, Route 53 health checks detect the impairment and Global Endpoints automatically fails over event ingestion to the secondary region's event bus within minutes, ensuring continuous event publishing availability without application reconfiguration.
