# Domain 4 — Quiz 2: Optimize Application Performance (Task 4.2)

**Instructions:** This 15-question Staff/Principal-level quiz tests **Task 4.2 (Optimize Application Performance)** of the **AWS Certified Generative AI Developer - Professional (AIP-C01)** exam. Questions include both Single-Response (`A`–`D`) and Multiple-Response (`Choose TWO`, `A`–`E`) items.

---

### Question 1
A healthcare analytics company operates an EHR document triage pipeline on Amazon Bedrock using Anthropic Claude 3.5 Sonnet. Each request passes a 45,000-token patient chart as input and asks the model to output a 15-token JSON triage priority object (`{"urgency": "HIGH", "department": "Cardiology"}`). End-to-end latency currently averages **5.8 seconds**. To speed up the pipeline, a developer switches the API call from `Converse` to `ConverseStream`, expecting total triage latency to drop below 1.0 second. However, after deployment, total triage latency remains **5.8 seconds**, with the first streamed token arriving at **5.6 seconds** and the remaining 14 tokens arriving in **0.2 seconds**.

Why did switching to `ConverseStream` fail to improve performance, and which architectural change will actually reduce the 5.6-second bottleneck?

- **A.** `ConverseStream` only improves perceived latency during the **output decoding phase (Inter-Token Latency)**, whereas this workload's 5.6-second bottleneck is **Time to First Token (TTFT)** caused by prefill computation over 45,000 input tokens. If multiple triage queries run against the same patient chart, place a `cachePoint` after the 45,000-token chart; if each chart is queried only once, use RAG / Bedrock Rerank to pass only relevant clinical sections instead of the full 45,000-token chart.
- **B.** `ConverseStream` throttles the first token by 5 seconds unless `inferenceConfig={"temperature": 1.0}` is explicitly configured.
- **C.** The bottleneck is Inter-Token Latency (ITL) caused by generating 15 output tokens. Switching from JSON output to YAML output will reduce latency from 5.8 seconds to 0.5 seconds.
- **D.** `ConverseStream` buffers up to 64 KB of output tokens before flushing the first byte to the client. Pad the output prompt to generate at least 10,000 tokens so the buffer flushes immediately.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
LLM inference latency consists of **Time to First Token (TTFT — Prefill Phase)** and **Inter-Token Latency (ITL — Decode Phase)**. With 45,000 input tokens and only 15 output tokens, **96.5% of total wall-clock time (5.6s of 5.8s) is spent in the compute-bound Prefill Phase** computing attention Key-Value ($K, V$) matrices over the 45,000 input tokens before the very first output token can be emitted. Response streaming (`ConverseStream`) delivers output tokens incrementally *after* TTFT completes, so it cannot speed up TTFT itself. To reduce TTFT, you must either reuse precomputed KV tensors via **Prompt Caching (`cachePoint`)** (for repeated queries on the same chart) or **reduce the input token length** via selective section retrieval / RAG reranking.

#### Distractor Analysis
- **Why B is incorrect:** `ConverseStream` never imposes an artificial 5-second delay based on `temperature`.
- **Why C is incorrect:** 15 tokens of decoding takes only ~200 milliseconds (`0.2s`); the format (JSON vs YAML) has zero impact on the 5.6-second input prefill bottleneck.
- **Why D is incorrect:** `ConverseStream` flushes tokens immediately per generated token chunk without a 64 KB output buffer; asking for 10,000 output tokens would dramatically *increase* total latency.
</details>

---

### Question 2
A consumer fintech application deployed in `us-east-1` uses Anthropic Claude 3.5 Sonnet (`anthropic.claude-3-5-sonnet-20241022-v2:0`) on Amazon Bedrock to provide interactive tax-filing guidance. Every weekday between 7:00 PM and 10:00 PM EST during tax season, traffic surges by 350%, causing thousands of requests to fail with HTTP 429 `ThrottlingException` due to regional On-Demand RPM and TPM quota limits. Traffic during the remaining 21 hours of the day is less than 15% of the peak.

The architecture team needs an immediate, zero-infrastructure-overhead solution that at least doubles effective On-Demand throughput resilience during evening surges **without** paying for idle 24x7 Provisioned Throughput Model Units or paying any cross-region routing surcharge. Which change should they implement?

