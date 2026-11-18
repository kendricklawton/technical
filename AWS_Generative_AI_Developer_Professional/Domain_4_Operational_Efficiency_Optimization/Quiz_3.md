# Domain 4 — Quiz 3: Implement Monitoring Systems for GenAI Applications (Task 4.3)

**Instructions:** This 15-question Staff/Principal-level quiz tests **Task 4.3 (Implement Monitoring Systems for GenAI Applications)** of the **AWS Certified Generative AI Developer - Professional (AIP-C01)** exam. Questions include both Single-Response (`A`–`D`) and Multiple-Response (`Choose TWO`, `A`–`E`) items.

---

### Question 1
A large enterprise shares a single AWS production account across eight internal business units (Retail, Wealth Management, Insurance, Compliance, HR, Legal, Marketing, and IT Operations). All eight business units invoke Anthropic Claude 3.5 Sonnet on Amazon Bedrock (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`). The Chief Financial Officer requires an automated monthly chargeback report in **AWS Cost Explorer** and per-department monthly spend alerts in **AWS Budgets**, alongside isolated `InputTokenCount`, `OutputTokenCount`, and `InvocationLatency` dashboards per business unit in **Amazon CloudWatch**.

An engineer attempts to attach AWS Cost Allocation Tags (`Department=WealthManagement`, `Department=Retail`) directly to the base Foundation Model ARN and the System-Defined Cross-Region Inference Profile ARN, but the `TagResource` API returns an error because AWS-managed base model ARNs cannot be tagged per department. What is the native AWS solution?

- **A.** Create eight separate AWS accounts, purchase dedicated Provisioned Throughput Model Units (MUs) in each account, and manually export CSV files from each account.
- **B.** Create eight custom **Amazon Bedrock Application Inference Profiles** (`CreateInferenceProfile`), specifying `modelSource={'copyFrom': 'arn:aws:bedrock:us-east-1:<account-id>:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0'}` for each, attach the respective `Department` and `CostCenter` **AWS Cost Allocation Tags** to each Application Inference Profile, and scope each business unit's IAM role to invoke its assigned Application Inference Profile ARN.
- **C.** Enable AWS CloudTrail Data Events for `bedrock-runtime:InvokeModel` and configure AWS Cost Explorer to parse the raw prompt string inside CloudTrail logs to calculate token billing.
- **D.** Pass `requestMetadata={"aws:costAllocation:Department": "WealthManagement"}` in the `Converse` API call, which automatically populates AWS Cost Explorer and AWS Budgets without creating any resources.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
**Amazon Bedrock Application Inference Profiles** (`create_inference_profile`) are customer-owned regional resources that wrap either a single-region Foundation Model ARN or a System-Defined Cross-Region Inference Profile ARN. Unlike base Foundation Model ARNs, Application Inference Profiles support **custom AWS Cost Allocation Tags** (which natively integrate with **AWS Cost Explorer**, **AWS Budgets**, and **AWS Cost and Usage Reports [CUR]**) AND emit isolated metrics in the `AWS/Bedrock` CloudWatch namespace where the `ModelId` dimension is the Application Inference Profile ID.

#### Distractor Analysis
- **Why A is incorrect:** Purchasing 8 separate Provisioned Throughput commitments across 8 accounts is massively expensive and unnecessary when Application Inference Profiles natively isolate cost and telemetry within a single account.
- **Why C is incorrect:** CloudTrail Data Events do not record full prompt/completion token usage, and AWS Cost Explorer only groups by resource/billing tags, not CloudTrail event bodies.
- **Why D is incorrect:** `requestMetadata` is logged inside Bedrock Model Invocation Logs for debugging/filtering, but it is **not** ingested into AWS Billing, AWS Cost Explorer, or AWS Budgets.
</details>

---

### Question 2
A multi-tenant B2B SaaS platform serves **35,000 small-and-medium business (SMB) customer tenants** from a shared microservice tier invoking Amazon Bedrock. The SaaS product manager needs to:
1. Maintain real-time operational CloudWatch dashboards and alarms grouped by **Subscription Tier** (`Free`, `Growth`, `Enterprise`) and **Feature Name** (`EmailDraft`, `ContractSummary`, `ChatAssistant`).
2. Calculate exact monthly token usage (`inputTokens`, `outputTokens`, `cacheReadInputTokens`) and exact dollar cost for each of the **35,000 individual `tenant_id`s** to generate itemized monthly customer invoices.

Why should the architect **avoid** creating 35,000 Application Inference Profiles or publishing `tenant_id` as a standard CloudWatch Metric dimension, and what is the most cost-effective architecture?

- **A.** CloudWatch Metrics are free for unlimited dimensions, so the architect should publish 35,000 `tenant_id` dimensions $\times$ 10 metrics ($350,000$ custom metrics) directly via `cloudwatch.put_metric_data()` on every synchronous request.
- **B.** Bedrock automatically sends itemized per-tenant SaaS invoices to Stripe if `tenant_id` is included in the system prompt.
- **C.** Store token counts in local container memory and write them to an Amazon RDS MySQL instance once every 30 days.
- **D.** Publishing 35,000 high-cardinality `tenant_id` values as a CloudWatch Metric dimension creates hundreds of thousands of custom metric time series (incurring over `$50,000+/month` in custom CloudWatch metric charges), and Application Inference Profiles have account quotas well below 35,000. Instead, emit structured **CloudWatch Embedded Metric Format (EMF)** JSON logs where only low-cardinality attributes (`SubscriptionTier`, `FeatureName`, `ModelId`) are defined in the `Dimensions` array, while `tenant_id` and token/cost fields are preserved as high-cardinality JSON properties queried via **CloudWatch Logs Insights** or streamed via **Amazon Data Firehose to Amazon S3 (Parquet/Iceberg) + Amazon Athena**.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
In Amazon CloudWatch, every unique combination of dimension values creates a separate billed Custom Metric (`$0.30/metric/month` for the first 10,000 metrics). Putting a 35,000-cardinality `tenant_id` into CloudWatch Metric `Dimensions` across 3 features and 5 metric names generates $35,000 \times 3 \times 5 = 525,000$ custom metrics! By using **CloudWatch Embedded Metric Format (EMF)**:
- Only low-cardinality operational dimensions (`SubscriptionTier`, `FeatureName`, `ModelId`) are placed inside `_aws.CloudWatchMetrics.Dimensions` for real-time CloudWatch alarms.
- High-cardinality attributes (`tenant_id`, `user_id`, `request_id`, `calculated_cost_usd`) sit in the root JSON log object at zero custom metric charge and can be aggregated on-demand with **CloudWatch Logs Insights** or **Amazon S3 + Athena** for tenant billing.

#### Distractor Analysis
- **Why A is incorrect:** Calling synchronous `put_metric_data()` adds network latency to every request and creates a massive custom metric bill due to high dimension cardinality.
- **Why B is incorrect:** Bedrock has no native integration with third-party billing systems like Stripe and never parses billing metadata from system prompt text.
- **Why C is incorrect:** Storing financial billing counters in volatile container memory for 30 days results in catastrophic data loss whenever containers scale in, crash, or redeploy.
</details>

---

### Question 3
A Site Reliability Engineer (SRE) is investigating a sudden spike in customer error reports for a RAG application powered by Amazon Bedrock. Looking at the `AWS/Bedrock` namespace in Amazon CloudWatch over the past 30 minutes:
- `InvocationThrottles` (`Sum`) is **`0`**.
- `InvocationServerErrors` (`Sum`) is **`0`**.
- `InvocationClientErrors` (`Sum`) has spiked from **`0` to `450 per minute`**.

Which of the following issues would cause `InvocationClientErrors` to spike while `InvocationThrottles` and `InvocationServerErrors` remain at zero?

- **A.** A recent code deployment removed the context truncation logic in the RAG orchestrator, causing prompts with large retrieved documents to exceed the model's maximum context window (or pass a malformed JSON schema in `toolConfig`), resulting in HTTP 400 `ValidationException` responses.
- **B.** The application exceeded its regional Tokens Per Minute (TPM) and Requests Per Minute (RPM) On-Demand quotas, triggering HTTP 429 `ThrottlingException` errors.
- **C.** An internal hardware failure occurred inside the Amazon Bedrock accelerator cluster, returning HTTP 500 `InternalServerException` and HTTP 503 `ServiceUnavailableException` errors.
- **D.** The Prompt Cache (`cachePoint`) 5-minute TTL expired, causing a cache miss that increments `InvocationClientErrors` whenever `CacheWriteInputTokenCount > 0`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
In the `AWS/Bedrock` CloudWatch namespace:
- **`InvocationClientErrors`** tracks HTTP 4xx client errors (excluding HTTP 429 throttles), such as HTTP 400 `ValidationException` (exceeding max input tokens, passing more than 4 `cachePoint` blocks, invalid JSON schema in `toolConfig`, alternating `user`/`assistant` role violations) or HTTP 403 `AccessDeniedException`.
- **`InvocationThrottles`** specifically tracks HTTP 429 `ThrottlingException` responses when RPM/TPM quotas are exceeded.
- **`InvocationServerErrors`** tracks HTTP 5xx server-side faults (`InternalServerException`, `ServiceUnavailableException`).

#### Distractor Analysis
- **Why B is incorrect:** Quota exhaustion (HTTP 429) increments `InvocationThrottles`, not `InvocationClientErrors`.
- **Why C is incorrect:** Hardware/service faults (HTTP 500/503) increment `InvocationServerErrors`.
- **Why D is incorrect:** A Prompt Cache miss is a normal HTTP 200 invocation that increments `CacheWriteInputTokenCount` and `Invocations`, never an error metric.
</details>

---

### Question 4
An enterprise security and compliance team wants to audit the exact full-text input prompts, generated model completions, guardrail intervention trace details, and request metadata for every Amazon Bedrock API call made in their production account, and retain these audit logs in an encrypted Amazon S3 bucket (`SSE-KMS`) and an Amazon CloudWatch Logs log group. When the security engineer checks S3 and CloudWatch Logs in the production account, no Bedrock prompt/completion logs exist.

Why are no prompt and completion logs present, and how should the engineer enable them?

- **A.** Bedrock automatically writes all prompts and completions to AWS CloudTrail management events by default; the engineer simply needs to enable S3 server access logging on the CloudTrail bucket.
- **B.** Full prompt/completion logging is only supported when running self-hosted models on Amazon SageMaker EC2 instances and is unavailable in Amazon Bedrock.
- **C.** By default, Amazon Bedrock **does not log or store customer prompts and completions**. The engineer must explicitly enable **Amazon Bedrock Model Invocation Logging** (`PutModelInvocationLoggingConfiguration`) at the account/region level, configuring the destination S3 bucket (with a KMS key policy and bucket policy allowing the `bedrock.amazonaws.com` service principal) and/or CloudWatch Logs log group + IAM role.
- **D.** Set `inferenceConfig={"temperature": 0.0}` on every `Converse` call, which automatically creates an S3 bucket named `aws-bedrock-default-logs` in the account.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
For privacy and data security, **Amazon Bedrock Model Invocation Logging is disabled by default**—Bedrock never persists customer prompt payloads or generated completions unless the account administrator explicitly configures **`PutModelInvocationLoggingConfiguration`** (via the Bedrock console, CLI, or SDK). Once enabled, you can choose to log `textDataDeliveryEnabled`, `imageDataDeliveryEnabled`, `embeddingDataDeliveryEnabled`, and `videoDataDeliveryEnabled` to **Amazon S3**, **Amazon CloudWatch Logs**, or both.

#### Distractor Analysis
- **Why A is incorrect:** AWS CloudTrail records API control-plane and data-plane metadata (who called `Converse`, timestamp, source IP, model ARN), but **CloudTrail never logs the request body (prompt text) or response body (completion text)** of Bedrock runtime calls.
- **Why B is incorrect:** Bedrock natively supports full payload logging via Model Invocation Logging.
- **Why D is incorrect:** `temperature` is an inference sampling hyperparameter and has nothing to do with logging configurations.
</details>

---

### Question 5
A platform team deployed Amazon Bedrock Prompt Caching (`cachePoint`) across five microservices last week and wants to build a single CloudWatch Dashboard gauge widget using **CloudWatch Metric Math** (`AWS/Bedrock` namespace) that displays the real-time **Prompt Cache Token Hit Percentage (0% to 100%)** for each microservice's Application Inference Profile.

Given the following CloudWatch metric IDs configured on the widget for a specific `ModelId`:
- `m_uncached` = `SUM(InputTokenCount)`
- `m_cache_read` = `SUM(CacheReadInputTokenCount)`
- `m_cache_write` = `SUM(CacheWriteInputTokenCount)`
- `m_out` = `SUM(OutputTokenCount)`

Which CloudWatch Metric Math expression accurately computes the percentage of total input tokens that were served from a Prompt Cache hit?

- **A.** `100 * (m_cache_read / (m_uncached + m_cache_read + m_cache_write))`
- **B.** `100 * (m_cache_write / (m_uncached + m_out))`
- **C.** `100 * ((m_cache_read + m_out) / m_uncached)`
- **D.** `100 * (m_uncached / (m_cache_read - m_cache_write))`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
In Amazon Bedrock (`Converse` usage metadata and `AWS/Bedrock` CloudWatch metrics), total input tokens processed across a request are partitioned into three mutually exclusive counters:
1. `InputTokenCount` (`m_uncached`): Standard input tokens not read from or written to cache.
2. `CacheReadInputTokenCount` (`m_cache_read`): Input tokens served from a valid Prompt Cache hit.
3. `CacheWriteInputTokenCount` (`m_cache_write`): Input tokens evaluated during prefill and written to a Prompt Cache checkpoint.

Therefore, total input tokens = `m_uncached + m_cache_read + m_cache_write`, and the **Prompt Cache Input Token Hit Ratio (%)** is:
$$\text{CacheHitRatio (\%)} = 100 \times \frac{m_{\text{cache\_read}}}{m_{\text{uncached}} + m_{\text{cache\_read}} + m_{\text{cache\_write}}}$$

#### Distractor Analysis
- **Why B is incorrect:** Uses `m_cache_write` (cache misses/writes) in the numerator and includes `m_out` (output tokens, which are never cached by Prompt Caching) in the denominator.
- **Why C is incorrect:** Divides by `m_uncached` alone (which can yield >1,000% when most tokens are cached!) and includes output tokens in the numerator.
- **Why D is incorrect:** Measures uncached tokens divided by the difference of reads and writes, which can be negative or divide by zero.
</details>

---

### Question 6
An engineering team operates a complex agentic workflow on AWS ECS Fargate where a single user request triggers an orchestrator service that calls:
1. Amazon Titan Text Embeddings V2 (`bedrock-runtime:InvokeModel`)
2. Amazon OpenSearch Serverless (`aoss` HTTP query)
3. Anthropic Claude 3.5 Sonnet Turn 1 (`bedrock-runtime:ConverseStream` -> requests a tool execution)
4. An internal Order Management REST microservice hosted on Amazon EKS
5. Anthropic Claude 3.5 Sonnet Turn 2 (`bedrock-runtime:ConverseStream` -> synthesizes final response)

During peak hours, 5% of user requests exceed the 8-second SLA, but the `AWS/Bedrock` `InvocationLatency` p99 metric in CloudWatch is only 1.4 seconds. The team cannot tell whether the 8-second tail latency is caused by ECS thread contention, OpenSearch queries, the EKS Order microservice, or multiple sequential tool-call loops inside the agent.

Which observability solution provides end-to-end correlated trace waterfalls with GenAI-specific token and TTFT attributes across all five distributed components?

- **A.** Enable S3 Server Access Logging on the bucket that stores static website images.
- **B.** Create a CloudWatch alarm on ECS `CPUUtilization` averaging over 24-hour periods.
- **C.** Query AWS Cost and Usage Reports (CUR) in Amazon Athena once a week.
- **D.** Instrument the ECS orchestrator and EKS microservice using the **AWS Distro for OpenTelemetry (ADOT) SDK** / **Amazon CloudWatch Application Signals** (propagating W3C `traceparent` / AWS X-Ray trace headers across HTTP and `boto3` calls) and enrich the Bedrock child spans with OpenTelemetry GenAI semantic attributes (`gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.server.time_to_first_token_ms`).

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
Aggregate service metrics (`AWS/Bedrock` `InvocationLatency`) measure individual API calls in isolation—if a single user request triggers an agent loop that makes 4 sequential 1.4-second Bedrock calls plus a 2.5-second EKS REST call, each individual Bedrock call looks healthy (`1.4s`), while the user experiences `8.1s` of latency! Instrumenting with **OpenTelemetry (ADOT) + AWS X-Ray / CloudWatch Application Signals** links every child span (Embedding, OpenSearch, LLM Turn 1, EKS Tool Call, LLM Turn 2) under a single distributed `TraceId` waterfall and records `gen_ai.*` attributes (`input_tokens`, `output_tokens`, `ttft_ms`) on each LLM span.

#### Distractor Analysis
- **Why A is incorrect:** S3 Server Access Logs only record S3 object requests and have zero visibility into ECS, Bedrock, OpenSearch, or EKS API calls.
- **Why B is incorrect:** A 24-hour average CPU metric hides sub-second I/O bottlenecks and provides no per-request trace breakdown.
- **Why C is incorrect:** Billing reports (CUR) have a 24-hour delay and do not capture millisecond distributed latency traces.
</details>

---

### Question 7
An enterprise AI Governance board is tracking model lifecycle risk across 40 AWS accounts. AWS recently announced an upcoming deprecation date for a legacy version of a third-party Foundation Model on Amazon Bedrock, and the board wants to automatically detect any AWS account or application still invoking deprecated or legacy models so teams can migrate to newer model versions before the End-of-Life (EOL) cutoff.

Which native metric in the `AWS/Bedrock` CloudWatch namespace should the Governance team monitor and alarm on across their accounts?

- **A.** `InvocationClientErrors`
- **B.** `LegacyModelInvocations`
- **C.** `CacheWriteInputTokenCount`
- **D.** `ModelUnitUtilization`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
Amazon Bedrock emits the **`LegacyModelInvocations`** metric in the `AWS/Bedrock` namespace (dimensioned by `ModelId`) whenever an application invokes a Foundation Model version that has been marked as legacy / approaching End-of-Life (EOL) deprecation. Creating a CloudWatch alarm (`LegacyModelInvocations > 0`) via CloudWatch Cross-Account Observability allows platform governance teams to pinpoint and migrate workloads well ahead of model retirement dates.

#### Distractor Analysis
- **Why A is incorrect:** `InvocationClientErrors` only increments *after* a model is completely retired and begins throwing HTTP 400/404 errors (or when requests are malformed), which is too late for proactive migration.
- **Why C is incorrect:** `CacheWriteInputTokenCount` tracks Prompt Caching prefix writes.
- **Why D is incorrect:** `ModelUnitUtilization` is not the metric used for detecting legacy On-Demand model deprecation.
</details>

---

### Question 8
A DevOps engineer configures **Amazon Bedrock Model Invocation Logging** to deliver invocation logs to an **Amazon CloudWatch Logs** log group (`/aws/bedrock/modelinvocations`). The engineer wants to create a real-time CloudWatch Dashboard table via **CloudWatch Logs Insights** that ranks the top 10 IAM roles or application callers (`identity.arn`) consuming the highest total tokens (`inputTokenCount + outputTokenCount`) over the past 1 hour.

Which CloudWatch Logs Insights query syntax accurately extracts and ranks these callers from native Bedrock Model Invocation Logs?

- **A.** `SELECT identity.arn FROM /aws/bedrock/modelinvocations WHERE tokens > 1000 ORDER BY SQL_SUM()`
- **B.** `DESCRIBE TABLE bedrock_logs PARTITION BY identity.arn`
- **C.** `fields identity.arn | stats sum(input.inputTokenCount + output.outputTokenCount) as totalTokens, count(*) as invocations by identity.arn | sort totalTokens desc | limit 10`
- **D.** `grep "identity.arn" /var/log/bedrock | awk '{print $1}' | head -n 10`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
Amazon Bedrock Model Invocation Logs are delivered as structured JSON events containing `identity.arn`, `modelId`, `operation`, `input.inputTokenCount`, and `output.outputTokenCount` (alongside cache token counters). In **CloudWatch Logs Insights**, the query language uses pipe-delimited commands (`fields`, `filter`, `stats sum(...) as ... by ...`, `sort ... desc`, `limit 10`) to aggregate JSON properties across millions of log events in seconds.

#### Distractor Analysis
- **Why A is incorrect:** Uses invalid pseudo-SQL (`SQL_SUM()`) rather than valid CloudWatch Logs Insights syntax.
- **Why B is incorrect:** `DESCRIBE TABLE` is a DDL metadata statement, not an aggregation query.
- **Why D is incorrect:** `grep` and `awk` are Linux bash commands, not CloudWatch Logs Insights queries.
</details>

---

### Question 9
An SRE team manages a mission-critical customer support agent on Amazon Bedrock. Previously, they configured a simple static CloudWatch alarm that paged the on-call engineer whenever `InvocationLatency` (`Maximum`) exceeded `5,000 ms` in any 1-minute period. However, the on-call team suffers from severe **alarm fatigue** because once or twice an hour, a single user submits an unusually long prompt that takes 5.2 seconds (`Maximum > 5000ms`) even though 99.9% of concurrent requests are completing in 800ms (`p99 = 1,400ms`) and `InvocationThrottles = 0`. Furthermore, average latency naturally varies between daytime complex analytical queries and nighttime quick FAQ queries.

How should the SRE team re-architect their CloudWatch alarming strategy to eliminate single-request false-positive pages while still detecting genuine service degradation or quota saturation?

- **A.** Delete all CloudWatch alarms and rely on customers posting complaints on social media.
- **B.** Change the alarm statistic from `Maximum` to `Minimum` with a threshold of `10 ms`.
- **C.** Keep the `Maximum` statistic, set `EvaluationPeriods: 1`, and lower the threshold from `5,000 ms` to `500 ms`.
- **D.** Replace the `Maximum` statistic with a percentile statistic (**`p95` or `p99`**) paired with **CloudWatch Anomaly Detection** (`ANOMALY_DETECTION_BAND(m_lat_p99, 3)`) using `M out of N` datapoints (e.g., `2 out of 3` minutes), and combine it in a **CloudWatch Composite Alarm** with a token burn-rate / `InvocationThrottles` alarm so the pager only fires during sustained, correlated degradation.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
In LLM workloads where latency scales linearly with output token length, using the **`Maximum`** statistic on `InvocationLatency` with `EvaluationPeriods=1` is a classic observability anti-pattern because a single outlier request generating 1,500 tokens will breach the ceiling every time. Best practices dictate:
1. Using **`p95` or `p99` extended percentile statistics** to measure true tail latency across the population rather than a single outlier.
2. Using **CloudWatch Anomaly Detection** (`ANOMALY_DETECTION_BAND`) to adapt dynamically to diurnal (day/night) query complexity cycles.
3. Using **`M out of N` evaluation periods** (`DatapointsToAlarm: 2, EvaluationPeriods: 3`) and **CloudWatch Composite Alarms** (`ALARM(LatencyP99Anomaly) AND (ALARM(HighTokenBurn) OR ALARM(ThrottleRate))`) to suppress transient 1-minute blips.

#### Distractor Analysis
- **Why A is incorrect:** Removing monitoring entirely violates operational excellence requirements.
- **Why B is incorrect:** Alarming on `Minimum > 10ms` will stay in permanent `ALARM` state 24x7 because no LLM call completes in under 10 milliseconds.
- **Why C is incorrect:** Lowering a `Maximum` threshold to `500 ms` on a 1-minute window will page the engineer continuously every minute of the day.
</details>

---

### Question 10
A company uses **Amazon Bedrock Intelligent Prompt Routing** (`default-prompt-router/anthropic.claude:1`) in production to dynamically route requests between **Anthropic Claude 3.5 Sonnet** and **Anthropic Claude 3.5 Haiku**. The FinOps and ML Engineering teams want to track the exact daily routing split (for example, verifying that ~72% of requests are routed to Haiku and ~28% to Sonnet) and compare accuracy feedback scores between the two routed models.

Where can the application code or logging pipeline inspect which specific underlying Foundation Model executed each request routed by the Intelligent Prompt Router?

- **A.** Inspect the `trace.promptRouter.invokedModelId` field returned inside the `Converse` response (or inside the `metadata` event of `ConverseStream` / Bedrock Model Invocation Logs), which records the exact backing model ARN invoked for that turn.
- **B.** Intelligent Prompt Routing is a black-box hardware multiplexer that never exposes which backing model executed the request in any API response or log.
- **C.** Query the AWS Route 53 DNS resolver query logs to see whether the CNAME resolved to `haiku.amazonaws.com` or `sonnet.amazonaws.com`.
- **D.** Count the number of characters in the output string; if the output length is an even number, it was generated by Haiku, and if odd, by Sonnet.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
When invoking an **Amazon Bedrock Intelligent Prompt Router**, the `Converse` response payload (and the final `metadata` event in `ConverseStream`, as well as Bedrock Model Invocation Logs) includes a `trace` object:
```json
"trace": {
  "promptRouter": {
    "invokedModelId": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-haiku-20241022-v1:0"
  }
}
```
Additionally, CloudWatch `AWS/Bedrock` metrics are emitted for the invoked model, allowing full visibility into the exact Haiku-vs-Sonnet routing ratio and cost savings.

#### Distractor Analysis
- **Why B is incorrect:** Bedrock explicitly surfaces the `invokedModelId` in both runtime responses and invocation logs.
- **Why C is incorrect:** All Bedrock runtime calls hit the same regional `bedrock-runtime.<region>.amazonaws.com` endpoint; model selection does not happen via DNS CNAMEs.
- **Why D is incorrect:** Output character parity has no relation to which model generated the response.
</details>

---

### Question 11
A healthcare organization has enabled **Amazon Bedrock Model Invocation Logging** to an Amazon S3 bucket and an Amazon CloudWatch Logs log group in `us-east-1` for debugging latency issues. During a HIPAA compliance audit, the Chief Information Security Officer (CISO) notes that while the CloudWatch Logs log group is needed by the SRE team to monitor token counts, latency, and error codes, general SREs with read access to CloudWatch Logs must **not** be able to view unmasked Patient Health Information (PHI) such as SSNs, patient names, or medical record numbers that might appear inside the logged `inputBodyJson` or `outputBodyJson` fields.

Which native Amazon CloudWatch capability allows the organization to automatically detect and mask sensitive PHI/PII inside `/aws/bedrock/modelinvocations` log events before any unauthorized IAM principal can read them in CloudWatch Logs Insights?

- **A.** Rename the CloudWatch Logs log group to `/aws/public/unencrypted` so HIPAA auditors ignore it.
- **B.** Configure an Amazon Route 53 health check on the CloudWatch Logs endpoint to drop packets containing numbers.
- **C.** Enable an **Amazon CloudWatch Logs Data Protection** policy on the `/aws/bedrock/modelinvocations` log group configured with managed Healthcare/PHI and PII data identifiers (`Name`, `SSN`, `Address`, `HealthInsuranceID`), which automatically masks matching sensitive data in transit/at rest for all readers who lack the explicit `logs:Unmask` IAM permission.
- **D.** Set `inferenceConfig={"stopSequences": ["SSN"]}` on the Bedrock call.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon CloudWatch Logs Data Protection** uses pattern matching and machine learning to automatically scan ingested log events (including **Amazon Bedrock Model Invocation Logs**) for over 100 managed sensitive data identifiers (PII, PHI, financial credentials). Once a Data Protection Policy is attached to the log group, any sensitive PHI fields inside the JSON payload are **automatically redacted/masked (`*****`)** when viewed in the CloudWatch console, CLI, or Logs Insights, unless the caller's IAM role has been explicitly granted the tightly restricted `logs:Unmask` permission for compliance investigations.

#### Distractor Analysis
- **Why A is incorrect:** Renaming a log group provides zero data protection and fails the HIPAA audit.
- **Why B is incorrect:** Route 53 health checks monitor endpoint reachability and do not inspect or redact CloudWatch log payloads.
- **Why D is incorrect:** A stop sequence only stops output generation if the literal string `"SSN"` is emitted; it does not redact input prompts or patient names/numbers in logs.
</details>

---

### Question 12
A global enterprise operates workloads in `us-east-1` (N. Virginia) and recently updated its production GenAI applications in `us-east-1` to invoke the System-Defined **US Cross-Region Inference (CRIS) Profile** (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`), which dynamically routes inference execution across `us-east-1`, `us-east-2` (Ohio), and `us-west-2` (Oregon).

