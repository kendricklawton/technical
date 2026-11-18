# Domain 5 — Quiz 2: Troubleshooting RAG Pipelines, Vector Stores, Chunking Boundaries, Hallucinations & KB Ingestion Failures (Task 5.2)

---

### Question 1
An industrial manufacturing company has deployed an Amazon Bedrock Knowledge Base backed by Amazon OpenSearch Serverless (`VECTORSEARCH`) and `amazon.titan-embed-text-v2:0` (`1024` dimensions). Technicians query the assistant using exact 12-character alphanumeric spare-part SKUs (such as `"SKU-8841-ZX9"`) and PLC fault codes (such as `"FLT_0x0049A"`). While conceptual questions like *"How do I lubricate the primary turbine bearing?"* return accurate answers with high `ContextRelevance` (`0.92`), queries containing exact SKUs or hexadecimal fault codes consistently retrieve chunks for completely different parts (`ContextRelevance = 0.24`), causing the assistant to return inaccurate repair procedures.

What is the root cause of this retrieval failure, and what is the MOST effective architectural fix?

- **A.** The `amazon.titan-embed-text-v2:0` model truncates any token containing numbers or underscores; the developer must preprocess all PDFs with a Lambda function that converts numbers into spelled-out English words before ingestion.
- **B.** The generator model's `temperature` is set too low (`0.0`); increasing `temperature` to `0.8` will allow the foundation model to infer the correct SKU from neighboring vector clusters.
- **C.** The OpenSearch Serverless collection is missing a Network Security Policy; adding a VPC Endpoint will enable exact string matching on vector fields.
- **D.** Dense vector embeddings compress text into continuous semantic representations that lose exact character-level fidelity for out-of-vocabulary alphanumeric SKUs and fault codes; the developer should configure `overrideSearchType="HYBRID"` in `vectorSearchConfiguration` so OpenSearch Serverless combines BM25 lexical keyword matching with dense vector similarity.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
Pure semantic (dense vector) search maps text into a high-dimensional semantic space based on meaning. Rare alphanumeric identifiers (`"SKU-8841-ZX9"`, `"FLT_0x0049A"`) share similar contextual surroundings (e.g., *"Replace part SKU-XXXX when fault FLT_YYYY occurs"*), causing their dense vectors to cluster tightly together regardless of the exact digits. Enabling **Hybrid Search (`overrideSearchType="HYBRID"`)** combines dense vector k-NN search with BM25 lexical text search in OpenSearch Serverless, ensuring exact token matches on part numbers and fault codes are ranked at the top.

**Distractor Analysis:**
- **Why A is incorrect:** Embedding tokenizers do not strip numbers or underscores, and converting hexadecimal fault codes into English words destroys user query matching.
- **Why B is incorrect:** `temperature` only affects the generator LLM's output token sampling *after* retrieval has already failed; it cannot fix the retriever returning the wrong part manual.
- **Why C is incorrect:** AOSS Network Security Policies control network connectivity (public vs VPC endpoint access), not search ranking algorithms.
</details>

---

### Question 2
A pharmaceutical research team configures an Amazon Bedrock Knowledge Base to ingest 2,500 clinical trial protocol PDFs stored in an Amazon S3 bucket (`s3://pharma-protocols-prod/`). Both the S3 bucket and the Amazon OpenSearch Serverless vector collection are encrypted using AWS KMS Customer Managed Keys (CMKs). The developer attaches the `AmazonBedrockFullAccess` and `AmazonS3ReadOnlyAccess` managed policies plus `aoss:APIAccessAll` to the Knowledge Base IAM Execution Role.

However, when the developer starts the ingestion sync (`StartIngestionJob`), the job immediately transitions to `FAILED`. Inspecting `GetIngestionJob` reveals two errors across different data sources:
1. `AccessDeniedException` when attempting to read objects from `s3://pharma-protocols-prod/`.
2. `403 Forbidden: security_exception: Permission denied` when attempting to write vector embeddings into the OpenSearch Serverless index.

Which two actions must the developer take to resolve these ingestion failures? **(Choose TWO.)**

- **A.** Disable server-side encryption on the S3 bucket and recreate the OpenSearch Serverless collection as a `SEARCH` collection instead of a `VECTORSEARCH` collection.
- **B.** Update the Amazon OpenSearch Serverless **Data Access Policy** for the collection to explicitly include the Bedrock Knowledge Base IAM Execution Role ARN as a `Principal` with index (`aoss:CreateIndex`, `aoss:UpdateIndex`, `aoss:DescribeIndex`, `aoss:ReadDocument`, `aoss:WriteDocument`) and collection permissions.
- **C.** Add a Lambda Resource-Based Policy granting `bedrock.amazonaws.com` permission to invoke OpenSearch Serverless directly.
- **D.** Grant `kms:Decrypt` and `kms:DescribeKey` permissions on the S3 bucket's KMS Customer Managed Key to the Bedrock Knowledge Base IAM Execution Role (and ensure the KMS Key Policy permits the role).
- **E.** Change the S3 bucket ACL to `authenticated-read` and attach an IAM Service Control Policy (SCP) allowing `s3:*`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: B and D**

