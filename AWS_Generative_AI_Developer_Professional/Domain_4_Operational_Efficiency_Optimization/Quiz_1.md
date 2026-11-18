# Domain 4 — Quiz 1: Cost Optimization & Resource Efficiency Strategies (Task 4.1)

**Instructions:** This 15-question Staff/Principal-level quiz tests **Task 4.1 (Implement Cost Optimization & Resource Efficiency Strategies)** of the **AWS Certified Generative AI Developer - Professional (AIP-C01)** exam. Questions include both Single-Response (`A`–`D`) and Multiple-Response (`Choose TWO`, `A`–`E`) items.

---

### Question 1
A financial services company operates a multi-turn regulatory assistant on Amazon Bedrock using Anthropic Claude 3.5 Sonnet via the `Converse` API. Every request includes an 18,000-token system prompt containing static compliance rules, 4,500 tokens of JSON tool schemas, and the user's question. To reduce input token costs and Time to First Token (TTFT), an engineer adds `{"cachePoint": {"type": "default"}}` at the end of the `system` array and at the end of the `toolConfig.tools` array. However, after deploying to production with 80 requests per minute, Amazon CloudWatch metrics show `CacheWriteInputTokenCount` incrementing on 100% of invocations while `CacheReadInputTokenCount` remains at zero.

Upon inspecting the application code, the architect discovers the `system` prompt starts with:
`"System Context [RequestUUID: " + str(uuid.uuid4()) + " | Timestamp: " + datetime.utcnow().isoformat() + "]: You are a regulatory compliance auditor..."`

What is the root cause of the 0% cache hit rate, and how should the architect fix it?

- **A.** Move the dynamic `RequestUUID` and `Timestamp` metadata out of the `system` prompt and append them to the final `user` message block after all `cachePoint` markers, ensuring the token prefix preceding the `cachePoint` checkpoints remains 100% byte-for-byte identical across requests.
- **B.** Increase the size of the `system` prompt from 18,000 tokens to at least 32,768 tokens, because Anthropic Claude 3.5 Sonnet requires a minimum prefix threshold of 32,768 tokens before activating regional KV cache reads.
- **C.** Replace `"cachePoint": {"type": "default"}` with `"cachePoint": {"type": "semantic", "similarityThreshold": 0.95}` so Amazon Bedrock ignores minor UUID and timestamp token variations at the beginning of the prompt.
- **D.** Provision an Amazon ElastiCache for Valkey cluster and attach its VPC Endpoint ARN to the `Converse` API `cacheConfig` parameter, because Amazon Bedrock does not persist prefix KV caches across separate stateless HTTP invocations by default.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
Amazon Bedrock Prompt Caching (`cachePoint`) works via **exact cryptographic prefix matching** of the tokenized sequence from the very first token of the request up to the `cachePoint` boundary. Injecting a unique `uuid4()` and `Timestamp` at the beginning of the `system` block alters the very first tokens of the prefix on every single API call, causing every request to compute a brand-new prefix hash (`CacheWriteInputTokenCount`) and pay the 25% cache write premium without ever achieving a cache read hit. Moving all dynamic per-request variables (timestamps, request IDs, user profile attributes) to the end of the `messages` array *after* the final `cachePoint` guarantees an identical static prefix across all calls within the 5-minute TTL window.

#### Distractor Analysis
- **Why B is incorrect:** The minimum token threshold for Prompt Caching on Anthropic Claude 3.5 Sonnet is **1,024 tokens** (and 2,048 tokens for Claude 3.5 Haiku), not 32,768 tokens. An 18,000-token system prompt already far exceeds the minimum threshold.
- **Why C is incorrect:** Amazon Bedrock `cachePoint` only supports `"type": "default"` for deterministic token-prefix KV caching; there is no `"type": "semantic"` parameter native to the `Converse` API `cachePoint` specification.
- **Why D is incorrect:** Amazon Bedrock Prompt Caching is a fully managed internal KV-cache within the Bedrock service infrastructure; it does not accept or require external ElastiCache cluster ARNs in the API payload.
</details>

---

### Question 2
A B2B SaaS analytics platform serves 400 enterprise customers using a multi-tenant RAG architecture on Amazon Bedrock. Each tenant has strict data isolation requirements and custom pricing catalogs stored in tenant-scoped knowledge bases. Telemetry reveals that 55% of the 1.2 million daily queries are semantically identical questions asked repeatedly by different users within the same tenant (for example, *"What is our standard SLA credit for a P1 outage?"* vs. *"How much SLA credit do we give for Severity-1 downtime?"*).

The VP of Engineering wants to cut Bedrock token expenditure by at least 45% and achieve sub-20ms response times for these repetitive queries without risking cross-tenant data leakage. Which architecture meets these requirements?

- **A.** Enable Amazon Bedrock Prompt Caching (`cachePoint`) on the user message block so that semantically similar questions automatically reuse the KV cache of previous users across the regional Bedrock cluster.
- **B.** Deploy a semantic cache on Amazon MemoryDB using Vector Search and Amazon Titan Text Embeddings V2, storing cached responses in Redis hashes indexed with a mandatory `tenant_id` `TAG` field, and executing a pre-filtered $k$-NN query (`@tenant_id:{<caller_tenant_id>}`) with a cosine similarity threshold of $\ge 0.95$ before invoking Bedrock.
- **C.** Configure Amazon CloudFront in front of Amazon API Gateway with `Cache-Control: max-age=3600` keyed on the HTTP `POST` request body SHA-256 hash and the `X-Tenant-Id` header.
- **D.** Deploy an Amazon OpenSearch Serverless vector index that stores all query embeddings globally without metadata attributes, and apply an Amazon Bedrock Guardrail PII redaction filter on the cached output before returning it to any tenant.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
To bypass Foundation Model invocation altogether for differently worded but semantically equivalent queries, you need a **Semantic Cache**. Embedding the incoming query with **Amazon Titan Text Embeddings V2** (normalized 512- or 1,024-dimension vectors) and querying an in-memory **Amazon MemoryDB** HNSW index delivers sub-15ms latency. Crucially, in a multi-tenant environment, pre-filtering the vector search by a mandatory `@tenant_id:{...}` tag guarantees that a semantic hit only matches cached responses belonging to the exact same tenant, eliminating cross-tenant data leakage.

