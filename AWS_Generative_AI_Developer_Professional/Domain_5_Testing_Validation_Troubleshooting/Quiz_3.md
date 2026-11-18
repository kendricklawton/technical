# Domain 5 — Quiz 3: Troubleshooting Bedrock Agents (`enableTrace`), Multi-Agent Routing, Action Group Errors, `Converse` Exceptions & Throttling (Task 5.2)

---

### Question 1
A developer creates an Amazon Bedrock Agent (`arn:aws:bedrock:us-east-1:111122223333:agent/AGENT998877`) with an Action Group backed by an AWS Lambda function (`OrderLookupFunction`). The developer attaches an IAM policy granting `lambda:InvokeFunction` on `OrderLookupFunction` directly to the Bedrock Agent's IAM Execution Role (`BedrockAgentExecutionRole`).

However, when testing the Agent in the console with `enableTrace=True`, the Agent correctly generates an `invocationInput` for `OrderLookupFunction`, but immediately terminates with a `failureTrace` containing:
`"failureReason": "DependencyFailedException: Access denied while invoking Lambda function arn:aws:lambda:us-east-1:111122223333:function:OrderLookupFunction"`
Furthermore, no invocation log entry appears in the CloudWatch Log Group for `OrderLookupFunction`.

Why is Amazon Bedrock unable to invoke the Lambda function, and how should the developer fix it?

- **A.** The Lambda function must be deployed inside a private VPC subnet with a NAT Gateway before Amazon Bedrock can reach it.
- **B.** The developer must attach the `AWSLambdaBasicExecutionRole` managed policy to the user's browser session.
- **C.** Amazon Bedrock Agents invoke Action Group Lambda functions using the **Bedrock service principal (`bedrock.amazonaws.com`)** rather than assuming the Agent's IAM Execution Role to call Lambda; the developer must attach a **Lambda Resource-Based Policy** (`aws lambda add-permission`) to `OrderLookupFunction` allowing principal `bedrock.amazonaws.com` the `lambda:InvokeFunction` action scoped to `--source-arn arn:aws:bedrock:us-east-1:111122223333:agent/AGENT998877`.
- **D.** Amazon Bedrock Agents can only invoke Lambda functions written in Node.js 20.x, not Python 3.12.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
This is one of the highest-frequency troubleshooting scenarios on the exam:
- For **Knowledge Bases and Foundation Models**, the Bedrock Agent uses its **IAM Execution Role** (`roleArn`).
- For **Action Group Lambda Functions**, the Amazon Bedrock service control plane invokes the Lambda directly using the **`bedrock.amazonaws.com` service principal**. Therefore, adding `lambda:InvokeFunction` to the Agent's IAM Role has zero effect! You **must** add a resource-based permission policy on the Lambda function itself granting `lambda:InvokeFunction` to `bedrock.amazonaws.com` (with `AWS:SourceAccount` and `AWS:SourceArn` conditions).

**Distractor Analysis:**
- **Why A is incorrect:** Bedrock invokes Lambda via the internal AWS Lambda control plane API regardless of whether the function is VPC-attached or non-VPC.
- **Why B is incorrect:** `AWSLambdaBasicExecutionRole` is for the Lambda function's own IAM role to write logs to CloudWatch Logs; it does not grant Bedrock permission to invoke the function.
- **Why D is incorrect:** Bedrock Action Group Lambdas support any AWS Lambda runtime (Python, Node.js, Java, Go, Rust, .NET).
</details>

---

### Question 2
An engineer has configured an Amazon Bedrock Agent Action Group using an **OpenAPI 3.0 schema** (`apiSchema`) to handle RMA warranty claims via an AWS Lambda function (`RmaProcessorFn`). When a user asks the Agent to check claim `RMA-55201`, the engineer checks CloudWatch Logs for `RmaProcessorFn` and verifies that the Python handler executed without throwing any exceptions and returned the following dictionary:

```python
return {
    "statusCode": 200,
    "headers": {"Content-Type": "application/json"},
    "body": json.dumps({"rma_id": "RMA-55201", "status": "APPROVED"})
}
```

However, in the Bedrock Agent trace (`enableTrace=True`), the orchestration loop immediately fails with:
`"failureReason": "BadGatewayException: Received invalid response from API execution"` (HTTP 502).

What caused the `BadGatewayException`, and how must the Lambda return payload be changed?

- **A.** The `httpStatusCode` for Bedrock Agents must be returned as a string (`"200 OK"`) instead of an integer (`200`).
- **B.** The Lambda payload must be compressed with `gzip` and Base64-encoded before returning to Bedrock.
- **C.** The OpenAPI 3.0 schema must be converted to GraphQL SDL format before Lambda can return JSON strings.
- **D.** The Lambda function returned an Amazon API Gateway proxy response dictionary (`statusCode`, `headers`, `body`) instead of the mandatory **Bedrock Agent Action Group response schema**; the function must return a dictionary containing `"messageVersion": "1.0"` and a `"response"` object echoing `"actionGroup"`, `"apiPath"`, `"httpMethod"`, `"httpStatusCode": 200`, and `"responseBody": {"application/json": {"body": "..."}}`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
When a Lambda function executes cleanly in CloudWatch Logs yet the Bedrock Agent throws `BadGatewayException (502)`, the root cause is almost always a **malformed Lambda return payload**. Bedrock Agents do NOT accept the standard API Gateway proxy format (`{"statusCode": 200, "body": "..."}`). For an OpenAPI-based Action Group, the Lambda must return:
```json
{
  "messageVersion": "1.0",
  "response": {
    "actionGroup": "<event.actionGroup>",
    "apiPath": "<event.apiPath>",
    "httpMethod": "<event.httpMethod>",
    "httpStatusCode": 200,
    "responseBody": {
      "application/json": {
        "body": "{\"rma_id\": \"RMA-55201\", \"status\": \"APPROVED\"}"
      }
    }
  }
}
```

