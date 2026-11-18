# Domain 2 — Quiz 3: Enterprise Event-Driven GenAI Architectures, Streaming APIs, Tool Use, Resiliency & IaC (Tasks 2.3, 2.4, 2.5)

> **Track**: AWS Certified Generative AI Developer - Professional (AIP-C01)  
> **Scope**: Tasks 2.3, 2.4, and 2.5 — Event-Driven GenAI Workflows (EventBridge, SQS, Step Functions Distributed Map), Real-Time Token Streaming (AppSync, API Gateway REST Streaming, Lambda Response Streaming), AWS PrivateLink, `Converse` / `ConverseStream` Native Tool Use, Resiliency Engineering, IaC, CI/CD, and Amazon Q Developer.  
> **Total Questions**: 15 (12 Single-Response `A`–`D`, 3 Multiple-Response `Choose TWO` `A`–`E`)

---

### Question 1

An insurance provider processes incoming medical claim packages using an event-driven architecture where an Amazon S3 upload triggers an Amazon EventBridge rule that sends messages to an **Amazon SQS standard queue**. An **AWS Lambda** function consumes batches of 5 messages from the SQS queue and calls Amazon Bedrock `Converse` (`anthropic.claude-3-5-sonnet-20241022-v2:0`) to extract ICD-10 diagnosis codes. Each Lambda invocation takes up to **120 seconds** (2 minutes), and the Lambda function timeout is configured to `150 seconds`. The SQS queue is currently configured with the default `VisibilityTimeout` of `30 seconds` and no concurrency limit on the Event Source Mapping.

During Monday morning claim surges (3,000 claims uploaded in 2 minutes), the FinOps and engineering teams observe two severe failures:
1. The exact same claim is processed 3 to 4 times by different Lambda instances simultaneously, tripling the Amazon Bedrock token bill.
2. Lambda rapidly scales to 600 concurrent executions, causing 80% of Bedrock calls to fail with `ThrottlingException (HTTP 429)` and exhausting retry attempts on entire batches even when 4 out of 5 messages in a batch succeeded.

Which architecture configuration will resolve BOTH issues?

- **A.** Increase the SQS queue `VisibilityTimeout` to **at least `900 seconds`** (6x the `150s` Lambda timeout), configure `ScalingConfig={"MaximumConcurrency": 20}` on the Lambda SQS Event Source Mapping to match the Bedrock RPM quota, and enable `FunctionResponseTypes=["ReportBatchItemFailures"]` so only throttled messages are retried.
- **B.** Reduce the SQS queue `VisibilityTimeout` to `0 seconds`, increase the Lambda `BatchSize` to `10,000`, and disable the SQS Dead-Letter Queue (DLQ).
- **C.** Replace the Amazon SQS queue with synchronous Amazon API Gateway HTTP API (`v2`) calls that chain 5 Lambda functions sequentially.
- **D.** Increase the Lambda memory to `10,240 MB` while keeping `VisibilityTimeout` at `30 seconds` and `MaximumConcurrency` unlimited.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: A**

#### Architectural Explanation
1. **Preventing Duplicate LLM Invocations (`VisibilityTimeout >= 6x Lambda Timeout`)**: Because the Lambda function takes up to 120 seconds (timeout `150s`) while the SQS `VisibilityTimeout` was left at `30s`, SQS makes the in-flight message visible again at the 30-second mark and hands it to a second Lambda worker while the first worker is still waiting on Bedrock. Setting `VisibilityTimeout` to **at least 6x the Lambda timeout (`900 seconds`)** prevents premature message re-delivery.
2. **Preventing Bedrock `ThrottlingException` Storms (`MaximumConcurrency` + `ReportBatchItemFailures`)**: Capping `MaximumConcurrency` on the SQS Event Source Mapping throttles Lambda polling to match your Bedrock quota, and `ReportBatchItemFailures` ensures that if 1 message in a batch of 5 fails, the 4 successful LLM extractions are committed rather than re-run.

#### Distractor Analysis
- **B is incorrect**: Setting `VisibilityTimeout=0` makes messages immediately visible to all pollers simultaneously, maximizing duplicate token spend.
- **C is incorrect**: Synchronous API Gateway HTTP APIs (`v2`) time out at 30 seconds and offer no backpressure buffering for 3,000-claim surges.
- **D is incorrect**: Increasing Lambda RAM does not speed up remote Bedrock LLM token generation and leaves the 30-second visibility timeout bug intact.

</details>

---

### Question 2

A developer is using the Amazon Bedrock `Converse` API (`bedrock-runtime`) with `us.anthropic.claude-3-5-sonnet-20241022-v2:0` to build an order assistant. The developer defines two tools in `toolConfig.tools`: `get_shipping_status` and `get_billing_invoice`. When a user asks, "Can you check the shipping status AND the billing invoice for order ORD-7721?", the model returns a single `assistant` message with `stopReason == "tool_use"` containing **two** `toolUse` content blocks (`toolUseId: "tooluse_01"` and `toolUseId: "tooluse_02"`).