The Observability Architect needs to know **which AWS Region** they must configure their CloudWatch Alarms (`AWS/Bedrock`), CloudTrail Data Event queries, and Bedrock Model Invocation Logging destinations in to capture 100% of the invocations made by their `us-east-1` application through the CRIS profile. Which statement is true?

- **A.** They must deploy duplicate CloudWatch Alarms and Model Invocation Logging buckets in all three regions (`us-east-1`, `us-east-2`, `us-west-2`) and manually stitch the logs together, because metrics and logs are only written in whichever destination region executed the GPU compute.
- **B.** All `AWS/Bedrock` CloudWatch metrics, AWS CloudTrail API audit events, and Amazon Bedrock Model Invocation Logs are centralized and emitted in the **calling (source) region (`us-east-1`)** where the request entered the CRIS Inference Profile, regardless of whether the underlying stateless compute was routed to `us-east-2` or `us-west-2`.
- **C.** Cross-Region Inference disables all CloudWatch metrics and Model Invocation Logging to save cross-region network bandwidth.
- **D.** All CRIS logs and metrics are written exclusively to `af-south-1` (Cape Town).

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
One of the core architectural advantages of **Amazon Bedrock Cross-Region Inference (CRIS)** is **Source-Region Control & Observability Anchoring**:
Even when Bedrock transparently routes a request from `us-east-1` to available accelerator capacity in `us-west-2` or `us-east-2`, all observability and governance artifacts—including **`AWS/Bedrock` CloudWatch Metrics**, **AWS CloudTrail logs**, **Bedrock Model Invocation Logs**, and **Billing charges**—are recorded in the **source region (`us-east-1`)** where the API call was made. You do not need to stitch logs or metrics across peer regions.