**Distractor Analysis:**
- **Why A is incorrect:** `httpStatusCode` in the Bedrock Agent response schema must be an integer (`200`), not a string.
- **Why B is incorrect:** Bedrock Agents expect plain UTF-8 JSON dictionaries from Lambda, not gzipped Base64 blobs.
- **Why C is incorrect:** Bedrock Action Groups use OpenAPI 3.0 JSON/YAML schemas or Function Details schemas, not GraphQL SDL.
</details>

---

### Question 3
A financial technology company built a custom tool-calling agent in Python using the Amazon Bedrock **`Converse`** API (`bedrock_runtime.converse`). The agent has access to a `get_stock_quote` tool and a `calculate_portfolio_var` tool. During testing, when `get_stock_quote` raises a temporary third-party API timeout exception inside the Python application, the developer's `except` block logs the error and immediately calls `bedrock_runtime.converse` again with a new `user` message saying:
`{"role": "user", "content": [{"text": "The stock quote API timed out. Please apologize to the user."}]}`

However, `bedrock_runtime.converse` immediately fails with an HTTP 400 error:
`botocore.errorfactory.ValidationException: An error occurred (ValidationException) when calling the Converse operation: Expected toolResult blocks for the following toolUseIds: [tooluse_k9P2mX8qR1...]`

Why did `Converse` throw this `ValidationException`, and how should the developer fix the error handling logic?

- **A.** Whenever a model responds in `Converse` with `stopReason == "tool_use"`, the assistant message added to the `messages` history contains a `toolUse` content block with a specific `toolUseId`; the `Converse` API contract strictly requires the immediately following `user` message to contain a matching `toolResult` block (`{"toolResult": {"toolUseId": "...", "content": [{"text": "..."}], "status": "error"}}`) for **every** requested `toolUseId` before continuing the conversation.
- **B.** The `Converse` API does not allow `user` messages to contain the word `"error"` or `"timed out"`; the developer must sanitize the string.
- **C.** When a tool fails, the next message in `Converse` must use `"role": "system"` instead of `"role": "user"`.
- **D.** The developer must generate a brand-new random UUID for `toolUseId` in the `user` message so it does not collide with the assistant's `toolUseId`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
In the Amazon Bedrock `Converse` and `ConverseStream` APIs, once an `assistant` message containing one or more `toolUse` blocks is appended to the `messages` list, the state machine mandates that the **very next message** in `messages` must have `"role": "user"` and must include a corresponding `toolResult` block matching **each** `toolUseId` from that assistant turn. If a Python tool fails with an exception, you must catch the exception and still return a `toolResult` block with the exact `toolUseId` and `"status": "error"` (containing the error description in `content`), allowing the LLM to gracefully recover or inform the user without violating the `Converse` message schema.

**Distractor Analysis:**
- **Why B is incorrect:** There is no restriction on the words `"error"` or `"timed out"` in `Converse`.
- **Why C is incorrect:** System prompts in `Converse` are passed in the top-level `system=[...]` parameter; the `messages` array only accepts `"user"` and `"assistant"` roles.
- **Why D is incorrect:** The `toolResult.toolUseId` **must exactly match** the `toolUse.toolUseId` emitted by the assistant; generating a different UUID causes the exact same `ValidationException`.
</details>

---

### Question 4
An airline's Customer Service Bedrock Agent has two Action Groups:
1. `FlightStatusActionGroup` with operations `getFlightByNumber` and `getFlightsByRoute`.
2. `BookingModificationActionGroup` with operations `rebookPassenger` (requires `booking_reference_uuid` in UUID v4 format) and `cancelBooking` (requires `booking_reference_uuid` and `reason_code`).

Customers complain that when they say: *"My flight AA-402 is delayed, please rebook me on the next flight to Chicago; my confirmation code is X7K9P2"*, the Agent immediately invokes `rebookPassenger` passing `"X7K9P2"` into `booking_reference_uuid`, which causes the backend API to fail with a 400 UUID validation error because the Agent skipped calling `lookupBookingUuidByConfirmationCode` first!

Upon inspecting the `orchestrationTrace` (`enableTrace=True`), the developer sees the `rationale`:
`"The user provided confirmation code X7K9P2, so I will pass X7K9P2 directly as the booking_reference_uuid parameter to rebookPassenger."`

How should the developer modify the Agent configuration so the orchestration LLM reliably executes the lookup step first?

