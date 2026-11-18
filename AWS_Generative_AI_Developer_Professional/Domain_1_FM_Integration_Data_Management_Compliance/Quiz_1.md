# Domain 1 — Quiz 1: FM Selection, Customization, and Multimodal Data Pipelines (Tasks 1.1, 1.2, 1.3)

**Instructions**: This quiz contains 15 Professional-level scenario questions covering Foundation Model selection, customization architectures (Supervised Fine-Tuning, Continued Pre-Training, Model Distillation, Custom Model Import), inference provisioning, and multimodal data processing pipelines.

---

### Question 1
A multinational pharmaceutical company possesses a 450-GB repository of unlabeled proprietary molecular synthesis logs, clinical assay notes, and internal chemical nomenclature written over the past 20 years. In addition, the company maintains a separate repository of 85,000 regulatory standard operating procedures (SOPs) in Amazon S3 that are revised weekly and must be cited verbatim with document version numbers whenever scientists ask compliance questions. When testing base foundation models on Amazon Bedrock with a standard Knowledge Base, the models fail to comprehend the company's proprietary chemical abbreviations and syntax, resulting in poor query formulation and inaccurate reasoning over the retrieved SOPs.

Which architectural approach will resolve the domain comprehension gap while ensuring all compliance answers cite the latest weekly SOP revisions?