#### Distractor Analysis
- **Why A is incorrect:** Bedrock Prompt Caching (`cachePoint`) only matches **exact byte-for-byte token prefixes** and still invokes the FM to decode output tokens; it cannot match differently phrased questions or return a cached completion in $<20\text{ ms}$.
- **Why C is incorrect:** CloudFront request-body SHA-256 hashing only matches **100% character-identical** queries; phrasing variations produce completely different SHA-256 hashes and miss the cache.
- **Why D is incorrect:** An unfiltered global semantic cache across tenants allows Tenant B to receive Tenant A's proprietary business numbers (such as custom SLA credits or revenue figures), which PII redaction filters do not block because corporate SLA percentages are not personal identifiable information (PII).
</details>

---

### Question 3
An e-commerce marketplace ingests 4 million new seller product listings every night between 10:00 PM and 11:30 PM. A processing pipeline must analyze each listing's title and raw description using Anthropic Claude 3.5 Haiku on Amazon Bedrock to generate standardized taxonomy tags and compliance flags before the catalog search index refreshes at 6:00 AM the next morning. Currently, an AWS Step Functions Distributed Map invokes `InvokeModel` synchronously using On-Demand throughput, which causes severe `ThrottlingException` (HTTP 429) errors that spill over and impact the company's live customer-facing shopping assistant, while costing \$180,000 per month.

What is the MOST cost-effective architectural change that eliminates throttling contention with the live shopping assistant while meeting the 6:00 AM deadline?

- **A.** Purchase 15 Model Units (MUs) of Provisioned Throughput with a 6-month commitment for Claude 3.5 Haiku and dedicate the Provisioned Model ARN to the nightly Step Functions Distributed Map.
- **B.** Switch the nightly Step Functions Distributed Map from the regional Claude 3.5 Haiku model ID to the `us.anthropic.claude-3-5-haiku-20241022-v1:0` Cross-Region Inference (CRIS) profile.
- **C.** Aggregate the nightly product listings into JSONL files in Amazon S3 and submit an Amazon Bedrock Batch Inference job (`CreateModelInvocationJob`), triggering downstream catalog indexing via an Amazon EventBridge rule upon job completion.
- **D.** Deploy an Amazon SQS FIFO queue with a concurrency limit of 5 AWS Lambda workers to trickle requests to the On-Demand `InvokeModel` endpoint over 8 hours.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon Bedrock Batch Inference** is purpose-built for asynchronous, high-volume workloads that can tolerate multi-hour completion windows (up to 24 hours). Batch Inference provides two decisive advantages here:
1. **50% Flat Cost Discount**: It cuts the \$180,000/month token bill in half to \$90,000/month immediately.
2. **Quota Isolation**: Batch Inference jobs execute against dedicated asynchronous batch capacity and **do not consume regional On-Demand RPM/TPM quotas**, completely eliminating any `ThrottlingException` blast radius on the live customer-facing shopping assistant.

#### Distractor Analysis
- **Why A is incorrect:** Purchasing 24x7 Provisioned Throughput Model Units for a workload that only runs for a few hours overnight results in $>75\%$ idle capacity waste, costing far more than Batch Inference.
- **Why B is incorrect:** While Cross-Region Inference (CRIS) increases effective On-Demand quotas, it still bills at 100% On-Demand token prices (missing the 50% Batch discount) and still shares the account's CRIS On-Demand quota pool with live interactive traffic.
- **Why D is incorrect:** Trickling 4 million requests through 5 synchronous Lambda workers over 8 hours (28,800 seconds) requires $\approx 139\text{ requests/sec}$ ($8,333\text{ RPM}$), which will still contend with regional On-Demand quotas and pay 100% On-Demand pricing instead of the 50% Batch Inference discount.
</details>

---

### Question 4
A healthcare technology company runs an internal clinical coding assistant on Amazon Bedrock using Anthropic Claude 3.5 Sonnet (`$3.00` per 1M input tokens, `$15.00` per 1M output tokens On-Demand). An architecture review of 30 days of Bedrock Model Invocation Logs shows that 70% of incoming prompts are straightforward ICD-10 code lookups or simple formatting requests that Anthropic Claude 3.5 Haiku (`$0.80` per 1M input tokens, `$4.00` per 1M output tokens) answers with identical clinical accuracy, while only 30% of prompts involve complex multi-pathology differential coding requiring Claude 3.5 Sonnet.

The engineering team has only two developers and wants to reduce inference spend by at least 35% and improve average response latency **without** building, training, or maintaining a custom classification model or client-side routing microservice. Which solution should they implement?

- **A.** Configure an Amazon Bedrock Guardrail topic filter that blocks simple ICD-10 queries from reaching Claude 3.5 Sonnet and redirects the client application via an HTTP 302 status code to Claude 3.5 Haiku.
- **B.** Enable `performanceConfig={"latency": "optimized"}` on the existing Claude 3.5 Sonnet invocations so Bedrock dynamically prunes 70% of the model parameters for simple prompts.
- **C.** Switch the `modelId` to the System-Defined Cross-Region Inference profile `us.anthropic.claude-3-5-sonnet-20241022-v2:0` so Bedrock automatically routes simple prompts to regions with cheaper spot token pricing.
- **D.** Replace the Claude 3.5 Sonnet `modelId` in the `Converse` API call with the Amazon Bedrock Intelligent Prompt Router ARN for the Anthropic Claude family (`default-prompt-router/anthropic.claude:1`).

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon Bedrock Intelligent Prompt Routing** evaluates the complexity of each incoming prompt in real time and automatically routes simple requests to the smaller, faster, cheaper model (**Claude 3.5 Haiku**) and complex requests to the frontier model (**Claude 3.5 Sonnet**) within the same model family—all behind a single unified Prompt Router ARN. It requires zero custom classifier training or application routing logic and cuts inference costs by 30%–50%+ while maintaining response quality.

