# AWS Certified Generative AI Developer – Professional (AIP-C01) Practice Exam 2

**Duration:** 60 Minutes (25 Full-Length Multi-Paragraph Scenario Questions)  
**Passing Score Target:** $\ge 20 / 25$ (80%)  
**Exam Blueprint Alignment:** Domain 1 (31% — 8 questions), Domain 2 (26% — 7 questions), Domain 3 (20% — 5 questions), Domain 4 (12% — 3 questions), Domain 5 (11% — 2 questions)

---

### Question 1 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A luxury fashion e-commerce platform is building a multimodal visual discovery assistant. Customers can either upload a smartphone photo of an outfit or type a rich stylistic description (*"vintage 1970s herringbone wool trench coat with tortoiseshell buttons"*) to retrieve matching catalog items from a repository of 3 million high-resolution product images and descriptions. The engineering team wants a single vector index where both product images and product text descriptions are mapped into a **shared unified vector space**, allowing cross-modal retrieval (image-to-text, text-to-image, and combined image+text-to-image) without running separate OCR pipelines. Furthermore, to fit 3 million vectors into their existing in-memory vector cache with 50% less memory overhead, they want to configure the output embedding dimension to `384` (instead of `1024`) with L2-normalized vectors.

**Which Amazon Bedrock embedding model and configuration natively supports this cross-modal, configurable-dimension architecture?**
- A. Use `amazon.titan-embed-text-v2:0` and pass raw JPEG binary bytes directly into the `inputText` string field.
- B. Use **`amazon.titan-embed-image-v1`** (Amazon Titan Multimodal Embeddings G1), passing either `inputImage` (Base64), `inputText`, or both in the request payload, and setting `embeddingConfig={"outputEmbeddingLength": 384}` to generate normalized embeddings in a unified multimodal vector space.
- C. Use `anthropic.claude-3-5-haiku-20241022-v1:0` via the `Converse` API to output 384 random floating-point numbers as text.
- D. Use Amazon Rekognition `DetectLabels` and index the label strings in Amazon Route 53.

**Answer:** B  
**Explanation:** **Amazon Titan Multimodal Embeddings G1 (`amazon.titan-embed-image-v1`)** embeds both images (`inputImage` up to 25 MB) and text (`inputText` up to 128 tokens)—either individually or jointly—into the **same shared semantic vector space**, enabling direct text-to-image, image-to-image, and multimodal similarity search. It also natively supports configurable output embedding dimensions (`outputEmbeddingLength` of `1024`, `384`, or `256`) so architects can trade a fraction of accuracy for dramatic reductions in vector index memory footprint and search latency.  
**Distractor Analysis:**
- *A is incorrect:* `amazon.titan-embed-text-v2:0` is a text-only embedding model (`1024`, `512`, or `256` dimensions) and cannot encode images into a visual vector space.
- *C is incorrect:* Decoder-only generative LLMs do not generate valid metric-space vector embeddings through text completion prompts.
- *D is incorrect:* Route 53 is a DNS service, and Rekognition discrete object labels lose fine-grained visual style similarity compared to dense multimodal embeddings.

---

### Question 2 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A medical device manufacturer maintains a technical RAG system over 90,000 dense biomedical engineering specifications and FDA 510(k) regulatory filings. However, external patients and hospital administrators query the system using short, non-technical consumer questions—such as *"Why is my insulin pump beeping three times after I change the cartridge?"* Because the user's short conversational question shares almost zero vocabulary or sentence structure with the formal technical manual prose (*"Acoustic alarm pattern A-03 indicates occlusion pressure transducer calibration offset following reservoir seating"*), standard dense vector similarity between the raw user query embedding and the manual chunk embeddings suffers from a severe **query-to-document asymmetry gap**, yielding low recall.

**Which advanced retrieval augmentation pattern bridges the structural gap between short non-technical user queries and formal technical documents before querying the vector store?**
- A. Encrypt the user's query with AWS KMS `Encrypt` and pass the ciphertext blob directly to the embedding model.

- B. Truncate the user's query to only its first two words before generating the query embedding.
- C. Set `numberOfResults=1` in the vector search configuration and disable BM25 hybrid search.
- D. Implement **Hypothetical Document Embeddings (HyDE)** (or Query Expansion/Reformulation): first invoke a fast foundation model (such as `amazon.nova-lite-v1:0` or `claude-3-5-haiku`) to generate a hypothetical formal technical manual excerpt or expanded set of technical search queries that would answer the user's question, embed that hypothetical technical passage using `amazon.titan-embed-text-v2:0`, and use the resulting vector to query the Amazon Bedrock Knowledge Base.
**Answer:** D  
**Explanation:** **Hypothetical Document Embeddings (HyDE)** and **Query Reformulation/Expansion** directly address **document-query asymmetry** (where user queries are short and colloquial, while indexed documents are long and technical). By having a lightweight LLM first synthesize a hypothetical technical answer document (including plausible technical terminology like *"acoustic alarm"*, *"occlusion transducer"*, and *"reservoir seating"*), the embedding model maps the hypothetical document into the exact same dense representation neighborhood as the real technical manual chunks—dramatically improving vector retrieval recall.  
**Distractor Analysis:**
- *A is incorrect:* Passing binary KMS ciphertext to an embedding model embeds random base64 noise and returns garbage vectors.
- *B is incorrect:* Truncating the query to two words (`"Why is"`) destroys all semantic meaning.
- *C is incorrect:* Lowering `numberOfResults` to `1` reduces recall even further when top-1 similarity is weak.

---

### Question 3 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A media production studio archives 250,000 hours of raw broadcast video footage (`.mp4` and `.mov` files) in Amazon S3. Video editors want to search the archive using natural-language scene queries (such as *"drone aerial shot of a container ship passing under the Golden Gate Bridge at sunset"*) and also automatically generate chapter summaries, spoken dialogue transcripts with speaker timestamps, and on-screen text (OCR) extractions for ingestion into an Amazon Bedrock Knowledge Base without building and maintaining a custom fleet of FFmpeg frame-extraction, Whisper transcription, and computer-vision EC2 servers.

**Which TWO AWS capabilities can the architect combine to build this managed multimodal video ingestion and RAG pipeline? (Choose TWO.)**
- A. Configure **Amazon Bedrock Data Automation (BDA)** (`InvokeDataAutomationAsync`) with a video blueprint to automatically extract timestamped scene descriptions, audio transcripts, chapter summaries, and on-screen text metadata from the S3 video files into structured JSON/Markdown for Knowledge Base indexing.
- B. Use `amazon.titan-embed-text-v1` to embed the raw `.mp4` video binary stream without transcription or frame parsing.
- C. Upload the 50-GB `.mp4` video files directly into AWS Systems Manager Parameter Store standard parameters.
- D. Use **Amazon Nova Pro / Nova Premier** (or TwelveLabs Marengo/Pegasus models on Amazon Bedrock) with direct S3 URI video input (`video.source.s3Location`) to perform native temporal video understanding and multimodal embedding generation.
- E. Configure Amazon Route 53 Health Checks to transcribe the audio track of each video.

**Answer:** A, D  
**Explanation:**
- **Why A is correct**: **Amazon Bedrock Data Automation (BDA)** natively processes multimodal video, audio, image, and document assets stored in S3, automatically generating scene-by-scene visual descriptions, audio transcriptions, IAB content classifications, and chapter timestamps ready for RAG ingestion.
- **Why D is correct**: **Amazon Nova (Lite / Pro / Premier)** and specialized video foundation models on Bedrock accept direct `s3Location` video inputs (`s3://bucket/video.mp4`), enabling native temporal video Q&A, scene summarization, and multimodal indexing without manual frame-splitting pipelines.  
**Distractor Analysis:**
- *C is incorrect:* SSM Parameter Store has a 4 KB (standard) or 8 KB (advanced) limit and is for configuration strings, not 50-GB video files.
- *B is incorrect:* `titan-embed-text-v1` only accepts UTF-8 text strings up to 8,192 tokens and cannot parse binary `.mp4` video streams.
- *E is incorrect:* Route 53 Health Checks monitor endpoint reachability, not audio transcription.

---