- **A.** Deploy a custom AWS Lambda proxy that catches `ThrottlingException` and copies the prompt into an Amazon S3 bucket for asynchronous Amazon Bedrock Batch Inference.
- **B.** Update the `modelId` in the `Converse` / `ConverseStream` API call from the single-region model ID to the System-Defined **US Cross-Region Inference (CRIS) Profile** (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`) and grant the IAM execution role `bedrock:InvokeModel*` permissions on the profile and underlying models across the mapped US peer regions (`us-east-1`, `us-east-2`, `us-west-2`).
- **C.** Purchase 20 Model Units (MUs) of Provisioned Throughput with a 6-month commitment in `us-east-1`.
- **D.** Increase the `boto3` client `max_attempts` retry configuration to `500` with a fixed 30-second sleep interval on synchronous API Gateway requests.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
**System-Defined Cross-Region Inference (CRIS) Profiles** (such as `us.anthropic.claude-3-5-sonnet-20241022-v2:0`) dynamically distribute On-Demand inference traffic across multiple AWS Regions within the geographic boundary (`us-east-1`, `us-east-2`, `us-west-2`). CRIS provides **up to 2x+ the effective RPM and TPM quotas** of a single region, absorbs unplanned traffic bursts seamlessly, and incurs **zero additional routing or cross-region data transfer charges** (requests are billed at the calling region's standard On-Demand token price).

#### Distractor Analysis
- **Why A is incorrect:** Batch Inference is an asynchronous offline queue with up to a 24-hour completion window; it cannot serve an interactive live user waiting in a tax-filing chat UI.
- **Why C is incorrect:** Because traffic is below 15% for 21 hours a day, purchasing 24x7 Provisioned Throughput on a 6-month commitment wastes >80% of committed capacity.
- **Why D is incorrect:** Amazon API Gateway has a hard maximum integration timeout of **29 seconds** (unless raised via quota override on specific endpoints), and sleeping 30 seconds across 500 retries would block users for hours and exhaust Lambda concurrency.
</details>

---

### Question 3
An interactive AI voice agent uses Amazon Bedrock `ConverseStream` with Anthropic Claude 3.5 Haiku and attaches an **Amazon Bedrock Guardrail** (`guardrailIdentifier`, `guardrailVersion`) to enforce PII masking and topic restrictions. During live user testing, customers report an unnatural, jarring experience: instead of hearing smooth, continuous speech synthesis starting within 400ms, the voice agent pauses silently for 1.8 to 2.5 seconds at a time, bursts out a full paragraph of audio, pauses again for 2 seconds, and bursts out the next paragraph.

What is causing this choppy "burst-and-pause" streaming behavior, and how should the developer resolve it while keeping the Bedrock Guardrail active?

- **A.** `ConverseStream` only emits events after a newline (`\n\n`) character is generated. Modify the system prompt to insert a newline after every single word.
- **B.** Anthropic Claude 3.5 Haiku uses non-autoregressive batch generation that produces 100 tokens simultaneously every 2.5 seconds. Switch the model to Claude 3 Opus to get token-by-token generation.
- **C.** By default, Bedrock Guardrails on `ConverseStream` run in `SYNCHRONOUS` mode, which buffers generated token chunks on the server until Guardrail validation completes before releasing them to the stream. Set `guardrailConfig={"streamProcessingMode": "ASYNCHRONOUS", ...}` so Bedrock streams tokens immediately to the client while evaluating buffered token windows in parallel in the background.
- **D.** Remove the `guardrailConfig` parameter from `ConverseStream` and instead run a nightly Amazon Macie batch scan over the recorded S3 voice audio files.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
When an **Amazon Bedrock Guardrail** is attached to `ConverseStream` (or `InvokeModelWithResponseStream`), the default `streamProcessingMode` is **`SYNCHRONOUS`**. In `SYNCHRONOUS` mode, Bedrock holds back (buffers) generated tokens until a full evaluation segment is accumulated and verified by all configured Guardrail policies before sending those tokens down the wire—creating noticeable 1.5–2.5 second stalls between bursts. Setting **`streamProcessingMode="ASYNCHRONOUS"`** releases generated tokens to the client stream immediately with zero buffering delay while the Guardrail inspects segments asynchronously in parallel (emitting an immediate stream termination/intervention event if a policy violation is detected).

#### Distractor Analysis
- **Why A is incorrect:** `ConverseStream` emits subword token deltas continuously as they are decoded, regardless of newline characters.
- **Why B is incorrect:** Claude 3.5 Haiku is a standard autoregressive transformer that generates tokens sequentially at very high TPS; switching to a much larger model (Claude 3 Opus) would *increase* Inter-Token Latency (ITL).
- **Why D is incorrect:** Removing the Guardrail entirely violates the requirement to keep real-time Guardrail protection active during the live conversation.
</details>

---

### Question 4
A software company is building a real-time IDE code-completion extension backed by Meta Llama 3.1 70B Instruct on Amazon Bedrock. Developers expect inline ghost-text suggestions to complete within **600ms end-to-end**, which requires maximizing output Tokens Per Second (TPS) / minimizing Inter-Token Latency (ITL). The team wants to achieve the fastest possible hardware inference speed on Amazon Bedrock without provisioning or managing Amazon SageMaker GPU endpoints.

Which configuration parameter should the team add to their `ConverseStream` or `InvokeModelWithResponseStream` calls?

- **A.** `inferenceConfig={"topK": 500, "temperature": 1.0}`
- **B.** `additionalModelRequestFields={"useSpotCapacity": True}`
- **C.** `requestMetadata={"priorityTier": "ULTRA_LOW_LATENCY"}`
- **D.** `performanceConfig={"latency": "optimized"}`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon Bedrock Latency-Optimized Inference** is enabled by passing `performanceConfig={"latency": "optimized"}` in the `Converse`, `ConverseStream`, `InvokeModel`, or `InvokeModelWithResponseStream` API request. Bedrock routes latency-optimized requests to specialized hardware and optimized inference compilation paths that significantly increase Tokens Per Second (TPS) and reduce both TTFT and Inter-Token Latency (ITL) with zero infrastructure management and identical model accuracy.

#### Distractor Analysis
- **Why A is incorrect:** Sampling parameters (`topK`, `temperature`) adjust token probability selection and do not accelerate hardware decoding speed.
- **Why B is incorrect:** `useSpotCapacity` is not a valid Bedrock parameter.
- **Why C is incorrect:** `requestMetadata` is a key-value map logged for observability/filtering in Bedrock logs and Guardrails; it does not alter hardware inference speed.
</details>

---

### Question 5
An AI engineering team hosts a custom fine-tuned 70-billion-parameter open-weight LLM (`BF16` precision, 140 GB model weights) on an **Amazon SageMaker Real-Time Endpoint** using an `ml.p4d.24xlarge` instance (8 x NVIDIA A100 40GB GPUs = 320 GB total GPU HBM). Using a legacy HuggingFace `transformers` serving script with static batching, the endpoint crashes with `CUDA out of memory (OOM)` whenever concurrent requests exceed 6 users, and GPU utilization averages only 28% because fast 50-token responses sit idle waiting for a single 2,048-token response in the same batch to finish.

Which combination of serving engine and optimization techniques on Amazon SageMaker will eliminate the KV-cache memory fragmentation, prevent GPU idle bubble time, and increase concurrent throughput by 3x–4x?

- **A.** Keep the legacy HuggingFace script, set `batch_size=1`, and mount a 2 TB Amazon EFS volume as Linux swap space for GPU memory.
- **B.** Migrate to the **AWS Deep Learning Container for Large Model Inference (LMI)** powered by **vLLM** or **TensorRT-LLM** configured with **PagedAttention**, **Continuous Batching (iteration-level scheduling)**, **`option.tensor_parallel_degree=8`**, and **AWQ `INT4` or `FP8` weight quantization**.
- **C.** Switch from Tensor Parallelism to Data Parallelism (`option.data_parallel_degree=8`) so a full unquantized 140 GB copy of the model is loaded onto each individual 40 GB A100 GPU.
- **D.** Deploy the 140 GB model onto an AWS Lambda function with 10 GB of memory using ONNX Runtime CPU execution.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
Serving a 70B LLM efficiently on multi-GPU SageMaker instances (`ml.p4d.24xlarge` or `ml.g5.48xlarge`) requires the **SageMaker Large Model Inference (LMI)** container (`vLLM` or `TensorRT-LLM`) with four complementary optimizations:
1. **Tensor Parallelism (`tensor_parallel_degree=8`)**: Shards the weight matrices across all 8 GPUs over high-bandwidth NVLink, reducing per-GPU weight footprint and lowering Inter-Token Latency (ITL).
2. **Quantization (`AWQ INT4` or `FP8`)**: Shrinks total weight memory from 140 GB down to ~35–70 GB, freeing massive amounts of GPU HBM for concurrent KV caches and cutting memory bandwidth bottlenecks.
3. **PagedAttention**: Partitions the KV cache into non-contiguous memory blocks, eliminating the 60%–80% contiguous HBM fragmentation waste that caused the `CUDA OOM` at concurrency > 6.
4. **Continuous Batching**: Evicts finished short sequences immediately at each token iteration and inserts new waiting requests, eliminating static batching "bubble time."

#### Distractor Analysis
- **Why A is incorrect:** Setting `batch_size=1` destroys throughput, and swapping GPU tensors over network-attached EFS degrades token generation speed by 1,000x+.
- **Why C is incorrect:** A 140 GB unquantized model cannot fit onto a single 40 GB GPU; Data Parallelism (`dp=8`) without Tensor Parallelism immediately fails with `CUDA OOM` on model load.
- **Why D is incorrect:** AWS Lambda has a maximum memory limit of 10 GB and no GPU access, making it physically impossible to load a 140 GB model.
</details>

---

### Question 6
A quantitative hedge fund runs a custom 70B parameter LLM on an Amazon SageMaker `ml.p5.48xlarge` endpoint (8 x NVIDIA H100 GPUs). Because each analyst query runs with a concurrency of 1 to 2 requests and generates long 1,500-token financial syntheses, the workload is strictly **memory-bandwidth bound during autoregressive decoding (Inter-Token Latency)**. The fund requires **100% mathematically identical output token distributions** to the un-approximated 70B model (no lossy weight pruning or distillation degradation), while doubling Tokens Per Second (TPS).

Which inference optimization technique in TensorRT-LLM / vLLM satisfies this requirement?

- **A.** **2-Bit Extreme Weight Quantization (`INT2`)**, which compresses the 70B weights by 87.5% with zero change to the output probability distribution.
- **B.** **Prompt Caching (`cachePoint`)**, which speeds up the generation of all 1,500 output tokens by 3x.
- **C.** **Speculative Decoding**, pairing a small, fast Draft Model (such as a 1B/8B model from the same family or Medusa draft heads) that proposes $K=5$ lookahead tokens per step with the 70B Target Model verifying all $K$ candidate tokens in a single parallel forward pass.
- **D.** **Static Batching with a 500ms Queue Delay**, holding incoming analyst requests in a queue for 500ms before starting inference.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
At low batch sizes, autoregressive decoding is bottlenecked by reading 70B model weights from GPU HBM for *every single output token*. **Speculative Decoding** solves this by using a tiny **Draft Model** (or Medusa heads) to rapidly draft $K$ candidate tokens (e.g., 4–5 tokens) and then passing all $K$ drafted tokens into the 70B **Target Model** in a **single parallel forward pass** (taking nearly the same time as generating 1 token because GPU compute is underutilized at small batch sizes). Using rejection sampling against the Target Model's exact logits, Speculative Decoding guarantees a **100% mathematically identical output distribution (lossless)** while accelerating TPS by **2x to 3x**.

#### Distractor Analysis
- **Why A is incorrect:** 2-bit (`INT2`) quantization is severely lossy and significantly degrades model perplexity and reasoning accuracy.
- **Why B is incorrect:** Prompt Caching only accelerates the **input prefill phase (TTFT)**; it does not reduce the number of HBM weight reads required during the 1,500-token output decode loop.
- **Why D is incorrect:** Adding a 500ms queue delay *worsens* end-to-end latency when concurrency is only 1–2 requests.
</details>

---

### Question 7
A machine learning platform team is migrating a high-volume open-weight Llama 3.1 8B Instruct service from GPU-based `ml.g5.12xlarge` instances to **AWS Inferentia2 (`ml.inf2.24xlarge`)** instances on Amazon SageMaker to achieve 40% better price-performance. When deploying the model using the HuggingFace Optimum Neuron / SageMaker DJL NeuronX container, what architectural requirement of the **AWS Neuron SDK (`torch-neuronx`)** must the engineer account for during model compilation and endpoint deployment?

- **A.** Inferentia2 NeuronCores only support 32-bit floating point (`FP32`) weights and automatically reject `BF16` or `FP16` models.
- **B.** Ahead-of-Time (AOT) compilation with the AWS Neuron compiler compiles the model into a static execution graph (`.neff` — Neuron Executable File Format) bound to specific compile-time constants such as **`tensor_parallel_degree` (number of NeuronCores), maximum `batch_size`, and maximum `sequence_length` (context window)**; changing these structural dimensions later requires recompiling the `.neff` artifact (and storing the compiled graph in S3 / HuggingFace Neuron Cache to avoid 25-minute compilation cold starts on endpoint scale-out).
- **C.** Inferentia2 instances do not support intra-instance chip-to-chip communication, so any model exceeding 16 GB of HBM must shard layers over TCP/IP across multiple separate EC2 instances.
- **D.** Inferentia2 instances can only execute Amazon Bedrock Batch Inference jobs and cannot be used behind real-time Amazon SageMaker endpoints.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
Unlike eager-mode PyTorch on CUDA GPUs, **AWS Inferentia2 (`ml.inf2`)** executes Ahead-of-Time (AOT) compiled **Neuron Executable File Format (`.neff`)** graphs generated by the **AWS Neuron Compiler (`neuronx-cc` / `torch-neuronx`)**. During compilation, you must specify static graph boundaries including `tensor_parallel_degree` (mapping to physical NeuronCores), `batch_size`, and `sequence_length` (or compiled bucket sizes). Furthermore, in production auto-scaling, you should **pre-compile the `.neff` artifacts and store them in Amazon S3** (setting `OPTION_NEURON_COMPILE_CACHE_URL`) so newly launched `ml.inf2` instances load the compiled `.neff` in seconds rather than compiling from scratch for 20+ minutes during a traffic scale-out!

#### Distractor Analysis
- **Why A is incorrect:** AWS Inferentia2 natively accelerates `BF16`, `FP16`, `FP8`, and `INT8` data types at hardware speed.
- **Why C is incorrect:** `ml.inf2.24xlarge` and `ml.inf2.48xlarge` feature ultra-high-speed **NeuronLink** inter-chip interconnects specifically designed for low-latency Tensor Parallelism across up to 24 NeuronCores (192 GB–384 GB total HBM) within a single instance.
- **Why D is incorrect:** `ml.inf2` is a first-class instance family for Amazon SageMaker Real-Time Endpoints.
</details>

---

### Question 8
An enterprise RAG system uses **Amazon Aurora PostgreSQL** with the `pgvector` extension (`HNSW` index) on a `db.r6g.2xlarge` instance (64 GB RAM). For the first six months, the vector table held 8 million 1,024-dimensional embeddings (~38 GB HNSW index), and $k$-NN queries consistently completed in **11 milliseconds** with 98% recall. Over the past month, a data expansion increased the table to 22 million embeddings (~105 GB HNSW index). Immediately after the index size exceeded the instance's `shared_buffers` RAM, p95 vector query latency degraded from **11ms to 680ms**, causing severe TTFT spikes across the entire GenAI application.

The database team wants to restore sub-25ms vector query latency **without** losing data and without paying to over-provision a massive `db.r6g.16xlarge` (512 GB RAM) instance just to hold the uncompressed FP32 index in memory. Which combination of Aurora and `pgvector` optimizations should they implement?

- **A.** Drop the `HNSW` index completely and run sequential full-table scans (`enable_indexscan = off`) using Aurora Parallel Query.
- **B.** Keep the `db.r6g.2xlarge` instance, increase `hnsw.ef_search` from `100` to `800`, and set `maintenance_work_mem` to `64MB`.
- **C.** Export the vectors to comma-separated text files in Amazon S3 and query them at runtime using Amazon S3 Select.
- **D.** Migrate the Aurora instance to an **Aurora Optimized Reads** instance class (`db.r6gd` / `db.r7gd` / `db.r8gd`) that uses local **NVMe SSD ephemeral storage** as a tiered cache for `pgvector` pages exceeding `shared_buffers`, convert the index to **`halfvec` (`FP16` scalar quantization)** to cut the index footprint by ~50%, and tune query-time `SET LOCAL hnsw.ef_search` (e.g., from `100` to `40`).

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
When a `pgvector` HNSW graph exceeds PostgreSQL `shared_buffers` memory, random graph traversal pointers cause thrashing against network-attached Aurora storage, spiking latency from ~11ms to >500ms. Three targeted optimizations solve this at a fraction of the cost of a 512 GB RAM instance:
1. **Half-Precision Quantization (`halfvec` / `FP16`)**: Shrinks the 105 GB `FP32` HNSW index in half to ~52 GB with negligible recall loss, allowing most or all of the index to fit back inside the 64 GB instance's RAM!
2. **Aurora Optimized Reads (`db.r6gd` / `db.r7gd` / `db.r8gd`)**: Provides a local high-IOPS **NVMe SSD tiered cache** that delivers up to **8x–9x faster vector read performance** whenever working sets spill outside `shared_buffers`.
3. **Query-Time `hnsw.ef_search` Tuning**: Lowering `hnsw.ef_search` reduces the number of graph nodes visited per query without rebuilding the index.

#### Distractor Analysis
- **Why A is incorrect:** A sequential scan over 22 million 1,024-dim vectors takes several seconds per query and saturates CPU cores.
- **Why B is incorrect:** Increasing `hnsw.ef_search` from 100 to 800 forces PostgreSQL to visit **8x more candidate nodes** per query, worsening disk paging and increasing latency to several seconds.
- **Why C is incorrect:** Scanning 22 million vectors in S3 CSV files takes tens of seconds per query.
</details>

---

### Question 9
A customer-facing GenAI travel planner executes four pre-inference operations in Python before calling `bedrock_runtime.converse_stream()`:
1. Calls `bedrock_runtime.apply_guardrail(source='INPUT')` on the user's message (**180ms**).
2. Calls `dynamodb.get_item()` to fetch the user's loyalty profile and seat preferences (**35ms**).
3. Calls `bedrock_runtime.invoke_model(modelId='amazon.titan-embed-text-v2:0')` to embed the user's query (**95ms**) followed by an OpenSearch $k$-NN query (**40ms** = **135ms total for RAG**).
4. Calls `dynamodb.query()` to load the last 5 conversation turns (**45ms**).

Currently, the Lambda handler runs Steps 1, 2, 3, and 4 sequentially in a single synchronous thread, adding **395ms** of pre-inference overhead before `converse_stream()` is even called. How can the engineer reduce this pre-inference overhead from **395ms to ~180ms** without removing any security or personalization checks?

- **A.** Combine Steps 1, 2, 3, and 4 into a single AWS Glue Spark job triggered synchronously by API Gateway.
- **B.** Remove the `apply_guardrail` check and DynamoDB lookups on Turn 1 and run them only after the user closes their browser tab.
- **C.** Execute Steps 1 (`apply_guardrail`), 2 (loyalty profile lookup), 3 (embedding + OpenSearch RAG), and 4 (chat history lookup) **concurrently in parallel** using `asyncio.gather()` or `concurrent.futures.ThreadPoolExecutor`; if Step 1 (`apply_guardrail`) returns `action == 'GUARDRAIL_INTERVENED'`, immediately abort and return the blocked response without invoking `converse_stream()`.
- **D.** Increase the Lambda function timeout from 30 seconds to 900 seconds.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
Steps 1, 2, 3, and 4 have **zero data dependencies on one another**—all four only require the incoming `user_id`, `session_id`, and raw `user_prompt`! Running independent I/O-bound network calls sequentially sums their latencies ($180 + 35 + 135 + 45 = 395\text{ ms}$). Running them concurrently via `ThreadPoolExecutor` or `asyncio` bounds total pre-inference time to the single slowest parallel branch ($\max(180, 35, 135, 45) = \mathbf{180\text{ ms}}$), shaving **215ms** off Time to First Token (TTFT) on every single request while still enforcing the Input Guardrail gate before invoking the LLM.

#### Distractor Analysis
- **Why A is incorrect:** AWS Glue Spark jobs have multi-second to multi-minute startup overhead and are designed for batch ETL, not millisecond real-time request orchestration.
- **Why B is incorrect:** Running input guardrails and fetching personalization data *after* the user closes their browser defeats both security enforcement and personalized response generation.
- **Why D is incorrect:** Changing the Lambda timeout ceiling has zero effect on execution speed (only increasing Lambda **memory allocation** increases vCPU share, and even with more vCPUs, sequential synchronous I/O still waits 395ms unless parallelized).
</details>

---

### Question 10
A web application streams long-form technical reports generated by Amazon Bedrock `ConverseStream` to end users in a React browser client. Currently, the browser calls an **Amazon API Gateway HTTP API** backed by a standard synchronous AWS Lambda function (`buffered` invocation mode). Users complain that when they ask for a 1,200-token report, the browser shows a loading spinner for **11 full seconds** and then dumps the entire completed 1,200-token report onto the screen all at once, even though the Lambda function internally uses `bedrock_runtime.converse_stream()`.

Why are users not seeing tokens stream incrementally in their browser, and what is the most direct serverless architecture fix?

- **A.** Standard Lambda `buffered` invocations accumulate the entire function return payload in memory and only return the HTTP response once the handler exits. Reconfigure the Lambda function to use **AWS Lambda Response Streaming** (`InvokeWithResponseStream` via a **Lambda Function URL** or **Amazon CloudFront + Lambda Function URL** or **API Gateway REST API with `ResponseTransferMode: STREAM`**) or stream chunks over **AWS AppSync Events / WebSockets** so each `contentBlockDelta` is flushed to the browser immediately.
- **B.** `bedrock_runtime.converse_stream()` only streams tokens when called from an Amazon EC2 instance and falls back to synchronous batching when called from AWS Lambda.
- **C.** React browsers cannot render text chunks smaller than 1 MB over HTTPS; configure the Lambda function to pad each token with 1 MB of whitespace.
- **D.** Replace `converse_stream()` with `create_model_invocation_job()` so Amazon S3 streams the JSONL output directly to the React browser.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
Calling `converse_stream()` inside a Lambda function only streams tokens from **Amazon Bedrock to the Lambda execution environment**. If the upstream client-to-Lambda transport uses a standard `BUFFERED` Lambda invocation, Lambda buffers everything until the function returns, completely negating the UX benefit of streaming! To achieve **end-to-end streaming** to the browser, every hop in the chain must stream: use **AWS Lambda Response Streaming** (via Lambda Function URLs fronted by CloudFront, or API Gateway REST API response streaming integration) or push incremental token deltas over **AWS AppSync WebSockets / API Gateway WebSocket APIs**.

#### Distractor Analysis
- **Why B is incorrect:** `converse_stream()` works identically inside AWS Lambda, ECS, EKS, and EC2.
- **Why C is incorrect:** Browsers natively read chunked transfer encoding / Server-Sent Events (`text/event-stream`) of a few bytes via the JavaScript `ReadableStream` / `fetch` API; padding 1 MB per token would consume gigabytes of bandwidth per answer.
- **Why D is incorrect:** `create_model_invocation_job()` is asynchronous Batch Inference (taking hours), not real-time UI streaming.
</details>

---

### Question 11
A European banking institution operates its workloads in `eu-central-1` (Frankfurt) and is subject to strict **European Union (GDPR) data residency regulations** stipulating that customer prompts and model completions must **never leave the European Union** under any circumstances, even ephemerally in memory during inference compute. However, the bank's compliance team *does* permit processing within any AWS Region located inside the EU (`eu-west-1` Ireland, `eu-west-3` Paris, `eu-central-1` Frankfurt).

The bank's wealth-advisory assistant on Amazon Bedrock is experiencing `ThrottlingException` errors during morning market openings in `eu-central-1`. How should the architect maximize On-Demand throughput capacity while strictly adhering to the EU data residency mandate?

- **A.** Configure the application to invoke the **`us.anthropic.claude-3-5-sonnet-20241022-v2:0`** Cross-Region Inference profile with an HTTP header `X-Amz-Data-Residency: EU`.
- **B.** Configure the application to invoke a global Foundation Model ARN without a region prefix so Bedrock routes traffic to whichever region worldwide (`us-west-2`, `ap-northeast-1`, `sa-east-1`) has the lowest utilization.
- **C.** Disable encryption in transit (TLS) to reduce regional handshake overhead in `eu-central-1`.
- **D.** Configure the application in `eu-central-1` to invoke the **EU System-Defined Cross-Region Inference (CRIS) Profile** (`eu.anthropic.claude-3-5-sonnet-20241022-v2:0`), which pools On-Demand capacity exclusively across EU AWS Regions while keeping all control-plane logs and data residency strictly inside the EU boundary.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
Amazon Bedrock **System-Defined Cross-Region Inference (CRIS) Profiles** are strictly partitioned by sovereign geographic boundaries:
- `us.*` routes only across US regions.
- `eu.*` routes **exclusively across European Union AWS Regions** (`eu-central-1`, `eu-west-1`, `eu-west-3`, `eu-north-1`, etc.).
- `apac.*` routes only across Asia-Pacific regions.

By invoking the `eu.` CRIS profile from `eu-central-1`, the bank gains multi-region quota elasticity across EU data centers while guaranteeing that no prompt or completion ever leaves the European Union boundary.

#### Distractor Analysis
- **Why A is incorrect:** Invoking a `us.` CRIS profile routes inference compute to US regions (`us-east-1`, `us-west-2`), directly violating the EU data residency mandate; there is no `X-Amz-Data-Residency` override header.
- **Why B is incorrect:** Bedrock does not have a global worldwide routing profile that ignores geography, and routing to US/APAC/SA regions would violate GDPR residency constraints.
- **Why C is incorrect:** AWS API endpoints strictly require TLS 1.2+ and cannot be called over unencrypted HTTP; furthermore, TLS handshakes have nothing to do with TPM/RPM quota throttling.
</details>

---

### Question 12
An autonomous supply-chain agent on Amazon Bedrock executes a complex workflow that requires generating three completely independent analysis sections for a quarterly supplier audit:
1. A 600-token **Financial Solvency Assessment** based on the supplier's balance sheet.
2. A 600-token **Geopolitical & Logistics Risk Assessment** based on shipping lane data.
3. A 600-token **ESG & Carbon Compliance Summary** based on sustainability disclosures.

Currently, a single prompt asks Anthropic Claude 3.5 Sonnet to generate all three sections sequentially in one 1,800-token output response. Because autoregressive decoding is sequential ($O(N_{\text{output}})$), generating 1,800 output tokens at 60 Tokens Per Second takes **30.0 seconds** of decode time.

How can the architect redesign this step to cut wall-clock generation latency from **30.0 seconds down to ~10.5 seconds** while producing the exact same three 600-token sections?

- **A.** Decompose the monolithic prompt into **three parallel `Converse` / `ConverseStream` calls** (Prompt Decomp / Scatter-Gather pattern)—one for Financial Solvency, one for Geopolitical Risk, and one for ESG Compliance—executed concurrently via `asyncio.gather()`, so each branch decodes only 600 output tokens in parallel (~10 seconds) before programmatic assembly.
- **B.** Keep the single monolithic 1,800-token output prompt, but place three `cachePoint` markers inside the system prompt.
- **C.** Increase `maxTokens` from `2048` to `8192` on the single monolithic call so the model generates tokens 3x faster.
- **D.** Run the single monolithic call sequentially three times and average the latency.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
Autoregressive LLM decoding is inherently sequential *within a single generation stream*—Token $t$ cannot be generated until Token $t-1$ has been emitted. Therefore, generating 1,800 tokens in a single call takes $3\times$ longer than generating 600 tokens. When an output consists of **independent sections** that do not depend on one another's generated text, applying the **Parallel Prompt Decomposition (Scatter-Gather)** pattern dispatches 3 concurrent inference requests that each generate 600 tokens simultaneously across separate accelerator threads, reducing wall-clock decode latency by **~66% (from 30s to ~10s)**.

#### Distractor Analysis
- **Why B is incorrect:** `cachePoint` only accelerates **input prefill (TTFT)**; it has zero impact on the speed of generating 1,800 sequential output tokens.
- **Why C is incorrect:** Raising `maxTokens` only raises the upper truncation limit; it does not increase the hardware's Tokens Per Second (TPS) rate.
- **Why D is incorrect:** Running the call three times sequentially takes 90 seconds and triples the cost.
</details>

---

### Question 13 (Multiple Response — Choose TWO)
A platform engineering team manages an **Amazon SageMaker Real-Time Endpoint** hosting a 34B-parameter code-generation LLM on `ml.g5.12xlarge` instances using **SageMaker Inference Components** and Application Auto Scaling. During morning developer login surges at 9:00 AM, the endpoint scales out from 2 instances to 8 instances. However, the team observes that scaling out takes **14 minutes** before the new instances become `InService`, causing high latency and queue timeouts during the 14-minute gap. Analysis shows that 11 of those 14 minutes are spent downloading 68 GB of uncompressed model weights from standard Amazon S3 over the network and initializing the model container on fresh EC2 instances.

Which **TWO** architectural optimizations on Amazon SageMaker will dramatically reduce scale-out latency from 14 minutes to under 60–90 seconds? *(Choose TWO.)*

- **A.** Configure **SageMaker Managed Warm Pools** / pre-provisioned underlying instance capacity with **Inference Component model artifact caching** and/or store uncompressed `safetensors` weights in **Amazon S3 Express One Zone** streamed directly into GPU memory via **SageMaker Fast Model Loader (`sagemaker-fast-model-loader` / LMI streaming weight loader)** without compressing into a monolithic `.tar.gz` archive.
- **B.** Compress the 68 GB model weights using maximum-level `bzip2 -9` into a single archive so the single-threaded CPU decompression on `ml.g5` takes 25 minutes instead of 11 minutes.
- **C.** Configure **Predictive Scaling** or **Scheduled Scaling** in Application Auto Scaling to pre-scale the SageMaker endpoint capacity at 8:45 AM ahead of the predictable 9:00 AM weekday traffic surge, combined with scaling on the `ConcurrentRequestsPerCopy` / `InvocationsPerInstance` metric.
- **D.** Reduce the SageMaker endpoint health check timeout (`ModelDataDownloadTimeoutInSeconds`) from 3,600 seconds to 5 seconds so new instances fail and terminate before downloading the weights.
- **E.** Store the 68 GB model weights inside an AWS Systems Manager Parameter Store `SecureString` parameter.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and C**

#### Explanation
- **Why A is correct:** Packaging 68 GB of LLM weights into a monolithic `.tar.gz` file forces SageMaker to download the archive to disk and run slow single-threaded `tar -xzf` decompression before loading weights into GPU HBM. Using **uncompressed `safetensors` shards** stored in S3 (or **S3 Express One Zone**) with the **SageMaker LMI Fast Model Loader** streams weight tensors in parallel directly from S3 into GPU memory, cutting model load times by 5x–8x.
- **Why C is correct:** Because the 9:00 AM login surge is a predictable weekday pattern, configuring **Scheduled Scaling / Predictive Scaling** in Application Auto Scaling warms the extra instances at 8:45 AM *before* the surge hits.

#### Distractor Analysis
- **Why B is incorrect:** `bzip2 -9` is single-threaded and notoriously slow to decompress, increasing startup time to 25+ minutes.
- **Why D is incorrect:** Setting download timeout to 5 seconds causes every scale-out attempt to fail with a timeout error and roll back.
- **Why E is incorrect:** Systems Manager Parameter Store has a maximum value size limit of **4 KB (standard) or 8 KB (advanced)**—it cannot store a 68 GB LLM!
</details>

---

### Question 14 (Multiple Response — Choose TWO)
An enterprise is deploying a latency-sensitive conversational RAG application in `us-east-1` that uses **Amazon OpenSearch Serverless** (Vector Search collection) as the backing vector store for an **Amazon Bedrock Knowledge Base**. During a major company-wide town hall event, 15,000 employees simultaneously query the assistant within a 2-minute window. While Amazon Bedrock handles the LLM calls cleanly via a Cross-Region Inference profile, the OpenSearch Serverless vector search stage experiences high p99 latency spikes (3.8 seconds) and HTTP 429 rejections for the first 6 minutes while OpenSearch Compute Units (OCUs) reactively scale out.

Which **TWO** architectural actions will prevent vector store latency spikes during high-concurrency events? *(Choose TWO.)*

- **A.** Set the OpenSearch HNSW query parameter `ef_search` to `10,000` during peak traffic so every vector search scans the entire graph.
- **B.** Prior to known high-concurrency events (or to accommodate burst headroom), **increase the minimum/maximum Search OCU capacity limits** on the OpenSearch Serverless collection and configure **active-active standby replicas**, ensuring pre-warmed Search OCUs and HNSW index graphs are already resident in memory before the traffic spike arrives.
- **C.** Delete the HNSW vector index before the town hall begins so OpenSearch falls back to regex wildcard (`*query*`) scans.
- **D.** Route all 15,000 concurrent employee queries through a single-threaded AWS Lambda function with `ReservedConcurrentExecutions: 1`.
- **E.** Introduce an in-memory **Semantic Cache** (Amazon MemoryDB or ElastiCache for Valkey) in front of the RAG pipeline so common town-hall questions are answered in `<15ms` without hitting OpenSearch Serverless or Bedrock at all.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: B and E**

#### Explanation
- **Why B is correct:** OpenSearch Serverless scales Search OCUs automatically based on load, but loading multi-gigabyte HNSW graphs onto newly provisioned OCUs takes several minutes. Pre-scaling minimum/maximum OCUs (with active standby replicas) ahead of a 15,000-user town hall ensures hot memory capacity and read replicas are immediately available.
- **Why E is correct:** During a company town hall, thousands of employees ask variations of the exact same 20–30 questions about announcements made on stage. A **Semantic Cache** in MemoryDB / ElastiCache absorbs 80%–90% of this duplicate query storm in `<15ms`, protecting both OpenSearch Serverless and Bedrock.

#### Distractor Analysis
- **Why A is incorrect:** Raising `ef_search` to `10,000` multiplies CPU and memory contention per query by orders of magnitude, worsening the outage.
- **Why C is incorrect:** Regex wildcard scans cannot perform semantic vector search and are orders of magnitude slower on large corpora.
- **Why D is incorrect:** Setting `ReservedConcurrentExecutions: 1` throttles 14,999 concurrent users immediately at the Lambda layer.
</details>

---

### Question 15 (Multiple Response — Choose TWO)
A security engineer is troubleshooting why switching an existing production application in `us-east-1` from the single-region model ID `anthropic.claude-3-5-sonnet-20241022-v2:0` to the System-Defined Cross-Region Inference (CRIS) profile `us.anthropic.claude-3-5-sonnet-20241022-v2:0` immediately resulted in `AccessDeniedException` errors in staging.

The application's IAM execution role currently has the following policy statement, and the AWS account is governed by an AWS Organizations Service Control Policy (SCP):
```json
{
  "Effect": "Allow",
  "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"],
  "Resource": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0"
}
```

Which **TWO** changes are required so the application can successfully invoke the `us.` Cross-Region Inference profile? *(Choose TWO.)*

- **A.** Update the IAM role policy `Resource` array to allow `bedrock:InvokeModel*` on **both** the System-Defined Inference Profile ARN in the source region (`arn:aws:bedrock:us-east-1:<account-id>:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0`) **and** the underlying Foundation Model ARNs in all destination US regions mapped to that profile (`us-east-1`, `us-east-2`, `us-west-2`).
- **B.** Grant the IAM role `iam:PassRole` and `ec2:RunInstances` permissions in `eu-central-1` and `ap-northeast-1`.
- **C.** Make the S3 bucket storing the application's source code publicly readable via an S3 Bucket Policy (`Principal: "*"`).
- **D.** Ensure the AWS Organizations **Service Control Policy (SCP)** (and any `aws:RequestedRegion` IAM condition keys) permits `bedrock:InvokeModel*` in the destination peer regions (`us-east-1`, `us-east-2`, `us-west-2`) that the `us.` CRIS profile routes to.
- **E.** Attach the `AdministratorAccess` managed policy to the anonymous Cognito Unauthenticated Identity Pool role.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and D**

#### Explanation
- **Why A is correct:** When you invoke a Cross-Region Inference Profile (`us.anthropic...`), Amazon Bedrock evaluates IAM authorization against **two resource types**:
  1. The **Inference Profile ARN** in the calling (source) region (`arn:aws:bedrock:us-east-1:<account-id>:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0`).
  2. The **Foundation Model ARN** in each potential destination region (`arn:aws:bedrock:us-east-1::foundation-model/...`, `arn:aws:bedrock:us-east-2::foundation-model/...`, `arn:aws:bedrock:us-west-2::foundation-model/...`).
- **Why D is correct:** If an AWS Organizations SCP (or a role boundary condition like `aws:RequestedRegion`) blocks all actions outside `us-east-1`, cross-region inference calls routed to `us-west-2` or `us-east-2` will be denied with `AccessDeniedException` unless `bedrock:InvokeModel*` is allowed in those peer US regions.

#### Distractor Analysis
- **Why B is incorrect:** Bedrock CRIS is serverless and does not launch EC2 instances or route `us.` profile traffic to Europe/Asia.
- **Why C is incorrect:** Public S3 buckets are a critical security anti-pattern and have nothing to do with Bedrock CRIS IAM authorization.
- **Why E is incorrect:** Granting `AdministratorAccess` to unauthenticated users is a catastrophic security violation.
</details>