#### Distractor Analysis
- **Why A is incorrect:** Amazon Bedrock Guardrails evaluate safety, PII, and denied topics to block or mask content; they cannot return HTTP 302 redirects or act as a dynamic cost-routing proxy between models.
- **Why B is incorrect:** `performanceConfig={"latency": "optimized"}` optimizes hardware execution and Inter-Token Latency (ITL) for the specified model, but it still invokes the exact same model at standard (or latency-optimized) token rates and does not route to a cheaper model.
- **Why C is incorrect:** Cross-Region Inference (CRIS) routes requests across regions for capacity and quota elasticity using the **same model** (`Claude 3.5 Sonnet`), billed at the calling region's standard On-Demand rate. There is no "spot token pricing" in Bedrock CRIS.
</details>

---

### Question 5
A FinOps architect is evaluating whether to purchase **Amazon Bedrock Provisioned Throughput** or remain on **On-Demand (via Cross-Region Inference)** for a 24x7 automated fraud-narrative generator.
- **Traffic Profile**: Steady 24x7x30 workload (720 hours/month) generating a constant **150,000 Input Tokens Per Minute (TPM)** and **30,000 Output Tokens Per Minute (TPM)** with virtually zero hourly variance.
- **On-Demand Pricing**: `$3.00` per 1M input tokens and `$15.00` per 1M output tokens.
- **Provisioned Throughput Specification**: 1 Model Unit (MU) provides a maximum capacity of **100,000 Input TPM** and **20,000 Output TPM** at a 1-Month Commitment price of **`$32.00` per MU per hour**.

Based on exact FinOps token math for a 720-hour month, what is the monthly cost of each option, and which procurement strategy is optimal?

- **A.** On-Demand costs `$38,880/month`; purchasing 2 MUs of Provisioned Throughput costs `$46,080/month`. Remain 100% on On-Demand.
- **B.** On-Demand costs `$38,880/month`; purchasing 1 MU of Provisioned Throughput costs `$23,040/month`. Purchase 1 MU of Provisioned Throughput to handle 100% of the workload and save `$15,840/month`.
- **C.** On-Demand costs `$77,760/month`; purchasing 2 MUs of Provisioned Throughput costs `$46,080/month`. Purchase 2 MUs of Provisioned Throughput to cover the full workload.
- **D.** On-Demand costs `$38,880/month`; purchasing 1 MU of Provisioned Throughput (`$23,040/month`) handles two-thirds of the steady traffic (100,000 Input TPM / 20,000 Output TPM), while routing the remaining 50,000 Input TPM / 10,000 Output TPM spillover to On-Demand (`$12,960/month`) yields a combined hybrid cost of **`$36,000/month`**, saving `$2,880/month`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
Let's perform the exact FinOps calculation over $720\text{ hours} = 43,200\text{ minutes}$ per month:
1. **100% On-Demand Monthly Cost**:
   - Monthly Input Tokens: $150,000 \times 43,200 = 6,480,000,000 = 6,480\text{M input tokens}$.
   - Input Token Cost: $6,480 \times \$3.00 = \$19,440$.
   - Monthly Output Tokens: $30,000 \times 43,200 = 1,296,000,000 = 1,296\text{M output tokens}$.
   - Output Token Cost: $1,296 \times \$15.00 = \$19,440$.
   - **Total 100% On-Demand Cost** = $\$19,440 + \$19,440 = \mathbf{\$38,880/\text{month}}$.
2. **100% Provisioned Throughput Monthly Cost (2 MUs)**:
   - Because 1 MU only supports $100,000\text{ Input TPM}$ and $20,000\text{ Output TPM}$, covering 100% of the $150\text{K}/30\text{K}$ workload solely with Provisioned Throughput requires $\lceil 1.5 \rceil = 2\text{ MUs}$ (leaving the second MU at only 50% utilization).
   - Cost of 2 MUs = $2 \times \$32.00/\text{hr} \times 720\text{ hrs} = \mathbf{\$46,080/\text{month}}$ (more expensive than On-Demand!).
3. **Hybrid Baseline + Spillover Cost (1 MU at 100% Utilization + 0.5 MU equivalent on On-Demand)**:
   - 1 MU at 100% utilization ($100\text{K Input TPM}, 20\text{K Output TPM}$) costs $1 \times \$32.00 \times 720 = \mathbf{\$23,040/\text{month}}$.
   - Remaining spillover ($50\text{K Input TPM}, 10\text{K Output TPM}$, which is exactly $1/3$ of total traffic) on On-Demand costs $\$38,880 \times \frac{1}{3} = \mathbf{\$12,960/\text{month}}$.
   - **Total Hybrid Cost** = $\$23,040 + \$12,960 = \mathbf{\$36,000/\text{month}}$, achieving the lowest total cost (`$2,880/month` cheaper than pure On-Demand and `$10,080/month` cheaper than over-provisioning 2 MUs).