- **A.** Increase the Agent orchestration foundation model's `temperature` from `0.0` to `1.0` so it explores alternative API call sequences randomly.
- **B.** Update the OpenAPI 3.0 schema parameter `description` (and regex `pattern`) for `booking_reference_uuid` in `rebookPassenger` and `cancelBooking` to explicitly state: *"Internal 36-character UUID (e.g., 123e4567-e89b-12d3-a456-426614174000). Do NOT pass a 6-character alphanumeric confirmation code (PNR) here; you MUST first invoke `lookupBookingUuidByConfirmationCode` with the 6-character PNR to obtain this UUID"*, and reinforce this prerequisite rule in the Agent Instructions.
- **C.** Enable the Post-Processing trace step (`postProcessingTrace`) so the Agent rewrites the API parameters after calling the Lambda function.
- **D.** Attach a Bedrock Guardrail PII filter configured to redact 6-character alphanumeric strings before the Agent sees the user prompt.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
In Amazon Bedrock Agents, the orchestration foundation model decides which Action Group operation to call and how to populate parameters based entirely on:
1. The **Agent Instructions**
2. The **Action Group & Operation `description` fields**
3. The **Parameter `description`, `type`, `enum`, and `pattern` fields** in the OpenAPI 3.0 / Function schema.
When an agent passes a user-facing 6-character PNR (`"X7K9P2"`) into a parameter expecting an internal 36-character UUID, it is because the schema failed to distinguish the two concepts or specify the prerequisite tool dependency. Adding explicit parameter descriptions, regex patterns, and prerequisite instructions (`"Always call lookupBookingUuidByConfirmationCode first..."`) directly guides the ReAct orchestration loop's `rationale`.

**Distractor Analysis:**
- **Why A is incorrect:** Increasing `temperature` to `1.0` degrades tool-calling determinism and increases schema violations.
- **Why C is incorrect:** Post-processing runs *after* the orchestration loop has already finished all tool calls and generated a final answer; it cannot fix tool selection during orchestration.
- **Why D is incorrect:** Redacting the user's confirmation code prevents `lookupBookingUuidByConfirmationCode` from working at all.
</details>

---

### Question 5
A software engineering team has built a **Multi-Agent Collaboration** system in Amazon Bedrock consisting of a **Supervisor Agent** (`SupervisorRouter`) with `routingClassifier` mode enabled (`SUPERVISOR_ROUTER`) and three specialist sub-agent collaborators:
1. `BillingSpecialistAgent`
2. `TechnicalNetworkingAgent`
3. `SecurityComplianceAgent`

Users report that when they ask questions about **"VPC Security Group ingress rules and network ACL packet drops"**, the request is intermittently routed to `SecurityComplianceAgent` (which only knows about SOC2/ISO27001 audit certifications) instead of `TechnicalNetworkingAgent` (which owns AWS VPC, Security Groups, NACLs, and Transit Gateway troubleshooting).

Which step should the developer take to diagnose and fix this misrouting?

- **A.** Delete `SecurityComplianceAgent` and merge all three agents into a single Lambda function.
- **B.** Disable `enableTrace` in production because tracing alters the routing classifier weights.
- **C.** Change the vector store of `SecurityComplianceAgent` from OpenSearch Serverless to Pinecone.
- **D.** Invoke the Supervisor Agent with `enableTrace=True`, inspect `routingClassifierTrace` (`modelInvocationOutput` and `agentCollaboratorInvocationInput`) to confirm the routing classifier selected `SecurityComplianceAgent` due to the keyword `"Security"`, and update the `collaboratorInstruction` on both sub-agent associations to clearly disambiguate their scope (specifying that `TechnicalNetworkingAgent` handles VPCs, Security Groups, NACLs, and firewall rules, while `SecurityComplianceAgent` handles regulatory audit frameworks like SOC2, HIPAA, and ISO27001).

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
In Amazon Bedrock Multi-Agent Collaboration with `SUPERVISOR_ROUTER` mode, the routing classifier LLM inspects the incoming user prompt against the **`collaboratorInstruction`** defined for each associated sub-agent collaborator. Because `"VPC Security Group"` contains the word `"Security"`, vague collaborator descriptions (e.g., *"Handles security questions"* vs *"Handles networking questions"*) cause boundary ambiguity. Inspecting **`routingClassifierTrace`** confirms the classifier's decision, and refining each sub-agent's `collaboratorInstruction` with explicit positive and negative routing boundaries resolves the collision.

**Distractor Analysis:**
- **Why A is incorrect:** Collapsing a modular multi-agent architecture into a monolith is unnecessary when collaborator instructions simply need scope disambiguation.
- **Why B is incorrect:** `enableTrace` is an observability flag and has zero impact on routing classifier probabilities.
- **Why C is incorrect:** The supervisor routes to a sub-agent *before* the sub-agent queries its own Knowledge Base vector store.
</details>

---

### Question 6
An enterprise tax advisory application exposes a synchronous REST endpoint via **Amazon API Gateway (REST API)** backed by an AWS Lambda function that invokes an Amazon Bedrock Agent (`InvokeAgent`). For complex corporate tax filings, the Bedrock Agent performs 4 sequential orchestration turns (querying two Knowledge Bases and running two tax-calculation Action Group tools), taking a total of **34 seconds** to complete.

In production, whenever a query takes longer than **29 seconds**, the user's browser immediately receives an **`HTTP 504 Gateway Timeout`** error from Amazon API Gateway, even though CloudWatch Logs show that the Lambda function and Bedrock Agent finished successfully at the 34-second mark!

Which two architectural patterns will eliminate this 29-second timeout bottleneck while delivering a responsive user experience? **(Choose TWO.)**