The developer's Python loop executes `get_shipping_status`, appends a `user` message containing the `toolResult` for `tooluse_01`, and immediately calls `bedrock_runtime.converse(messages=messages)` before executing `tooluse_02`. The `converse()` call fails with a **`ValidationException`**.

Why did `Converse` throw a `ValidationException`, and how must the developer fix the message history construction?

- **A.** Bedrock `Converse` does not support invoking more than one tool per conversation session (`sessionId`).
- **B.** When an `assistant` message contains multiple parallel `toolUse` blocks, the subsequent `user` message **must** contain the corresponding `toolResult` blocks for **all** `toolUseId`s from that assistant turn inside a **single** `user` message `content` array before calling `Converse` again.
- **C.** Each `toolResult` block must be appended with `"role": "assistant"` instead of `"role": "user"`.
- **D.** The developer must switch from `Converse` to `InvokeModel` with `amazon.titan-embed-text-v2:0`.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
In the Amazon Bedrock `Converse` / `ConverseStream` API, a single `assistant` turn can emit **multiple parallel `toolUse` content blocks**. The API contract strictly enforces that the very next message in the `messages` array must have `"role": "user"` and its `"content"` list must include a matching **`toolResult` block for every `toolUseId`** requested in the preceding assistant message. Calling `Converse` with only one of the two `toolUseId`s resolved triggers a `ValidationException`.

#### Distractor Analysis
- **A is incorrect**: `Converse` natively supports both multi-step sequential tool calls and multi-tool parallel calls within a single turn.
- **C is incorrect**: `toolResult` blocks are always sent in a `"role": "user"` message, never `"role": "assistant"`.
- **D is incorrect**: `amazon.titan-embed-text-v2:0` is an embedding model, not a generative tool-calling LLM.

</details>

---

### Question 3

A media publishing company is building a collaborative web and mobile workspace where 15 editors in a live newsroom view the same real-time investigative article draft being generated and refined by an asynchronous Amazon Bedrock Agent and multiple background fact-checking workers. Each generation workflow takes 45–90 seconds. As tokens and agent trace citations are produced by the background workers, all 15 connected browsers and mobile apps (some on flaky cellular networks that frequently drop and reconnect) must receive real-time incremental token updates over persistent WebSockets with Amazon Cognito user pool authentication.

Which AWS architecture provides managed WebSocket fan-out, automatic client reconnection, and Cognito authentication for this multi-viewer streaming pattern?

- **A.** Deploy an **Amazon API Gateway HTTP API (`v2`)** integrated synchronously with an AWS Lambda function and have all 15 clients poll `GET /status` every 50 milliseconds.
- **B.** Store each generated token as a separate object in an Amazon S3 Glacier Deep Archive bucket and use S3 Inventory reports to update the browsers.
- **C.** Use **AWS AppSync** (GraphQL Subscriptions or AppSync Event APIs) with Amazon Cognito authentication; have the client trigger an asynchronous mutation (`startArticleDraft`) that invokes a background worker Lambda/ECS task calling Bedrock `ConverseStream`, which publishes token chunks back through AppSync to all subscribed WebSocket clients.
- **D.** Configure an Amazon Route 53 CNAME record pointing browser clients directly to `bedrock-runtime.us-east-1.amazonaws.com` with hardcoded root IAM access keys in the frontend JavaScript bundle.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: C**

#### Architectural Explanation
**AWS AppSync** (via GraphQL Subscriptions or AppSync Event APIs) manages persistent, scalable WebSocket connections (`wss://`) for millions of concurrent web/mobile clients with built-in **Amazon Cognito** authentication and automatic reconnection handling in the AWS Amplify client SDK. When an asynchronous backend worker (up to 15 minutes in Lambda or unlimited in ECS Fargate) consumes tokens from Bedrock `ConverseStream` or `InvokeAgent`, it publishes incremental chunks to AppSync, which fans them out in real time to all 15 subscribed editors in the newsroom channel.

#### Distractor Analysis
- **A is incorrect**: API Gateway HTTP APIs (`v2`) do not support WebSockets or response streaming, have a hard 30-second timeout (failing on 45–90s workflows), and 50ms polling from 15 clients creates massive request overhead.
- **B is incorrect**: S3 Glacier Deep Archive has a hours-long retrieval time and is for cold archival storage.
- **D is incorrect**: Embedding IAM credentials in frontend browser JavaScript is a critical security vulnerability.

</details>

---

### Question 4

A healthcare enterprise processes 80,000 scanned hospital discharge summaries stored in an Amazon S3 bucket every evening. Each document must go through a stateful 4-step workflow:
1. Extract raw layout tables using Amazon Textract.
2. Call Amazon Bedrock (`Converse`) to structure medication dosages and generate a confidence score (`0.0` to `1.0`).
3. If the confidence score is `< 0.90`, send a review task to a clinical pharmacist queue and **pause the workflow execution for up to 48 hours** without incurring compute charges until the pharmacist submits approval.
4. Write the verified clinical record to Amazon DynamoDB.