#### Distractor Analysis
- **Why A is incorrect:** While A calculates the pure On-Demand (`$38,880`) and 2-MU (`$46,080`) numbers accurately, it misses the hybrid baseline + On-Demand spillover pattern (`$36,000/month`) recommended by the AWS Well-Architected Generative AI Lens whenever fractional MU utilization (`0.5 MU` on the second unit) is below the ~60% break-even threshold.
- **Why B is incorrect:** 1 MU cannot handle 150,000 Input TPM and 30,000 Output TPM on its own; requests exceeding the 100K/20K ceiling will fail with `ThrottlingException`.
- **Why C is incorrect:** Miscomputes the On-Demand monthly cost by a factor of 2 (`$77,760` instead of `$38,880`).
</details>

---

### Question 6
A legal-tech startup built a contract review agent using Meta Llama 3.1 405B Instruct on Amazon Bedrock to extract 14 specific indemnification risk clauses as a structured JSON object. While Llama 3.1 405B achieves 97.5% F1 accuracy on this narrow extraction task, its per-token cost and 4.2-second latency violate the startup's unit economics. Zero-shot testing of Meta Llama 3.1 8B Instruct yields acceptable latency (0.6 seconds) and 80% lower cost, but its out-of-the-box F1 accuracy on complex legalese drops to 78%.

The team has 15,000 historical contract snippets stored in Amazon S3, complete with verified invocation logs, and wants the 8B model to match the 405B model's extraction accuracy without manually labeling thousands of new training records by hand. What is the MOST operationally efficient AWS solution?

- **A.** Configure an Amazon Bedrock Knowledge Base over the 15,000 historical contracts and attach it to the un-tuned Meta Llama 3.1 8B Instruct model with `top_k=50` chunk retrieval on every request.
- **B.** Enable Amazon Bedrock Prompt Caching (`cachePoint`) on Meta Llama 3.1 405B Instruct to reduce its output decoding latency from 4.2 seconds to 0.6 seconds.
- **C.** Create an Amazon Bedrock Model Distillation job specifying Meta Llama 3.1 405B Instruct as the Teacher model and Meta Llama 3.1 8B Instruct as the Student model using the S3 contract prompts dataset, and deploy the distilled custom student model.
- **D.** Run Continued Pre-Training (CPT) on Meta Llama 3.1 405B Instruct using raw unlabeled SEC filings so the 405B model requires fewer output tokens.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon Bedrock Model Distillation** automates transferring domain-specific reasoning and structured extraction capability from a large Teacher model (**Llama 3.1 405B**) into a smaller, faster Student model (**Llama 3.1 8B**). Bedrock invokes the Teacher model on the S3 prompt dataset (and optionally synthesizes augmented training examples) to generate high-accuracy golden labels, and then fine-tunes the Student model. This enables the 8B Student model to reach near-parity accuracy with the 405B Teacher at up to 75%–80% lower inference cost and 3x–5x faster latency.

#### Distractor Analysis
- **Why A is incorrect:** Stuffing `top_k=50` historical contract chunks into every prompt balloons input token consumption by tens of thousands of tokens per call, drastically increasing TTFT and cost without teaching the 8B model weight-level legal reasoning.
- **Why B is incorrect:** Prompt Caching (`cachePoint`) only speeds up **input prefill (TTFT)** for repeated static prefixes; it does not speed up autoregressive output decoding (ITL) or reduce the per-token cost of unique contract snippets.
- **Why D is incorrect:** Continued Pre-Training on the 405B model makes an already expensive frontier model even more costly to host on Provisioned Throughput without reducing its parameter size or latency.
</details>

---

### Question 7
An enterprise RAG pipeline queries an Amazon OpenSearch Serverless vector index with `top_k=30`, returning thirty 600-token document chunks (18,000 tokens of context) on every user query, and passes all 30 chunks to Anthropic Claude 3.5 Sonnet. An audit shows that on 92% of queries, the entire answer is contained within just 3 of those 30 chunks, while the remaining 27 chunks add irrelevant noise ("lost in the middle" phenomenon) and account for `$42,000/month` in unnecessary Bedrock input token charges. However, simply lowering OpenSearch vector search from `top_k=30` to `top_k=3` causes retrieval recall to drop by 22% because bi-encoder vector embeddings occasionally rank the best chunk at position 8 or 12.

Which architectural change preserves the high retrieval recall of `top_k=30` while cutting Claude 3.5 Sonnet context token costs by ~85%?

- **A.** Insert a cross-encoder reranking stage using the **Amazon Bedrock Rerank API** (`Cohere Rerank 3.5` or `Amazon Rerank 1.0`) to score the 30 retrieved OpenSearch chunks against the user query and pass only the top 4 highest-scoring chunks (2,400 tokens) to Claude 3.5 Sonnet.
- **B.** Change the OpenSearch Serverless vector index distance metric from Cosine Similarity to Euclidean ($L_2$) distance and reduce `top_k` to 3.
- **C.** Pass all 30 retrieved chunks to Claude 3.5 Sonnet, but place a `{"cachePoint": {"type": "default"}}` marker at the end of the 30 dynamically retrieved chunks so every query gets a 90% cache read discount.
- **D.** Increase the chunk size during document ingestion from 600 tokens to 6,000 tokens and set `top_k=3`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
Two-stage retrieval (**Bi-Encoder Vector Search + Cross-Encoder Reranking**) is the industry standard for optimizing both RAG recall and LLM token efficiency. Bi-encoder embeddings compress query and document independently into single vectors, making `top_k=30` retrieval fast ($<20\text{ ms}$) for high recall. Passing those 30 candidate chunks to the **Amazon Bedrock Rerank API** uses a specialized cross-encoder model to compute deep token-level relevance scores between the query and each candidate chunk at a fraction of LLM cost, filtering the context down to the top 3–4 chunks (reducing LLM input context from 18,000 tokens to 2,400 tokens—an 86.7% reduction) while improving answer precision.