- **A.** Change the Lambda function memory allocation from `512 MB` to `10,240 MB` so the Bedrock Agent foundation model generates tokens $10\times$ faster on AWS servers.
- **B.** Refactor the frontend-to-backend communication to use **Response Streaming** via **Amazon API Gateway WebSocket APIs** (or an **AWS Lambda Function URL configured with `InvokeMode: RESPONSE_STREAM`** / **AWS AppSync Event Subscriptions**) and stream both intermediate agent trace status updates and output chunks (`completion` event stream) incrementally to the client as they arrive.
- **C.** Switch the API Gateway endpoint type from `REGIONAL` to `EDGE_OPTIMIZED` so CloudFront caches the 34-second POST request.
- **D.** Set the Lambda function timeout to `3 seconds` so it fails before API Gateway reaches 29 seconds.
- **E.** Implement an **Asynchronous Request-Reply / Polling (or Webhook) Pattern** where the REST API immediately returns `HTTP 202 Accepted` with a `job_id`, enqueues the request into Amazon SQS / Step Functions for background Agent execution, and persists the streamed progress/result in Amazon DynamoDB for client polling or WebSocket push.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: B and E**

**Explanation:**
Amazon API Gateway REST APIs and HTTP APIs enforce a synchronous integration timeout (29 seconds default for REST, 30 seconds hard limit for HTTP APIs). When a multi-step Bedrock Agent workflow takes 34 seconds:
- **B is correct:** Using **API Gateway WebSockets**, **AWS AppSync**, or **Lambda Function URLs with `RESPONSE_STREAM`** allows streaming intermediate progress events (e.g., *"Searching 2026 Tax Code..."*, *"Running AMT Calculation..."*) and token chunks incrementally over a persistent connection without hitting a 29-second blocking HTTP response wall.
- **E is correct:** Decoupling long-running agentic tasks via an asynchronous `HTTP 202 Accepted` + SQS/Step Functions + DynamoDB/WebSocket notification pattern avoids holding synchronous HTTP integrations open altogether.

**Distractor Analysis:**
- **Why A is incorrect:** Increasing the caller Lambda's memory increases local CPU/network in the Lambda container, but 99% of the 34 seconds is spent waiting on remote Amazon Bedrock LLM inference and KB retrieval; Lambda memory has zero effect on Bedrock server-side token generation speed.
- **Why C is incorrect:** Edge-optimized API Gateway still enforces the 29-second origin integration limit, and dynamic `POST` requests to a conversational agent cannot be served from a static CDN cache.
- **Why D is incorrect:** Failing the request at 3 seconds breaks the application for all users instead of fixing the timeout.
</details>

---

### Question 7
A high-volume B2C travel assistant calls `bedrock_runtime.converse` using the regional model ID `anthropic.claude-3-5-sonnet-20241022-v2:0` in `us-east-1`. During a flash sale, traffic spikes $3\times$, and 40% of user requests fail with `ThrottlingException (HTTP 429): Too many tokens per minute`.

When the engineering team inspects Amazon CloudWatch metrics (`InvocationClientErrors`, `InputTokenCount`, `OutputTokenCount`), they notice a puzzling discrepancy:
- Their regional Tokens Per Minute (TPM) quota is **400,000 TPM**.
- Each incoming request has an `InputTokenCount` of **800 tokens**, and the model's actual generated response averages an `OutputTokenCount` of **200 tokens** (1,000 total tokens per request).
- At 120 requests per minute, the actual token consumption is only $120 \times 1,000 = 120,000\text{ TPM}$ (30% of their 400,000 TPM quota), yet Bedrock is throwing `ThrottlingException: Too many tokens per minute`!
- Upon inspecting the `converse` code, the developer finds: `inferenceConfig={"maxTokens": 8192, "temperature": 0.0}`.

Why is `ThrottlingException` occurring at only 120 RPM, and what two actions should the developer take immediately? **(Choose TWO.)**