#### Distractor Analysis
- **Why A is incorrect:** Bedrock abstracts the destination compute routing and aggregates all metrics and invocation logs back into the calling source region (`us-east-1`).
- **Why C is incorrect:** CRIS fully supports CloudWatch metrics, CloudTrail, and Model Invocation Logging.
- **Why D is incorrect:** `af-south-1` is completely unrelated to the `us.` CRIS profile.
</details>

---

### Question 13 (Multiple Response — Choose TWO)
A FinOps team manages an AWS account where five autonomous AI research agents (`Bedrock Agents`) have been deployed for internal data analysts. Last weekend, a developer accidentally triggered an infinite recursive loop in one of the agents, which ran undetected for 48 hours and consumed `$14,500` in unexpected Amazon Bedrock token charges before anyone noticed on Monday morning.

The CTO requires a defense-in-depth cost governance architecture that provides **both** automated billing anomaly alerts AND an automated technical circuit breaker that blocks or throttles the offending agent within minutes if its hourly token burn rate spikes abnormally. Which **TWO** solutions should the architect implement together? *(Choose TWO.)*

- **A.** Assign each agent a dedicated **Application Inference Profile** tagged with `AgentName`, and configure **AWS Cost Anomaly Detection** + **AWS Budgets** (with daily/monthly thresholds and SNS/Chatbot alerts) scoped to those cost allocation tags.
- **B.** Wait for the monthly PDF invoice to arrive on the 3rd business day of the following month and review it manually.
- **C.** Increase the regional On-Demand TPM quota by 1,000% so the infinite loop runs faster.
- **D.** Create a **CloudWatch Alarm** (1-minute period) on `InputTokenCount + OutputTokenCount` (`Sum`) for each agent's Application Inference Profile ID that triggers an **AWS Lambda circuit-breaker function** via Amazon SNS to automatically attach a deny-all IAM policy (or disable the Agent alias / revoke the profile permission) whenever the 5-minute token burn rate exceeds the agent's safety ceiling.
- **E.** Store the AWS root account credentials in the agent's system prompt so the agent can check its own billing console.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and D**