To avoid exhausting the account's Amazon Bedrock On-Demand RPM quota, no more than 35 documents may be processed concurrently.

Which orchestration architecture meets all of these requirements?

- **A.** Use an **AWS Step Functions Express Workflow** with an Inline Map state (`MaxConcurrency=5000`) and a `Wait` state of 48 hours.
- **B.** Use a single AWS Lambda function with a 15-minute timeout that loops sequentially over all 80,000 S3 objects and calls `time.sleep(172800)` while waiting for the pharmacist.
- **C.** Submit an **Amazon Bedrock Batch Inference** job (`CreateModelInvocationJob`), which natively runs Amazon Textract, pauses for 48-hour human-in-the-loop callbacks, and writes to DynamoDB.
- **D.** Use an **AWS Step Functions Standard Workflow** featuring a **Distributed Map** state (`ProcessorConfig.Mode="DISTRIBUTED"`, `MaxConcurrency=35`) iterating over the S3 bucket, an optimized/SDK Bedrock task state with exponential backoff `Retry`, and an Amazon SQS **`sendMessage.waitForTaskToken`** callback state (`TimeoutSeconds=172800`) for low-confidence records.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: D**

#### Architectural Explanation
- **High-Scale S3 Iteration with Concurrency Control**: **AWS Step Functions Distributed Map** (`Mode="DISTRIBUTED"`) can iterate over millions of S3 objects while enforcing **`MaxConcurrency: 35`** so child executions never exceed your Bedrock token/request quota.
- **48-Hour Zero-Cost Human-in-the-Loop Pause**: Step Functions **Standard Workflows** support execution durations up to **1 year** and the **Callback Pattern (`.waitForTaskToken`)**, pausing execution at zero compute cost until the pharmacist UI calls `SendTaskSuccess` with the task token.

#### Distractor Analysis
- **A is incorrect**: Step Functions **Express Workflows** have a maximum execution duration of **5 minutes** (cannot wait 48 hours) and do **not** support `.waitForTaskToken` callbacks.
- **B is incorrect**: AWS Lambda has a hard maximum execution timeout of **15 minutes** (900 seconds).
- **C is incorrect**: Bedrock Batch Inference only executes a single FM inference pass over an S3 JSONL file; it cannot orchestrate multi-step Textract calls, conditional branching, or 48-hour human callbacks.

</details>

---

### Question 5

A financial data provider is building a microservice that calls Amazon Bedrock `Converse` (`anthropic.claude-3-5-sonnet-20241022-v2:0`) to extract quarterly earnings metrics from SEC 10-Q filings. Downstream Java services require the output to strictly conform to a complex nested JSON Schema (`EarningsReportSchema` with 18 required typed fields).

Currently, the developer instructs the model in the `system` prompt: *"You must output ONLY valid JSON matching the schema below and never include markdown code blocks or conversational text."* However, on roughly 1.5% of complex filings, the model prefixes its output with `"Here is the extracted JSON from the filing:\n```json..."`, causing `json.loads()` in the downstream parser to crash.

How can the developer use native `Converse` API capabilities to guarantee structured JSON output matching `EarningsReportSchema` without conversational preamble?

- **A.** Define `EarningsReportSchema` as the `inputSchema.json` of a tool named `submit_earnings_report` inside `toolConfig.tools`, set **`toolChoice={"tool": {"name": "submit_earnings_report"}}`**, and extract the pre-parsed dictionary directly from `output["message"]["content"][0]["toolUse"]["input"]`.
- **B.** Set `toolChoice={"auto": {}}` and remove `EarningsReportSchema` from the request.
- **C.** Increase `temperature` to `1.0` and add `"Please do not use markdown"` three times in the `user` message.
- **D.** Switch from `Converse` to `RetrieveAndGenerate` with an empty S3 bucket.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: A**

#### Architectural Explanation
In the Amazon Bedrock `Converse` API, defining your target JSON Schema as the `inputSchema` of a tool in `toolConfig` and setting **`toolChoice={"tool": {"name": "submit_earnings_report"}}`** forces the model to invoke that exact tool and populate its arguments according to the JSON Schema. Bedrock parses and returns the structured JSON object directly as a Python dictionary inside `contentBlock["toolUse"]["input"]`, eliminating conversational preambles and markdown code fence parsing errors.

#### Distractor Analysis
- **B is incorrect**: `toolChoice={"auto": {}}` allows the model to reply with plain conversational text instead of invoking the structured schema tool.
- **C is incorrect**: Raising `temperature` to `1.0` increases output variance and format violations.
- **D is incorrect**: `RetrieveAndGenerate` is a Knowledge Base RAG API and does not enforce arbitrary custom JSON output schemas.

</details>

---

### Question 6