**Explanation:**
- **B is correct:** In Amazon OpenSearch Serverless, granting `aoss:APIAccessAll` in an IAM policy is necessary for IAM authorization, but AOSS enforces its own data-plane security boundary via **Data Access Policies**. If the Bedrock Knowledge Base IAM Role ARN is not listed as a `Principal` in the AOSS Data Access Policy with `aoss:WriteDocument`/`aoss:ReadDocument`/`aoss:DescribeIndex`, all writes fail with `403 Forbidden: security_exception`.
- **D is correct:** `AmazonS3ReadOnlyAccess` grants `s3:GetObject` and `s3:ListBucket`, which is sufficient for `SSE-S3` (AES-256) buckets. However, when an S3 bucket is encrypted with a **KMS Customer Managed Key (CMK)**, reading objects requires explicit `kms:Decrypt` permission in both the IAM Role policy and the KMS Key Policy.

**Distractor Analysis:**
- **Why A is incorrect:** Disabling encryption violates pharmaceutical compliance, and Bedrock Knowledge Bases strictly require an AOSS collection of type `VECTORSEARCH` (not `SEARCH`).
- **Why C is incorrect:** OpenSearch Serverless does not use Lambda resource policies.
- **Why E is incorrect:** S3 ACLs do not bypass KMS CMK decryption permissions or AOSS Data Access Policies.
</details>

---

### Question 3
A commercial real estate firm uses an Amazon Bedrock Knowledge Base to answer legal questions across 800 commercial lease agreements. Each lease contains a `"Rent Escalation & Default Remedies"` section structured as a 2-page clause where Paragraph 1 defines the base CPI escalation formula and Paragraph 6 (1,100 tokens later) defines a mandatory 4.5% annual cap for anchor tenants. The Knowledge Base is currently configured with **Fixed-Size Chunking** (`maxTokens: 300`, `overlapPercentage: 10`).

When leasing agents ask: *"What is the maximum annual CPI rent escalation for anchor tenants in the Metro Plaza lease?"*, the assistant answers: *"The rent escalates annually according to the Consumer Price Index (CPI) formula with no stated cap."* Inspecting the `Retrieve` trace shows that the retriever fetched Paragraph 1 (because it matched the keywords `"CPI"` and `"rent escalation"`), while Paragraph 6 (which only states *"Notwithstanding the foregoing, for Anchor Tenants the adjustment shall not exceed 4.5% per annum"*) was split into a separate chunk 4 chunks away that did not match the query's vector similarity.

Which architectural change will directly solve this context-splitting failure while maintaining high vector retrieval precision?

- **A.** Reduce the fixed chunk size from `300` tokens to `100` tokens and set `numberOfResults` to `1` so the retriever focuses on single sentences.
- **B.** Attach an Amazon Bedrock Guardrail Word Policy that blocks the phrase `"no stated cap"`.
- **C.** Switch the Knowledge Base data source chunking strategy to **Hierarchical Chunking** (`parentMaxTokens: 1500`, `childMaxTokens: 300`, `overlapTokens: 60`) and re-ingest the lease corpus so vector similarity matches the granular 300-token child chunk while returning the full 1,500-token parent clause (containing both Paragraph 1 and Paragraph 6) to the generator FM.
- **D.** Keep the 300-token fixed chunking strategy, do not re-ingest the documents, and simply change the generator model from `anthropic.claude-3-haiku-20240307-v1:0` to `anthropic.claude-3-5-sonnet-20241022-v2:0`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
This is the classic **Chunk Boundary / Context Fragmentation** problem in RAG:
- Small chunks (300 tokens) have high embedding precision because their vector represents a single focused concept, but they lack surrounding context (such as exceptions and caps stated several paragraphs below).
- Large chunks (1,500 tokens) preserve complete clauses and tables, but their averaged embedding vector is diluted across many sub-topics.
**Hierarchical Chunking** in Amazon Bedrock Knowledge Bases gives you the best of both worlds: it embeds 300-token child chunks for high-precision vector matching, but when a child chunk matches, it retrieves the encompassing 1,500-token parent chunk and passes that complete clause to the generator LLM.

**Distractor Analysis:**
- **Why A is incorrect:** Shrinking chunks to 100 tokens with $k=1$ worsens context fragmentation even further, guaranteeing Paragraph 6 is never included.
- **Why B is incorrect:** Blocking the phrase `"no stated cap"` via a Guardrail Word Policy does not provide the missing 4.5% cap to the generator; it merely blocks the output.
- **Why D is incorrect:** Even the most capable generator model (`Claude 3.5 Sonnet`) cannot cite the 4.5% cap if the retriever never includes Paragraph 6 in the prompt context.
</details>

---

### Question 4
An insurance company's RAG assistant answers questions about policy coverage using an Amazon Bedrock Knowledge Base (`RetrieveAndGenerate`). During a post-incident review of a complaint where a customer was given inaccurate deductible information, the engineering team inspects the `orchestrationTrace` and the Knowledge Base evaluation metrics:
- The `retrievedReferences` in the trace show that the retriever fetched the **exact correct paragraph** stating: *"Comprehensive glass coverage carries a mandatory $250 deductible in Florida and Massachusetts, and a $0 deductible in Arizona."*
- However, the generator foundation model (`temperature: 0.85`) responded to a Florida policyholder: *"Good news! Comprehensive glass replacements have a $0 deductible across all states, including Florida, provided you use a preferred repair shop."*
- Furthermore, the `"preferred repair shop"` condition does not appear anywhere in the retrieved chunks and was fabricated from the model's general pre-trained knowledge of auto insurance.