### Question 4 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A software engineering organization wants to index 450 internal Git repositories (containing Python, TypeScript, Go, and Rust source code) into an Amazon Bedrock Knowledge Base so developers can ask architectural questions about internal microservices. When the team ingested the `.py` and `.ts` files using standard Fixed-Size Chunking at `400` tokens, the chunk boundaries frequently sliced functions, class definitions, and docstrings right in the middle of code blocks, stripping method signatures from their implementations and causing the generator model to hallucinate broken imports and invalid method arguments.

**How should the architect configure the Amazon Bedrock Knowledge Base ingestion pipeline to preserve syntactic Abstract Syntax Tree (AST) class and function boundaries while also injecting each file's Git commit SHA and repository owner into chunk metadata?**
- A. Convert all Python and TypeScript source files into `.bmp` bitmap images before uploading to S3.
- B. Set `chunkingStrategy="FIXED_SIZE"` with `maxTokens=20` and `overlapPercentage=1`.
- C. Configure a **Custom Transformation Lambda** (`customTransformationConfiguration`) in the Knowledge Base data source's `vectorIngestionConfiguration`: specify an intermediate S3 bucket and an AWS Lambda function that parses each source file using language-aware AST/tree-sitter boundaries, writes syntactically complete function/class chunks alongside enriched `.metadata.json` attributes (`git_sha`, `repo_owner`) back to the S3 staging bucket, and lets Bedrock embed the resulting chunks.
- D. Disable vector embeddings and use AWS CodeDeploy to copy the source code onto the developer's local browser cache.

**Answer:** C  
**Explanation:** Amazon Bedrock Knowledge Bases support **Custom Transformation Lambda functions** (`customTransformationConfiguration`) inside the ingestion pipeline. When configured (commonly with `chunkingStrategy="NONE"` or after initial parsing), Bedrock passes the ingested S3 document references to your Lambda function via an intermediate S3 staging bucket. Your Lambda can execute custom **AST / Tree-Sitter code splitting** so every function or class remains syntactically intact, attach custom per-chunk metadata (`git_sha`, `repo_owner`, `module_path`), and return the transformed chunks to Bedrock for embedding and vector indexing.  
**Distractor Analysis:**
- *A is incorrect:* Converting source code text files into bitmap images wastes storage and degrades code token precision.
- *B is incorrect:* Lowering fixed chunk size to 20 tokens slices every single line of code into isolated fragments, destroying all function context.
- *D is incorrect:* CodeDeploy is an application deployment service and does not provide semantic code retrieval.

---

### Question 5 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A data science team is building a RAG application using Amazon Aurora PostgreSQL Serverless v2 with the `pgvector` extension (`v0.7.0+`). The `document_chunks` table stores 12 million 1,024-dimensional embeddings indexed with an HNSW index (`USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 128)`). During production load testing, data scientists report that recall on complex multi-concept queries is `0.81`, whereas exact brute-force k-NN search achieves `0.96`. The team wants to increase query-time recall to `> 0.93` **immediately** during the current user session **without** dropping or rebuilding the 12-million-row HNSW index (which would take hours).

**Which PostgreSQL session or transaction parameter should the application set before executing the `SELECT ... ORDER BY embedding <=> $1 LIMIT 10` query?**
- A. Execute `SET LOCAL hnsw.ef_search = 150;` (increasing `hnsw.ef_search` from its default of `40` to `150`) within the query transaction to expand the dynamic candidate list evaluated during HNSW graph traversal at runtime without rebuilding the index.
- B. Execute `ALTER INDEX ... SET (ef_construction = 512);` without reindexing, expecting `ef_construction` to dynamically alter runtime search paths on an already-built graph.
- C. Execute `TRUNCATE TABLE document_chunks;` before every `SELECT` query.
- D. Set `ivfflat.probes = 50;` to tune the HNSW index.

**Answer:** A  
**Explanation:** In `pgvector`, HNSW has two distinct sets of parameters:
* **Build-time parameters (`m` and `ef_construction`)**: Govern the graph topology constructed during `CREATE INDEX` / `REINDEX`. Changing them requires a full index rebuild.
* **Query-time parameter (`hnsw.ef_search`, default `40`)**: Specifies the size of the dynamic candidate list maintained during greedy HNSW graph search at query time. Increasing `hnsw.ef_search` (e.g., `SET LOCAL hnsw.ef_search = 150;`) immediately improves recall at the trade-off of slightly higher query CPU/latency, with **zero index rebuild required**.  
**Distractor Analysis:**
- *B is incorrect:* `ef_construction` is only used when inserting nodes into the graph during index build; changing it has no effect on existing index search behavior without a `REINDEX`.
- *C is incorrect:* `TRUNCATE TABLE` deletes all 12 million rows from the database!
- *D is incorrect:* `ivfflat.probes` only applies to `ivfflat` indexes, not `hnsw` indexes.

---

### Question 6 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A multinational consumer goods company operates an enterprise RAG knowledge base in Amazon Bedrock. Business analysts frequently ask natural-language questions that contain implicit temporal and categorical constraints inside the conversational text—for example: *"Show me the EMEA marketing campaign post-mortems from 2025 that had a budget over \$500,000."* Every document in the Knowledge Base already has structured metadata attributes (`region: "EMEA"`, `year: 2025`, `budget: 750000`), but the frontend chat UI only collects a single free-form text box from the user without dropdown filter menus. When the raw query is sent to `Retrieve` without a `RetrievalFilter`, the vector search returns semantically similar 2022 and 2023 North America post-mortems.

**How can the developer automatically enforce hard metadata pre-filtering on `region`, `year`, and `budget` from the user's free-form natural language prompt?**
- A. Set `overrideSearchType="SEMANTIC"` and disable metadata indexing in OpenSearch Serverless.