A large multinational bank operates a multi-account AWS Organization with 60 spoke workload VPCs connected to a central **Network Inspection & Shared Services VPC** via **AWS Transit Gateway**, plus an on-premises data center connected via **AWS Direct Connect**. Security policy mandates that all `InvokeModel`, `Converse`, and `InvokeAgent` calls from all 60 spoke VPCs and on-premises servers must traverse **AWS PrivateLink** without ever touching the public internet, and the network architecture must minimize hourly VPC Endpoint Elastic Network Interface (ENI) charges across the 60 VPCs.

Which two components are required to implement this centralized PrivateLink architecture? **(Choose TWO)**

- **A.** Provision Interface VPC Endpoints for `com.amazonaws.<region>.bedrock-runtime` and `com.amazonaws.<region>.bedrock-agent-runtime` **once** inside the central Shared Services VPC with **Private DNS disabled** on the endpoints.
- **B.** Create **Amazon Route 53 Private Hosted Zones** for `bedrock-runtime.<region>.amazonaws.com` and `bedrock-agent-runtime.<region>.amazonaws.com` with Alias `A` records pointing to the central Interface VPC Endpoints, associate the Private Hosted Zones with the 60 spoke VPCs, and configure a Route 53 Inbound Resolver Endpoint for on-premises DNS resolution.
- **C.** Provision 120 separate Interface VPC Endpoints (2 per VPC across 3 AZs = 360 ENIs) inside every single spoke VPC and attach Internet Gateways to route traffic over public IP addresses.
- **D.** Use an **Amazon S3 Gateway VPC Endpoint** to route `bedrock-runtime` API traffic, because Gateway Endpoints are free and support all AWS services.
- **E.** Configure AWS Direct Connect Public VIFs to route traffic to public internet Bedrock endpoints without encryption.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answers: A and B**

#### Architectural Explanation
1. **Centralized Interface VPC Endpoints (Option A)**: Amazon Bedrock uses **Interface VPC Endpoints** (`AWS PrivateLink`) for `com.amazonaws.<region>.bedrock-runtime` and `com.amazonaws.<region>.bedrock-agent-runtime`. Provisioning them once in the central Shared Services VPC (with built-in Private DNS disabled so you can manage custom Private Hosted Zones) avoids paying hourly ENI charges across 60 separate spoke VPCs.
2. **Hub-and-Spoke DNS Resolution via Route 53 Private Hosted Zones (Option B)**: Creating Route 53 Private Hosted Zones for the Bedrock service domain names (`bedrock-runtime.<region>.amazonaws.com`) with Alias records pointing to the central VPC Endpoint DNS name—and associating those PHZs with all spoke VPCs over Transit Gateway (plus Route 53 Inbound Resolver for on-premises Direct Connect)—allows standard `boto3` SDK clients in any spoke VPC or on-premises server to transparently resolve Bedrock to the private central ENI IPs.

#### Distractor Analysis
- **C is incorrect**: Provisioning duplicate Interface Endpoints in all 60 spoke VPCs multiplies hourly ENI costs by 60x, and using an Internet Gateway violates the private connectivity mandate.
- **D is incorrect**: Gateway VPC Endpoints only exist for **Amazon S3** and **Amazon DynamoDB**; Amazon Bedrock requires **Interface VPC Endpoints**.
- **E is incorrect**: Public VIFs route over public IP space rather than AWS PrivateLink.

</details>

---

### Question 7

A SaaS company is building a single-tenant customer support web widget that sends complex multi-document prompts to Amazon Bedrock `ConverseStream`. Generating the full response takes between **35 and 75 seconds**. The frontend team wants to make a single standard HTTPS request that streams tokens back progressively using Server-Sent Events (SSE) / chunked transfer encoding, while enforcing **Amazon API Gateway Usage Plans, API Keys, and a custom Lambda Authorizer** at the edge.

Initially, the team deployed an **Amazon API Gateway HTTP API (`v2`)** backed by an AWS Lambda function, but every request failed with an `HTTP 503 / 504 Service Unavailable (Integration Timeout)` at exactly **30 seconds**, and tokens were buffered instead of streamed.

How should the architect redesign the API layer to preserve API Gateway Usage Plans and Lambda Authorizers while supporting progressive token streaming for 75-second generations?