Which combination of changes will remediate this hallucination issue? **(Choose TWO.)**

- **A.** Lower the generator model's `temperature` to `0.0` in `generationConfiguration.inferenceConfig` and update the Knowledge Base generation prompt template with strict negative grounding constraints instructing the model to rely exclusively on `<search_results>` and never extrapolate from prior knowledge.
- **B.** Switch the OpenSearch Serverless vector index from `HNSW` to `FAISS` and increase `numberOfResults` from 5 to 25.
- **C.** Change the chunking strategy from Semantic Chunking to No Chunking (`NONE`) so each PDF is indexed as a single vector.
- **D.** Enable AWS WAF with an SQL Injection managed rule group in front of the Amazon API Gateway endpoint.
- **E.** Attach an **Amazon Bedrock Guardrail** configured with **Contextual Grounding Checks** (`GROUNDING` and `RELEVANCE` filters enabled with thresholds $\ge 0.85$) to the Knowledge Base `generationConfiguration` so any response containing claims not entailed by the retrieved source chunks is blocked and replaced.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and E**

**Explanation:**
The trace proves that **Retrieval succeeded 100%** (high `ContextRelevance` and `ContextCoverage`), meaning the fault lies entirely in the **Generation stage** (`Low Faithfulness / Hallucination`):
- **A is correct:** Lowering `temperature` from `0.85` to `0.0` forces greedy/deterministic token decoding, and hardening the prompt template prevents the LLM from blending pre-trained parametric knowledge (`"preferred repair shop"`) with the retrieved context.
- **E is correct:** Enabling **Amazon Bedrock Guardrails Contextual Grounding** (`GROUNDING` check) evaluates at runtime whether every factual statement in the generated output is strictly supported by the retrieved Knowledge Base chunks. Because `"$0 deductible across all states, including Florida"` contradicts the retrieved source text, the Grounding filter intervenes and blocks the hallucinated response.

**Distractor Analysis:**
- **Why B is incorrect:** The retriever already fetched the exact right paragraph; changing vector engine parameters or increasing $k$ to 25 only adds noise to the context window.
- **Why C is incorrect:** Changing chunking strategy addresses retrieval recall/precision, not generator hallucination when the right chunk is already present.
- **Why D is incorrect:** AWS WAF inspects HTTP network traffic for web exploits (SQLi/XSS), not semantic groundedness of LLM responses.
</details>

---

### Question 5
A data engineering team is building a multi-tenant SaaS Knowledge Base in Amazon Bedrock where documents from 50 different enterprise customers are stored in a single S3 bucket (`s3://saas-kb-docs/tenant_A/policy.pdf`, `s3://saas-kb-docs/tenant_B/policy.pdf`). To enforce tenant isolation and allow filtering by `tenant_id` and `department`, the team uploads a JSON metadata file alongside each PDF (for example, `s3://saas-kb-docs/tenant_A/policy.metadata.json`) with the following content:

```json
{
  "tenant_id": "tenant_A",
  "department": "finance"
}
```

After running `StartIngestionJob`, the team tests `Retrieve` with a metadata filter:
`filter={"equals": {"key": "tenant_id", "value": "tenant_A"}}`.
Surprisingly, `Retrieve` returns **zero results**, and when they run an unfiltered query, they discover that the raw JSON text `{ "tenant_id": "tenant_A" }` was chunked and embedded as a searchable document rather than being attached as filterable metadata attributes on `policy.pdf`!

Why did this happen, and how must the team fix it?

- **A.** Amazon Bedrock Knowledge Bases only support metadata files written in YAML format (`.metadata.yaml`), so any `.json` file is treated as a standard text document.
- **B.** Both the metadata filename and the JSON schema are incorrect: the file must be named `<exact_document_filename>.metadata.json` (`policy.pdf.metadata.json`, NOT `policy.metadata.json`), and the JSON payload must wrap all key-value pairs inside a top-level `"metadataAttributes"` object (`{"metadataAttributes": {"tenant_id": "tenant_A", "department": "finance"}}`).
- **C.** Metadata files must be stored in a separate S3 bucket from the source PDFs and linked via an Amazon DynamoDB global secondary index.
- **D.** The `equals` filter operator in Bedrock Knowledge Bases only supports integer values, not string values like `"tenant_A"`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
Amazon Bedrock Knowledge Base S3 metadata ingestion enforces two strict conventions:
1. **Naming Convention**: The sidecar file must be in the exact same S3 prefix and named `<source_filename_including_extension>.metadata.json`. For `policy.pdf`, the metadata file **must** be named `policy.pdf.metadata.json`. Because the team named it `policy.metadata.json` (dropping `.pdf`), Bedrock treated `policy.metadata.json` as an independent JSON document to be chunked and indexed!
2. **JSON Schema Convention**: The content of `.metadata.json` must contain a root `"metadataAttributes"` object:
```json
{
  "metadataAttributes": {
    "tenant_id": "tenant_A",
    "department": "finance"
  }
}
```