#### Explanation
- **Why A is correct:** Tagging each agent's **Application Inference Profile** and configuring **AWS Cost Anomaly Detection** + **AWS Budgets** provides native FinOps governance and financial anomaly alerting in the AWS Billing plane.
- **Why D is correct:** Because AWS Billing / Cost Explorer metrics update several times a day (not sub-minute), stopping a runaway infinite agent loop within *minutes* requires a real-time operational circuit breaker: a **1-minute CloudWatch Metric Math Alarm** on `InputTokenCount + OutputTokenCount` dimensioned by the agent's `ModelId` (Application Inference Profile ID) that invokes an **AWS Lambda remediation function** to quarantine the runaway agent role or alias immediately.

#### Distractor Analysis
- **Why B is incorrect:** Waiting for the end-of-month PDF invoice allows runaway loops to burn budget for weeks.
- **Why C is incorrect:** Raising the TPM quota by 10x allows a runaway loop to burn `$145,000` instead of `$14,500` over the weekend!
- **Why E is incorrect:** Embedding root credentials in a prompt is a catastrophic security violation.
</details>

---

### Question 14 (Multiple Response — Choose TWO)
An enterprise ML observability team is building an executive Quality & Efficiency Scorecard for their customer-facing RAG assistant. They have enabled **Amazon Bedrock Model Invocation Logging** to an Amazon S3 data lake (partitioned by date) and **Amazon Bedrock Guardrails** with `trace="ENABLED"`. They want to run automated daily SQL analytics in **Amazon Athena** over the S3 invocation logs without impacting live production latency.