- **A.** Keep the API Gateway HTTP API (`v2`) and raise its integration timeout to 900 seconds in the AWS Console.
- **B.** Migrate to an **Amazon API Gateway REST API (`v1`)** configured with **Response Streaming** (`ResponseTransferMode: STREAM` on the `AWS_PROXY` integration, supporting up to a 15-minute streaming timeout and 10 MB payload) invoking an AWS Lambda function written with `InvokeMode: RESPONSE_STREAM`.
- **C.** Replace the Lambda function with an Amazon S3 static website hosting a pre-recorded `.mp4` video.
- **D.** Use an Amazon SQS FIFO queue directly as the browser's SSE endpoint.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
- **Why HTTP API (`v2`) Failed**: Amazon API Gateway **HTTP APIs (`v2`)** have a hard maximum integration timeout of **30 seconds** and do **not** support response streaming or API Gateway Usage Plans/API Keys.
- **Why REST API (`v1`) with Response Streaming Succeeds**: **Amazon API Gateway REST APIs (`v1`)** support **Response Streaming** (`ResponseTransferMode: STREAM`) with **`AWS_PROXY` (`InvokeWithResponseStream`)**, extending the timeout up to **15 minutes (900 seconds)** for progressive token streaming while natively providing **Usage Plans, API Keys, AWS WAF, and Lambda/Cognito Authorizers**.

#### Distractor Analysis
- **A is incorrect**: API Gateway HTTP API (`v2`) has a hard non-configurable 30-second limit and does not support response streaming.
- **C is incorrect**: Static S3 hosting cannot execute dynamic LLM streaming.
- **D is incorrect**: SQS does not support SSE HTTP streaming to browser clients.

</details>

---

### Question 8

A platform engineering team is deploying an enterprise-wide Python SDK wrapper around `boto3.client("bedrock-runtime")` used by 40 internal microservices. During peak traffic hours, microservices intermittently experience `ThrottlingException (HTTP 429)` and `ModelTimeoutException (HTTP 408)` errors. Furthermore, when a 120,000-token legal contract is sent to `Converse` (non-streaming), the `boto3` client raises a `ReadTimeoutError` at exactly 60 seconds even though the Bedrock model finishes generating the response at 78 seconds.

How should the platform team configure `botocore.config.Config` inside their shared Python SDK wrapper to mitigate retry storms, dynamically pace outbound request rates, and prevent premature client-side socket disconnects on long-context prompts?

- **A.** Configure `Config(read_timeout=300, connect_timeout=5, retries={"max_attempts": 8, "mode": "adaptive"})`.
- **B.** Configure `Config(read_timeout=10, retries={"max_attempts": 0, "mode": "legacy"})` and wrap every call in an infinite `while True:` loop with `time.sleep(0.01)`.
- **C.** Set `endpoint_url="http://localhost:8080"` and disable SSL verification (`verify=False`).
- **D.** Use `boto3.client("bedrock")` control-plane client with default settings instead of `bedrock-runtime`.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: A**

#### Architectural Explanation
1. **Preventing Premature Socket Disconnects (`read_timeout=300`)**: The default `botocore` `read_timeout` is **60 seconds**. For non-streaming `Converse` calls on large 100k+ token contexts or deep reasoning models, increasing `read_timeout` to `300` seconds prevents the client from severing the socket at 60 seconds.
2. **Client-Side Token-Bucket Rate Limiting (`mode="adaptive"`)**: Setting `retries={"max_attempts": 8, "mode": "adaptive"}` enables exponential backoff with jitter **plus** a client-side token-bucket rate limiter that automatically throttles outbound call frequency as soon as `ThrottlingException (HTTP 429)` responses are detected.

#### Distractor Analysis
- **B is incorrect**: Setting `read_timeout=10` will fail almost all LLM requests, and an un-jittered 10ms `while True` retry loop creates a catastrophic retry storm (thundering herd) against the Bedrock endpoint.
- **C is incorrect**: Pointing to localhost and disabling TLS breaks AWS connectivity and security.
- **D is incorrect**: `bedrock` is the control-plane client and does not support `converse()`.

</details>

---

### Question 9

A DevOps team manages a production Amazon Bedrock Agent (`agentId="AGENT12345"`) using AWS CDK and AWS CodePipeline. Currently, their web application invokes the agent using the built-in test alias `agentAliasId="TSTALIASID"` (which points directly to the mutable `DRAFT` working version). Yesterday, when a prompt engineer updated the `DRAFT` orchestration prompt in the staging branch and ran `prepare_agent` to test an experimental tool, live production users immediately started experiencing broken tool calls for 25 minutes.

How should the DevOps team re-architect their Agent versioning and CI/CD deployment pipeline to guarantee zero-downtime deployments and instant rollback isolation between `DRAFT` edits and live production traffic?

- **A.** Continue pointing production clients to `TSTALIASID`, but restrict developers to editing the agent only between 2:00 AM and 4:00 AM UTC.
- **B.** Create a brand-new AWS account every time a prompt template is modified and ask customers to update their DNS records.
- **C.** Create a named **Agent Alias** (e.g., `prod-live-alias`) pinned to an immutable numbered **Agent Version** (e.g., `Version 1`), configure the production web application to invoke `prod-live-alias`, and configure the CI/CD pipeline to: 1) update and `prepare_agent` on `DRAFT`, 2) create a new immutable numbered version (`Version 2`) and run automated evaluation gates against a staging alias, and 3) update `prod-live-alias` `routingConfiguration` to point to `Version 2` (with instant rollback to `Version 1` if alarms fire).
- **D.** Export the Bedrock Agent to an Amazon EC2 Auto Scaling Group launch template.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: C**