**Distractor Analysis:**
- **Why A is incorrect:** Bedrock Knowledge Bases require `.metadata.json` in JSON format, not YAML.
- **Why C is incorrect:** Metadata sidecar files must reside in the **same** S3 bucket and prefix as their corresponding source document.
- **Why D is incorrect:** The `equals` filter operator supports strings, numbers, and booleans.
</details>

---

### Question 6
A global logistics company is migrating its Amazon Bedrock Knowledge Base vector store to an existing Amazon Aurora PostgreSQL-Compatible Edition cluster using the `pgvector` extension. The database administrator created the vector table using the following DDL statement copied from an older project that used `amazon.titan-embed-text-v1`:

```sql
CREATE TABLE bedrock_integration.bedrock_kb (
    id uuid PRIMARY KEY,
    embedding vector(1536),
    chunks text,
    metadata jsonb
);
```

When the GenAI developer configures the new Knowledge Base in Amazon Bedrock using **`cohere.embed-multilingual-v3`** (or **`amazon.titan-embed-text-v2:0`** with default settings) and starts an ingestion job, the ingestion fails with a `ValidationException` / database vector dimension error.

What is the cause of this failure, and how should it be resolved?

- **A.** Amazon Aurora PostgreSQL `pgvector` only supports 512-dimensional vectors; the developer must switch to Amazon DynamoDB for 1,024-dimensional embeddings.
- **B.** The `metadata` column in Aurora PostgreSQL must be of type `xml` instead of `jsonb` for Bedrock Knowledge Base compatibility.
- **C.** `cohere.embed-multilingual-v3` outputs sparse BM25 vectors rather than dense floating-point vectors, which cannot be stored in PostgreSQL.
- **D.** Both `cohere.embed-multilingual-v3` and `amazon.titan-embed-text-v2:0` (default) generate **1,024-dimensional** vectors, whereas `amazon.titan-embed-text-v1` generated **1,536-dimensional** vectors; the administrator must alter or recreate the `pgvector` column as `embedding vector(1024)` and rebuild the HNSW index before running the ingestion job.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
Vector column dimensions in `pgvector` (and OpenSearch Serverless `knn_vector` mappings) must match the exact dimensionality of the selected embedding model:
- `amazon.titan-embed-text-v1`: `1536` dimensions
- `amazon.titan-embed-text-v2:0`: `1024` (default), `512`, or `256` dimensions
- `cohere.embed-english-v3` / `cohere.embed-multilingual-v3`: `1024` dimensions
Attempting to insert a 1,024-dimensional vector into a `vector(1536)` column triggers an immediate dimension mismatch error. Updating the table schema to `vector(1024)` fixes the error.

**Distractor Analysis:**
- **Why A is incorrect:** `pgvector` supports up to 2,000 dimensions (and higher with halfvec) for HNSW indexes, and DynamoDB is not a supported native vector store for Bedrock Knowledge Bases.
- **Why B is incorrect:** Bedrock Knowledge Bases explicitly require the `metadata` column in Aurora PostgreSQL to be `json` or `jsonb`.
- **Why C is incorrect:** `cohere.embed-multilingual-v3` outputs dense 1,024-dimensional floating-point vectors.
</details>

---

### Question 7
An energy utility's RAG application ingests scanned PDF engineering schematics, multi-column regulatory tariff tables, and embedded bar charts into an Amazon Bedrock Knowledge Base using the **Default text parser**. Users report that whenever a question requires reading a value from a multi-column tariff table or an embedded chart inside a PDF, the assistant either returns garbled numbers (reading across columns out of order) or claims the data does not exist. Text-only paragraphs in the same PDFs are answered accurately.

Which Knowledge Base configuration change will resolve the extraction accuracy for tables and charts without requiring custom OCR preprocessing microservices?

- **A.** Configure the Knowledge Base data source `parsingConfiguration` to use **Amazon Bedrock Data Automation (BDA)** or **Bedrock Foundation Model Parsing** (using a multimodal model such as `anthropic.claude-3-5-sonnet-20241022-v2:0` with custom parsing instructions to convert tables and charts into structured Markdown) and re-ingest the data source.
- **B.** Keep the Default text parser and change the vector search type from `SEMANTIC` to `HYBRID`.
- **C.** Increase the chunk overlap percentage on the Default text parser from 10% to 50%.
- **D.** Attach a Reranker model (`amazon.rerank-v1:0`) at query time so the garbled table strings are ranked higher.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
The **Default parser** in Amazon Bedrock Knowledge Bases performs standard text extraction, which cannot interpret visual charts/graphs and often flattens complex multi-column PDF layouts or merged-cell tables in the wrong reading order. Configuring **Amazon Bedrock Data Automation (BDA)** or **Foundation Model Parsing** (`parsingStrategy="BEDROCK_FOUNDATION_MODEL"` or `"BEDROCK_DATA_AUTOMATION"`) uses multimodal vision-language models during ingestion to visually inspect PDF pages, transcribe charts, and render complex tables into clean, semantically intact Markdown before chunking and embedding.