Which **TWO** operational insights can the team directly extract by querying the Amazon Bedrock Model Invocation Logs in S3 with Amazon Athena? *(Choose TWO.)*

- **A.** The physical CPU fan speed (RPM) and datacenter room temperature of the AWS facility hosting the Bedrock servers.
- **B.** The exact **Guardrail intervention rates, triggered policy types** (e.g., Topic Policy vs. Content Filter vs. Contextual Grounding check scores), and **guardrail evaluation latency overhead** recorded inside the `output.outputBodyJson` / guardrail trace fields.
- **C.** The private weights and bias matrices (`safetensors`) of Anthropic Claude 3.5 Sonnet.
- **D.** The credit card CVV numbers of other AWS customers sharing the same region.
- **E.** Detailed **token distribution histograms** (`inputTokenCount`, `outputTokenCount`, `cacheReadInputTokenCount`, `cacheWriteInputTokenCount`) correlated with `requestMetadata` attributes (such as `tenant_id` or `prompt_template_version`) to identify which prompt templates have the lowest Prompt Cache hit efficiency.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: B and E**

#### Explanation
- **Why B is correct:** When `trace="ENABLED"` is passed with `guardrailConfig`, Bedrock Model Invocation Logs record the complete Guardrail trace—including exact contextual grounding scores, PII/content filter assessments, intervened policy names, and the millisecond processing time of each guardrail policy.
- **Why E is correct:** Model Invocation Logs capture per-request token counters (`inputTokenCount`, `outputTokenCount`, `cacheReadInputTokenCount`, `cacheWriteInputTokenCount`) alongside any custom key-value pairs passed in `requestMetadata`, enabling Athena SQL queries to correlate prompt template versions with cache hit ratios and token bloat.

