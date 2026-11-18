# Domain 3 — Quiz 2: Data Security, Privacy Controls, KMS, PrivateLink, & Multi-Tenant RAG (Task 3.2)

**Instructions:** This 15-question Staff/Principal-level diagnostic quiz tests **Task 3.2 (Implement Data Security and Privacy Controls)** of the AWS Certified Generative AI Developer - Professional (AIP-C01) exam. Questions include single-select (`A`–`D`) and multiple-response (`Choose TWO` from `A`–`E`) items.

---

### Question 1
A SaaS provider serves 400 enterprise customers using a single shared **Amazon Bedrock Knowledge Base** backed by an **Amazon OpenSearch Serverless** vector collection. Every document ingested into the Knowledge Base has a companion `.metadata.json` file containing `"tenant_id"` and `"department"`. Users authenticate to a React SPA via **Amazon Cognito**, which issues an ID JWT containing a verified custom attribute `custom:tenant_id`. A security penetration test discovers that the current API accepts `{"query": "...", "tenant_id": "acme-corp"}` in the HTTP POST body from the browser and passes `body.tenant_id` into the `RetrieveAndGenerate` filter—allowing an attacker to modify `"tenant_id"` in Burp Suite and read competitor documents.

How should the Principal Security Architect remediate this cross-tenant data leakage vulnerability without provisioning 400 separate OpenSearch Serverless collections?

- **A.** Remove `"tenant_id"` from the client HTTP POST body entirely. Configure an Amazon Cognito User Pool Authorizer on Amazon API Gateway, extract the cryptographically verified `event["requestContext"]["authorizer"]["claims"]["custom:tenant_id"]` inside the backend Lambda function, and inject that value into `retrievalConfiguration.vectorSearchConfiguration.filter` (`{"equals": {"key": "tenant_id", "value": verified_tenant_id}}`).
- **B.** Keep `"tenant_id"` in the HTTP POST body, but append a system prompt instruction in `RetrieveAndGenerate` stating: `"CRITICAL SECURITY RULE: Only synthesize information from chunks whose metadata matches the user's company."`
- **C.** Enable an Amazon Bedrock Guardrail with `PROMPT_ATTACK` set to `HIGH` and `ContextualGroundingPolicy` set to `0.95` to prevent cross-tenant parameter tampering in the HTTP POST body.
- **D.** Attach an OpenSearch Serverless Network Policy with `AllowFromPublic: false` so users cannot modify JSON fields in the browser request.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: A**