#### Architectural Explanation
In Amazon Bedrock Agents:
- **`TSTALIASID`** always points directly to the mutable **`DRAFT`** working version and must **never** be used for production traffic.
- **Named Agent Aliases (`aws_bedrockagent_agent_alias` / `AWS::Bedrock::AgentAlias`)** point to **immutable numbered Agent Versions (`1`, `2`, `3`...)** via their `routingConfiguration`. By having production applications invoke the stable `prod-live-alias` ID, engineers can freely modify and `prepare_agent` on `DRAFT`, validate a newly snapshotted numbered version via a staging alias, and atomically cut over `prod-live-alias` to `Version 2` with zero downtime and 1-second rollback capability to `Version 1`.

#### Distractor Analysis
- **A is incorrect**: Editing `DRAFT` while production points to `TSTALIASID` still causes production outages during the maintenance window.
- **B is incorrect**: Creating new AWS accounts per prompt edit is an operational anti-pattern.
- **D is incorrect**: Bedrock Agents are serverless managed resources and cannot be exported as EC2 AMIs/Launch Templates.

</details>

---

### Question 10

A Principal Software Architect at a financial technology company wants to improve the code generation accuracy of **Amazon Q Developer** for the company's 450 backend engineers. The company maintains a proprietary internal Python and Java microservices framework (`FinCore-SDK`, comprising 180 MB of source code across 1,400 files in a private GitHub Enterprise repository) with custom authentication decorators, internal Protobuf schemas, and DynamoDB data-access patterns that public Foundation Models have never seen.

The architect wants Amazon Q Developer inline completions and chat to generate idiomatic `FinCore-SDK` code across all 450 engineers' IDEs without requiring every engineer to clone all 1,400 framework files into their local laptop workspace, while ensuring the proprietary source code is never used to train base models shared with other AWS customers.

Which configuration meets all of these requirements?

- **A.** Ask all 450 engineers to paste the entire 180 MB `FinCore-SDK` codebase into the Amazon Q chat window on every prompt.
- **B.** Use the free individual tier of Amazon Q Developer with only local `@workspace` indexing on an empty folder.
- **C.** Upload the `FinCore-SDK` source code to a public S3 bucket with `Principal: "*"` so Amazon Q public crawlers index it.
- **D.** Subscribe to **Amazon Q Developer Pro**, connect the private GitHub Enterprise repository (or an S3 bucket sync) to create an **Amazon Q Developer Customization**, verify that the **Customization Evaluation** score meets quality thresholds, and assign access to the 450 engineers' groups via **AWS IAM Identity Center**.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: D**

#### Architectural Explanation
**Amazon Q Developer Customizations** (available on the **Amazon Q Developer Pro** tier) allow enterprises to connect private repositories (GitHub, GitLab, Bitbucket, or Amazon S3 containing at least 10–20 MB of high-quality internal code) to customize Amazon Q on private internal SDKs and architectural idioms. AWS computes a **Customization Evaluation** metric to verify quality gains before administrators publish the customization to authorized developer groups via **AWS IAM Identity Center**. All enterprise code and customizations remain strictly isolated, KMS-encrypted, and are **never** used to train shared AWS foundation models.

#### Distractor Analysis
- **A is incorrect**: 180 MB of code vastly exceeds IDE chat context limits.
- **B is incorrect**: Local `@workspace` indexing only indexes files currently open/cloned in the developer's local IDE project, not uncloned enterprise-wide shared framework repositories.
- **C is incorrect**: Exposing proprietary financial source code in a public S3 bucket is a severe security violation.

</details>

---

### Question 11

An e-commerce company runs a high-volume customer service chatbot on Amazon Bedrock using the `Converse` API. Analysis of 10 million monthly prompts reveals that **72% of customer queries** are simple greetings, store-hour lookups, or order-tracking status checks that can be answered accurately by `anthropic.claude-3-5-haiku-20241022-v1:0` (or `amazon.nova-lite-v1:0`), while the remaining **28% of queries** involve complex multi-item warranty policy interpretation requiring the deeper reasoning of `anthropic.claude-3-5-sonnet-20241022-v2:0` (or `amazon.nova-pro-v1:0`).

The engineering team wants a **fully managed, zero-code-maintenance Amazon Bedrock feature** that automatically inspects each incoming prompt's complexity and routes simple prompts to the faster/cheaper model and complex prompts to the flagship model within the same model family, without building and maintaining a custom classifier microservice.

Which Amazon Bedrock capability should the team use?