#### Distractor Analysis
- **Why A is incorrect:** AWS manages physical datacenter telemetry under the Shared Responsibility Model and never exposes hardware fan speeds in customer logs.
- **Why C is incorrect:** Proprietary model weights are never exposed in invocation logs.
- **Why D is incorrect:** Bedrock enforces strict multi-tenant isolation; customer logs only ever contain invocations from your own AWS account.
</details>

---

### Question 15 (Multiple Response — Choose TWO)
An engineering team hosts a critical open-weight LLM on an **Amazon SageMaker Real-Time Endpoint** using **SageMaker Inference Components** (where multiple fine-tuned LoRA adapters / model copies share a fleet of `ml.g5.48xlarge` instances). To optimize cost and performance, the team needs to configure **Application Auto Scaling** and **Amazon CloudWatch Alarms** using the native `AWS/SageMaker` metrics emitted for endpoints and inference components.

Which **TWO** CloudWatch metrics and monitoring configurations are appropriate for monitoring and auto-scaling this SageMaker LLM endpoint? *(Choose TWO.)*

- **A.** Monitor `CacheReadInputTokenCount` in the `AWS/Bedrock` namespace to auto-scale the SageMaker `ml.g5.48xlarge` instances.
- **B.** Use `AWS/RDS` `ReadIOPS` to scale the SageMaker `ml.g5` instances.
- **C.** Configure Application Auto Scaling target tracking on `InvocationsPerCopy` (or `ConcurrentRequestsPerCopy` / `ConcurrentRequestsPerModel`) dimensioned by `InferenceComponentName` so SageMaker scales out inference component copies before in-flight batch queues saturate GPU KV-cache memory.
- **D.** Monitor `ModelLatency` (`p95` / `p99`, measuring container inference processing time) and `OverheadLatency` (measuring SageMaker routing/queueing overhead) in the `AWS/SageMaker` namespace, along with GPU memory/compute utilization (`GPUUtilization`, `GPUMemoryUtilization`) in the `/aws/sagemaker/Endpoints` namespace.
- **E.** Poll `sagemaker.describe_endpoint()` in a `while True` loop every 1 millisecond from 1,000 client laptops to measure GPU utilization.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: C and D**

#### Explanation
- **Why C is correct:** When using **SageMaker Inference Components**, scaling on `InvocationsPerCopy` or `ConcurrentRequestsPerCopy` (dimensioned by `InferenceComponentName`) allows Application Auto Scaling to dynamically add or remove model copies across the instance fleet based on actual concurrent request pressure.
- **Why D is correct:** For SageMaker Real-Time Endpoints, `ModelLatency` (container execution time) and `OverheadLatency` (SageMaker infrastructure + queueing time) in `AWS/SageMaker`—paired with hardware metrics `GPUUtilization` and `GPUMemoryUtilization` in `/aws/sagemaker/Endpoints`—provide the exact separation between container compute bottlenecks and request queueing saturation.

#### Distractor Analysis
- **Why A is incorrect:** `AWS/Bedrock` metrics only apply to Amazon Bedrock serverless/provisioned invocations, not self-hosted SageMaker endpoints.
- **Why B is incorrect:** `AWS/RDS` metrics monitor relational databases, not SageMaker GPU endpoints.
- **Why E is incorrect:** Polling control-plane `DescribeEndpoint` APIs at 1,000 RPS triggers control-plane API throttling (`ThrottlingException`) and returns only endpoint status (`InService`), not GPU utilization.
</details>