**Distractor Analysis:**
- **Why B, C, and D are incorrect:** Neither Hybrid Search, chunk overlap, nor query-time reranking can recover data that was destroyed or skipped during the initial PDF-to-text parsing step ("garbage in, garbage out").
</details>

---

### Question 8
A financial analyst assistant uses an Amazon Bedrock Knowledge Base containing quarterly earnings reports from 2023 through 2026. When an analyst asks a single-entity question like *"What was the EMEA operating margin in Q1 2026?"*, the assistant answers accurately. However, when an analyst asks a complex comparative question spanning multiple quarters and regions—such as *"How did the EMEA operating margin trend from Q1 2024 to Q1 2026 compare against the APAC operating margin over the same period, and which region achieved higher R&D efficiency?"*—the single vector lookup only retrieves chunks about EMEA 2026 and misses APAC 2024 and R&D efficiency chunks (`Builtin.ContextCoverage = 0.35`).

What is the MOST effective architectural solution to improve retrieval coverage for complex, multi-faceted prompts?

- **A.** Set `numberOfResults = 1` in `vectorSearchConfiguration` so the vector search focuses on the single highest-scoring chunk.
- **B.** Lower the embedding dimension of `amazon.titan-embed-text-v2:0` from `1024` to `256` so the vector representation becomes less specific.
- **C.** Enable **Query Decomposition** in Amazon Bedrock Knowledge Base (or orchestrate sub-queries via a Bedrock Agent) so the compound user prompt is automatically decomposed into atomic sub-queries (`"EMEA operating margin Q1 2024"`, `"EMEA operating margin Q1 2026"`, `"APAC operating margin Q1 2024 to Q1 2026"`, `"EMEA vs APAC R&D efficiency"`), executed in parallel against the retriever, and aggregated before generation.
- **D.** Add a Bedrock Guardrail Topic Policy blocking comparative questions.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
When a compound question asks about 4 distinct sub-topics across different regions and years, encoding the entire compound query into a single vector creates an "averaged" embedding vector that lands near one dominant sub-topic (or halfway between them), failing to retrieve the top chunks for each individual facet. **Query Decomposition** breaks a complex multi-hop or comparative prompt into independent atomic sub-queries, runs separate retrievals for each sub-query, and merges the retrieved chunks so all required facts are present in the generator's context window.

**Distractor Analysis:**
- **Why A is incorrect:** Setting `numberOfResults = 1` guarantees that at most one chunk is retrieved, making it impossible to answer a question spanning 4 different quarterly reports.
- **Why B is incorrect:** Reducing embedding dimensions from `1024` to `256` lowers vector precision and does not solve single-query vector averaging across multiple distinct documents.
- **Why D is incorrect:** Blocking comparative financial questions defeats the core business purpose of a financial analyst assistant.
</details>

---

### Question 9
An enterprise has connected an Amazon Bedrock Knowledge Base to a Confluence Cloud data source (`type="CONFLUENCE"`) to index internal engineering runbooks. During a scheduled nightly `StartIngestionJob`, the job completes with status `COMPLETE`, but inspecting `ingestionJob.statistics` reveals:
- `numberOfDocumentsScanned`: `1,420`
- `numberOfNewDocumentsIndexed`: `1,100`
- `numberOfDocumentsFailed`: `320`

The job status itself is `COMPLETE` (not `FAILED`), yet 320 Confluence pages are missing from search results. How should the engineer programmatically identify the exact 320 failed Confluence pages and their specific failure reasons?

- **A.** Call `bedrock-runtime.converse` and ask the foundation model to list which Confluence pages failed during last night's sync.
- **B.** Call the `bedrock-agent` API `ListKnowledgeBaseDocuments` (or inspect the CloudWatch Logs delivery stream configured for the Knowledge Base) filtering for documents with status `FAILED` or `PARTIALLY_INDEXED` to inspect the document-level `statusReason` for each failed URI.
- **C.** Check AWS Trusted Advisor Service Limits because `StartIngestionJob` never logs document-level errors.
- **D.** Delete and recreate the OpenSearch Serverless collection after every sync job.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
In Amazon Bedrock Knowledge Bases, an ingestion job only enters the terminal `FAILED` state if a global infrastructure error prevents the job from running (such as an invalid IAM role or unreachable vector store). When individual documents fail (for example, due to corrupted PDFs, password-protected files, oversized files exceeding quota, or connector rate limits), the job finishes as `COMPLETE` while incrementing `statistics.numberOfDocumentsFailed`. Engineers can inspect document-level status and `statusReason` using **`ListKnowledgeBaseDocuments` / `GetKnowledgeBaseDocuments`** and by configuring **Amazon CloudWatch Logs / S3 log delivery** on the Knowledge Base resource.

**Distractor Analysis:**
- **Why A is incorrect:** A foundation model invoked via `Converse` has no visibility into Bedrock control-plane ingestion job execution logs.
- **Why C is incorrect:** Trusted Advisor does not track document-level Knowledge Base ingestion failures.
- **Why D is incorrect:** Deleting the vector collection wipes out the 1,100 successfully indexed documents without diagnosing why the 320 documents failed.
</details>

---