- **A.** Amazon Bedrock deducts `InputTokenCount + maxTokens` ($800 + 8,192 = 8,992\text{ tokens}$) **upfront** from the regional TPM token bucket at the moment a request arrives, and only refunds the unused output tokens ($8,192 - 200 = 7,992$) after generation finishes; lowering `maxTokens` from `8192` to a realistic ceiling (such as `512` or `1024`) prevents upfront TPM quota exhaustion during concurrent bursts.
- **B.** Increase `maxTokens` from `8192` to `64000` so Bedrock allocates a dedicated GPU cluster.
- **C.** Switch from the `Converse` API to the legacy `InvokeModel` API because `InvokeModel` bypasses regional TPM quotas.
- **D.** Update the `modelId` from the single-region model ID (`anthropic.claude-3-5-sonnet-20241022-v2:0`) to the **Cross-Region Inference (CRIS) Profile ID** (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`) and configure `botocore.config.Config(retries={"max_attempts": 6, "mode": "adaptive"})` so burst traffic seamlessly routes across multiple US Regions with up to $2\times$ effective quota.
- **E.** Set `retries={"max_attempts": 0}` and retry failed requests in a `while True` loop with `time.sleep(0.001)`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and D**

**Explanation:**
- **A is correct:** This is a critical production nuance of Amazon Bedrock rate limiting: when a request arrives, Bedrock does not know in advance whether the model will generate 10 tokens or `maxTokens` (8,192 tokens). Therefore, Bedrock reserves `input_tokens + maxTokens` upfront from your TPM bucket and credits back unused tokens upon completion. At 120 concurrent requests with `maxTokens: 8192`, the upfront reservation spikes to $120 \times 8,992 = 1,079,040\text{ TPM}$, instantly blowing past the 400,000 TPM quota! Right-sizing `maxTokens` to `512`–`1024` immediately eliminates 85% of the phantom quota reservation.
- **D is correct:** Switching to a **Cross-Region Inference Profile (`us.anthropic...`)** dynamically distributes burst traffic across multiple AWS Regions (e.g., `us-east-1`, `us-west-2`, `us-east-2`) at no additional surcharge and provides up to $2\times$ higher burst quotas, while Botocore `adaptive` retry mode implements client-side token-bucket rate limiting and exponential backoff with jitter.

**Distractor Analysis:**
- **Why B is incorrect:** Raising `maxTokens` increases upfront TPM deduction even further and triggers a `ValidationException` if it exceeds the model's maximum output token limit.
- **Why C is incorrect:** `Converse` and `InvokeModel` share the exact same underlying model quotas (`RPM` and `TPM`).
- **Why E is incorrect:** Retrying every 1 millisecond without exponential backoff creates a retry storm that guarantees continuous 429 throttling.
</details>

---

### Question 8
A developer is building a multi-turn document Q&A chat interface using the Amazon Bedrock `Converse` API. To save UI state, whenever a user types a follow-up question quickly before the network responds—or when the application injects a background context reminder—the frontend appends a second `"role": "user"` message to the `messages` array:

```python
messages = [
    {"role": "user", "content": [{"text": "Summarize this contract."}]},
    {"role": "assistant", "content": [{"text": "Here is the summary..."}]},
    {"role": "user", "content": [{"text": "Wait, what about the termination clause?"}]},
    {"role": "user", "content": [{"text": "Specifically section 14.2."}]}
]
```

When the backend passes this `messages` list to `bedrock_runtime.converse`, the call fails with:
`ValidationException: A conversation must alternate between user and assistant roles.`

How should the backend sanitize the `messages` list before invoking `Converse`?

- **A.** Change the role of the fourth message to `"role": "moderator"`.
- **B.** Base64-encode the entire `messages` JSON array and pass it as a single string inside the `system` parameter.
- **C.** Merge consecutive messages that share the same `"role"` into a single message object by concatenating their `content` block arrays (`{"role": "user", "content": [{"text": "Wait, what about the termination clause?"}, {"text": "Specifically section 14.2."}]}`) and ensure the first message in `messages` always has `"role": "user"`.
- **D.** Delete all previous turns from `messages` on every request so `Converse` only ever receives a 1-element list.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
The Amazon Bedrock `Converse` API enforces two strict structural invariants on the `messages` array:
1. The first message (`messages[0]`) **must** have `"role": "user"`.
2. Roles **must strictly alternate** between `"user"` and `"assistant"` (`user` $\rightarrow$ `assistant` $\rightarrow$ `user` $\rightarrow$ `assistant`).
Because each message's `"content"` field is an array of content blocks (`List[ContentBlock]`), multiple consecutive user inputs (e.g., text + uploaded document + follow-up clarification) should be merged into the `content` list of a single `"role": "user"` message object.

**Distractor Analysis:**
- **Why A is incorrect:** `Converse` only accepts `"user"` and `"assistant"` as valid `role` values in `messages`; `"moderator"` triggers a `ValidationException`.
- **Why B is incorrect:** Base64-encoding the conversation into `system` breaks multi-turn conversation role tracking and tool-use state management.
- **Why D is incorrect:** Deleting all prior turns destroys multi-turn conversational memory (so the model no longer knows what `"Specifically section 14.2"` refers to).
</details>

---

### Question 9
An enterprise procurement Bedrock Agent is experiencing intermittent failures where a user asks a vague question like *"Check the status of my vendor onboarding"*, and the Agent enters a repetitive loop—calling the `searchVendorsByName` Action Group 5 times in a row with different guessed strings before finally failing with a `failureTrace` stating that the maximum orchestration iterations were exceeded.

Upon inspecting the Agent trace (`enableTrace=True`), the developer notices that when `searchVendorsByName` returns `{"vendors": []}`, the Agent's `rationale` says: *"No vendor found, let me try guessing another name"* instead of simply asking the user for their exact Vendor Tax ID or Legal Entity Name!

How should the developer configure the Action Group and Agent Instructions to stop the Agent from guessing parameters and force it to ask the user for clarifying information?

- **A.** Increase the maximum orchestration iteration quota from 5 to 500 via AWS Support so the Agent can keep guessing for 10 minutes.
- **B.** Enable **User Confirmation** (`requireConfirmation="ENABLED"`) or update the Agent Instructions and OpenAPI/Function parameter descriptions to explicitly instruct: *"Never guess or fabricate `vendor_name` or `vendor_tax_id`. If the user has not provided an exact Vendor Tax ID or Legal Entity Name, or if `searchVendorsByName` returns an empty list, immediately ask the user for clarification rather than retrying the tool."*
- **C.** Remove the `searchVendorsByName` Action Group and hardcode all vendor records inside the Agent's system prompt.
- **D.** Change the Lambda function to return `httpStatusCode: 500` whenever `vendors` is empty so the Agent crashes on the first turn.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
When a ReAct orchestration loop receives an empty search result (`{"vendors": []}`), an LLM without explicit negative constraints on retry behavior will often attempt "self-correction" by guessing alternative query strings until it hits the maximum iteration limit (`MaxIterationsExceeded`). By explicitly instructing the Agent in its **Agent Instructions** and **Tool/Parameter descriptions** (and returning a helpful hint in the Lambda observation body like `{"vendors": [], "next_step": "No match found. Ask the user to provide their 9-digit Vendor Tax ID."}`), the Agent immediately transitions from tool invocation to asking the user a clarifying question.

**Distractor Analysis:**
- **Why A is incorrect:** Allowing 500 blind guesses wastes massive token quotas and causes extreme latency without finding a vendor name the user never provided.
- **Why C is incorrect:** Hardcoding thousands of dynamic vendor records in a system prompt exceeds context limits and makes real-time status checks impossible.
- **Why D is incorrect:** Returning HTTP 500 for an empty search result causes the Agent to report a system outage rather than asking the user to clarify their vendor name.
</details>

---

### Question 10
A security-conscious insurance firm configured an Amazon Bedrock Agent with **Pre-Processing** enabled (`promptCreationMode="OVERRIDDEN"` or default pre-processing classifier enabled). Legitimate customers who start a conversation with short greetings or polite conversational openers like *"Hi, I need some help with a recent hail storm claim"* are immediately rejected by the Agent with a generic refusal message before the Agent ever checks the Knowledge Base or Action Groups!

When the developer invokes the Agent with `enableTrace=True`, they observe that `orchestrationTrace` is completely absent; only `preProcessingTrace` is emitted, and `preProcessingTrace.modelInvocationOutput.parsedResponse` shows:
```json
{
  "isValid": false,
  "rationale": "The user input does not contain a policy number, date of loss, or specific claim question."
}
```

Why is this happening, and how should the developer fix it?

- **A.** Someone customized the Agent's **Pre-Processing prompt template** with overly rigid validation criteria requiring a policy number and date of loss on the very first turn, causing the Pre-Processing classifier to mark conversational openers as `isValid: false`; the developer should either relax the Pre-Processing prompt criteria to allow conversational greetings/clarification turns or disable the Pre-Processing step and let the Orchestration loop ask the user for their policy number.
- **B.** The Amazon Bedrock Guardrail PII filter blocked the phrase `"hail storm"`.
- **C.** The Action Group Lambda function timed out after 30 seconds.
- **D.** The OpenSearch Serverless Knowledge Base index is corrupted.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
In Amazon Bedrock Agents, the **Pre-Processing** step runs *before* the Orchestration loop to classify whether the user's input is valid and safe to process (`isValid: true | false`). When `parsedResponse.isValid` evaluates to `false`, the Agent short-circuits immediately and never enters `orchestrationTrace`. Because the trace explicitly shows `preProcessingTrace` setting `isValid: false` due to missing policy numbers in the initial greeting, either updating the Pre-Processing prompt template to permit multi-turn information gathering or disabling Pre-Processing (allowing Orchestration + Bedrock Guardrails to handle input validation and slot-filling) resolves the issue.

**Distractor Analysis:**
- **Why B is incorrect:** If a Bedrock Guardrail had blocked the input, `guardrailTrace` would show `action: "INTERVENED"`, not `preProcessingTrace` returning `isValid: false`.
- **Why C and D are incorrect:** Neither Action Group Lambdas nor Knowledge Bases are invoked until the Agent reaches `orchestrationTrace`.
</details>

---

### Question 11
A developer is implementing a conversational memory manager for a long-running coding assistant using the Amazon Bedrock `Converse` API. To keep input token counts below 16,000 tokens, the developer wrote a sliding-window truncation function that simply keeps the last 10 messages in the `messages` list:
`messages = messages[-10:]`

In production, whenever a user conversation exceeds 10 messages and the truncation slice happens to cut right between Message 8 (`role: "assistant"` containing a `toolUse` block) and Message 9 (`role: "user"` containing the corresponding `toolResult` block)—so that the new `messages[0]` is Message 9 containing an orphaned `toolResult` block—the very next `Converse` API call crashes with a `ValidationException`!

How must the developer redesign the sliding-window truncation algorithm to prevent `ValidationException` errors?

- **A.** Always slice at an odd index so `messages[0]` has `"role": "assistant"`.
- **B.** Strip all `toolUseId` fields from the remaining messages after slicing.
- **C.** Pass truncated messages inside `additionalModelRequestFields` instead of `messages`.
- **D.** Ensure the sliding-window truncation boundary is **turn-aware and tool-pair atomic**: never slice between an `assistant` `toolUse` message and its subsequent `user` `toolResult` message, and always advance the window start index until `messages[0]` is a standard `"role": "user"` message that does **not** contain orphaned `toolResult` blocks.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
In the Amazon Bedrock `Converse` API:
1. `messages[0]` **must** be a `"role": "user"` message.
2. A `"role": "user"` message containing a `toolResult` block is **only valid** if it is immediately preceded by an `"role": "assistant"` message containing the matching `toolUse` block (`toolUseId`).
Naive array slicing (`messages[-10:]`) can easily make a `user` `toolResult` message the first element of the array (orphaning it from its preceding `assistant` `toolUse` message) or make an `assistant` message the first element. A production-safe sliding-window trimmer must treat `[assistant(toolUse), user(toolResult)]` as an indivisible atomic pair and ensure `messages[0]` starts on a clean `user` prompt message.

**Distractor Analysis:**
- **Why A is incorrect:** Starting `messages[0]` with `"role": "assistant"` immediately triggers a `ValidationException`.
- **Why B is incorrect:** Removing `toolUseId` from `toolUse` or `toolResult` blocks violates the `Converse` JSON schema and triggers a `ValidationException`.
- **Why C is incorrect:** `additionalModelRequestFields` is for model-specific inference parameters (like `top_k`), not conversation history.
</details>

---

### Question 12
A logistics company uses an Amazon Bedrock Agent with an Action Group that calls a legacy on-premises ERP SOAP service via an AWS Lambda function inside a VPC. Under heavy warehouse load, the legacy ERP occasionally takes **195 seconds** to compute a global inventory reconciliation report. Even though the developer configured the AWS Lambda function timeout to `300 seconds` (5 minutes), the Bedrock Agent terminates the tool invocation earlier and fails with a `DependencyFailedException` / timeout in `failureTrace`.

Why did the Bedrock Agent fail before the 300-second Lambda timeout elapsed, and how should the architecture be redesigned?

- **A.** AWS Lambda functions inside a VPC are limited to a maximum execution time of 15 seconds.
- **B.** The developer must enable Provisioned Concurrency on the Lambda function, which increases the Bedrock Agent synchronous wait limit to 60 minutes.
- **C.** Amazon Bedrock Agents enforce a service quota timeout on synchronous Action Group Lambda invocations (defaulting to 30–180 seconds depending on region/limit), making multi-minute synchronous ERP jobs unsuitable for direct blocking tool calls; the developer should refactor the Action Group into two asynchronous tools (`startInventoryReconciliationJob` returning a `job_id` in $<1\text{ second}$, and `checkReconciliationJobStatus`) or use **Return of Control (`RETURN_CONTROL`)** with an asynchronous orchestration workflow.
- **D.** The developer must switch the Bedrock Agent foundation model to `amazon.titan-text-lite-v1`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
Blocking a synchronous LLM agent tool invocation for 3+ minutes is an architectural anti-pattern that hits Bedrock Agent Lambda invocation timeout limits and holds open client connections. By refactoring slow backend operations into an **Asynchronous Job Pattern** (`start_job` $\rightarrow$ returns `job_id` immediately; `get_job_status` $\rightarrow$ checks status or notifies via `RETURN_CONTROL`/webhook), each Action Group Lambda invocation completes in milliseconds while handling arbitrarily long backend jobs gracefully.

**Distractor Analysis:**
- **Why A is incorrect:** VPC-attached Lambda functions support up to 15 minutes (900 seconds) execution timeout, just like non-VPC Lambdas.
- **Why B is incorrect:** Lambda Provisioned Concurrency eliminates cold starts; it does not change Bedrock Agent synchronous tool invocation timeout limits.
- **Why D is incorrect:** Changing the foundation model does not change the Action Group execution timeout limit.
</details>

---

### Question 13
A media analytics platform processes user-uploaded PDFs and images using the Amazon Bedrock `Converse` API (`document` and `image` content blocks). In production, roughly 3% of customer requests fail immediately with:
`ValidationException: Document name contains invalid characters` or `ValidationException: Image size exceeds maximum allowed limit`.

Upon inspecting the failing requests, the developer notices two patterns:
1. Users upload PDFs with filenames like `"Q4_Financials (Final) [v2.1].pdf"`, and the code passes the raw filename directly into `document={"name": raw_filename, "format": "pdf", "source": {"bytes": pdf_bytes}}`.
2. Users upload uncompressed 12 MB smartphone photos directly into `image={"format": "jpeg", "source": {"bytes": img_bytes}}`.

How should the developer remediate both `ValidationException` failures?

- **A.** Rename all PDF files to include `.pdf.exe` at the end of `document.name` and convert 12 MB JPEGs into uncompressed 40 MB BMP files.
- **B.** Sanitize the `document.name` field using a regex so it contains only alphanumeric characters, single spaces, hyphens, parentheses, and square brackets without file extensions or consecutive whitespace (or normalize to a clean slug like `"Q4-Financials-Final-v2-1"`), and add a pre-processing step that downscales/compresses images to stay under the `Converse` API's per-image byte and dimension limits (e.g., $\le 3.75\text{ MB}$ / $5\text{ MB}$ and $\le 8000 \times 8000\text{ px}$) before calling `Converse`.
- **C.** Pass the raw 12 MB binary image bytes directly inside the `system` prompt string as UTF-8 text.
- **D.** Retry the exact same `ValidationException` request 10 times using Botocore adaptive retry mode.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
In the Amazon Bedrock `Converse` API:
1. **`document.name` Constraints**: The `name` field of a `DocumentBlock` has strict character validation rules (to prevent prompt-injection via filenames) and must **not** include periods/file extensions (like `.pdf` or `.2.1`) or consecutive whitespace. Normalizing the filename to an alphanumeric/hyphenated slug (`re.sub(r"[^a-zA-Z0-9\-\s]", "-", stem)`) prevents filename `ValidationException` errors.
2. **Image Payload Constraints**: The `Converse` API enforces strict per-image byte limits (3.75 MB raw / 5 MB depending on model) and pixel dimension bounds. Resizing and compressing oversized images client-side or in preprocessing prevents payload `ValidationException` errors.

**Distractor Analysis:**
- **Why A is incorrect:** Including periods (`.pdf.exe`) in `document.name` and increasing image size to 40 MB guarantees a `ValidationException`.
- **Why C is incorrect:** Binary JPEG bytes are not valid UTF-8 text and cannot be passed in `system`.
- **Why D is incorrect:** `ValidationException` (HTTP 400) is a deterministic client error; retrying an invalid payload 10 times will fail 10 times.
</details>

---

### Question 14
A healthcare claims processor uses an Amazon Bedrock Agent with a Guardrail attached (`guardrailConfiguration`). During testing, whenever an agent processes a claim containing a US Social Security Number (SSN) and a legitimate 10-digit National Provider Identifier (NPI) number (e.g., `1234567890`), the user receives an output where the SSN is properly masked as `{SSN}`, **and** the doctor's public 10-digit NPI number is falsely masked as `{PHONE}` or `{US_BANK_ACCOUNT_NUMBER}`, breaking downstream claim adjudication!

How can the developer troubleshoot which specific Guardrail filter caused the false-positive masking on the 10-digit NPI number and fix it?

- **A.** Inspect `guardrailTrace` (`outputAssessments[].sensitiveInformationPolicy.piiEntities` and `regexes`) to identify the exact built-in PII entity type (`PHONE` or `US_BANK_ACCOUNT_NUMBER`) that matched the 10-digit NPI number, adjust the built-in PII entity configuration (or switch to a custom contextual regex pattern that requires prefixes like `NPI:` vs phone formatting), and verify the fix against a test set with `ApplyGuardrail`.
- **B.** Disable the Bedrock Guardrail entirely in production so SSNs and NPIs are both returned in cleartext.
- **C.** Increase the Knowledge Base chunk size to 2,000 tokens so PII entities are split across chunks.
- **D.** Change the Agent's IAM Execution Role trust policy.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
When a Bedrock Guardrail masks or blocks content unexpectedly, inspecting **`guardrailTrace`** (specifically `inputAssessments` and `outputAssessments` $\rightarrow$ `sensitiveInformationPolicy`) reveals the exact `piiEntities` entry (`type`, `match`, `action: "ANONYMIZED"`) or custom `regexes` rule that fired. Because a bare 10-digit number (`1234567890`) can overlap with unformatted 10-digit US phone numbers or bank account numbers, identifying the exact colliding `piiEntities.type` in `guardrailTrace` allows the developer to replace overly broad numeric PII entity types with targeted custom regular expressions while keeping `US_SOCIAL_SECURITY_NUMBER` masking active.

**Distractor Analysis:**
- **Why B is incorrect:** Disabling the Guardrail exposes raw patient SSNs, violating HIPAA/PII security mandates.
- **Why C is incorrect:** Chunk size has no effect on output PII masking performed by Bedrock Guardrails.
- **Why D is incorrect:** IAM trust policies control authentication, not Guardrail PII entity detection.
</details>

---

### Question 15
A global financial news platform runs a critical real-time summarization service on Amazon Bedrock in `us-east-1`. To meet a 99.99% availability SLA and protect against rare regional service degradation (`ServiceUnavailableException` HTTP 503 or `ModelTimeoutException` HTTP 408), the principal architect wants to design a multi-layered resiliency architecture that handles both **regional infrastructure impairments** and **model-family capacity saturation**.

Which architecture provides the highest availability and lowest failover latency?

- **A.** Hardcode `us-east-1` with a single foundation model, set `read_timeout=3600` (1 hour), and open an AWS Support ticket whenever an HTTP 503 error occurs.
- **B.** Store a static copy of yesterday's news summary in Amazon S3 and return yesterday's news whenever a user submits a new article during an outage.
- **C.** Configure the primary invocation path to use an **Amazon Bedrock Cross-Region Inference (CRIS) Profile** (e.g., `us.anthropic.claude-3-5-sonnet-20241022-v2:0`) with aggressive `read_timeout` bounds and Botocore adaptive retries, and wrap the client in a circuit-breaker fallback handler that automatically fails over to a secondary heterogeneous CRIS model family (e.g., `us.amazon.nova-pro-v1:0`) if the primary model family returns `503 ServiceUnavailableException` or `408 ModelTimeoutException`.
- **D.** Run `StartIngestionJob` on every user request to check if OpenSearch Serverless is healthy before calling `Converse`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
A resilient tier-1 production GenAI architecture defends against two distinct failure domains:
1. **Single-Region Capacity / Infrastructure Bottlenecks**: Using a **Cross-Region Inference (CRIS) Profile** (`us.*` or `eu.*`) allows Amazon Bedrock to automatically route requests across multiple AWS Regions within the geography without requiring the customer to manage multi-region VPC peering or duplicate endpoints.
2. **Model-Family / Provider-Specific Saturation**: Implementing a bounded `read_timeout` (e.g., 15–30 seconds) with a circuit breaker that fails over to a **secondary model family** (`us.amazon.nova-pro-v1:0`) via the unified `Converse` API ensures that even if `Claude 3.5 Sonnet` experiences a cross-region spike or degradation, user requests continue succeeding within SLA.

**Distractor Analysis:**
- **Why A is incorrect:** Setting a 1-hour `read_timeout` causes worker threads to hang indefinitely during network stalls and provides zero failover capability.
- **Why B is incorrect:** Returning yesterday's static news summary when a user submits a new article returns completely fabricated/wrong content for the user's input.
- **Why D is incorrect:** `StartIngestionJob` is an asynchronous batch indexing control-plane API with strict rate limits; calling it on every user request would immediately throttle and break the system.
</details>