- B. Increase the embedding vector dimension from `1,024` to `10,000` so cosine similarity acts as a numeric SQL `>` comparator.
- C. Delete all documents from 2022, 2023, and 2024 from Amazon S3 every time a user asks about 2025.
- D. Enable **Implicit Metadata Filtering** (or a pre-retrieval LLM filter-extraction step using `Converse` `toolConfig`) to parse the natural-language query against the Knowledge Base's metadata schema (`region`, `year`, `budget`) and dynamically generate the structured `RetrievalFilter` JSON (`andAll` with `equals` and `greaterThan`) passed into the `Retrieve` / `RetrieveAndGenerate` call.
**Answer:** D  
**Explanation:** Dense vector embeddings struggle with strict numerical inequalities (`budget > 500000`) and exact temporal/region filtering (`year == 2025`, `region == "EMEA"`). Using **Implicit Metadata Filtering** (auto-filter generation where a fast FM inspects the user's natural language query alongside the catalog of metadata attribute names/types and emits a structured `RetrievalFilter` JSON object) allows the vector store to execute hard pre-filtering on `region`, `year`, and `budget` while using semantic similarity for the remaining topical query (*"marketing campaign post-mortems"*).  
**Distractor Analysis:**
- *A is incorrect:* Disabling metadata indexing prevents pre-filtering altogether.
- *B is incorrect:* Embedding dimensions cannot be arbitrarily increased to 10,000, and cosine similarity over dense vectors never guarantees exact numeric inequality filtering (`> 500,000`).
- *C is incorrect:* Deleting historical production data on every query is destructive and breaks concurrent users' queries.

---

### Question 7 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A tax advisory firm is developing an interactive tax-strategy assistant using `anthropic.claude-3-7-sonnet-20250219-v1:0` via the Amazon Bedrock `Converse` API. When users submit intricate multinational transfer-pricing scenarios involving 12 tax treaties and multi-step algebraic tax credit calculations, standard single-pass decoding occasionally makes arithmetic or deduction leaps. The lead AI engineer enables **Extended Thinking** (`reasoningConfig={"thinking": {"type": "enabled", "budgetTokens": 4096}}`) in `additionalModelRequestFields`. However, the API call immediately fails with a `ValidationException`. Upon reviewing the request payload, the engineer notices that `inferenceConfig` is configured with `temperature=0.0`, `topP=0.85`, and `topK=10`.

**Why did enabling Extended Thinking trigger a `ValidationException`, and how must the parameters be adjusted?**
- A. Extended Thinking is only supported on `amazon.titan-text-lite-v1`.
- B. When **Extended Thinking** is enabled on supported Claude models in Amazon Bedrock, modifying `temperature` away from `1.0` (or setting `topK` / `topP` outside supported reasoning bounds) is disallowed, and `maxTokens` must be strictly greater than `budgetTokens`. The engineer must omit custom `temperature`/`topK` overrides (leaving `temperature` at its required value for extended thinking) and ensure `inferenceConfig.maxTokens > budgetTokens`.
- C. `budgetTokens` must be set to a negative integer (such as `-4096`).
- D. Extended Thinking requires disabling HTTPS TLS encryption on the `bedrock-runtime` client.

**Answer:** B  
**Explanation:** When invoking Claude 3.7 Sonnet (or Claude 4 models) with **Extended Thinking** (`thinking: {"type": "enabled", "budgetTokens": N}`), Amazon Bedrock enforces two critical validation rules:
1. **`budgetTokens` must be less than `maxTokens`** (minimum `1,024` tokens, because the thinking budget is a subset of total `maxTokens`).
2. **`temperature` cannot be set to `0.0`**, and `topK` cannot be modified when extended thinking is active (extended thinking requires standard sampling freedom during its internal chain-of-thought exploration). Passing `temperature=0.0` or `topK=10` alongside `thinking.type="enabled"` raises a `ValidationException`.  
**Distractor Analysis:**
- *A is incorrect:* Titan Text Lite is a legacy lightweight model that does not support Extended Thinking.
- *C is incorrect:* `budgetTokens` must be a positive integer $\ge 1024$.
- *D is incorrect:* All Amazon Bedrock API calls require HTTPS TLS encryption.

---

### Question 8 (Domain 1: Foundation Model Integration, Data Management, and Compliance)
A clinical genomics startup wants to fine-tune an `anthropic.claude-3-haiku-20240307-v1:0` (or `amazon.nova-lite-v1:0`) model on Amazon Bedrock using Supervised Fine-Tuning (SFT) so it learns to convert multi-turn doctor-patient genetic counseling dialogues into standardized HL7 FHIR JSON bundles. The data engineering team prepares an S3 training dataset (`train.jsonl`) containing 5,000 records. However, when they submit `CreateModelCustomizationJob`, the job fails during dataset validation. Inspection of `train.jsonl` reveals that the team formatted every line as raw plain text paragraphs without JSON keys, and placed `train.jsonl` in an S3 bucket located in `eu-central-1` while launching the Bedrock customization job in `us-east-1`.

**Which TWO corrections are required for the Supervised Fine-Tuning job to pass validation? (Choose TWO.)**
- A. Delete all `"assistant"` responses from the dataset so the Supervised Fine-Tuning job only sees unlabeled `"user"` questions.

- B. Format every line of `train.jsonl` as a valid JSON object following the Bedrock Converse/Messages fine-tuning schema (`{"schemaVersion": "bedrock-conversation-2024", "system": [...], "messages": [{"role": "user", "content": [...]}, {"role": "assistant", "content": [...]}]}` or prompt/completion pairs depending on the model) with UTF-8 encoding.
- C. Convert `train.jsonl` into a Microsoft Excel `.xlsx` workbook with macro-enabled VBA scripts.
- D. Compress all 5,000 lines into a single line exceeding 2 GB without newline `\n` delimiters.
- E. Copy the `train.jsonl` dataset to an Amazon S3 bucket in the **same AWS Region (`us-east-1`)** where the `CreateModelCustomizationJob` API is executed, and ensure the Bedrock customization IAM role has `s3:GetObject` and `s3:ListBucket` permissions on that bucket.
**Answer:** B, E  
**Explanation:**
- **Why B is correct**: Amazon Bedrock Supervised Fine-Tuning (SFT) requires a valid **JSON Lines (`.jsonl`)** file where each newline-delimited record is a syntactically valid JSON object containing labeled training examples (such as the `messages` conversational format with alternating `user` and `assistant` turns, or `prompt`/`completion` pairs).
- **Why E is correct**: Amazon Bedrock Model Customization jobs require the training and validation S3 buckets to reside in the **exact same AWS Region** as the customization job (`us-east-1`), accessible by the customization execution role.  
**Distractor Analysis:**
- *C is incorrect:* Bedrock customization jobs do not accept `.xlsx` spreadsheets.
- *D is incorrect:* JSONL requires one JSON object per line separated by `\n`.
- *A is incorrect:* Supervised Fine-Tuning requires labeled ground-truth `assistant` completions (omitting completions is only valid for Distillation prompt-only datasets or Continued Pre-Training `{"input": "..."}` schemas).

---

### Question 9 (Domain 2: Implementation and Integration)
A cybersecurity operations center (SOC) is building an automated threat-hunting assistant using **Amazon Bedrock Agents**. When a security analyst asks the agent to investigate a suspicious IP address (`198.51.100.42`), the agent must query 8 months of historical firewall Parquet logs stored in Amazon S3, calculate statistical connection entropy, fit an anomaly score across 50,000 rows, and generate a matplotlib time-series chart of connection spikes for the analyst. The team wants the agent to write and execute the Python `pandas` and `matplotlib` analysis dynamically inside a secure, isolated sandbox managed by Amazon Bedrock without provisioning or maintaining external EC2 or Lambda Python sandboxes.

**Which built-in Amazon Bedrock Agent capability provides this managed Python execution sandbox?**
- A. Attach an AWS WAF WebACL to the agent to execute Python `pandas` scripts.
- B. Enable `AMAZON.UserInput` and ask the human analyst to calculate the Shannon entropy on a pocket calculator.
- C. Enable the built-in **`AMAZON.CodeInterpreter`** (`parentActionGroupSignature="AMAZON.CodeInterpreter"`) Action Group on the Amazon Bedrock Agent so the agent can dynamically write, execute, debug Python code, analyze data files, and return generated chart files in a managed, isolated sandbox environment.
- D. Store the Python script inside an IAM Role trust policy document.

**Answer:** C  
**Explanation:** Amazon Bedrock Agents provide a built-in **`AMAZON.CodeInterpreter`** Action Group (`parentActionGroupSignature: "AMAZON.CodeInterpreter"`). When enabled, the agent can securely write and execute Python code inside an ephemeral, isolated AWS-managed sandbox environment to perform complex mathematical calculations, data analysis (`pandas`, `numpy`), file parsing, and chart generation (`matplotlib`), returning both stdout results and generated artifact files (`files` in the `InvokeAgent` stream) back to the user.  
**Distractor Analysis:**
- *A is incorrect:* AWS WAF inspects HTTP web traffic and cannot execute Python data science scripts.
- *B is incorrect:* `AMAZON.UserInput` simply allows the agent to ask clarifying questions to the human user; it does not execute Python code.
- *D is incorrect:* IAM trust policies only define principal assumption rules in JSON, not executable Python environments.

---

### Question 10 (Domain 2: Implementation and Integration)
A global airline is building a personalized VIP concierge assistant using **Amazon Bedrock Agents**. When a frequent flyer chats with the concierge on Monday (`sessionId="sess-mon-101"`), they mention: *"I am severely allergic to peanuts and always prefer a window seat in the forward cabin."* When the same flyer opens a brand-new chat session inside the mobile app on Friday (`sessionId="sess-fri-999"`), they simply ask: *"Book me on the 6:00 PM flight to London and order my usual meal."* The airline wants the Bedrock Agent to automatically recall the passenger's peanut allergy and window-seat preference from Monday's session without the mobile app having to store, summarize, and re-upload all historical chat transcripts on every new session.

**How should the developer configure the Amazon Bedrock Agent and its `InvokeAgent` API calls to support this cross-session long-term personalization?**
- A. Enable **Memory** (`memoryConfiguration={"enabledMemoryTypes": ["SESSION_SUMMARY"], "storageDays": 90}`) on the Amazon Bedrock Agent, and pass the passenger's unique profile identifier (`memoryId="passenger-uuid-8841"`) alongside each new `sessionId` in every `InvokeAgent` call.
- B. Pass the same hardcoded `sessionId="global-shared-session"` for all 10 million airline passengers simultaneously.
- C. Set `endSession=True` on the very first message of Friday's chat so the agent deletes its current context.
- D. Increase the foundation model's `topK` parameter to `365`.

**Answer:** A  
**Explanation:** **Amazon Bedrock Agent Memory** (`SESSION_SUMMARY`) provides managed cross-session episodic memory. When a session concludes (either via idle timeout or `endSession=True`), Bedrock automatically uses the foundation model to extract and summarize key facts, user preferences, and context discussed during that `sessionId` and stores them under the user's **`memoryId`** for the configured retention period (`storageDays`). In any future session (`sess-fri-999`), passing the same `memoryId="passenger-uuid-8841"` allows the agent to retrieve those historical session summaries automatically via `$memory_search$` / `$memory_synopsis$`.  
**Distractor Analysis:**
- *B is incorrect:* Sharing a single `sessionId` across 10 million users mixes concurrent passengers' conversations together and leaks PII across customers!
- *C is incorrect:* Setting `endSession=True` immediately terminates the current session rather than loading historical cross-session memories.
- *D is incorrect:* `topK` controls token vocabulary sampling and has nothing to do with cross-session memory retention.

---

### Question 11 (Domain 2: Implementation and Integration)
A radiology AI company has trained a custom 3D Vision-Language foundation model (`45 GB` model weights) that analyzes high-resolution volumetric DICOM CT scans (`450 MB` to `850 MB` per patient study) and generates a structured oncology diagnostic report. Processing a single 3D CT volume takes **4 to 7 minutes** of GPU compute on an `ml.g6e.4xlarge` instance. Hospital radiology departments upload studies in bursts during daytime clinic hours, and zero studies are uploaded between 22:00 and 06:00. When the engineering team attempted to deploy this model on an **Amazon SageMaker Real-Time Endpoint**, every request failed because payloads exceeded the 6-MB limit and execution exceeded the 60-second synchronous timeout.

**Which AWS deployment architecture natively supports 850-MB payloads, 7-minute inference durations, and automatic scale-to-zero (`0` GPU instances) overnight?**
- A. Deploy the model inside an AWS Lambda function triggered synchronously by Amazon CloudFront Lambda@Edge.

- B. Deploy the model on **Amazon SageMaker Serverless Inference** with `MemorySizeInMB=6144`.
- C. Base64-encode the 850-MB DICOM file and pass it inside an Amazon API Gateway URL query string parameter (`?scan=...`).
- D. Deploy the model on an **Amazon SageMaker Asynchronous Inference** endpoint: upload the 850-MB DICOM archives to Amazon S3, invoke `InvokeEndpointAsync` with `InputLocation` pointing to the S3 URI and `InvocationTimeoutSeconds=900`, configure Amazon SNS completion/error notifications, and attach an Application Auto Scaling policy with `MinCapacity=0` so the GPU fleet scales to zero instances when the internal queue is empty overnight.
**Answer:** D  
**Explanation:** **Amazon SageMaker Asynchronous Inference** is specifically designed for large payloads (**up to 1 GB** stored in Amazon S3) and long processing times (**up to 1 hour / 3,600 seconds**). It maintains an internal managed request queue, writes output artifacts to S3, publishes completion events to **Amazon SNS**, and—unlike standard Real-Time Endpoints when using legacy configurations—supports scaling down to **`0` instances (`MinCapacity=0`)** via Application Auto Scaling when `ApproximateBacklogSizePerInstance` drops to zero overnight, eliminating idle GPU costs.  
**Distractor Analysis:**
- *A is incorrect:* Lambda@Edge has a 5-second (viewer) or 30-second (origin) timeout and a 1-MB/40-KB payload limit.
- *B is incorrect:* SageMaker Serverless Inference has a hard **4-MB payload limit**, a **60-second timeout**, no GPU support, and max 6 GB RAM.
- *C is incorrect:* URL query strings are limited to a few kilobytes, and API Gateway has a 10-MB payload limit and 29-second synchronous timeout.

---

### Question 12 (Domain 2: Implementation and Integration)
A financial trading desktop application installed on 5,000 traders' corporate Windows workstations uses an **Amazon Bedrock Agent** to assist with portfolio rebalancing. One of the agent's tools (`inspectLocalExcelPortfolio`) must read an unsaved, active Microsoft Excel spreadsheet (`C:\Users\trader\Desktop\live_positions.xlsx`) that exists **only on the trader's local desktop memory/disk** and is unreachable from the AWS Cloud or any VPC.

**How should the Action Group for `inspectLocalExcelPortfolio` be configured in the Amazon Bedrock Agent?**
- A. Upload the entire Windows C:\ drive of all 5,000 laptops to Amazon OpenSearch Serverless every 5 seconds.
- B. Configure an AWS Lambda function in `us-east-1` to open a reverse RDP connection into all 5,000 traders' laptops over the public internet.
- C. Configure the Action Group with **`customControl="RETURN_CONTROL"`** (Return of Control) instead of a Lambda function ARN. When the agent decides to call `inspectLocalExcelPortfolio`, `InvokeAgent` returns a `returnControl` event containing the `invocationId` and parameters to the trader's desktop client; the desktop client reads the local Excel file and passes the result back in a subsequent `InvokeAgent` call via `sessionState.returnControlInvocationResults`.
- D. Set `agentCollaboration="SUPERVISOR"` so Bedrock can bypass corporate firewalls and read local laptop RAM directly.

**Answer:** C  
**Explanation:** **Return of Control (ROC)** (`customControl: "RETURN_CONTROL"`) in Amazon Bedrock Agents delegates tool execution back to the calling client application rather than invoking an AWS Lambda function. When the agent's ReAct loop selects an ROC tool, Bedrock pauses orchestration and emits a `returnControl` payload in the `InvokeAgent` response stream (`invocationId` + `invocationInputs`). The desktop application executes the local COM/Excel inspection on the workstation and resumes the agent turn by sending the output inside `sessionState.returnControlInvocationResults`.  
**Distractor Analysis:**
- *A is incorrect:* Continuously syncing 5,000 full hard drives to OpenSearch every 5 seconds is impossible and still cannot read unsaved in-memory Excel buffers.
- *B is incorrect:* Opening inbound RDP ports from the cloud into trader workstations is a severe security violation and architecturally unworkable across NAT/firewalls.
- *D is incorrect:* Bedrock runs in the AWS Cloud and cannot directly access a client workstation's local filesystem without Return of Control.

---

### Question 13 (Domain 2: Implementation and Integration)
A global SaaS company is building a mission-critical customer service AI backend in Python (`boto3`) that invokes Amazon Bedrock (`Converse` API). During peak flash-sale events, regional traffic spikes occasionally trigger HTTP `429 ThrottlingException` or transient HTTP `503 ServiceUnavailableException` responses from the primary model in `us-east-1`. The Principal Engineer wants to configure the `botocore` client and application invocation layer to implement AWS best practices for **token-aware retry resilience** and **automated fallback** without causing retry storms (thundering herds) that amplify the outage.

**Which TWO engineering practices should be implemented? (Choose TWO.)**
- A. Catch `ThrottlingException` in a `while True:` infinite loop with `time.sleep(0)` so failed requests are retried 1,000 times per second.
- B. Disable TLS certificate verification (`verify=False`) on the `boto3` client to bypass HTTP 429 throttling quotas.
- C. Configure the `boto3` `bedrock-runtime` client using `botocore.config.Config(retries={"mode": "adaptive", "max_attempts": 8})`, which implements client-side token-bucket rate limiting and exponential backoff with full random jitter on retryable `429`/`5xx` exceptions.
- D. Configure the primary invocation to use a **Cross-Region Inference Profile** (e.g., `us.anthropic.claude-3-5-sonnet-20241022-v2:0`), and implement a **Circuit Breaker** fallback chain that switches to a secondary approved model family or region if consecutive retries fail.
- E. Replace `boto3` with raw UDP broadcast packets sent to port 53 of `bedrock-runtime.us-east-1.amazonaws.com`.

**Answer:** C, D  
**Explanation:**
- **Why C is correct**: `botocore`'s **`adaptive` retry mode** combines standard exponential backoff with **full jitter** (preventing synchronized retry waves across thousands of workers) and a **client-side token bucket rate limiter** that dynamically throttles outgoing request rates when `429 ThrottlingException` responses are observed.
- **Why D is correct**: Combining **Cross-Region Inference Profiles (`us.*`)** (which automatically load-balance across multiple AWS Regions) with an application-level **Circuit Breaker** that fails over to a backup model (e.g., Amazon Nova Pro or Claude Haiku) prevents cascading timeouts during regional capacity events.  
**Distractor Analysis:**
- *A is incorrect:* An immediate `while True` retry loop without backoff or jitter creates a catastrophic retry storm that exhausts quotas and worsens throttling.
- *B is incorrect:* Disabling TLS verification introduces man-in-the-middle vulnerabilities and has zero effect on server-side RPM/TPM quotas.
- *E is incorrect:* Bedrock APIs run over HTTPS/TCP port 443, not UDP port 53.

---

### Question 14 (Domain 2: Implementation and Integration)
A platform engineering team manages an enterprise **Amazon Bedrock Agent** and its associated **Amazon Bedrock Guardrail** using AWS CloudFormation / AWS CDK across `Dev`, `Staging`, and `Prod` environments. Currently, a microservice in `Prod` invokes the Agent using the mutable `TSTALIASID` (working draft test alias) and sets `guardrailVersion="DRAFT"`. Last week, a developer experimenting with a new prompt template in the Bedrock console saved a broken orchestration prompt to the working draft, which immediately caused `Prod` customer traffic to fail.

**How should the team refactor their CI/CD deployment and runtime invocation architecture to guarantee zero-downtime immutable releases and instant rollback?**
- A. Continue using `TSTALIASID` and `DRAFT` in production, but revoke console access for all developers except on Fridays.
- B. Configure the CI/CD pipeline to publish an **immutable numbered Agent Version** (via an **Agent Alias** such as `prod-alias` pointing to `agentVersion: "4"`) and an **immutable numbered Guardrail Version** (`CreateGuardrailVersion`, e.g., `version: "3"`). Configure the production application to invoke `prod-alias` (`agentAliasId`) and the numbered Guardrail version—enabling atomic alias pointer updates and sub-second rollbacks if canary metrics degrade.
- C. Create a new AWS account every time a prompt is edited.
- D. Hardcode the Agent's internal foundation model weights inside an Amazon Route 53 TXT record.

**Answer:** B  
**Explanation:** In Amazon Bedrock, both **Agents** (`TSTALIASID` / `DRAFT`) and **Guardrails** (`DRAFT`) have mutable working drafts intended strictly for iterative development. Production workloads must **never** invoke `TSTALIASID` or `DRAFT`. Instead, CI/CD pipelines must snapshot an **immutable numbered version** (`CreateAgentAlias` / `UpdateAgentAlias` creating/updating an alias pointing to a numbered `agentVersion`, and `CreateGuardrailVersion` producing an immutable numbered guardrail version). Decoupling application code (which calls the stable `agentAliasId`) from the underlying immutable `agentVersion` allows atomic traffic shifting and instant rollback without redeploying application code.  
**Distractor Analysis:**
- *A is incorrect:* Using `TSTALIASID` and `DRAFT` in production exposes live users to any in-progress draft edit and provides no immutable rollback artifact.
- *C is incorrect:* Creating new AWS accounts per prompt edit is an operational anti-pattern.
- *D is incorrect:* Route 53 TXT records are limited to 255-character DNS strings and cannot store model weights or agent versions.

---

### Question 15 (Domain 3: AI Safety, Security, and Governance)
A hospital network is deploying a patient-intake assistant powered by Amazon Bedrock. Patients type symptom descriptions into a web portal before their appointment. Often, patients inadvertently include their **US Social Security Number (SSN)**, **Credit Card Number**, and proprietary **Hospital Medical Record Number (`MRN-` followed by 8 digits and 2 uppercase letters, e.g., `MRN-49102844-TX`)** in their free-text message. HIPAA security policy requires that:
1. All standard PII (SSNs, Credit Cards, Phone Numbers, Emails) AND the custom `MRN-########-XX` identifier must be **masked/redacted (`ANONYMIZE`)** before the prompt is logged or processed, rather than rejecting the patient's message with an error screen.
2. However, if a user attempts to ask the assistant for instructions on synthesizing controlled narcotics, the request must be **completely blocked (`BLOCK`)** with a standard refusal message.

**How should the **Amazon Bedrock Guardrail** be configured to meet both requirements simultaneously?**
- A. Configure **Sensitive Information Filters** by adding managed PII entities (`US_SOCIAL_SECURITY_NUMBER`, `CREDIT_DEBIT_CARD_NUMBER`, `PHONE`, `EMAIL`) and a **Custom Regex** (`^MRN-\d{8}-[A-Z]{2}$` / `MRN-[0-9]{8}-[A-Z]{2}`) with both `inputAction` and `outputAction` set to **`ANONYMIZE`**, and configure a **Denied Topic** (for controlled narcotics synthesis) with `inputAction` and `outputAction` set to **`BLOCK`**.
- B. Set all Sensitive Information Filters to `BLOCK` so patients who accidentally type their phone number or MRN receive an error message and cannot book an appointment.
- C. Disable Sensitive Information Filters and rely on the foundation model's `topP` setting to hide SSNs.
- D. Use AWS Certificate Manager (ACM) to redact `MRN-########-XX` strings inside JSON payloads.

**Answer:** A  
**Explanation:** **Amazon Bedrock Guardrails Sensitive Information Filters** support both built-in PII entity recognition (30+ categories including SSN, credit cards, email, phone) and **Custom Regular Expressions (`regexes`)** for organization-specific identifiers like `MRN-[0-9]{8}-[A-Z]{2}`. Each PII entity and custom regex can be independently configured with action **`ANONYMIZE`** (which replaces the sensitive string with a placeholder tag like `{US_SOCIAL_SECURITY_NUMBER_1}` or `{MRN_ID_1}` while allowing the conversation to proceed smoothly) or **`BLOCK`**. Meanwhile, **Denied Topics** block prohibited intents (narcotics synthesis) outright and return the configured `blockedInputMessaging`.  
**Distractor Analysis:**
- *B is incorrect:* Setting PII action to `BLOCK` rejects the patient's message entirely instead of masking/anonymizing the sensitive fields as required.
- *C is incorrect:* `topP` is a token sampling hyperparameter and provides zero PII redaction.
- *D is incorrect:* ACM manages TLS/SSL X.509 certificates and does not inspect or redact application text payloads.

---

### Question 16 (Domain 3: AI Safety, Security, and Governance)
A financial institution operates a strict Zero-Trust VPC architecture in `us-east-1`. Security policy mandates that:
1. All `bedrock-runtime:Converse` and `bedrock-runtime:InvokeModel` API traffic originating from private ECS tasks in VPC `vpc-0a1b2c3d` must remain entirely on the private AWS network via an **AWS PrivateLink Interface VPC Endpoint** (`com.amazonaws.us-east-1.bedrock-runtime`).
2. Even if an attacker compromises an IAM role inside the VPC or imports external AWS credentials from an unauthorized AWS account, the VPC endpoint itself must **only** permit invocations originating from principals belonging to AWS Organization `o-corp999999` and must **only** permit invoking `anthropic.claude-3-5-sonnet-20241022-v2:0` and `amazon.titan-embed-text-v2:0`.

**Which configuration enforces this perimeter control at the network endpoint boundary?**
- A. Use an Amazon S3 Gateway VPC Endpoint to route `bedrock-runtime` API calls.
- B. Attach a public Internet Gateway to the ECS subnets and configure a Route 53 CNAME record.
- C. Attach a custom **VPC Endpoint Policy** to the `com.amazonaws.us-east-1.bedrock-runtime` Interface VPC Endpoint that allows `bedrock:InvokeModel*` only on the two approved Foundation Model ARNs and includes a `Condition` verifying `"StringEquals": {"aws:PrincipalOrgID": "o-corp999999"}`, while removing any NAT Gateway or Internet Gateway route from the private ECS subnets.
- D. Add `"aws:PrincipalOrgID": "o-corp999999"` inside the foundation model's `system` prompt string.

**Answer:** C  
**Explanation:** An **AWS PrivateLink Interface VPC Endpoint** (`com.amazonaws.<region>.bedrock-runtime`) routes all Bedrock runtime API traffic privately over the AWS network without traversing the public internet or a NAT Gateway. Attaching a **VPC Endpoint Policy** directly to the interface endpoint creates a hard network-perimeter choke point: even if stolen credentials from an external AWS account are used inside the VPC, the VPC Endpoint Policy rejects any request where `aws:PrincipalOrgID != "o-corp999999"` or where the target resource ARN is not one of the two explicitly listed foundation models.  
**Distractor Analysis:**
- *A is incorrect:* Gateway VPC Endpoints only exist for **Amazon S3** and **Amazon DynamoDB**; Amazon Bedrock requires an **Interface VPC Endpoint** (AWS PrivateLink).
- *B is incorrect:* Routing over a public Internet Gateway violates the Zero-Trust private networking requirement.
- *D is incorrect:* Putting an Organization ID in a text prompt has zero IAM or network security enforcement effect.

---

### Question 17 (Domain 3: AI Safety, Security, and Governance)
A digital publishing marketplace allows freelance graphic designers to upload promotional illustrations. However, the marketplace's terms of service require automatic disclosure whenever an uploaded image was generated by an AI model. Specifically, the platform team needs an automated API step in their S3 upload Lambda pipeline to verify whether an incoming PNG/JPEG image was generated by **Amazon Titan Image Generator** (or **Amazon Nova Canvas**), even if the designer cropped the image, applied a slight JPEG re-compression filter, or stripped the EXIF metadata headers.

**Which AWS capability detects this provenance reliably?**
- A. Use Amazon Textract `AnalyzeExpense` on the illustration.

- B. Check whether the S3 object key ends with `_ai_generated.png`.
- C. Inspect the standard EXIF `CameraMake` tag using Python `Pillow`.
- D. Invoke Amazon Bedrock's **Watermark Detection** capability (`DetectGeneratedContent` / Titan Image Generator watermark verification), which inspects the tamper-resistant, invisible cryptographic watermark embedded directly into pixel frequency domains by Amazon Titan Image Generator and Amazon Nova Canvas.
**Answer:** D  
**Explanation:** All images generated by **Amazon Titan Image Generator** and **Amazon Nova Canvas** automatically contain an **invisible, tamper-resistant watermark** embedded directly into the image pixel data (independent of C2PA/EXIF file headers). Amazon Bedrock's **Watermark Detection** API analyzes the image and returns a confidence score indicating whether the image was generated by an Amazon foundation model, remaining resilient against common manipulations such as cropping, resizing, compression, and EXIF stripping.  
**Distractor Analysis:**
- *A is incorrect:* Textract `AnalyzeExpense` extracts line items from receipts and invoices, not invisible AI image watermarks.
- *B is incorrect:* Filenames are trivially renamed by the uploader.
- *C is incorrect:* EXIF headers are stripped in seconds by any image editor or social media upload tool.

---

### Question 18 (Domain 3: AI Safety, Security, and Governance)
A clinical decision-support system uses an Amazon Bedrock RAG pipeline with **Amazon Bedrock Guardrails Contextual Grounding Checks** to draft discharge instructions for cardiology patients. Hospital risk management policy defines three grounding score tiers for every generated draft:
* If `grounding_score >= 0.90`: Immediately approve and send the instructions to the electronic health record (EHR).
* If `0.65 <= grounding_score < 0.90`: Do **not** discard the draft, and do **not** auto-send it; instead, route the generated draft alongside the source medical chart passages to a queue of on-duty Registered Nurses for human verification, editing, and sign-off before releasing to the EHR.
* If `grounding_score < 0.65`: Automatically block the draft and trigger a deterministic regeneration fallback.

**Which AWS architecture implements the middle tier (`0.65 <= grounding_score < 0.90`) with a managed human review portal and workflow state tracking?**
- A. Email the raw JSON trace to a public mailing list and ask nurses to SSH into the production database to run `UPDATE` queries.
- B. Orchestrate the pipeline with **AWS Step Functions**: inspect the Guardrail trace's `contextualGroundingPolicy` score from `ApplyGuardrail` / `Converse`, and when the score falls between `0.65` and `0.90`, invoke an **Amazon Augmented AI (Amazon A2I)** human review loop (`StartHumanLoop`) backed by a private **Amazon SageMaker Ground Truth** workforce of Registered Nurses, resuming the Step Functions execution once the nurse submits the reviewed output to S3.
- C. Set `temperature=0.99` whenever `grounding_score < 0.90`.
- D. Use AWS Auto Scaling lifecycle hooks to pause EC2 instances for 48 hours.

**Answer:** B  
**Explanation:** **Amazon Augmented AI (Amazon A2I)** integrates with **AWS Step Functions** and **Amazon SageMaker Ground Truth** private workforces to build managed **Human-in-the-Loop (HITL)** review workflows for generative AI applications. By inspecting the numeric `score` returned in the Guardrail `contextualGroundingPolicy` trace (`GROUNDING` and `RELEVANCE`), Step Functions can branch deterministically: auto-approving high-confidence responses ($\ge 0.90$), routing borderline responses ($0.65\text{–}0.89$) to a secure nurse review UI via `StartHumanLoop`, and rejecting low-score hallucinations ($< 0.65$).  
**Distractor Analysis:**
- *A is incorrect:* Emailing PHI to a mailing list and granting SSH database access violates HIPAA security and auditability mandates.
- *C is incorrect:* Raising `temperature` to `0.99` increases hallucination randomness rather than providing human clinical oversight.
- *D is incorrect:* EC2 Auto Scaling lifecycle hooks manage VM launch/termination states, not human review of LLM completions.

---

### Question 19 (Domain 3: AI Safety, Security, and Governance)
An enterprise CISO manages an AWS Organization with 80 member accounts. While application teams in member accounts are permitted to invoke Amazon Bedrock foundation models, corporate AI governance policy mandates that **no IAM user, role, or even a member-account `AdministratorAccess` principal** may create, update, or delete an Amazon Bedrock Guardrail (`gr-corp-mandatory-01`), nor may they invoke any Bedrock model in any region other than `us-east-1` and `us-west-2`.

**Which governance control enforces these guardrails across all 80 member accounts so that even member-account administrators cannot bypass them?**
- A. Attach an **AWS Organizations Service Control Policy (SCP)** to the workload Organizational Units (OUs) that denies `bedrock:*` in all regions except `us-east-1` and `us-west-2` (`aws:RequestedRegion`), denies `bedrock:DeleteGuardrail` and `bedrock:UpdateGuardrail`, and denies `bedrock:InvokeModel*` when `bedrock:GuardrailIdentifier` does not equal the approved corporate Guardrail ARN.
- B. Add a comment inside each developer's `~/.bashrc` file asking them only to use `us-east-1`.
- C. Create an IAM inline policy on a single developer user in one account; it will automatically apply to all 80 accounts.
- D. Configure an Amazon Bedrock Guardrail Word Filter containing the string `"eu-west-1"`.

**Answer:** A  
**Explanation:** **AWS Organizations Service Control Policies (SCPs)** define the maximum permission guardrails for all IAM users and roles (including `AdministratorAccess` and the member account root user) across member accounts in an Organization. An SCP combining `aws:RequestedRegion` restrictions, `Deny` on `bedrock:DeleteGuardrail`/`bedrock:UpdateGuardrail`, and a `StringNotEquals` condition on `bedrock:GuardrailIdentifier` guarantees that no principal in any member account can invoke models outside approved regions or bypass the mandatory corporate Guardrail.  
**Distractor Analysis:**
- *B is incorrect:* Shell comments provide zero security enforcement.
- *C is incorrect:* IAM inline policies only apply to the single identity they are attached to and can be removed by a member-account administrator.
- *D is incorrect:* A Guardrail Word Filter inspects prompt text, not the AWS Region control plane endpoint where an API call is sent.

---

### Question 20 (Domain 4: Operational Efficiency and Optimization for GenAI Applications)
An enterprise customer-service platform routes 3 million daily prompts to Amazon Bedrock. An audit of the production logs shows that 72% of customer prompts are straightforward intent-classification, greeting, or simple FAQ lookups that `anthropic.claude-3-5-haiku-20241022-v1:0` (or `amazon.nova-lite-v1:0`) answers with identical quality to `anthropic.claude-3-5-sonnet-20241022-v2:0` (or `amazon.nova-pro-v1:0`), while the remaining 28% involve complex multi-clause contract disputes that require the larger frontier model. The engineering team wants a **single managed Bedrock endpoint ARN** that automatically evaluates each incoming prompt's complexity in real time and routes simple requests to the smaller, faster model and complex requests to the larger model within the same model family—reducing token costs by 35% without writing or maintaining custom classifier models.

**Which native Amazon Bedrock feature provides this capability?**
- A. Enable S3 Intelligent-Tiering on the application's log bucket.
- B. Use Amazon Route 53 Geolocation DNS routing to split traffic by continent.
- C. Configure **Amazon Bedrock Intelligent Prompt Routing** by selecting or creating a Prompt Router ARN (e.g., for the Claude family or Nova family) and tuning the router's response quality/cost threshold criteria, then pass the Prompt Router ARN as the `modelId` in the `Converse` / `InvokeModel` API call.
- D. Set `maxTokens=1` on half of all `Converse` requests at random.

**Answer:** C  
**Explanation:** **Amazon Bedrock Intelligent Prompt Routing** provides a single unified router endpoint across models within the same family (such as Claude 3.5 Sonnet + Claude 3.5 Haiku, or Amazon Nova Pro + Amazon Nova Lite). When you pass the Prompt Router ARN as the `modelId` in `Converse` or `InvokeModel`, Bedrock's built-in ultra-low-latency routing predictor analyzes the complexity of each prompt before inference and routes it to the most cost-effective model capable of meeting your configured quality threshold—reducing cost and latency by up to 30–50% with zero custom routing code.  
**Distractor Analysis:**
- *A is incorrect:* S3 Intelligent-Tiering optimizes object storage classes, not foundation model selection.
- *B is incorrect:* Route 53 Geolocation routes network packets based on client IP geography, not prompt semantic complexity.
- *D is incorrect:* Setting `maxTokens=1` truncates responses to a single token and breaks the application.

---

### Question 21 (Domain 4: Operational Efficiency and Optimization for GenAI Applications)
A real-time voice-to-voice AI contact center agent powered by Amazon Bedrock has a strict end-to-end voice turn latency budget of **700 milliseconds** before human callers perceive an awkward pause. Profiling via AWS X-Ray and CloudWatch (`AWS/Bedrock`) reveals that **Time to First Token (TTFT)** on the foundation model is currently averaging 920 ms during peak afternoon call volumes. The engineering team wants to minimize TTFT and inter-token generation latency without changing their application business logic or training a custom model.

**Which THREE optimizations directly reduce Time to First Token (TTFT) and streaming latency on Amazon Bedrock? (Choose THREE.)**
- A. Enable **Latency-Optimized Inference** by setting `performanceConfig={"latency": "optimized"}` in the `ConverseStream` / `InvokeModelWithResponseStream` call.
- B. Prepend a 50,000-token uncached random encyclopedia article to every single voice turn.
- C. Enable **Prompt Caching (`cachePoint`)** after the static system prompt and tool definitions so Bedrock reuses precomputed KV-cache attention states rather than recomputing prefill on every voice turn.
- D. Switch from `ConverseStream` to the non-streaming `Converse` API and wait for the entire 500-token paragraph to finish generating before starting text-to-speech synthesis.
- E. Configure the attached Amazon Bedrock Guardrail to use **`streamProcessingMode="ASYNCHRONOUS"`** so token chunks stream immediately to the text-to-speech engine while guardrail validation runs in parallel.
- F. Route all API calls through a proxy server in a distant continent over a 56 kbps dial-up VPN.

**Answer:** A, C, E  
**Explanation:**
- **Why A is correct**: Setting **`performanceConfig={"latency": "optimized"}`** activates **Amazon Bedrock Latency-Optimized Inference**, running inference on specialized high-speed hardware configurations that significantly reduce both TTFT and inter-token latency (TPS).
- **Why C is correct**: TTFT is dominated by the **prefill phase** (computing KV attention states over all input tokens). **Prompt Caching (`cachePoint`)** skips prefill computation for cached static prefixes, cutting TTFT by up to 85%.
- **Why E is correct**: Setting Guardrail **`streamProcessingMode="ASYNCHRONOUS"`** in `ConverseStream` eliminates synchronous guardrail buffering latency before the first token is emitted to the downstream Text-to-Speech (Amazon Polly / Nova Sonic) stream.  
**Distractor Analysis:**
- *D is incorrect:* Switching from streaming (`ConverseStream`) to blocking (`Converse`) forces the caller to wait for the *last* token before audio synthesis can begin, adding several seconds of silence.
- *B is incorrect:* Adding 50,000 uncached input tokens dramatically increases prefill compute time and worsens TTFT.
- *F is incorrect:* Hair-pinning traffic across continents adds hundreds of milliseconds of network RTT latency.

---

### Question 22 (Domain 4: Operational Efficiency and Optimization for GenAI Applications)
A FinOps architect is evaluating whether to migrate a production document-extraction workload from **Amazon Bedrock On-Demand** to **Amazon Bedrock Provisioned Throughput** (`1-month` commitment). The workload runs exclusively between `09:00` and `17:00` Monday through Friday (40 hours per week, or ~173 hours per month) and is completely idle for the remaining 557 hours of the month. During active hours, the workload consumes a steady 2 million input tokens and 400,000 output tokens per hour (`346M` input tokens and `69.2M` output tokens per month). At On-Demand rates (\$0.003 per 1K input tokens and \$0.015 per 1K output tokens), the monthly On-Demand bill is:
$$\text{On-Demand Monthly Cost} = (346,000 \times \$0.003) + (69,200 \times \$0.015) = \$1,038 + \$1,038 = \$2,076/\text{month}$$
Purchasing the 2 Model Units (MUs) of Provisioned Throughput required to handle peak daytime concurrency costs \$32.00/hour per MU (\$64.00/hour total $\times 730\text{ hours} = \$46,720/\text{month}$). However, the application occasionally experiences brief `ThrottlingException` spikes at 09:05 AM on Mondays.

**What should the FinOps architect recommend to eliminate the Monday 09:05 AM throttling spikes while minimizing monthly AWS spend?**
- A. Delete the extraction application and process documents manually by fax.

- B. Purchase the 2 Model Units of Provisioned Throughput with a 6-month commitment so the company pays over \$35,000/month for capacity that sits idle 76% of the month.
- C. Purchase 10 Model Units of Provisioned Throughput with no commitment.
- D. Keep the workload on **On-Demand billing** (~$2,076/month instead of $46,720/month), switch the `modelId` to a **Cross-Region Inference Profile (`us.*`)** to multiply available burst TPM/RPM quotas across US regions at zero extra routing cost, and enable **Prompt Caching (`cachePoint`)** on the static extraction instructions.
**Answer:** D  
**Explanation:** Provisioned Throughput Model Units (MUs) are billed **24/7 (730 hours/month)** regardless of utilization. When a workload is only active 24% of the month (40 hours/week) and its On-Demand token spend is \$2,076/month, paying \$46,720/month for dedicated MUs is a 22x cost increase. Switching to a **Cross-Region Inference Profile (`us.*`)** dynamically pools On-Demand capacity across multiple regions to absorb the Monday 09:05 AM burst without throttling, while maintaining On-Demand per-token pricing and pairing with **Prompt Caching** to lower the \$2,076/month bill even further.  
**Distractor Analysis:**
- *B and C are incorrect:* Both options incur tens of thousands of dollars per month in wasted idle MU capacity compared to \$2,076/month on On-Demand.
- *A is incorrect:* Manual fax processing fails the automation requirement.

---

### Question 23 (Domain 5: Testing, Validation, and Troubleshooting)
An enterprise AI team wants to enforce an automated **LLMOps Quality Gate** inside their **AWS CodePipeline / AWS CodeBuild** deployment pipeline whenever an engineer modifies a prompt template in Amazon Bedrock Prompt Management or updates chunking parameters on their Knowledge Base. Specifically, before the pipeline updates the production Agent Alias (`prod-alias`), it must automatically run a 200-question golden evaluation benchmark (`s3://llmops-eval-gold/benchmark_v4.jsonl`), score **Context Relevance**, **Groundedness (Faithfulness)**, and **Answer Correctness** using an LLM-as-a-Judge evaluator (`anthropic.claude-3-5-sonnet-20241022-v2:0`), and **fail the CodePipeline build stage** if the mean Groundedness score drops below `0.90` or if Answer Correctness drops below `0.85`.

**How should the engineer implement this automated CI/CD regression gate?**
- A. Rely on AWS CloudFormation `cfn-lint` syntax checking to measure whether RAG Groundedness is above `0.90`.
- B. Configure an AWS CodeBuild step (or Step Functions state machine invoked by CodePipeline) that calls `bedrock:CreateEvaluationJob` (`applicationType="RAG_EVALUATION"`) against the candidate Knowledge Base/Prompt version and `benchmark_v4.jsonl`, polls `GetEvaluationJob` until `status == "Completed"`, parses the output JSONL summary scores from the S3 evaluation results bucket, and exits with a non-zero exit code (`sys.exit(1)`) to block `UpdateAgentAlias` if Groundedness $< 0.90$ or Correctness $< 0.85$.
- C. Deploy the unverified prompt directly to `prod-alias` and wait 30 days to see if customers complain on social media.
- D. Use Amazon Inspector container CVE scanning to calculate LLM-as-a-Judge Answer Correctness scores.

**Answer:** B  
**Explanation:** Integrating **Amazon Bedrock Evaluations (`CreateEvaluationJob`)** into CI/CD pipelines provides deterministic, quantitative regression gating for non-deterministic GenAI changes (prompt edits, model upgrades, chunking changes). By running an automated RAG evaluation job against a curated S3 golden dataset (`prompt`, `referenceResponse`, `referenceContexts`) and asserting threshold gates (`Groundedness >= 0.90`, `Correctness >= 0.85`) on the JSONL output metrics written to S3 by Bedrock before executing `UpdateAgentAlias`, the team prevents hallucination or accuracy regressions from reaching production.  
**Distractor Analysis:**
- *A is incorrect:* `cfn-lint` only validates static CloudFormation YAML/JSON syntax; it cannot evaluate runtime LLM factual groundedness.
- *C is incorrect:* Releasing untested prompt changes to 100% of production traffic violates LLMOps safety and regression standards.
- *D is incorrect:* Amazon Inspector scans OS packages and container images for software vulnerabilities (CVEs), not RAG semantic quality metrics.

---

### Question 24 (Domain 5: Testing, Validation, and Troubleshooting)
A developer is building an **Amazon Bedrock Agent** that invokes an Action Group backed by an AWS Lambda function (`calculateLoanAmortization`). When testing in the Bedrock console with **Trace** enabled, the developer sees that:
1. The Agent correctly generates the tool call in `orchestrationTrace.invocationInput`.
2. The AWS Lambda function executes without errors in CloudWatch Logs and returns:
   ```json
   {
     "statusCode": 200,
     "headers": {"Content-Type": "application/json"},
     "body": "{\"monthlyPayment\": 2415.50, \"totalInterest\": 369580.00}"
   }
   ```
3. Immediately after the Lambda returns, the Bedrock Agent fails with a `DependencyFailedException` / `BadGatewayException` in `failureTrace`, stating that the response received from the Lambda function could not be parsed!

**Why did the Bedrock Agent reject the Lambda function's `200 OK` response, and how must the Lambda return value be fixed?**
- A. The developer formatted the Lambda return dictionary using the **Amazon API Gateway proxy integration schema** (`statusCode`, `headers`, `body`) instead of the mandatory **Amazon Bedrock Agent Action Group response schema**. The Lambda must return a dictionary containing `"messageVersion": "1.0"` and a `"response"` object specifying `"actionGroup"`, `"apiPath"` (or `"function"`), `"httpMethod"`, `"httpStatusCode": 200`, and `"responseBody": {"application/json": {"body": "..."}}`.
- B. The Lambda function must return raw HTML (`<html><body>...</body></html>`) instead of JSON.
- C. Amazon Bedrock Agents only accept `httpStatusCode: 404` from Lambda functions.
- D. The developer must base64-encode the entire Python source file of the Lambda function at runtime.

**Answer:** A  
**Explanation:** A classic implementation error when building Amazon Bedrock Agent Action Groups is returning an API Gateway proxy payload (`{"statusCode": 200, "body": "..."}`) from the Lambda function. Amazon Bedrock Agents strictly require the **Bedrock Agent Action Group response envelope**:
* For **OpenAPI schema** Action Groups:
  ```json
  {
    "messageVersion": "1.0",
    "response": {
      "actionGroup": event["actionGroup"],
      "apiPath": event["apiPath"],
      "httpMethod": event["httpMethod"],
      "httpStatusCode": 200,
      "responseBody": {
        "application/json": {
          "body": "{\"monthlyPayment\": 2415.50, \"totalInterest\": 369580.00}"
        }
      }
    }
  }
  ```
* (Or for **Function Details** Action Groups: `"function"` and `"functionResponse": {"responseBody": {"TEXT": {"body": "..."}}}`). Returning an API Gateway envelope causes Bedrock to fail response deserialization with `DependencyFailedException` / `BadGatewayException`.  
**Distractor Analysis:**
- *B is incorrect:* Returning raw un-enveloped HTML also violates the mandatory `messageVersion: "1.0"` JSON envelope.
- *C is incorrect:* `httpStatusCode: 200` is the standard success code inside the `response` object.
- *D is incorrect:* Base64-encoding the Lambda's source code does not fix the response payload contract.

---

### Question 25 (Domain 5: Testing, Validation, and Troubleshooting)
An enterprise AI team currently uses an Amazon Bedrock Knowledge Base backed by an Amazon OpenSearch Serverless `VECTORSEARCH` collection (`index-prod-v1`). When the Knowledge Base was originally deployed 10 months ago, it was configured with `amazon.titan-embed-text-v1` (`1,536` dimensions). Today, the team wants to migrate the Knowledge Base to `amazon.titan-embed-text-v2:0` configured with `512` dimensions to cut vector storage footprint by 66% and support normalized cosine similarity. When an engineer runs `aws bedrock-agent update-knowledge-base` to change the `embeddingModelArn` on the existing Knowledge Base and existing OpenSearch vector index in-place, the API call fails with a `ValidationException`.

**Why does updating the `embeddingModelArn` and vector dimension in-place fail, and what is the correct migration procedure?**
- A. The engineer must stop and restart the OpenSearch Serverless EC2 instance via the EC2 console before changing the dimension.

- B. The engineer only needs to truncate the last 1,024 floating-point numbers from each existing `titan-embed-text-v1` vector using an OpenSearch `DELETE` query.
- C. `amazon.titan-embed-text-v2:0` cannot be used with Amazon OpenSearch Serverless.
- D. Both the `embeddingModelArn` on an existing Amazon Bedrock Knowledge Base and the `dimension` mapping (`1536` vs `512`) of an existing OpenSearch Serverless `knn_vector` index field are **immutable after creation** (because mixing 1,536-dimensional and 512-dimensional vectors in the same index is mathematically invalid). The team must create a **new vector index** configured with `dimension: 512` and a **new Knowledge Base** (using a blue/green cutover pattern), run `StartIngestionJob` to re-embed all source S3 documents with `titan-embed-text-v2:0` (`512` dims), validate retrieval quality, and repoint the application's `knowledgeBaseId` configuration to the new Knowledge Base.
**Answer:** D  
**Explanation:** Different embedding models (`titan-embed-text-v1` at `1,536` dimensions vs. `titan-embed-text-v2:0` at `512` dimensions) produce incompatible vector spaces and incompatible vector lengths. Consequently, Amazon Bedrock treats `knowledgeBaseConfiguration.vectorKnowledgeBaseConfiguration.embeddingModelArn` as **immutable** after `CreateKnowledgeBase`, and OpenSearch `knn_vector` field `dimension` mappings cannot be altered in-place. Migrating to a new embedding model or dimension requires a **Blue/Green Knowledge Base & Index deployment**: provisioning a new `512`-dimension vector index and new Knowledge Base, running a full `StartIngestionJob` to re-embed the S3 corpus, and atomically switching the application pointer (or Agent Knowledge Base association) once ingestion completes.  
**Distractor Analysis:**
- *A is incorrect:* OpenSearch Serverless is fully managed and does not expose EC2 instances to customers, nor does restarting compute allow mutating a Lucene/Faiss `knn_vector` dimension mapping.
- *B is incorrect:* `titan-embed-text-v1` was not trained with Matryoshka Representation Learning, and vectors from `v1` and `v2` occupy completely different latent spaces—truncating `v1` vectors produces unusable similarity scores.
- *C is incorrect:* `amazon.titan-embed-text-v2:0` (`1024`, `512`, or `256` dimensions) is the primary recommended AWS embedding model for OpenSearch Serverless Knowledge Bases.