### Question 10
A legal discovery platform uses an Amazon Bedrock Knowledge Base with `RetrieveAndGenerate`. The application requires every generated paragraph to include verifiable source citations (`citations[].retrievedReferences[].location.s3Location.uri`) that the frontend renders as clickable footnote links.

However, when the engineering team changes the `RetrieveAndGenerate` configuration by supplying a custom `promptTemplate` inside `generationConfiguration`, the frontend team reports that while the assistant still generates text answers, the `citations` array in the `RetrieveAndGenerate` API response is now completely empty (`[]`) on every request!

What caused the `citations` array to disappear, and how can the developer restore source citations while keeping custom prompt instructions?

- **A.** Custom prompt templates are incompatible with Amazon S3 data sources; the team must migrate the documents to Amazon Kendra.
- **B.** Citations are only returned when `temperature` is set to `1.0`.
- **C.** The developer must switch from `RetrieveAndGenerate` to the `Retrieve` API, because `Retrieve` automatically generates natural-language answers with footnote citations.
- **D.** The custom `promptTemplate` omitted or modified the mandatory `$output_format_instructions$` placeholder variable that Amazon Bedrock uses to instruct the generator FM on how to emit inline XML/JSON citation markers mapped to `retrievedReferences`; restoring `$output_format_instructions$` alongside `$search_results$` in the `promptTemplate` restores the `citations` array.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
In Amazon Bedrock Knowledge Bases (`RetrieveAndGenerate`), source attribution (`citations`) works by injecting special formatting instructions into the generation prompt via the reserved placeholder **`$output_format_instructions$`** (alongside `$search_results$`). When the LLM generates its response following `$output_format_instructions$`, Bedrock's post-processor parses the citation tags and populates the structured `citations` array in the API response. If a developer writes a custom `promptTemplate` and forgets to include `$output_format_instructions$`, the LLM generates plain text without citation markers, resulting in an empty `citations` list.

**Distractor Analysis:**
- **Why A is incorrect:** Custom prompt templates fully support S3 data sources and citations when `$output_format_instructions$` is preserved.
- **Why B is incorrect:** `temperature` does not enable or disable the `citations` response field.
- **Why C is incorrect:** The `Retrieve` API only returns raw vector chunks (`retrievalResults`); it does not invoke an LLM or generate natural-language answers.
</details>

---

### Question 11
A medical device manufacturer stores user manuals for 15 different ventilator models (`Model-V100` through `Model-V250`) in an Amazon Bedrock Knowledge Base. Every chunk inside a 200-page manual simply refers to the device as `"the ventilator"` or `"the unit"`, while the model name `"Model-V180"` only appears on the cover page (Page 1) of the PDF.

When a clinician asks: *"What is the high-pressure alarm threshold for the Model-V180 ventilator?"*, vector search retrieves chunks from Page 85 of the `Model-V120` and `Model-V200` manuals because those chunks contain the exact phrase `"high-pressure alarm threshold for the ventilator"`, whereas Page 85 of the `Model-V180` manual never repeats the string `"Model-V180"`.

Which two solutions can the developer implement so that document-level context (`"Model-V180"`) is preserved across every chunk on every page? **(Choose TWO.)**

- **A.** Switch the chunking strategy to Fixed-Size Chunking with `maxTokens: 20` and `overlapPercentage: 1`.
- **B.** Disable vector embeddings and use Amazon S3 Select to grep for the word `"threshold"`.
- **C.** Create `.metadata.json` sidecar files in S3 containing `"device_model": "Model-V180"` with `inclusionForEmbedding` / implicit metadata filtering (or explicit `retrievalFilter` extracted from the query) so only chunks belonging to `Model-V180` are searched.
- **D.** Set `maxTokens` on the generator model to `4096`.
- **E.** Implement **Contextual Retrieval (Chunk Header Injection)** via a Custom Transformation Lambda function (`customTransformationConfiguration` in `vectorIngestionConfiguration`) that prepends document-level context (`"[Document: Model-V180 Ventilator Manual | Section: Alarms]"`) to every chunk before it is embedded and indexed.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: C and E**

**Explanation:**
This scenario illustrates **Document-Level Context Loss** in individual chunks:
- **C is correct:** Attaching `"device_model": "Model-V180"` in `manual_v180.pdf.metadata.json` allows the application (or an Agent / Implicit Filter) to apply a strict `equals` filter (`{"equals": {"key": "device_model", "value": "Model-V180"}}`), eliminating cross-model chunk collisions.
- **E is correct:** Amazon Bedrock Knowledge Bases support **Custom Transformation Lambda functions** during ingestion (`STEP_POST_CHUNKING`). By prepending a contextual header (`"Device Model: Model-V180 | ..."`) to the text of every chunk prior to embedding, both the dense vector embedding and the BM25 lexical index for Page 85 explicitly contain `"Model-V180"`.

**Distractor Analysis:**
- **Why A is incorrect:** 20-token chunks destroy context further and still lack the string `"Model-V180"` on Page 85.
- **Why B is incorrect:** S3 Select cannot query PDF files semantically across 15 manuals, nor is it integrated into Bedrock Knowledge Base vector retrieval.
- **Why D is incorrect:** Generator `maxTokens` controls output length; it does not affect which chunks the retriever fetches.
</details>