#### Distractor Analysis
- **Why B is incorrect:** For normalized embeddings (like Titan Text Embeddings V2), Cosine Similarity and $L_2$ distance produce the exact same ranking order; lowering `top_k` to 3 without a reranker still suffers the 22% recall drop.
- **Why C is incorrect:** Because every user query retrieves a *different* dynamic combination and ordering of 30 chunks from the vector store, the token sequence of those 30 chunks changes on every request, resulting in a 0% Prompt Cache hit rate and a 25% cache write penalty!
- **Why D is incorrect:** 3 chunks $\times$ 6,000 tokens still equals 18,000 input tokens (zero token savings) and degrades retrieval granularity.
</details>

---

### Question 8
A customer support chatbot allows users to engage in long troubleshooting sessions that frequently reach 40 to 60 back-and-forth turns. The orchestration Lambda function retrieves the full conversation array from Amazon DynamoDB and passes every historical `user` and `assistant` message to Amazon Bedrock `Converse` on each turn. FinOps telemetry reveals that while Turn 1 consumes 800 input tokens, Turn 50 consumes 48,000 input tokens, causing cumulative token consumption to grow quadratically ($O(N^2)$) across the session length and driving up per-session cost by 900%.

Which context management pattern maintains both recent conversational precision and long-range session memory while bounding input tokens to a predictable constant ceiling ($O(1)$ per turn)?

- **A.** Configure DynamoDB Time-To-Live (TTL) to delete all conversation items older than 60 seconds so the history never exceeds 2 turns.
- **B.** Implement a sliding window buffer that retains the most recent $K=6$ turns verbatim while asynchronously invoking a lightweight model (such as Amazon Nova Micro or Claude 3.5 Haiku) every 6 turns to fold older evicted turns into a rolling 300-token `<session_summary>` block injected into the prompt.
- **C.** Encode the entire 50-turn JSON conversation history using `gzip` and Base64 before passing the compressed string inside the `user` message block to Claude 3.5 Sonnet.
- **D.** Set `inferenceConfig={"maxTokens": 100}` on the `Converse` API call so Bedrock automatically truncates the oldest input messages from the `messages` array when billing input tokens.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
A **Hierarchical Rolling Summary + Sliding Window** memory architecture bounds input token consumption per turn to a fixed constant ($K$ verbatim recent turns + a compact 300-token rolling summary of turns $1 \dots T-K$). By using a low-cost model (Amazon Nova Micro or Claude 3.5 Haiku) asynchronously to update the summary as turns roll off the window, the assistant preserves critical entities mentioned in Turn 2 (such as account IDs, device serial numbers, or prior troubleshooting steps tried) while keeping verbatim fidelity for immediate follow-up questions in the last 6 turns.

#### Distractor Analysis
- **Why A is incorrect:** Deleting history after 60 seconds causes the chatbot to suffer total amnesia mid-conversation if the user pauses for a minute to check their router or screen.
- **Why C is incorrect:** Base64-encoded `gzip` binary strings cannot be natively reasoned over by an LLM tokenizer and actually tokenize into *more* BPE subword tokens of random alphanumeric characters while destroying semantic comprehension.
- **Why D is incorrect:** `maxTokens` in `inferenceConfig` only limits the number of **generated output tokens**; it has no effect on truncating the input `messages` array.
</details>

---

### Question 9
An enterprise insurance platform uses Anthropic Claude 3.5 Sonnet to extract structured claim attributes from scanned repair estimates and route them into a downstream ERP database. The downstream database only consumes a 12-field JSON object (~120 tokens). However, invocation logs show that Claude 3.5 Sonnet averages **1,150 output tokens per request** because it prefaces every response with a polite paragraph, outputs the JSON inside markdown blocks, and follows up with a 900-token bulleted explanation of how it interpreted each line item. Because output tokens cost `$15.00 / 1M` (5x the input token price), these superfluous 1,030 output tokens per call waste `$28,000/month` and add 8 seconds of latency.

Which combination of techniques in the `Converse` API will deterministically restrict generation to the required 120-token JSON payload?

- **A.** Set `inferenceConfig={"temperature": 1.0, "topP": 0.99}` so the model skips deterministic chain-of-thought explanations.
- **B.** Attach an Amazon Bedrock Guardrail with a Word Policy that blocks the words `"Here"`, `"Explanation"`, and `"Summary"`, which forces the model to regenerate only the JSON block at zero token charge.
- **C.** Define the 12-field schema as a tool in `toolConfig.tools`, set `toolChoice={"tool": {"name": "submit_claim_schema"}}` to force structured tool-use output, include a system instruction prohibiting conversational prose, and configure a tight `maxTokens` ceiling (e.g., `250`).
- **D.** Switch the API call from `Converse` to `ConverseStream` and close the client TCP socket after receiving 120 tokens so Bedrock only bills for the tokens received before the socket closed.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
Forcing structured output via `toolConfig` with `toolChoice={"tool": {"name": "submit_claim_schema"}}` instructs Anthropic Claude (and Amazon Nova) models to immediately emit a structured `toolUse` JSON block matching your exact JSON Schema without generating conversational preamble or postamble explanations. Pairing this with a concise system instruction and a tight `maxTokens` cap reduces output tokens from 1,150 to ~120 tokens, cutting output token cost and decoding latency by nearly 90%.

#### Distractor Analysis
- **Why A is incorrect:** Raising `temperature` to `1.0` increases randomness and hallucination risk in data extraction and does nothing to shorten output verbosity.
- **Why B is incorrect:** When a Bedrock Guardrail intervenes on output tokens, **you are still billed for all output tokens generated by the FM** before the guardrail blocked the response, AND the user receives the blocked intervention message rather than the extracted JSON!
- **Why D is incorrect:** Abruptly closing a client TCP socket mid-stream yields truncated, unparseable JSON and still bills for all tokens generated on the Bedrock server side before stream termination was processed.
</details>