#### Explanation
In a pooled multi-tenant RAG architecture using metadata filtering, the tenant identifier used to construct `retrievalConfiguration.vectorSearchConfiguration.filter` **must originate from a trusted, server-side, cryptographically verified identity context** (such as Amazon API Gateway's Cognito Authorizer claims or an IAM Session Policy), never from client-supplied request parameters or prompt text. Filtering at the vector search layer ensures OpenSearch Serverless only returns vector chunks matching `verified_tenant_id` before any context reaches the foundation model.

#### Distractor Analysis
- **Why B is incorrect:** System prompt instructions run *after* vector retrieval has already fetched cross-tenant chunks and cannot stop parameter tampering or prompt injection.
- **Why C is incorrect:** Bedrock Guardrails inspect natural-language text for jailbreaks and grounding; they cannot detect broken object-level authorization (BOLA) when a user changes `"tenant_id"` in an API payload.
- **Why D is incorrect:** Network policies control VPC network reachability to OpenSearch Serverless, not application-level HTTP request body authorization.

</details>

---

### Question 2
A healthcare network runs a clinical summarization microservice on AWS Fargate inside private VPC subnets (`10.50.0.0/16`) in `us-east-1` with **no Internet Gateway and no NAT Gateway**. The microservice must invoke `anthropic.claude-3-5-sonnet-20241022-v2:0` using the `Converse` API and query an **Amazon Bedrock Knowledge Base** using the `Retrieve` API. Additionally, the corporate network security team mandates a perimeter control ensuring that no workload inside the VPC can ever use the VPC network path to invoke any foundation model other than `anthropic.claude-3-5-sonnet-20241022-v2:0` or access resources outside AWS Organization `o-healthorg777`, even if a developer configures an overly permissive `*` IAM role inside the account.

Which network architecture meets all of these requirements?

- **A.** Provision an AWS Direct Connect public VIF to the Amazon Bedrock public regional endpoints and attach an AWS WAF WebACL to the Direct Connect router restricting the `modelId` header.
- **B.** Provision AWS PrivateLink Interface VPC Endpoints for both `com.amazonaws.us-east-1.bedrock-runtime` and `com.amazonaws.us-east-1.bedrock-agent-runtime` with Private DNS enabled, and attach a **VPC Endpoint Policy** to `bedrock-runtime` scoping `bedrock:InvokeModel*` to the `anthropic.claude-3-5-sonnet-20241022-v2:0` ARN with a condition `"StringEquals": {"aws:PrincipalOrgID": "o-healthorg777"}`.
- **C.** Provision a VPC Gateway Endpoint for `com.amazonaws.us-east-1.bedrock` and add a route in the private subnet route table pointing `0.0.0.0/0` to the Gateway Endpoint.
- **D.** Provision an Interface VPC Endpoint for `com.amazonaws.us-east-1.bedrock` (control plane only) and use NACLs to filter the foundation model ARN at Layer 4.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: B**

#### Explanation
To invoke `Converse`/`InvokeModel` and `Retrieve`/`RetrieveAndGenerate` privately without internet routing, you must provision **Interface VPC Endpoints (AWS PrivateLink)** for **`bedrock-runtime`** (model inference data plane) and **`bedrock-agent-runtime`** (Knowledge Base & Agent runtime data plane). Attaching a **VPC Endpoint Policy** restricting the allowed `Resource` ARN (`anthropic.claude-3-5-sonnet-20241022-v2:0`) and `aws:PrincipalOrgID` enforces a hard network-perimeter boundary regardless of how permissive a caller's IAM role might be.

#### Distractor Analysis
- **Why A is incorrect:** Direct Connect Public VIF routes to public IP space rather than private VPC endpoints, and AWS WAF cannot be attached directly to a Direct Connect router or Bedrock API endpoint.
- **Why C is incorrect:** Gateway Endpoints only exist for Amazon S3 and Amazon DynamoDB; all Amazon Bedrock endpoints are Interface Endpoints (`AWS PrivateLink`).
- **Why D is incorrect:** `com.amazonaws.us-east-1.bedrock` is the control plane endpoint (for `CreateGuardrail`, `ListFoundationModels`), not the runtime data plane (`bedrock-runtime` / `bedrock-agent-runtime`), and Layer 4 NACLs only inspect IP/port, not IAM ARNs.

</details>

---

### Question 3
A financial institution uses an **Amazon Bedrock Agent** to assist commercial loan officers. When a loan officer invokes the Agent via `InvokeAgent`, the backend application needs to pass two distinct pieces of context:
1. The loan officer's preferred output currency (`"USD"`) and today's date (`"2026-11-16"`), which the foundation model should actively use when formatting its reasoning and responses.
2. A short-lived internal OAuth 2.0 Bearer Token and the officer's `employee_clearance_guid`, which the Action Group AWS Lambda function needs to authenticate against the core banking mainframe, but which **must never be exposed to the foundation model's prompt context** where a prompt injection attack could trick the LLM into printing the token in the chat response.

How should the engineer pass these two sets of attributes in the `InvokeAgent` API call?

- **A.** Pass both sets of attributes inside `sessionState.promptSessionAttributes` and enable KMS encryption on the Agent.
- **B.** Pass the OAuth Bearer Token and `employee_clearance_guid` inside `sessionState.promptSessionAttributes`, and pass the preferred currency and date inside `sessionState.sessionAttributes`.
- **C.** Pass the preferred currency and today's date inside `sessionState.promptSessionAttributes`, and pass the OAuth Bearer Token and `employee_clearance_guid` inside `sessionState.sessionAttributes`.
- **D.** Append the OAuth Bearer Token to the `inputText` field wrapped in XML `<secret>` tags and instruct the Agent's orchestration prompt to strip `<secret>` tags before responding.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: C**

#### Explanation
Amazon Bedrock Agents provides two separate state maps in `InvokeAgent.sessionState`:
- **`promptSessionAttributes`**: Exposed directly to the foundation model inside the `$prompt_session_attributes$` placeholder in orchestration/pre-processing/post-processing prompts. Suitable for non-sensitive contextual hints like current date, timezone, or preferred currency.
- **`sessionAttributes`**: Persisted across the session and forwarded directly in the JSON event to Action Group Lambda functions (`event["sessionAttributes"]`), **without ever being injected into the foundation model prompt**. Essential for passing credentials, OAuth tokens, and verified identity claims safely past the LLM.

#### Distractor Analysis
- **Why A is incorrect:** Putting the OAuth token in `promptSessionAttributes` injects it directly into the LLM prompt context, making it vulnerable to prompt-injection exfiltration.
- **Why B is incorrect:** Reverses the two attribute types, exposing the secret token to the LLM while hiding the date/currency from the LLM.
- **Why D is incorrect:** Passing secrets in `inputText` directly exposes them to the LLM and Model Invocation Logs.

</details>

---

### Question 4
A security engineer is configuring an IAM Execution Role (`AppBedrockCallerRole`) in `us-east-1` (`Account 111122223333`) for an application that invokes the **US Cross-Region Inference Profile** `us.anthropic.claude-3-5-sonnet-20241022-v2:0` to dynamically route traffic across `us-east-1`, `us-east-2`, and `us-west-2` for high availability. The engineer attaches the following least-privilege IAM policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"],
      "Resource": "arn:aws:bedrock:us-east-1:111122223333:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0"
    }
  ]
}
```
When the application runs `converse(modelId="us.anthropic.claude-3-5-sonnet-20241022-v2:0")`, every request fails immediately with `AccessDeniedException`.

Why is `AccessDeniedException` occurring, and how must the IAM policy be updated?

- **A.** Cross-Region Inference Profiles require `bedrock:CreateInferenceProfile` permission on `*` for every runtime `Converse` call.
- **B.** Cross-Region Inference Profiles can only be invoked by the AWS Account Root user or roles with `AdministratorAccess`.
- **C.** The `Action` must be changed from `bedrock:InvokeModel` to `bedrock:ConverseCrossRegion`.
- **D.** Invoking a Cross-Region Inference Profile requires `bedrock:InvokeModel` permission on **both** the Inference Profile ARN in the requesting region (`us-east-1`) **and** the underlying Foundation Model ARNs (`arn:aws:bedrock:<region>::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0`) in every destination region included in that inference profile (`us-east-1`, `us-east-2`, and `us-west-2`).

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: D**

#### Explanation
When you invoke a Cross-Region Inference Profile (`us.*` or `eu.*`), Amazon Bedrock first authorizes your call against the **Inference Profile ARN** in the source region and then routes and authorizes the underlying model execution against the **Foundation Model ARN** in whichever target region (`us-east-1`, `us-east-2`, or `us-west-2`) serves the request. If the IAM policy omits the underlying regional Foundation Model ARNs, authorization fails with `AccessDeniedException`.

#### Distractor Analysis
- **Why A is incorrect:** `CreateInferenceProfile` is a control-plane API for creating application inference profiles, not required at runtime.
- **Why B is incorrect:** Standard least-privilege IAM roles can invoke Cross-Region Inference Profiles once both the profile ARN and destination FM ARNs are granted.
- **Why C is incorrect:** There is no `bedrock:ConverseCrossRegion` IAM action; both `InvokeModel` and `Converse` authorize against `bedrock:InvokeModel` / `bedrock:InvokeModelWithResponseStream`.

</details>

---

### Question 5
A biotech firm is building a RAG pipeline that ingests 2 million historical clinical trial transcripts into an Amazon Bedrock Knowledge Base. Because of European GDPR "Right to Erasure" and US HIPAA Privacy Rule mandates, the security architect wants to ensure that **Protected Health Information (PHI) and Personally Identifiable Information (PII)**—such as patient names, Social Security Numbers, phone numbers, and dates of birth—are discovered and redacted **before** the text is chunked, converted into dense vector embeddings by `amazon.titan-embed-text-v2:0`, and stored in Amazon OpenSearch Serverless.

Why is pre-ingestion PII/PHI redaction required in addition to runtime Bedrock Guardrails, and which AWS architecture implements it effectively?

- **A.** Vector embeddings are automatically decrypted by client browsers; therefore, the team should use AWS Certificate Manager (ACM) to redact PII before ingestion.
- **B.** Once raw PII is embedded into dense high-dimensional vectors and stored in chunk metadata in OpenSearch Serverless, sensitive details reside at rest in the vector index (vulnerable to embedding inversion or retrieval bugs) and require expensive re-indexing to honor deletion requests; the team should build an ingestion pipeline using **Amazon Comprehend (`DetectPiiEntities` / Comprehend Medical `DetectPHI`)** to mask sensitive spans in a curated S3 bucket **before** triggering Bedrock Knowledge Base ingestion.
- **C.** Amazon Bedrock Guardrails cannot mask PII in outputs from Knowledge Bases; therefore, the team must manually review all 2 million transcripts using Amazon Mechanical Turk.
- **D.** OpenSearch Serverless rejects any document containing an `@` symbol or 9-digit number during `StartIngestionJob`.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: B**

#### Explanation
Sanitizing unstructured documents **prior to embedding generation** prevents sensitive PII/PHI from ever being encoded into vector representations or stored in the `AMAZON_BEDROCK_TEXT_CHUNK` field inside OpenSearch Serverless. An AWS Step Functions or Lambda pre-processing pipeline invoking **Amazon Comprehend (`DetectPiiEntities`)** or **Amazon Comprehend Medical (`DetectPHI`)** redacts the raw text and writes clean documents to the Knowledge Base source S3 bucket, ensuring both defense-in-depth and compliance with data minimization / right-to-erasure regulations.

#### Distractor Analysis
- **Why A is incorrect:** ACM manages TLS certificates for encryption in transit; it does not inspect or redact PII in text documents.
- **Why C is incorrect:** Bedrock Guardrails *can* mask PII at runtime, and public Amazon Mechanical Turk workers must never be given access to unredacted HIPAA PHI.
- **Why D is incorrect:** OpenSearch Serverless does not block documents containing `@` symbols or numbers during ingestion.

</details>

---

### Question 6
A defense contractor is creating an **Amazon Bedrock Guardrail** and an **Amazon Bedrock Knowledge Base** in `us-east-1` (`Account 222233334444`) and specifies a KMS Customer Managed Key (`arn:aws:kms:us-east-1:222233334444:key/cmk-defense-01`) for encryption at rest. To prevent **cross-service Confused Deputy attacks**—where an attacker in another AWS account tricks `bedrock.amazonaws.com` into using the defense contractor's KMS key to decrypt data—the security engineer must add a statement to the KMS Key Policy granting permissions to `"Service": "bedrock.amazonaws.com"`.

Which IAM Condition block in the KMS Key Policy statement provides the strongest protection against Confused Deputy attacks?

- **A.** `"Condition": {"Bool": {"aws:SecureTransport": "true"}}`
- **B.** `"Condition": {"StringEquals": {"aws:PrincipalType": "Service"}}`
- **C.** `"Condition": {"StringEquals": {"aws:SourceAccount": "222233334444"}, "ArnLike": {"aws:SourceArn": ["arn:aws:bedrock:us-east-1:222233334444:guardrail/*", "arn:aws:bedrock:us-east-1:222233334444:knowledge-base/*"]}}`
- **D.** `"Condition": {"IpAddress": {"aws:SourceIp": "10.0.0.0/8"}}`

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: C**

#### Explanation
When granting an AWS service principal (`"Service": "bedrock.amazonaws.com"`) permission in a resource-based policy (such as a KMS Key Policy, S3 Bucket Policy, or Lambda Resource Policy), AWS security best practices mandate combining **`aws:SourceAccount`** and **`aws:SourceArn`** in the `Condition` block. This guarantees that `bedrock.amazonaws.com` can only invoke `kms:Encrypt` / `kms:Decrypt` / `kms:GenerateDataKey*` when acting specifically on behalf of a Guardrail or Knowledge Base resource owned by Account `222233334444`.

#### Distractor Analysis
- **Why A is incorrect:** `aws:SecureTransport` only verifies that the KMS API call used HTTPS; a confused deputy call from `bedrock.amazonaws.com` triggered by another account also uses HTTPS.
- **Why B is incorrect:** Every call from `bedrock.amazonaws.com` has `PrincipalType == Service`, regardless of which AWS account initiated the workload.
- **Why D is incorrect:** Internal AWS service-to-service calls from `bedrock.amazonaws.com` to KMS do not originate from the customer's private VPC CIDR (`10.0.0.0/8`).

</details>

---

### Question 7
An enterprise is deploying an **Amazon Bedrock Agent** that invokes an Action Group backed by an AWS Lambda function (`arn:aws:lambda:us-east-1:111122223333:function:ProcessRefundAction`) to issue customer refunds up to \$5,000. During testing, when the Agent attempts to invoke the Action Group, the Agent trace shows an immediate permission failure invoking the Lambda function, even though the Agent's IAM Service Role (`BedrockAgentServiceRole`) has an identity policy allowing `lambda:InvokeFunction` on `ProcessRefundAction`. Furthermore, product management wants to ensure that whenever the Agent decides to call `ProcessRefundAction`, the Agent pauses and requires the human customer service representative in the UI to click **"Approve Refund"** before the Lambda function executes.

What combination of changes resolves the permission error and implements the human approval gate?

- **A.** Add a **Lambda resource-based policy** (`aws lambda add-permission`) on `ProcessRefundAction` granting `lambda:InvokeFunction` to principal `bedrock.amazonaws.com` conditioned on `AWS:SourceAccount` and the Agent's `AWS:SourceArn`; and set **`requireConfirmation: "ENABLED"`** on the `ProcessRefundAction` function definition in the Agent Action Group.
- **B.** Attach `AWSLambdaBasicExecutionRole` to the Bedrock Agent Service Role, and set `temperature: 0.0` in the Agent orchestration prompt so it asks the user for permission in plain text.
- **C.** Change the Lambda function's execution role trust policy to trust `bedrock.amazonaws.com`, and configure an Amazon Bedrock Guardrail Word Filter for the word `"refund"`.
- **D.** Place the Lambda function in a public subnet and enable `returnControl` by deleting the Lambda function ARN from the Action Group.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: A**

#### Explanation
1. **Lambda Invocation Permission**: For an Amazon Bedrock Agent to invoke an Action Group Lambda function, AWS requires a **resource-based policy on the Lambda function itself** granting `lambda:InvokeFunction` to the service principal `bedrock.amazonaws.com` (scoped to `AWS:SourceAccount` and `AWS:SourceArn: arn:aws:bedrock:region:account:agent/agent-id`).
2. **Explicit Human Confirmation**: Setting **`requireConfirmation: "ENABLED"`** on the Action Group function schema instructs the Bedrock Agent runtime to pause before executing the Lambda function and return a confirmation prompt parameters payload in the `InvokeAgent` response stream (`returnControl` / confirmation flow), executing the tool only after the caller confirms `"CONFIRM"`.

#### Distractor Analysis
- **Why B is incorrect:** `AWSLambdaBasicExecutionRole` only grants CloudWatch Logs permissions, and relying on LLM prompt instructions for confirmation is non-deterministic and can be bypassed by prompt injection.
- **Why C is incorrect:** The Lambda function's *execution role* must trust `lambda.amazonaws.com` (not `bedrock.amazonaws.com`); it is the Lambda *resource-based policy* that grants invocation rights to `bedrock.amazonaws.com`.
- **Why D is incorrect:** Deleting the Lambda ARN removes server-side Lambda execution entirely rather than adding a confirmation gate on the Lambda Action Group.

</details>

---

### Question 8
A cloud platform team is provisioning an **Amazon OpenSearch Serverless (AOSS)** vector collection (`collection/corp-legal-kb`) to serve as the vector store for a new Amazon Bedrock Knowledge Base. The team creates the AOSS **Encryption Policy** and **Network Policy**, creates the vector collection and vector index, and configures the Bedrock Knowledge Base with IAM Role `arn:aws:iam::111122223333:role/BedrockKBExecutionRole`. Although `BedrockKBExecutionRole` has an IAM identity policy granting `"aoss:APIAccessAll"` on `"arn:aws:aoss:us-east-1:111122223333:collection/*"`, calling `StartIngestionJob` on the Knowledge Base fails with a `403 Forbidden` error when Bedrock attempts to write vector documents into the OpenSearch Serverless index.

What is the root cause of the `403 Forbidden` error?

- **A.** OpenSearch Serverless requires an **AOSS Data Access Policy (`type: data`)** that explicitly lists `arn:aws:iam::111122223333:role/BedrockKBExecutionRole` as a `Principal` and grants index-level permissions (`aoss:ReadDocument`, `aoss:WriteDocument`, `aoss:CreateIndex`, `aoss:DescribeIndex`, `aoss:UpdateIndex`) on `index/corp-legal-kb/*` and collection-level permissions (`aoss:DescribeCollectionItems`) on `collection/corp-legal-kb`.
- **B.** The `BedrockKBExecutionRole` IAM policy must use the action `es:ESHttpPost` instead of `aoss:APIAccessAll`.
- **C.** OpenSearch Serverless vector collections can only be written to by IAM Users with static access keys, not IAM Roles assumed by `bedrock.amazonaws.com`.
- **D.** The AOSS Encryption Policy must be deleted after the collection reaches the `ACTIVE` state so `StartIngestionJob` can write unencrypted float vectors.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: A**

#### Explanation
Amazon OpenSearch Serverless separates **IAM authentication** (`aoss:APIAccessAll` in the IAM identity policy) from **collection/index authorization** governed by an **OpenSearch Serverless Data Access Policy (`type: data`)**. Even if an IAM role has `aoss:APIAccessAll`, every read or write against an AOSS index will fail with `403 Forbidden` unless an AOSS Data Access Policy explicitly grants `aoss:WriteDocument`, `aoss:ReadDocument`, `aoss:DescribeIndex`, etc., to that exact IAM Role ARN.

#### Distractor Analysis
- **Why B is incorrect:** `es:ESHttpPost` applies to managed Amazon OpenSearch Service domains (`es`), whereas OpenSearch Serverless (`aoss`) uses `aoss:APIAccessAll` plus AOSS Data Access Policies.
- **Why C is incorrect:** Bedrock Knowledge Bases always use an IAM Service Role (`bedrock.amazonaws.com`) listed in the AOSS Data Access Policy.
- **Why D is incorrect:** An AOSS Encryption Policy is mandatory and cannot be removed to allow unencrypted storage.

</details>

---

### Question 9
A global investment bank operates three business divisions (`InvestmentBanking`, `WealthManagement`, and `RetailBanking`) within a single AWS account. To prevent conflict-of-interest ("Chinese Wall") violations, each division has its own dedicated Amazon Bedrock Knowledge Base tagged with `Division = InvestmentBanking`, `Division = WealthManagement`, or `Division = RetailBanking`. Each application microservice runs under an IAM Role tagged with the corresponding `Division` principal tag. Instead of maintaining dozens of hardcoded Knowledge Base ARNs across IAM policies whenever new Knowledge Bases are created, the IAM Architect wants a single scalable **Attribute-Based Access Control (ABAC)** IAM policy statement that guarantees a microservice can only call `bedrock:Retrieve` and `bedrock:RetrieveAndGenerate` on Knowledge Bases matching its own `Division` tag, and prohibits callers from modifying tags on Knowledge Bases.

Which IAM policy configuration implements this ABAC boundary securely?

- **A.** Allow `bedrock:*` on `*` with Condition `"StringEquals": {"aws:RequestTag/Division": "${aws:PrincipalTag/Division}"}`.
- **B.** Allow `bedrock:Retrieve` on `*` and pass the `Division` tag inside the `InvokeModel` `temperature` field.
- **C.** Allow `bedrock:Retrieve` and `bedrock:RetrieveAndGenerate` on `arn:aws:bedrock:us-east-1:111122223333:knowledge-base/*` with Condition `"StringEquals": {"aws:ResourceTag/Division": "${aws:PrincipalTag/Division}"}`, and explicitly omit or deny `bedrock:TagResource` and `bedrock:UntagResource` for application roles.
- **D.** Attach an S3 Bucket Policy to the Knowledge Base vector store with `"StringLike": {"s3:prefix": "${aws:PrincipalTag/Division}/*"}` while granting `bedrock:Retrieve` on `*` without conditions.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: C**

#### Explanation
**Attribute-Based Access Control (ABAC)** in AWS IAM matches the caller's principal tag (`${aws:PrincipalTag/Division}`) against the target Bedrock resource's existing tag (`aws:ResourceTag/Division`). Pairing this condition on `bedrock:Retrieve` and `bedrock:RetrieveAndGenerate` with a prohibition on `bedrock:TagResource` / `bedrock:UntagResource` (so a principal cannot re-tag a Knowledge Base to match its own tag) enforces dynamic, conflict-of-interest isolation across divisions.

#### Distractor Analysis
- **Why A is incorrect:** `bedrock:*` grants `TagResource` and administrative actions, and `aws:RequestTag` only evaluates tags passed *during resource creation/tagging requests*, not during runtime `Retrieve` calls.
- **Why B is incorrect:** Model inference hyperparameters (`temperature`) have no relationship to IAM authorization.
- **Why D is incorrect:** When a caller invokes `bedrock:Retrieve`, the Bedrock Knowledge Base service role (not the caller role) reads from the vector store; if `bedrock:Retrieve` is allowed on `*` without conditions, any division can query any Knowledge Base.

</details>

---

### Question 10
A regulated European insurance company is evaluating multi-tenant isolation architectures for its new AI claims processing platform on Amazon Bedrock. While 300 small brokerages share a standard tier, **12 Tier-1 national banks** have signed enterprise contracts containing two non-negotiable legal clauses:
1. **Cryptographic Isolation & Tenant-Controlled Revocation (BYOK)**: Each Tier-1 bank must be able to supply and revoke its own AWS KMS Customer Managed Key (CMK) to immediately render all of its raw documents and vector embeddings unreadable ("crypto-shredding") without impacting any other tenant.
2. **Zero Shared Data-Plane Storage**: Neither raw documents nor vector indexes may reside in a shared S3 bucket or shared OpenSearch Serverless collection alongside other tenants.

Which RAG isolation architecture should the Principal Architect design for the 12 Tier-1 banks?

- **A.** Use a single shared Amazon Bedrock Knowledge Base and single OpenSearch Serverless collection for all 312 tenants, and store each Tier-1 bank's KMS Key ARN as a string attribute in the `.metadata.json` file alongside each document chunk.
- **B.** Provision a **Siloed Per-Tenant Stack** for each of the 12 Tier-1 banks consisting of a dedicated S3 bucket, a dedicated OpenSearch Serverless collection configured with a collection-specific **AOSS Encryption Policy** referencing that tenant's KMS CMK, and a dedicated Bedrock Knowledge Base encrypted with that tenant's KMS CMK.
- **C.** Use a single OpenSearch Serverless collection with 12 separate OpenSearch vector indexes, as AOSS Encryption Policies support assigning a different KMS CMK to each individual index within the same collection.
- **D.** Store all 12 Tier-1 banks' documents in a single S3 bucket encrypted with an AWS-owned key (`SSE-S3`) and use Amazon Macie to delete a tenant's files if they revoke their contract.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: B**

#### Explanation
In Amazon OpenSearch Serverless, **Encryption Policies (`type: encryption`) apply at the Collection level, not the individual index level**. Therefore, to satisfy contractual requirements for both physical storage isolation and per-tenant KMS Customer Managed Key (CMK) revocation (crypto-shredding of vector embeddings at rest), each Tier-1 tenant requires a dedicated OpenSearch Serverless collection, dedicated S3 bucket, and dedicated Knowledge Base configured with that tenant's KMS CMK.

#### Distractor Analysis
- **Why A is incorrect:** Storing a KMS ARN string inside `.metadata.json` does not encrypt the collection with that KMS key or isolate the storage plane.
- **Why C is incorrect:** OpenSearch Serverless Encryption Policies assign one KMS key per *collection*, not per *index* within a shared collection.
- **Why D is incorrect:** Violates both the zero-shared-storage requirement and the BYOK KMS CMK cryptographic revocation requirement.

</details>

---

### Question 11
A software company is fine-tuning `amazon.titan-text-express-v1` (or `amazon.nova-micro-v1:0`) using an Amazon Bedrock Model Customization Job (`CreateModelCustomizationJob`) on proprietary source code and internal architecture documentation stored in Amazon S3. The security team requires that:
1. The training job must run inside a private VPC environment so training data transfer to and from Amazon S3 never traverses the public internet.
2. Both the fine-tuning job's temporary storage/output artifacts and the resulting **Custom Model** weights must be encrypted at rest using the company's KMS Customer Managed Key (CMK).
3. No data submitted to the fine-tuning job or subsequent inference calls may ever be shared with third-party model providers or used to train base foundation models.

Which statement and configuration accurately address these three requirements?

- **A.** Amazon Bedrock fine-tuning jobs cannot run with VPC configurations; the company must export the base model weights to EC2 instances to use VPC isolation and KMS encryption.
- **B.** Pass `customModelKmsKeyId` in `CreateModelCustomizationJob`, and submit an opt-out support ticket to AWS Support within 24 hours of job completion so the fine-tuning dataset is not merged into the public Titan/Nova base model weights.
- **C.** Encrypt the S3 training JSONL file using client-side Base64 encoding before calling `CreateModelCustomizationJob`, because Bedrock Model Customization jobs only support AWS-owned keys.
- **D.** Pass `vpcConfig` (`subnetIds` and `securityGroupIds`) and `customModelKmsKeyId` in the `CreateModelCustomizationJob` API call; by design, Amazon Bedrock never uses customer fine-tuning datasets or prompts to train base foundation models, and keeps custom model weights isolated to the customer's account.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: D**

#### Explanation
`CreateModelCustomizationJob` in Amazon Bedrock natively supports:
1. **`vpcConfig` (`subnetIds`, `securityGroupIds`)**: Protects S3 training and validation data access within your private VPC subnets via S3 VPC Endpoints.
2. **`customModelKmsKeyId`**: Encrypts the resulting custom model weights and transient training artifacts at rest with your KMS Customer Managed Key.
3. **Built-in Data Privacy**: Amazon Bedrock's core service guarantee is that customer training data, fine-tuned weights, and runtime prompts/completions are **never** used to train base foundation models or shared with third-party model providers (no opt-out ticket is required).

#### Distractor Analysis
- **Why A is incorrect:** `CreateModelCustomizationJob` natively supports `vpcConfig` and `customModelKmsKeyId` (and proprietary base model weights cannot be exported to raw EC2).
- **Why B is incorrect:** Bedrock never uses customer data to train base models by default; no opt-out ticket exists or is needed.
- **Why C is incorrect:** Base64 is encoding (not encryption) and would corrupt the JSONL training schema, whereas `customModelKmsKeyId` natively supports KMS CMKs.

</details>

---

### Question 12
A developer is building an internal HR policy chatbot using Amazon Bedrock `Converse`. To prevent developers or compromised application containers from invoking foundation models *without* applying the mandatory corporate HR Guardrail (`arn:aws:bedrock:us-east-1:111122223333:guardrail/gr-hr-mandatory`), the Principal Security Engineer wants to enforce at the **AWS IAM / orchestration boundary** that any model invocation made by the HR application **must** apply `gr-hr-mandatory`.

How should the Principal Security Engineer architect the permissions and invocation path for the HR application?

- **A.** Grant `bedrock:ApplyGuardrail` on `*` and deny `bedrock:InvokeModel` on all foundation models, because `ApplyGuardrail` automatically invokes Claude 3.5 Sonnet.
- **B.** Add `"Condition": {"Bool": {"bedrock:EncryptionEnabled": "true"}}` to the IAM role trust policy.
- **C.** Configure an S3 Bucket Policy on the Model Invocation Logging bucket to reject logs that do not contain a Guardrail trace.
- **D.** Scope the execution role to allow `bedrock:ApplyGuardrail` strictly on `arn:aws:bedrock:us-east-1:111122223333:guardrail/gr-hr-mandatory` and enforce mandatory guardrail invocation via IAM condition constraints / a locked-down inference proxy or Bedrock Agent alias where the guardrail is attached at the resource level and callers only have permission to invoke that protected alias.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: D**

#### Explanation
To guarantee that application callers cannot bypass a mandatory Guardrail, you restrict `bedrock:ApplyGuardrail` exclusively to the approved Guardrail ARN (`gr-hr-mandatory`) and enforce guardrail execution through IAM condition keys or by exposing only a protected Bedrock Agent Alias / Knowledge Base / internal Gateway Lambda where `guardrailConfiguration` is hardcoded on the backend resource and direct unguarded `bedrock:InvokeModel` access is denied to application roles.

#### Distractor Analysis
- **Why A is incorrect:** `ApplyGuardrail` only evaluates text/images against guardrail policies; it does not invoke a foundation model to generate completions.
- **Why B is incorrect:** `bedrock:EncryptionEnabled` is not a valid IAM condition key for enforcing Guardrail attachment.
- **Why C is incorrect:** An S3 Bucket Policy on the logging bucket only causes log delivery failures after inference has already completed.

</details>

---

### Question 13 *(Multiple Response — Choose TWO)*
A financial services enterprise is deploying an Amazon Bedrock Knowledge Base backed by an Amazon OpenSearch Serverless vector collection inside a zero-trust AWS environment. The CISO mandates that:
1. The OpenSearch Serverless vector collection must be encrypted at rest using an enterprise KMS Customer Managed Key (CMK), and neither the vector endpoint nor OpenSearch Dashboards may be accessible from the public internet.
2. The Amazon S3 bucket storing the raw source PDF documents and the Bedrock Model Invocation Logs must reject any HTTP request that does not use TLS 1.2 or higher.

Which **TWO** security configurations must the engineering team implement to satisfy these requirements? *(Choose TWO.)*

- **A.** Create an OpenSearch Serverless **Encryption Policy (`type: encryption`)** specifying the KMS CMK ARN *before* creating the collection, and create an OpenSearch Serverless **Network Policy (`type: network`)** with `AllowFromPublic: false` routing traffic through an `aoss` VPC Endpoint and allowing Bedrock service access.
- **B.** Create the OpenSearch Serverless collection first with default encryption, and then call `UpdateCollectionEncryption` to switch the active collection to the KMS CMK without downtime.
- **C.** Attach an S3 Bucket Policy containing a `Deny` statement on `s3:*` for `Principal: "*"` with Condition `{"Bool": {"aws:SecureTransport": "false"}}` and `{"NumericLessThan": {"s3:TlsVersion": "1.2"}}`.
- **D.** Set `AllowFromPublic: true` in the OpenSearch Serverless Network Policy and rely on the Bedrock Guardrail Word Filter to block unauthorized IP addresses.
- **E.** Store the KMS CMK private key material inside the Bedrock Knowledge Base `.metadata.json` files.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answers: A and C**

#### Explanation
- **A is correct:** In Amazon OpenSearch Serverless, the **Encryption Policy** (specifying the KMS CMK ARN) must be defined prior to collection creation, and the **Network Policy** with `AllowFromPublic: false` restricts access exclusively to specified VPC Endpoints (`com.amazonaws.<region>.aoss`) and AWS service principals (`bedrock.amazonaws.com`).
- **C is correct:** Enforcing TLS 1.2+ on Amazon S3 buckets is achieved via an explicit `Deny` statement in the S3 Bucket Policy checking `aws:SecureTransport == false` and `s3:TlsVersion < 1.2`.

#### Distractor Analysis
- **Why B is incorrect:** You cannot change an OpenSearch Serverless collection's KMS key after the collection has been created.
- **Why D is incorrect:** Setting `AllowFromPublic: true` exposes the collection endpoint publicly, and Bedrock Guardrails do not filter network IP connections to OpenSearch.
- **Why E is incorrect:** KMS key material never leaves AWS KMS HSMs and must never be placed in metadata files.

</details>

---

### Question 14 *(Multiple Response — Choose TWO)*
A healthcare analytics company ingests daily patient feedback surveys from an S3 landing bucket into an Amazon Bedrock Knowledge Base. During a security review, the Data Privacy Officer (DPO) identifies two gaps:
1. Unredacted credit card numbers and Social Security Numbers occasionally appear in the raw S3 landing bucket files, and the company has no automated alerting when sensitive data is uploaded to S3 buckets across the account.
2. Even if an S3 file is later cleaned, employees querying the chatbot sometimes type their own phone numbers or email addresses into the live chat prompt, which should be masked as `[PHONE-1]` and `[EMAIL-1]` before reaching the foundation model.

Which **TWO** AWS controls should the architect combine to address both gaps? *(Choose TWO.)*

- **A.** Enable **Amazon Macie** automated sensitive data discovery jobs across the S3 buckets (with EventBridge alerts to Security Hub) and run an **Amazon Comprehend `DetectPiiEntities`** pre-processing Lambda step to redact PII before copying files to the Knowledge Base source S3 bucket.
- **B.** Configure an **Amazon Bedrock Guardrail Sensitive Information Policy** with `PHONE` and `EMAIL` set to `action: "ANONYMIZE"`, and attach the Guardrail to the `RetrieveAndGenerate` / `Converse` calls.
- **C.** Enable AWS Shield Advanced on the S3 landing bucket to automatically mask credit card numbers inside PDF files.
- **D.** Set `embeddingDataDeliveryEnabled: true` in Bedrock Model Invocation Logging to strip phone numbers from live chat prompts.
- **E.** Use Amazon Route 53 Resolver DNS Firewall to block emails and phone numbers in HTTPS POST bodies.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answers: A and B**

#### Explanation
- **A is correct:** **Amazon Macie** provides continuous automated discovery and classification of sensitive data (PII/financial data) stored in S3 buckets, while an **Amazon Comprehend (`DetectPiiEntities`)** pre-ingestion step redacts sensitive spans before documents are indexed into the Knowledge Base.
- **B is correct:** Configuring `PHONE` and `EMAIL` with `action: "ANONYMIZE"` in an **Amazon Bedrock Guardrail** masks live user input spans with `[PHONE-1]` and `[EMAIL-1]` before the prompt is sent to the foundation model.

#### Distractor Analysis
- **Why C is incorrect:** AWS Shield Advanced protects against DDoS attacks at Layers 3/4/7; it does not scan S3 objects for PII.
- **Why D is incorrect:** Model Invocation Logging records telemetry and never modifies or redacts live prompts.
- **Why E is incorrect:** Route 53 DNS Firewall filters domain name resolution queries, not HTTPS application payloads.

</details>

---

### Question 15 *(Multiple Response — Choose TWO)*
A cloud security architect is configuring **Amazon Bedrock Model Invocation Logging** to deliver full prompt and completion logs to an encrypted **Amazon CloudWatch Log Group** (`/aws/bedrock/modelinvocations`, encrypted with a KMS Customer Managed Key) and an **Amazon S3 Bucket** (`corp-bedrock-audit-logs-prod`, encrypted with the same KMS CMK). After calling `PutModelInvocationLoggingConfiguration`, the architect invokes `anthropic.claude-3-5-sonnet-20241022-v2:0` with a multimodal prompt containing a 4 MB high-resolution architectural diagram image and a text question. However, no logs appear in CloudWatch Logs or S3.

Which **TWO** configuration requirements must be verified and fixed so both text and multimodal image invocation logs are delivered successfully? *(Choose TWO.)*

- **A.** Change the CloudWatch Log Group name to `/aws/lambda/bedrock`, because Bedrock cannot write to log groups prefixed with `/aws/bedrock/`.
- **B.** Disable KMS encryption on both the S3 bucket and the CloudWatch Log Group, because Bedrock Model Invocation Logging only supports unencrypted targets.
- **C.** Ensure `put_model_invocation_logging_configuration` includes `largeDataDeliveryS3Config` inside `cloudWatchConfig` (pointing to the S3 bucket for oversized payloads and binary images), and that the IAM Role ARN passed in `cloudWatchConfig.roleArn` has `logs:CreateLogStream` and `logs:PutLogEvents` permissions on `/aws/bedrock/modelinvocations*` with a trust policy allowing `bedrock.amazonaws.com`.
- **D.** Ensure the KMS Customer Managed Key policy explicitly grants `kms:Encrypt`, `kms:Decrypt`, `kms:GenerateDataKey*`, and `kms:DescribeKey` to **both** `bedrock.amazonaws.com` (for S3 log encryption) and `logs.<region>.amazonaws.com` (for CloudWatch Log Group encryption, conditioned on `kms:EncryptionContext:aws:logs:arn`).
- **E.** Attach the `AmazonBedrockFullAccess` managed policy to every end user's browser session.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answers: C and D**

#### Explanation
- **C is correct:** CloudWatch Logs delivery in Bedrock Model Invocation Logging requires an IAM Role (`roleArn`) trusted by `bedrock.amazonaws.com` with `logs:CreateLogStream` and `logs:PutLogEvents` permissions, plus `largeDataDeliveryS3Config` to store multimodal images and payloads exceeding CloudWatch event size limits.
- **D is correct:** When the S3 bucket and CloudWatch Log Group are encrypted with a KMS CMK, the CMK key policy must grant encryption permissions to `bedrock.amazonaws.com` (for S3 object delivery) and `logs.<region>.amazonaws.com` (for CloudWatch Logs encryption).

#### Distractor Analysis
- **Why A is incorrect:** Any valid CloudWatch Log Group name (such as `/aws/bedrock/modelinvocations`) is supported.
- **Why B is incorrect:** Both S3 SSE-KMS and CloudWatch Logs KMS CMK encryption are fully supported with proper key policies.
- **Why E is incorrect:** End-user browser permissions have nothing to do with Bedrock's service-side log delivery role.

</details>