---

### Question 12
A telecommunications company uses an Amazon Bedrock Knowledge Base with Amazon OpenSearch Serverless. During a routine audit, engineers notice that whenever an outdated promotional tariff PDF is **deleted** from the source Amazon S3 bucket and `StartIngestionJob` is executed, the ingestion job succeeds (`COMPLETE`), yet customers asking about promotions still receive answers citing the deleted PDF from the OpenSearch Serverless vector index!

Upon inspecting the Data Source configuration in `GetDataSource`, the engineer sees:
```json
"dataDeletionPolicy": "RETAIN"
```

Why did the deleted S3 documents remain in the vector index, and what is the behavior difference between `RETAIN` and `DELETE`?

- **A.** When `dataDeletionPolicy` is set to `"RETAIN"`, Bedrock preserves vector embeddings in the vector store when source documents are deleted from S3 (or when the data source itself is deleted); updating `dataDeletionPolicy` to `"DELETE"` and running `StartIngestionJob` (or deleting the document via `DeleteKnowledgeBaseDocuments`) instructs Bedrock to purge vector embeddings corresponding to deleted S3 objects.
- **B.** OpenSearch Serverless collections are append-only immutable ledgers that do not support document deletion under any configuration.
- **C.** `StartIngestionJob` only indexes new files; the developer must manually run `DROP TABLE` in OpenSearch Dashboards every night.
- **D.** The `RETAIN` policy only applies to AWS CloudTrail logs, and the old answers are coming from the generator model's pre-training weights.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
In Amazon Bedrock Knowledge Bases, the Data Source `dataDeletionPolicy` parameter controls how deletions are handled:
- **`DELETE` (Default)**: When a file is deleted from the S3 source bucket and an ingestion sync runs (or when the data source is deleted), Bedrock deletes the corresponding vector chunks from the vector store.
- **`RETAIN`**: Bedrock retains the vector embeddings in the underlying vector database even after the source S3 object or data source is removed. Changing `dataDeletionPolicy` back to `"DELETE"` (or invoking `DeleteKnowledgeBaseDocuments`) cleans up the orphaned vectors.

**Distractor Analysis:**
- **Why B is incorrect:** OpenSearch Serverless `VECTORSEARCH` collections fully support document updates and deletions (`aoss:WriteDocument`).
- **Why C is incorrect:** `StartIngestionJob` tracks document state deltas (new, modified, deleted) automatically when `dataDeletionPolicy="DELETE"`.
- **Why D is incorrect:** `dataDeletionPolicy` is a Bedrock Knowledge Base Data Source parameter governing vector store lifecycle, not CloudTrail logs.
</details>

---

### Question 13
A software company indexes 10,000 Python and TypeScript source code files from an internal repository into an Amazon Bedrock Knowledge Base so developers can ask architectural questions. Initially, the team used **Semantic Chunking** (`bufferSize: 1`, `breakpointPercentileThreshold: 95`). Developers report that while natural-language Markdown README files are chunked cleanly, source code files are frequently split right in the middle of class methods or JSON configuration blocks, causing syntax context to be lost.

Why is Semantic Chunking suboptimal for structured source code files, and what should the team use instead?

- **A.** Semantic Chunking only supports Spanish and French text; switching to `amazon.titan-embed-image-v1` will fix Python code chunking.
- **B.** Semantic Chunking encrypts source code in Base64 before sending it to the generator LLM.
- **C.** Semantic Chunking computes sentence-to-sentence embedding cosine similarity based on natural-language punctuation boundaries (`.` `?` `!`), which splits source code arbitrarily at periods (`object.method()`) or fails to respect AST/function block boundaries; the team should use a **Custom Transformation Lambda function** (`STEP_POST_CHUNKING` or custom chunking logic) that splits code files along language-aware syntax boundaries (classes, functions, methods) or Markdown headers.
- **D.** The `breakpointPercentileThreshold` must be set to `0` so every single character becomes its own chunk.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
**Semantic Chunking** works by splitting text into sentences, computing dense embeddings for sliding sentence windows (`bufferSize`), and inserting a chunk boundary wherever cosine similarity drops below the `breakpointPercentileThreshold`. While effective for prose articles, source code contains dot-notation (`client.invoke_model()`), braces, and indentation rather than natural sentences. Using a **Custom Chunking / Transformation Lambda** in the Knowledge Base ingestion pipeline allows parsing source code with AST/tree-sitter or syntax-aware splitters so complete functions and classes stay intact.

**Distractor Analysis:**
- **Why A is incorrect:** Semantic chunking works with any language supported by the text embedding model, and `titan-embed-image-v1` is for multimodal image embeddings, not source code chunking.
- **Why B is incorrect:** Semantic chunking does not Base64-encode text chunks.
- **Why D is incorrect:** Setting a threshold that turns every sentence/fragment into a tiny isolated chunk destroys all code context.
</details>

---

### Question 14
An enterprise RAG system uses `RetrieveAndGenerate` with an Amazon Bedrock Guardrail configured with a Contextual Grounding policy:
- `GROUNDING` threshold: `0.99`
- `RELEVANCE` threshold: `0.99`