- **A.** Format the 450-GB unlabeled molecular repository into JSONL records (`{"input": "<text>"}`) and run a Continued Pre-Training (CPT) job on a supported Amazon Bedrock base model, followed by a Supervised Fine-Tuning (SFT) job on domain Q&A pairs. Deploy the customized model with Provisioned Throughput and configure it as the generation model for an Amazon Bedrock Knowledge Base synchronized with the weekly SOP S3 bucket.
- **B.** Convert both the 450-GB molecular repository and the 85,000 weekly SOPs into prompt-completion pairs and run a weekly Supervised Fine-Tuning (SFT) job on Amazon Bedrock so all domain vocabulary and updated SOP versions are encoded directly into the model parameters.
- **C.** Enable Cross-Region Inference (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`) with Prompt Caching and inject the entire 450-GB proprietary nomenclature corpus into the system prompt before querying the S3 bucket via Amazon S3 Select.
- **D.** Run an Amazon Bedrock Model Distillation job using the 450-GB unlabeled molecular repository as the teacher dataset, and configure the distilled student model to answer SOP questions using zero-shot parametric memory.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
This scenario combines two distinct requirements that must be addressed by complementary mechanisms:
1. **Proprietary Unlabeled Domain Vocabulary (450 GB)**: **Continued Pre-Training (CPT)** uses unlabeled raw text (`{"input": "..."}`) to adapt a foundation model's token embeddings and internal representations to specialized, non-public domain language. Running a subsequent lightweight **Supervised Fine-Tuning (SFT)** pass restores optimal instruction-following behavior, and deploying via **Provisioned Throughput** makes the custom model available for inference.
2. **Weekly Mutating SOPs with Verbatim Citations**: Because SOPs change weekly and require verifiable citations, **Retrieval-Augmented Generation (RAG)** via Amazon Bedrock Knowledge Bases is required. Combining the CPT/SFT custom model with the Bedrock Knowledge Base satisfies both requirements.

#### Distractor Analysis
- **Why B is incorrect**: Supervised Fine-Tuning (SFT) requires labeled prompt-completion pairs rather than 450 GB of unlabeled domain text. Furthermore, encoding weekly-changing SOPs into model weights cannot provide deterministic source citations and incurs prohibitive weekly retraining and Provisioned Throughput replacement costs.
- **Why C is incorrect**: A 450-GB corpus (~100 billion tokens) exceeds any foundation model's context window (200K–1M tokens) by several orders of magnitude and cannot be placed in a system prompt or Prompt Cache.
- **Why D is incorrect**: Model Distillation transfers task-specific behavior from a Teacher FM to a Student FM using prompt-response pairs; it does not perform unsupervised continued pre-training on 450 GB of unlabeled nomenclature, nor can zero-shot parametric memory cite weekly updated SOPs.
</details>

---

### Question 2
An insurance technology firm runs a claims-triage workload in `us-east-1` using `anthropic.claude-3-5-sonnet-20241022-v2:0`. Traffic averages 50 requests per minute (RPM) during normal hours, but spikes to 1,400 RPM for 20 minutes after severe weather events, causing `ThrottlingException` (HTTP 429) errors. The company's compliance policy permits data processing anywhere within the United States, and the CFO wants to eliminate throttling during weather spikes without paying fixed hourly charges for idle capacity during normal hours. When the engineering team replaces the `modelId` with the US Cross-Region Inference Profile (`us.anthropic.claude-3-5-sonnet-20241022-v2:0`), the application immediately fails with an `AccessDeniedException`.

What is the root cause of the `AccessDeniedException` and how should the IAM policy for the application's execution role be configured?

- **A.** Cross-Region Inference requires purchasing at least 1 Model Unit of Provisioned Throughput in `us-east-1` before the `us.*` prefix can be invoked; the IAM role needs `bedrock:CreateProvisionedModelThroughput`.
- **B.** The IAM execution role only grants `bedrock:InvokeModel` on `arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0`. It must be updated to allow `bedrock:InvokeModel` on both the `inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0` ARN in `us-east-1` AND the underlying `foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0` ARNs across all US destination regions (`us-east-1`, `us-east-2`, `us-west-2`).
- **C.** Cross-Region Inference requires enabling AWS Organizations Service Control Policies (SCPs) with `bedrock:AllowCrossRegionRouting`; the application must also switch from the `Converse` API to the `InvokeModelWithResponseStream` API.
- **D.** The IAM role lacks `sts:AssumeRole` permissions in `us-west-2`; the team must deploy a separate IAM role in each US region and configure Amazon Route 53 latency-based routing across regional VPC endpoints.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
When invoking an **Amazon Bedrock Cross-Region Inference (CRIS)** profile (such as `us.anthropic.claude-3-5-sonnet-20241022-v2:0`), Bedrock authorizes the request against **two sets of resources**:
1. The **Inference Profile resource** (`arn:aws:bedrock:<calling-region>:<account-id>:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0`) in the region where the request originates.
2. The **Foundation Model resource** (`arn:aws:bedrock:<dest-region>::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0`) in **every possible destination region** within that geography (`us-east-1`, `us-east-2`, `us-west-2`). If the IAM policy restricts `foundation-model` access strictly to `us-east-1`, Bedrock denies the call with `AccessDeniedException`.

#### Distractor Analysis
- **Why A is incorrect**: Cross-Region Inference is an On-Demand routing mechanism billed per token at standard source-region On-Demand rates; it does not require or use Provisioned Throughput Model Units.
- **Why C is incorrect**: There is no `bedrock:AllowCrossRegionRouting` IAM action, and Cross-Region Inference works seamlessly with both `Converse`/`ConverseStream` and `InvokeModel`/`InvokeModelWithResponseStream`.
- **Why D is incorrect**: Bedrock manages cross-region request routing transparently over the AWS backbone using the caller's single regional endpoint; customers do not assume separate roles per region or configure Route 53 for CRIS.
</details>

---

### Question 3
A quantitative hedge fund has fine-tuned a custom 8-billion-parameter Qwen-2.5 model on its on-premises GPU cluster and exported the model weights, tokenizer configuration, and `config.json` in HuggingFace `safetensors` format to an encrypted Amazon S3 bucket. The engineering team wants to invoke this model using the standard Amazon Bedrock `Converse` API and connect it to Amazon Bedrock Guardrails. The workload is bursty—running intensively for 3 hours at market open and market close, with zero traffic for the remaining 18 hours of the day. The team wants to avoid managing EC2 or SageMaker instance fleets and wants compute charges to scale down automatically during the 18 hours of inactivity.

Which solution meets these requirements with the lowest operational overhead and cost?

- **A.** Create an Amazon SageMaker Real-Time Inference endpoint using an `ml.g5.12xlarge` instance with a HuggingFace Deep Learning Container, and configure an AWS Lambda proxy to translate Bedrock `Converse` API calls into SageMaker `InvokeEndpoint` calls.
- **B.** Upload the `safetensors` weights to an Amazon ECR container image and deploy it on AWS Fargate Spot tasks behind an Application Load Balancer integrated with Amazon Bedrock Agents.

- **C.** Import the `safetensors` weights into Amazon Bedrock using `CreateModelCustomizationJob` with `customizationType="CONTINUED_PRE_TRAINING"`, and purchase a 6-month commitment of 2 Model Units of Provisioned Throughput.
- **D.** Register the S3 `safetensors` artifacts using Amazon Bedrock Custom Model Import (`CreateModelImportJob`), and invoke the imported model ARN directly via the Bedrock `Converse` API with Guardrails attached, leveraging serverless Custom Model Units (CMUs) that scale down to zero when idle.
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon Bedrock Custom Model Import (CMI)** allows you to import supported open-weight architectures (including Llama, Mistral, Mixtral, Flan-T5, and Qwen) trained externally and stored as HuggingFace `safetensors` in Amazon S3. Once imported via `CreateModelImportJob`, the model is served serverlessly using **Custom Model Units (CMUs)**, supports the native Bedrock `Converse`/`InvokeModel` APIs and **Bedrock Guardrails**, and **automatically scales to zero** after a period of inactivity (billing in 5-minute active windows when invoked), making it ideal for bursty workloads with 18 hours of idle time.

#### Distractor Analysis
- **Why A is incorrect**: SageMaker Real-Time Endpoints require managing instance fleets (`ml.g5.12xlarge`), auto-scaling policies, and custom translation proxies to use Bedrock Guardrails (`ApplyGuardrail`) or `Converse`.
- **Why B is incorrect**: AWS Fargate does not support GPU acceleration (`ml.g5`/`p4d`), making an 8B parameter LLM unacceptably slow on CPU-only Fargate tasks, and lacks native Bedrock API/Guardrails hosting.
- **Why C is incorrect**: `CreateModelCustomizationJob` customize existing Bedrock base models rather than importing external weights, and Provisioned Throughput charges a fixed hourly fee 24/7 (including during the 18 idle hours).
</details>

---

### Question 4
A healthcare technology company is building an automated medical record summarization pipeline that processes 250,000 scanned hospital discharge packets each night. Each packet is a 40-page PDF containing multi-column clinical narratives, complex nested laboratory tables, handwritten physician annotations, and embedded echocardiogram trend charts. Furthermore, the downstream clinical audit system requires a validated JSON output matching a strict 35-field schema, along with a field-level confidence score and visual bounding-box coordinates on the source PDF page so human nurses can review any field with a confidence score below 0.85.

Which AWS architecture provides these capabilities with the least custom code and maintenance?

- **A.** Use Amazon S3 Event Notifications to trigger an AWS Lambda function that extracts raw text using the open-source `pypdf` library, splits the text into 500-token chunks, and passes them to Amazon Nova Micro.
- **B.** Configure an Amazon Textract `DetectDocumentText` asynchronous job to extract lines of text, pass the extracted text to Amazon Comprehend Medical `DetectEntitiesV2`, and write a custom Python parser to estimate bounding boxes for the echocardiogram charts.
- **C.** Configure an Amazon Bedrock Data Automation (BDA) project with a custom Blueprint defining the 35-field schema, enable visual grounding and confidence scores in the output configuration, and invoke BDA asynchronously (`InvokeDataAutomationAsync`) on the S3 PDF packets, routing fields with confidence `< 0.85` to human review.
- **D.** Convert each PDF page into a 300-DPI PNG image using AWS Batch, embed the images using Amazon Titan Multimodal Embeddings G1, and query the embeddings in Amazon OpenSearch Serverless to reconstruct the 35 fields.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon Bedrock Data Automation (BDA)** is purpose-built for multimodal extraction from complex documents (including nested tables, charts, and handwritten notes), images, audio, and video. Using **BDA Blueprints**, you define the exact 35-field schema and instructions; BDA automatically outputs structured JSON accompanied by **field-level confidence scores** and **visual bounding-box coordinates (grounding)** on the source PDF page, enabling seamless confidence-threshold routing to human reviewers.

#### Distractor Analysis
- **Why A is incorrect**: `pypdf` only extracts embedded digital text layers; it fails completely on scanned PDFs, handwritten physician annotations, and echocardiogram trend charts, nor does Nova Micro accept image/visual inputs or generate PDF bounding boxes.
- **Why B is incorrect**: Textract `DetectDocumentText` only performs basic word/line OCR (ignoring table structure and visual charts), and Comprehend Medical extracts clinical entities from text without interpreting echocardiogram trend charts or mapping to a custom 35-field schema with chart bounding boxes.
- **Why D is incorrect**: Embedding page images with Titan Multimodal Embeddings G1 produces dense 1024-d vectors for similarity search; embeddings cannot extract 35 structured JSON fields or return bounding-box coordinates.
</details>

---

### Question 5
An e-commerce marketplace processes 40,000 customer support chats per hour. Currently, the team uses `anthropic.claude-3-5-sonnet-20241022-v2:0` to classify each chat into one of 120 granular return-reason taxonomy codes and extract structured order metadata. While Claude 3.5 Sonnet achieves 98.2% classification accuracy, its P99 latency (1.8 seconds) and per-token cost are too high for their real-time chat router. The team has collected 6 months of production prompt-response pairs in Amazon S3 via Amazon Bedrock Model Invocation Logging (`requestMetadata: {"task": "return-taxonomy-v2"}`). They want to train a smaller, faster model (`anthropic.claude-3-5-haiku-20241022-v1:0` or `amazon.nova-micro-v1:0`) to match the Teacher model's 98% accuracy while reducing P99 latency below 350 ms.

Which sequence of steps should the GenAI Developer implement?

- **A.** Configure an Amazon Bedrock Model Distillation job (`customizationType="DISTILLATION"`) specifying the Teacher model ARN, the Student base model ARN, and the S3 `invocationLogsConfig` with a `requestMetadataFilters` rule matching `{"task": "return-taxonomy-v2"}`. Once the job completes, purchase Provisioned Throughput for the resulting custom Student model and route production traffic to the Provisioned Model ARN.
- **B.** Create an Amazon Bedrock Knowledge Base backed by Amazon MemoryDB for Redis, ingest the 6 months of S3 invocation logs as vector embeddings, and configure Claude 3.5 Sonnet to retrieve the top 5 historical logs before classifying each new chat.
- **C.** Launch an Amazon Bedrock Model Evaluation job (`CreateEvaluationJob`) with `amazon.nova-micro-v1:0` as the judge model and `anthropic.claude-3-5-sonnet-20241022-v2:0` as the generator model, which automatically updates the weights of Nova Micro in the On-Demand pool.
- **D.** Export the proprietary model weights of `anthropic.claude-3-5-sonnet-20241022-v2:0` to an Amazon S3 bucket, run 4-bit AWQ quantization using an AWS Glue Python Shell job, and re-import the quantized weights using Bedrock Custom Model Import.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
**Amazon Bedrock Model Distillation** transfers specialized task knowledge from a large Teacher FM (`Claude 3.5 Sonnet` or `Nova Pro/Premier`) to a compact, low-latency Student FM (`Claude 3.5 Haiku` or `Nova Micro/Lite`). Bedrock Distillation natively supports reading directly from **Amazon Bedrock Model Invocation Logs in S3** and filtering by `requestMetadataFilters` (`equals: {"task": "return-taxonomy-v2"}`) so only relevant production examples are used. After distillation finishes, the custom Student model must be deployed on **Provisioned Throughput** to serve real-time traffic.

#### Distractor Analysis
- **Why B is incorrect**: Continuing to use Claude 3.5 Sonnet while adding a vector retrieval step and 5 extra context examples will *increase* both P99 latency and input token cost rather than reducing them.
- **Why C is incorrect**: Bedrock Model Evaluation (`CreateEvaluationJob`) only scores and benchmarks model outputs; it never modifies model weights or updates shared On-Demand models.
- **Why D is incorrect**: Proprietary foundation model weights (such as Anthropic Claude 3.5 Sonnet) can never be exported from Amazon Bedrock to S3, nor can AWS Glue Python Shell jobs perform GPU quantization.
</details>

---

### Question 6
A media analytics company needs to generate marketing tags and sentiment summaries for 4 million archived product reviews stored as JSONL files in Amazon S3. This is a back-office enrichment task that must complete within 18 hours, has no interactive user waiting on responses, and must not consume any of the company's real-time On-Demand RPM/TPM quotas in `us-east-1` because their live customer-facing storefront assistant runs in the same AWS account and region. Furthermore, management requires the lowest possible token cost.

Which Amazon Bedrock inference mechanism should the developer select?

- **A.** Invoke `ConverseStream` across 200 concurrent AWS Lambda functions using the standard On-Demand model ID with exponential backoff.
- **B.** Purchase a 6-month commitment of 10 Model Units of Amazon Bedrock Provisioned Throughput, run the 4 million records in 6 hours, and immediately terminate the Provisioned Throughput to avoid further charges.
- **C.** Submit an Amazon Bedrock Batch Inference job (`CreateModelInvocationJob`) pointing to the S3 JSONL input bucket and an S3 output bucket.
- **D.** Configure an Amazon Bedrock Prompt Flow with an `Iterator` node and invoke it synchronously via Amazon API Gateway.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon Bedrock Batch Inference** (`CreateModelInvocationJob`) is purpose-built for asynchronous bulk processing of large datasets stored in S3 JSONL format. It provides two critical architectural benefits that directly satisfy the prompt:
1. It executes on separate asynchronous capacity pools that **do not consume your real-time On-Demand RPM/TPM quotas**, protecting the live customer-facing storefront assistant.
2. It provides a **50% cost discount** on input and output tokens compared to On-Demand pricing.

#### Distractor Analysis
- **Why A is incorrect**: Invoking On-Demand `ConverseStream` from 200 concurrent Lambda functions directly competes for the account's real-time RPM/TPM quotas (causing throttling on the live storefront) and pays full On-Demand token prices.
- **Why B is incorrect**: A 6-month commitment of Provisioned Throughput cannot be terminated after 6 hours without paying for the full 6-month commitment term, which would cost tens of thousands of dollars unnecessarily.
- **Why D is incorrect**: Bedrock Prompt Flows `Iterator` nodes execute against real-time quotas, pay standard On-Demand token rates, and will time out behind API Gateway's 29-second integration limit on 4 million records.
</details>

---

### Question 7
A data engineering team is preparing a 12-terabyte corpus of scraped web articles, internal wiki pages, and historical support tickets stored in Amazon S3 for a Continued Pre-Training (CPT) job. During an initial pilot training run, the resulting model exhibited severe memorization—repeating identical boilerplate legal footers verbatim—and inadvertently emitted customer Social Security Numbers (SSNs) and credit card numbers that were buried in raw support tickets.

Which preprocessing pipeline should the team implement before running the full Continued Pre-Training job? (Choose TWO.)

- **A.** Run an Apache Spark job on Amazon EMR Serverless (or AWS Glue) using Spark MLlib `MinHashLSH` (Locality-Sensitive Hashing) over $n$-gram token shingles to identify and remove near-duplicate documents and repetitive boilerplate across the 12-TB corpus.
- **B.** Increase the foundation model's inference `temperature` to `1.0` and `top_k` to `500` during production invocation so the model randomly skips memorized SSNs and boilerplate phrases.
- **C.** Integrate an AWS Glue PII Detection and Redaction transform (or batch Amazon Comprehend `StartPiiEntitiesDetectionJob` with `Mode="ONLY_REDACTION"`) into the data preparation pipeline to mask SSNs, credit card numbers, and personal identifiers prior to generating the training JSONL files.
- **D.** Convert the 12-TB text corpus into 256-dimensional vectors using Amazon Titan Text Embeddings V2 and pass the floating-point vectors directly into `CreateModelCustomizationJob`.
- **E.** Set the Continued Pre-Training hyperparameter `epochCount` to `50` and `learningRate` to `0.1` so gradient descent overwrites any memorized PII sequences.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answers: A, C**

#### Explanation
- **Why A is correct**: Near-duplicate documents and templated boilerplate footers cause foundation models to overfit and memorize repetitive sequences during Continued Pre-Training. **MinHash LSH (Locality-Sensitive Hashing)** in Apache Spark (EMR Serverless / AWS Glue) efficiently computes Jaccard similarity over document shingles at multi-terabyte scale to deduplicate fuzzy matches prior to training.
- **Why C is correct**: Any PII present in the CPT training data can be encoded into the model's parametric weights and leaked at inference time. Scrubbing and redacting PII via **AWS Glue PII transforms** or **Amazon Comprehend batch PII redaction** before training prevents PII from ever entering the model weights.

#### Distractor Analysis
- **Why B is incorrect**: Raising `temperature` and `top_k` actually *increases* the likelihood of sampling unpredictable long-tail tokens (including memorized PII) and degrades factual accuracy without removing PII from the weights.
- **Why D is incorrect**: Continued Pre-Training requires tokenized text in JSONL format (`{"input": "..."}`), not dense embedding vectors.
- **Why E is incorrect**: Setting `epochCount = 50` and a massive `learningRate = 0.1` causes catastrophic forgetting, gradient explosion, and *worse* overfitting/memorization of training examples.
</details>

---

### Question 8
A financial compliance application uses an foundation model on Amazon Bedrock to convert natural language regulatory changes into deterministic SQL migration scripts. During testing, developers notice that running the exact same prompt three times produces slightly different column aliases and SQL formatting. When inspects the codebase, the lead architect finds the following `inferenceConfig` and `additionalModelRequestFields`:
```json
{
  "temperature": 0.85,
  "topP": 0.95,
  "topK": 250,
  "maxTokens": 2048
}
```
How should the developer modify the inference configuration to maximize output determinism and consistency across runs?

- **A.** Set `temperature` to `1.0`, `topP` to `0.01`, and `maxTokens` to `100`.
- **B.** Keep `temperature` at `0.85`, lower `topP` to `0.50`, and add `"stopSequences": ["SELECT"]`.
- **C.** Enable Extended Thinking with `budget_tokens: 16000` and set `temperature` to `0.99`.

- **D.** Set `temperature` to `0.0` (leaving `topP` at its default `1.0`), so the decoder performs greedy argmax selection of the highest-probability token at each step.
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
To maximize determinism for structured code/SQL generation, set **`temperature = 0.0`** (and avoid simultaneously modifying `top_p`, leaving it at `1.0`). As $T \to 0$ in the scaled softmax distribution $P(w_i) = \frac{\exp(z_i/T)}{\sum_j \exp(z_j/T)}$, the probability mass concentrates entirely on the single highest-logit token (greedy decoding), minimizing run-to-run variance.

#### Distractor Analysis
- **Why A is incorrect**: Tuning both `temperature = 1.0` and `topP = 0.01` violates AWS and model-provider best practices (modify `temperature` OR `top_p`, not both), and `maxTokens = 100` will truncate SQL migration scripts prematurely.
- **Why B is incorrect**: Keeping `temperature = 0.85` maintains high randomness, and adding `"SELECT"` to `stopSequences` will immediately halt generation the moment the SQL script attempts to write a `SELECT` statement!
- **Why C is incorrect**: Extended Thinking does not allow setting `temperature = 0.99` (temperature cannot be modified when Extended Thinking is enabled), nor is high temperature appropriate for deterministic SQL output.
</details>

---

### Question 9
A global industrial equipment manufacturer wants to build a visual parts-identification and troubleshooting assistant. Field technicians working on wind turbines will upload a 15-second video clip (`.mp4`) showing a vibrating gearbox along with a photo of the corroded serial plate and ask: *"Identify the gearbox model from the plate, analyze the vibration pattern in the video, and list the torque specifications for replacing the main bearing."* The engineering team wants to handle the video, image, and text prompt in a single unified Amazon Bedrock `Converse` API call without deploying separate video frame-extraction microservices.

Which Amazon Bedrock foundation model should the architect select as the primary model for this `Converse` call?

- **A.** `amazon.nova-micro-v1:0`
- **B.** `amazon.nova-pro-v1:0`

- **C.** `cohere.command-r-plus-v1:0`
- **D.** `amazon.titan-embed-image-v1`
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
**Amazon Nova Pro (`amazon.nova-pro-v1:0`)** (as well as Amazon Nova Lite and Nova Premier) is a native multimodal foundation model on Amazon Bedrock that accepts **video (`.mp4`, `.mov`, etc.), images, and text** directly inside the `Converse` API (either as inline bytes or direct S3 URI references) and performs joint multimodal reasoning across all three inputs in a single API call.

#### Distractor Analysis
- **Why A is incorrect**: **Amazon Nova Micro (`amazon.nova-micro-v1:0`) is a text-only model**; it does not accept image or video inputs and will return a `ValidationException` if passed media blocks.
- **Why C is incorrect**: `cohere.command-r-plus-v1:0` is a text-only model tailored for text RAG and tool use; it does not support native video or image inputs.
- **Why D is incorrect**: `amazon.titan-embed-image-v1` (Titan Multimodal Embeddings G1) only outputs fixed-dimension numerical vectors for text and single images; it cannot process video or generate natural language answers.
</details>

---

### Question 10
A legal-tech startup is building an Amazon Bedrock Knowledge Base over 250,000 commercial real estate lease agreements. Each lease is a 60-page PDF containing dense cross-referenced indemnification clauses. During retrieval testing with Fixed-Size Chunking (300 tokens), the team observes a severe trade-off:
- When chunks are small (150–300 tokens), vector cosine similarity accurately locates the exact sentence mentioning *"Landlord's Environmental Liability Cap"*, but the generator FM hallucinates or misses exceptions because the defined terms and carve-outs appear three paragraphs earlier or later in the same section.
- When chunks are large (1,500 tokens), the vector embedding represents an average of too many topics, causing retrieval recall to drop by 35%.

Which native Amazon Bedrock Knowledge Base chunking strategy directly resolves this retrieval-precision vs. generation-context dilemma?

- **A.** Decreasing the vector dimension of Amazon Titan Text Embeddings V2 from `1024` to `256`.

- **B.** No Chunking (`chunkingStrategy="NONE"`), relying on the embedding model to index each 60-page lease as a single vector.
- **C.** Fixed-Size Chunking with `maxTokens = 300` and `overlapPercentage = 99`.
- **D.** Hierarchical Chunking configured with a 1,500-token Parent chunk level and a 200-token Child chunk level.
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Hierarchical Chunking** in Amazon Bedrock Knowledge Bases splits documents into two linked tiers: **Child chunks** (e.g., 200 tokens) and **Parent chunks** (e.g., 1,500 tokens). During indexing, Bedrock embeds the small Child chunks so dense vector search achieves pinpoint semantic precision on specific sentences. At query time, when a Child chunk matches, Bedrock automatically swaps in its broader **Parent chunk** (deduplicating if multiple children belong to the same parent) so the generator FM receives the full surrounding legal context!

#### Distractor Analysis
- **Why A is incorrect**: Reducing embedding dimensions from 1024 to 256 reduces vector storage memory, but does nothing to resolve the chunk-size trade-off between retrieval specificity and surrounding paragraph context.
- **Why B is incorrect**: A 60-page lease (~30,000+ tokens) exceeds the 8,192-token input limit of Amazon Titan Text Embeddings V2 (failing ingestion) and would suffer extreme semantic dilution if compressed into one vector.
- **Why C is incorrect**: Amazon Bedrock Knowledge Bases does not allow `overlapPercentage = 99`, and excessive overlap creates massive index bloat without expanding the context window of any single retrieved chunk beyond 300 tokens.
</details>

---

### Question 11
A biotechnology research firm is building a RAG system over 15,000 multi-topic scientific symposium transcripts. Each transcript is a continuous stream of text without section headers where speakers transition unpredictably between oncology trial statistics, gene-editing CRISPR protocols, and biomanufacturing supply chains. Some topics last 80 tokens, while others span 900 tokens. The architect wants the Bedrock Knowledge Base to dynamically place chunk boundaries wherever the semantic subject shifts, rather than cutting at arbitrary token counts.

Which chunking configuration should the architect configure on the Bedrock Knowledge Base data source?

- **A.** `DEFAULT` chunking strategy, which automatically runs topic-modeling LDA across all documents.
- **B.** `HIERARCHICAL` chunking strategy with three levels of fixed token counts.
- **C.** `SEMANTIC` chunking strategy configured with `maxTokens`, `bufferSize`, and `breakpointPercentileThreshold` (e.g., `95`).
- **D.** `NONE` chunking strategy combined with an Amazon S3 Lifecycle policy.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Semantic Chunking (`chunkingStrategy="SEMANTIC"`)** in Amazon Bedrock Knowledge Bases calculates embeddings across sliding sentence windows (`bufferSize`) and measures the cosine distance between adjacent sentence groups. Whenever the semantic dissimilarity exceeds the configured `breakpointPercentileThreshold` (e.g., 95th percentile of distances within the document), Bedrock inserts a chunk boundary—creating variable-length, semantically cohesive chunks up to `maxTokens`.

#### Distractor Analysis
- **Why A is incorrect**: `DEFAULT` chunking simply splits text into ~300-token chunks using basic character/paragraph separators; it does not perform semantic similarity breakpoint analysis.
- **Why B is incorrect**: `HIERARCHICAL` chunking supports exactly two levels (Parent and Child) using fixed token thresholds, not dynamic topic-shift detection.
- **Why D is incorrect**: `NONE` treats the entire file as one chunk, which fails on long multi-topic transcripts and mixes unrelated topics into a single vector.
</details>

---

### Question 12
A developer is running a Supervised Fine-Tuning (SFT) job on Amazon Bedrock to customize a model for extracting structured cybersecurity threat indicators from firewall logs. The developer has only 180 labeled training examples and 40 validation examples. The developer configures `epochCount = 25`, `batchSize = 1`, and a high `learningRate = 0.005`. When inspecting the training metrics file (`training_metrics.jsonl`) in the output S3 bucket after the job completes, the developer observes that `training_loss` decreased steadily from `2.41` at Epoch 1 down to `0.02` at Epoch 25, while `validation_loss` decreased from `2.45` to `0.68` at Epoch 3 and then climbed continuously to `3.95` by Epoch 25.

What does this metric divergence indicate, and what adjustments should the developer make?

- **A.** The model is underfitting the training distribution; the developer should increase `epochCount` to `50` and remove the validation dataset.
- **B.** The model is severely overfitting the small 180-example dataset after Epoch 3; the developer should reduce `epochCount` to `3–4`, lower the `learningRate` (e.g., `0.00005`), and increase the size and diversity of the labeled training dataset.
- **C.** The S3 bucket KMS key is throttling read requests after Epoch 3; the developer should switch from SSE-KMS to SSE-S3 encryption.
- **D.** The base model has reached its maximum context window limit; the developer should switch from Supervised Fine-Tuning to Continued Pre-Training.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
When `training_loss` drops toward zero (`0.02`) while `validation_loss` reaches a minimum early (Epoch 3 at `0.68`) and then rises sharply (`3.95`), the model has **overfitted** (memorized the noise and exact phrasing of the 180 training examples while losing generalization on unseen validation examples). Reducing `epochCount` to match the validation-loss elbow (3–4 epochs), lowering the `learningRate`, and curating more diverse training samples resolves overfitting.

#### Distractor Analysis
- **Why A is incorrect**: Underfitting is characterized by *both* training loss and validation loss remaining high; increasing epochs to 50 would worsen the overfitting.
- **Why C is incorrect**: S3 or KMS throttling causes API errors or job failures, not mathematical divergence between validation loss and training loss curves.
- **Why D is incorrect**: Context window limits truncate individual long examples; they do not cause validation loss to climb across epochs, nor is CPT appropriate for labeled extraction tasks.
</details>

---

### Question 13
A defense contractor operates an isolated workload in `eu-central-1` (Frankfurt). Under a strict national defense contract, all raw prompts, retrieved documents, and model completions must remain strictly within the physical boundaries of the `eu-central-1` AWS Region at all times—even transiently in memory during inference execution. Additionally, the workload requires guaranteed capacity for 2,000 input tokens per second with a predictable monthly budget and must use a custom fine-tuned model trained on internal engineering specifications.

Which two configuration choices MUST the architect combine to satisfy both the compliance mandate and the custom model hosting requirement? (Choose TWO.)

- **A.** Configure the application to invoke the `eu.` Cross-Region Inference Profile (`eu.anthropic.claude-3-5-sonnet-20241022-v2:0`) so requests are load-balanced across EU regions.
- **B.** Purchase Amazon Bedrock Provisioned Throughput (Model Units) directly in `eu-central-1` associated with the custom fine-tuned model ARN, and invoke the Provisioned Model ARN via an interface VPC endpoint (`com.amazonaws.eu-central-1.bedrock-runtime`).
- **C.** Disable Cross-Region Inference profiles and ensure IAM policies and AWS Organizations Service Control Policies (SCPs) restrict `bedrock:*` actions strictly to `aws:RequestedRegion == "eu-central-1"`.
- **D.** Enable Amazon Bedrock Batch Inference with Cross-Account S3 Replication to `eu-west-1` (Ireland).
- **E.** Store the custom model weights in a public Amazon S3 bucket in `eu-central-1` and invoke them using Amazon CloudFront edge functions.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answers: B, C**

#### Explanation
- **Why B is correct**: Custom fine-tuned models on Amazon Bedrock **require Provisioned Throughput (Model Units)** for real-time inference. Provisioning Model Units in `eu-central-1` and invoking the Provisioned Model ARN through a regional `bedrock-runtime` VPC Interface Endpoint guarantees dedicated throughput within `eu-central-1`.
- **Why C is correct**: Because the contract mandates that data must *never* leave `eu-central-1` (even transiently during inference), the architect must **not** use `eu.*` Cross-Region Inference profiles (which route across `eu-west-1`, `eu-west-3`, `eu-central-1`, `eu-north-1`) and should enforce single-region execution in `eu-central-1` via IAM/SCP conditions.

#### Distractor Analysis
- **Why A is incorrect**: The `eu.*` Cross-Region Inference Profile routes inference requests to peer EU AWS Regions outside Germany (such as Ireland `eu-west-1`, Paris `eu-west-3`, or Stockholm `eu-north-1`), violating the strict `eu-central-1`-only contract mandate. Furthermore, CRIS does not support custom fine-tuned models.
- **Why D is incorrect**: Replicating data to `eu-west-1` violates the single-region residency requirement, and Batch Inference does not provide real-time synchronous throughput.
- **Why E is incorrect**: Bedrock custom model weights are managed securely by Bedrock (never exposed in public S3 buckets) and cannot be executed inside CloudFront edge functions.
</details>

---

### Question 14
An enterprise software company wants its Amazon Bedrock Knowledge Base to ingest a repository of 40,000 legacy source code files (`.py`, `.java`, `.ts`) and architectural Markdown specifications stored in Amazon S3. Standard Fixed-Size and Hierarchical chunkers split Python classes and Java methods mid-function, breaking syntactic validity and stripping class-level docstrings from their member methods. The team has written a custom Python AST (Abstract Syntax Tree) splitter that parses code into complete method definitions and prepends the parent class signature and docstring to every method chunk.

How can the team integrate this AST splitter into the managed Amazon Bedrock Knowledge Base ingestion workflow without abandoning Bedrock Knowledge Bases?

- **A.** Package the Python AST splitter in an AWS Lambda function and configure a `customTransformationConfiguration` on the Bedrock Knowledge Base data source (`stepToApply="POST_CHUNKING"` with an intermediate S3 bucket location) while setting the Knowledge Base chunking strategy to `NONE` (or applying custom chunking inside the Lambda).
- **B.** Upload the Python AST script to Amazon Bedrock Prompt Management and attach it as a system prompt to Amazon Titan Text Embeddings V2.
- **C.** Enable `SEMANTIC` chunking with `breakpointPercentileThreshold = 100` so the embedding model automatically compiles the Java and Python AST before splitting.
- **D.** Configure an Amazon OpenSearch Serverless ingest pipeline with a `gsub` regex processor to reassemble split functions after vector embedding has already occurred.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
Amazon Bedrock Knowledge Bases natively supports **Custom Transformation AWS Lambda functions** (`customTransformationConfiguration`) during the ingestion pipeline. By specifying an intermediate S3 storage location and pointing `transformationLambdaConfiguration` to your Lambda ARN, your function can read parsed documents, execute custom **AST-based code splitting** and docstring header injection, and return multiple custom chunks and chunk-level metadata dictionaries back to the Knowledge Base pipeline prior to vector embedding!

#### Distractor Analysis
- **Why B is incorrect**: Prompt Management templates cannot be attached to embedding models (`Titan Text Embeddings V2`) to execute Python AST parsing code.
- **Why C is incorrect**: `SEMANTIC` chunking splits text based on sentence embedding cosine distance, not programming language Abstract Syntax Tree grammar rules.
- **Why D is incorrect**: Modifying text in OpenSearch *after* the vector embedding has already been generated by Bedrock leaves the stored vector mismatched with the modified text, and regex cannot reconstruct broken AST contexts across chunks.
</details>

---

### Question 15
A global retail company is building a cross-modal visual discovery engine. Shoppers can either upload a photograph of a piece of furniture from a magazine to find visually similar items in the company's 2-million-SKU product catalog, OR type a natural language description (e.g., *"mid-century walnut coffee table with tapered brass legs"*) to search across the exact same vector index of product images. To minimize vector database RAM costs, the architect wants both image queries and text queries to map into a single unified vector space without running an intermediate image-captioning LLM at query time.

Which embedding model and vector indexing strategy should the architect select?

- **A.** Use `amazon.titan-embed-text-v2:0` to embed the text queries into 1024-dimensional vectors, and use Amazon Rekognition `DetectLabels` to convert product images into comma-separated keyword strings.
- **B.** Train a custom Word2Vec model on Amazon SageMaker using product SKU numbers and store the outputs in Amazon DynamoDB Scan tables.

- **C.** Use `cohere.embed-english-v3` with `input_type="image"` for the catalog photos and `amazon.nova-canvas-v1:0` to embed the text queries.
- **D.** Use `amazon.titan-embed-image-v1` (Amazon Titan Multimodal Embeddings G1) to embed all catalog product images (and optional titles) into a shared 1024-dimensional (or 384-dimensional) vector index, and use the same model at query time to embed either the shopper's uploaded photo or natural language text query into the same vector space.
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon Titan Multimodal Embeddings G1 (`amazon.titan-embed-image-v1`)** projects both images and text into a **single aligned multimodal vector space** (configurable at 1024 or 384 dimensions). This enables direct **image-to-image**, **text-to-image**, and **image+text-to-image** k-NN vector search against a single unified vector index without requiring an intermediate image-to-text captioning LLM at query time.

#### Distractor Analysis
- **Why A is incorrect**: `amazon.titan-embed-text-v2:0` is strictly a text embedding model and cannot process uploaded shopper photographs; Rekognition `DetectLabels` outputs coarse object labels (e.g., `"Table"`, `"Wood"`) that lose fine-grained visual style details like *"mid-century tapered brass legs"*.
- **Why B is incorrect**: Word2Vec cannot process images, and DynamoDB Scan operations do not support sub-linear HNSW dense vector similarity search across 2 million SKUs.
- **Why C is incorrect**: `amazon.nova-canvas-v1:0` is a text-to-image *generation* model, not an embedding model.
</details>