---

### Question 10
A global retail company uses Anthropic Claude 3.5 Haiku on Amazon Bedrock for an interactive order-status chatbot. Each request has a 1,400-token static system prompt followed by `{"cachePoint": {"type": "default"}}` and a 60-token user query. Despite sending 200 requests per minute with an identical 1,400-token prefix, CloudWatch shows `CacheWriteInputTokenCount = 0` and `CacheReadInputTokenCount = 0` on every request, and all 1,460 input tokens are billed at the standard uncached input rate.

Why is Amazon Bedrock Prompt Caching not activating for these requests?

- **A.** Anthropic Claude 3.5 Haiku does not support Prompt Caching on Amazon Bedrock under any circumstances; only Claude 3 Opus supports `cachePoint`.
- **B.** The static prefix before the `cachePoint` is only 1,400 tokens, which is below the **2,048-token minimum cacheable prefix requirement** for Anthropic Claude 3.5 Haiku.
- **C.** Prompt Caching is only supported when calling the `InvokeModel` API with raw model-specific JSON bodies and is ignored by the unified `Converse` API.
- **D.** The request rate of 200 requests per minute exceeds the 5-minute TTL window, causing the cache to evict before any read hit can occur.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
Every model supporting Amazon Bedrock Prompt Caching enforces a **minimum token threshold** per checkpoint before it will write or read from the prefix cache. While **Anthropic Claude 3.5 Sonnet, Claude 3 Opus, and Amazon Nova models require at least 1,024 tokens**, **Anthropic Claude 3.5 Haiku (and Claude 3 Haiku) requires a minimum of 2,048 tokens** preceding the `cachePoint`. When a prefix is shorter than the model's minimum threshold (1,400 tokens < 2,048 tokens), Bedrock silently ignores the `cachePoint` and processes the prompt as a standard uncached invocation (`CacheWriteInputTokenCount = 0`, `CacheReadInputTokenCount = 0`).

#### Distractor Analysis
- **Why A is incorrect:** Claude 3.5 Haiku fully supports Prompt Caching on Amazon Bedrock once the 2,048-token minimum prefix threshold is met.
- **Why C is incorrect:** The `Converse` and `ConverseStream` APIs natively support `cachePoint` blocks in `system`, `tools`, and `messages`.
- **Why D is incorrect:** High request frequency (200 RPM) *improves* cache hit rates and continuously refreshes the 5-minute TTL on every hit.
</details>

---

### Question 11
A media analytics company processes 10,000 podcast episodes per day. For each episode, an orchestrator makes 12 sequential questions against the same 35,000-token transcript using Amazon Nova Pro on Amazon Bedrock. Currently, each of the 12 questions is sent as an independent `Converse` call with `[System Prompt] + [User Question i] + [35,000-Token Transcript]`. Even after adding a `cachePoint` at the very end of the `user` message block, none of the subsequent 11 questions for the same podcast episode achieve a cache hit.

How should the engineer restructure the prompt and `cachePoint` placement so that Questions 2 through 12 for the same podcast episode achieve a 100% prefix cache hit on the 35,000-token transcript?

- **A.** Split the 35,000-token transcript into 35 separate content blocks of 1,000 tokens each and place 35 `cachePoint` markers between them.
- **B.** Keep `[User Question i]` before the `[35,000-Token Transcript]`, but set `inferenceConfig={"temperature": 0.0}` so the cache matcher ignores the question tokens at the start of the message.
- **C.** Add a `sleep(360)` delay between Question 1 and Question 2 so that Bedrock has time to replicate the 35,000-token KV cache across all Availability Zones before Question 2 is sent.
- **D.** Place `[System Prompt]`, followed by the `[35,000-Token Transcript]` content block, followed immediately by `{"cachePoint": {"type": "default"}}`, and place the dynamic `[User Question i]` content block *after* the `cachePoint`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
Because Prompt Caching hashes the **exact contiguous token prefix from token 0 up to the `cachePoint`**, placing `[User Question i]` *before* the `[35,000-Token Transcript]` changes the early prefix tokens on every single question (`Question 1` vs `Question 2`), invalidating the entire downstream 35,000-token transcript! By ordering the prompt as `[Static System Prompt] -> [Static 35,000-Token Transcript] -> [cachePoint] -> [Dynamic Question i]`, Question 1 writes the 35,000-token transcript KV cache once, and Questions 2 through 12 hit the exact same prefix hash, saving up to 90% on input tokens and up to 85% on TTFT.

#### Distractor Analysis
- **Why A is incorrect:** Bedrock allows a **maximum of 4 `cachePoint` checkpoints per request**; passing 35 `cachePoint` blocks raises a `ValidationException`.
- **Why B is incorrect:** `temperature` only controls output token sampling probabilities; it has zero effect on prefix token hashing.
- **Why C is incorrect:** The Prompt Cache TTL is **5 minutes (300 seconds)**! Sleeping for 360 seconds guarantees that the cache entry expires and is evicted before Question 2 is even sent.
</details>

---

### Question 12
An enterprise software company is designing a dynamic code-review assistant in Python that passes 18 tool definitions in `toolConfig.tools` (totalling 3,800 tokens) followed by `{"cachePoint": {"type": "default"}}` to Anthropic Claude 3.5 Sonnet. The 18 tool specifications are loaded at runtime by iterating over a Python `set` of plugin classes across a fleet of 50 AWS Lambda execution environments. The team notices that while warm invocations within the *same* Lambda container achieve high `CacheReadInputTokenCount`, invocations routed to *newly initialized* Lambda containers frequently trigger a fresh `CacheWriteInputTokenCount` for the tool definitions even within the 5-minute TTL window.