After deploying this guardrail configuration, customer support managers complain that 85% of valid user requests—such as *"Summarize our 3-page return shipping guide into 3 bullet points"*—are being blocked and replaced with the Guardrail blocked message (`"Sorry, I cannot answer this question"`), even though inspecting the trace shows the model's 3-bullet summary was 100% accurate and faithful to the retrieved return shipping guide!

Why is the Guardrail blocking valid abstractive summaries, and how should the developer tune it?

- **A.** Contextual Grounding checks cannot be used with `RetrieveAndGenerate`; the developer must disable Contextual Grounding completely and use a PII Regex filter instead.
- **B.** Setting the Contextual Grounding `GROUNDING` and `RELEVANCE` thresholds to `0.99` requires near-verbatim extractive copying from the source chunks; valid abstractive summarization and paraphrasing naturally score between `0.75` and `0.92`, so lowering the thresholds to a calibrated range (such as `0.80`–`0.85` validated against a golden test set) eliminates false-positive blocks while still catching true hallucinations.
- **C.** The `GROUNDING` score in Bedrock Guardrails is inverted (where `0.0` means grounded and `1.0` means hallucinated); the developer should set the threshold to `0.01`.
- **D.** Summarization queries trigger the Guardrail's `HATE` and `VIOLENCE` content filters by default.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
In Amazon Bedrock Guardrails Contextual Grounding, the `score` ($0.0$ to $1.0$) represents the confidence that the response is grounded in the source (`GROUNDING`) and relevant to the query (`RELEVANCE`), and the Guardrail blocks the output whenever `score < threshold`. Setting the threshold to an extreme `0.99` creates an ultra-strict gate that rejects almost any abstractive synthesis, bulleted condensation, or phrasing change that isn't a near-verbatim quote. Tuning the threshold to `0.75`–`0.85` using an evaluation dataset balances hallucination prevention (Precision) against false-positive blocking (Recall).

**Distractor Analysis:**
- **Why A is incorrect:** Contextual Grounding is specifically designed for RAG (`RetrieveAndGenerate` and `ApplyGuardrail`).
- **Why C is incorrect:** Higher scores ($1.0$) indicate *higher* grounding confidence, not higher hallucination; the Guardrail blocks when `score < threshold`.
- **Why D is incorrect:** Inspecting `guardrailTrace` would show `contextualGroundingPolicy` triggered the block, not `contentPolicy`.
</details>

---

### Question 15
A healthcare provider uses an Amazon Bedrock Knowledge Base backed by an Amazon S3 bucket containing 50,000 clinical guideline documents. Doctors report that when they ask a question, the `RetrieveAndGenerate` API call takes **11.5 seconds**, violating their 4-second clinical UI SLA.

The developer profiles the pipeline and discovers the following configuration in `RetrieveAndGenerate`:
- `numberOfResults`: `40`
- Chunking strategy: Hierarchical Chunking (`parentMaxTokens: 2000`, `childMaxTokens: 400`)
- No Reranker configured
- Generator model: `anthropic.claude-3-opus-20240229-v1:0` (or a large reasoning model) receiving $40 \times 2,000 = 80,000$ input tokens of context on every query!

Which combination of architectural optimizations will bring latency under the 4-second SLA while preserving answer quality?

- **A.** Reduce initial vector retrieval `numberOfResults` to `15`–`20`, add an **Amazon Bedrock Reranker** (`cohere.rerank-v3-5:0` or `amazon.rerank-v1:0`) with `numberOfRerankedResults: 4` so only $4 \times 2,000 = 8,000$ tokens are sent to the generator FM, and switch to a lower-latency generator model (`anthropic.claude-3-5-sonnet-20241022-v2:0` or `amazon.nova-pro-v1:0`) with response streaming (`RetrieveAndGenerateStream`).
- **B.** Increase `numberOfResults` from `40` to `100` so the vector store does not have to sort the top results.
- **C.** Remove all indexes from the vector database so it performs an exact brute-force scan across all 50,000 documents.
- **D.** Keep sending 80,000 input tokens per query, disable streaming, and increase `maxTokens` to `8192`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
Passing 40 parent chunks of 2,000 tokens each stuffs **80,000 input tokens** into the generator LLM on every single query. Because LLM Time-To-First-Token (TTFT) scales super-linearly/linearly with input context length, prefill processing of 80,000 tokens on a heavyweight model consumes 8–10+ seconds before a single output token is generated. By retrieving 15–20 candidates and using a fast cross-encoder **Reranker (`numberOfRerankedResults: 4`)** to filter down to the 4 most relevant parent chunks (cutting generator input tokens by 90% from 80k to 8k), switching to a faster generator model (`Claude 3.5 Sonnet` or `Nova Pro`), and using **`RetrieveAndGenerateStream`**, TTFT drops below 1.5 seconds.

**Distractor Analysis:**
- **Why B is incorrect:** Increasing `numberOfResults` to 100 would exceed the context window and worsen prefill latency.
- **Why C is incorrect:** Removing HNSW/IVFFlat vector indexes forces an $O(N)$ linear scan, increasing retrieval latency.
- **Why D is incorrect:** Disabling streaming and keeping 80,000 input tokens maintains the 11.5-second latency bottleneck.
</details>