- **A.** **Amazon Bedrock Cross-Region Inference Profiles** (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`).
- **B.** **Amazon Bedrock Intelligent Prompt Routing** (invoking a default or custom Prompt Router ARN in the `modelId` parameter of `Converse` / `ConverseStream`).
- **C.** **Amazon Bedrock Custom Model Import (CMI)**.
- **D.** **Amazon Bedrock Model Evaluation** jobs.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
**Amazon Bedrock Intelligent Prompt Routing** provides a single managed Prompt Router endpoint ARN that dynamically evaluates the complexity of each incoming prompt in real time and routes it between models within the same family (e.g., between Claude 3.5 Haiku and Claude 3.5 Sonnet, or between Amazon Nova Lite and Nova Pro) based on a configurable response quality difference tolerance. This reduces inference cost by up to 30–50% and lowers latency without requiring custom classifier infrastructure.

#### Distractor Analysis
- **A is incorrect**: Cross-Region Inference Profiles route traffic across *geographical regions* for the *same* model to increase burst quota, not between different model sizes (Haiku vs. Sonnet) based on prompt complexity.
- **C is incorrect**: Custom Model Import is for hosting customer-owned `.safetensors` weights.
- **D is incorrect**: Model Evaluation is an offline benchmarking tool, not a real-time runtime prompt router.

</details>

---

### Question 12

A developer is building a multimodal document analysis service on Amazon Bedrock that needs to:
1. Send a 3-page PDF contract (`byte` array) alongside a natural language prompt to Anthropic Claude 3.5 Sonnet, and later switch to Amazon Nova Pro without changing the request/response Python code.
2. Generate 1,024-dimensional dense vector embeddings for text chunks using **Amazon Titan Text Embeddings V2** (`amazon.titan-embed-text-v2:0`).

Which combination of `bedrock-runtime` APIs should the developer use for Task 1 and Task 2?

- **A.** Use `InvokeModel` for Task 1 (multimodal PDF chat across multiple model families) and `Converse` for Task 2 (generating Titan Text Embeddings V2 vectors).
- **B.** Use `ConverseStream` for both Task 1 and Task 2, because `ConverseStream` natively generates 1,024-dimensional float arrays for Titan Text Embeddings V2.
- **C.** Use **`Converse` (or `ConverseStream`)** with a `document` content block for Task 1 (enabling model-agnostic switching between Claude 3.5 Sonnet and Nova Pro), and use **`InvokeModel`** for Task 2 (because embedding models do not use the conversational `Converse` API).
- **D.** Use `RetrieveAndGenerate` for both Task 1 and Task 2 without a Knowledge Base.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: C**

#### Architectural Explanation
- **Task 1 (`Converse` / `ConverseStream`)**: The `Converse` API provides a unified, model-agnostic schema supporting text, images, and `document` blocks (PDF, CSV, DOCX, etc.) across all conversational models (Claude, Nova, Llama, Mistral), allowing seamless model switching by changing only the `modelId` string.
- **Task 2 (`InvokeModel`)**: Embedding models (`amazon.titan-embed-text-v2:0`, `cohere.embed-*`) and image generation models return numerical vector arrays or image bytes rather than conversational messages; they **must** be invoked using **`InvokeModel`**.

#### Distractor Analysis
- **A is incorrect**: Reverses the APIs; calling `Converse` on `amazon.titan-embed-text-v2:0` fails with a `ValidationException` because embedding models do not support the `Converse` operation.
- **B is incorrect**: `ConverseStream` does not support embedding models.
- **D is incorrect**: `RetrieveAndGenerate` requires an existing Bedrock Knowledge Base or S3 source and does not return raw 1,024-dimensional embedding vectors.

</details>

---

### Question 13

An enterprise AI team is invoking Anthropic Claude 3.7 Sonnet (`us.anthropic.claude-3-7-sonnet-20250219-v1:0`) via the Amazon Bedrock **`Converse`** API. Because the `Converse` API normalizes common parameters (`maxTokens`, `temperature`, `topP`, `stopSequences`) inside `inferenceConfig`, the team needs to pass model-specific parameters that are not part of the standard `inferenceConfig` object—specifically, enabling Claude's **Extended Thinking** (`{"thinking": {"type": "enabled", "budget_tokens": 4000}}`) or passing model-specific top-K sampling parameters.

How should the developer pass model-specific request parameters when using the unified `Converse` or `ConverseStream` API?

- **A.** Abandon the `Converse` API completely and rewrite the entire application to use raw HTTP `curl` requests against `InvokeModel`.
- **B.** Pass the model-specific dictionary inside the **`additionalModelRequestFields`** parameter of `converse()` / `converse_stream()`.
- **C.** Embed `"budget_tokens": 4000` inside `sessionState.sessionAttributes`.
- **D.** Pass the parameters as environment variables on the AWS IAM Role.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
The Amazon Bedrock `Converse` and `ConverseStream` APIs provide the **`additionalModelRequestFields`** parameter (accepting a JSON-serializable `dict`) specifically so developers can pass provider-specific or model-specific inference attributes (such as `top_k`, Claude `thinking` configuration, or custom beta flags) that lie outside the standardized `inferenceConfig` structure, without sacrificing the unified `Converse` message and tool-use schema. Similarly, `additionalModelResponseFieldPaths` extracts model-specific fields from the response.

#### Distractor Analysis
- **A is incorrect**: You do not need to abandon `Converse` to pass model-specific fields.
- **C is incorrect**: `sessionState.sessionAttributes` belongs to the `InvokeAgent` (`bedrock-agent-runtime`) API, not `Converse` (`bedrock-runtime`).
- **D is incorrect**: IAM roles do not carry runtime LLM inference parameters.

</details>

---

### Question 14

A cloud security architect is writing a VPC Endpoint Policy for the `com.amazonaws.us-east-1.bedrock-runtime` Interface VPC Endpoint in a regulated healthcare VPC. The architect wants to ensure that:
1. Only IAM principals belonging to the company's AWS Organization (`o-healthcorp99`) can invoke models through this VPC Endpoint.
2. Callers are **only** permitted to invoke `anthropic.claude-3-5-sonnet-20241022-v2:0` and **must** include the company's mandatory HIPAA Guardrail (`arn:aws:bedrock:us-east-1:111122223333:guardrail/gr-hipaa-01`).

Which two IAM / VPC Endpoint Policy mechanisms are required to enforce these controls? **(Choose TWO)**

- **A.** In the VPC Endpoint Policy, include a `Condition` block checking `"StringEquals": {"aws:PrincipalOrgID": "o-healthcorp99"}` on `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream`.
- **B.** Scope the `Resource` element of the VPC Endpoint Policy (and/or the caller's IAM policy) to the specific foundation model ARN `arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0` and the guardrail ARN `arn:aws:bedrock:us-east-1:111122223333:guardrail/gr-hipaa-01`.
- **C.** Set `"Principal": {"AWS": "anonymous"}` and `"Action": "*"` in the VPC Endpoint Policy.
- **D.** Attach the VPC Endpoint Policy to an Amazon S3 bucket instead of the Interface VPC Endpoint.
- **E.** Open inbound TCP port `22` (SSH) from `0.0.0.0/0` on the VPC Endpoint Security Group.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answers: A and B**

#### Architectural Explanation
1. **Organization Boundary Enforcement (Option A)**: Attaching `"Condition": {"StringEquals": {"aws:PrincipalOrgID": "o-healthcorp99"}}` to the `bedrock-runtime` Interface VPC Endpoint Policy ensures that stolen credentials from external AWS accounts cannot use the private network path to exfiltrate data or invoke models.
2. **Model & Guardrail Resource Scoping (Option B)**: Scoping the allowed `Resource` ARNs in the VPC Endpoint Policy and IAM policies to the exact approved `foundation-model` ARN and `guardrail` ARN prevents workloads from invoking unapproved models.

#### Distractor Analysis
- **C is incorrect**: Allowing wildcard actions to anonymous principals removes all access control.
- **D is incorrect**: S3 Bucket Policies do not govern `bedrock-runtime` Interface VPC Endpoints.
- **E is incorrect**: Interface VPC Endpoints accept HTTPS traffic on TCP port `443` from within the VPC CIDR, never SSH (`22`) from `0.0.0.0/0`.

</details>

---

### Question 15

A financial structured-products desk has an automated data pipeline that calls Amazon Bedrock `Converse` with three data-lookup tools (`get_yield_curve`, `get_cds_spread`, `get_volatility_surface`). In 15% of user queries, the user asks a general educational question (e.g., "What is convexity in bond pricing?"), yet the developer wants to force the model to **always** invoke **at least one** of the three market-data tools on every single turn so that every answer is grounded in live market numbers, while still letting the model choose *which* of the three tools is most appropriate.

How should the developer configure `toolChoice` inside `toolConfig` on the `Converse` request?

- **A.** Set `toolChoice={"auto": {}}`.
- **B.** Set `toolChoice={"tool": {"name": "get_yield_curve"}}`.
- **C.** Omit `toolConfig` from the `Converse` call and list the tools in `system`.
- **D.** Set **`toolChoice={"any": {}}`**.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: D**

#### Architectural Explanation
In the Amazon Bedrock `Converse` and `ConverseStream` APIs, `toolConfig.toolChoice` supports three modes:
- `{"auto": {}}`: The model decides whether to call a tool or respond directly with text.
- `{"tool": {"name": "<tool_name>"}}`: Forces the model to call that **one specific named tool**.
- **`{"any": {}}`**: Forces the model to call **at least one** of the tools defined in `toolConfig.tools`, while allowing the model to autonomously select which tool(s) among the three best match the prompt.

#### Distractor Analysis
- **A is incorrect**: `{"auto": {}}` allows the model to answer general questions like "What is convexity?" from pre-trained weights without invoking any tool.
- **B is incorrect**: `{"tool": {"name": "get_yield_curve"}}` forces the model to always call `get_yield_curve` even when the user asked about CDS spreads or volatility surfaces.
- **C is incorrect**: Omitting `toolConfig` disables native tool calling completely.

</details>