What is the root cause of these unexpected cache writes across different Lambda containers, and how should it be resolved?

- **A.** Python randomizes hash seeds (`PYTHONHASHSEED`) across processes, causing iteration over a `set` to serialize the 18 tools in a different order in each Lambda container, which changes the token prefix before the `cachePoint`. Sort the `tools` list deterministically by `toolSpec.name` before appending the `cachePoint`.
- **B.** Amazon Bedrock Prompt Caching is pinned to the source NAT Gateway IP address of the calling AWS Lambda VPC subnet. Route all 50 Lambda containers through a single static Elastic IP.
- **C.** Each AWS Lambda execution role session (`sts:AssumeRole`) generates a unique temporary `AccessKeyId`, and Bedrock salts the Prompt Cache hash with the caller's temporary `AccessKeyId`. Use long-lived IAM User access keys instead.
- **D.** The `toolConfig` field in `Converse` is only cached in local Lambda `/tmp` memory by `boto3`. Mount an Amazon EFS file system across all 50 Lambda functions to share the `boto3` cache directory.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
In Python 3.3+, hash randomization is enabled by default (`PYTHONHASHSEED=random`), meaning iterating over a `set` (or dynamically discovered unordered dictionary keys) produces a **different ordering in every new Python process**. If Container 1 serializes `[Tool_A, Tool_B, Tool_C]` and Container 2 serializes `[Tool_C, Tool_A, Tool_B]`, the tokenized sequence preceding the `cachePoint` is different, causing a cache miss and a paid cache write! Sorting the tool list deterministically (`tools.sort(key=lambda t: t["toolSpec"]["name"])`) and setting `json.dumps(..., sort_keys=True)` on schemas guarantees identical token serialization across all containers.

#### Distractor Analysis
- **Why B is incorrect:** Bedrock Prompt Caching is scoped at the AWS Account/Region level, not by client NAT IP address.
- **Why C is incorrect:** Bedrock does not salt prefix cache keys with temporary STS session credentials; all containers in the same AWS account share the regional prompt cache.
- **Why D is incorrect:** Prompt Caching happens on Amazon Bedrock's server-side accelerator cluster, not inside `boto3` or local Lambda `/tmp` storage.
</details>

---

### Question 13 (Multiple Response — Choose TWO)
A telecommunications provider is re-architecting its AI customer care platform on AWS to reduce its monthly Amazon Bedrock and vector database bill by 50% without degrading response accuracy. Currently:
- The platform embeds every user query using Amazon Titan Text Embeddings V2 (`dimensions=1024`, `float32`) and stores 80 million vectors in an Amazon OpenSearch Serverless Vector Search collection, consuming 32 Search OCUs continuously to hold the uncompressed HNSW graph in RAM.
- Every query invokes Anthropic Claude 3.5 Sonnet with a 6,000-token system prompt and 10 retrieved RAG chunks.

Which **TWO** optimizations will directly reduce both the OpenSearch Serverless OCU memory footprint and the Bedrock per-invocation token cost? *(Choose TWO.)*

- **A.** Configure Titan Text Embeddings V2 to generate normalized **512-dimensional** (or 256-dimensional) embeddings with OpenSearch **Scalar Quantization (`fp16` or `sq8`) / Binary Quantization**, cutting the HNSW vector index RAM footprint by 50%–87% and allowing OCU scale-down.
- **B.** Increase the OpenSearch HNSW `ef_construction` parameter from 128 to 1,024 and `m` from 16 to 64 to reduce index memory consumption.
- **C.** Restructure the `Converse` request to place a `{"cachePoint": {"type": "default"}}` immediately after the 6,000-token static system prompt, and integrate an **Amazon Bedrock Rerank** step to filter the 10 retrieved chunks down to the top 3 before invoking the LLM.
- **D.** Switch the OpenSearch Serverless collection type from `VECTORSEARCH` to `TIMESERIES` so vectors are automatically tiered to Amazon S3 Glacier Instant Retrieval after 24 hours.
- **E.** Enable `trace="ENABLED"` on all Amazon Bedrock Guardrails and Model Invocation Logging to automatically trigger a 30% observability discount on input tokens.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and C**

#### Explanation
- **Why A is correct:** **Amazon Titan Text Embeddings V2** uses Matryoshka Representation Learning (MRL), allowing you to request `dimensions=512` or `256` (retaining ~97%–99% of 1024-dim retrieval accuracy). Combining lower dimensionality with OpenSearch quantization (`fp16`/`sq8`/binary) shrinks HNSW memory requirements by 50%–87%, directly reducing the number of billed OpenSearch Compute Units (OCUs).
- **Why C is correct:** Caching the 6,000-token static system prompt via `cachePoint` cuts system prompt token read costs by 90%, while using **Bedrock Rerank** to prune 10 chunks down to the 3 most relevant chunks cuts dynamic RAG context tokens by 70%.

#### Distractor Analysis
- **Why B is incorrect:** Increasing `m` (links per node) from 16 to 64 *increases* HNSW graph memory consumption significantly rather than reducing it.
- **Why D is incorrect:** A `TIMESERIES` collection in OpenSearch Serverless does not support $k$-NN HNSW vector search, and S3 Glacier cannot serve real-time millisecond vector queries.
- **Why E is incorrect:** Enabling Guardrail traces and Model Invocation Logging incurs CloudWatch/S3 storage charges and never provides a discount on Bedrock token pricing.
</details>

---

### Question 14 (Multiple Response — Choose TWO)
A media company runs a multi-agent content localization workflow on Amazon Bedrock. The workflow consists of:
1. **Agent 1 (Language & Dialect Classifier)**: Inspects a 200-token article summary and outputs a 3-field JSON tag (`source_lang`, `target_dialect`, `formality_tier`).
2. **Agent 2 (Cultural Nuance & Idiom Translator)**: Translates a 4,000-token investigative journalism piece while preserving complex legal disclaimers and literary idioms.
3. **Nightly Archive Backfill**: Translates 500,000 legacy archive articles stored in Amazon S3 over a 2-week migration window with no real-time user waiting.

Currently, all three workloads invoke **Anthropic Claude 3.5 Sonnet On-Demand**. Which **TWO** architectural changes follow the AWS Well-Architected Generative AI Lens to minimize cost without sacrificing quality? *(Choose TWO.)*

- **A.** Migrate **Agent 2 (Cultural Nuance & Idiom Translator)** to **Amazon Titan Text Embeddings V2** to generate the translated localized articles at embedding token prices.
- **B.** Migrate **Agent 1 (Language & Dialect Classifier)** from Claude 3.5 Sonnet to a compact, low-cost model such as **Amazon Nova Micro** or **Anthropic Claude 3.5 Haiku** with structured tool/JSON output constraints.
- **C.** Purchase a 6-month commitment of 50 Model Units (MUs) of Provisioned Throughput on Anthropic Claude 3 Opus exclusively for the 2-week Nightly Archive Backfill.
- **D.** Execute the **Nightly Archive Backfill** of 500,000 legacy S3 articles using **Amazon Bedrock Batch Inference** jobs (`CreateModelInvocationJob`) to receive a 50% token cost discount and avoid exhausting real-time On-Demand quotas.
- **E.** Configure **Agent 1** to prepend 2,500 tokens of random padding characters to every 200-token summary so that it qualifies for the 1,024-token Prompt Caching minimum threshold.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: B and D**

#### Explanation
- **Why B is correct:** **Right-sizing models per task (`GENAI-COST-01`)** dictates using a fast, ultra-low-cost model (**Amazon Nova Micro** or **Claude 3.5 Haiku**) for deterministic classification and routing tasks like Agent 1, reducing that step's cost by 80%–95% and cutting latency by 3x.
- **Why D is correct:** The 500,000-article archive migration is an offline bulk workload in S3 with no interactive user waiting—the textbook use case for **Amazon Bedrock Batch Inference** (50% cost discount + isolated batch capacity).

#### Distractor Analysis
- **Why A is incorrect:** **Amazon Titan Text Embeddings V2** is an embedding model that outputs numerical floating-point vectors (`float32`), not natural language text translations.
- **Why C is incorrect:** Purchasing a **6-month commitment** of Provisioned Throughput on an ultra-expensive model (Claude 3 Opus) for a temporary **2-week** migration job leaves 5.5 months of stranded, unrecoverable commitment cost.
- **Why E is incorrect:** Prepending 2,500 tokens of useless padding to a dynamic 200-token input just to hit a cache threshold wastes tokens and lowers SNR, especially when the article summary itself is unique per article.
</details>

---

### Question 15 (Multiple Response — Choose TWO)
An enterprise platform runs a high-traffic semantic cache in **Amazon ElastiCache for Valkey** (with vector search enabled) in front of Amazon Bedrock. Over time, product prices and return policies in the underlying Amazon Aurora PostgreSQL database change several times a day. Customers have started complaining that when they ask the assistant about return windows or promotional pricing, the semantic cache occasionally returns outdated prices that were cached 3 days earlier, or returns the promotional price of *"Product X Pro"* when the user asked about *"Product X Lite"* because the two queries had a cosine similarity of `0.89`.

Which **TWO** engineering controls must be implemented to eliminate stale and false-positive semantic cache responses? *(Choose TWO.)*

- **A.** Lower the cosine similarity threshold from `0.88` to `0.70` so the vector search evaluates a broader neighborhood of cached answers.
- **B.** Disable normalization (`normalize=False`) in Amazon Titan Text Embeddings V2 so that older embeddings naturally decay in vector magnitude over time.
- **C.** Raise the semantic cache cosine similarity threshold from `0.88` to a strict threshold (such as **`0.96`**) and extract/pre-filter exact product SKU or entity tags alongside the vector $k$-NN lookup so queries about different product tiers never collide.
- **D.** Replace Amazon ElastiCache for Valkey with an Amazon S3 Glacier Deep Archive bucket to force automatic cache refreshes every 12 hours.
- **E.** Configure an **Amazon EventBridge Pipes** or **AWS Lambda** invalidation consumer triggered by Aurora database updates (via DynamoDB Streams / DMS / RDS Event Notifications) to delete or expire cached Redis vector keys tagged with the modified `product_sku` or `policy_id`, combined with a bounded key `EXPIRE` TTL.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: C and E**

#### Explanation
- **Why C is correct:** A cosine similarity of `0.89` is too loose for transactional/product queries where single-word modifiers (*"Pro"* vs *"Lite"*, *"upgrade"* vs *"cancel"*) flip the meaning. Tightening the cosine similarity threshold to **$\ge 0.95\text{–}0.97$** and enforcing exact entity/SKU metadata tag filtering (`@sku:{PROD_X_LITE}`) eliminates false-positive collisions.
- **Why E is correct:** Combining a bounded Redis `EXPIRE` TTL with **event-driven cache invalidation** (deleting keys matching `@product_sku:{...}` whenever the source-of-truth Aurora record changes) prevents stale pricing or policy responses.

#### Distractor Analysis
- **Why A is incorrect:** Lowering the similarity threshold to `0.70` drastically increases false-positive cache poisoning.
- **Why B is incorrect:** Vector embeddings are static numerical arrays stored in memory; setting `normalize=False` does not cause vectors to "decay over time" and breaks standard cosine similarity inner-product assumptions.
- **Why D is incorrect:** S3 Glacier Deep Archive has a retrieval time of **12 to 48 hours** and has no vector search capability whatsoever.
</details>
